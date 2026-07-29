# CAP AIRLINE

비행기를 타고 세계를 누비며 각국의 수도를 맞히는 웹 게임. 순수 HTML/CSS/JS 단일 파일이라
설치·빌드 없이 브라우저에서 바로 실행된다.

## 실행 / 배포 현황
- **서비스 중**: https://capairline.com/
- **저장소**: https://github.com/TIONBARY/cap_airline (`main` 브랜치, MIT)
- **랭킹 서버**: Supabase 연결 완료 → 전 세계 공용 랭킹이 실제로 돌고 있다
- 로컬 실행은 `index.html` 더블클릭이면 끝. 서버 불필요.
  (랭킹 테스트까지 하려면 `python -m http.server 5599` 로 띄우는 게 편하다)
- **국기 이미지만** 인터넷에서 불러온다 (flagcdn.com). 지도·사운드·이펙트는 전부 오프라인 내장.

## 파일 구조
- `index.html` — 게임 전체 (HTML + CSS + JS 인라인). 이 파일 하나가 게임의 전부다.
- `CLAUDE.md` — 이 문서 (작업용 상세 아키텍처).
- `README.md` — 저장소 공개용 소개. 게임 규칙·특징이 바뀌면 여기도 같이 고칠 것.
- `LICENSE` — MIT (© 2026 TIONBARY).
- `.gitignore` — OS 부산물(Thumbs.db 등)만 제외.
- `CNAME` — 커스텀 도메인(`capairline.com`) 지정. **GitHub Pages가 자동 생성한 파일이라 지우면 안 된다.**
- `docs/` — README용 스크린샷 (`start.jpg`, `play.jpg`).

## 배포 (GitHub Pages + 커스텀 도메인)
- 방식은 **Deploy from a branch** (`main` / `root`). **`main`에 푸시하면 자동 재배포**된다.
  누를 버튼 없음. Actions 탭의 `pages build and deployment`로 진행 상황을 본다.
- 푸시 후 실제 반영까지 **보통 40초~2분**.
- **도메인**: `capairline.com` (Cloudflare Registrar 구매, Cloudflare DNS).
  - apex에 GitHub Pages IP **A 레코드 4개**(`185.199.108~111.153`), `www`는 CNAME → `tionbary.github.io`
  - Cloudflare 프록시는 **회색 구름(DNS only)**. 주황 구름이면 GitHub이 인증서를 발급하지 못한다
  - `www` / `http` 진입은 전부 `https://capairline.com/` 으로 301된다 (Enforce HTTPS 켜짐)
  - 저장소 루트의 **`CNAME` 파일이 도메인을 지정한다. 지우면 도메인 연결이 끊긴다**
- ⚠️ **Pages 설정에서 도메인을 건드리면 GitHub이 원격에 `CNAME` 커밋을 직접 만든다.**
  로컬에서 작업하기 전에 반드시 `git pull --rebase origin main` — 안 하면 푸시가 거절된다(실제로 겪었음).
- ⚠️ Enforce HTTPS를 켜도 http→https 리다이렉트가 도는 데 **2분쯤** 걸린다. 바로 안 된다고 당황하지 말 것.
- ⚠️ **캐시 2종을 구분할 것**
  1. HTML은 `max-age=600`이라 최대 10분간 옛 버전이 보일 수 있다 → `Ctrl+Shift+R`
  2. 카톡·트위터는 **URL별로 미리보기를 캐시**한다. OG 태그를 고쳤는데 미리보기가 그대로면
     [카카오 공유 디버거](https://developers.kakao.com/tool/debugger/sharing)에서 초기화해야 한다
- ⚠️ Pages는 리눅스라 **파일명 대소문자를 구분한다.** 윈도우에서 개발하면 이 버그가 안 보인다.
- 배포 후 검증은 실제 주소로 직접 접속해서 할 것. 콘솔 에러 / 국기 로딩 / 랭킹 조회를 확인한다.

## 공개용 메타 (`<head>`)
- **파비콘** — 🌍 이모지를 인라인 SVG data URI로 넣었다. 외부 파일도, 추가 요청도 없다.
  바꾸려면 data URI 안의 이모지(`%F0%9F%8C%8D`)만 교체하면 된다.
- **OG / 트위터 카드** — 카톡·트위터·디스코드 링크 미리보기용. 이미지는 `docs/play.jpg`.
  `og:url` / `og:image` / `twitter:image`는 **`https://capairline.com/` 절대 URL**이다.
  상대 경로면 크롤러가 못 읽으므로 반드시 절대 URL이어야 한다.
  → 도메인이 또 바뀌면 이 세 곳 + README 플레이 링크를 같이 고치고, 카카오 공유 디버거로 캐시를 비울 것.
- 게임 규칙·소개 문구를 고치면 `og:description`, `meta[name=description]`, README도 같이 맞출 것.

## 기술 스택 / 설계 원칙
- **언어**: HTML5 + CSS + 바닐라 JavaScript (프레임워크·라이브러리 없음)
- **렌더링**: 블럭 지도와 이펙트는 `<canvas>`, 국기 핀/비행기는 DOM 요소, 지도 줌은 CSS transform
- **사운드**: WebAudio로 실시간 합성 (오실레이터 + 노이즈 버퍼). 음원 파일 없음.
- **외부 의존성**: 국기 이미지(`https://flagcdn.com/w160/<code>.png`) 뿐. 그 외 전부 자체 포함.

## 게임 규칙
- 시작 전 **이름 입력** 필수. 이름은 `localStorage["cap_name"]`에 저장돼 다음 방문에 자동으로 채워진다.
- 게임오버 시 점수가 랭킹에 등록된다. **시작 화면은 TOP 5**, **결과 화면은 TOP 10**,
  양쪽의 "전체 랭킹 보기" 버튼으로 **전체 명단 모달**을 연다 (내 이름 행은 파랗게 강조).
- **이름이 곧 식별자다.** 계정이 없어서 같은 이름으로 여러 명이 플레이하면 **한 칸을 공유**하고
  그중 최고점만 표시된다. `max()` 집계라 남의 기록을 낮추는 건 불가능하지만 선점·사칭은 가능하다.
  (해결하려면 `이름#태그` 방식이나 로그인이 필요 — 캐주얼 게임이라 의도적으로 안 넣었다)
- **목숨 ❤️❤️ 2개**로 시작 (`MAX_LIVES`). 목숨이 0이 되면 HUD에 **"마지막 기회!"**가 뜨고,
  그 상태에서 한 번 더 틀리면(`lives < 0`) 게임오버. 즉 **총 3번 틀려야 끝난다**.
  (`updateHud`는 하트 표시를 0으로 clamp하고 `#lives.last-chance` 클래스를 토글한다)
- 나라는 **무한 출제**하되 **중복 없음**. 출제 덱(`deck`)에서 한 장씩 뽑아 쓰고,
  덱이 비면 다시 만든다 → 한 바퀴(100문제) 안에서는 같은 나라가 절대 안 나온다.
  덱 이음매에서만 직전 나라와 겹칠 수 있어 그건 따로 막는다(`lastCode`). 점수 = 맞힌 개수.
- **출제 순서는 인구 순위 쪽으로 기울어 있다** (`buildDeck`). 초반엔 인구 많은(=아는) 나라,
  후반으로 갈수록 생소한 나라가 나와 난이도가 올라간다. 완전 무작위였을 때 첫 문제부터
  부르키나파소가 나오던 문제를 해결한 것. 순열이라 **중복 없음·전 국가 출제** 성질은 그대로다.
- 비행기가 나라에 도착하면 그 자리(핀)에 **국기 + 나라명**이 뜬다. 하단 바에 수도를 입력.
- **제한시간 `ANSWER_TIME` (현재 5초).** 도착하는 순간 입력창 위 게이지가 줄어들기 시작하고,
  다 되면 **오답 처리**된다(`⏰ 시간 초과!` → 미사일 연출 → 목숨 -1).
  검색해서 답을 찾는 걸 막는 장치라 **시간 계산이 곧 치트 방지**다. 아래 두 가지를 깨뜨리지 말 것:
  1. **만료 판정은 `setTimeout`이 한다.** rAF는 게이지 그리는 용도일 뿐이다.
     rAF로 판정하면 탭을 백그라운드로 돌리는 순간 타이머가 멈춰서 그게 곧 치트가 된다
  2. **타이머는 `flyTo` 완료 지점에서만 시작한다.** `idleAtDest()` 안에 넣으면 리사이즈 때도
     호출돼서 창 크기만 바꾸면 시간이 리셋된다
- **Enter 한 번** = 채점 + 즉시 다음 나라로 출발. (이전 문제 결과는 비행 중에 표시되다가 도착 시 사라짐)
  **단 Enter가 먹는 구간은 "도착해서 제한시간이 도는 동안"뿐이다.** 비행 중·미사일 연출 중·완주 연출 중엔
  무시된다 (`checkAnswer`의 `answered || midFlight || celebrating` 가드).
- **정답**: 폭죽 🎉 + 점수 +1
- **세계일주 완주**: 덱의 **100번째(마지막) 나라까지 맞히면** 완주 연출.
  팡파르 + 색종이 + 폭죽 연발 + 지도 전체로 줌아웃 + "🌍 세계일주 완주!" 배너(3초).
  이후 새 덱으로 다음 바퀴가 이어진다(2바퀴째부터는 배너에 "N바퀴째" 표시).
- **오답**: 미사일이 날아오지만 **비행기는 플레어로 회피한다** (약 1~2초) — 목숨만 -1
  1. 무작위 공해상에서 **미사일** 발사 → 붉은 비네트 + "⚠️ 미사일 접근" 경보 + 접근할수록 빨라지는 경고음
  2. 비행기는 지그재그 **회피 기동**, 주위에 **록온 조준선**이 조여든다
  3. 근접하면 비행기가 **플레어** 사출 → 미사일이 속아 록온이 풀리고 옆으로 빗나감
  4. 비행기를 스쳐 **충분히 멀어진 뒤**(`missDist`, 110~190px) 엉뚱한 곳에서 유폭 → 작은 폭발 + 약한 셰이크
  - 피드백: `🔥 플레어로 회피! 정답은 …`
- **마지막 오답(격추)**: 목숨 0에서 또 틀리면 **플레어를 아예 못 쓴다.**
  미사일이 록온을 놓치지 않고 그대로 파고들어 **명중** →
  큰 폭발 + 강한 셰이크 + 비행기 소멸 → 게임오버 (`💥 격추당했다!`)

## 코드 아키텍처 (index.html 내 `<script>`)

### 데이터
- `COUNTRIES[]` — `{ code, country, capital, lat, lon, pop }` **100개국**.
  UN 회원국(193개국) 중 **인구 상위 100개국**이고, **배열 순서 = 인구 순위(내림차순)**.
  - `code` — ISO 3166-1 alpha-2 **소문자**. 국기 URL에 그대로 들어간다
  - `capital` / `lat` / `lon` — 정답이 되는 수도와 그 **수도의 좌표** (나라 중심이 아님)
  - `pop` — 인구(백만 명, UN 세계인구전망 2024 추계). 게임 로직엔 안 쓰이고 **목록 갱신용 기준**
- `WORLD_GRID` — 120×60 블럭 세계지도 (문자: `.`바다 `g`초원 `d`사막 `i`빙하).
  실제 위성지도를 다운샘플링해 만든 것. 대륙 모양 인식 가능.

### 문제 출제 알고리즘 (`buildDeck` + `pickCountry`)

**만족해야 하는 제약 3가지** — 하나라도 깨면 다른 기능이 망가진다.
1. **한 바퀴(100문제) 안에 중복 없음**
2. **한 바퀴에 100개국이 전부 등장** — 안 그러면 "세계일주 완주" 연출이 거짓말이 된다
3. **초반엔 아는 나라, 후반엔 생소한 나라** (난이도 상승 곡선)

**왜 가중 추출이 아니라 "흔들린 정렬"인가**
확률 가중 추출(뽑을 때마다 인구 가중치로 고르기)을 쓰면 3번은 되지만 **1·2번이 깨진다** —
같은 나라가 또 뽑히거나 끝까지 안 나오는 나라가 생긴다. 그래서 **순열을 만드는 방식**을 쓴다.

```js
const DECK_SPREAD = 65;
function buildDeck() {
  return COUNTRIES
    .map((c, i) => ({ c, key: i + (Math.random() - .5) * DECK_SPREAD }))  // i = 인구 순위
    .sort((a, b) => a.key - b.key)
    .map(o => o.c);
}
```

`COUNTRIES`가 **인구 내림차순**이라 인덱스 `i`가 곧 인구 순위다. 거기에 `±DECK_SPREAD/2`
무작위 흔들림을 더해 정렬하면, **대체로 인구 순서인데 매판 다른 순열**이 나온다.
정렬이라 100개국이 정확히 한 번씩 들어가므로 제약 1·2가 자동으로 지켜진다.

**`DECK_SPREAD` 튜닝 근거** (2000판 시뮬레이션)

| SPREAD | 첫 문제 10위내 | 첫 문제 최악 순위 | 첫 문제 후보 수 |
|---|---|---|---|
| 35 | 97% | 17위 | 17개국 (너무 뻔함) |
| **65 ← 채택** | **90%** | **25위** | **24개국** |
| 110 | 80% | 33위 | 28개국 (어려운 게 초반에 나옴) |

실제 3바퀴 플레이 시 구간별 평균 인구 순위: `1~5문제 ≈ 8위` → `6~15 ≈ 18위` →
`16~30 ≈ 32위` → `31~60 ≈ 44위` → `61~100 ≈ 76위`.

**값을 바꿨다면 콘솔에서 재측정할 것**
```js
const rank = new Map(COUNTRIES.map((c,i)=>[c.code,i+1]));
const f=[]; for(let n=0;n<2000;n++) f.push(rank.get(buildDeck()[0].code));
({ 첫문제_10위내: f.filter(r=>r<=10).length/20+'%', 최악: Math.max(...f), 후보수: new Set(f).size })
```

**한계** — 인구 순위는 **인지도의 대용일 뿐 난이도가 아니다.**
오스트리아(99위)의 빈은 쉽고 에티오피아(10위)의 아디스아바바는 어렵다.
정확히 하려면 `COUNTRIES`에 수동 `difficulty` 필드를 붙여 정렬 기준을 바꿔야 한다.

**`pickCountry()`** 는 이 덱에서 앞에서부터 한 장씩 뽑고, 덱이 비면 `buildDeck()`으로 재충전한다.
덱 이음매에서만 직전 나라와 겹칠 수 있어 `lastCode`로 그 한 경우만 막는다.

### 좌표계 (중요)
- **월드 좌표**: 등거리원통도법. `WORLD_W=1680, WORLD_H=840` (블럭 CELL=14).
  - `project(lat, lon)` → `{x, y}` 월드 좌표
- **카메라**: 월드→화면 변환. `#camera` div에 `translate + scale` transform.
  - `worldToScreen(wx, wy)` → `{x, y}` 화면 좌표. **인자는 x, y 각각 전달** (객체 통째로 넘기면 NaN 됨! 과거 미사일 안 보이던 버그 원인)
  - `focusCamera(fx, fy, s)` — 월드 좌표 (fx,fy)를 화면 중앙에 배율 s로
  - `overviewCamera()` — 지도 전체 보기 (시작/결과 화면)
  - `shake` + `screenShake()` — 폭발 시 카메라 흔들림 (cam transform에 오프셋 합성)

### 모바일 대응
- **소프트 키보드 가림 (핵심)** — `position: fixed`는 **레이아웃** 뷰포트 기준이라 키보드가 올라와도
  바닥에 붙어 가려진다. `visualViewport`(실제로 보이는 영역)와의 차이를 `--kb` CSS 변수로 내보내고,
  `#answer-bar`는 `bottom: var(--kb)`, `.feedback`은 `bottom: calc(var(--kb) + 86px)`로 따라 올라간다.
  → `syncKeyboardInset()`. `visualViewport`가 없는 구형 브라우저에서는 `--kb`가 0이라 기존 동작 그대로다.
  - ⚠️ 뷰포트 메타에 `interactive-widget=resizes-content`를 **넣지 않았다.** 그걸 넣으면 키보드가
    열릴 때 레이아웃 뷰포트가 줄면서 `resize` 이벤트가 터지고, 리사이즈 핸들러가 카메라를 다시 잡아
    화면이 튄다. `visualViewport` 방식은 `resize`를 발생시키지 않는다.
- **비행기 크기 연동** — 모바일에서 `#plane`을 44px로 줄인다. 말풍선 오프셋과 록온 조준선 반경은
  `planePx`(=`#plane`의 실제 font-size)에서 **계산**하므로 CSS만 바꾸면 자동으로 따라온다.
  계수는 60px 기준 실측값에서 뽑았다 (말풍선 `×0.90 / ×0.53`, 조준선 `×2.13 → ×1.50`).
  `syncPlaneSize()`는 로드 시와 `resize`마다 호출된다.
- **입력창 글꼴은 16px 미만으로 줄이지 말 것** — iOS 사파리가 입력 시 화면을 확대해버린다.
- `touch-action: manipulation`(더블탭 확대 방지), `overscroll-behavior: none`(당겨서 새로고침 방지),
  `viewport-fit=cover` + `env(safe-area-inset-bottom)`(노치 대응).
- 브레이크포인트는 `@media (max-width: 560px)` 하나뿐이다.

### 지도/비행 레이어
- `#camera > #map(canvas) + #trail(svg)` — 이 둘만 줌/이동에 함께 스케일됨
- `#pin`, `#plane` — 화면 고정(fixed), 크기 일정. `worldToScreen`으로 위치만 잡음
- 비행 중에만 표시: `body.flying` 클래스로 토글

### 핵심 함수
- `startGame()` — 목숨/점수 초기화, **지난 판 잔상 정리**(`armAnswerInput()` + 피드백 비우기 —
  안 하면 새 판 첫 비행 내내 지난 판의 "⏰ 시간 초과! …"가 남아 보인다), `body.flying` 추가, `goNext()`
- `goNext()` — 다음 나라 뽑고 `flyTo()`로 출발. **입력창을 열지 않는다** (placeholder만 "✈️ 비행 중…"으로).
  `focus()`는 여기서만 잡는다 — 이 함수는 유저 제스처(Enter) 안에서 불리지만 `flyTo` 완료 콜백은
  rAF라 제스처 밖이라 모바일에서 키보드를 못 올린다
- `armAnswerInput()` — **답을 받기 시작하는 유일한 지점.** `answered=false` + 입력값·정오답 색 리셋 +
  placeholder 원복. `flyTo` 완료 시 `startAnswerTimer()`와 나란히 호출된다 (새 판은 `startGame`에서도 1회)
- `pickCountry()` / `buildDeck()` — 출제 순서. 위 **"문제 출제 알고리즘"** 섹션 참고
- `flyTo(q)` — 베지어 곡선 비행 + 카메라 줌인→줌아웃→줌인. 도착 시 `idleAtDest()`
  - 핀 내용은 **출발 시 비우고, 도착 시(`idleAtDest`)에만 채운다** → 출발 순간 다음 나라 노출 방지
- `idleAtDest()` — 도착지 줌인, 핀(국기+나라명) 표시, 비행기를 도착지에 세워둠, 피드백 지움
- `placePin(x, y)` — 핀은 도시 지점 **오른쪽 위로 비켜서** 배치(`+24, -14`, bottom-left 기준).
  **생각 말풍선** 모양이라 꼬리가 삼각형이 아니라 왼쪽 아래(비행기 쪽)로 작아지는 점 두 개(`.pin-tail`).
  비행기를 가리지 않게 하려는 것이므로, 카드 크기나 오프셋을 바꾸면 겹치는지 꼭 확인할 것
- `checkAnswer(timedOut)` — 채점. 정답→폭죽+goNext / 오답·시간초과→목숨-1+미사일 연출
  - **맨 앞 가드 `if (answered || midFlight || celebrating) return;` 를 지우지 말 것.**
    비행 중·연출 중 채점이 통과하면 도착도 안 한 좌표로 미사일이 날아가고, `flyTo`의 rAF와
    `updateMissile`이 둘 다 `placePlane()`을 호출해 충돌하며 `goNext()`가 이중으로 돈다
  - `timedOut=true`면 입력 내용과 무관하게 오답. 피드백 머리말이 `⏰ 시간 초과!`로 바뀐다
  - 정답 시 `deck.length === 0`이면 방금 맞힌 게 덱의 마지막 장 → `celebrateLap()`
- `startAnswerTimer()` / `stopAnswerTimer()` — 제한시간 시작·정지.
  시작은 **`flyTo` 완료 지점 한 곳**, 정지는 채점·비행 시작·완주 연출·게임오버·새 판에서 각각 호출한다
- `timerFrame(now)` — 게이지(`scaleX`)와 숫자 갱신, 60%/30% 지점에서 색 전환, 마지막 3초 `sfx.tick()`
- `onAnswerTimeout()` — `setTimeout` 만료 콜백. `checkAnswer(true)`로 넘긴다
- `celebrateLap()` — 완주 연출. `celebrating` 플래그로 리사이즈 등이 끼어드는 걸 막고,
  핀·비행기를 잠시 숨긴 뒤(줌아웃하면 화면 고정 좌표가 어긋나므로) 3초 후 `goNext()`
- `tweenCamera(x, y, s, ms)` — 카메라를 목표 위치·배율로 부드럽게 이동 (완주 줌아웃용)
- `spawnConfetti()` — 완주 색종이, `sfx.fanfare()` — 완주 팡파르
- `gameOver()` — 결과 화면
- `updateHud()` — 하트/점수 갱신

### 사운드 (`sfx`)
- `ac()` — AudioContext 지연 생성(첫 클릭 이후). 음소거면 `null` 반환해 전부 무시됨.
- `tone(f0, f1, dur, vol, type, when)` — 주파수가 미끄러지는 오실레이터
- `noiseHit(dur, vol, c0, c1, q, when)` — 로우패스 컷오프를 쓸며 감쇠하는 노이즈
- `sfx.launch / warn / flare / boom / pop / tick` — 발사·경고음·플레어·폭발·폭죽·제한시간 초읽기
- HUD 우측 🔊 버튼(`#sfx-btn`)으로 음소거 토글, `localStorage["cap_muted"]`에 저장

### 이펙트 시스템 (단일 `fxLoop`)
한 canvas(`#fx`)에서 전부 렌더. 렌더 순서가 곧 레이어 순서다:
연기(일반 합성) → 불꽃·파편(**가산 합성** `lighter`) → 충격파 링 → 화면 섬광 → 미사일 본체(일반 합성).
- `fxParticles[]` — 입자. 필드: `x,y,vx,vy,life,decay,size,color` +
  `grav`(중력) `drag`(항력) `growth`(프레임당 반경 증가) `smoke`(true면 일반 합성·저알파)
- `fxRings[]` — 충격파 링 `{x,y,r,vr,w,life,decay,color}` (color는 `"r,g,b"` 문자열)
- `flash` — 폭발 섬광 세기(0~1). 매 프레임 `*= .79` 감쇠
- `launchFireworks()` / `spawnBurst()` — 정답 폭죽
- `explode(x, y, k)` — 섬광 + 링 2개 + 불덩이 + 파편 + 연기. `k`는 폭발 규모(1=격추, 0.6=회피 유폭)
- `sparkPuff()` — 작은 불티

### 미사일 (페이즈 머신)
`launchMissile(tx, ty, done, fatal)` — tx,ty는 도착지 비행기의 **화면 좌표**.
`fatal=false`(평소 오답)면 **회피 성공**으로, `true`(마지막 오답)면 **격추**로 끝난다.
`updateMissile(now)`가 매 프레임 갱신.
- 등속직선이 아니라 **선회율 제한 유도(호밍)**: 목표 방향과의 각도차를 `m.turn` 이내로만 꺾어 자연스러운 호를 그린다.
  `dt`는 60fps 기준 프레임 배수(최대 3으로 클램프)라 프레임률이 달라도 속도가 같다.
- 페이즈 — **`fatal` 여부로 경로가 완전히 갈린다**
  - `chase` — 추격. 일부러 빗나간 각도로 출발해 크게 휘어 들어온다
    - `fatal=false`: 300px까지 접근하면 플레어 사출 → `decoyed`
    - `fatal=true`: **플레어를 쓰지 않는다.** 록온이 풀리지 않고 선회율을 `.34`까지 올리며
      그대로 파고들어 `hitPlane()`. 목숨을 다 쓴 뒤엔 회피 수단이 없다는 설정
  - `decoyed` — **회피 전용 경로.** 플레어(`m.decoy`)를 쫓아가고, 록온이 풀려 조준선을 안 그린다.
    비행기와 `m.missDist`(110~190px) 이상 벌어지면 `evadeMissile()` → 유폭(비행기 생존)
- ⚠️ 격추 쪽 선회율을 초반에 더 낮추면 "크게 휘어 들어와 극적일 것" 같지만 **반대다** —
  직선으로 꽂혀 오히려 빨리 끝난다(측정으로 확인). 연출을 늘리려면 속도를 낮추는 쪽이 맞다.
- `m.plane` — 회피 기동 중인 실제 비행기 위치. `m.home`(도착지) 기준으로 sin 파형 흔들림 + 뱅크 회전
- `drawLockOn()` — 비행기 주위 록온 조준선(가까울수록 조여들고 빨리 깜빡임)
- `evadeMissile()` — 작은 폭발(k=0.6) + 약한 셰이크, **520ms 뒤** 콜백. 비행기는 그대로 살아서 다음 나라로
- `hitPlane()` — 큰 폭발 + 강한 셰이크 + 비행기 숨김, **620ms 뒤** 콜백(게임오버)
- 경보 UI는 `body.alarm` 클래스로 토글 (`#danger` 붉은 비네트, `#warn` 경고 배너)
- 소요(20회 측정): **회피 1.3~1.9초 / 격추 0.9~1.5초**. 격추가 더 짧은 건 곧장 파고들기 때문이다.
  늘어지지 않게 `decoyed` 1300ms / 전체 5000ms 초과 시 비행기 자리에서 강제 폭발

## 온라인 랭킹 (Supabase)
이름별 최고 점수 랭킹. **이미 연결돼 있고 운영 중이다** — `RANK.url` / `RANK.key`에 실제 값이 들어가 있다.
서버가 죽거나 키가 비면 자동으로 `localStorage` 로컬 랭킹으로 **폴백**해서 게임은 멈추지 않는다.

아래는 **처음부터 다시 세팅하거나 포크한 사람이 자기 프로젝트에 붙일 때** 필요한 절차다.

1. [supabase.com](https://supabase.com)에서 프로젝트 생성 (무료 티어, **리전은 Seoul** — 나중에 변경 불가)
   - 생성 화면의 Security 옵션: **Enable Data API 켜기(필수)** /
     **Automatically expose new tables 끄기** / **Enable automatic RLS 켜기**
   - "자동 노출"을 껐기 때문에 아래 SQL에 **명시적 `grant`가 반드시 필요**하다.
     RLS 정책만으로는 안 된다 — PostgREST는 정책 이전에 테이블 권한을 먼저 본다.
2. **SQL Editor**에 아래를 그대로 실행
   ```sql
   -- 점수는 append-only. 남의 기록을 덮어쓰거나 지울 수 없게 insert만 허용한다.
   create table public.scores (
     id         bigint generated always as identity primary key,
     name       text   not null check (char_length(name) between 1 and 12),
     score      int    not null check (score >= 0 and score <= 100000),
     created_at timestamptz not null default now()
   );

   -- RLS: 익명에게 등록/조회만 허용. update·delete는 정책이 없으니 자동 차단.
   alter table public.scores enable row level security;
   create policy "anon insert" on public.scores for insert to anon with check (true);
   create policy "anon select" on public.scores for select to anon using (true);

   -- "새 테이블 자동 노출"을 껐으므로 권한을 직접 부여 (update/delete는 주지 않는다)
   grant usage on schema public to anon;
   grant select, insert on public.scores to anon;

   -- 이름별 최고점만 집계한 랭킹 뷰 (게임은 이 뷰만 읽는다)
   -- security_invoker = on → 뷰가 호출자(anon) 권한으로 돌아 RLS가 그대로 적용된다.
   --   끄면 뷰가 소유자 권한으로 실행되어 RLS를 우회한다 (Supabase 보안 린터가 지적하는 패턴)
   create view public.leaderboard with (security_invoker = on) as
     select name, max(score) as score, max(created_at) as at
     from public.scores group by name;
   grant select on public.leaderboard to anon;
   ```
   `id`가 `generated always as identity`라 시퀀스 권한은 따로 줄 필요 없다.
   > SQL의 `to anon`은 **Postgres 역할 이름**이지 API 키 이름이 아니다. Publishable 키를 쓰든
   > 구형 anon 키를 쓰든 둘 다 이 `anon` 역할로 매핑되므로 이 SQL은 그대로 쓰면 된다.
3. **Project Settings → API**에서 `Project URL`과 **Publishable 키**(`sb_publishable_...`)를 복사
   - 구형 `anon` JWT 키(`eyJhbGci...`)도 아직 동작하지만 **Publishable을 쓴다.**
     구형은 만료가 있고(JWT `exp`), 폐기하려면 JWT 시크릿을 갈아야 해서 모든 키·세션이 함께 무효화된다.
     Publishable은 개별 발급·폐기가 된다.
   - ⚠️ `sb_secret_...` (구 `service_role`)는 절대 넣지 말 것. RLS를 통째로 무시한다.
4. `index.html` 상단 `── 랭킹` 블록의 `const RANK = { url, key }` 두 줄을 자기 값으로 바꾼다
   (두 값을 빈 문자열로 두면 로컬 랭킹 모드로 떨어진다)
5. 검증은 브라우저 콘솔에서 — 등록·조회뿐 아니라 **막혀야 할 것이 막히는지도** 확인할 것:
   ```js
   await submitScore('검증', 9); await submitScore('검증', 3);   // 등록
   (await topScores(10)).rows                                    // 검증:9 만 나와야 함(max 집계)
   const H = { ...rankHeaders(), 'Content-Type': 'application/json' };
   await fetch(RANK.url + '/rest/v1/scores?name=eq.검증',
     { method: 'PATCH', headers: H, body: '{"score":99999}' });   // 401/42501 이어야 정상
   await fetch(RANK.url + '/rest/v1/scores?name=eq.검증',
     { method: 'DELETE', headers: H });                           // 401/42501 이어야 정상
   ```
   검증용 행은 anon 권한으로 못 지운다(그게 설계 의도). Supabase SQL Editor에서
   `delete from public.scores where name = '검증';` 로 정리한다.

**랭킹 관련 함수**
- `submitScore(name, score)` — 서버 등록 + 로컬에도 항상 기록(폴백용)
- `topScores(n)` — `leaderboard` 뷰에서 상위 n명. 미설정·오프라인이면 로컬에서 반환
- `bestScore(name)` — 그 이름의 최고점. **서버 기준**이라 결과 화면의 "최고 N"과 랭킹표가 항상 일치한다.
  ⚠️ 반드시 `submitScore` **전에** 호출할 것 (안 그러면 방금 낸 점수가 포함돼 항상 경신 실패로 보인다)
- `rankRow(i, r, me)` — 한 줄 생성. **이름은 `textContent`로만** (공용 랭킹이라 XSS 방지, `innerHTML` 금지)
- `renderRank(box, me, limit)` — 시작(5) / 결과(10) 목록
- `renderAllRanks(me)` / `openRankModal()` / `closeRankModal()` — 전체 랭킹 모달(`#rank-modal`).
  최대 `RANK_ALL_MAX`(500)명, 내 행은 `scrollIntoView`로 보여준다. ✕ · 배경 클릭 · ESC로 닫힌다

**설계 메모**
- **왜 upsert가 아니라 insert-only인가** — 클라이언트 키는 공개되므로 upsert를 허용하면 아무나 남의 이름 점수를
  낮게 덮어쓸 수 있다. 전 기록을 쌓고 뷰에서 `max()`를 집계하면 그 공격이 원천 봉쇄된다.
- **권한은 2중 방어** — ① 테이블 GRANT를 `select, insert`로만 준다(권한 자체가 없으면 RLS 이전에 막힌다)
  ② 그 위에 RLS 정책. 둘 중 하나만 잘못돼도 뚫리지 않는다.
- **어뷰징** — 클라이언트가 점수를 보내는 구조라 마음먹으면 가짜 점수를 넣을 수 있다.
  `check` 제약이 말도 안 되는 값만 거른다. 더 막으려면 Edge Function에 검증/레이트리밋을 두는 방향.
- **XSS** — 남이 입력한 이름을 그리므로 `renderRank()`는 반드시 `textContent`만 쓴다. `innerHTML` 금지.
- **Publishable 키 공개** — 정상이다. 어차피 HTML에 박혀 모든 방문자 브라우저로 전송된다.
  GRANT와 RLS가 권한을 묶고 있어 노출돼도 문제되지 않는다 (`sb_secret_...`는 절대 넣지 말 것).
- **요청 헤더는 `apikey` 하나만** 보낸다. `Authorization: Bearer`는 로그인 사용자의 세션 JWT 자리라
  Publishable 키를 넣는 건 의미상 틀렸다(넣어도 동작은 함). `rankHeaders()` 참고.

## 나라 목록 갱신 (UN 인구 기준)
`COUNTRIES[]`는 **UN 세계인구전망(World Population Prospects)** 최신 추계를 기준으로 주기적으로 갱신한다.

1. UN WPP에서 회원국 인구 순위를 받아 상위 100개국을 추린다 (UN 회원국만 — 홍콩·대만·팔레스타인 등 비회원은 제외)
2. `pop`(백만 명)을 새 수치로 갱신하고, **인구 내림차순으로 배열을 다시 정렬**한다
3. 새로 100위 안에 든 나라를 추가 / 밀려난 나라를 삭제한다. 추가할 땐 네 가지가 다 필요하다:
   `code`(ISO alpha-2 소문자) · 한글 나라명 · 한글 수도명 · **수도의** 위경도
4. 갱신 후 브라우저 콘솔에서 아래를 돌려 검증한다 (개수 / 중복 / 좌표 / 정렬 / 국기 로드):
   ```js
   COUNTRIES.length                                  // 100
   COUNTRIES.map(c=>c.code).filter((v,i,a)=>a.indexOf(v)!==i)   // 코드 중복 []
   COUNTRIES.filter((c,i)=>i>0 && COUNTRIES[i-1].pop < c.pop)   // 정렬 오류 []
   await Promise.all(COUNTRIES.map(c=>new Promise(r=>{const i=new Image();
     i.onload=()=>r(null); i.onerror=()=>r(c.code); i.src=`https://flagcdn.com/w160/${c.code}.png`;})))
     .then(a=>a.filter(Boolean))                     // 국기 로드 실패 []
   ```

**수도 표기 원칙** — 공식 수도를 쓰되, 타이핑 게임이라 입력이 사실상 불가능하면 실질 수도를 쓴다.
판단이 갈린 나라는 데이터에 한 줄 주석을 달아뒀다 (남아공/코트디부아르/베냉/볼리비아/스리랑카/탄자니아/이스라엘 등).

## 알려진 이슈 / 참고
- ✅ (해결됨) **비행 중 Enter가 먹던 버그** — 원인은 "문제 시작 지점"이 둘로 갈려 있던 것.
  `goNext()`가 출발하면서 입력창을 열어(`readOnly=false`) 비행 중 Enter가 그대로 채점됐다.
  → **답을 받는 구간을 도착~채점 사이로 못 박았다.** ① `checkAnswer()` 맨 앞 가드
  ② 입력 리셋을 `armAnswerInput()`으로 묶어 `flyTo` 완료 지점에서만 호출.
  - **`readOnly`는 일부러 쓰지 않았다** — iOS 사파리는 readonly 입력창에 소프트 키보드를 안 띄운다.
    이미 포커스된 입력창을 readonly로 바꾸면 키보드가 내려가고, 1.8초 비행마다 오르내리면
    `--kb` 인셋이 같이 출렁여 화면이 요동친다. **포커스는 한 번 잡으면 끝까지 놓지 않는다.**
  - 비행 중 타이핑 자체는 되지만 Enter가 안 먹고, 도착 시 `armAnswerInput()`이 값을 비운다.
    비행 중이라는 표시는 placeholder(`✈️ 비행 중…`)와 `body:not(.timing) #answer-input { opacity: .55 }`
    로만 한다 (`body.timing`이 곧 "지금 답 받는 중"이라 새 플래그가 필요 없다)
- ✅ (해결됨) 미사일이 안 보이던 버그: `worldToScreen(project(...))`로 객체를 통째 넘겨 NaN 좌표가 됐던 것.
  `worldToScreen(wp.x, wp.y)`로 수정.
- ✅ (해결됨) 미사일 연출 조잡함 → 호밍·플레어 기만·경보 UI·폭발/사운드까지 폴리싱 완료.
- 국기 핀 도착 시 팝인 지연 방지를 위해 `flyTo`에서 다음 국기를 `new Image()`로 프리로드.
- ⚠️ 지도가 120×60 블럭(1블럭 ≈ 3°)으로 저해상도라 **해안 도시 14곳은 바다 블럭 위에 찍힌다**
  (도쿄·로마·리스본·스톡홀름·자카르타·알제·튀니스·아크라 등). 데이터 오류가 아니라 지도 해상도 한계.
  거슬리면 `MAP_COLS/MAP_ROWS`를 키우거나 해당 좌표를 내륙으로 조금 밀면 된다.
- 넓은 데스크탑 화면에서 가장 잘 보이지만 모바일도 대응돼 있다 ("모바일 대응" 참고).
  단 **실기기 검증은 아직**이다 — 키보드 인셋·비행기 축소 로직은 코드로 검증했으나
  실제 폰에서 눈으로 확인한 적은 없다. iOS 사파리 / 안드로이드 크롬에서 한 번씩 볼 것.
- **테스트 팁**: 브라우저 탭이 백그라운드면 `requestAnimationFrame`이 멈춰 연출 검증이 안 된다.
  콘솔에서 `fxLoop(performance.now() + k*16.7)`를 직접 반복 호출하면 프레임을 수동으로 돌릴 수 있다.

## ▶ 다음 작업
1. **실기기에서 모바일 확인** — 코드 검증은 끝났지만 실제 폰에서 본 적이 없다.
   iOS 사파리(키보드 인셋이 가장 까다롭다) / 안드로이드 크롬에서 한 판씩 돌려볼 것.
2. **진짜 난이도 필드** — 지금은 인구 순위를 인지도 대용으로 쓰는데 완벽하진 않다.
   오스트리아(99위)의 빈은 쉽고 에티오피아(10위)의 아디스아바바는 어렵다.
   정확히 하려면 `COUNTRIES`에 수동 `difficulty` 필드를 붙여 `buildDeck`의 정렬 기준을 바꿔야 한다
3. **점수 연동 난이도 상승** — 점수가 오를수록 미사일이 빨라지게
   (`launchMissile`의 `speed`/`turn`을 `score`로 스케일). 제한시간을 점점 줄이는 방법도 있다.

**완료된 것** — 비행 중 Enter 채점 버그 수정 /
미사일 연출 폴리싱 / 100개국 확장 / 중복 없는 출제 / 세계일주 완주 연출 /
이름 입력 + 온라인 랭킹(Supabase) / 파비콘·OG / GitHub Pages 배포 /
**커스텀 도메인 `capairline.com` + HTTPS** / 정답 제한시간 / 인구 순위 기반 출제 순서 / MIT 라이선스

## 튜닝 포인트 (자주 만지는 숫자)
- `MAX_LIVES` — 시작 목숨 (현재 2)
- **`ANSWER_TIME`** — 정답 입력 제한시간(현재 **7000ms**). 체감 난이도를 가장 크게 바꾸는 숫자다.
  5초로 시작했다가 한글 IME로 긴 수도를 치기엔 빡세서 7초로 올렸다.
- **`DECK_SPREAD`** — 출제 순서가 인구 순위를 얼마나 따르는지 (현재 **65**).
  0이면 매판 인구순 그대로라 지루하고, 400이면 사실상 무작위. 2000판 시뮬레이션 결과 65에서
  첫 문제가 90% 확률로 인구 10위 안 / 최악 25위 / 첫 문제 후보 24개국으로 균형이 좋았다
- `CELL` — 블럭 크기 (현재 14), `PALETTE` — 지도 색상
- `flyTo`의 `zoomScale = fit * 3.0`, `DURATION = 1800` — 카메라 확대율 / 비행 시간
- `launchMissile`의 `speed / accel / maxSpeed / turn` — 미사일 속도·선회 성능
- `chase`의 `dist < 300` — 플레어 사출 거리
- `missDist = 110 + Math.random() * 80` — 회피 시 미사일이 얼마나 빗나가서 터지는지
- `screenShake(power, ms)` — 흔들림 세기, `explode()`의 입자 개수 — 폭발 규모
- `sfx.*`의 `vol` 인자 — 효과음 볼륨
- `COUNTRIES[]` — 나라 추가/삭제. 위 "나라 목록 갱신(UN 인구 기준)" 절차를 따를 것

## 향후 아이디어 (미구현)
- 나라별 정답률 통계 (`scores` 테이블에 오답 기록을 남기면 "가장 많이 틀린 수도" 같은 걸 뽑을 수 있다)
- 랭킹 기간 필터 (오늘 / 이번 주 / 전체) — `created_at`이 이미 있으니 뷰만 추가하면 된다
- 지도 해상도 상향 (해안 수도가 바다에 찍히는 문제 해결)
- (원대한 목표) Electron으로 감싸 Steam 출시 — 단, 현재는 웹 전용으로만 개발 중.
  MIT로 공개했으므로 상업화하려면 그 점을 먼저 고려할 것

## 개발 히스토리 요약
객체식 4지선다 → 타이핑 입력 → 국기 이미지 → 세계지도 배경 + 비행기 이동 → 하단 입력바 →
블럭 지도 + 카메라 줌인아웃 → Enter 즉시 진행 → 목숨제 + 미사일 회피 →
미사일 연출 폴리싱(호밍·플레어 기만·경보 UI·폭발) + WebAudio 사운드 →
**목숨 0에서 마지막 기회 1회 추가** → **오답은 회피 성공, 마지막 오답만 격추로 분리** →
**국기 핀을 생각 말풍선으로 바꿔 비행기 안 가리게** → **30개국 → UN 인구 상위 100개국** →
**덱 방식으로 중복 출제 제거 + 세계일주 완주 연출** → **이름 입력 + 온라인 공용 랭킹(Supabase)** →
**CAP AIRLINE으로 개명 + GitHub Pages 배포** 순으로 발전.
