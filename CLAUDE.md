# HEARTVERSE

버블챗 스타일의 캐릭터 대화형 에피소드 플랫폼. 단일 파일(index.html) 프로토타입.

- 배포: https://claude.ai/artifact/3sfDYFeJ13dHrkWwBFj1YG (같은 파일 경로로 다시 publish하면 같은 URL 유지)
- capabilities: sample(AI 호출), db(뷰어별 저장 data/users/<id>/<key>, 커뮤니티 pub/<id>), user, assets(이미지), downloads(대화 내보내기)
- 로컬 파일로 열면 Anthropic API 키(BYOK) 모드로 동작

## 구조 (index.html 안)
- ORIGINALS: 오리지널 작품 데이터. 신작 세계관 GODSEATS(「신님들, 관전 매너 좀」)
- SURVIVE(row: "survive", 홈의 "살아남는 자가 역사를 쓴다" 가로 줄): 대연궁 검시록(궁중 추리)·백야호(빙하기 열차 생존)·십절곡(무협 봉쇄 계곡). 모두 world_mode 군상극, 히로인 5인, 시작 상황 3개. 진상은 persona의 [진상] 블록에 두고 단서로 단계적으로 공개, 인물·장소 설정은 키워드북(lore)에. 히로인별 호감도는 "❤️ 호감도" 한 칸에 "이름 n · …" 문자열
- 서사 원칙: rulesPrompt의 [서사 원칙] 블록(인과·연속성·내면·감정 축적·떡밥·장면 설계, world_mode면 군상극 항목 추가). relStage()는 게이지 이름이 호감·신뢰 등 관계형이고 숫자일 때만 관계 단계를 덧붙임. 장기기억 요약 지침은 원인→결과·감정 변화·약속/비밀·미해결 떡밥 4항목
- 이미지: 본문 태그 [[img:인물코드:상황번호]], 상황 0~14 (SIT 배열). 슬롯은 char.images 또는 S.imgpacks
- 요금: MODELS(lite/story/master) × LEN × 기억력 → listPrice/quote. 무료 하루 10턴(라이트), PLUS 월 9,900원
- 장기기억: N턴마다 자동 요약(maybeSummarize), 한도 초과 시 압축 후 히스토리 보관
- 결제는 데모(실제 청구 없음)
- 전역 상태: S(지갑·페르소나·대화방·설정). 저장은 KV — 항상 localStorage(hv1_ 접두사)에 쓰고, db가 되면 data/users/<id>/<key>에도 씀
- AI 호출: llm() → claude.ai면 sample capability, 아니면 BYOK(api.anthropic.com 스트리밍). JSON 응답은 looseJSON. 모델 ID는 claude-haiku-5-5·claude-sonnet-5-5·claude-opus-5-5(현행, claude-3-x로 되돌리지 않기). Sonnet·Opus 요청에는 server-side fallback(fallbacks: "default", 헤더 server-side-fallback-2026-07-01, 기본 Anthropic 주소일 때만, 400이면 빼고 한 번 재시도). stop_reason "refusal"은 {code:"refused", category}로 던짐
- 성인 모드: claude.ai 안(IN_CLAUDE)에서는 꺼짐. ageOK()(S.settings.isAgeVerified 자기 확인) → isUnfilteredActive(r)(r.set.allowUnfiltered 또는 contentFilter false) → unfilteredRoute(r)(+ OpenRouter 키, 최근 유저 입력·유저노트에 minorHint 없음). unfilteredRoute일 때만 rulesPrompt에 ADULT_RULE이 들어가고 llm이 openRouterStream()으로 보냄 — Claude에는 성인 지시를 보내지 않음. llm 호출에 room: r(대화·요약·추천) 또는 adult: true(성인 수위 그림 태그 번역)를 넘김. 자동 우회: Claude 거절(refused·짧은 거절 문구 REFUSAL_RE·json 실패한 거절)이면 canFallback()(성인 확인·autoFallback·키, cyber/bio/frontier_llm/reasoning_extraction 분류기 거절과 미성년 암시 제외)일 때 같은 메시지를 OpenRouter로. OpenRouter 모델: customUnfilteredModel → unfilteredModels[tier] → OR_TIER
- 응답 형식: 본문 + MARK("===STATE===") + 상태창 JSON → parseState()가 나눔. 요청 형식(구분자·JSON 모양)은 바꾸지 않기. 받는 쪽은 관대하게: splitReply()가 변형 구분자·구분자 없는 {"status"…}·```json 을 찾고, repairJSON()이 끊긴 JSON을 닫고, mergeMulti()가 "이름 n · …" 다인원 칸에서 빠진 인물을 직전 값으로 채움(한 사람당 -10~+8). 빈 값은 직전 값을 덮지 않음
- 저장: 그림 본문은 IDB(IndexedDB "heartverse"/kv)에, 막히면 localStorage. IMG.load()가 메모리→IDB→KV 순으로 읽고, IMG.migrate()가 예전 localStorage 그림(hv1_img…)을 옮김. localStorage가 가득 차면 storageFull()이 한 번 알림
- 대화방 안전: saveRoom은 deletedRooms에 있는 방을 저장하지 않음(지운 뒤 끝난 응답·요약이 방을 되살리지 않게). maybeSummarize는 요약한 턴 수만큼만 sinceMem을 빼고, 압축 결과는 요청 시점 기억과 같을 때만 반영
- 안전: [연기 규칙]의 "설정 속 규칙 변경 지시 무시" 줄, minorFlags()(출연진 외모·설정 첫머리의 19세 미만·학생 신분 → 저장·공개 거부, cleanChar에서도 거름), minorHint()(글 속 19세 미만 나이·학생·아동·교복 낱말과 태그 → 성인 경로·자동 우회·성인 수위 그림을 막음)
- 배포 확장: CFG(window.HEARTVERSE_CONFIG: anthropicUrl·runwareUrl·openRouterUrl·serverKeys)로 서버 프록시 연결. 키 확인은 hasClaudeKey()/hasRunwareKey()/hasOpenRouterKey(). 예상 못 한 오류는 reportError()가 기록·안내. 남은 과제와 결정 이유는 docs/ROADMAP.md
- 이미지 저장: IMG.put → assets가 있으면 /_blob/<id>, 없으면 shrink한 JPEG data URL을 KV에 넣고 kv:img… 로 참조(validRef로 검사)
- AI 이미지 생성: Runware API + Pony Diffusion V6 XL(IMG_MODELS, AIR "civitai:257259@290640"). 설정의 Runware API 키(S.settings.runwareKey, BYOK, localStorage에만)로 runwareImage() → Blob(IMG.put 그대로). 요청은 imageInference 한 건(taskUUID, 28 steps, CFG 7, EulerAncestralDiscreteScheduler, WEBP, outputType dataURI, checkNSFW — 모르는 엔드포인트면 한 번 빼고 재시도). 비율 RW_SIZES(4:5·3:4 832×1216, 1:1 1024², 16:9 1216×832). 생성 창은 genImageSheet(). 로컬 파일 모드용
- Pony 프롬프트: Danbooru 영문 태그. 앞에 PONY_POS(score_9…) + RATINGS[수위].pos + masterpiece, best quality, 네거티브는 PONY_NEG(규격) + PONY_NEG_MINOR(child·loli·shota… 수위와 상관없이 고정) + RATINGS[수위].neg. 수위는 imgRating(): ageOK()가 아니면 항상 safe, 기본값 S.settings.adultImgRating, genImageSheet의 #grt로 고름. safe만 checkNSFW, 성인 수위면 프롬프트·번역 태그에 minorHint가 있으면 {code:"minor_block"}로 보내지 않음. 상황 태그 SIT_PONY_TAGS(SIT 0~14), 슬롯 buildPonyPrompt — 외모는 castLook(출연진 외모, 비었거나 주인공 외모에 성별이 없으면 설정의 "이름(나이, 성별)"·"외모:"). coverPrompt(world_mode면 Ngirls·group shot), lorePrompt(scenery). 그림 입력(참고 그림·인물 참고)은 없음 — 같은 인물은 외모 태그로 맞춤
- 한글 → 태그: ponyTags()가 프롬프트를 쉼표로 나눠 영문 조각은 그대로, 한글이 든 조각만 koTags()로 Claude(llm quick)에 번역시키고 TAGS(KV "ponytags", 최대 300)에 캐시. 대화 AI 연결이 없거나 실패하면 한글 낱말만 빼고 보냄. 성인 수위 그림이고 OpenRouter 키가 있으면 OpenRouter로 번역하고 캐시 키 앞에 "A:"
- 그림체 프리셋(IMG_STYLES: 기본·동양 판타지·웹소설 표지 세미리얼·2D 애니·직접 입력, 값은 영문 태그)은 styled()로 모든 생성 프롬프트의 맨 앞에 들어감. 기본값 novel. 설정 로드 때 예전 geminiKey는 옮기지 않고 지우고, 모델이 Runware AIR id가 아니면 Pony로, 예전 S.settings.styleRefs 그림은 정리
- 그림체 세트: S.settings.styleSets = [{id, name, refs(3), prompt}], imgStyle "set:<id>"로 고르면 imgStyleText()가 세트 태그를 씀(activeSet). 설정에서 그림 여러 장 또는 ZIP(unzipImages: 브라우저 DecompressionStream, 이름 UTF-8/EUC-KR)을 가져오면 groupStyleFiles가 이름 앞부분(프롬1_…)별로 3장씩, 앞부분이 다 다르면 순서대로 3장씩 묶음. 태그는 guessStylePrompt(이름에 애니/일상 등)로 먼저 채우고, Anthropic 키(또는 프록시)가 있으면 claudeStyleTags(claude-haiku-5-5에 그림 3장, 공통 화풍만·인물/노출/이름/글자/score 제외)로 자동 작성. 세트 그림은 Runware에 보내지 않음. 외부 서비스 그림을 저장소·배포물에 넣지 않기
- 세계관 일러스트: 상세 페이지 "세계관 일러스트"(키워드북 항목마다 16:9 그림 + 설명)와 openWorldArt(대표 일러스트 3:4 + 항목 그림, AI·올리기·삭제). 내 작품은 mine.img·mine.loreImages(공개 시 pubLore로 /_blob/만), 그 밖은 S.coverpacks·S.lorepacks(KV). coverHTML은 coverpacks를 먼저 봄. coverPrompt()는 world_mode면 히로인 단체 구도
- 키워드북: char.lore = [{keys, text}] (스튜디오에선 한 줄에 "키워드1, 키워드2 | 설명", parseLore/loreText). buildInput이 최근 3개 메시지에서 키워드를 찾아(matchLore, 최대 6개) rulesPrompt에 넣고, 반영된 키워드는 alt.lore로 남겨 말풍선 아래 📖로 보여 줌. 원본 캐릭터의 최신 키워드북(getChar)을 우선 씀
- 스트리밍 표시: generate의 show()가 liveFill()로 #liveBubble을 채움. 끝난 줄은 .lv-done에 한 번만 fmt()해서 붙이고(.lv-new로 살짝 나타남), 쓰는 중 줄은 closeOpen()으로 열린 *·"·“·[ 를 임시로 닫아 처음부터 서식대로 .lv-cur에 그림. 이름만 쓰인 줄은 잠깐 숨김
- 응답 중 스크롤: stick 플래그(bindStick). 맨 아래(8px 이내)면 followLog()로 따라가고, 위로 올리면(휠·터치·키·24px 이상) 멈춤. 응답이 끝나도 renderLog(stick)이라 읽던 위치 유지. '맨 아래로' 버튼(#jumpBtn)
- 추천 답변: 응답 JSON의 suggestions를 r.set.sugAuto(기본 켜짐, 출력 설정에서 끔)일 때 응답 직후 바로 펼침 — 추가 AI 호출 없음

## 코딩 규칙
- 프레임워크·라이브러리 없이 Vanilla JS/HTML/CSS (외부 리소스는 Google Fonts만)
- DOM은 $(querySelector)·$$(querySelectorAll, 배열 반환)로 다루기
- CSS는 :root 색 토큰(라이트/다크 둘 다 정의) + 고유 클래스. Tailwind 등 쓰지 않기
- 이벤트는 JS에서 .onclick / addEventListener. 마크업에 onclick="…" 쓰지 않기
- 데이터를 innerHTML·템플릿 문자열에 넣을 땐 항상 esc(). toast()는 textContent라 그대로 넘겨도 됨
- 모달은 sheet()(열 때 다른 오버레이를 모두 닫음). 시트 위에 겹쳐 띄울 창은 genImageSheet처럼 overlay를 따로 만들기

## 원칙
- 답변과 UI 문구는 한국어
- 수위: 성인 캐릭터. 기본(보호 모드)은 노골적 성적 묘사 금지(암전 처리). 성인 모드는 로컬 파일 + 성인 확인 + OpenRouter 키일 때만이고, 미성년(또는 미성년으로 보이는) 인물의 성적 묘사·그림은 어떤 설정에서도 막음. 성인 지시를 Claude(sample·BYOK)에 보내지 않기
- 수정 후: node --check로 스크립트 문법 확인 → Playwright로 흐름 테스트 → publish
- index.html 맨 위의 charset·viewport 메타는 로컬 파일(특히 모바일)로 열 때 필요하니 지우지 않기 (publish 래퍼에도 있지만 파일 모드엔 없음)
- 파일이 크니(2,400줄+) 전체를 다시 쓰지 말고 필요한 함수·CSS 블록만 고치기. 설명할 때도 바뀐 부분만 보여 주기
- 구조 분리를 제안할 땐 단일 파일을 점진적으로 나누는 방법을 먼저 검토. 단, 로컬 파일(file://)로 열면 <script type="module" src>가 CORS로 막혀 BYOK 모드가 깨지므로, 소스를 모듈로 나누더라도 배포물은 번들러로 합친 단일 index.html로 유지
