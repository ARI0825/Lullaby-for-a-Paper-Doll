# Lullaby for a Paper Doll — 프로젝트 안내

사용자는 한국어로 대화한다. 답변과 보고는 한국어로 쓴다.
스토리: 괴물들은 인형이 가정폭력을 겪은 기억을 동화 형태로 바꾼 것이다. 5개 챕터(12개 막), 엔딩 3종(배드 / 노말 / 해피).

## 작업 규칙 (사용자 선호)
- **GitHub 푸시는 사용자가 "푸시해"라고 할 때만 한다.** 먼저 푸시하지 않는다. 푸시 전에 한 번 묻는다.
- 저장소에는 `index.html`(게임), `test_full.html`(테스트용 빌드), `CLAUDE.md`만 올린다. 챕터별 별도 HTML 파일은 만들어서 올리지 않는다.
- 커밋을 만들 때는 시스템이 알려 주는 공동 작성자 줄을 붙인다. 강제 푸시(히스토리 재작성)는 사용자가 분명히 요청했을 때만 한다.
- 보고할 때는 확인한 것과 확인하지 못한 것(소리, 실제 조작감, 실시간 연출)을 구분해서 쓴다.
- 배포: GitHub Pages가 `main`의 루트를 그대로 서비스한다.
  - 게임: https://ari0825.github.io/Lullaby-for-a-Paper-Doll/
  - 테스트: https://ari0825.github.io/Lullaby-for-a-Paper-Doll/test_full.html

## 구조
- 모든 게임 코드는 `index.html` 하나에 들어 있다 (three.js 0.160을 CDN에서 불러오는 `<script type="module">`).
- 렌더링: 포스터라이즈/디더 후처리 셰이더(`postMat`), 종이책 위에서 진행되는 탑뷰.
- 소리: 외부 파일 없이 WebAudio로 생성하는 `AU` 객체 (`tone/grain/out/reverb`, `titleOn/titleOff`, `proOn/proOff`, `lullaby`, `endMusic` 등).
- 상태 머신 `G.state`: `title, busy, play, choice, pause, end, chapEnd, room(1인칭 집), rseq(문 연출)`.
- 레벨: `LEVELS` 12개 막, `CHAPTERS` 5개. 마지막 막(`no:12`, 긴 통로)을 지나면 `nextPage()`가 `enterRoom()`을 부른다.
  - 11막: 휠 확대로 작은 글씨를 읽는 퍼즐(`zspots`). 확대는 `G.zoom`(0.65 가까움 ~ 1.5 멂).
  - 12막: 어두운 긴 통로(`corridor:true`), 끝 영역(`endZone`)에 들어가면 집으로 넘어간다.
- 1인칭 집(`buildRoom`, `ROOMZ=-300` 위치에 별도 그룹): 현관(z≈26) → 긴 복도 → 거실(엔딩 문 3개, 북쪽 벽). 복도 양쪽에 방 4개(아이 방, 욕실, 엄마 방, 부엌). `MEMS`가 [E]로 살펴보는 물건, `DOORS`가 엔딩 문(차가운 문=배드, 포근한 문=노말, 연필 문=해피).
  - 카메라는 `RC={x,y,z,yaw,pitch}`, 이동은 WASD, 마우스는 클릭 없이도 시선이 돌아간다(클릭하면 포인터 고정).
  - 점광원은 8개 이하로 유지한다 (많으면 헤드리스/약한 GPU에서 렌더가 깨진다). 방별 미니 룸은 그룹 `visible`로 켜고 끈다.
- 문을 고르면 `enterDoor(key)` → `ROOMSEQ[key]` 연출(배드: 창문, 노말: 침대에 눕고 눈꺼풀 감김, 해피: 책상에서 연필로 쓰기) → `finishGame` → 엔딩 영상(`playCinematic`, 640x360 캔버스) → 책이 덮이는 연출(`bookClose`) → 엔딩 화면.
- "처음부터 다시 읽기"(`againBtn`)는 타이틀 화면으로 돌아간다.
- 디버그 훅: `window.__game`에 `G, P, L, RC, ROOM, ROOMZ, enterRoom, enterDoor, adv, step, finishGame, cineFrame` 등이 들어 있다.

## 레벨 그룹 주의
- `buildLevel`은 새 레벨을 만들 때 이전 레벨 그룹(`userData.levelGroup`)을 장면에서 지운다. 시작할 때 `disposeLevel()` 없이 `buildLevel`을 다시 부르면 이전 맵이 남는다.
- 엔딩 후 책 덮는 연출 때는 `hideLevelGroups()`로 모든 레벨 그룹을 숨긴다.

## 테스트 빌드 `test_full.html`
- `index.html`에 패치를 적용해서 만든다 (직접 편집하지 말고 `index.html`을 고친 뒤 다시 만든다).
- 패치 내용: 제목, 타이틀의 챕터/막 선택 버튼과 무적·"이전 장 진행 채우기" 체크박스, 피격 무시(`GOD`), 시작 시 `disposeLevel(); buildLevel(START)`, 시작 레벨 `START`. `?chap=2..5&act=1..N&god=0`, `?t= &s= &pencil=1 &scissors=1 &mother=1 &friends=rabbit,thread` 주소 옵션도 지원한다.
- `index.html`을 수정하면 `test_full.html`도 같은 패치로 다시 만들어 함께 커밋한다.

## 개발/테스트 팁 (Windows)
- 파일은 LF 줄바꿈이다. Python으로 고칠 때는 `open(..., encoding='utf-8', newline='')`를 쓴다. 한글이 든 스크립트는 `python -X utf8`로 실행한다.
- 헤드리스 Chrome 예: `chrome --headless=new --disable-gpu --use-angle=swiftshader --enable-unsafe-swiftshader --virtual-time-budget=60000 --screenshot=out.png file:///.../index.html`
- 장면을 시간 순서대로 찍으려면 `requestAnimationFrame`을 막고 `__game.adv(1/30)`로 직접 진행한 뒤, 마지막에 한 프레임만 그려서 캡처하는 방식이 안정적이다.
- 한 번의 명령이 2분을 넘으면 백그라운드로 돌리고 끝나기를 기다린다. Chrome을 동시에 여러 개 돌리지 않는다.
- 타이틀 앞의 "화면을 눌러 시작" 화면(`#tap`)이 자동화 테스트를 가릴 수 있으므로 테스트 스크립트에서 먼저 제거한다.
- 브라우저는 클릭·키 입력 전에는 소리를 낼 수 없다. 그래서 타이틀 앞에 "화면을 눌러 시작"이 있다.

## 아직 직접 확인하지 못한 것
- 소리 전반, 실제 걷는 조작감과 마우스 감도, 연출의 실시간 속도감, 책 덮는 움직임의 실제 느낌, 실제 GPU에서의 성능.
