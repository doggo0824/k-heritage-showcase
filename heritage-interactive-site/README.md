# 🏛️ 국가유산 인터랙티브 체험관

GitHub Pages 기반 8모듈 인터랙티브 국가유산 체험 웹사이트

## 📦 배포 방법

1. 이 폴더 전체를 GitHub 리포지토리에 push
2. Settings → Pages → Source: `main` branch, `/ (root)` 선택
3. 수 분 후 `https://username.github.io/repo-name/` 에서 접속 가능

## 📁 폴더 구조

```
/
├── index.html              ← 메인 허브 (웹캠 배경 + 8개 카드)
├── README.md
├── assets/
│   ├── patterns/           ← 단청 문양 이미지 (PNG/JPG) 넣는 곳
│   └── sounds/             ← 전통 악기 음원 (MP3/WAV/OGG) 넣는 곳
└── pages/
    ├── 1-dancheong.html    ← 단청 물결
    ├── 2-crt-heritage.html ← 국가유산 CRT
    ├── 3-mudra.html        ← 색즉시空
    ├── 4-incredibox.html   ← 전통 인크레디박스
    ├── 5-photobooth.html   ← 전통문양 인생네컷
    ├── 6-historian.html    ← 사관 LLM
    ├── 7-conservation.html ← 보존과학 체험
    └── 8-interactive.html  ← 인터랙티브 국가유산
```

## 🎨 사용자 에셋 업로드

### 단청 문양 (Feature 1)
- 페이지 내 왼쪽 패널에서 **드래그 앤 드롭** 또는 **클릭하여 업로드**
- 지원 형식: PNG, JPG, WebP
- 여러 장 업로드 가능, 자동으로 물 위에 배치됨

### 전통 악기 음원 (Feature 4)
- 각 악기 카드를 **클릭하여 음원 파일 업로드**
- 지원 형식: MP3, WAV, OGG
- 업로드 후 우클릭으로 슬롯에 추가

### 3D 모델 (Feature 2)
- GLB/GLTF 파일 로드 지원 (코드 내 fileInput)

---

## 🔑 API 키 관리 & 보안 가이드

### 현재 방식 (클라이언트 전용)
- API 키는 **`sessionStorage`** 에만 저장됩니다
- **탭을 닫으면 자동 삭제**됩니다
- GitHub에 push되는 코드에 키가 포함되지 않습니다
- 사용하는 기능: Feature 6 (사관 LLM), Feature 8 (DALL-E 이미지)

### ⚠️ 주의사항
클라이언트에서 직접 OpenAI API를 호출하면:
- 브라우저 개발자 도구(Network 탭)에서 API 키가 노출될 수 있음
- 공용 PC에서는 사용 후 반드시 "삭제" 버튼 클릭
- 전시 환경에서는 아래 프록시 방식 권장

### ✅ 프로덕션 배포 시 추천 방식

#### 방법 1: Cloudflare Workers 프록시 (무료, 추천)
```javascript
// worker.js (Cloudflare Workers에 배포)
export default {
  async fetch(request, env) {
    const url = new URL(request.url);

    // CORS
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': 'https://yourdomain.github.io',
          'Access-Control-Allow-Methods': 'POST',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }

    // 프록시: 클라이언트 → Worker → OpenAI
    const body = await request.json();
    const resp = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${env.OPENAI_API_KEY}` // 환경변수에서 로드
      },
      body: JSON.stringify(body)
    });

    const data = await resp.json();
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': 'https://yourdomain.github.io'
      }
    });
  }
}
```

설정:
1. Cloudflare Workers 대시보드에서 Worker 생성
2. 환경변수에 `OPENAI_API_KEY` 추가
3. 클라이언트 코드에서 fetch URL을 Worker URL로 변경

#### 방법 2: Vercel Edge Functions
```javascript
// api/openai.js (Vercel 프로젝트에 배포)
export const config = { runtime: 'edge' };

export default async function handler(req) {
  const body = await req.json();
  const resp = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
    },
    body: JSON.stringify(body)
  });
  return new Response(await resp.text(), {
    headers: { 'Content-Type': 'application/json' }
  });
}
```

#### 방법 3: GitHub Actions + Secrets
- Repository Secrets에 API 키 저장
- GitHub Actions로 빌드 시 config.js 파일 생성
- 단점: 빌드 시점에 주입되므로 public 리포에서는 여전히 노출 가능

### 🛡️ 추가 보안 조치
1. **API 키에 사용량 제한** 설정 (OpenAI 대시보드)
2. **Allowed origins** 설정 (프록시에서 특정 도메인만 허용)
3. **Rate limiting** 적용 (프록시에서 분당 요청 수 제한)
4. API 키에 **최소 권한** 부여 (Chat + Images만)

---

## 🌐 다국어 지원
- 한국어 (기본), English, 日本語, 中文
- 메인 화면 우상단 언어 선택 버튼

## 🖥️ 기술 스택
- MediaPipe Hands / Face Mesh
- Web Audio API
- Three.js (3D 렌더링)
- Canvas 2D
- OpenAI GPT-4o-mini / DALL-E 3
