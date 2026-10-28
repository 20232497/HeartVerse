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

## 원칙
- 답변과 UI 문구는 한국어
- 수위: 성인 캐릭터, 노골적 성적 묘사 금지(암전 처리)
- 수정 후: node --check로 스크립트 문법 확인 → Playwright로 흐름 테스트 → publish
