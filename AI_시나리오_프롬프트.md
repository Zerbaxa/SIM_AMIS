# AI 전용 시나리오 작성 프롬프트 (Claude · Gemini · ChatGPT 공용)

**사용법**
1. 아래 `===== 복사 시작 =====` 부터 `===== 복사 끝 =====` 사이를 **전부 복사**해 AI 대화창에 붙여넣습니다.
2. 맨 아래 `[증례 자료]` 부분에 **증례 줄글 / 활력징후 / 검사 결과지(사진·표)** 를 채우거나 파일로 첨부합니다.
3. AI가 출력한 JSON을 `.json` 파일로 저장 → SIM AMIS **설정 화면 → 📥 가져오기** → **미리보기**로 검증합니다.

> 팁: 검사 결과지는 캡처 이미지를 그대로 첨부하면 멀티모달 AI가 수치를 읽어 옮깁니다.
> 한 번에 여러 시나리오를 만들려면 "각 증례마다 JSON 객체를 만들어 배열로 출력하라"고 덧붙이세요.

---

===== 복사 시작 =====

너는 소아·응급 시뮬레이션 EMR(SIM AMIS)의 시나리오 데이터를 만드는 보조자다.
아래 [규칙], [JSON 스키마], [항목명 목록]을 정확히 지켜서, 맨 끝 [증례 자료]에 해당하는
**하나의 JSON 객체만** 출력하라. 설명 문장이나 코드펜스(```) 없이 **순수 JSON만** 출력한다.

[규칙]
1. 항목명(n)·패널명·영상명은 [항목명 목록]의 표기와 **글자 하나까지 동일**해야 한다. 다르면 앱이 무시한다.
2. 결과값(v)과 age는 **문자열**, ageMonths·delayMinutes·GCS(E/V/M)는 **숫자**로 쓴다.
3. **정상이거나 증례와 무관한 검사·항목은 넣지 않는다.** 설정하지 않은 검사는 앱이 자동으로 정상값을 만들어 학생에게 보여준다. → 이상 소견이 있는 항목만 넣는다.
4. 나이가 12개월 미만이면 age는 "0", ageMonths에 개월 수를 넣는다 (예: 생후 3개월 → age:"0", ageMonths:3).
5. 영상은 그림을 만들 수 없으므로 resultType는 "report"로 하고 **판독문 텍스트(report)**를 쓰며, images는 빈 배열([])로 둔다. 이상 소견이면 abnormal:true, 정상이면 abnormal:false.
6. 검사 결과지 사진/표가 있으면 수치를 그대로 옮기되, 항목명·단위는 [항목명 목록]과 이 앱 기준으로 매핑·변환한다.
7. 확실하지 않은 값은 비워 두거나 생략한다. **추측해서 채우지 않는다.**
8. ABGA/VBGA가 없으면 해당 값은 null. labOrders/imaging이 없으면 빈 배열([]). id는 넣지 않는다.

[JSON 스키마]
{
  "name": "시나리오 제목",
  "description": "한 줄 소개",
  "patient": {
    "id": "등록번호(임의)",
    "name": "환자명",
    "age": "나이(세, 문자열)",
    "ageMonths": 0,
    "sex": "M 또는 F",
    "chiefComplaint": "주소증",
    "pastHistory": "과거력 (줄바꿈은 \\n)",
    "history": "현병력",
    "vitals": { "SBP":"", "DBP":"", "HR":"", "RR":"", "SpO2":"", "Temp":"" },
    "mentalStatus": { "type":"GCS", "E":4, "V":5, "M":6, "avpu":"A" }
  },
  "labOrders": [
    {
      "name": "패널명(항목명 목록 참고)",
      "code": "",
      "delayMinutes": 7,
      "resultSets": [
        { "label":"1회차", "items":[ {"n":"항목명","v":"값"} ] }
      ]
    }
  ],
  "abga": null,
  "vbga": {
    "delayMinutes": 3,
    "resultSets": [
      { "label":"Stage1", "items":[ {"n":"pH","v":""} ] }
    ]
  },
  "imaging": [
    {
      "name": "영상명(항목명 목록 참고)",
      "delayMinutes": 5,
      "displayMode": "pages",
      "images": [],
      "resultType": "report",
      "report": "판독문\\n\\nFindings:\\n...\\n\\n[ 결  론 ]\\n...",
      "abnormal": true
    }
  ]
}
- mentalStatus가 AVPU면: { "type":"AVPU", "avpu":"A" }  (A/V/P/U 중 하나)
- displayMode는 "pages" 또는 "stack" (CT/MRI는 보통 "stack")

[항목명 목록]  (n, 패널명, 영상명은 아래 표기와 정확히 일치)

· 혈액검사 패널 → 항목(n)
CBC: WBC, RBC, Hb, Hct, MCV, MCH, MCHC, RDW, PLT, MPV, PDW, E-Neutrophil, E-Lymphocyte, E-Monocyte, E-Eosinophil, E-Basophil
Blood chemistry: Total calcium, Phosphorus, Glucose, Creatinine, BUN, Uric acid, Total cholesterol, Total protein, Albumin, AST(SGOT), ALT(SGPT), ALP, GGT, Total bilirubin, Direct bilirubin, Na, K, Cl, Total CO2, CK, LD, Magnesium, B-hydroxybutyrate, Ammonia, Amylase, Lipase
Inflammatory markers: CRP, ESR, Procalcitonin
Cardiac markers: Troponin I, CK-MB, Myoglobin, BNP
Coagulation profile: PT(sec), PT(%), PT(INR), aPTT, Fibrinogen, D-dimer
Routine urinalysis: SG, pH, Albumin, Glucose, Ketone, Bilirubin, Occult blood, Urobilinogen, Nitrite, WBC(UA)
Thyroid Function Test: TSH, Free T4, Free T3
Culture.Blood, Culture.Urine: 결과값 없음 → "resultSets": []

· ABGA 항목: pH, pCO2, pO2, HCO3, SaO2, BE, Na, K, iCa, Glucose, Hct, Lactate
· VBGA 항목: pH, pCO2, pO2, HCO3, SvO2, BE, Na, K, iCa, Glucose, Hct, Lactate   (FiO2 사용 안 함)

· 영상 검사명(부위별)
머리: XR.Skull / CT.Brain (PED)(without enhance) / MRI.Brain (without enhance)
얼굴: CT.Facial bone 3D (without enhance)
경추: XR.C-spine / CT.C-spine (PED)(without enhance)
흉부: DXR.Chest (PA/AP) / CT.Chest (PED)(with enhance)
복부: XR.Abdomen / US.Upper abdomen / CT.Abdomen & Pelvis (PED)(with enhance) / Multiple focused US (in trauma)
골반: XR.Pelvis / CT.Abdomen & Pelvis (PED)(with enhance)
요추: XR.L-spine
전신: CT.Whole body trauma dynamic (with enhance)

[출력 예시]  (이상 소견만 입력 — 정상 항목은 생략됨)
{
  "name": "소아 세균성 폐렴",
  "description": "3세 남아, 발열·기침·빈호흡으로 내원",
  "patient": {
    "id": "20250101", "name": "홍길동", "age": "3", "ageMonths": 0, "sex": "M",
    "chiefComplaint": "발열, 기침", "pastHistory": "특이사항 없음",
    "history": "3일 전부터 고열·기침, 오늘 호흡이 빨라져 내원",
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

[증례 자료]
(여기에 증례 줄글 / 활력징후 / 검사 결과지 사진·표를 붙여넣거나 첨부하세요)

===== 복사 끝 =====
