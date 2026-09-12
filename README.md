# VK API Token Helper

VK 커뮤니티(그룹) 자동화를 위한 **User Access Token 발급 도구**입니다.
GitHub Pages에 그대로 올려서 VK Mini App으로 실행하면 됩니다.

- 빌드 과정 없음 (Node.js, npm 불필요)
- 서버 없음, 데이터베이스 없음 — 100% static
- 공식 [`@vkontakte/vk-bridge`](https://github.com/VKCOM/vk-bridge) v3.0.2 사용 (CDN)

> ⚠️ **Never commit your VK access token or client secret.**
> 발급받은 토큰은 이 저장소에, 그리고 어떤 Git 저장소에도 절대 커밋하지 마세요.

---

## 1. 무엇을 하는 도구인가

VK Mini App 안에서 실행되면:

1. `VKWebAppInit` 으로 VK Bridge를 초기화하고
2. URL 쿼리 파라미터 `vk_app_id` 에서 App ID를 읽고 (코드에 하드코딩하지 않음)
3. 버튼을 누르면 `VKWebAppGetAuthToken` 으로 `wall,groups,photos` 권한을 요청하고
4. 발급된 User Access Token을 화면에 표시 + 복사할 수 있게 해줍니다.
5. (선택) `VKWebAppCallAPIMethod` 로 `users.get` 을 호출해 토큰이 실제로 동작하는지 확인합니다.

### 요청하는 권한 (scope)

| scope | 용도 |
|---|---|
| `wall` | 그룹 게시물 작성 / 예약 게시 |
| `groups` | 그룹 관련 API |
| `photos` | 게시물 이미지 업로드 |

나중에 댓글 처리에 `wall` 외 권한이 더 필요해지면 `index.html` 상단의
`var SCOPE = "wall,groups,photos";` 값만 바꾸면 됩니다.

---

## 2. 파일 구성

```text
vk-token-helper/
├── index.html    # 전체 앱 (HTML + CSS + JS 한 파일)
├── README.md
└── .gitignore
```

---

## 3. GitHub에 올리고 Pages 켜기

### 3-1. 저장소 만들고 push

```bash
cd vk-token-helper
git init
git add .
git commit -m "VK API Token Helper"
git branch -M main
git remote add origin https://github.com/USERNAME/vk-token-helper.git
git push -u origin main
```

### 3-2. GitHub Pages 활성화

```text
GitHub Repository
→ Settings
→ Pages
→ Source: Deploy from a branch
→ Branch: main
→ Folder: / (root)
→ Save
```

1~2분 뒤 아래 주소에서 열립니다.

```text
https://USERNAME.github.io/vk-token-helper/
```

> 이 주소를 브라우저에서 그냥 열면 `vk_app_id를 찾을 수 없습니다` 라고 나오는 게 **정상**입니다.
> VK 내부에서 실행돼야 App ID가 URL로 전달되기 때문입니다.

---

## 4. VK Developers 설정 (Настройки → Размещение)

VK Developers → 내 앱 → **Настройки** → **Размещение** 으로 갑니다.

여기에 세 가지 영역이 있습니다.

| 러시아어 | 뜻 |
|---|---|
| `Мобильное приложение` | 모바일 **앱**(iOS/Android 네이티브 클라이언트)용 |
| `Десктопная версия сайта` | **데스크톱 웹** vk.com 용 |
| `Мобильная версия сайта` | **모바일 웹** m.vk.com 용 |

### 우선 데스크톱만 설정하면 됩니다 (테스트 목적)

```text
Десктопная версия сайта

URL:
https://USERNAME.github.io/vk-token-helper/

Режим разработки:
Включено          ← 개발 모드 켜기
```

- **URL** — GitHub Pages 주소를 그대로 붙여넣습니다. 끝의 `/` 를 포함하세요.
- **Режим разработки (개발 모드)** — `Включено`(켬)로 두면 앱이 공개 심사 없이
  **앱 관리자(본인) 계정에서만** 실행됩니다. 토큰 발급 용도로는 이 상태가 맞습니다.

저장 후 `Состояние для пользователей`(사용자 공개 상태)는 테스트 중에는
공개(`Включено`)로 바꾸지 않아도 됩니다. 본인 계정으로는 개발 모드로 실행됩니다.

### 모바일에서도 쓰고 싶다면

`Мобильная версия сайта` 와 `Мобильное приложение` 의 URL 에도
**똑같은 GitHub Pages 주소**를 넣고 개발 모드를 켜면 됩니다.
(세 곳 모두 같은 URL을 써도 문제없습니다.)

---

## 5. 실행 & 토큰 발급

1. VK Developers 앱 페이지에서 **"Запустить приложение"(앱 실행)** 을 누르거나
   `https://vk.com/app{APP_ID}` 주소로 접속합니다.
2. 화면에 아래처럼 표시되면 정상입니다.

   ```text
   VK Mini App detected
   App ID: 54767220
   요청할 권한: wall, groups, photos
   ```

3. **`VK 권한 승인 및 Token 발급`** 버튼 클릭 → VK 권한 승인 창에서 승인.
4. 토큰이 textarea에 표시되고, 실제 승인된 권한이 함께 나옵니다.

   ```text
   Granted scopes: wall, groups, photos
   ```

5. **`Token 복사`** 버튼으로 복사합니다. (`Token copied.` 표시)
6. (선택) **`VK API 연결 테스트`** 로 토큰이 실제 동작하는지 확인합니다.

   ```text
   API 연결 성공
   User ID: 12345678
   Name: Sam Kim
   ```

---

## 6. 보안 — 이 앱이 지키는 규칙

토큰은 **브라우저 메모리(JS 변수)에만** 잠깐 존재합니다.

- ❌ localStorage / sessionStorage / cookie 저장 안 함
- ❌ 외부 서버 전송 안 함 (fetch/XHR 없음)
- ❌ analytics 없음
- ❌ `console.log(token)` 없음
- ❌ client secret / service token 사용 안 함, 코드에 하드코딩 없음
- ✅ **페이지를 새로고침하면 토큰이 즉시 사라집니다**
- ✅ 에러 메시지에는 `error_type` / `error_code` / `reason` 만 표시하고 토큰은 넣지 않음
- ✅ `Token 지우기` 버튼으로 즉시 메모리에서 제거 가능

### 복사한 토큰은 어떻게 보관해야 하나

Python 자동화 스크립트를 만들 때는 **코드에 직접 적지 말고** 환경변수로 넘기세요.

```bash
export VK_USER_TOKEN="발급받은_토큰"
```

```python
import os
import requests

TOKEN = os.environ["VK_USER_TOKEN"]   # 코드에 하드코딩 금지
API_VERSION = "5.131"

r = requests.get(
    "https://api.vk.com/method/users.get",
    params={"access_token": TOKEN, "v": API_VERSION},
    timeout=10,
)
print(r.json())
```

`.env` 파일을 쓴다면 이 저장소의 `.gitignore` 가 이미 막아두었습니다.

> **Never commit your VK access token or client secret.**

---

## 7. 오류 메시지 대응표

| 화면 메시지 | 원인 / 해결 |
|---|---|
| `vk_app_id를 찾을 수 없습니다` | VK 밖에서 GitHub Pages 주소를 직접 열었습니다. `https://vk.com/app{APP_ID}` 로 실행하세요. |
| `VK Bridge를 불러오지 못했습니다` | CDN 로딩 실패. 네트워크 확인 후 새로고침. |
| `VKWebAppInit 실패` | VK 클라이언트 밖에서 실행 중이거나 Mini App 설정 문제. Размещение의 URL이 실제 Pages 주소와 정확히 일치하는지 확인. |
| `VK 권한 승인이 취소되었거나 거부되었습니다` | 승인 창에서 거부/닫기를 누른 경우. 버튼을 다시 누르면 됩니다. |
| `일부 권한이 승인되지 않았습니다` | 누락된 scope가 표시됩니다. 다시 발급하면서 전부 승인하세요. |
| `Token 발급에 실패했습니다` + `error_type` | VK Bridge가 반환한 코드입니다. 앱 설정(도메인/개발 모드)을 먼저 확인하세요. |

---

## 8. 기술 메모

- VK Bridge: `https://unpkg.com/@vkontakte/vk-bridge@3.0.2/dist/browser.min.js`
  (실패 시 jsDelivr로 자동 fallback). 전역 객체는 `window.vkBridge`.
  v3.0.1 에서 CDN(IIFE) 번들의 변수 충돌 버그가 수정되었으므로 **3.0.2 이상**을 쓰세요.
- `VKWebAppGetAuthToken` 요청: `{ app_id: number, scope: string }`
  응답: `{ access_token, scope, expires? }`
- `VKWebAppCallAPIMethod` 요청: `{ method, params }` — `params` 에 **`access_token` 과 `v` 가 반드시 포함**되어야 합니다.
- VK API 버전은 `index.html` 의 `VK_API_VERSION = "5.131"` 에서 변경할 수 있습니다.
- 에러 형식: `{ error_type: 'client_error' | 'api_error' | 'auth_error', error_data: { error_code, error_reason | error_msg, error_description } }`
- 복사는 `VKWebAppCopyText`(VK 네이티브) → `navigator.clipboard` → `document.execCommand('copy')` →
  수동 선택 안내 순으로 자동 폴백합니다. VK iframe에서 클립보드 권한이 막혀도 멈추지 않습니다.

---

## 9. Standalone 토큰 (그룹 설정 변경용)

`groups.edit`, `groups.getSettings` 등 **그룹 관리 메서드는 Standalone 앱 토큰에서만** 동작한다.
Mini App 토큰으로 호출하면 이렇게 거부된다:

```text
error_code=15  Permission to perform this action is denied for non-standalone applications
```

이건 scope 문제가 아니라 **앱 타입** 문제다. 해결하려면:

1. VK Developers에서 앱을 하나 더 만들되 타입을 **`Standalone-приложение`** 으로 선택
2. `standalone.html` 페이지를 연다 → https://movingredstone.github.io/vk-token-helper/standalone.html
3. Standalone App ID 입력 → 권한 승인 → 이동한 빈 페이지의 **주소창을 통째로 복사** → 붙여넣기

Standalone은 VK Bridge가 아니라 implicit OAuth를 쓴다. 토큰이 리다이렉트 URL의 `#` 뒤에 실려오는데,
`oauth.vk.com/blank.html` 은 우리 도메인이 아니라 JS로 읽을 수 없다. 그래서 주소창을 복사해 붙여넣는 방식이다.

`offline` 권한을 켜면 토큰이 만료되지 않는다. 자동화에는 편하지만 유출 시 계속 유효하니 주의.

## 10. 다음 단계 (자동화)

이 토큰으로 이후 구현할 수 있는 것들:

- `wall.post` — 그룹 자동 게시 (`owner_id` 는 그룹 ID에 `-` 를 붙인 음수)
- `wall.post` + `publish_date` — 예약 게시
- `photos.getWallUploadServer` → 업로드 → `photos.saveWallPhoto` — 이미지 게시
- `wall.getComments` / `wall.createComment` — 댓글·문의 자동 처리
- 위 위에 AI 콘텐츠 생성 붙이기

토큰이 만료되면 이 페이지를 VK에서 다시 열어 재발급하면 됩니다.
