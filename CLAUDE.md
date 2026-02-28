# .NET Universe 2026 Banner

.NET Universe 2026 컨퍼런스 현수막 디자인 프로젝트 (5000mm x 900mm 인쇄용)

## 프로젝트 구조

```
banner-2026.pen       # Pencil Dev 원본 디자인 파일 (18898x3402px)
banner-2026.html      # HTML 재현 파일 (1/2 스케일: 9449x1701px)
banner-2026.png       # 최종 출력 PNG (18898x3402px)
reference/            # 이미지 리소스
  dotnetdev-logo.png  # 좌측 dotnetdev 커뮤니티 로고
  dotnet_hero_transparent.png  # 우측 닷넷 히어로 마스코트 (투명 배경)
  dotnet_hero.svg     # 닷넷 히어로 원본 SVG
  2026-hero.webp      # 원본 히어로 이미지 (봇 크롭 소스)
  bot-{color}-circle.png  # 6개 닷넷봇 원형 아바타 (purple, red, blue, green, yellow, orange)
  sponsors/           # 후원사 로고 (aws, enterpriseDB, infragistics, openboxlab)
```

## PNG 생성 방법 (banner-2026.png)

`banner-2026.html`은 .pen 파일의 디자인을 HTML/CSS로 1/2 스케일(9449x1701px)로 재현한 파일입니다.
Playwright의 `deviceScaleFactor=2`를 사용하여 정확히 18898x3402px PNG를 생성합니다.

### 사전 준비

```bash
npm install playwright
npx playwright install chromium
```

### 캡처 스크립트 (capture.js)

```javascript
const { chromium } = require('playwright');
const path = require('path');

(async () => {
  const browser = await chromium.launch();
  const context = await browser.newContext({
    viewport: { width: 9449, height: 1701 },
    deviceScaleFactor: 2,
  });
  const page = await context.newPage();

  // 로컬 HTTP 서버 또는 file:// 프로토콜 사용
  // file:// 프로토콜이 차단될 경우 아래 HTTP 서버 방식 사용
  const http = require('http');
  const fs = require('fs');
  const mimeTypes = {
    '.html': 'text/html', '.png': 'image/png',
    '.jpg': 'image/jpeg', '.webp': 'image/webp',
    '.svg': 'image/svg+xml', '.css': 'text/css',
  };

  const server = http.createServer((req, res) => {
    const filePath = path.join(__dirname, decodeURIComponent(req.url));
    const ext = path.extname(filePath);
    fs.readFile(filePath, (err, data) => {
      if (err) { res.writeHead(404); res.end(); return; }
      res.writeHead(200, { 'Content-Type': mimeTypes[ext] || 'application/octet-stream' });
      res.end(data);
    });
  });

  server.listen(8888, async () => {
    await page.goto('http://localhost:8888/banner-2026.html', { waitUntil: 'networkidle' });
    await page.waitForTimeout(2000); // 폰트 로딩 대기

    const banner = await page.$('.banner');
    await banner.screenshot({ path: 'banner-2026.png', type: 'png' });

    console.log('banner-2026.png 생성 완료 (18898x3402px)');
    server.close();
    await browser.close();
  });
})();
```

### 실행

```bash
node capture.js
```

출력: `banner-2026.png` (18898 x 3402px, 약 4MB)

## 인쇄 사양

- 출력 크기: 5000mm x 900mm
- 해상도: 18898 x 3402px (약 96 DPI)
- 현수막 인쇄 기준 적합 (일반적으로 72~150 DPI 권장)

## HTML 수정 시 주의사항

- `banner-2026.html`의 모든 CSS 치수는 .pen 파일 원본의 **1/2 스케일**
- 예: .pen에서 750px 원형 → HTML에서 375px
- `deviceScaleFactor: 2`로 캡처하여 원본 크기 복원
- 이미지 경로는 `./reference/` 상대경로 사용
