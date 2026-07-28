# CAP AIRLINE

비행기를 타고 세계를 누비며 각국의 수도를 맞히는 웹 게임. 순수 HTML/CSS/JS 단일 파일이라
설치·빌드 없이 브라우저에서 바로 실행된다.

## 실행 방법
- `index.html`을 브라우저에서 열면 끝 (더블클릭 또는 `start D:\dev\etc\game\index.html`)
- 서버 불필요. 단, **국기 이미지**는 인터넷에서 불러온다 (flagcdn.com). 지도는 오프라인 내장.

## 파일 구조
- `index.html` — 게임 전체 (HTML + CSS + JS 인라인). 이 파일 하나가 게임의 전부다.
- `CLAUDE.md` — 이 문서 (작업용 상세 아키텍처).
- `README.md` — 저장소 공개용 소개. 게임 규칙·특징이 바뀌면 여기도 같이 고칠 것.
- `LICENSE` — MIT (© 2026 TIONBARY).
- `docs/` — README용 스크린샷 (`start.jpg`, `play.jpg`).

## 공개용 메타 (`<head>`)
- **파비콘** — 🌍 이모지를 인라인 SVG data URI로 넣었다. 외부 파일도, 추가 요청도 없다.
  바꾸려면 data URI 안의 이모지(`%F0%9F%8C%8D`)만 교체하면 된다.
- **OG / 트위터 카드** — 카톡·트위터·디스코드 링크 미리보기용. 이미지는 `docs/play.jpg`.
  ⚠️ 지금은 **상대 경로**라 배포 후 절대 URL로 바꿔야 미리보기가 뜬다 ("다음 작업" 참고).
- 게임 규칙·소개 문구를 고치면 `og:description`, `meta[name=description]`, README도 같이 맞출 것.

## 기술 스택 / 설계 원칙
- **언어**: HTML5 + CSS + 바닐라 JavaScript (프레임워크·라이브러리 없음)
- **렌더링**: 블럭 지도와 이펙트는 `<canvas>`, 국기 핀/비행기는 DOM 요소, 지도 줌은 CSS transform
- **사운드**: WebAudio로 실시간 합성 (오실레이터 + 노이즈 버퍼). 음원 파일 없음.
- **외부 의존성**: 국기 이미지(`https://flagcdn.com/w160/<code>.png`) 뿐. 그 외 전부 자체 포함.

## 게임 규칙
- 시작 전 **이름 입력** 필수. 이름은 `localStorage["cap_name"]`에 저장돼 다음 방문에 자동으로 채워진다.
- 게임오버 시 점수가 랭킹에 등록되고, 시작/결과 화면에 **이름별 최고 점수 TOP 10**이 뜬다
  (내 이름 행은 파랗게 강조). 셋업은 "온라인 랭킹 셋업" 참고.
- **목숨 ❤️❤️ 2개**로 시작 (`MAX_LIVES`). 목숨이 0이 되면 HUD에 **"마지막 기회!"**가 뜨고,
  그 상태에서 한 번 더 틀리면(`lives < 0`) 게임오버. 즉 **총 3번 틀려야 끝난다**.
  (`updateHud`는 하트 표시를 0으로 clamp하고 `#lives.last-chance` 클래스를 토글한다)
- 나라는 **무한 출제**하되 **중복 없음**. 100개국을 섞은 출제 덱(`deck`)에서 한 장씩 뽑아 쓰고,
  덱이 비면 다시 섞는다 → 한 바퀴(100문제) 안에서는 같은 나라가 절대 안 나온다.
  덱 이음매에서만 직전 나라와 겹칠 수 있어 그건 따로 막는다(`lastCode`). 점수 = 맞힌 개수.
- 비행기가 나라에 도착하면 그 자리(핀)에 **국기 + 나라명**이 뜬다. 하단 바에 수도를 입력.
- **Enter 한 번** = 채점 + 즉시 다음 나라로 출발. (이전 문제 결과는 비행 중에 표시되다가 도착 시 사라짐)
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
- **마지막 오답(격추)**: 목숨 0에서 또 틀리면 미사일이 플레어에 속았다가 **재포착**,
  급선회로 되돌아와 **명중** → 큰 폭발 + 강한 셰이크 + 비행기 소멸 → 게임오버 (`💥 격추당했다!`)

## 코드 아키텍처 (index.html 내 `<script>`)

### 데이터
- `COUNTRIES[]` — `{ code, country, capital, lat, lon, pop }` **100개국**.
  UN 회원국(193개국) 중 **인구 상위 100개국**이고, **배열 순서 = 인구 순위(내림차순)**.
  - `code` — ISO 3166-1 alpha-2 **소문자**. 국기 URL에 그대로 들어간다
  - `capital` / `lat` / `lon` — 정답이 되는 수도와 그 **수도의 좌표** (나라 중심이 아님)
  - `pop` — 인구(백만 명, UN 세계인구전망 2024 추계). 게임 로직엔 안 쓰이고 **목록 갱신용 기준**
- `WORLD_GRID` — 120×60 블럭 세계지도 (문자: `.`바다 `g`초원 `d`사막 `i`빙하).
  실제 위성지도를 다운샘플링해 만든 것. 대륙 모양 인식 가능.

### 좌표계 (중요)
- **월드 좌표**: 등거리원통도법. `WORLD_W=1680, WORLD_H=840` (블럭 CELL=14).
  - `project(lat, lon)` → `{x, y}` 월드 좌표
- **카메라**: 월드→화면 변환. `#camera` div에 `translate + scale` transform.
  - `worldToScreen(wx, wy)` → `{x, y}` 화면 좌표. **인자는 x, y 각각 전달** (객체 통째로 넘기면 NaN 됨! 과거 미사일 안 보이던 버그 원인)
  - `focusCamera(fx, fy, s)` — 월드 좌표 (fx,fy)를 화면 중앙에 배율 s로
  - `overviewCamera()` — 지도 전체 보기 (시작/결과 화면)
  - `shake` + `screenShake()` — 폭발 시 카메라 흔들림 (cam transform에 오프셋 합성)

### 지도/비행 레이어
- `#camera > #map(canvas) + #trail(svg)` — 이 둘만 줌/이동에 함께 스케일됨
- `#pin`, `#plane` — 화면 고정(fixed), 크기 일정. `worldToScreen`으로 위치만 잡음
- 비행 중에만 표시: `body.flying` 클래스로 토글

### 핵심 함수
- `startGame()` — 목숨/점수 초기화, `body.flying` 추가, `goNext()`
- `goNext()` — 다음 나라 뽑고 `flyTo()`로 출발
- `pickCountry()` — 출제 덱에서 뽑기. 덱이 비면 `shuffle(COUNTRIES)`로 재충전(중복 출제 방지의 핵심)
- `flyTo(q)` — 베지어 곡선 비행 + 카메라 줌인→줌아웃→줌인. 도착 시 `idleAtDest()`
  - 핀 내용은 **출발 시 비우고, 도착 시(`idleAtDest`)에만 채운다** → 출발 순간 다음 나라 노출 방지
- `idleAtDest()` — 도착지 줌인, 핀(국기+나라명) 표시, 비행기를 도착지에 세워둠, 피드백 지움
- `placePin(x, y)` — 핀은 도시 지점 **오른쪽 위로 비켜서** 배치(`+24, -14`, bottom-left 기준).
  **생각 말풍선** 모양이라 꼬리가 삼각형이 아니라 왼쪽 아래(비행기 쪽)로 작아지는 점 두 개(`.pin-tail`).
  비행기를 가리지 않게 하려는 것이므로, 카드 크기나 오프셋을 바꾸면 겹치는지 꼭 확인할 것
- `checkAnswer()` — 채점. 정답→폭죽+goNext / 오답→목숨-1+미사일 연출
  - 정답 시 `deck.length === 0`이면 방금 맞힌 게 덱의 마지막 장 → `celebrateLap()`
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
- `sfx.launch / warn / flare / boom / pop` — 발사·경고음·플레어·폭발·폭죽
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
- 페이즈:
  - `chase` — 추격. 일부러 빗나간 각도로 출발해 크게 휘어 들어온다. 300px까지 접근하면 플레어 사출
  - `decoyed` — 플레어(`m.decoy`)를 쫓아감. 록온이 풀린 상태라 조준선을 그리지 않는다
    - `fatal=false`: 비행기와 `m.missDist`(110~190px) 이상 벌어지면 `evadeMissile()` → 유폭(회피 성공, 비행기 생존)
    - `fatal=true`: 기만탄에 닿으면 `reacquire`로 전환
  - `reacquire` — 재포착. 선회율을 점점 올려(`.14 → .34`) 확실히 명중시킨다 → `hitPlane()`
- `m.plane` — 회피 기동 중인 실제 비행기 위치. `m.home`(도착지) 기준으로 sin 파형 흔들림 + 뱅크 회전
- `drawLockOn()` — 비행기 주위 록온 조준선(가까울수록 조여들고 빨리 깜빡임)
- `evadeMissile()` — 작은 폭발(k=0.6) + 약한 셰이크, **520ms 뒤** 콜백. 비행기는 그대로 살아서 다음 나라로
- `hitPlane()` — 큰 폭발 + 강한 셰이크 + 비행기 숨김, **620ms 뒤** 콜백(게임오버)
- 경보 UI는 `body.alarm` 클래스로 토글 (`#danger` 붉은 비네트, `#warn` 경고 배너)
- 소요: 회피 약 **1~2초**, 격추 약 **2~3초**. 늘어지지 않게 `decoyed` 1300ms /
  `reacquire` 1500ms / 전체 5000ms 초과 시 강제 종료

## 온라인 랭킹 셋업 (Supabase)
이름별 최고 점수 랭킹. **키를 안 넣으면 이 브라우저에만 저장되는 로컬 랭킹으로 동작**하고,
아래 셋업을 마치고 키 두 줄만 채우면 전 세계 공용 랭킹이 된다. 게임 코드는 바꿀 게 없다.

1. [supabase.com](https://supabase.com)에서 프로젝트 생성 (무료 티어)
2. **SQL Editor**에 아래를 그대로 실행
   ```sql
   -- 점수는 append-only. 남의 기록을 덮어쓰거나 지울 수 없게 insert만 허용한다.
   create table public.scores (
     id         bigint generated always as identity primary key,
     name       text   not null check (char_length(name) between 1 and 12),
     score      int    not null check (score >= 0 and score <= 100000),
     created_at timestamptz not null default now()
   );
   alter table public.scores enable row level security;
   create policy "anon insert" on public.scores for insert to anon with check (true);
   create policy "anon select" on public.scores for select to anon using (true);

   -- 이름별 최고점만 집계한 랭킹 뷰 (게임은 이 뷰만 읽는다)
   create view public.leaderboard as
     select name, max(score) as score, max(created_at) as at
     from public.scores group by name;
   grant select on public.leaderboard to anon;
   ```
3. **Settings → API**에서 `Project URL`과 `anon public` 키를 복사
4. `index.html`의 `const RANK = { url: "", key: "" }` 두 줄을 채운다 (파일 상단 `── 랭킹` 블록)

**설계 메모**
- **왜 upsert가 아니라 insert-only인가** — anon 키는 공개되므로 upsert를 허용하면 아무나 남의 이름 점수를
  낮게 덮어쓸 수 있다. 전 기록을 쌓고 뷰에서 `max()`를 집계하면 그 공격이 원천 봉쇄된다.
- **어뷰징** — 클라이언트가 점수를 보내는 구조라 마음먹으면 가짜 점수를 넣을 수 있다.
  `check` 제약이 말도 안 되는 값만 거른다. 더 막으려면 Edge Function에 검증/레이트리밋을 두는 방향.
- **XSS** — 남이 입력한 이름을 그리므로 `renderRank()`는 반드시 `textContent`만 쓴다. `innerHTML` 금지.
- **anon 키 공개** — 정상이다. RLS로 권한이 묶여 있어 노출돼도 문제되지 않는다(service_role 키는 절대 넣지 말 것).

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
- ✅ (해결됨) 미사일이 안 보이던 버그: `worldToScreen(project(...))`로 객체를 통째 넘겨 NaN 좌표가 됐던 것.
  `worldToScreen(wp.x, wp.y)`로 수정.
- ✅ (해결됨) 미사일 연출 조잡함 → 호밍·플레어 기만·경보 UI·폭발/사운드까지 폴리싱 완료.
- 국기 핀 도착 시 팝인 지연 방지를 위해 `flyTo`에서 다음 국기를 `new Image()`로 프리로드.
- ⚠️ 지도가 120×60 블럭(1블럭 ≈ 3°)으로 저해상도라 **해안 도시 14곳은 바다 블럭 위에 찍힌다**
  (도쿄·로마·리스본·스톡홀름·자카르타·알제·튀니스·아크라 등). 데이터 오류가 아니라 지도 해상도 한계.
  거슬리면 `MAP_COLS/MAP_ROWS`를 키우거나 해당 좌표를 내륙으로 조금 밀면 된다.
- 넓은 데스크탑 화면에서 가장 잘 보임. 모바일도 동작하나 하단 입력바가 키보드에 가릴 수 있음.
- **테스트 팁**: 브라우저 탭이 백그라운드면 `requestAnimationFrame`이 멈춰 연출 검증이 안 된다.
  콘솔에서 `fxLoop(performance.now() + k*16.7)`를 직접 반복 호출하면 프레임을 수동으로 돌릴 수 있다.

## ▶ 다음 작업
- **Supabase 프로젝트 만들고 `RANK.url` / `RANK.key` 채우기** (그 전까진 로컬 랭킹으로 동작)
- 배포: GitHub Pages 또는 Cloudflare Pages + 도메인 연결
- **배포 직후 반드시**: `<head>`의 `og:url`(현재 빈 값)과 `og:image` / `twitter:image`를
  실제 도메인의 **절대 URL**로 교체. 상대 경로는 카톡·트위터 크롤러가 못 읽는다.
  README의 "플레이하기" 링크도 같이 채울 것
- 모바일 대응: 하단 입력바가 소프트 키보드에 가리는 문제
- 난이도 티어 — 100개국이 되면서 편차가 커졌다. 인구 상위권부터 나오다가 점수가 오를수록
  하위권(부르키나파소·베냉 등)이 섞이게 하면 자연스럽다 (`pop` 필드를 그대로 쓰면 됨)
- 난이도 상승(점수 오를수록 미사일 빨라지게 — `launchMissile`의 speed/turn을 score로 스케일)
- 모바일 입력바 가림 대응

## 튜닝 포인트 (자주 만지는 숫자)
- `MAX_LIVES` — 시작 목숨 (현재 2)
- `CELL` — 블럭 크기 (현재 14), `PALETTE` — 지도 색상
- `flyTo`의 `zoomScale = fit * 3.0`, `DURATION = 1800` — 카메라 확대율 / 비행 시간
- `launchMissile`의 `speed / accel / maxSpeed / turn` — 미사일 속도·선회 성능
- `chase`의 `dist < 300` — 플레어 사출 거리
- `missDist = 110 + Math.random() * 80` — 회피 시 미사일이 얼마나 빗나가서 터지는지
- `screenShake(power, ms)` — 흔들림 세기, `explode()`의 입자 개수 — 폭발 규모
- `sfx.*`의 `vol` 인자 — 효과음 볼륨
- `COUNTRIES[]` — 나라 추가/삭제. 위 "나라 목록 갱신(UN 인구 기준)" 절차를 따를 것

## 향후 아이디어 (미구현)
- 난이도/속도 상승, 최고점수 저장(localStorage)
- 모바일 최적화
- (원대한 목표) Electron으로 감싸 Steam 출시 — 단, 현재는 웹 전용으로만 개발 중

## 개발 히스토리 요약
객체식 4지선다 → 타이핑 입력 → 국기 이미지 → 세계지도 배경 + 비행기 이동 → 하단 입력바 →
블럭 지도 + 카메라 줌인아웃 → Enter 즉시 진행 → 목숨제 + 미사일 회피 →
미사일 연출 폴리싱(호밍·플레어 기만·경보 UI·폭발) + WebAudio 사운드 순으로 발전.
