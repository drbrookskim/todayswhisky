# 데일리픽 상세정보/DB확장/무드 이모지 개편 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 홈 탭 데일리 픽 카드에 원산지/증류소/구매링크를 추가하고, `SPIRITS_DB`를 30종에서 60종으로 확장하며, 무드 선택 화면의 이모지를 얼굴 표정이 아닌 상황을 묘사하는 이모지로 교체한다.

**Architecture:** 순수 데이터(`SPIRITS_DB`, `MOODS`) 추가/수정과 `HomeTab`의 JSX 확장. React 상태/스코어링 로직은 변경하지 않는다 — 기존 30종과 동일한 필드 shape을 신규 30종에도 적용해 `CurateTab.run`/`HomeTab`의 `dailyPick` 스코어링이 그대로 작동하게 한다.

**Tech Stack:** React 18 (UMD, CDN) / Babel standalone / Tailwind CDN. 빌드/테스트 도구 없음 — 검증은 `preview_*` 브라우저 도구로 실행.

## Global Constraints

- `SPIRITS_DB`의 모든 항목(기존+신규)은 동일한 필드 shape을 가진다: `name`, `nameEn`, `age`, `style`, `tags`, `abv`, `archetype`, `priceKrw`, `priceRangeKrw`, `priceUsd`, `variants`, `desc`, `descEn`, `country`, `distillery`.
- `tags`는 `MBTI_TRAIT_PREFS`(`index.html:289-294`)에 이미 정의된 태그 어휘만 사용한다: `smooth, balanced, classic, sweet, light, creamy, fresh, clean, complex, smoky, intense, rich, seaweed, medicinal, cask_strength, oak, vanilla, caramel, honey, value, standard, spicy, fruity, floral, unique, sherry, orange, citrus, bold, peppery, maritime, salty, mellow, velvet, elegant, chocolate, corn, nutty, mizunara, smoke`.
- 큐레이션 결과 화면(`CurateTab`)의 구매 버튼은 변경하지 않는다 — 이번 구매 버튼은 홈 탭 데일리 픽 카드 전용.
- MBTI 분석/큐레이션 스코어링(`CurateTab.run`, `HomeTab`의 `dailyPick` `useMemo`) 로직 자체는 수정하지 않는다.
- 이 프로젝트는 자동화 테스트 러너가 없다 — 검증은 컨트롤러가 `preview_*` 브라우저 도구로 직접 수행한다(서브에이전트는 해당 도구에 접근할 수 없음, 이전 작업들에서 확인됨).

---

### Task 1: 기존 30종에 `country`/`distillery` 필드 추가

**Files:**
- Modify: `index.html` (`SPIRITS_DB` 배열, 306-338번째 줄 전체)

**Interfaces:**
- Consumes: 없음
- Produces: 기존 30개 항목 각각에 `country: {ko, en}`, `distillery: {ko, en}` 필드 추가. Task 3(`HomeTab` UI)이 `dailyPick.country`/`dailyPick.distillery`를 참조한다.

- [ ] **Step 1: `SPIRITS_DB` 배열의 기존 30개 항목 전체를 country/distillery 필드가 추가된 버전으로 교체**

`index.html`에서 `const SPIRITS_DB = [` 다음 줄부터(306번째 줄) `{ name: "로얄살루트 21년", ...}` 줄(338번째 줄)까지 30개 항목 전체를 찾는다(현재 파일과 정확히 일치해야 함 — 각 줄의 `desc`/`descEn` 뒤 ` }`로 끝나는 형태). 각 줄의 마지막 `descEn: "..."` 다음, 닫는 `}` 앞에 `, country: { ko: "...", en: "..." }, distillery: { ko: "...", en: "..." }`를 삽입한다. 30개 항목에 적용할 정확한 값:

| name | country.ko / en | distillery.ko / en |
|---|---|---|
| 랭스 블렌디드 위스키 | 스코틀랜드 / Scotland | 이안 맥클라우드 디스틸러스 / Ian Macleod Distillers |
| 길리듀 | 스코틀랜드 / Scotland | 앵거스 던디 디스틸러스 / Angus Dundee Distillers |
| 존바 리저브 | 스코틀랜드 / Scotland | 와이트 앤 맥케이 / Whyte & Mackay |
| 그란츠 트리플우드 | 스코틀랜드 / Scotland | 윌리엄그랜트앤선즈 / William Grant & Sons |
| 골든블루 쿼츠 | 한국 / Korea | ㈜골든블루 / Golden Blue Co., Ltd. |
| 짐빔 화이트 | 미국(켄터키) / USA (Kentucky) | 짐빔 증류소 (빔산토리) / Jim Beam Distillery (Beam Suntory) |
| 에반 윌리엄스 블랙 | 미국(켄터키) / USA (Kentucky) | 헤븐힐 증류소 / Heaven Hill Distillery |
| 제임슨 스탠다드 | 아일랜드 / Ireland | 미들턴 증류소 (아이리시 디스틸러스) / Midleton Distillery (Irish Distillers) |
| 잭다니엘스 넘버 7 | 미국(테네시) / USA (Tennessee) | 잭다니엘 증류소 (브라운포먼) / Jack Daniel Distillery (Brown-Forman) |
| 조니워커 블랙라벨 | 스코틀랜드 / Scotland | 디아지오 / Diageo |
| 산토리 가쿠빈 | 일본 / Japan | 산토리 / Suntory |
| 버팔로 트레이스 | 미국(켄터키) / USA (Kentucky) | 버팔로 트레이스 증류소 / Buffalo Trace Distillery |
| 몽키숄더 | 스코틀랜드 / Scotland | 윌리엄그랜트앤선즈 / William Grant & Sons |
| 와일드터키 101 8년 | 미국(켄터키) / USA (Kentucky) | 와일드터키 증류소 (캄파리) / Wild Turkey Distillery (Campari) |
| 와일드터키 레어브리드 | 미국(켄터키) / USA (Kentucky) | 와일드터키 증류소 / Wild Turkey Distillery |
| 벤리악 12년 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 벤리악 증류소 / The BenRiach Distillery |
| 탈리스커 10년 | 스코틀랜드(스카이섬) / Scotland (Isle of Skye) | 탈리스커 증류소 (디아지오) / Talisker Distillery (Diageo) |
| 조니워커 그린라벨 | 스코틀랜드 / Scotland | 디아지오 / Diageo |
| 네이키드 몰트 | 스코틀랜드 / Scotland | 윌리엄그랜트앤선즈 / William Grant & Sons |
| 더 글렌리벳 12년 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 글렌리벳 증류소 (시바스브라더스) / The Glenlivet Distillery (Chivas Brothers) |
| 글렌드로낙 12년 | 스코틀랜드(하일랜드) / Scotland (Highland) | 글렌드로낙 증류소 / The GlenDronach Distillery |
| 맥캘란 12년 더블 캐스크 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 맥캘란 증류소 (에드링턴) / The Macallan Distillery (Edrington) |
| 로얄 브라클라 12년 | 스코틀랜드(하일랜드) / Scotland (Highland) | 로얄브라클라 증류소 (바카디) / Royal Brackla Distillery (Bacardi) |
| 라가불린 16년 | 스코틀랜드(아일라) / Scotland (Islay) | 라가불린 증류소 (디아지오) / Lagavulin Distillery (Diageo) |
| 글렌피딕 15년 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 글렌피딕 증류소 (윌리엄그랜트앤선즈) / Glenfiddich Distillery (William Grant & Sons) |
| 발베니 12년 더블우드 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 발베니 증류소 (윌리엄그랜트앤선즈) / The Balvenie Distillery (William Grant & Sons) |
| 아드벡 코리브레칸 | 스코틀랜드(아일라) / Scotland (Islay) | 아드벡 증류소 (글렌모렌지컴퍼니/LVMH) / Ardbeg Distillery (Glenmorangie Co./LVMH) |
| 조니워커 블루라벨 | 스코틀랜드 / Scotland | 디아지오 / Diageo |
| 야마자키 12년 | 일본 / Japan | 야마자키 증류소 (산토리) / Yamazaki Distillery (Suntory) |
| 로얄살루트 21년 | 스코틀랜드 / Scotland | 시바스브라더스 (페르노리카) / Chivas Brothers (Pernod Ricard) |

구체적으로, 예를 들어 첫 항목(랭스)은:

before (줄 끝부분):
```
..., desc: "[보틀벙커 TOP 2] 극강의 가성비. 하이볼로 탔을 때 가볍고 부드러운 단맛이 일품인 데일리 위스키.", descEn: "Extreme value. A daily whisky with a light, soft sweetness that shines in highballs." },
```
after:
```
..., desc: "[보틀벙커 TOP 2] 극강의 가성비. 하이볼로 탔을 때 가볍고 부드러운 단맛이 일품인 데일리 위스키.", descEn: "Extreme value. A daily whisky with a light, soft sweetness that shines in highballs.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "이안 맥클라우드 디스틸러스", en: "Ian Macleod Distillers" } },
```

이 패턴을 위 표의 30개 항목 전체에 동일하게 적용한다(각 줄 맨 끝 `descEn: "..."` 뒤, 닫는 `}` 앞에 `, country: {...}, distillery: {...}` 삽입 — 줄의 나머지 부분은 전혀 건드리지 않는다).

- [ ] **Step 2: 구조적 확인**

`grep -c "country: {" index.html`로 정확히 30개(이 태스크 완료 시점 기준)가 나오는지 확인한다. `grep -c "distillery: {" index.html`도 동일하게 30이어야 한다.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: 기존 위스키 30종에 원산지/증류소 데이터 추가"
```

---

### Task 2: 신규 위스키 30종 추가 (`SPIRITS_DB` 30 → 60)

**Files:**
- Modify: `index.html` (`SPIRITS_DB` 배열, Task 1 완료 후의 마지막 항목 `로얄살루트 21년` 다음)

**Interfaces:**
- Consumes: Task 1에서 확립된 `country`/`distillery` 필드 shape
- Produces: `SPIRITS_DB.length === 60`. 신규 30종도 `tags`가 `MBTI_TRAIT_PREFS` 어휘를 사용하므로 `CurateTab.run`/`HomeTab`의 `dailyPick` 스코어링에 자동으로 포함된다.

- [ ] **Step 1: 로얄살루트 21년 항목 뒤, 배열을 닫는 `];` 앞에 30개 신규 항목 추가**

다음 줄을 찾는다(Task 1 완료 후 country/distillery가 이미 붙어 있음):

```
            { name: "로얄살루트 21년", nameEn: "Royal Salute 21 Year", age: "21년", style: "Royal Blend", tags: ["smooth", "sweet", "floral", "classic"], abv: 40, archetype: ["ESTJ", "ESFJ"], priceKrw: 200000, priceRangeKrw: "200,000원 ~ 260,000원", priceUsd: 160, variants: [{vol: "700ml", price: 210000}], desc: "[주류상회Be TOP 9] 여왕을 위한 술. 도자기 병에 담긴 부드러움의 극치.", descEn: "Fit for a Queen. Ultimate smoothness in a porcelain flagon.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } }
        ];
```

다음으로 바꾼다(로얄살루트 21년 뒤에 콤마를 추가하고, 30개 신규 항목을 이어서 삽입한 뒤 배열을 닫는다):

```
            { name: "로얄살루트 21년", nameEn: "Royal Salute 21 Year", age: "21년", style: "Royal Blend", tags: ["smooth", "sweet", "floral", "classic"], abv: 40, archetype: ["ESTJ", "ESFJ"], priceKrw: 200000, priceRangeKrw: "200,000원 ~ 260,000원", priceUsd: 160, variants: [{vol: "700ml", price: 210000}], desc: "[주류상회Be TOP 9] 여왕을 위한 술. 도자기 병에 담긴 부드러움의 극치.", descEn: "Fit for a Queen. Ultimate smoothness in a porcelain flagon.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } },

            { name: "발렌타인 파이니스트", nameEn: "Ballantine's Finest", age: "NAS", style: "Blended Scotch", tags: ["honey", "smooth", "light", "floral"], abv: 40, archetype: ["ESFJ", "ESFP"], priceKrw: 28000, priceRangeKrw: "20,000원 ~ 30,000원", priceUsd: 20, variants: [{vol: "700ml", price: 28000}], desc: "부드럽고 가벼운 스코틀랜드 정통 블렌디드. 꿀과 꽃향이 은은하게 어우러진 데일리 위스키.", descEn: "A light, smooth classic Scotch blend with gentle honey and floral notes for everyday drinking.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } },
            { name: "발렌타인 17년", nameEn: "Ballantine's 17 Year", age: "17년", style: "Blended Scotch", tags: ["rich", "smooth", "oak", "honey"], abv: 40, archetype: ["ISFJ", "ENFJ"], priceKrw: 115000, priceRangeKrw: "100,000원 ~ 130,000원", priceUsd: 85, variants: [{vol: "700ml", price: 115000}], desc: "오랜 숙성이 만든 균형감 있는 부드러움. 오크와 꿀 향이 깊게 어우러진 프리미엄 블렌디드.", descEn: "Long maturation brings balanced smoothness with deep oak and honey notes.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } },
            { name: "시바스리갈 12년", nameEn: "Chivas Regal 12 Year", age: "12년", style: "Blended Scotch", tags: ["honey", "fruity", "smooth", "vanilla"], abv: 40, archetype: ["ESFJ", "ENFP"], priceKrw: 55000, priceRangeKrw: "48,000원 ~ 60,000원", priceUsd: 42, variants: [{vol: "700ml", price: 55000}], desc: "사과와 꿀 향이 조화를 이루는 스코틀랜드의 대표 블렌디드. 부드러운 목넘김이 특징.", descEn: "Scotland's iconic blend with apple and honey harmony and a smooth finish.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } },
            { name: "시바스리갈 18년", nameEn: "Chivas Regal 18 Year", age: "18년", style: "Blended Scotch", tags: ["rich", "sherry", "smoky", "complex"], abv: 40, archetype: ["INFJ", "INTJ"], priceKrw: 150000, priceRangeKrw: "140,000원 ~ 170,000원", priceUsd: 110, variants: [{vol: "700ml", price: 150000}], desc: "건포도와 스모키함이 어우러진 깊고 복합적인 풍미의 18년 숙성 블렌디드.", descEn: "A deep, complex 18-year blend with dried fruit and smoky notes.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "시바스브라더스 (페르노리카)", en: "Chivas Brothers (Pernod Ricard)" } },
            { name: "듀어스 화이트라벨", nameEn: "Dewar's White Label", age: "NAS", style: "Blended Scotch", tags: ["honey", "vanilla", "smooth", "light"], abv: 40, archetype: ["ESFP", "ISFJ"], priceKrw: 32000, priceRangeKrw: "28,000원 ~ 38,000원", priceUsd: 24, variants: [{vol: "700ml", price: 32000}], desc: "꿀과 바닐라의 부드러운 단맛. 하이볼로도 니트로도 무난한 스코틀랜드 정통 블렌디드.", descEn: "Soft honey and vanilla sweetness. A versatile classic Scotch blend.", country: { ko: "스코틀랜드", en: "Scotland" }, distillery: { ko: "존 듀어 앤 선즈 (바카디)", en: "John Dewar & Sons (Bacardi)" } },
            { name: "글렌모렌지 오리지널 10년", nameEn: "Glenmorangie Original 10 Year", age: "10년", style: "Highland Single Malt", tags: ["floral", "citrus", "fresh", "fruity"], abv: 40, archetype: ["INFP", "ENFP"], priceKrw: 75000, priceRangeKrw: "65,000원 ~ 85,000원", priceUsd: 55, variants: [{vol: "700ml", price: 75000}], desc: "가장 키가 큰 증류기에서 태어난 우아함. 시트러스와 꽃향의 산뜻한 하일랜드 몰트.", descEn: "Born from Scotland's tallest stills. An elegant Highland malt with citrus and floral notes.", country: { ko: "스코틀랜드(하일랜드)", en: "Scotland (Highland)" }, distillery: { ko: "글렌모렌지컴퍼니 (LVMH)", en: "Glenmorangie Co. (LVMH)" } },
            { name: "하이랜드파크 12년", nameEn: "Highland Park 12 Year", age: "12년", style: "Island Single Malt", tags: ["honey", "smoky", "balanced", "rich"], abv: 40, archetype: ["ISFJ", "ISTJ"], priceKrw: 88000, priceRangeKrw: "80,000원 ~ 100,000원", priceUsd: 65, variants: [{vol: "700ml", price: 88000}], desc: "오크니 섬의 헤더 향과 은은한 스모키함이 꿀 풍미와 조화를 이루는 밸런스형 몰트.", descEn: "Orkney heather smoke meets honeyed sweetness in perfect balance.", country: { ko: "스코틀랜드(오크니)", en: "Scotland (Orkney)" }, distillery: { ko: "하이랜드파크 증류소 (에드링턴)", en: "Highland Park Distillery (Edrington)" } },
            { name: "라프로익 10년", nameEn: "Laphroaig 10 Year", age: "10년", style: "Peated Islay", tags: ["smoky", "medicinal", "seaweed", "intense"], abv: 40, archetype: ["ISTJ", "INTJ"], priceKrw: 78000, priceRangeKrw: "70,000원 ~ 88,000원", priceUsd: 60, variants: [{vol: "700ml", price: 78000}], desc: "아일라 피트의 정석. 요오드와 해조류 향이 강렬하게 남는 마니아를 위한 한 잔.", descEn: "The textbook Islay peat. Intense iodine and seaweed notes for peat enthusiasts.", country: { ko: "스코틀랜드(아일라)", en: "Scotland (Islay)" }, distillery: { ko: "라프로익 증류소 (빔산토리)", en: "Laphroaig Distillery (Beam Suntory)" } },
            { name: "보모아 12년", nameEn: "Bowmore 12 Year", age: "12년", style: "Peated Islay", tags: ["smoky", "citrus", "honey", "balanced"], abv: 40, archetype: ["ISFP", "INFJ"], priceKrw: 80000, priceRangeKrw: "72,000원 ~ 90,000원", priceUsd: 62, variants: [{vol: "700ml", price: 80000}], desc: "은은한 스모키함과 시트러스, 꿀향이 균형있게 어우러진 아일라의 순한 얼굴.", descEn: "A gentler Islay face with balanced smoke, citrus, and honey.", country: { ko: "스코틀랜드(아일라)", en: "Scotland (Islay)" }, distillery: { ko: "보모아 증류소 (빔산토리)", en: "Bowmore Distillery (Beam Suntory)" } },
            { name: "오첸토션 12년", nameEn: "Auchentoshan 12 Year", age: "12년", style: "Lowland Single Malt", tags: ["light", "citrus", "vanilla", "smooth"], abv: 40, archetype: ["ISFP", "INFP"], priceKrw: 75000, priceRangeKrw: "65,000원 ~ 85,000원", priceUsd: 55, variants: [{vol: "700ml", price: 75000}], desc: "삼중 증류로 완성한 가볍고 부드러운 로우랜드 몰트. 시트러스와 바닐라의 산뜻함.", descEn: "Triple-distilled Lowland malt, light and smooth with citrus and vanilla.", country: { ko: "스코틀랜드(로우랜드)", en: "Scotland (Lowland)" }, distillery: { ko: "오첸토션 증류소 (빔산토리)", en: "Auchentoshan Distillery (Beam Suntory)" } },
            { name: "아벨라워 12년 더블캐스크", nameEn: "Aberlour 12 Year Double Cask", age: "12년", style: "Sherry Cask", tags: ["sherry", "sweet", "chocolate", "spicy"], abv: 40, archetype: ["INFJ", "ISFJ"], priceKrw: 90000, priceRangeKrw: "80,000원 ~ 100,000원", priceUsd: 68, variants: [{vol: "700ml", price: 90000}], desc: "셰리 캐스크 숙성이 선사하는 진한 초콜릿과 스파이스의 달콤한 여운.", descEn: "Sherry-cask maturation brings rich chocolate and spice sweetness.", country: { ko: "스코틀랜드(스페이사이드)", en: "Scotland (Speyside)" }, distillery: { ko: "아벨라워 증류소 (페르노리카)", en: "Aberlour Distillery (Pernod Ricard)" } },
            { name: "글렌파클라스 12년", nameEn: "Glenfarclas 12 Year", age: "12년", style: "Speyside Single Malt", tags: ["sherry", "rich", "spicy", "oak"], abv: 43, archetype: ["ISTJ", "INTJ"], priceKrw: 85000, priceRangeKrw: "75,000원 ~ 95,000원", priceUsd: 65, variants: [{vol: "700ml", price: 85000}], desc: "가족 경영 증류소가 지켜온 전통. 셰리와 오크의 묵직하고 진한 풍미.", descEn: "Family-owned tradition delivering rich sherry and oak depth.", country: { ko: "스코틀랜드(스페이사이드)", en: "Scotland (Speyside)" }, distillery: { ko: "제이앤지그랜트 / J&G Grant", en: "J&G Grant" } },
            { name: "버나하벤 12년", nameEn: "Bunnahabhain 12 Year", age: "12년", style: "Unpeated Islay", tags: ["maritime", "fruity", "mellow", "sherry"], abv: 46.3, archetype: ["INFP", "ISFJ"], priceKrw: 95000, priceRangeKrw: "85,000원 ~ 105,000원", priceUsd: 72, variants: [{vol: "700ml", price: 95000}], desc: "피트를 쓰지 않는 아일라의 이단아. 바다내음과 과일향이 부드럽게 어우러집니다.", descEn: "Islay's unpeated rebel, blending sea air with mellow fruit.", country: { ko: "스코틀랜드(아일라)", en: "Scotland (Islay)" }, distillery: { ko: "버나하벤 증류소", en: "Bunnahabhain Distillery" } },
            { name: "오반 14년", nameEn: "Oban 14 Year", age: "14년", style: "Highland Single Malt", tags: ["maritime", "smoky", "citrus", "balanced"], abv: 43, archetype: ["ISTP", "ENTP"], priceKrw: 150000, priceRangeKrw: "135,000원 ~ 165,000원", priceUsd: 110, variants: [{vol: "700ml", price: 150000}], desc: "하일랜드와 아일랜드의 경계에서 태어난 바다향과 스모키함의 조화.", descEn: "Born at the crossroads of Highland and Island, balancing sea air and smoke.", country: { ko: "스코틀랜드(하일랜드)", en: "Scotland (Highland)" }, distillery: { ko: "오반 증류소 (디아지오)", en: "Oban Distillery (Diageo)" } },
            { name: "달모어 12년", nameEn: "Dalmore 12 Year", age: "12년", style: "Highland Single Malt", tags: ["sherry", "orange", "chocolate", "rich"], abv: 40, archetype: ["ENFJ", "ISFJ"], priceKrw: 110000, priceRangeKrw: "95,000원 ~ 120,000원", priceUsd: 82, variants: [{vol: "700ml", price: 110000}], desc: "오렌지와 다크초콜릿, 셰리의 진한 풍미가 어우러진 스코틀랜드의 왕관 문양 몰트.", descEn: "Rich orange, dark chocolate, and sherry notes from Scotland's stag-crowned malt.", country: { ko: "스코틀랜드(하일랜드)", en: "Scotland (Highland)" }, distillery: { ko: "달모어 증류소 (와이트앤맥케이)", en: "The Dalmore Distillery (Whyte & Mackay)" } },
            { name: "메이커스마크", nameEn: "Maker's Mark", age: "NAS", style: "Wheated Bourbon", tags: ["sweet", "vanilla", "smooth", "mellow"], abv: 45, archetype: ["ESFP", "ISFP"], priceKrw: 48000, priceRangeKrw: "42,000원 ~ 55,000원", priceUsd: 36, variants: [{vol: "700ml", price: 48000}], desc: "밀을 사용해 부드럽고 달콤한 목넘김을 만든 손으로 봉인하는 켄터키 버번.", descEn: "Wheated for smooth sweetness, hand-dipped Kentucky bourbon.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "메이커스마크 증류소 (빔산토리)", en: "Maker's Mark Distillery (Beam Suntory)" } },
            { name: "우드포드리저브", nameEn: "Woodford Reserve", age: "NAS", style: "Bourbon", tags: ["spicy", "oak", "fruity", "rich"], abv: 43.2, archetype: ["ENTJ", "ISTJ"], priceKrw: 62000, priceRangeKrw: "55,000원 ~ 70,000원", priceUsd: 45, variants: [{vol: "700ml", price: 62000}], desc: "말린 과일과 스파이스, 오크의 풍미가 복합적으로 어우러지는 프리미엄 버번.", descEn: "Complex dried fruit, spice, and oak notes in a premium bourbon.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "우드포드리저브 증류소 (브라운포먼)", en: "Woodford Reserve Distillery (Brown-Forman)" } },
            { name: "포로지스", nameEn: "Four Roses", age: "NAS", style: "Bourbon", tags: ["fruity", "floral", "mellow", "sweet"], abv: 40, archetype: ["ENFP", "ISFP"], priceKrw: 38000, priceRangeKrw: "32,000원 ~ 45,000원", priceUsd: 28, variants: [{vol: "700ml", price: 38000}], desc: "10가지 배합의 조화가 만든 부드럽고 과일향 가득한 버번.", descEn: "A blend of ten recipes creating a mellow, fruit-forward bourbon.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "포로지스 증류소 (기린)", en: "Four Roses Distillery (Kirin)" } },
            { name: "노브크릭", nameEn: "Knob Creek", age: "9년", style: "Small Batch Bourbon", tags: ["bold", "oak", "spicy", "cask_strength"], abv: 50, archetype: ["ENTJ", "ESTJ"], priceKrw: 52000, priceRangeKrw: "45,000원 ~ 60,000원", priceUsd: 40, variants: [{vol: "700ml", price: 52000}], desc: "9년 숙성과 50도의 묵직함이 만든 오크향 가득한 스몰배치 버번.", descEn: "9-year aged, 50% ABV small batch bourbon packed with oak.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "짐빔 증류소 (빔산토리)", en: "Jim Beam Distillery (Beam Suntory)" } },
            { name: "불렛 버번", nameEn: "Bulleit Bourbon", age: "NAS", style: "High Rye Bourbon", tags: ["spicy", "bold", "oak", "vanilla"], abv: 45, archetype: ["ENTP", "ESTP"], priceKrw: 42000, priceRangeKrw: "35,000원 ~ 50,000원", priceUsd: 32, variants: [{vol: "700ml", price: 42000}], desc: "높은 호밀 함량이 주는 스파이시함과 바닐라의 대비가 인상적인 버번.", descEn: "High rye content brings striking spice against vanilla sweetness.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "불렛 디스틸링 컴퍼니 (디아지오)", en: "Bulleit Distilling Co. (Diageo)" } },
            { name: "베이질헤이든", nameEn: "Basil Hayden's", age: "NAS", style: "Bourbon", tags: ["light", "spicy", "smooth", "peppery"], abv: 40, archetype: ["ISFP", "INFP"], priceKrw: 65000, priceRangeKrw: "58,000원 ~ 75,000원", priceUsd: 48, variants: [{vol: "700ml", price: 65000}], desc: "낮은 도수에도 은은한 페퍼 향이 살아있는 가볍고 부드러운 버번.", descEn: "Light bourbon with a gentle peppery spice despite its lower proof.", country: { ko: "미국(켄터키)", en: "USA (Kentucky)" }, distillery: { ko: "짐빔 증류소 (빔산토리)", en: "Jim Beam Distillery (Beam Suntory)" } },
            { name: "부시밀 오리지널", nameEn: "Bushmills Original", age: "NAS", style: "Irish Blend", tags: ["light", "honey", "vanilla", "smooth"], abv: 40, archetype: ["ESFJ", "ISFJ"], priceKrw: 38000, priceRangeKrw: "32,000원 ~ 45,000원", priceUsd: 28, variants: [{vol: "700ml", price: 38000}], desc: "아일랜드에서 가장 오래된 증류소가 만든 가볍고 부드러운 정통 블렌디드.", descEn: "Light and smooth from Ireland's oldest licensed distillery.", country: { ko: "아일랜드", en: "Ireland" }, distillery: { ko: "올드부시밀 증류소 (프록시모)", en: "Old Bushmills Distillery (Proximo Spirits)" } },
            { name: "레드브레스트 12년", nameEn: "Redbreast 12 Year", age: "12년", style: "Single Pot Still", tags: ["rich", "spicy", "fruity", "creamy"], abv: 40, archetype: ["INFJ", "ISFJ"], priceKrw: 95000, priceRangeKrw: "85,000원 ~ 105,000원", priceUsd: 72, variants: [{vol: "700ml", price: 95000}], desc: "싱글 팟 스틸 특유의 크리미함과 스파이시한 과일향이 풍성한 아이리시 위스키.", descEn: "Creamy, spicy, fruit-rich single pot still Irish whiskey.", country: { ko: "아일랜드", en: "Ireland" }, distillery: { ko: "미들턴 증류소 (아이리시 디스틸러스)", en: "Midleton Distillery (Irish Distillers)" } },
            { name: "털리모어듀", nameEn: "Tullamore D.E.W.", age: "NAS", style: "Irish Blend", tags: ["light", "smooth", "honey", "fresh"], abv: 40, archetype: ["ESFP", "ISFP"], priceKrw: 35000, priceRangeKrw: "28,000원 ~ 42,000원", priceUsd: 25, variants: [{vol: "700ml", price: 35000}], desc: "가볍고 상쾌한 목넘김의 아이리시 데일리 블렌디드.", descEn: "A light, refreshing everyday Irish blend.", country: { ko: "아일랜드", en: "Ireland" }, distillery: { ko: "털리모어 증류소 (윌리엄그랜트앤선즈)", en: "Tullamore Distillery (William Grant & Sons)" } },
            { name: "히비키 하모니", nameEn: "Hibiki Harmony", age: "NAS", style: "Japanese Blend", tags: ["floral", "honey", "elegant", "fruity"], abv: 43, archetype: ["INFJ", "ENFJ"], priceKrw: 135000, priceRangeKrw: "120,000원 ~ 150,000원", priceUsd: 100, variants: [{vol: "700ml", price: 135000}], desc: "일본 6곳의 원액이 만드는 조화. 꽃과 꿀 향이 우아하게 어우러진 블렌디드.", descEn: "Harmony from six Japanese distilleries, elegant floral and honey notes.", country: { ko: "일본", en: "Japan" }, distillery: { ko: "산토리", en: "Suntory" } },
            { name: "니카 프롬 더 배럴", nameEn: "Nikka From The Barrel", age: "NAS", style: "Blended Whisky", tags: ["bold", "rich", "spicy", "cask_strength"], abv: 51.4, archetype: ["ENTJ", "ISTP"], priceKrw: 65000, priceRangeKrw: "55,000원 ~ 75,000원", priceUsd: 48, variants: [{vol: "500ml", price: 65000}], desc: "캐스크 스트렝스에 가까운 51.4도의 진하고 강렬한 일본 블렌디드.", descEn: "Bold, near-cask-strength Japanese blend at 51.4% ABV.", country: { ko: "일본", en: "Japan" }, distillery: { ko: "니카 위스키 (아사히)", en: "Nikka Whisky (Asahi)" } },
            { name: "니카 코피그레인", nameEn: "Nikka Coffey Grain", age: "NAS", style: "Grain Whisky", tags: ["sweet", "vanilla", "corn", "smooth"], abv: 45, archetype: ["ESFP", "ISFP"], priceKrw: 90000, priceRangeKrw: "80,000원 ~ 100,000원", priceUsd: 68, variants: [{vol: "700ml", price: 90000}], desc: "커피식 증류기로 만든 달콤한 옥수수향의 부드러운 그레인 위스키.", descEn: "Coffey-still grain whisky with sweet corn smoothness.", country: { ko: "일본", en: "Japan" }, distillery: { ko: "니카 위스키 (아사히)", en: "Nikka Whisky (Asahi)" } },
            { name: "하쿠슈 12년", nameEn: "Hakushu 12 Year", age: "12년", style: "Japanese Single Malt", tags: ["fresh", "unique", "smoky", "floral"], abv: 43, archetype: ["INFP", "INTP"], priceKrw: 180000, priceRangeKrw: "160,000원 ~ 200,000원", priceUsd: 130, variants: [{vol: "700ml", price: 180000}], desc: "숲 속의 증류소가 빚어낸 신선하고 청량한 그린 노트의 싱글몰트.", descEn: "Fresh, green, forest-distillery single malt with a unique herbal character.", country: { ko: "일본", en: "Japan" }, distillery: { ko: "산토리 (하쿠슈 증류소)", en: "Suntory (Hakushu Distillery)" } },
            { name: "김창수 위스키", nameEn: "Kim Chang Soo Whisky", age: "NAS", style: "Korean Single Malt", tags: ["unique", "complex", "fruity", "oak"], abv: 53, archetype: ["INTJ", "INFP"], priceKrw: 140000, priceRangeKrw: "120,000원 ~ 160,000원", priceUsd: 105, variants: [{vol: "700ml", price: 140000}], desc: "한국 최초의 개인 증류소가 빚어낸 독창적이고 복합적인 캐스크 스트렝스 몰트.", descEn: "Korea's first personal distillery, a unique and complex cask-strength malt.", country: { ko: "한국", en: "Korea" }, distillery: { ko: "김창수 증류소", en: "Kichang Distillery" } },
            { name: "기원", nameEn: "Ki-One", age: "NAS", style: "Korean Single Malt", tags: ["fresh", "fruity", "honey", "unique"], abv: 43, archetype: ["ENFP", "ISFP"], priceKrw: 130000, priceRangeKrw: "110,000원 ~ 150,000원", priceUsd: 95, variants: [{vol: "700ml", price: 130000}], desc: "한국의 물과 보리로 빚은 신선하고 과일향 가득한 국산 싱글몰트의 시작.", descEn: "Korea's pioneering single malt, fresh and fruity from local water and barley.", country: { ko: "한국", en: "Korea" }, distillery: { ko: "쓰리소사이어티스", en: "Three Societies" } }
        ];
```

- [ ] **Step 2: 구조적 확인**

`preview_*` 없이도 확인 가능한 정적 체크: `grep -c "nameEn:" index.html`이 60이어야 한다.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: 위스키 30종 신규 추가 (SPIRITS_DB 30 -> 60)"
```

---

### Task 3: 홈 탭 데일리 픽 카드에 원산지/증류소/구매 버튼 추가

**Files:**
- Modify: `index.html` (`HomeTab` 컴포넌트, 데일리 픽 `GlassCard` 블록)

**Interfaces:**
- Consumes: Task 1/2에서 추가된 `dailyPick.country`, `dailyPick.distillery`, `dailyPick.age`. 기존 `TEXT[lang].buy_btn`(이미 정의됨, `index.html:242,255`).
- Produces: 없음 (leaf 변경)

- [ ] **Step 1: 데일리 픽 `GlassCard` 내부에 메타데이터 행 + 구매 버튼 추가**

다음 블록을 찾는다:

```jsx
                            <GlassCard className="p-5 flex gap-5 items-center">
                                <div className="w-20 h-24 rounded-2xl flex items-center justify-center shrink-0 shadow-inner" style={{ background: `linear-gradient(135deg, ${dailyPick.color}90, ${dailyPick.color}50)` }}>
                                    <Icon name="glass-water" className="w-10 h-10 text-slate-900/90 dark:text-white/90" />
                                </div>
                                <div className="flex-1">
                                    <p className="text-sm text-[#c8962a] font-bold mb-1.5 uppercase tracking-wide">{dailyPick.style} · {dailyPick.abv}%</p>
                                    <h4 className="text-[1.35rem] font-bold text-slate-900 dark:text-white mb-2 leading-tight">{lang==='ko'?dailyPick.name:dailyPick.nameEn}</h4>
                                    <p className="text-sm text-slate-500 dark:text-slate-400 line-clamp-2 leading-relaxed">{lang==='ko'?dailyPick.desc:dailyPick.descEn}</p>
                                </div>
                            </GlassCard>
```

다음으로 바꾼다:

```jsx
                            <GlassCard className="p-5">
                                <div className="flex gap-5 items-center">
                                    <div className="w-20 h-24 rounded-2xl flex items-center justify-center shrink-0 shadow-inner" style={{ background: `linear-gradient(135deg, ${dailyPick.color}90, ${dailyPick.color}50)` }}>
                                        <Icon name="glass-water" className="w-10 h-10 text-slate-900/90 dark:text-white/90" />
                                    </div>
                                    <div className="flex-1">
                                        <p className="text-sm text-[#c8962a] font-bold mb-1.5 uppercase tracking-wide">{dailyPick.style} · {dailyPick.abv}%</p>
                                        <h4 className="text-[1.35rem] font-bold text-slate-900 dark:text-white mb-2 leading-tight">{lang==='ko'?dailyPick.name:dailyPick.nameEn}</h4>
                                        <p className="text-sm text-slate-500 dark:text-slate-400 line-clamp-2 leading-relaxed">{lang==='ko'?dailyPick.desc:dailyPick.descEn}</p>
                                    </div>
                                </div>
                                <div className="flex items-center gap-2 mt-4 pt-4 border-t border-black/5 dark:border-white/10 text-xs text-slate-500 dark:text-slate-400">
                                    <Icon name="map-pin" className="w-3.5 h-3.5" />
                                    <span>{lang==='ko' ? dailyPick.country.ko : dailyPick.country.en}</span>
                                    <span>·</span>
                                    <span>{lang==='ko' ? dailyPick.distillery.ko : dailyPick.distillery.en}</span>
                                    {dailyPick.age !== 'NAS' && (<><span>·</span><span>{dailyPick.age}</span></>)}
                                </div>
                                <button
                                    onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent((lang==='ko'?dailyPick.name:dailyPick.nameEn) + ' 위스키')}`, '_blank')}
                                    className="w-full mt-3 py-3 rounded-2xl bg-slate-100 dark:bg-slate-800 border-2 border-slate-200 dark:border-slate-700 text-[#c8962a] font-bold text-sm hover:bg-slate-200 dark:hover:bg-slate-700 transition-all flex items-center justify-center gap-2"
                                >
                                    <Icon name="shopping-bag" className="w-4 h-4" />
                                    {TEXT[lang].buy_btn}
                                </button>
                            </GlassCard>
```

- [ ] **Step 2: 커밋**

```bash
git add index.html
git commit -m "feat: 홈 탭 데일리 픽에 원산지/증류소/구매 버튼 추가"
```

---

### Task 4: 무드 이모지를 상황 묘사형으로 교체

**Files:**
- Modify: `index.html` (`MOODS` 배열, 274-286번째 줄 부근 — Task 1/2/3과 무관한 독립적 변경)

**Interfaces:**
- Consumes: 없음
- Produces: 없음 (leaf 변경, `id`/`conflicts`/`flavorProfile`/`label`/`desc`/`expr`는 전혀 건드리지 않는다 — `emoji` 필드만 교체)

- [ ] **Step 1: 12개 무드의 `emoji` 값을 얼굴 표정이 아닌 상황 묘사 이모지로 교체**

각 무드 객체 줄에서 `emoji: "..."` 부분만 아래 표대로 교체한다(그 외 필드는 전혀 수정하지 않는다):

| id | 기존 emoji | 신규 emoji | 의도 |
|---|---|---|---|
| delight | 😆 | 🎈 | 이유 없이 붕 뜨는 기분 — 풍선 |
| joy | 😁 | 🎉 | 축하할 일 — 축하 파티 |
| sadness | 😢 | 🌧️ | 위로가 필요한 날 — 비 내리는 하늘 |
| anger | 😡 | 🌋 | 스트레스 폭발 — 화산 폭발 |
| trust | 😌 | ☕ | 평화롭고 편안한 하루 — 따뜻한 커피 한 잔 |
| fear | 😨 | ⛈️ | 두렵고 걱정되는 — 폭풍우 |
| surprise | 😲 | 🎇 | 예상 못한 깜짝 놀람 — 스파클러 불꽃 |
| anticipation | 🤩 | 🎁 | 다가올 즐거운 일정 — 선물상자 |
| disgust | 😖 | 🌡️ | 불쾌지수가 끝까지 올라간 — 치솟는 온도계 |
| boredom | 🥱 | 🕰️ | 시간만 흘러간 따분함 — 시계 |
| nostalgia | 🍂 | 🍂 (변경 없음) | 이미 상황 묘사형 |
| pride | 😎 | 🏆 | 해낸 일에 대견함 — 트로피 |

구체적으로, 예를 들어 `delight` 줄은:

before:
```
{ id: 'delight', label: { ko: '즐거움', en: 'Delight' }, desc: { ko: "즐거운 날이에요. 즐거움에는 이유가 없어요.", en: "It's a joyful day." }, expr: { ko: "이유 없이 자꾸만 웃음이 나는 기분 좋은 하루", en: "A pleasant day that makes me smile" }, emoji: "😆", conflicts: ['sadness', 'anger', 'fear', 'disgust'], flavorProfile: ['sweet', 'fruity', 'floral', 'fresh'] },
```
after (emoji만 변경):
```
{ id: 'delight', label: { ko: '즐거움', en: 'Delight' }, desc: { ko: "즐거운 날이에요. 즐거움에는 이유가 없어요.", en: "It's a joyful day." }, expr: { ko: "이유 없이 자꾸만 웃음이 나는 기분 좋은 하루", en: "A pleasant day that makes me smile" }, emoji: "🎈", conflicts: ['sadness', 'anger', 'fear', 'disgust'], flavorProfile: ['sweet', 'fruity', 'floral', 'fresh'] },
```

이 방식으로 `joy`(`emoji: "😁"` → `"🎉"`), `sadness`(`"😢"` → `"🌧️"`), `anger`(`"😡"` → `"🌋"`), `trust`(`"😌"` → `"☕"`), `fear`(`"😨"` → `"⛈️"`), `surprise`(`"😲"` → `"🎇"`), `anticipation`(`"🤩"` → `"🎁"`), `disgust`(`"😖"` → `"🌡️"`), `boredom`(`"🥱"` → `"🕰️"`), `pride`(`"😎"` → `"🏆"`)를 각각 해당 줄의 `emoji: "..."` 부분만 교체한다. `nostalgia`(`"🍂"`)는 변경하지 않는다.

- [ ] **Step 2: 구조적 확인**

`grep -o 'emoji: "[^"]*"' index.html`로 12개 줄이 출력되는지, 그리고 더 이상 `😆|😁|😢|😡|😌|😨|😲|🤩|😖|🥱|😎` 얼굴 이모지가 `MOODS` 배열 안에 남아있지 않은지 확인한다.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: 무드 선택 이모지를 얼굴 표정에서 상황 묘사형으로 교체"
```

---

### Task 5: 전체 회귀 확인

**Files:**
- 변경 없음 (검증 전용 태스크)

**Interfaces:**
- Consumes: Task 1~4의 모든 변경사항
- Produces: 없음

- [ ] **Step 1: `SPIRITS_DB` 크기와 필드 무결성 확인**

`preview_eval`로 `SPIRITS_DB.length === 60`, 그리고 모든 항목이 `country`/`distillery` 필드를 가지고 있는지(`SPIRITS_DB.every(w => w.country && w.distillery)`) 확인한다.

- [ ] **Step 2: 홈 탭 데일리 픽 카드 확인 (라이트/다크)**

여러 MBTI(예: INTJ, ENFP, ISFJ)로 홈 탭에 진입해 데일리 픽 카드에 원산지/증류소/구매 버튼이 라이트·다크 모드 양쪽에서 잘 보이는지 스크린샷으로 확인한다. 구매 버튼 클릭 시 새 탭에서 구글 검색이 열리는지(`preview_network` 또는 새 탭 URL) 확인한다.

- [ ] **Step 3: 신규 60종이 스코어링에 정상 포함되는지 확인**

큐레이션 플로우(무드 선택 → 가격대 → 결과)를 몇 차례 실행해, 결과 화면에 신규 30종 중 최소 1개 이상이 등장하는지 확인한다(예: 고가 구간을 선택해 하쿠슈/히비키/야마자키 등 프리미엄 항목이 나오는지, 저가 구간에서 발렌타인 파이니스트/포로지스 등이 나오는지).

- [ ] **Step 4: 무드 이모지 교체 확인**

큐레이션 1단계 화면에서 12개 무드 카드에 더 이상 얼굴 이모지가 아닌 상황 묘사 이모지(🎈🎉🌧️🌋☕⛈️🎇🎁🌡️🕰️🍂🏆)가 표시되는지 확인한다.

- [ ] **Step 5: 문제 발견 시 처리**

발견된 문제는 해당 태스크로 돌아가 수정 후 그 태스크의 커밋을 새로 추가한다. 이 태스크 자체는 코드 변경이 없으므로 문제가 없다면 커밋 없이 종료한다.
