Wish RP Manager 0.17.11 → 사용자 패치 0.17.12(TEST) · 원작자 READ용 변경보고서
작성일: 2026-10-03
용도: 원작자/사람 READ용 · 0.17.11 대비 사용자 패치 0.17.12의 실제 변경점·의도·보존계약을 빠르게 확인하는 문서

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0. 버전 정의
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[원작 기준]
- 0.17.11 = 원작자 제공 기준본
- 기준 파일: RP_Manager_0.17.11_BASELINE.user.js
- SHA-256: 89d51c962a5ac78154ede8768cc51bf58502a0693019cc01685b7c1817dbc78e

[사용자 패치 기준]
- 0.17.12(TEST) = 0.17.11을 베이스로 Core 1.5.4의 선택 안전기능 + CrackSafe의 고속 로그 수집 장점 + 응답교정 UX/경합 안전성 강화를 선택 이식한 현재 테스트판
- 기준 파일: RP_Manager_0.17.12_RefinerEnhanced_TEST_20261003.txt
- SHA-256: c17e08d00fafaf6aa96048b086f17572d38548b1a3fed66a8d57e62bf52381d0

※ 0.17.12는 원작자의 공식 버전이라는 뜻이 아니라 이 사용자 패치 계보에서 붙인 로컬 버전명이다.
※ 기능 비교 기준은 0.17.11 원본과 0.17.12(TEST) 기준본이다.

정적 diff 규모(0.17.11 원본 → 최종 0.17.12):
- unified diff hunk: 47개
- 추가 행: 572행
- 삭제 행: 62행
- 원본 전체를 갈아엎은 것이 아니라 일부 callsite 수정 + 후단 선택이식 블록 추가가 중심이다.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. 한 줄 요약
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

0.17.12는 RP Manager의 기억 구조·자료집·검색·SOURCE SHA·턴 조립 규칙은 그대로 두고,
“대용량 전체 RP 읽기 / 로그 다운로드 / 외부 JSON 복구 / 진단 / ZIP / 안전 취소” 계층과 “응답교정 검토 UX / 원자적 적용 / 수정창 경합 보호”를 강화한 버전이다.

가장 큰 체감 변화는 다음 세 가지다.

1) 전체 RP 읽기: 기존 50 messages/page 중심 → 300/page 우선 + 장애 시 50/page 복구
2) 전체 재구축 파일: 여러 TXT를 한꺼번에 브라우저로 던지던 방식 → ZIP 1개 권장/자동 다운로드
3) 응답교정: 개별 replacement 중심 검토 → 논리적 issue 단위 원자 적용 + 전체 응답 강조 미리보기 + 직접수정 보호 + Crack 수정창 경합 보호

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2. 0.17.11 대비 실제 추가·변경 기능
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[2-1. 고속 Full Read / 로그 다운로드]
- 기본 페이지 크기 300 messages/page.
- 호환 또는 장애가 있으면 50 messages/page로 복구.
- 첫 페이지뿐 아니라 “중간 페이지”에서 300 모드가 반복 실패해도 같은 cursor에서 50 모드로 전환.
- 401/403 인증 오류는 즉시 중단. 무의미한 fallback으로 숨기지 않음.
- 408/425/429/5xx/네트워크 계열은 bounded retry.
- retry: 최대 5회 × recovery 2라운드.
- exponential backoff: 1.2s → 2.4s → 4.8s → 9.6s, 최대 12s.
- 서버 Retry-After 헤더가 있으면 반영.
- 요청 timeout: 60초.
- 정상 페이지 사이 100ms 안정화 지연.
- 안전상한: 1,000,000 messages / 20,000 pages.
- 안전상한에 걸리면 일부 결과를 “전체 완료”라고 가장하지 않고 실패 처리.

기존 Manager에서 반드시 유지한 검사:
- cursor 반복 탐지.
- 빈 페이지 + nextCursor 오류 탐지.
- 동일 message ID가 다른 내용으로 다시 나타난 경우 원문 변경으로 중단.
- 조회 중 방/분기 변경 stale 처리.
- USER+AI 완료턴 조립 규칙(buildCompletedAiDialogueTurns).
- 페이지 경계의 미완성 턴 carry.
- 단일 carry 2,000,000자 안전선.

[2-2. RP 원문 내보내기 개선]
- 수동 업데이트 센터의 “RP 원문 내보내기”도 새 Full Read 엔진 사용.
- 최종 다운로드용 원문은 finalFresh physical read로 읽음.
- 진행 표시: 메시지 수 / 페이지 수 / 고속 300 또는 복구 50 모드.
- 읽기 중단 버튼이 실제 Abort 제어와 연결.
- 전체 읽기가 정상 완료되어야 TXT 다운로드 버튼 활성화.
- 저장 데이터·체크포인트는 원문 내보내기로 변경하지 않음.

[2-3. 전체 재구축 소스 읽기 강화]
- 전체 재구축 source 확정 시 cache 결과에 의존하지 않고 finalFresh physical read 수행.
- 재구축 파일 생성 중 읽기 단계는 취소 가능.
- 실제 저장/commit 단계가 시작되면 중간 취소 금지.
- SOURCE SHA-256, first/last turn key, turnCount, part hash, guide manifest 검증은 원래 Manager 규칙 유지.

[2-4. 전체 재구축 다운로드 안정화]
- 여러 PART TXT를 동시에 자동 다운로드하던 경로 제거.
- PART가 여러 개면 “Wish-전체재구축-TXT.zip” 한 개를 권장.
- ZIP은 STORE 방식, UTF-8 파일명, CRC32 사용.
- 개별 TXT 받기 유지.
- “TXT 순차 받기” fallback 유지.
- ZIP 자동 생성은 예상 원문 약 192MiB 이하에서 사용.
- 약 256MiB 이상은 수동 ZIP 생성 전 경고.
- 브라우저의 다중 다운로드 차단으로 일부 PART가 누락되는 문제를 줄이기 위한 변경.

[2-5. Full Read single-flight]
- 같은 방의 전체 RP를 여러 기능이 동시에 요구하면 물리 Full Read 하나를 공유.
- subscriber 한 명만 취소하면 다른 요청은 유지.
- 모든 subscriber가 취소하면 물리 요청 abort.
- abort 중인 오래된 flight에 새 caller가 잘못 합류하지 않도록 분리.

[2-6. Full Read 캐시]
- 캐시 TTL: 120초.
- 최근 50 메시지 frontier signature가 같을 때만 재사용.
- <=12MiB: deep snapshot.
- 12~36MiB: shallow snapshot.
- >36MiB: full-history cache 사용하지 않음.
- 메시지 mutation 성공 시 해당 chatId 캐시 hard invalidation.
- append/reroll/generation 변화는 frontier mismatch로 캐시 폐기.
- finalFresh=true 호출은 캐시와 shared flight를 우회하고 새 physical read 수행.

[2-7. 최종 원문 검증 callsite 강화]
다음 종류의 “적용 직전 사실 판정”은 finalFresh를 요구하도록 변경했다.
- 전체 수동 구축 source 확인.
- 통합 이어갱신 source 확인.
- 복구 작업 source 확인.
- full/custom 수동 업데이트 source 확인.
- 전체 재구축 파일 생성 source.
- 전체 재구축 JSON 적용 전 source 재검증.

목적: 캐시가 빠르더라도 최종 저장 판정은 실제 최신 RP로 다시 확인하기 위함.

[2-8. JSON Repair]
- 전체 재구축 JSON / 통합 이어갱신 JSON의 로컬 JSON·schema·필드 형식 오류는 인앱 수정창에서 고칠 수 있음.
- 최대 편집 크기: 2,000,000자.
- 수정한 뒤 반드시 원래 parse/prepare/Diff/commit 검증 경로를 다시 통과.
- source hash, 다른 방, 원문/분기 변경, 지침 hash, stale/revision 오류는 Repair로 우회 금지.

[2-9. Hold / Resume]
- 다른 작업이 진행 중일 때 붙여넣은 JSON을 바로 버리지 않고 창 메모리에 보관.
- 작업이 끝난 뒤 계속할 수 있음.
- 다만 방, route, job identity가 바뀌면 재개 금지.

[2-10. Full Read 진단]
처리·오류 기록에 다음 진단 추가:
- physical reads / completed / failed.
- cancelled / stale / error 분리.
- 300 시도 / 300 실패 / 50 fallback.
- transient retry / recovery round.
- cache hit/miss/invalidation.
- single-flight join.
- 마지막 작업의 mode/pages/messages/elapsed/error.
- 진단 복사 / JSON 저장.
- RP 본문은 telemetry에 저장하지 않음.

[2-11. Lazy Blob / 다운로드 수명 관리]
- 재다운로드용 TXT는 필요할 때만 Blob/ObjectURL 생성.
- 사용 후 revoke.
- ZIP 중단/닫기 시 임시 URL 해제.
- 기존 최초 단일 TXT 다운로드 동작은 불필요하게 장시간 Blob을 붙잡지 않도록 유지.

[2-12. 인채팅 “🪽 주입” 확인]
- 현재 pending carrier 메시지 옆에 “🪽 주입” 버튼 표시.
- 클릭 시 기존 reverifyPending을 사용해 서버 raw를 다시 확인.
- 별도 background polling 추가 없음.
- MutationObserver/event 기반 UI 갱신.
- 이 inspector 자체는 DB mutation/patchMessage를 수행하지 않음.


[2-13. 응답교정 RefinerEnhanced]
- 기존 0.17.12의 서버 source fingerprint 검증, verdict/category/severity/evidence, 좌우 before/after 비교, AI bridge 저장/read-back 검증은 유지.
- 전체 응답 문맥 안에서 실제 변경부만 강조하는 최종 미리보기 추가.
- 삭제되는 내용은 〔삭제〕 마커로 표시.
- 교정 체크/해제 후 긴 미리보기의 보던 위치를 가능한 한 유지하는 scroll anchor 추가.
- 명시적 직접수정 모드 추가. 직접수정 중에는 issue 체크박스/전체선택을 잠가 사용자 편집본이 selection rerender로 덮이지 않게 함.
- 기본 출력계약을 issues[].replacements[]로 확장하고, 하나의 논리적 issue 안의 복수 replacement를 체크박스 하나로 원자 선택/적용.
- legacy top-level replacements[]도 계속 허용.
- auto-apply도 replacement 단위가 아니라 issue 전체가 조건을 만족할 때만 원자 적용.
- partial auto-apply 뒤 남은 issue index를 0..N-1로 재정렬해 metadata 연결 오류 방지.
- Crack 네이티브 수정창이 열려 있으면 닫힘을 기다린 뒤 최신 서버 원문 재확인.
- 같은 message ID인데 본문이 사용자가 직접 바뀐 경우 기존 교정 제안 폐기.
- 완전히 새로운 최신 AI 응답이 생긴 경우 stale 교정을 적용하지 않고 최신 응답 재검수.
- 기존 assertReviewSource 서버 stale 검증과 bridge 저장/read-back 검증은 제거하지 않음.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
3. 의도적으로 바꾸지 않은 것
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

다음은 0.17.11의 구조를 정본으로 유지했다.

- IndexedDB schema / DB version 2.
- 기억 슬롯 구조.
- 날짜로그 구조.
- 인물정보 구조.
- 타임라인 구조.
- 자료집/Lore 구조와 WISH_LORE_V2.
- 기억 검색/recall 알고리즘.
- alias/triggers/entities.
- 핵심 대사 exactQuote 구조.
- 주입 엔진의 핵심 동작.
- SOURCE SHA-256 의미.
- FULL_REBUILD_FORMAT = wish-rp-manager-full-rebuild-1.0.
- buildCompletedAiDialogueTurns의 정사 턴 조립 의미.
- 전체 재구축 영역별 Diff/선택 적용.
- 기존 전문 지침과 프롬프트.
- 응답교정의 기존 verdict/category/severity/evidence 의미.
- 응답교정의 서버 source fingerprint stale guard.
- RP Manager AI bridge의 writeReviewedResponse/read-back 검증과 source mutation stale 전파.

Core 1.5.4에서 통째로 가져오지 않은 것:
- Core DB/schema.
- Core WishHistory 객체 전체.
- Core UI shell.
- Core prompts/지침.
- Core NativeBundles/Lore API.
- Core Quick Injection.
- Core recall 알고리즘.
- Core RoomJobFSM 전체.
- Core MemoryDiff 전체.

핵심 원칙:
“Core/CrackSafe를 RP Manager에 합친 버전”이 아니라,
“RP Manager를 그대로 두고 안전·성능 계약만 선택 이식한 버전”이다.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
4. 왜 0.17.12에서 고쳤는가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

사전 회귀에서 확인된 실제 문제:

1. 300/page가 첫 페이지 이후 중간에서 실패하면 기존 TEST가 복구하지 못하고 종료.
2. 50/page fallback 시 maxPages=1000이면 50,000 messages 부근에서 구조적으로 끊길 수 있음.
3. 20초 timeout은 대형 300-page 응답에 너무 짧을 수 있음.
4. 3회 고정 retry는 429/5xx 순간 장애에 취약.
5. Retry-After 미반영.
6. RP 원문 내보내기의 실제 읽기 진행/취소 UX 부족.
7. 전체 재구축 PART 여러 개를 자동 다운로드하면 Chromium/Tampermonkey에서 일부 파일이 차단될 수 있음.

0.17.12는 위 문제를 먼저 회귀로 재현한 뒤 수정했다.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
5. 회귀테스트 결과
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

현재 정적/합성 결과:
- JavaScript syntax: PASS.
- 응답교정 정적 보호계약: 36/36 PASS.
- 응답교정 기능 합성: 12/12 PASS.
- Crack 네이티브 수정창 경합 합성: 6/6 PASS.
- 정적 보호계약: PASS.
- Full Read / cache / single-flight / 무결성: PASS.
- 중간 300 장애 → 같은 cursor에서 50 복구 → 결과 동일: PASS.
- 50/page 1,001페이지 초과 완주: PASS.
- permanent 503 bounded failure: PASS.
- cursor 반복 / 빈 page+cursor / duplicate-ID content change / stale / 2M carry guard: PASS.
- JSON repair 우회방지: PASS.
- cache 12/36MiB 경계: PASS.
- STORE ZIP UTF-8 filename / BOM / CRC / cancel / Blob release: PASS.

합성 요청 수:
- 240 messages: 5 → 1 (-80.0%).
- 2,400 messages: 48 → 8 (-83.3%).
- 20,000 messages: 400 → 67 (-83.3%).

주의:
실제 소요시간은 Crack 서버 응답속도, 429, 메시지 크기, 브라우저 상태에 따라 달라진다.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
6. 아직 라이브에서 확인할 것
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

- 실제 Crack 서버가 장시간 300/page를 안정적으로 허용하는지.
- 수만~수십만 messages 방에서 끝까지 완주하는지.
- 실제 429 Retry-After 패턴.
- 네트워크 단절 후 recovery/fallback UX.
- 브라우저가 ZIP 자동 다운로드를 막는 경우 재다운로드 UX.
- 장시간 사용 후 캐시/Blob 메모리 누수 여부.
- 긴 RP 응답에서 전체 강조 미리보기 렌더/스크롤 체감.
- issue 체크/해제 반복 시 scroll anchor 유지.
- 직접수정 진입/취소/적용 UX.
- 실제 Crack 네이티브 수정창과 자동검수의 DOM 경합.
- 모바일/좁은 화면의 응답교정 팝업 레이아웃.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
7. 다음 원작자 업데이트 때 반드시 기억할 것
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

새 원작자 버전이 나오면 0.17.12 전체 코드를 새 버전에 덮어쓰지 않는다.
새 원작자 버전을 정본으로 두고, 이 문서와 AI 인수인계서를 이용해 아래 기능 계약만 다시 판정·이식한다.

특히 새 원작자가 이미:
- 300/page.
- retry/backoff.
- single-flight.
- cache.
- ZIP.
- JSON repair.
- source fresh verification.

등을 구현했다면 동일 기능을 중복 추가하지 않는다.

응답교정 쪽도 새 upstream이 이미 동등 이상으로 구현했다면 다음을 중복 이식하지 않는다:
- 전체 응답 변경부 강조 미리보기
- scroll anchor
- 직접수정 모드/선택 잠금
- logical issue atomic apply
- native edit stale guard

upstream 구현이 동등하거나 더 강하면 upstream을 유지하고 사용자 패치 코드는 생략한다.
기존 upstream 구현이 동등하거나 더 강하면 upstream을 유지하고 사용자 패치 코드는 생략한다.

최종 판단 우선순위:
1. 새 원작자 버전의 기능과 버그 수정 보존.
2. 0.17.12에서 추가한 안전계약 보존.
3. 중복 구현 제거.
4. 성능 최적화.
5. UI 편의 기능.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
8. 기준 파일 식별
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

0.17.11 원본 SHA-256
89d51c962a5ac78154ede8768cc51bf58502a0693019cc01685b7c1817dbc78e

0.17.12(TEST) 기준본 SHA-256
c17e08d00fafaf6aa96048b086f17572d38548b1a3fed66a8d57e62bf52381d0

이 SHA가 다르면 “동일한 기준본”이라고 가정하지 말고 실제 diff를 다시 확인할 것.