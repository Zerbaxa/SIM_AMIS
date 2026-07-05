# SIM AMIS — 시나리오 제작 가이드

소아청소년과 · 응급의학 전공의 시뮬레이션 교육용 EMR입니다.
이 문서는 **시나리오를 만드는 분(교육자)** 을 위한 안내서입니다. 개발 지식은 필요 없습니다.

---

## 1. 시작하기

1. 배포받은 주소(URL)로 접속하면 시작 화면이 나옵니다.
2. 두 가지 모드가 있습니다.

| 모드 | 용도 |
|---|---|
| 🩺 **시나리오 실행** | 학생(전공의)이 검사·처방을 내고 결과를 해석하는 화면 |
| ⚙️ **시나리오 설정** | 교육자가 시나리오를 만들고 관리하는 화면 |

> **시나리오는 어디 있나?** 공식 시나리오의 **정본(원본)은 저장소의 `scenarios/` 폴더 JSON**입니다.
> 배포 주소(GitHub Pages)로 접속하면 이 파일들이 **자동으로 로딩**됩니다.
> 앱에서 편집·저장한 내용은 **내 브라우저(IndexedDB)에만** 남고, 새로고침하면 원본으로 되돌아갑니다.
> 편집을 **모두에게 반영하려면 "발행"** 해야 합니다 (5장 참고).

---

## 2. 시나리오 만들기 (단계별)

**시나리오 설정 → + 새 시나리오** 를 누른 뒤 위에서부터 채웁니다.

### 2-1. 기본 정보
- **제목 / 설명**: 학생 목록에 보이는 시나리오 이름과 한 줄 소개
- **환자 ID · 이름**
- **나이**: **세**와 **개월**을 따로 입력합니다.
  - 예) 생후 3개월 → 나이 `0`, 개월 `3` → 화면에 "3개월"으로 표시되고, **CBC 등 정상값이 영아 기준으로 자동 계산**됩니다.
- **성별 / 주소(Chief complaint) / 과거력 / 현병력**
- **활력징후**: SBP, DBP, HR, RR, SpO₂, 체온
- **의식 수준**: GCS(E·V·M) 또는 AVPU

### 2-2. 혈액검사 처방
- **처방명**을 검색해 추가합니다 (예: CBC, Blood chemistry, Inflammatory markers).
- 각 항목에 **결과값**을 입력합니다.
- **회차 추가**: 같은 검사를 시간차로 다시 시행했을 때의 추적 결과(F/U)를 넣을 수 있습니다.
- **검체불량 메시지**: 결과 대신 "검체 불량" 안내를 띄울 수 있습니다.

> 💡 **핵심:** 값을 비워 두거나 검사를 아예 추가하지 않으면, 학생이 그 검사를 처방했을 때
> **시스템이 자동으로 "정상값"을 만들어 보여줍니다.** 학생은 설정 여부를 알 수 없습니다.
> → **이상 소견이 있는 항목만** 채우고, 정상인 항목은 비워 두면 됩니다.

### 2-3. ABGA / VBGA
- 동맥혈/정맥혈 가스분석. 회차별(Stage별)로 값을 넣어 환자 상태 변화를 표현할 수 있습니다.
- 하나만 설정해도 학생 화면에는 둘 다 처방 가능하게 보이며, 설정 안 한 쪽은 "검체 오류"로 처리됩니다.

> 💉 **"처음엔 채혈 실패, 재시도하면 성공" 연출**: **1회차를 비워 두고(회차 값 미입력) 2회차에 실제 값**을 넣으면,
> 학생이 처음 처방했을 때는 **"채혈량 부족"**(검체불량)이 뜨고, 다시 처방하면 값이 나옵니다.
> (예: 패혈성 쇼크에서 IV/라인 확보 전엔 채혈 실패 → 확보 후 재검) 사유 문구는 "검체불량 메시지" 칸에서 바꿀 수 있습니다.

### 2-4. 영상 검사 ⭐ (부위 → 검사 순서)
1. **부위 선택** (머리/흉부/복부 등) → **검사 종류 선택** → **+ 추가**
2. 각 검사마다 설정:
   - **결과 유형**: `이미지` / `판독결과(텍스트)` / `둘 다`
   - **소견 판정**: `정상` / `이상소견`
   - 이미지 업로드 또는 판독문 입력 (판독문을 비우면 정상 판독 템플릿이 자동 사용)
3. **부위 일관성 확인**: 한 검사를 **이상소견**으로 설정하면, **같은 부위의 다른 검사**(예: Chest XR ↔ Chest CT)가 정상으로 남아 있을 때 경고 팝업이 뜹니다.
   - 환자 상태가 모순되지 않도록 관련 검사에도 이상소견을 작성하세요. (미설정 검사는 "일괄 추가" 버튼 제공)

### 2-5. 미리보기 👁
- 편집 화면 상단 **미리보기** 버튼 → 학생이 보는 화면 그대로 임시 실행됩니다 (저장 안 됨).
- **⏩ 결과 즉시 도착** 버튼으로 검사 대기시간을 건너뛰며 흐름을 확인할 수 있습니다.
- **← 편집으로 돌아가기** 로 복귀합니다.

### 2-6. 저장
- **💾 저장** 또는 **← 목록(자동저장)** 으로 저장합니다.

---

## 3. 알아두면 좋은 동작 규칙

| 규칙 | 설명 |
|---|---|
| **미설정 = 자동 정상** | 설정하지 않은 검사는 학생이 처방하면 정상값으로 자동 응답 (설정 여부 숨김) |
| **결과 대기시간** | `보고 대기(분)` 만큼 실제 시간이 지나야 결과가 도착. 아래 **표준값**으로 통일되어 있음 |
| **회차(F/U)** | 같은 검사를 반복 처방하면 다음 회차 결과가 나옴. 설정한 회차를 초과하거나 **비운 회차**는 **검체불량**(채혈량 부족) 처리 |
| **상한전용 마커** | Troponin I·BNP·D-dimer 등은 정상으로 둘 경우 실제 건강인 수치를 반환 |
| **배양(Culture)** | 처방하면 결과값과 무관하게 **"배양 진행 중(2일 후 보고)"** 메시지만 표시 |

**검사별 표준 보고 대기시간** (모든 시나리오 공통 — 10분 진행 기준)

| 검사 | 대기 | 검사 | 대기 |
|---|---|---|---|
| VBGA / ABGA | 1분 | Simple X-ray (Chest/Abd/…) | 2분 |
| CBC | 2분 | FAST(focused US) | 3분 |
| Blood chemistry / Coagulation / Inflammatory | 4분 | CT (전체) | 8분 |
| Culture | (시나리오 중 결과 안 나옴) | CTA (혈관조영) | 10분 |

> 편집 화면에서 개별 검사의 `보고 대기(분)`를 바꿔 예외를 둘 수도 있습니다.

---

## 4. 주요 기능 요약

- 처방 검색·선택(스테이징) 후 일괄 처방
- 결과 탭 / **추세 비교 탭**(2회 이상 시 회차별 비교 표)
- 영상 PACS 뷰어(스크롤/페이지), 판독문 표시
- 시나리오 **JSON 내보내기 / 가져오기**
- **🚀 GitHub 발행**: 편집한 시나리오를 저장소에 바로 반영 (5장)
- `scenarios/` 폴더 JSON **자동 로딩** (배포 시)
- 자동 저장(IndexedDB) + 저장 실패 시 경고

---

## 5. 저장 · 공유 · 발행 (구조 이해가 중요)

이 앱은 **껍데기 HTML + 외부 시나리오 JSON** 구조입니다. 시나리오의 **정본은 저장소의 `scenarios/` 폴더 JSON**이고, 앱은 접속할 때 이 파일들을 자동으로 불러옵니다.

### 5-1. 시나리오가 뜨는 방식

| 접속 방법 | 동작 |
|---|---|
| **배포 주소(GitHub Pages)** | `scenarios/`의 JSON을 **자동 로딩**. 접속한 모두가 같은 시나리오를 봄 → **공유는 주소만 알려주면 끝** |
| **로컬 파일 더블클릭(file://)** | 보안상 자동 로딩 불가. 하단 안내대로 **📥 가져오기 → `scenarios/_all.json`**(4개 통합본) 한 번 불러오면 됨 (혼자 미리보기용) |

### 5-2. 편집 내용을 모두에게 반영 = "발행"

앱에서 편집·저장해도 그 내용은 **내 브라우저에만** 남습니다(새로고침하면 원본으로 복귀). 진짜로 반영하려면 **발행**합니다.

**방법 A — 🚀 발행 버튼 (권장, 터미널 불필요)**
1. 시나리오 설정에서 편집 → 저장
2. 시나리오 카드의 **🚀 발행** (또는 툴바 **🚀 전체 발행**) 클릭
3. 처음이면 **GitHub 토큰** 입력 (5-3)
4. 1~2분 후 사이트에 자동 반영 (안 보이면 ⌘⇧R)

**방법 B — 파일 교체 후 git (개발자용)**
`📤 내보내기` → `scenarios/`의 해당 파일 교체 (새 시나리오면 `manifest.json`에 파일명 추가) → `git add/commit/push`

### 5-3. 발행용 GitHub 토큰 만들기 (한 번만)

GitHub → 프로필 **Settings** → **Developer settings** → **Fine-grained tokens** → **Generate new token**
- **Repository access**: `Only select repositories` → 이 저장소만
- **Permissions** → **Contents: Read and write**
- 생성된 토큰(`github_pat_…`)을 발행 시 붙여넣기

> 🔒 토큰은 코드에 저장되지 않고 **본인 브라우저에만** 보관됩니다. 만료기간과 이 저장소만으로 범위를 제한하고, 공용 PC에서는 발행 후 **🔑 토큰 초기화**를 누르세요.

### 5-4. 백업 / 오프라인 공유

- **📤 내보내기 / 전체 내보내기**로 `.json` 파일 생성. 이 파일에는 **영상 이미지까지 포함**됩니다.
- 이메일·USB로 JSON만 주고받아도 **📥 가져오기**로 그대로 재현됩니다.

> ⚠️ 앱에서만 편집하고 발행/내보내기를 안 하면, 브라우저 데이터를 지울 때 사라집니다. **정본은 저장소에 발행**하거나 **JSON으로 백업**하세요.

---

## 6. AI(Claude · Gemini)로 시나리오 자동 채우기

증례 줄글이나 검사 결과지를 일일이 입력하기 번거로울 때, 멀티모달 AI에게 **아래 JSON 스키마**와 자료를 주고 자동으로 채우게 한 뒤, 결과 JSON을 앱에 **가져오기** 하면 됩니다.

> ⭐ **가장 쉬운 방법**: 같은 폴더의 **`AI_시나리오_프롬프트.md`** 파일을 여세요. 규칙·스키마·항목명·예시가 한 덩어리로 들어 있어, `===== 복사 시작 ~ 끝 =====` 구간을 그대로 복사해 AI에 붙이고 증례만 채우면 됩니다.
> 아래 6-2~6-7은 그 내용의 상세 설명입니다.

### 6-1. 작업 흐름
1. 증례 자료 준비(줄글 케이스, 검사 결과지 사진/표, 활력징후 등)
2. AI에게 **6-6 프롬프트 + 6-2 스키마 + 6-3 항목명 + 자료** 를 함께 제공
3. AI가 출력한 JSON을 `.json` 파일로 저장
4. 앱 설정 화면에서 **📥 가져오기** → 미리보기로 검증

### 6-2. JSON 스키마 (필드 설명)
```jsonc
{
  "name": "시나리오 제목",
  "description": "한 줄 소개",
  "patient": {
    "id": "등록번호(임의)",
    "name": "환자명",
    "age": "나이(세, 문자열)",          // 영아는 "0"
    "ageMonths": 3,                      // 개월(0~11, 숫자). 해당 없으면 0
    "sex": "M",                          // "M" 또는 "F"
    "chiefComplaint": "주소증",
    "pastHistory": "과거력(줄바꿈은 \\n)",
    "history": "현병력",
    "vitals": { "SBP":"96","DBP":"50","HR":"161","RR":"30","SpO2":"88","Temp":"40.5" },
    "mentalStatus": { "type":"GCS","E":4,"V":4,"M":6,"avpu":"A" }  // AVPU면 type:"AVPU", avpu:"A/V/P/U"
  },
  "labOrders": [
    {
      "name": "CBC",                     // 6-3의 정확한 패널명
      "code": "BB0001",                  // 임의(빈 문자열 가능)
      "delayMinutes": 7,                 // 결과 대기(분)
      "resultSets": [                    // 회차 배열
        { "label":"1회차",
          "items":[ {"n":"WBC","v":"18.5"}, {"n":"Hb","v":"11.6"} ]  // n=항목명, v=값(문자열)
        }
      ]
    }
  ],
  "abga": null,                          // 없으면 null
  "vbga": {                              // ABGA/VBGA 구조 동일
    "delayMinutes": 3,
    "resultSets": [
      { "label":"Stage1", "items":[ {"n":"pH","v":"7.38"}, {"n":"pCO2","v":"38"} ] }
    ]
  },
  "imaging": [
    {
      "name": "DXR.Chest (PA/AP)",       // 6-3의 정확한 영상명
      "delayMinutes": 5,
      "displayMode": "pages",            // "pages" 또는 "stack"
      "images": [],                      // AI는 비워둘 것(실제 영상은 사람이 업로드)
      "resultType": "report",            // AI 작성 시 "report" 권장
      "report": "CHEST PA/AP\\n\\nFindings:\\n...\\n\\n[ 결  론 ]\\n...",
      "abnormal": true                   // 이상소견이면 true
    }
  ]
}
```

> **규칙 요약**
> - 값(`v`)·나이(`age`)는 **문자열**, `ageMonths`/GCS/`delayMinutes`는 숫자.
> - **정상이거나 시나리오와 무관한 검사는 넣지 마세요** → 자동 정상 처리됩니다.
> - 영상은 AI가 그림을 만들 수 없으므로 **`resultType:"report"`** 로 **판독문 텍스트**를 작성하고 `images`는 빈 배열로 둡니다.
> - `id`는 넣지 않아도 됩니다(가져올 때 자동 부여).

### 6-3. 사용 가능한 항목명 (정확히 일치해야 자동 판정·정상값이 작동)

**혈액검사 패널 → 항목(`n`)**
- `CBC`: WBC, RBC, Hb, Hct, MCV, MCH, MCHC, RDW, PLT, MPV, PDW, E-Neutrophil, E-Lymphocyte, E-Monocyte, E-Eosinophil, E-Basophil
- `Blood chemistry`: Total calcium, Phosphorus, Glucose, Creatinine, BUN, Uric acid, Total cholesterol, Total protein, Albumin, AST(SGOT), ALT(SGPT), ALP, GGT, Total bilirubin, Direct bilirubin, Na, K, Cl, Total CO2, CK, LD, Magnesium, B-hydroxybutyrate, Ammonia, Amylase, Lipase
- `Inflammatory markers`: CRP, ESR, Procalcitonin
- `Cardiac markers`: Troponin I, CK-MB, Myoglobin, BNP
- `Coagulation profile`: PT(sec), PT(%), PT(INR), aPTT, Fibrinogen, D-dimer
- `Routine urinalysis`: SG, pH, Albumin, Glucose, Ketone, Bilirubin, Occult blood, Urobilinogen, Nitrite, WBC(UA)
- `Thyroid Function Test`: TSH, Free T4, Free T3
- `Culture.Blood`, `Culture.Urine`: 결과값 없음(접수만). `resultSets:[]`

**ABGA 항목**: pH, pCO2, pO2, HCO3, SaO2, BE, Na, K, iCa, Glucose, Hct, Lactate
**VBGA 항목**: pH, pCO2, pO2, HCO3, SvO2, BE, Na, K, iCa, Glucose, Hct, Lactate *(FiO2는 사용하지 않음)*

**영상 검사명 (부위별)**
- 머리: `XR.Skull`, `CT.Brain (PED)(without enhance)`, `CT.Brain angiography (CTA)`, `MRI.Brain (without enhance)`
- 얼굴: `CT.Facial bone 3D (without enhance)`
- 경추: `XR.C-spine`, `CT.C-spine (PED)(without enhance)`
- 흉부: `DXR.Chest (PA/AP)`, `CT.Chest (PED)(with enhance)`
- 복부: `XR.Abdomen`, `US.Upper abdomen`, `CT.Abdomen & Pelvis (PED)(with enhance)`, `Multiple focused US (in trauma)`
- 골반: `XR.Pelvis`, `CT.Abdomen & Pelvis (PED)(with enhance)`
- 요추: `XR.L-spine`
- 전신: `CT.Whole body trauma dynamic (with enhance)`

### 6-4. 빈 템플릿 (복사해서 사용)
```json
{
  "name": "",
  "description": "",
  "patient": {
    "id": "", "name": "", "age": "", "ageMonths": 0, "sex": "M",
    "chiefComplaint": "", "pastHistory": "", "history": "",
    "vitals": { "SBP":"","DBP":"","HR":"","RR":"","SpO2":"","Temp":"" },
    "mentalStatus": { "type":"GCS","E":4,"V":5,"M":6,"avpu":"A" }
  },
  "labOrders": [],
  "abga": null,
  "vbga": null,
  "imaging": []
}
```

### 6-5. 작성 예시 (발췌)
```json
{
  "name": "소아 세균성 폐렴",
  "description": "3세 남아, 발열·기침·빈호흡으로 내원",
  "patient": {
    "id": "20250101", "name": "홍길동", "age": "3", "ageMonths": 0, "sex": "M",
    "chiefComplaint": "발열, 기침", "pastHistory": "특이사항 없음",
    "history": "3일 전부터 고열과 기침, 오늘 호흡이 빨라져 내원",
    "vitals": { "SBP":"95","DBP":"55","HR":"150","RR":"40","SpO2":"91","Temp":"39.6" },
    "mentalStatus": { "type":"GCS","E":4,"V":5,"M":6,"avpu":"A" }
  },
  "labOrders": [
    { "name":"CBC","code":"","delayMinutes":7,
      "resultSets":[{"label":"1회차","items":[
        {"n":"WBC","v":"19.2"},{"n":"E-Neutrophil","v":"80"},{"n":"Hb","v":"11.8"},{"n":"PLT","v":"360"}
      ]}] },
    { "name":"Inflammatory markers","code":"","delayMinutes":8,
      "resultSets":[{"label":"1회차","items":[{"n":"CRP","v":"9.4"},{"n":"Procalcitonin","v":"1.8"}]}] }
  ],
  "abga": null,
  "vbga": null,
  "imaging": [
    { "name":"DXR.Chest (PA/AP)","delayMinutes":5,"displayMode":"pages","images":[],
      "resultType":"report","abnormal":true,
      "report":"CHEST PA/AP\n\nFindings:\nConsolidation in the right lower lobe with air-bronchogram.\nNo pleural effusion.\n\n[ 결  론 ]\nRLL pneumonia." }
  ]
}
```
> 위 예시는 이상 소견만 입력했습니다. Na·K·Glucose 등 정상 항목은 넣지 않아 **자동 정상값**으로 처리됩니다.

### 6-6. AI에게 줄 프롬프트 (복사해서 사용)
```
너는 소아·응급 시뮬레이션 EMR의 시나리오 데이터를 만드는 助手다.
아래 [증례 자료]를 읽고, 첨부한 [JSON 스키마]와 [항목명 목록] 규칙에 정확히 맞는
하나의 JSON 객체만 출력하라. 설명·코드펜스 없이 순수 JSON만 출력한다.

규칙:
1. 항목명(n)·패널명·영상명은 [항목명 목록]의 표기와 글자 하나까지 동일해야 한다.
2. 결과값(v)과 age는 문자열, ageMonths·delayMinutes·GCS는 숫자로 쓴다.
3. 정상이거나 증례와 무관한 검사·항목은 넣지 않는다(앱이 자동으로 정상 처리한다).
4. 영상은 그림을 만들 수 없으므로 resultType는 "report"로 하고 판독문 텍스트를 쓰며,
   images는 빈 배열([])로 둔다. 이상 소견이면 abnormal:true.
5. 검사 결과지 사진이 있으면 수치를 그대로 옮기되, 단위·항목명은 목록 기준으로 매핑한다.
6. 확실하지 않은 값은 비워 두거나 생략한다(추측해서 채우지 않는다).

[JSON 스키마]
(6-2의 스키마를 붙여넣기)

[항목명 목록]
(6-3의 목록을 붙여넣기)

[증례 자료]
(여기에 증례 줄글 / 활력징후 / 검사 결과지 사진 또는 표를 첨부)
```

### 6-7. 멀티모달 자료를 잘 주는 팁
- **검사 결과지/표는 이미지로 직접 첨부**하면 AI가 수치를 읽어 옮깁니다. 단, 항목명은 6-3 표기로 매핑하라고 지시하세요.
- **단위가 다른 자료**(예: Troponin ng/mL vs pg/mL)는 프롬프트에 "이 앱의 단위 기준에 맞춰 변환하라"고 명시하세요.
- **이상 소견만** 자료에 강조해 주면 불필요한 정상값 입력을 줄일 수 있습니다.
- 출력 JSON을 **앱에 가져오기 → 미리보기**로 반드시 검증하세요. 항목명이 틀리면 그 값은 표시되지 않거나 자동 정상값으로 대체됩니다.

---

### 부록 — 영상 검사 부위 매핑
한 검사가 여러 부위에 속할 수 있습니다(예: `CT.Abdomen & Pelvis`=복부+골반). 이상소견 일관성 경고는 검사가 속한 **모든 부위**를 함께 검토합니다.
