# HEARTVERSE

버블챗 스타일의 캐릭터 대화형 에피소드 플랫폼. 단일 파일(index.html) 프로토타입.

- 배포: https://claude.ai/artifact/3sfDYFeJ13dHrkWwBFj1YG (같은 파일 경로로 다시 publish하면 같은 URL 유지)
- capabilities: sample(AI 호출), db(뷰어별 저장 data/users/<id>/<key>, 커뮤니티 pub/<id>), user, assets(이미지), downloads(대화 내보내기)
- 로컬 파일로 열면 Anthropic API 키(BYOK) 모드로 동작

## 구조 (index.html 안)
- ORIGINALS: 오리지널 작품 데이터. 신작 세계관 GODSEATS(「신님들, 관전 매너 좀」)
- 이미지: 본문 태그 [[img:인물코드:상황번호]], 상황 0~14 (SIT 배열). 슬롯은 char.images 또는 S.imgpacks
- 요금: MODELS(lite/story/master) × LEN × 기억력 → listPrice/quote. 무료 하루 10턴(라이트), PLUS 월 9,900원
- 장기기억: N턴마다 자동 요약(maybeSummarize), 한도 초과 시 압축 후 히스토리 보관
- 결제는 데모(실제 청구 없음)
- 전역 상태: S(지갑·페르소나·대화방·설정). 저장은 KV — 항상 localStorage(hv1_ 접두사)에 쓰고, db가 되면 data/users/<id>/<key>에도 씀
- AI 호출: llm() → claude.ai면 sample capability, 아니면 BYOK(api.anthropic.com 스트리밍). JSON 응답은 looseJSON
- 응답 형식: 본문 + MARK("===STATE===") + 상태창 JSON → parseState()가 나눔. 렌더링을 고쳐도 이 구분자와 파싱은 건드리지 않기
- 이미지 저장: IMG.put → assets가 있으면 /_blob/<id>, 없으면 shrink한 JPEG data URL을 KV에 넣고 kv:img… 로 참조(validRef로 검사)
- AI 이미지 생성: 설정의 Gemini API 키(BYOK, localStorage에만 저장)로 geminiImage() 호출, 생성 창은 genImageSheet(). 이미지 슬롯(칸별 AI·빈 칸 채우기)·표지·페르소나에서 사용. 로컬 파일 모드용

## 코딩 규칙
- 프레임워크·라이브러리 없이 Vanilla JS/HTML/CSS (외부 리소스는 Google Fonts만)
- DOM은 $(querySelector)·$$(querySelectorAll, 배열 반환)로 다루기
- CSS는 :root 색 토큰(라이트/다크 둘 다 정의) + 고유 클래스. Tailwind 등 쓰지 않기
- 이벤트는 JS에서 .onclick / addEventListener. 마크업에 onclick="…" 쓰지 않기
- 데이터를 innerHTML·템플릿 문자열에 넣을 땐 항상 esc(). toast()는 textContent라 그대로 넘겨도 됨
- 모달은 sheet()(열 때 다른 오버레이를 모두 닫음). 시트 위에 겹쳐 띄울 창은 genImageSheet처럼 overlay를 따로 만들기

## 원칙
- 답변과 UI 문구는 한국어
- 수위: 성인 캐릭터, 노골적 성적 묘사 금지(암전 처리)
- 수정 후: node --check로 스크립트 문법 확인 → Playwright로 흐름 테스트 → publish
- 파일이 크니(2,400줄+) 전체를 다시 쓰지 말고 필요한 함수·CSS 블록만 고치기. 설명할 때도 바뀐 부분만 보여 주기
- 구조 분리를 제안할 땐 단일 파일을 점진적으로 나누는 방법을 먼저 검토. 단, 로컬 파일(file://)로 열면 <script type="module" src>가 CORS로 막혀 BYOK 모드가 깨지므로, 소스를 모듈로 나누더라도 배포물은 번들러로 합친 단일 index.html로 유지
