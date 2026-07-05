# SIM AMIS — 작업 핸드오프 (다음 세션용)

> 마지막 업데이트: 2026-07-01 / SimEMR **v12** / 작성: Claude
> 새 세션 시작 시 이 문서 + `memory/project_sim_amis.md`를 먼저 읽으면 됨.

---

## 1. 프로젝트 개요
소아청소년과/응급의학 **전공의 시뮬레이션 교육용 단일 HTML EMR**.
학생(전공의)이 환자에게 검사·처방을 내고 결과를 해석하는 훈련. 교육자 모드에서 시나리오를 직접 제작.

- **개발자**: 비전공자 (코드 설명은 단계별·초보자 친화적으로)
- 한 파일 안에 HTML+CSS+JS+데이터(+base64 이미지) 전부 들어있음
- **배포 예정**: GitHub 정적 호스팅(GitHub Pages). 시나리오는 JSON으로 공유.

---

## 2. 파일 현황 (`/Users/minkyo/Documents/Claude/Projects/SIM_AMIS/`)

| 파일 | 설명 |
|---|---|
| **SimEMR_v12.html** | ⭐ 현재 작업 파일 (최신, 약 6MB) |
| SimEMR_v12_BACKUP.html | 백업 (이번 세션 대규모 작업 직전 상태) |
| **README.md** | 시나리오 제작자용 가이드 (기능·JSON 구조·AI 자동채우기) |
| **AI_시나리오_프롬프트.md** | AI(Claude/Gemini)에 통째로 복사해 줄 자체완결형 프롬프트 |
| index.html | 배포용 루트 진입점 (→ SimEMR_v12.html 리다이렉트) |
| .nojekyll / .gitignore | GitHub Pages 설정 (Jekyll 비활성 / 백업·DS_Store 제외) |
| HANDOFF.md | 이 문서 |

> ⚠️ 파일 약 6MB(프리셋 base64 이미지 한 줄). **전체 Read/Write 금지** → Edit(고유 문자열) 또는 bash targeted 수정.
> ⚠️ 큰 변경 전 백업: `cp SimEMR_v12.html SimEMR_v12_BACKUP.html`
> ⚠️ 저장소 폴더가 동기화 마운트라 **샌드박스에서 git 쓰기 불가**(Operation not permitted) → git은 맥 터미널에서 직접.

---

## 3. 코드 구조 (SimEMR_v12.html, `<script>` ~242행~)

| 위치(대략) | 내용 |
|---|---|
| `LAB_ITEM_DEF` (~244) | 검사항목별 참고치 {u,lo,hi} — 성인 고정. **Troponin I 상한 60** |
| `ORDER_TEMPLATES` (~290) | 처방명 → 결과항목 목록 |
| `IMAGING_TEMPLATES` (~315) | 영상 정상 판독문 (미설정 시 자동 반환, 16종) |
| `IMG_REGION_MAP` / `imgRegionsOf()` | **영상 부위 매핑** (8부위, 다부위검사 복수소속) |
| `ABGA_DEF` / `VBGA_DEF` (~340) | 혈액가스 참고치 (VBGA FiO2 없음) |
| `PRESETS` (~394) | 기본 시나리오 2개: 김한빛(호흡부전), 허○엘(trauma) |
| `IDB` 객체 / `save()` / `load()` | **IndexedDB 저장소** (async). 저장 실패 시 alert |
| `App` 객체 | 화면/처방/결과/교육자모드/미리보기 전부 |
| `fmtAge()` / `effAge()` | 나이 표시 / 유효나이(연+월/12) |
| `genNormalValue` / `CBC_AGE_REF` / `MARKER_NORMAL` | 미설정 검사 자동 정상값 |

### 핵심 함수 (이번 세션 추가)
- 영상: `onImgRegionPick`, `addImg`(부위→검사), `onImgAbnormalChange`(이상 일관성 팝업), `addRelatedImaging`
- 미리보기: `previewScenario`, `_injectPreviewBar`, `previewSkipTime`, `exitPreview`, `_clearPreviewUI`

---

## 4. 데이터 구조 (시나리오 JSON)
```
scenario = { id, name, description, patient, labOrders[], abga, vbga, imaging[] }
patient  = { id, name, age, ageMonths, sex, chiefComplaint, pastHistory, history,
             vitals:{SBP,DBP,HR,RR,SpO2,Temp}, mentalStatus:{type:'GCS',E,V,M,avpu} }
labOrder = { name, code, delayMinutes, resultSets:[{label, items:[{n,v}]}], isCulture?, failMessage? }
abga/vbga= { delayMinutes, resultSets:[{label, items:[{n,v}]}] } | null
imaging  = { name, delayMinutes, displayMode:'stack'|'pages', images:[base64],
             resultType:'image'|'report'|'both', report, abnormal }   // region은 name으로 파생
```
> 상세 스키마·항목명 전체 목록은 **README.md 6장** 및 **AI_시나리오_프롬프트.md** 참조.

---

## 5. 세션별 변경 내역 (요약)

### 2026-06-04 ~ 06-05
SimEMR+trauma 합치기, localStorage 오류 수정, 혈액검사 패널 구조 개편, BGA 에디터·드롭다운 개선, 미설정 검사=정상 자동반환, CBC 나이별 정상값(Nathan & Oski), JSON 내보내기/가져오기.

### 2026-06-06 (오전) — 검사/영상 정비
Chemical battery→Blood chemistry 통합, Culture 2회 제한, IMAGING_TEMPLATES 정리(PED 버전명·Whole body trauma 추가), 결과/추세 탭 분리, 열 리사이즈 제거, VBGA FiO2 제거, UA- 접두사 제거, FAST→Multiple focused US.
- 통합검사목록 가로스크롤 제거(--wl 414, table-layout:fixed)
- 영상 결과유형(이미지/판독/둘다) 추가

### 2026-06-06 (오후) — 대규모 개선 5종 ⭐ (이번 핵심)
1. **IndexedDB 전환**: localStorage→IndexedDB(`IDB`, store 'kv'). load/init async화, 최초 1회 localStorage 자동 이전. **저장 실패 시 alert(JSON 백업 유도)**. 프리셋은 이미지 제외 저장 후 PRESETS 복원.
2. **나이 연·월 분리**: 편집 세+개월(ed-page/ed-pmonth), `patient.ageMonths`. `fmtAge`/`effAge`. 0세3개월→0.25세→영아 CBC.
3. **영상 부위 연동**: `IMG_REGION_MAP`(머리/얼굴/경추/흉부/복부/골반/요추/전신; CT A&P=복부+골반, FAST=복부, Whole body=전신). 추가 UI 부위→검사. 정상/이상 토글 + 같은 부위 일관성 팝업(미설정 일괄추가).
4. **편집 미리보기**: "👁 미리보기"로 학생화면 임시실행(저장X, `S._preview`), "⏩ 결과 즉시 도착", "편집으로 돌아가기". `save()`에 preview 가드.
5. **상한마커 정상값**: `MARKER_NORMAL`(Total chol 170 / AST·ALT 20 / Procalcitonin 0.05 / D-dimer 0.20 / Troponin I 2 / CK-MB 1.5 / Myoglobin 40 / BNP 12). **Troponin I 상한 5→60**.

추가 산출물: **README.md**, **AI_시나리오_프롬프트.md**, 배포파일(index.html/.nojekyll/.gitignore).

---

## 6. 매번 실행 블록 (load() 직후, 항상 실행)
```
1. sc.imaging 이름 정규화 (FAST → Multiple focused US 등)
2. Routine urinalysis UA- 접두사 제거
3. VBGA FiO2 제거
4. Chemical battery → Blood chemistry 정규화 + 중복 제거
5. Blood chemistry에서 CRP·ESR·Procalcitonin·eGFR 제거
```
> 신규 필드(ageMonths, imaging.abnormal/region)는 읽을 때 falsy 기본값으로 처리 → 별도 마이그레이션 불필요.

---

## 7. 배포 메모 (GitHub Pages)
- 맥 터미널: `rm -rf .git`(샌드박스가 만든 불완전 폴더 정리) → `git init && git add -A && git commit` → GitHub 저장소 생성 → remote 추가 → push.
- Settings → Pages → Branch main / (root). 무료 Pages는 **public 저장소** 필요.
- https 자동 → IndexedDB·파일저장창·폴더업로드 정상.
- 버전 올릴 때 `index.html` 내 파일명 2곳 함께 수정. 캐시 시 ⌘⇧R 안내.

---

## 7-1. 2026-07-01 — 시나리오 4종 등록/업데이트 (다른 Cowork 프로젝트에서 진행)

> 별도 프로젝트("전공의 시뮬레이션 교육")에서 정리한 `00_시나리오_데이터_통합정리.xlsx`를 근거로 PRESETS 갱신.
> 백업: `SimEMR_v12_BACKUP_pre_4scenarios.html` (이번 변경 직전 상태).
> 변경 후 `new Function(스크립트 전체)` 문법 검사 통과 + PRESETS 파싱/이미지 개수 재확인 완료.

- **preset-hanbit (호흡부전·김한빛)**: history 보강, 검사가 원자료상 VBGA가 아니라 **ABGA**였음을 확인해 `vbga→null, abga`에 Stage1~3 3회차(pH/pCO2/pO2/HCO3/SaO2)로 이전. 기존 CXR 항목(이미지 없음)에 판독문(report, Rt LLL pneumonia) 추가.
- **scenario-trauma-suyeong (Major Trauma·허○엘)**: Blood chemistry의 **K 값을 2.5→3.5로 정정**(xlsx 확인값 Na/K 138/3.5). 실제 업로드된 X-ray/CT 이미지 6종(7장)은 **손대지 않음**.
- **scenario-seizure-parkgeon (신규)**: Seizure→Refractory SE→ICH 시나리오 신규 등록. VBGA 1회차(Established SE) + ABGA 1회차(RSI 후, vent 조절) + Brain CT 판독문(report 방식, 실제 영상 없음 — 우측 측두엽 ICH/AVM 의심 소견 텍스트).
- **scenario-sepsis-kimyejin (신규)**: 패혈성 쇼크 시나리오 신규 등록. VBGA 3회차(Stage1→3, 마지막 회차는 사용자가 제공한 실측값 pH7.02/pCO2 49/BE-13/HCO3 10/Lactate 8 그대로 사용) + CXR 판독문(report 방식, Rt LLL pneumonia 추정 — **신규 작성, 검토 필요**).
- **미반영 항목**: 패혈증 시나리오의 CBC(호중구감소 관련 구체 수치)는 근거 자료에 정확한 값이 없어 **의도적으로 비워둠**(추측 입력 금지 원칙). 필요 시 추가 입력 권장. Seizure의 CTA는 앱의 영상 카탈로그에 항목이 없어 별도 오더로 등록 못 함(판독문 텍스트에만 권고 문구 포함).

## 8. 다음 작업 후보
- [ ] **투약 기능** 구현 (바로 다음 — 사용자가 계획 알려줄 예정)
- [ ] 실제 브라우저에서 전체 흐름 육안 확인 (특히 IndexedDB 영속성, 영상 이상 팝업, 미리보기 진입/복귀)
- [ ] (옵션) 미리보기 중 EMR 상단 "세션 종료" 버튼 숨김 처리
- [보류] 소아 정상 참고치(lo/hi 나이대별) / VBGA AirDrop 전송

---

## 9. 작업 방식 메모
- 변경 후 항상 JS 문법 검사 (script 추출 → `new Function(src)`), 가능하면 순수함수 node 검증
- 6MB 단일 파일 → targeted 수정. base64 라인 Read 금지
- 복잡 작업은 dev-protocol: 전체 그림 먼저 합의 후 구현
- 브라우저 file:// 은 Chrome 확장이 차단 → 자동 UI 테스트 불가, 사용자 육안 확인 안내
