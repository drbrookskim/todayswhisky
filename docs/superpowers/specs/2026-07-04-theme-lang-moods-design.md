# 언어 설정 통합 / 라이트-다크 테마 / 무드 확장 설계

## 배경
현재 세 가지 개선 요청을 다룬다:
1. 언어(한국어/English) 변경 버튼이 온보딩, 프로필, 데스크톱 상단 네비 3곳에 흩어져 있다. 프로필에서만 설정하도록 통합한다.
2. 다크 모드가 `<html class="dark">`로 하드코딩되어 있고 실제 라이트 모드가 전혀 없다. 실제 라이트/다크 토글을 프로필에 추가한다.
3. 큐레이션 1단계("오늘 하루를 한 문장으로 표현한다면?")의 무드 카드가 10개인데 12개로 확장한다.

## 범위 밖
- MBTI 분석, 위스키 추천 스코어링 로직 변경 없음
- 데스크톱 반응형 레이아웃(이전 작업) 재변경 없음
- 공유 카드(Canvas) 이미지는 테마와 무관하게 항상 다크 브랜드 톤 고정 — 라이트 모드 대응 대상에서 제외

---

## 1. 언어 설정 통합

- **제거**: `Onboarding` 컴포넌트의 우상단 EN/KR 토글 버튼(`index.html:580` 부근), `App`의 데스크톱 상단 네비 `.desktop-nav-lang` 버튼(`index.html:966-968` 부근)과 관련 CSS(`.desktop-nav-lang`, `.desktop-nav-lang:hover` 규칙, `index.html:178-193`)
- **유지**: `ProfileTab`의 언어 변경 버튼(`index.html:872-876`) — 유일한 언어 변경 경로가 된다
- **로직 변경 없음**: `lang`/`setLang` state, `whiskey_lang` localStorage 저장 로직(`App`의 `useEffect`, `index.html:906-908`)은 그대로 유지. 온보딩은 항상 마지막으로 저장된 언어(기본 'ko')로 표시되며, 사용자는 이제 프로필에 진입한 뒤에만 언어를 바꿀 수 있다.

## 2. 라이트/다크 테마

### 2.1 State & 초기화
`App`에 `theme` state 추가:
```js
const [theme, setTheme] = useState('dark'); // placeholder, replaced in useEffect below
```
초기화 `useEffect` (기존 mbti/lang 초기화 `useEffect`와 합치거나 별도로 추가):
```js
useEffect(() => {
    const savedTheme = safeStorage.getItem('whiskey_theme');
    if (savedTheme) setTheme(savedTheme);
    else if (window.matchMedia && window.matchMedia('(prefers-color-scheme: light)').matches) setTheme('light');
}, []);

useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
    safeStorage.setItem('whiskey_theme', theme);
}, [theme]);
```
- 저장된 값이 없으면 기기의 `prefers-color-scheme`을 따른다(라이트 선호 기기 → 라이트, 그 외 다크 — 현재 `<html class="dark">` 하드코딩과 동일한 기본값 유지).
- 사용자가 프로필에서 토글하면 즉시 `whiskey_theme`에 저장되어 다음 방문에도 유지된다.
- `<html>`의 정적 `class="dark"`는 그대로 두되(JS 비활성 시 폴백), 마운트 시 위 effect가 실제 상태에 맞게 클래스를 조정한다.

### 2.2 CSS 변수 구조 반전
현재 `:root`가 다크 값을 직접 정의한다(`index.html:22-30`). 이를 Tailwind `darkMode:'class'` 표준 패턴으로 반전:

```css
:root {
    /* Light (default) */
    --bg: #f7f5f2;
    --card: rgba(0, 0, 0, 0.03);
    --border: rgba(0, 0, 0, 0.08);
    --accent: #96701e;
    --accent-dim: rgba(150, 112, 30, 0.12);
    --text: #1a1512;
    --text-dim: #6b6b6b;
    --nav-bg: rgba(255, 255, 255, 0.85);
    --shell-shadow: 0 0 40px rgba(0, 0, 0, 0.12);
    --body-bg-solid: #f2ede6;
    --body-bg-image: radial-gradient(ellipse at 50% -10%, rgba(13, 148, 136, 0.08) 0%, #f7f5f2 70%);
}

html.dark {
    --bg: #080604;
    --card: rgba(255, 255, 255, 0.035);
    --border: rgba(255, 255, 255, 0.08);
    --accent: #c8962a;
    --accent-dim: rgba(200, 150, 42, 0.15);
    --text: #f8f8f8;
    --text-dim: #888888;
    --nav-bg: rgba(8, 6, 4, 0.88);
    --shell-shadow: 0 0 40px rgba(0, 0, 0, 0.8);
    --body-bg-solid: #0f172a;
    --body-bg-image: radial-gradient(ellipse at 50% -10%, rgba(13, 148, 136, 0.15) 0%, #080604 70%);
}
```

주의: `--accent`는 다크에서 `#c8962a`(기존 값 유지), 라이트에서는 밝은 배경 위 텍스트 대비를 위해 약간 더 어두운 `#96701e`를 사용한다. 골드 브랜드 톤은 유지하되 WCAG 대비를 확보한다.

`body`, `.app-container`, `.bottom-nav`, `.desktop-nav-lang`(제거 대상이므로 스킵) 규칙에서 하드코딩된 값을 변수로 교체:
```css
body {
    ...
    background-color: var(--body-bg-solid);
    background-image: var(--body-bg-image);
    ...
}
.app-container {
    ...
    box-shadow: var(--shell-shadow);
    ...
}
.bottom-nav {
    ...
    background: var(--nav-bg);
    ...
}
```
`@media (min-width:1024px)` 블록의 `.desktop-nav .nav-menu`, `.glass-panel` 등은 이미 `var(--card)`/`var(--border)`를 쓰고 있어 자동 대응된다. `.nav-item.active`와 `.desktop-nav .nav-menu-item.active`의 `#14b8a6`(활성 탭 틸 컬러)는 브랜드 포인트 컬러로 라이트/다크 공통 유지 — 변경하지 않는다.

### 2.3 컴포넌트별 Tailwind 클래스 (라이트 기본 + `dark:` variant)
다음 치환 규칙을 파일 전체(Splash, Onboarding, Quiz, DirectSelect, HomeTab, CurateTab, ProfileTab, ShareCard 패널(캔버스 제외), LoadingScreen)에 기계적으로 적용한다. **패턴: 기존 클래스가 라이트에서 부적절하면 라이트용 클래스를 앞에 추가하고 기존 클래스에 `dark:` 접두사를 붙여 뒤에 둔다.**

| 기존 (다크 전용) | 신규 (라이트 기본 + dark: variant) |
|---|---|
| `bg-slate-800` | `bg-slate-100 dark:bg-slate-800` |
| `bg-slate-800/50` | `bg-slate-100/70 dark:bg-slate-800/50` |
| `bg-slate-900` | `bg-white dark:bg-slate-900` |
| `bg-slate-700/50` | `bg-slate-300/50 dark:bg-slate-700/50` |
| `bg-black/40` | `bg-black/5 dark:bg-black/40` |
| `text-white` | `text-slate-900 dark:text-white` |
| `text-white/90` | `text-slate-900/90 dark:text-white/90` |
| `text-slate-100` | `text-slate-800 dark:text-slate-100` |
| `text-slate-200` | `text-slate-700 dark:text-slate-200` |
| `text-slate-300` | `text-slate-600 dark:text-slate-300` |
| `text-slate-400` | `text-slate-500 dark:text-slate-400` |
| `text-slate-500` | `text-slate-400 dark:text-slate-500` |
| `border-slate-700` | `border-slate-200 dark:border-slate-700` |
| `border-slate-500` | `border-slate-400 dark:border-slate-500` |
| `border-white/5` | `border-black/5 dark:border-white/5` |
| `border-white/10` | `border-black/10 dark:border-white/10` |

색상 있는 텍스트/배경(`text-purple-400`, `bg-purple-900/30`, `text-red-400`, `bg-red-900/20` 등 MBTI 그룹·리셋 버튼 색상)은 라이트에서도 무난히 보이므로 **변경하지 않는다** — 채도 있는 Tailwind 색상은 명도가 중간이라 밝은/어두운 배경 모두에서 허용 가능한 대비를 유지한다. `#c8962a`, `#eac266`, `#14b8a6` 같은 인라인 골드/틸 하드코딩 색상도 브랜드 포인트 컬러이므로 변경하지 않는다(이미 두 배경 모두에서 검증된 브랜드 색).

**예외 처리 대상(단순 치환표로 안 되는 곳):**
- `ShareCard`의 모달 패널(`index.html:539-542`): `bg-slate-900 border-slate-700`, `text-white`(닫기 버튼 옆 타이틀), `text-slate-400`(닫기 X) → 위 표 규칙 그대로 적용(패널 UI는 테마 대응, 내부 canvas 이미지는 대응 안 함). 배경 오버레이 `bg-black/80`(`index.html:538`)은 모달 뒷배경 딤 처리이므로 테마 무관하게 유지.
- `WaveChart`/`PourAnimation`: 아래 2.4 참조 (SVG는 Tailwind 클래스가 아니라 별도 처리)

### 2.4 SVG 컴포넌트: WaveChart, PourAnimation
Tailwind `dark:` 클래스가 적용되지 않는 raw SVG 속성(`stroke`, `fill`)이 있어, `theme` prop을 받아 JS에서 색상을 분기한다.

**`WaveChart`** — `HomeTab`에서 `<WaveChart bio={bio} lang={lang} />`로 호출 중(`index.html:694`); `theme` prop 추가 필요:
```js
const WaveChart = ({ bio, lang, theme }) => {
    const isLight = theme === 'light';
    const gridColor = isLight ? 'rgba(0,0,0,0.1)' : 'rgba(255,255,255,0.1)';
    const todayLineColor = isLight ? 'rgba(0,0,0,0.25)' : 'rgba(255,255,255,0.2)';
    const dotStroke = isLight ? '#f7f5f2' : '#1e293b';
    ...
    <line x1="0" y1={H/2} x2={W} y2={H/2} stroke={gridColor} strokeWidth="1"/>
    <line x1={x(14)} y1="0" x2={x(14)} y2={H} stroke={todayLineColor} strokeWidth="1.5" strokeDasharray="4,3"/>
    ...
    {Object.entries(cycles).map(([k,c]) => <circle key={`dot-${k}`} cx={x(14)} cy={y(c.v)} r="4.5" fill={c.color} stroke={dotStroke} strokeWidth="2"/>)}
```
바깥 wrapper의 `bg-black/40 border border-white/5`(`index.html:415`)는 2.3의 표 규칙대로 `bg-black/5 dark:bg-black/40 border-black/5 dark:border-white/5`로 변경(이미 Tailwind 클래스이므로 표로 처리 가능, `theme` prop과 무관).

**`PourAnimation`** — `CurateTab` cStep4에서 `<PourAnimation color={result.whiskey.color} />`로 호출(`index.html:824`); `theme` prop 추가 필요:
```js
const PourAnimation = ({ color, onComplete, theme }) => {
    const isLight = theme === 'light';
    const glassStroke = isLight ? 'rgba(0,0,0,0.15)' : 'rgba(255,255,255,0.2)';
    const glassGradStops = isLight
        ? [['0%','black',0.02], ['38%','black',0.08], ['100%','black',0.01]]
        : [['0%','white',0.04], ['38%','white',0.15], ['100%','white',0.02]];
    const rimHighlight = isLight ? 'rgba(0,0,0,0.06)' : 'rgba(255,255,255,0.1)';
    ...
```
- `glassBodyGrad`의 3개 `<stop>`을 `glassGradStops`로 매핑
- 유리잔 외곽선 `path ... stroke="rgba(255,255,255,0.2)"` (`index.html:477`) → `stroke={glassStroke}`
- 상단 림 하이라이트 `rect ... fill="rgba(255,255,255,0.1)"` (`index.html:488`) → `fill={rimHighlight}`
- 얼음(`iceG2` 그라디언트, `index.html:465`)과 액체 색(`color` prop 그대로 사용), 스플래시 효과는 두 테마에서 공통으로 잘 보이므로 변경 없음

### 2.5 ShareCard 캔버스 — 변경 없음
`ShareCard`의 `useEffect` 내 `canvas` 드로잉 로직(`index.html:500-528`)은 **완전히 그대로 유지**한다. 항상 다크 브랜드 톤(`#1e293b`→`#080604` 그라디언트, 흰 텍스트)으로 렌더링 — 라이트 모드에서 공유해도 이미지 자체는 변하지 않는다. `theme` prop을 `ShareCard`/canvas 로직에 전달하지 않는다.

### 2.6 Prop Drilling 경로
- `App` → `HomeTab`에 `theme` 전달 → `HomeTab` → `WaveChart`에 `theme` 전달
- `App` → `CurateTab`에 `theme` 전달 → `CurateTab` cStep4에서 `<PourAnimation color={...} theme={theme} />`
- `App` → `ProfileTab`에 `theme`, `setTheme` 전달 (토글 버튼용)
- 그 외 컴포넌트(Splash, Onboarding, Quiz, DirectSelect)는 `theme` prop 불필요 — 오직 Tailwind `dark:` 클래스만으로 대응(2.3)

### 2.7 ProfileTab 테마 토글 UI
기존 언어 변경 버튼 바로 아래(또는 위, 리셋 버튼 위)에 추가:
```jsx
<button
    onClick={() => setTheme(t => t === 'dark' ? 'light' : 'dark')}
    className="w-full py-5 rounded-[20px] bg-slate-100 dark:bg-slate-800 text-slate-900 dark:text-white font-bold text-lg border-2 border-slate-200 dark:border-slate-700 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors">
    {theme === 'dark' ? (lang === 'en' ? 'Switch to Light Mode' : '라이트 모드로 변경') : (lang === 'en' ? 'Switch to Dark Mode' : '다크 모드로 변경')}
</button>
```
버튼 순서: 언어 변경 → 테마 변경 → 프로필 초기화(기존 순서 유지, 테마 버튼만 삽입).

---

## 3. 무드 12개로 확장

`MOODS` 배열(`index.html:272-283`, 현재 10개)에 다음 2개 추가:

```js
{ id: 'nostalgia', label: { ko: '그리움', en: 'Nostalgia' }, desc: { ko: "지난 추억이 그리워지는 날이에요. 옛 생각에 잠겨 한 잔 어떠세요?", en: "A day of missing the past. A drink to reminisce?" }, expr: { ko: "옛 생각이 자꾸 나서 마음이 아련해지는 하루", en: "A wistful day full of old memories" }, emoji: "🍂", conflicts: ['anticipation', 'surprise'], flavorProfile: ['oak', 'sherry', 'rich', 'classic', 'nutty'] },
{ id: 'pride', label: { ko: '뿌듯함', en: 'Pride' }, desc: { ko: "스스로가 뿌듯한 날이에요. 자축하는 의미로 한 잔 어떠세요?", en: "A day of self-pride. A drink to celebrate yourself?" }, expr: { ko: "해낸 일에 스스로가 대견하고 뿌듯한 하루", en: "A day I'm proud of what I accomplished" }, emoji: "😎", conflicts: ['sadness', 'fear', 'disgust'], flavorProfile: ['bold', 'oak', 'cask_strength', 'rich', 'complex'] }
```

`flavorProfile` 태그(`oak`, `sherry`, `rich`, `classic`, `nutty`, `bold`, `cask_strength`, `complex`)는 모두 `SPIRITS_DB`에 이미 존재하는 태그이므로 큐레이션 스코어링 로직(`CurateTab.run`)이 그대로 작동한다. `conflicts`는 감정적으로 상충하는 기존 무드 id만 참조하며, 다른 무드의 `conflicts` 배열은 수정하지 않는다(단방향 충돌 체크로 충분 — 기존 `surprise`/`boredom` 등도 비대칭 참조 존재, 기존 패턴과 일관됨).

`CurateTab`의 무드 그리드(`grid grid-cols-2 lg:grid-cols-4`, `index.html:756`)는 12개 항목에서 모바일 6행×2열, 데스크톱 3행×4열로 자연스럽게 배치되므로 그리드 클래스 변경 불필요.

---

## 테스트 계획
- 자동화 테스트 러너 없음(순수 정적 HTML). 브라우저 프리뷰 도구로 검증:
  - 언어 버튼이 온보딩/데스크톱 네비에서 사라지고 프로필에만 남았는지 확인
  - 시스템이 라이트 선호일 때 첫 로드가 라이트로 뜨는지(`prefers-color-scheme` 에뮬레이션), 프로필에서 토글 시 전체 화면(스플래시 제외 — 이미 지나간 화면) 색상이 즉시 바뀌는지, 새로고침 후에도 유지되는지
  - 모든 화면(온보딩/퀴즈/다이렉트선택/홈/큐레이션 각 단계/결과화면/프로필)을 라이트·다크 각각에서 스크린샷으로 대비 확인 — 텍스트가 배경에 묻히는 곳이 없는지
  - `WaveChart`, `PourAnimation`이 라이트 모드에서도 선이/유리잔 형태가 보이는지(투명 배경에 묻히지 않는지)
  - 공유 카드 캔버스가 라이트 모드에서 열어도 여전히 다크 톤으로 렌더링되는지
  - 큐레이션 1단계에 무드 카드가 12개 모두 표시되고, 새 무드 선택 시 결과 스코어링이 에러 없이 동작하는지(그리움/뿌듯함 선택 후 결과 도출까지)
  - 데스크톱(이전 작업) 레이아웃에 회귀 없는지 375/1280px 재확인
