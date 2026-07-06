# SIM AMIS — 작업 핸드오프 (다음 세션용)

> 마지막 업데이트: 2026-07-06 / SimEMR **v12** / 작성: Claude
> 새 세션 시작 시 이 문서 + `memory/project_sim_amis.md`를 먼저 읽으면 됨.
> ⭐ **구조가 바뀌었습니다**: 단일 HTML → **껍데기 HTML + 외부 JSON**. 최신 요약은 아래 **7-3장**부터 읽을 것.

---

## 1. 프로젝트 개요
소아청소년과/응급의학 **전공의 시뮬레이션 교육용 단일 HTML EMR**.
학생(전공의)이 환자에게 검사·처방을 내고 결과를 해석하는 훈련. 교육자 모드에서 시나리오를 직접 제작.

- **개발자**: 비전공자 (코드 설명은 단계별·초보자 친화적으로)
- 한 파일 안에 HTML+CSS+JS+데이터(+base64 이미지) 전부 들어있음
- **배포 예정**: GitHub 정적 호스팅(GitHub Pages). 시나리오는 JSON으로 공유.

---

## 2. 파일 현황 (`/Users/minkyo/Documents/Claude/Projects/SIM_AMIS/`) — 2026-07-06 개편

| 파일 | 설명 |
|---|---|
| **SimEMR_v12.html** | ⭐ 껍데기(플레이어) 약 **0.15MB**. 시나리오 데이터 없음. `scenarios/*.json`을 fetch |
| **scenarios/manifest.json** | 로딩할 시나리오 파일 목록 `{scenarios:[...]}` |
| **scenarios/respiratory-hanbit.json** | 김한빛 호흡부전 (CXR 사진 1장 포함) |
| **scenarios/trauma-suyeong.json** | 허○엘 major trauma (영상 다수, ~6.9MB) |
| **scenarios/seizure-parkgeon.json** | 박○건 seizure→ICH (Brain CT/DSA 사진) |
| **scenarios/sepsis-kimyejin.json** | 김예진 패혈성 쇼크 |
| **scenarios/_all.json** | 위 4개 통합본 (로컬 file:// 미리보기용 "가져오기" 대상) |
| **SimEMR_v12_STANDALONE.html** | ⭐ **JSON 없이 단독 실행**되는 단일본 (PRESETS에 4개 임베드, ~8MB). gitignore됨(로컬 전용) |
| SimEMR_v12_BACKUP*.html | 백업 (gitignore됨) |
| **README.md** | 시나리오 제작자 가이드 (7-3 반영 갱신됨) |
| index.html | 루트 진입점 (→ SimEMR_v12.html) |
| .nojekyll / .gitignore | Pages 설정 / 교안 원본·백업 제외 |

> ⚠️ **시나리오 데이터는 이제 `scenarios/*.json`이 정본.** 값 수정은 그 JSON을 직접 편집(bash node). HTML의 PRESETS는 비어있음(`[]`).
> ⚠️ **JSON 편집 후 반드시 `_all.json` 재생성** + (standalone 필요 시) STANDALONE 재빌드.
> ⚠️ 저장소 폴더가 동기화 마운트라 **샌드박스에서 git 쓰기 불가** → git은 맥 터미널에서 직접.
> ⚠️ 큰 변경 전 백업: `cp SimEMR_v12.html SimEMR_v12_BACKUP.html` (BACKUP은 gitignore됨).

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

## 7-2. 2026-07-04 — 중대 버그 수정 + trauma 시나리오 갱신

### ⭐ 중대 버그: SIMEMR_VER 선언 소실 (7/1 작업의 부작용)
- 7/1 PRESETS 교체 때 배열 바로 뒤의 `const SIMEMR_VER='12';` 선언이 함께 삭제됨.
- 결과: load()의 `if(ver!==SIMEMR_VER)`에서 ReferenceError → App.init() 중단 → 프리셋 미표시·버튼(나가기 등) 미작동·옛 IndexedDB 데이터만 표시.
- `const PRESET_IDS=...` 앞에 선언 복구 완료. **교훈: PRESETS splice 후 배열 뒤 코드 보존 여부 반드시 확인.**

### trauma preset 갱신 (사용자 제공 신규 JSON 기반)
- K 3.5→2.5 (VBGA 2.6과 일관), 검사 delay 단축(CBC 2분/Chem·Coag 4분/VBGA 2분/DXR 3분/CT 6분).
- pptx('수영 major trauma 시뮬레이션' 폴더)에서 이미지 13장 삽입: CXR/AXR/C-spine/Pelvis 각 1, e-FAST 6(신규), chest CT 2, APCT 1.
- 구 이름 중복 항목(CT.Chest, CT.Abd&Pelvis) 삭제 → PED 카탈로그명으로 통일. 파일 6.0→7.1MB.

### load() 구조 개선
1. 깨진/불완전 시나리오 방어(최소 구조 보장) — 하나 깨져도 로드 계속.
2. 구버전 마이그레이션·매번실행 정규화 블록 try/catch.
3. **프리셋은 항상 파일 PRESETS로 덮어쓰기** (저장소의 옛 사본 폐기, 사용자 제작 시나리오는 유지). 프리셋 수정본 보관은 JSON 내보내기 사용.

### 검증/디버깅 노하우
- 브라우저가 옛 내용을 보여줄 때: 파일명 바꾼 사본으로 캐시 완전 우회 가능. macOS 폴더 동기화 지연 있음 → 수정 직후엔 ⌘⇧R.
- 런타임 에러는 화면에 안 보임 → 임시 진단 배너(window error/unhandledrejection + App.init().then/catch) 패턴 유효.
- 사용자 IndexedDB에 옛 "패혈증" 초안(uuid id) 잔존 — 필요 없으면 앱에서 삭제.

## 7-3. 2026-07-06 — ⭐ 아키텍처 대전환 + 웹 발행 + 전체 시나리오 확정

### A. 구조 전환: 단일 HTML → 껍데기 + 외부 JSON (7.8MB→0.15MB)
- HTML의 `const PRESETS`를 **빈 배열 `[]`**로. 시나리오는 `scenarios/*.json`으로 분리 + `manifest.json`.
- **load() 재작성**: `fetch('scenarios/manifest.json')`→각 json 로딩. 성공 시 `S._loadMode='auto'`(폴더가 정본, 매 로드 덮어씀), 실패(file://·오프라인) `'local'`.
  - `OFFICIAL_IDS`(fetch된 시나리오 id 집합)로 save()의 이미지-strip 판단 (기존 PRESET_IDS 대체).
  - **폴백**: `_loadMode==='local'` && `PRESETS.length` 있으면 내장 PRESETS로 로딩 → `'embedded'`. → **STANDALONE 파일이 JSON 없이 작동하는 원리**.
- 로컬 file:// = 자동로딩 불가 → `_localHintBanner()`로 "_all.json 가져오기" 안내(embedded 모드에선 배너 안 뜸).
- **배포/공유 = GitHub Pages URL 하나** (`https://zerbaxa.github.io/SIM_AMIS/`). 접속 전원 자동 로딩.
- git push 시 대용량(≈18MB) → HTTP 400 발생 → **`git config http.postBuffer 524288000`**로 해결(설정 완료됨).
- GitHub Pages 활성화 완료 (Settings→Pages, Source: master /root).

### B. 🚀 웹 발행 기능 (교육자 모드, GitHub Contents API)
- 시나리오 카드마다 **🚀 발행**, 툴바 **🚀 전체 발행** + **🔑 토큰 초기화**.
- `publishScenario/publishAll`: 편집본을 `scenarios/*.json`에 직접 커밋(신규는 manifest 자동 추가). `GH_REPO={owner:'Zerbaxa',repo:'SIM_AMIS',branch:'master',dir:'scenarios/'}`.
- 토큰(fine-grained PAT, Contents:R/W)은 발행 시 입력 → localStorage/sessionStorage(코드에 저장 안 함).
- **주의**: 앱 편집+저장은 브라우저(IndexedDB)에만 남고 새로고침 시 폴더 원본으로 복귀. 모두 반영하려면 **발행** 또는 JSON 직접 수정 후 push.

### C. STANDALONE 단일본
- `SimEMR_v12_STANDALONE.html` = 껍데기 HTML + PRESETS에 4개 임베드. **JSON 폴더 없이 더블클릭 실행**. gitignore(로컬 전용).
- 재빌드: thin HTML 복사 → `const PRESETS=[]`를 4개 시나리오 JSON으로 치환 (build_standalone 방식).

### D. 저장 버그 수정 (중요)
- saveSc의 BGA/labOrder 저장 루프가 **null 회차**(예: sepsis VBGA 1회차)에서 `rs.items` 접근 → TypeError → 저장 전체 중단(이름 변경도 안 됨). **null 가드 추가**(빈 회차는 값 입력 시만 채우고 아니면 null 유지). renderBGAed·FiO2 루프도 null 가드 있음.

### E. 검사 대기시간 전면 표준화 (10분 진행 기준)
VBGA/ABGA 1분 · CBC 2분 · Chem/Coag/Inflammatory 4분 · simple XR 2분 · FAST 3분 · CT 8분 · CTA 10분. PRESETS + `getUnsetDelay`(미설정 자동) + IMAGING_TEMPLATES 모두 통일.

### F. 신규 영상 카탈로그 항목 (IMAGING_TEMPLATES + IMG_REGION_MAP)
- `CT.Brain angiography (CTA)` (머리) — seizure용.
- `Single focused US (1point)` (흉부+복부, US, 3분) — sepsis POCUS용. 정상 템플릿 있음.

### G. 전체 시나리오 값 확정 (원칙: **사진 있으면 판독문 X, 사진 없으면 판독문 O**)
- **호흡기(한빛)**: ABGA 3회차 빈칸 채움(BE 계산, Na/K/Glu/Hct는 CBC·chem 참고 ±10%, Lactate 저산소 악화로 1.9→4.7). **CXR 실제 사진 1장 삽입**(resultType image, 판독문 미표시).
- **trauma(허○엘)**: CBC 빈칸(MCV/MCH/MCHC/RDW+5분획)을 사용자 제공 CBC값으로. 사진 있는 검사(CT·FAST·XR)는 판독문 X. **US.Upper abdomen(사진X)** = hepatorenal recess free fluid + CT 권고 판독문. **CT.Whole body trauma dynamic(사진X)** = Chest(hemothorax+multiple lung contusion)+Abd(liver laceration+active bleeding+hemoperitoneum) 통합 판독문.
- **seizure(박○건)**: BGA **전부 VBGA 3회차**로 통일(ABGA 제거). 빈칸 채움. CBC 완전 채움(WBC18.5/Hb12.3/Plt320 + 나머지 적당히). Blood chemistry에 electrolyte 반영. **Brain CT ← 축상+관상 2장 / CTA ← DSA 1장**으로 이미지 재분류(둘 다 사진만, 판독문 X).
- **sepsis(김예진)**: VBGA **1회차=null(채혈량 부족)/2회차=실측값**(첫 처방 실패→재검 연출). VBGA2 빈칸 채움. CBC/Inflammatory=채혈실패(failMessage). Culture.Blood=배양중. Chest PA=정상소견. **Single focused US(1point)=Collapsed IVC 소견만**(해석 힌트 없음).

### H. 저장소 정리
- `.gitignore`에 `*BACKUP*.html`, `*STANDALONE*.html`, 교안 원본 폴더(`Seizure/`,`SepticShock/`,`수영 major trauma 시뮬레이션/`) 추가 → 공개 저장소엔 배포 파일만.
- `00_시뮬레이션 도입.pptx`는 저장소에 포함(공개).

---

## 8. 다음 작업 후보
- [ ] **투약 기능** 구현 (사용자가 계획 알려줄 예정)
- [ ] (옵션) 영상 회차(stage) 기능 — 재처방 시 다른 판독(예: POCUS collapsed→plethora). 현재 영상은 재처방 차단(2번째 "검사 불가").
- [ ] (옵션) 발행 기능의 교육자 편집기 UI(회차/이미지) 보강, `_all.json` 자동 갱신 발행에 포함
- [ ] 실제 브라우저 전체 흐름 육안 확인
- [보류] 소아 정상 참고치(lo/hi 나이대별)

### ⚙️ 시나리오 값 수정 워크플로우 (중요)
1. `scenarios/<파일>.json` 직접 편집 (bash node) → 2. `_all.json` 재생성 → 3. (필요 시) STANDALONE 재빌드 → 4. 맥 터미널 `git add -A && git commit && git push` → 5. 1~2분 후 Pages 반영(⌘⇧R).

---

## 9. 작업 방식 메모
- 변경 후 항상 JS 문법 검사 (script 추출 → `new Function(src)`), 가능하면 순수함수 node 검증
- 6MB 단일 파일 → targeted 수정. base64 라인 Read 금지
- 복잡 작업은 dev-protocol: 전체 그림 먼저 합의 후 구현
- 브라우저 file:// 은 Chrome 확장이 차단 → 자동 UI 테스트 불가, 사용자 육안 확인 안내
