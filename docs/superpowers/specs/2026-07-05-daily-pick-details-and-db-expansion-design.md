# 데일리 픽 상세정보/구매링크 + 위스키 DB 확장 설계

## 배경
홈 탭 "오늘의 데일리 픽" 카드에는 현재 이름/스타일/도수/한 줄 설명만 있고, 원산지·증류소 정보나 구매 링크가 없다. 또한 추천 대상 위스키 DB(`SPIRITS_DB`)가 30종뿐이라 추천 다양성이 제한적이다. 이번 작업으로 (1) 데일리 픽 카드에 원산지/증류소/구매링크를 추가하고, (2) 한국에 잘 알려진 수입 위스키 30종을 신규로 추가해 DB를 30→60종으로 확장한다.

## 범위 밖
- 큐레이션 결과 화면(`CurateTab` cStep4)은 이미 구매 링크(`buy_btn`)가 있으므로 변경하지 않는다 — 이번 변경은 홈 탭 데일리 픽 카드에만 적용
- 실시간 가격 연동(API)은 하지 않는다 — 기존 30종과 동일하게 근사치 KRW 가격을 정적으로 기입
- MBTI 분석/큐레이션 스코어링 알고리즘(`CurateTab.run`, `HomeTab`의 `dailyPick` 스코어링) 자체는 변경하지 않는다 — 신규 60종도 기존 스코어링 로직이 그대로 적용되도록 동일한 데이터 shape을 따른다
- "한국에 수입된 모든 위스키"의 전수 조사는 하지 않는다 — 웹 검색으로 검증한 잘 알려진 브랜드 30종으로 확장 범위를 한정한다(사용자 확인 완료)

## 1. 데일리 픽 카드 UI 변경 (`HomeTab`)

현재 카드 구조(`index.html` `HomeTab`의 `GlassCard` 블록, 대략 710-719번째 줄):
```jsx
<GlassCard className="p-5 flex gap-5 items-center">
    <div className="w-20 h-24 ...">...</div>
    <div className="flex-1">
        <p>{dailyPick.style} · {dailyPick.abv}%</p>
        <h4>{name}</h4>
        <p>{desc}</p>
    </div>
</GlassCard>
```

변경 후 구조: 아이콘+이름/설명 행은 그대로 유지하고, 그 아래에 (a) 원산지/증류소 메타데이터 행, (b) "구매처 알아보기" 버튼을 세로로 추가한다. 즉 `GlassCard` 내부를 `flex-col`로 감싸는 wrapper를 하나 추가하고, 기존 `flex gap-5 items-center` 행을 그 안의 첫 자식으로 넣는다.

메타데이터 행 예시:
```jsx
<div className="flex items-center gap-2 mt-4 pt-4 border-t border-black/5 dark:border-white/10 text-xs text-slate-500 dark:text-slate-400">
    <Icon name="map-pin" className="w-3.5 h-3.5" />
    <span>{lang==='ko' ? dailyPick.country.ko : dailyPick.country.en}</span>
    <span>·</span>
    <span>{lang==='ko' ? dailyPick.distillery.ko : dailyPick.distillery.en}</span>
    {dailyPick.age !== 'NAS' && <><span>·</span><span>{dailyPick.age}</span></>}
</div>
```

구매 버튼(`CurateTab`의 기존 `buy_btn` 패턴과 동일한 구글 검색 링크 재사용):
```jsx
<button
    onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent((lang==='ko'?dailyPick.name:dailyPick.nameEn) + ' 위스키')}`, '_blank')}
    className="w-full mt-3 py-3 rounded-2xl bg-slate-100 dark:bg-slate-800 border-2 border-slate-200 dark:border-slate-700 text-[#c8962a] font-bold text-sm hover:bg-slate-200 dark:hover:bg-slate-700 transition-all flex items-center justify-center gap-2"
>
    <Icon name="shopping-bag" className="w-4 h-4" />
    {TEXT[lang].buy_btn}
</button>
```
`TEXT[lang].buy_btn`("구매처 알아보기"/"Find Where to Buy")은 이미 정의돼 있으므로 재사용한다.

## 2. `country`/`distillery` 필드 (전체 60종 공통)

`SPIRITS_DB`의 모든 항목(기존 30 + 신규 30)에 다음 두 필드를 추가한다:
```js
country: { ko: "스코틀랜드", en: "Scotland" },
distillery: { ko: "글렌피딕 증류소 (윌리엄그랜트앤선즈)", en: "Glenfiddich Distillery (William Grant & Sons)" },
```

### 기존 30종 country/distillery (웹 검색으로 검증됨)

| 위스키 (name) | country.ko / en | distillery.ko / en |
|---|---|---|
| 랭스 블렌디드 위스키 | 스코틀랜드 / Scotland | 이안 맥클라우드 디스틸러스 / Ian Macleod Distillers |
| 길리듀 | 스코틀랜드 / Scotland | 앵거스 던디 디스틸러스 / Angus Dundee Distillers |
| 존바 리저브 | 스코틀랜드 / Scotland | 와일드 앤 맥케이 / Whyte & Mackay |
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
| 맥캘란 12년 더블캐스크 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 맥캘란 증류소 (에드링턴) / The Macallan Distillery (Edrington) |
| 로얄 브라클라 12년 | 스코틀랜드(하일랜드) / Scotland (Highland) | 로얄브라클라 증류소 (바카디) / Royal Brackla Distillery (Bacardi) |
| 라가불린 16년 | 스코틀랜드(아일라) / Scotland (Islay) | 라가불린 증류소 (디아지오) / Lagavulin Distillery (Diageo) |
| 글렌피딕 15년 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 글렌피딕 증류소 (윌리엄그랜트앤선즈) / Glenfiddich Distillery (William Grant & Sons) |
| 발베니 12년 더블우드 | 스코틀랜드(스페이사이드) / Scotland (Speyside) | 발베니 증류소 (윌리엄그랜트앤선즈) / The Balvenie Distillery (William Grant & Sons) |
| 아드벡 코리브레칸 | 스코틀랜드(아일라) / Scotland (Islay) | 아드벡 증류소 (글렌모렌지컴퍼니/LVMH) / Ardbeg Distillery (Glenmorangie Co./LVMH) |
| 조니워커 블루라벨 | 스코틀랜드 / Scotland | 디아지오 / Diageo |
| 야마자키 12년 | 일본 / Japan | 야마자키 증류소 (산토리) / Yamazaki Distillery (Suntory) |
| 로얄살루트 21년 | 스코틀랜드 / Scotland | 시바스브라더스 (페르노리카) / Chivas Brothers (Pernod Ricard) |

## 3. 신규 30종 데이터 (`SPIRITS_DB`에 추가)

기존 항목과 동일한 필드 shape(`name`, `nameEn`, `age`, `style`, `tags`, `abv`, `archetype`, `priceKrw`, `priceRangeKrw`, `priceUsd`, `variants`, `desc`, `descEn`, `country`, `distillery`)을 따른다. `tags`는 `MBTI_TRAIT_PREFS`(E/I/S/N/T/F/J/P 취향 태그맵, `index.html:289-294`)에 이미 정의된 태그 어휘만 사용해 기존 스코어링 로직과 호환되게 한다. `archetype`은 태그와 `MBTI_TRAIT_PREFS`의 매칭도가 높은 2~3개 MBTI를 선정한다(기존 30종도 동일한 방식으로 수기 선정된 패턴).

| # | name / nameEn | country | distillery | age | style | abv | 근사 KRW |
|---|---|---|---|---|---|---|---|
| 1 | 발렌타인 파이니스트 / Ballantine's Finest | 스코틀랜드 | Chivas Brothers (Pernod Ricard) | NAS | Blended Scotch | 40 | 28,000 |
| 2 | 발렌타인 17년 / Ballantine's 17 Year | 스코틀랜드 | Chivas Brothers (Pernod Ricard) | 17년 | Blended Scotch | 40 | 115,000 |
| 3 | 시바스리갈 12년 / Chivas Regal 12 Year | 스코틀랜드 | Chivas Brothers (Pernod Ricard) | 12년 | Blended Scotch | 40 | 55,000 |
| 4 | 시바스리갈 18년 / Chivas Regal 18 Year | 스코틀랜드 | Chivas Brothers (Pernod Ricard) | 18년 | Blended Scotch | 40 | 150,000 |
| 5 | 듀어스 화이트라벨 / Dewar's White Label | 스코틀랜드 | John Dewar & Sons (Bacardi) | NAS | Blended Scotch | 40 | 32,000 |
| 6 | 글렌모렌지 오리지널 10년 / Glenmorangie Original 10 Year | 스코틀랜드(하일랜드) | Glenmorangie Co. (LVMH) | 10년 | Highland Single Malt | 40 | 75,000 |
| 7 | 하이랜드파크 12년 / Highland Park 12 Year | 스코틀랜드(오크니) | Highland Park Distillery (Edrington) | 12년 | Island Single Malt | 40 | 88,000 |
| 8 | 라프로익 10년 / Laphroaig 10 Year | 스코틀랜드(아일라) | Laphroaig Distillery (Beam Suntory) | 10년 | Peated Islay | 40 | 78,000 |
| 9 | 보모아 12년 / Bowmore 12 Year | 스코틀랜드(아일라) | Bowmore Distillery (Beam Suntory) | 12년 | Peated Islay | 40 | 80,000 |
| 10 | 오첸토션 12년 / Auchentoshan 12 Year | 스코틀랜드(로우랜드) | Auchentoshan Distillery (Beam Suntory) | 12년 | Lowland Single Malt | 40 | 75,000 |
| 11 | 아벨라워 12년 더블캐스크 / Aberlour 12 Year Double Cask | 스코틀랜드(스페이사이드) | Aberlour Distillery (Pernod Ricard) | 12년 | Sherry Cask | 40 | 90,000 |
| 12 | 글렌파클라스 12년 / Glenfarclas 12 Year | 스코틀랜드(스페이사이드) | J&G Grant | 12년 | Speyside Single Malt | 43 | 85,000 |
| 13 | 버나하벤 12년 / Bunnahabhain 12 Year | 스코틀랜드(아일라) | Bunnahabhain Distillery | 12년 | Unpeated Islay | 46.3 | 95,000 |
| 14 | 오반 14년 / Oban 14 Year | 스코틀랜드(하일랜드) | Oban Distillery (Diageo) | 14년 | Highland Single Malt | 43 | 150,000 |
| 15 | 달모어 12년 / Dalmore 12 Year | 스코틀랜드(하일랜드) | The Dalmore Distillery (Whyte & Mackay) | 12년 | Highland Single Malt | 40 | 110,000 |
| 16 | 메이커스마크 / Maker's Mark | 미국(켄터키) | Maker's Mark Distillery (Beam Suntory) | NAS | Wheated Bourbon | 45 | 48,000 |
| 17 | 우드포드리저브 / Woodford Reserve | 미국(켄터키) | Woodford Reserve Distillery (Brown-Forman) | NAS | Bourbon | 43.2 | 62,000 |
| 18 | 포로지스 / Four Roses | 미국(켄터키) | Four Roses Distillery (Kirin) | NAS | Bourbon | 40 | 38,000 |
| 19 | 노브크릭 / Knob Creek | 미국(켄터키) | Jim Beam Distillery (Beam Suntory) | 9년 | Small Batch Bourbon | 50 | 52,000 |
| 20 | 불렛 버번 / Bulleit Bourbon | 미국(켄터키) | Bulleit Distilling Co. (Diageo) | NAS | High Rye Bourbon | 45 | 42,000 |
| 21 | 베이질헤이든 / Basil Hayden's | 미국(켄터키) | Jim Beam Distillery (Beam Suntory) | NAS | Bourbon | 40 | 65,000 |
| 22 | 부시밀 오리지널 / Bushmills Original | 아일랜드 | Old Bushmills Distillery (Proximo Spirits) | NAS | Irish Blend | 40 | 38,000 |
| 23 | 레드브레스트 12년 / Redbreast 12 Year | 아일랜드 | Midleton Distillery (Irish Distillers) | 12년 | Single Pot Still | 40 | 95,000 |
| 24 | 털리모어듀 / Tullamore D.E.W. | 아일랜드 | Tullamore Distillery (William Grant & Sons) | NAS | Irish Blend | 40 | 35,000 |
| 25 | 히비키 하모니 / Hibiki Harmony | 일본 | Suntory | NAS | Japanese Blend | 43 | 135,000 |
| 26 | 니카 프롬 더 배럴 / Nikka From The Barrel | 일본 | Nikka Whisky (Asahi) | NAS | Blended Whisky | 51.4 | 65,000 |
| 27 | 니카 코피그레인 / Nikka Coffey Grain | 일본 | Nikka Whisky (Asahi) | NAS | Grain Whisky | 45 | 90,000 |
| 28 | 하쿠슈 12년 / Hakushu 12 Year | 일본 | Suntory (Hakushu Distillery) | 12년 | Japanese Single Malt | 43 | 180,000 |
| 29 | 김창수 위스키 / Kim Chang Soo Whisky | 한국 | 김창수 증류소 / Kichang Distillery | NAS | Korean Single Malt | 53 | 140,000 |
| 30 | 기원 / Ki-One | 한국 | 쓰리소사이어티스 / Three Societies | NAS | Korean Single Malt | 43 | 130,000 |

각 항목의 `tags`/`archetype`/`desc`/`descEn`/`variants`/`priceRangeKrw`/`priceUsd`는 구현 계획(plan) 단계에서 위 표의 style/abv/country 정보를 바탕으로 기존 30종과 같은 형식으로 구체적으로 작성한다(예: 라프로익 10년 → `tags: ['smoky','medicinal','seaweed','intense']`, `archetype: ['ISTJ','INTJ']` 같은 식 — 아드벡 코리브레�칸과 유사한 피트 계열이므로 유사 아키타입 참고).

## 4. 국기/증류소 아이콘

`Icon` 컴포넌트는 lucide 아이콘명을 그대로 사용하므로 `name="map-pin"`을 새로 사용한다(lucide 기본 제공 아이콘, 별도 등록 불필요).

## 테스트 계획
- 자동화 테스트 없음 — 브라우저로 확인
- 홈 탭 데일리 픽 카드에 원산지/증류소/구매 버튼이 라이트·다크 모드 양쪽에서 잘 보이는지 확인
- 구매 버튼 클릭 시 새 탭에서 구글 검색이 정상적으로 열리는지 확인
- `SPIRITS_DB.length`가 60인지 확인
- 신규 60종 전체가 큐레이션 스코어링(`CurateTab.run`)에서 에러 없이 후보로 고려되는지 확인 — 예를 들어 가격대 필터(`PRICE_RANGES`)에서 신규 고가 항목(하쿠슈, 히비키 등)이 상위 구간에서 정상 노출되는지 확인
- MBTI 홈 탭 데일리 픽에서 신규 항목이 하나 이상 실제로 추천되는 케이스를 최소 1회 확인(예: 여러 MBTI로 순회하며 다양성 확인)
