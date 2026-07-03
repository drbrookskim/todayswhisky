# 데스크톱 레이아웃 지원 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `index.html`(단일 파일, CDN React + Babel-standalone + Tailwind CDN)에 데스크톱(≥1024px) 레이아웃을 반응형으로 추가한다. 모바일(<1024px) 시각/동작은 완전히 그대로 유지한다.

**Architecture:** Tailwind의 `lg:` (min-width: 1024px) 브레이크포인트와 `<style>` 블록 안의 `@media (min-width: 1024px)` 규칙으로 CSS만 오버라이드한다. React 컴포넌트 트리, 상태(`useState`), 로직 함수(`calculateBiorhythm`, `getBlendedMood`, `getWhiskyDetails` 등)는 전혀 변경하지 않는다. 유일한 JSX 변경은 (a) `App`에 데스크톱 상단 네비바 마크업 추가, (b) `HomeTab`/`CurateTab`/`ProfileTab`의 카드 wrapper `className`에 `lg:grid lg:grid-cols-N` 계열 클래스 추가.

**Tech Stack:** React 18 (UMD, CDN) / Babel standalone (인브라우저 JSX 트랜스파일) / Tailwind CDN (`tailwind.config = { darkMode: 'class' }`) / 순수 CSS (`<style>` 블록). 빌드 도구/패키지 매니저 없음, 자동화 테스트 프레임워크 없음.

## Global Constraints

- 브레이크포인트는 정확히 `1024px` (Tailwind `lg:` 및 `@media (min-width: 1024px)`)를 사용한다. 다른 값을 쓰지 않는다.
- 1024px 미만에서는 기존 클래스/스타일/마크업이 하나도 바뀌지 않아야 한다 (신규 스타일은 전부 `lg:` prefix 또는 `@media (min-width: 1024px)` 안에만 존재).
- React 상태, useEffect, 데이터(`TEXT`, `SPIRITS_DB`, `MBTI_*`, `MOODS`, `PRICE_RANGES`), 분석 로직 함수는 수정하지 않는다.
- `Splash`, `Onboarding`, `Quiz`, `DirectSelect` 컴포넌트의 JSX는 수정하지 않는다 (좁은 카드형 그대로 유지).
- 이 프로젝트에는 자동화 테스트 러너가 없다. 각 태스크의 "테스트"는 Claude Code의 `preview_*` 도구(브라우저 프리뷰)로 뷰포트를 리사이즈하고 `preview_inspect`/`preview_snapshot`으로 실제 DOM/computed style을 확인하는 방식으로 대체한다. 각 스텝은 정확한 selector와 기대값을 명시한다.
- 커밋은 각 태스크 완료 시 1회, 메시지는 한국어 커밋 컨벤션(`feat:`, `fix:` 등 기존 로그 스타일)을 따른다.

---

### Task 1: 데스크톱 네비/컨테이너 CSS 기반 작업

**Files:**
- Modify: `index.html` (`<style>` 블록, `.app-container`/`.bottom-nav`/`.content-area` 규칙 근처)

**Interfaces:**
- Consumes: 없음 (순수 CSS 추가)
- Produces: 이후 태스크에서 마크업에 사용할 CSS 클래스 — `.desktop-nav`, `.desktop-nav-brand`, `.desktop-nav .nav-menu`, `.desktop-nav .nav-menu-item`, `.desktop-nav .nav-menu-item.active`, `.desktop-nav-lang`. `.content-area`는 `@media (min-width:1024px)`에서 `max-width: 1080px; margin: 0 auto; padding: 0 48px 64px 48px;`로 오버라이드됨. `.bottom-nav`는 같은 미디어 쿼리에서 `display: none`.

- [ ] **Step 1: `.content-area` 규칙 뒤에 데스크톱 오버라이드 CSS 추가**

`index.html`에서 다음 블록을 찾는다 (변경 전, `.content-area` 규칙 바로 다음, `.glass-panel` 규칙 앞):

```css
        .content-area {
            flex: 1;
            padding: 0 24px 40px 24px;
            overflow-y: auto;
            scroll-padding-top: calc(env(safe-area-inset-top, 48px) + 24px);
            display: flex;
            flex-direction: column;
        }

        .glass-panel {
```

다음과 같이 바꾼다 (기존 두 규칙은 그대로 두고 사이에 새 블록 삽입):

```css
        .content-area {
            flex: 1;
            padding: 0 24px 40px 24px;
            overflow-y: auto;
            scroll-padding-top: calc(env(safe-area-inset-top, 48px) + 24px);
            display: flex;
            flex-direction: column;
        }

        /* --- Desktop Layout (>=1024px) --- */
        @media (min-width: 1024px) {
            .app-container {
                max-width: 100%;
                box-shadow: none;
                background-color: transparent;
            }

            .bottom-nav {
                display: none;
            }

            .content-area {
                max-width: 1080px;
                margin: 0 auto;
                width: 100%;
                padding: 0 48px 64px 48px;
            }

            .desktop-nav {
                display: flex;
                align-items: center;
                justify-content: space-between;
                max-width: 1080px;
                margin: 0 auto;
                width: 100%;
                padding: 28px 48px 0 48px;
                flex-shrink: 0;
            }

            .desktop-nav-brand {
                display: flex;
                align-items: center;
                gap: 10px;
            }

            .desktop-nav .nav-menu {
                display: flex;
                gap: 6px;
                background: var(--card);
                border: 1px solid var(--border);
                border-radius: 999px;
                padding: 6px;
            }

            .desktop-nav .nav-menu-item {
                display: flex;
                align-items: center;
                gap: 8px;
                padding: 10px 20px;
                border-radius: 999px;
                color: var(--text-dim);
                font-weight: 700;
                font-size: 14px;
                background: transparent;
                border: none;
                cursor: pointer;
                transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            }

            .desktop-nav .nav-menu-item.active {
                color: #14b8a6;
                background: rgba(20, 184, 166, 0.12);
            }

            .desktop-nav-lang {
                padding: 8px 18px;
                border-radius: 999px;
                border: 1px solid var(--border);
                background: transparent;
                color: var(--text-dim);
                font-weight: 700;
                font-size: 13px;
                cursor: pointer;
                transition: all 0.2s ease;
            }

            .desktop-nav-lang:hover {
                color: var(--text);
                border-color: var(--accent);
            }
        }

        .desktop-nav {
            display: none;
        }

        .glass-panel {
```

- [ ] **Step 2: 프리뷰 서버 기동 확인**

`.claude/launch.json`이 없으면 다음 내용으로 만든다 (정적 파일이므로 아무 임의 정적 서버 사용 — `npx`가 있는 환경이면 `serve` 사용):

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "whisky-static",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["--yes", "serve", "-l", "5173", "."],
      "port": 5173
    }
  ]
}
```

`preview_start`로 `whisky-static` 서버를 시작한다.

- [ ] **Step 3: 데스크톱 폭에서 CSS만 적용됐는지 확인 (마크업은 아직 없음)**

`preview_resize`로 뷰포트를 `1280x800`으로 맞추고 페이지를 로드한 뒤, `preview_inspect`로 `.bottom-nav`를 조회한다.

기대값: `display: none` (computed style)

- [ ] **Step 4: 모바일 폭에서 회귀 없는지 확인**

`preview_resize`로 `375x812`로 맞추고 `preview_inspect`로 `.app-container`를 조회한다.

기대값: `max-width: 430px`, `box-shadow`가 `none`이 아님 (예: `rgba(0, 0, 0, 0.8) 0px 0px 40px 0px`)

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: 데스크톱 레이아웃용 CSS 기반 작업 추가"
```

---

### Task 2: 데스크톱 상단 네비바 마크업 추가

**Files:**
- Modify: `index.html` (`App` 컴포넌트, `appState === 'main'` 분기)

**Interfaces:**
- Consumes: Task 1의 `.desktop-nav`, `.desktop-nav-brand`, `.nav-menu`, `.nav-menu-item`, `.desktop-nav-lang` CSS 클래스. `App` 내부의 기존 `activeTab`, `setActiveTab`, `lang`, `setLang`, `TEXT[lang]`.
- Produces: 데스크톱 폭에서 보이는 상단 네비 마크업 (다른 태스크에 의존성 없음).

- [ ] **Step 1: `App`의 `appState === 'main'` 블록 앞에 데스크톱 네비 추가**

`index.html`에서 다음 블록을 찾는다:

```jsx
                    {appState === 'main' && (
                        <>
                            <div className="content-area hide-scroll">
                                {activeTab === 'home' && <HomeTab mbti={mbti} lang={lang} />}
                                {activeTab === 'curate' && <CurateTab mbti={mbti} lang={lang} bio={calculateBiorhythm(mbti)} />}
                                {activeTab === 'profile' && <ProfileTab mbti={mbti} lang={lang} onReset={handleReset} setLang={setLang} />}
                            </div>
```

다음으로 바꾼다:

```jsx
                    {appState === 'main' && (
                        <>
                            <nav className="desktop-nav">
                                <div className="desktop-nav-brand">
                                    <span className="text-xl">🥃</span>
                                    <span className="playfair font-bold text-lg text-white">{TEXT[lang].title}</span>
                                </div>
                                <div className="nav-menu">
                                    <button className={`nav-menu-item ${activeTab === 'home' ? 'active' : ''}`} onClick={() => setActiveTab('home')}>
                                        <Icon name="home" className="w-4 h-4" />
                                        <span>{TEXT[lang].tab_home}</span>
                                    </button>
                                    <button className={`nav-menu-item ${activeTab === 'curate' ? 'active' : ''}`} onClick={() => setActiveTab('curate')}>
                                        <Icon name="glass-water" className="w-4 h-4" />
                                        <span>{TEXT[lang].tab_curate}</span>
                                    </button>
                                    <button className={`nav-menu-item ${activeTab === 'profile' ? 'active' : ''}`} onClick={() => setActiveTab('profile')}>
                                        <Icon name="user" className="w-4 h-4" />
                                        <span>{TEXT[lang].tab_profile}</span>
                                    </button>
                                </div>
                                <button className="desktop-nav-lang" onClick={() => setLang(l => l === 'ko' ? 'en' : 'ko')}>
                                    {lang === 'en' ? 'KO' : 'EN'}
                                </button>
                            </nav>

                            <div className="content-area hide-scroll">
                                {activeTab === 'home' && <HomeTab mbti={mbti} lang={lang} />}
                                {activeTab === 'curate' && <CurateTab mbti={mbti} lang={lang} bio={calculateBiorhythm(mbti)} />}
                                {activeTab === 'profile' && <ProfileTab mbti={mbti} lang={lang} onReset={handleReset} setLang={setLang} />}
                            </div>
```

- [ ] **Step 2: 데스크톱 폭에서 네비바가 보이고 탭 전환이 되는지 확인**

`preview_resize`를 `1280x800`으로 설정. 온보딩을 스킵하기 위해 `preview_eval`로 다음을 실행한 뒤 새로고침한다:

```js
localStorage.setItem('whiskey_mbti', 'INTJ'); location.reload();
```

`preview_snapshot`으로 `desktop-nav` 안에 "홈", "큐레이션", "프로필" 텍스트가 보이는지 확인한다.

`preview_click`으로 큐레이션 메뉴 버튼(`.nav-menu-item` 중 두 번째)을 클릭한 뒤 `preview_snapshot`으로 큐레이션 화면(`step2_title` 텍스트 = "오늘 하루를 한 문장으로 표현한다면?")이 렌더링됐는지 확인한다.

- [ ] **Step 3: 모바일 폭에서 데스크톱 네비가 안 보이는지 확인**

`preview_resize`를 `375x812`로 설정하고 `preview_inspect`로 `.desktop-nav`의 `display`가 `none`인지 확인한다. 기존 `.bottom-nav`(하단 탭바)는 `preview_snapshot`으로 정상 노출 확인한다.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: 데스크톱 상단 네비바 추가"
```

---

### Task 3: HomeTab 데스크톱 그리드 재배치

**Files:**
- Modify: `index.html` (`HomeTab` 컴포넌트)

**Interfaces:**
- Consumes: 없음 (className만 변경, props/로직 그대로)
- Produces: 없음 (leaf 변경)

- [ ] **Step 1: 바이오리듬 카드와 데일리 픽 카드를 감싸는 grid wrapper 추가**

다음 블록을 찾는다:

```jsx
                    <GlassCard className="p-6">
                        <div className="flex justify-between items-center mb-6 px-1">
                            <h3 className="font-bold text-white flex items-center gap-2 text-lg"><Icon name="activity" className="w-5 h-5 text-[#c8962a]"/> {TEXT[lang].result_bio}</h3>
                        </div>
                        <WaveChart bio={bio} lang={lang} />
                    </GlassCard>

                    <div className="pb-4">
                        <h3 className="font-bold text-slate-300 mb-4 px-2 text-lg">{lang==='en'?"Today's Daily Pick":"오늘의 데일리 픽"}</h3>
                        <GlassCard className="p-5 flex gap-5 items-center">
                            <div className="w-20 h-24 rounded-2xl flex items-center justify-center shrink-0 shadow-inner" style={{ background: `linear-gradient(135deg, ${dailyPick.color}90, ${dailyPick.color}50)` }}>
                                <Icon name="glass-water" className="w-10 h-10 text-white/90" />
                            </div>
                            <div className="flex-1">
                                <p className="text-sm text-[#c8962a] font-bold mb-1.5 uppercase tracking-wide">{dailyPick.style} · {dailyPick.abv}%</p>
                                <h4 className="text-[1.35rem] font-bold text-white mb-2 leading-tight">{lang==='ko'?dailyPick.name:dailyPick.nameEn}</h4>
                                <p className="text-sm text-slate-400 line-clamp-2 leading-relaxed">{lang==='ko'?dailyPick.desc:dailyPick.descEn}</p>
                            </div>
                        </GlassCard>
                    </div>
```

다음으로 바꾼다 (두 블록을 `lg:grid-cols-2` wrapper로 감싼다):

```jsx
                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 lg:gap-8 lg:items-start">
                        <GlassCard className="p-6">
                            <div className="flex justify-between items-center mb-6 px-1">
                                <h3 className="font-bold text-white flex items-center gap-2 text-lg"><Icon name="activity" className="w-5 h-5 text-[#c8962a]"/> {TEXT[lang].result_bio}</h3>
                            </div>
                            <WaveChart bio={bio} lang={lang} />
                        </GlassCard>

                        <div className="pb-4">
                            <h3 className="font-bold text-slate-300 mb-4 px-2 text-lg">{lang==='en'?"Today's Daily Pick":"오늘의 데일리 픽"}</h3>
                            <GlassCard className="p-5 flex gap-5 items-center">
                                <div className="w-20 h-24 rounded-2xl flex items-center justify-center shrink-0 shadow-inner" style={{ background: `linear-gradient(135deg, ${dailyPick.color}90, ${dailyPick.color}50)` }}>
                                    <Icon name="glass-water" className="w-10 h-10 text-white/90" />
                                </div>
                                <div className="flex-1">
                                    <p className="text-sm text-[#c8962a] font-bold mb-1.5 uppercase tracking-wide">{dailyPick.style} · {dailyPick.abv}%</p>
                                    <h4 className="text-[1.35rem] font-bold text-white mb-2 leading-tight">{lang==='ko'?dailyPick.name:dailyPick.nameEn}</h4>
                                    <p className="text-sm text-slate-400 line-clamp-2 leading-relaxed">{lang==='ko'?dailyPick.desc:dailyPick.descEn}</p>
                                </div>
                            </GlassCard>
                        </div>
                    </div>
```

- [ ] **Step 2: 데스크톱 폭에서 2열 배치 확인**

`preview_resize`를 `1280x800`으로, `localStorage.setItem('whiskey_mbti','INTJ')` 후 새로고침, 홈 탭에서 `preview_inspect`로 바이오리듬 `GlassCard`와 데일리 픽 `div.pb-4`의 `bounding box`를 각각 조회해 두 요소의 `y` 좌표가 거의 같고(같은 행), `x` 좌표가 다른지(가로 배치) 확인한다.

- [ ] **Step 3: 모바일 폭에서 세로 스택 유지 확인**

`preview_resize`를 `375x812`로 맞추고 같은 두 요소를 `preview_inspect`로 조회해 데일리 픽 카드의 `y` 좌표가 바이오리듬 카드의 `y` + `height`보다 큰지(세로로 쌓여 있는지) 확인한다.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: HomeTab 데스크톱 2열 그리드 적용"
```

---

### Task 4: CurateTab 무드 선택 단계 데스크톱 그리드 재배치

**Files:**
- Modify: `index.html` (`CurateTab` 컴포넌트, `cStep === 1` 분기)

**Interfaces:**
- Consumes: 없음
- Produces: 없음

- [ ] **Step 1: 무드 카드 grid에 `lg:grid-cols-4` 추가**

다음 줄을 찾는다:

```jsx
                    <div className="grid grid-cols-2 gap-3 pb-6 flex-1">
                        {MOODS.map(m => (
```

다음으로 바꾼다:

```jsx
                    <div className="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-4 pb-6 flex-1">
                        {MOODS.map(m => (
```

- [ ] **Step 2: 데스크톱 폭에서 4열 배치 확인**

`1280x800`으로 리사이즈, `localStorage.setItem('whiskey_mbti','INTJ')` 후 새로고침하고 큐레이션 탭으로 이동(`preview_click`으로 상단 네비 큐레이션 버튼). `preview_inspect`로 첫 번째와 다섯 번째 무드 버튼(`MOODS` 배열의 1번째, 5번째 카드)의 `y` 좌표가 같은지 확인한다 (4열이면 5번째 카드는 2번째 행 시작 = 1번째 카드와 `y`가 다름을 기대. 대신 1번째와 4번째가 같은 행이어야 함 — 4번째와 1번째의 `y`가 같은지 확인).

- [ ] **Step 3: 모바일 폭에서 기존 2열 유지 확인**

`375x812`로 리사이즈, 같은 화면에서 `preview_inspect`로 1번째·3번째 카드의 `y`가 같은지(2열 유지) 확인한다.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: CurateTab 무드 선택 데스크톱 4열 그리드 적용"
```

---

### Task 5: CurateTab 가격대/결과 화면 데스크톱 재배치

**Files:**
- Modify: `index.html` (`CurateTab` 컴포넌트, `cStep === 3`과 `cStep === 4` 분기)

**Interfaces:**
- Consumes: 없음
- Produces: 없음

- [ ] **Step 1: 가격대 선택 grid에 `lg:grid-cols-3` 추가**

다음 줄을 찾는다:

```jsx
                    <div className="grid gap-4 flex-1">
                        {PRICE_RANGES.map(r => (
```

다음으로 바꾼다:

```jsx
                    <div className="grid gap-4 lg:grid-cols-3 flex-1">
                        {PRICE_RANGES.map(r => (
```

- [ ] **Step 2: 결과 화면의 Tasting Notes / Flavor Profile 카드를 나란히 배치**

다음 블록을 찾는다:

```jsx
                    <GlassCard className="p-8">
                        <h3 className="text-sm font-bold text-[#c8962a] mb-4 uppercase tracking-widest flex items-center gap-2"><Icon name="book-open" className="w-5 h-5"/> Tasting Notes</h3>
                        <p className="text-base text-slate-200 leading-relaxed font-medium">{lang==='ko'?result.whiskey.desc:result.whiskey.descEn}</p>
                    </GlassCard>
                    <GlassCard className="p-8">
                        <h3 className="text-sm font-bold text-[#c8962a] mb-6 uppercase tracking-widest flex items-center gap-2"><Icon name="bar-chart-2" className="w-5 h-5"/> Flavor Profile</h3>
                        <FlavorBars flavor={result.whiskey.flavor} lang={lang} />
                    </GlassCard>
```

다음으로 바꾼다:

```jsx
                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 lg:gap-8 lg:items-start">
                        <GlassCard className="p-8">
                            <h3 className="text-sm font-bold text-[#c8962a] mb-4 uppercase tracking-widest flex items-center gap-2"><Icon name="book-open" className="w-5 h-5"/> Tasting Notes</h3>
                            <p className="text-base text-slate-200 leading-relaxed font-medium">{lang==='ko'?result.whiskey.desc:result.whiskey.descEn}</p>
                        </GlassCard>
                        <GlassCard className="p-8">
                            <h3 className="text-sm font-bold text-[#c8962a] mb-6 uppercase tracking-widest flex items-center gap-2"><Icon name="bar-chart-2" className="w-5 h-5"/> Flavor Profile</h3>
                            <FlavorBars flavor={result.whiskey.flavor} lang={lang} />
                        </GlassCard>
                    </div>
```

- [ ] **Step 3: 데스크톱 폭에서 가격대 3열, 결과 카드 2열 확인**

`1280x800`, 큐레이션 탭 진입 → 무드 1개 클릭 → 다음 단계로 두 번 진행(뒤로가기 버튼 텍스트는 "뒤로", 다음 버튼은 "계속하기") → 가격대 화면에서 `preview_inspect`로 1번째·2번째·3번째 `PRICE_RANGES` 버튼의 `y`가 모두 같은지 확인. 가격대 선택 후 "찾아보기"(`run_btn`) 클릭 → 로딩(2초) 대기 → 결과 화면에서 Tasting Notes/Flavor Profile 두 `GlassCard`의 `y`가 같은지 확인.

- [ ] **Step 4: 모바일 폭에서 세로 스택 유지 확인**

`375x812`로 리사이즈, 같은 플로우를 반복해 가격대 카드들이 세로로 쌓이고(각 `y`가 다름), 결과 화면의 두 카드도 세로로 쌓이는지 확인한다.

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: CurateTab 가격대/결과 화면 데스크톱 그리드 적용"
```

---

### Task 6: ProfileTab 데스크톱 2열 배치

**Files:**
- Modify: `index.html` (`ProfileTab` 컴포넌트)

**Interfaces:**
- Consumes: 없음
- Produces: 없음

- [ ] **Step 1: 프로필 카드와 설정 버튼 그룹을 2열로 감싼다**

다음 블록을 찾는다:

```jsx
                    <GlassCard className="p-10 flex flex-col items-center text-center">
                        <div className="w-24 h-24 bg-slate-800 rounded-full flex items-center justify-center mb-6 border-2 border-slate-700 text-4xl">
                            👤
                        </div>
                        <h3 className={`text-4xl font-black mb-2 ${group?.color || 'text-white'}`}>{mbti}</h3>
                        <p className="text-sm font-bold uppercase tracking-widest text-slate-400">
                            {group ? group.label[lang] : 'Unknown'}
                        </p>
                    </GlassCard>

                    <div className="flex flex-col gap-4 mt-auto pb-4">
                        <button 
                            onClick={() => setLang(l => l === 'ko' ? 'en' : 'ko')} 
                            className="w-full py-5 rounded-[20px] bg-slate-800 text-white font-bold text-lg border-2 border-slate-700 hover:bg-slate-700 transition-colors">
                            {lang === 'en' ? '한국어로 변경' : 'Change to English'}
                        </button>
                        
                        <button 
                            onClick={onReset} 
                            className="w-full py-5 rounded-[20px] bg-red-900/20 text-red-400 font-bold text-lg border-2 border-red-900/30 hover:bg-red-900/40 transition-colors">
                            {lang === 'en' ? 'Reset MBTI Profile' : '프로필 초기화 (다시하기)'}
                        </button>
                    </div>
```

다음으로 바꾼다:

```jsx
                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 lg:gap-8 lg:items-start flex-1">
                        <GlassCard className="p-10 flex flex-col items-center text-center">
                            <div className="w-24 h-24 bg-slate-800 rounded-full flex items-center justify-center mb-6 border-2 border-slate-700 text-4xl">
                                👤
                            </div>
                            <h3 className={`text-4xl font-black mb-2 ${group?.color || 'text-white'}`}>{mbti}</h3>
                            <p className="text-sm font-bold uppercase tracking-widest text-slate-400">
                                {group ? group.label[lang] : 'Unknown'}
                            </p>
                        </GlassCard>

                        <div className="flex flex-col gap-4">
                            <button 
                                onClick={() => setLang(l => l === 'ko' ? 'en' : 'ko')} 
                                className="w-full py-5 rounded-[20px] bg-slate-800 text-white font-bold text-lg border-2 border-slate-700 hover:bg-slate-700 transition-colors">
                                {lang === 'en' ? '한국어로 변경' : 'Change to English'}
                            </button>
                            
                            <button 
                                onClick={onReset} 
                                className="w-full py-5 rounded-[20px] bg-red-900/20 text-red-400 font-bold text-lg border-2 border-red-900/30 hover:bg-red-900/40 transition-colors">
                                {lang === 'en' ? 'Reset MBTI Profile' : '프로필 초기화 (다시하기)'}
                            </button>
                        </div>
                    </div>
```

- [ ] **Step 2: 데스크톱 폭에서 2열 배치 확인**

`1280x800`, 상단 네비에서 프로필 탭 클릭. `preview_inspect`로 프로필 `GlassCard`와 설정 버튼 그룹(`div.flex.flex-col.gap-4`)의 `y`가 같고 `x`가 다른지 확인.

- [ ] **Step 3: 모바일 폭에서 세로 스택 + `mt-auto` 효과 유지 확인**

`375x812`로 리사이즈 후 `preview_snapshot`으로 프로필 화면이 기존과 동일하게(카드 위, 버튼 아래쪽에 붙어) 보이는지 확인한다.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: ProfileTab 데스크톱 2열 그리드 적용"
```

---

### Task 7: 전체 플로우 회귀 확인 (다중 브레이크포인트)

**Files:**
- 변경 없음 (검증 전용 태스크)

**Interfaces:**
- Consumes: Task 1~6의 모든 변경사항
- Produces: 없음

- [ ] **Step 1: 온보딩부터 결과까지 데스크톱 폭(1280x800)에서 전체 플로우 실행**

`preview_eval`로 `localStorage.clear(); location.reload();` 실행 (초기 상태로 리셋). `preview_resize`를 `1280x800`으로 설정.

- 스플래시 → 온보딩 화면이 좁은 카드형으로 화면 중앙에 뜨는지 `preview_screenshot`으로 확인
- "이미 내 MBTI를 알고 있어요"(`skip_quiz`) 클릭 → `DirectSelect` 화면도 좁은 카드형 유지 확인
- MBTI 하나 선택해 완료 → `main` 상태 진입, 상단 데스크톱 네비바 노출 확인
- 홈 → 큐레이션(무드 1개 → 계속하기 → 계속하기 → 가격대 선택 → 찾아보기 → 로딩 2초 → 결과) → 프로필까지 전체 탭 순회하며 `preview_console_logs`로 콘솔 에러 없는지 확인

- [ ] **Step 2: 태블릿 폭(768x1024)에서 모바일 레이아웃 유지 확인**

`preview_resize`를 `768x1024`로 설정. `preview_inspect`로 `.desktop-nav`가 `display: none`이고 `.bottom-nav`가 보이는지, `.app-container`가 `max-width: 430px`인지 확인 (스펙상 태블릿은 모바일 레이아웃을 그대로 사용).

- [ ] **Step 3: 초대형 데스크톱 폭(1440x900)에서 콘텐츠가 max-width를 넘지 않는지 확인**

`preview_resize`를 `1440x900`으로 설정. `preview_inspect`로 `.content-area`의 `width`가 `1080px` 이하인지 확인 (좌우 여백이 생겨야 함).

- [ ] **Step 4: 모바일 폭(375x812)에서 최초 목표대로 전혀 회귀 없는지 스크린샷 비교**

`preview_resize`를 `375x812`로 설정하고 `preview_screenshot`을 찍어, git 저장소의 기존 커밋(`bf0a1f4`) 시점 UI와 시각적으로 동일한지(레이아웃, 네비, 카드 배치) 육안 확인한다.

- [ ] **Step 5: 최종 커밋 (문서화 변경이 있다면)**

이 태스크는 코드 변경이 없으므로 커밋 없이 종료한다. 문제가 발견되면 해당 태스크로 돌아가 수정 후 그 태스크의 커밋을 새로 추가한다.
