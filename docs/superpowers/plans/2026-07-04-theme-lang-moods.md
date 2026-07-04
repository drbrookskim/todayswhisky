# 테마/언어통합/무드확장 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `index.html`에 실제 라이트/다크 테마 토글, 언어 변경 프로필 통합, 무드 12개 확장을 구현한다.

**Architecture:** CSS 변수를 라이트 기본값 + `html.dark` 오버라이드로 반전시키고, Tailwind `dark:` variant를 컴포넌트별 하드코딩 클래스에 기계적으로 추가한다. SVG 컴포넌트(WaveChart, PourAnimation)는 `theme` prop을 받아 JS에서 색상을 분기한다. 공유 카드 캔버스는 변경하지 않는다. 언어 토글은 프로필에만 남기고 나머지 두 곳에서 제거한다. MOODS 배열에 2개 항목을 추가한다.

**Tech Stack:** React 18 (UMD, CDN) / Babel standalone / Tailwind CDN (`darkMode: 'class'`). 빌드/테스트 도구 없음 — 검증은 `preview_*` 브라우저 도구로 실행.

## Global Constraints

- 다크 모드는 `html.dark` 클래스로 트리거된다(Tailwind `darkMode:'class'` 표준). 라이트가 기본(`:root`), 다크가 오버라이드(`html.dark`).
- 초기 테마: `localStorage.whiskey_theme`가 있으면 그 값, 없으면 `window.matchMedia('(prefers-color-scheme: light)').matches` → 'light', 아니면 'dark'.
- 언어 변경 UI는 `ProfileTab`에만 존재해야 한다. `Onboarding`, 데스크톱 상단 네비에서 완전히 제거한다.
- 공유 카드(`ShareCard`)의 `<canvas>` 드로잉 로직은 테마와 무관하게 항상 다크 톤 그대로 — 수정하지 않는다.
- `#c8962a`, `#eac266`, `#14b8a6` 등 인라인 하드코딩 브랜드 색상(골드/틸 포인트 컬러)과 MBTI 그룹/리셋 버튼의 채도 있는 Tailwind 색상(`text-purple-400`, `bg-red-900/20` 등)은 변경하지 않는다.
- MBTI 분석/위스키 스코어링 로직, `useState`/`useEffect` 구조는 이 작업의 범위가 아니면 변경하지 않는다.
- 이 프로젝트는 자동화 테스트 러너가 없다. 각 태스크의 "테스트"는 Claude Code의 `preview_*` 브라우저 도구로 실제 렌더링을 확인하는 방식으로 대체한다. 서브에이전트는 `preview_*` MCP 도구에 접근할 수 없으므로(이전 작업에서 확인됨), 브라우저 검증은 컨트롤러(세션)가 직접 수행한다 — 각 태스크의 구현 담당 서브에이전트는 구조적 자가검토만 하고 "브라우저 검증은 컨트롤러에게 위임"이라고 보고한다.

---

### Task 1: CSS 변수 반전 (라이트 기본 + 다크 오버라이드)

**Files:**
- Modify: `index.html` (`<style>` 블록, `:root` 및 `body`/`.app-container`/`.bottom-nav` 규칙, 대략 22-81번째 줄)

**Interfaces:**
- Consumes: 없음
- Produces: `--bg`, `--card`, `--border`, `--accent`, `--accent-dim`, `--text`, `--text-dim`, `--nav-bg`, `--shell-shadow`, `--body-bg-solid`, `--body-bg-image` CSS 변수. `html.dark`가 있으면 다크 값, 없으면 라이트 값. 이후 태스크들은 `document.documentElement.classList`에 `dark`를 추가/제거하는 방식으로 이 변수들을 제어한다.

- [ ] **Step 1: `:root` 블록을 라이트 기본값으로 교체하고 `html.dark` 오버라이드 추가**

`index.html`에서 다음 블록을 찾는다:

```css
        :root {
            --bg: #080604;
            --card: rgba(255, 255, 255, 0.035);
            --border: rgba(255, 255, 255, 0.08);
            --accent: #c8962a;
            --accent-dim: rgba(200, 150, 42, 0.15);
            --text: #f8f8f8;
            --text-dim: #888888;
        }

        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, 'Helvetica Neue', 'Segoe UI', 'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic', 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', sans-serif;
            background-color: #0f172a;
            color: var(--text);
            background-image: radial-gradient(ellipse at 50% -10%, rgba(13, 148, 136, 0.15) 0%, #080604 70%);
            min-height: 100vh;
            overflow-x: hidden;
            -webkit-tap-highlight-color: transparent;
            overscroll-behavior-y: none;
            margin: 0;
        }
```

다음으로 바꾼다:

```css
        :root {
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

        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, 'Helvetica Neue', 'Segoe UI', 'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic', 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', sans-serif;
            background-color: var(--body-bg-solid);
            color: var(--text);
            background-image: var(--body-bg-image);
            min-height: 100vh;
            overflow-x: hidden;
            -webkit-tap-highlight-color: transparent;
            overscroll-behavior-y: none;
            margin: 0;
        }
```

- [ ] **Step 2: `.app-container`와 `.bottom-nav`의 하드코딩 값을 변수로 교체**

다음 블록을 찾는다:

```css
        .app-container {
            width: 100%;
            max-width: 430px;
            margin: 0 auto;
            height: 100dvh;
            display: flex;
            flex-direction: column;
            position: relative;
            background-color: var(--bg);
            box-shadow: 0 0 40px rgba(0,0,0,0.8);
            overflow: hidden;
        }

        .bottom-nav {
            position: sticky;
            bottom: 0;
            left: 0;
            right: 0;
            background: rgba(8, 6, 4, 0.88);
            backdrop-filter: blur(20px);
```

다음으로 바꾼다:

```css
        .app-container {
            width: 100%;
            max-width: 430px;
            margin: 0 auto;
            height: 100dvh;
            display: flex;
            flex-direction: column;
            position: relative;
            background-color: var(--bg);
            box-shadow: var(--shell-shadow);
            overflow: hidden;
        }

        .bottom-nav {
            position: sticky;
            bottom: 0;
            left: 0;
            right: 0;
            background: var(--nav-bg);
            backdrop-filter: blur(20px);
```

- [ ] **Step 3: `.desktop-nav-lang`와 관련 hover 규칙 제거 (Task 2에서 버튼 자체를 지우므로 선점 삭제)**

다음 블록을 찾아서:

```css
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
```

다음으로 바꾼다(두 규칙 삭제, 미디어쿼리 닫는 중괄호만 유지):

```css
        }
```

- [ ] **Step 4: 컨트롤러가 `html.dark` 클래스를 수동으로 붙였다 뗐다 하며 색상이 반전되는지 확인 (참고용 — 실제 검증은 컨트롤러가 수행)**

이 태스크는 순수 CSS 변경이며 JS 상태 연결은 Task 3에서 이루어진다. 지금 시점엔 `<html class="dark">`가 정적으로 남아있으므로 페이지는 다크로 보여야 정상이다 — 구조적으로 `html.dark { ... }` 블록의 존재만 확인한다(`grep -c "html.dark {" index.html` → 1).

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: CSS 변수를 라이트 기본값 + html.dark 오버라이드로 반전"
```

---

### Task 2: 언어 변경 UI를 프로필로 통합 (온보딩/데스크톱 네비에서 제거)

**Files:**
- Modify: `index.html` (`Onboarding` 컴포넌트, `App`의 `Onboarding` 호출부, `App`의 데스크톱 상단 네비 마크업)

**Interfaces:**
- Consumes: 없음 (마크업/props 제거만)
- Produces: `Onboarding`은 더 이상 `setLang` prop을 받지 않는다. 이후 태스크(Task 4)는 `Onboarding` 시그니처가 `({ onStartQuiz, onDirectSelect, lang })`임을 전제로 작업한다.

- [ ] **Step 1: `Onboarding`에서 언어 토글 버튼과 `setLang` prop 제거**

다음 블록을 찾는다:

```jsx
        const Onboarding = ({ onStartQuiz, onDirectSelect, lang, setLang }) => (
            <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-8 animate-in slide-in-from-bottom-8 duration-500">
                <div className="flex justify-end mb-10 relative z-10">
                    <button onClick={() => setLang(l => l==='ko'?'en':'ko')} className="px-4 py-2 text-sm font-bold rounded-full border border-slate-700 bg-slate-800/50 text-slate-300">{lang==='ko'?'EN':'KR'}</button>
                </div>
                <div className="flex-1 flex flex-col justify-center items-center text-center">
```

다음으로 바꾼다:

```jsx
        const Onboarding = ({ onStartQuiz, onDirectSelect, lang }) => (
            <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-8 animate-in slide-in-from-bottom-8 duration-500">
                <div className="flex-1 flex flex-col justify-center items-center text-center">
```

- [ ] **Step 2: `App`에서 `Onboarding` 호출 시 `setLang` prop 제거**

다음 블록을 찾는다:

```jsx
                    {appState === 'onboard' && (
                        <Onboarding 
                            onStartQuiz={() => setAppState('quiz')} 
                            onDirectSelect={() => setAppState('direct')} 
                            lang={lang} 
                            setLang={setLang} 
                        />
                    )}
```

다음으로 바꾼다:

```jsx
                    {appState === 'onboard' && (
                        <Onboarding 
                            onStartQuiz={() => setAppState('quiz')} 
                            onDirectSelect={() => setAppState('direct')} 
                            lang={lang} 
                        />
                    )}
```

- [ ] **Step 3: 데스크톱 상단 네비에서 언어 토글 버튼 제거**

다음 블록을 찾는다:

```jsx
                                </div>
                                <button className="desktop-nav-lang" onClick={() => setLang(l => l === 'ko' ? 'en' : 'ko')}>
                                    {lang === 'en' ? 'KO' : 'EN'}
                                </button>
                            </nav>
```

다음으로 바꾼다:

```jsx
                                </div>
                            </nav>
```

- [ ] **Step 4: 구조적 확인**

`grep -n "setLang" index.html`을 실행해, 남은 참조가 `App`의 `useState`, `handleReset`이 없는 위치, `ProfileTab` 호출부(`setLang={setLang}`), `ProfileTab` 내부 버튼(`onClick={() => setLang(...)}`)뿐인지 확인한다. `Onboarding` 관련 줄에는 더 이상 나타나지 않아야 한다.

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: 언어 변경 UI를 프로필로 통합 (온보딩/데스크톱 네비에서 제거)"
```

---

### Task 3: 테마 state 및 프로필 토글 UI

**Files:**
- Modify: `index.html` (`App` 컴포넌트 state/useEffect, `ProfileTab` 컴포넌트, `App`의 `ProfileTab` 호출부)

**Interfaces:**
- Consumes: Task 1의 CSS 변수(`html.dark`가 트리거).
- Produces: `App`에 `theme`(`'light'|'dark'`), `setTheme` state. `ProfileTab`은 `theme`, `setTheme` prop을 받는다. 이후 태스크(Task 5의 `HomeTab`, Task 6의 `CurateTab`)는 `App`이 `theme`을 하위로 내려준다는 전제로 작업한다.

- [ ] **Step 1: `App`에 `theme` state와 초기화/동기화 `useEffect` 추가**

다음 블록을 찾는다:

```jsx
        const App = () => {
            const [appState, setAppState] = useState('splash'); // splash, onboard, quiz, direct, main
            const [mbti, setMbti] = useState(null);
            const [lang, setLang] = useState('ko');
            const [activeTab, setActiveTab] = useState('home'); // home, curate, profile

            useEffect(() => {
                const savedMbti = safeStorage.getItem('whiskey_mbti');
                const savedLang = safeStorage.getItem('whiskey_lang');
                if (savedLang) setLang(savedLang);
                if (savedMbti) {
                    setMbti(savedMbti);
                }
            }, []);

            useEffect(() => {
                safeStorage.setItem('whiskey_lang', lang);
            }, [lang]);
```

다음으로 바꾼다:

```jsx
        const App = () => {
            const [appState, setAppState] = useState('splash'); // splash, onboard, quiz, direct, main
            const [mbti, setMbti] = useState(null);
            const [lang, setLang] = useState('ko');
            const [theme, setTheme] = useState('dark');
            const [activeTab, setActiveTab] = useState('home'); // home, curate, profile

            useEffect(() => {
                const savedMbti = safeStorage.getItem('whiskey_mbti');
                const savedLang = safeStorage.getItem('whiskey_lang');
                const savedTheme = safeStorage.getItem('whiskey_theme');
                if (savedLang) setLang(savedLang);
                if (savedMbti) {
                    setMbti(savedMbti);
                }
                if (savedTheme) setTheme(savedTheme);
                else if (window.matchMedia && window.matchMedia('(prefers-color-scheme: light)').matches) setTheme('light');
            }, []);

            useEffect(() => {
                safeStorage.setItem('whiskey_lang', lang);
            }, [lang]);

            useEffect(() => {
                document.documentElement.classList.toggle('dark', theme === 'dark');
                safeStorage.setItem('whiskey_theme', theme);
            }, [theme]);
```

- [ ] **Step 2: `ProfileTab`에 테마 토글 버튼 추가**

다음 블록을 찾는다:

```jsx
        const ProfileTab = ({ mbti, lang, onReset, setLang }) => {
```

다음으로 바꾼다:

```jsx
        const ProfileTab = ({ mbti, lang, onReset, setLang, theme, setTheme }) => {
```

이어서 다음 블록을 찾는다:

```jsx
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
```

다음으로 바꾼다(테마 버튼을 언어 버튼과 리셋 버튼 사이에 삽입):

```jsx
                        <div className="flex flex-col gap-4">
                            <button
                                onClick={() => setLang(l => l === 'ko' ? 'en' : 'ko')}
                                className="w-full py-5 rounded-[20px] bg-slate-100 dark:bg-slate-800 text-slate-900 dark:text-white font-bold text-lg border-2 border-slate-200 dark:border-slate-700 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors">
                                {lang === 'en' ? '한국어로 변경' : 'Change to English'}
                            </button>

                            <button
                                onClick={() => setTheme(t => t === 'dark' ? 'light' : 'dark')}
                                className="w-full py-5 rounded-[20px] bg-slate-100 dark:bg-slate-800 text-slate-900 dark:text-white font-bold text-lg border-2 border-slate-200 dark:border-slate-700 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors">
                                {theme === 'dark' ? (lang === 'en' ? 'Switch to Light Mode' : '라이트 모드로 변경') : (lang === 'en' ? 'Switch to Dark Mode' : '다크 모드로 변경')}
                            </button>

                            <button
                                onClick={onReset}
                                className="w-full py-5 rounded-[20px] bg-red-900/20 text-red-400 font-bold text-lg border-2 border-red-900/30 hover:bg-red-900/40 transition-colors">
                                {lang === 'en' ? 'Reset MBTI Profile' : '프로필 초기화 (다시하기)'}
                            </button>
                        </div>
```

- [ ] **Step 3: `App`이 `ProfileTab`에 `theme`/`setTheme` 전달**

다음 줄을 찾는다:

```jsx
                                {activeTab === 'profile' && <ProfileTab mbti={mbti} lang={lang} onReset={handleReset} setLang={setLang} />}
```

다음으로 바꾼다:

```jsx
                                {activeTab === 'profile' && <ProfileTab mbti={mbti} lang={lang} onReset={handleReset} setLang={setLang} theme={theme} setTheme={setTheme} />}
```

- [ ] **Step 4: 컨트롤러 브라우저 검증 안내 (구현자는 수행하지 않음)**

구현 담당자는 `preview_*` 도구가 없으므로 이 스텝은 스킵하고 보고서에 "브라우저 검증은 컨트롤러가 수행"이라고 남긴다. 컨트롤러는 이후 다음을 확인한다: 시스템이 라이트 선호일 때 첫 로드가 라이트로 뜨는지, 프로필에서 토글 클릭 시 `<html>`의 `dark` 클래스가 토글되고 배경/텍스트 색이 즉시 바뀌는지, 새로고침 후에도 선택한 테마가 유지되는지(`localStorage.whiskey_theme`).

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: 라이트/다크 테마 state 및 프로필 토글 UI 추가"
```

---

### Task 4: dark: 클래스 변환 — Splash, Onboarding, Quiz, DirectSelect

**Files:**
- Modify: `index.html` (`Splash`, `Onboarding`, `Quiz`, `DirectSelect` 컴포넌트)

**Interfaces:**
- Consumes: Task 1의 CSS 변수, Task 3의 `html.dark` 토글 메커니즘(간접 — Tailwind `dark:` variant는 `html.dark` 존재 여부로 자동 활성화됨). Task 2 이후의 `Onboarding` 시그니처(`setLang` 없음).
- Produces: 없음 (leaf 변경)

- [ ] **Step 1: `Splash`의 `text-white`를 라이트 대응**

다음 줄을 찾는다:

```jsx
                    <h1 className="text-4xl font-black text-white playfair tracking-widest mb-3">WHISKY MBTI</h1>
```

다음으로 바꾼다:

```jsx
                    <h1 className="text-4xl font-black text-slate-900 dark:text-white playfair tracking-widest mb-3">WHISKY MBTI</h1>
```

- [ ] **Step 2: `Onboarding`의 텍스트/버튼 클래스 변환**

다음 블록을 찾는다(Task 2 적용 후 상태):

```jsx
                    <h2 className="text-[2rem] max-[400px]:text-[1.6rem] font-black text-white mb-5 leading-tight whitespace-nowrap tracking-tighter">{TEXT[lang].onboard_title}</h2>
                    <p className="text-slate-400 text-lg leading-relaxed mb-14 px-2 whitespace-pre-line">{TEXT[lang].onboard_sub}</p>
                </div>
                <div className="flex flex-col gap-4 pb-4">
                    <button onClick={onStartQuiz} className="w-full py-5 rounded-[20px] bg-[#c8962a] text-white font-bold text-xl shadow-xl shadow-[#c8962a]/30 hover:bg-[#a67c20] transition-colors">{TEXT[lang].start_quiz}</button>
                    <button onClick={onDirectSelect} className="w-full py-5 rounded-[20px] border-2 border-slate-700 text-slate-300 font-bold text-lg hover:bg-slate-800 transition-colors">{TEXT[lang].skip_quiz}</button>
                </div>
```

다음으로 바꾼다:

```jsx
                    <h2 className="text-[2rem] max-[400px]:text-[1.6rem] font-black text-slate-900 dark:text-white mb-5 leading-tight whitespace-nowrap tracking-tighter">{TEXT[lang].onboard_title}</h2>
                    <p className="text-slate-500 dark:text-slate-400 text-lg leading-relaxed mb-14 px-2 whitespace-pre-line">{TEXT[lang].onboard_sub}</p>
                </div>
                <div className="flex flex-col gap-4 pb-4">
                    <button onClick={onStartQuiz} className="w-full py-5 rounded-[20px] bg-[#c8962a] text-white font-bold text-xl shadow-xl shadow-[#c8962a]/30 hover:bg-[#a67c20] transition-colors">{TEXT[lang].start_quiz}</button>
                    <button onClick={onDirectSelect} className="w-full py-5 rounded-[20px] border-2 border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 font-bold text-lg hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors">{TEXT[lang].skip_quiz}</button>
                </div>
```

참고: `start_quiz` 버튼의 `bg-[#c8962a] text-white`는 그대로 유지한다(골드 배경 위 흰 텍스트는 라이트/다크 모두에서 대비가 충분하므로 브랜드 버튼은 변경 없음).

- [ ] **Step 3: `Quiz`의 진행바/텍스트/옵션 버튼 클래스 변환**

다음 블록을 찾는다:

```jsx
                <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-6 animate-in slide-in-from-right-8">
                    <div className="w-full h-2.5 bg-slate-800 rounded-full mb-10 overflow-hidden">
                        <div className="h-full bg-[#c8962a] transition-all duration-300" style={{ width: `${((qIdx+1)/QUIZ_QUESTIONS.length)*100}%` }}></div>
                    </div>
                    <div className="flex-1 flex flex-col justify-center pb-24">
                        <span className="text-[#c8962a] font-bold text-lg mb-5">Q{qIdx+1}.</span>
                        <h2 className="text-[1.75rem] font-bold text-white mb-12 leading-snug">{q[lang]}</h2>
                        <div className="flex flex-col gap-5">
                            {[q.A, q.B].map((opt, i) => (
                                <button key={i} onClick={() => handleAnswer(opt.val)} className="p-7 text-left rounded-[24px] border-2 border-slate-700 bg-slate-800/50 text-slate-200 hover:bg-[#c8962a]/10 hover:border-[#c8962a]/50 transition-all font-medium text-xl leading-relaxed shadow-sm">
                                    {opt.label[lang]}
                                </button>
                            ))}
                        </div>
                    </div>
                </div>
```

다음으로 바꾼다:

```jsx
                <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-6 animate-in slide-in-from-right-8">
                    <div className="w-full h-2.5 bg-slate-200 dark:bg-slate-800 rounded-full mb-10 overflow-hidden">
                        <div className="h-full bg-[#c8962a] transition-all duration-300" style={{ width: `${((qIdx+1)/QUIZ_QUESTIONS.length)*100}%` }}></div>
                    </div>
                    <div className="flex-1 flex flex-col justify-center pb-24">
                        <span className="text-[#c8962a] font-bold text-lg mb-5">Q{qIdx+1}.</span>
                        <h2 className="text-[1.75rem] font-bold text-slate-900 dark:text-white mb-12 leading-snug">{q[lang]}</h2>
                        <div className="flex flex-col gap-5">
                            {[q.A, q.B].map((opt, i) => (
                                <button key={i} onClick={() => handleAnswer(opt.val)} className="p-7 text-left rounded-[24px] border-2 border-slate-200 dark:border-slate-700 bg-slate-100/70 dark:bg-slate-800/50 text-slate-700 dark:text-slate-200 hover:bg-[#c8962a]/10 hover:border-[#c8962a]/50 transition-all font-medium text-xl leading-relaxed shadow-sm">
                                    {opt.label[lang]}
                                </button>
                            ))}
                        </div>
                    </div>
                </div>
```

- [ ] **Step 4: `DirectSelect`의 헤더/타입 버튼 클래스 변환**

다음 블록을 찾는다:

```jsx
            <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-6 animate-in slide-in-from-bottom-8 overflow-y-auto hide-scroll">
                <header className="mb-10 flex flex-col gap-3">
                    <button onClick={onBack} className="self-start p-2 -ml-2 text-slate-400 hover:text-white mb-2"><Icon name="arrow-left" className="w-7 h-7"/></button>
                    <h2 className="max-[375px]:text-2xl text-3xl font-extrabold text-white whitespace-nowrap tracking-tighter">{TEXT[lang].step1_title}</h2>
                </header>
                <div className="flex-1 space-y-8 pb-20">
                    {MBTI_GROUPS.map((group) => (
                        <div key={group.key} className="space-y-3">
                            <h3 className={`text-sm font-bold ${group.color} uppercase tracking-widest pl-1`}>{group.label[lang]}</h3>
                            <div className="grid grid-cols-4 gap-3">
                                {group.types.map(type => (
                                    <button key={type} onClick={() => onComplete(type)} className={`py-5 rounded-2xl border-2 transition-all flex flex-col items-center justify-center border-slate-700 bg-slate-800/50 text-slate-300 hover:bg-slate-700 hover:border-slate-500`}>
                                        <span className="font-bold text-lg">{type}</span>
                                    </button>
                                ))}
                            </div>
                        </div>
                    ))}
                </div>
            </div>
```

다음으로 바꾼다:

```jsx
            <div className="flex flex-col h-full pt-[calc(env(safe-area-inset-top,48px)+32px)] p-6 animate-in slide-in-from-bottom-8 overflow-y-auto hide-scroll">
                <header className="mb-10 flex flex-col gap-3">
                    <button onClick={onBack} className="self-start p-2 -ml-2 text-slate-500 dark:text-slate-400 hover:text-slate-900 dark:hover:text-white mb-2"><Icon name="arrow-left" className="w-7 h-7"/></button>
                    <h2 className="max-[375px]:text-2xl text-3xl font-extrabold text-slate-900 dark:text-white whitespace-nowrap tracking-tighter">{TEXT[lang].step1_title}</h2>
                </header>
                <div className="flex-1 space-y-8 pb-20">
                    {MBTI_GROUPS.map((group) => (
                        <div key={group.key} className="space-y-3">
                            <h3 className={`text-sm font-bold ${group.color} uppercase tracking-widest pl-1`}>{group.label[lang]}</h3>
                            <div className="grid grid-cols-4 gap-3">
                                {group.types.map(type => (
                                    <button key={type} onClick={() => onComplete(type)} className={`py-5 rounded-2xl border-2 transition-all flex flex-col items-center justify-center border-slate-200 dark:border-slate-700 bg-slate-100/70 dark:bg-slate-800/50 text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-700 hover:border-slate-400 dark:hover:border-slate-500`}>
                                        <span className="font-bold text-lg">{type}</span>
                                    </button>
                                ))}
                            </div>
                        </div>
                    ))}
                </div>
            </div>
```

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: Splash/Onboarding/Quiz/DirectSelect 라이트모드 대응"
```

---

### Task 5: dark: 클래스 변환 — HomeTab, ProfileTab, LoadingScreen, FlavorBars

**Files:**
- Modify: `index.html` (`HomeTab`, `ProfileTab`, `LoadingScreen`, `FlavorBars` 컴포넌트)

**Interfaces:**
- Consumes: Task 1 CSS 변수, Task 3의 `ProfileTab` 시그니처(`theme`/`setTheme` prop 포함 — 이미 Task 3에서 추가됨, 이 태스크는 나머지 클래스만 변환).
- Produces: 없음

- [ ] **Step 1: `FlavorBars`의 트랙 배경 변환**

다음 줄을 찾는다:

```jsx
                            <div className="h-1.5 bg-slate-700/50 rounded-full overflow-hidden"><div className="h-full rounded-full bg-gradient-to-r from-[#c8962a]/50 to-[#c8962a]" style={{ width: `${v}%`, transition: 'width 1.2s cubic-bezier(0.22,1,0.36,1)' }} /></div>
```

다음으로 바꾼다:

```jsx
                            <div className="h-1.5 bg-slate-300/50 dark:bg-slate-700/50 rounded-full overflow-hidden"><div className="h-full rounded-full bg-gradient-to-r from-[#c8962a]/50 to-[#c8962a]" style={{ width: `${v}%`, transition: 'width 1.2s cubic-bezier(0.22,1,0.36,1)' }} /></div>
```

- [ ] **Step 2: `LoadingScreen`의 배경/텍스트 클래스 변환**

다음 블록을 찾는다:

```jsx
                <div className="min-h-screen flex flex-col items-center justify-center p-6 text-center animate-in fade-in" style={{ paddingTop: 'env(safe-area-inset-top)' }}>
                    <div className="relative mb-12">
                        <div className="absolute inset-0 bg-[#c8962a] rounded-full blur-[40px] opacity-30 animate-pulse"></div>
                        <div className="relative p-8 bg-slate-800 rounded-full shadow-2xl border border-slate-700"><Icon name="loader-2" className="w-14 h-14 text-[#c8962a] animate-spin" /></div>
                    </div>
                    <h3 className="text-2xl font-bold text-white mb-3 min-h-[32px]">{msgs[msg]}</h3>
                    <div className="w-72 h-2 bg-slate-800 rounded-full mt-6 overflow-hidden"><div className="h-full bg-[#c8962a] animate-progress rounded-full"></div></div>
                </div>
```

다음으로 바꾼다:

```jsx
                <div className="min-h-screen flex flex-col items-center justify-center p-6 text-center animate-in fade-in" style={{ paddingTop: 'env(safe-area-inset-top)' }}>
                    <div className="relative mb-12">
                        <div className="absolute inset-0 bg-[#c8962a] rounded-full blur-[40px] opacity-30 animate-pulse"></div>
                        <div className="relative p-8 bg-slate-100 dark:bg-slate-800 rounded-full shadow-2xl border border-slate-200 dark:border-slate-700"><Icon name="loader-2" className="w-14 h-14 text-[#c8962a] animate-spin" /></div>
                    </div>
                    <h3 className="text-2xl font-bold text-slate-900 dark:text-white mb-3 min-h-[32px]">{msgs[msg]}</h3>
                    <div className="w-72 h-2 bg-slate-200 dark:bg-slate-800 rounded-full mt-6 overflow-hidden"><div className="h-full bg-[#c8962a] animate-progress rounded-full"></div></div>
                </div>
```

- [ ] **Step 3: `HomeTab`의 인사말/아바타/데일리픽 텍스트 클래스 변환**

다음 블록을 찾는다:

```jsx
                    <div className="flex items-center justify-between px-2">
                        <div>
                            <p className="text-slate-400 text-base mb-1.5">{lang==='en'?'Welcome back,':'안녕하세요,'}</p>
                            <h2 className="max-[375px]:text-[1.8rem] text-[2.2rem] font-black text-white whitespace-nowrap tracking-tighter">
                                <span className={group.color}>{mbti}</span> {group.label[lang]}
                            </h2>
                        </div>
                        <div className="w-14 h-14 rounded-full bg-slate-800 border-2 border-slate-700 flex items-center justify-center text-3xl shadow-lg">🥃</div>
                    </div>
```

다음으로 바꾼다:

```jsx
                    <div className="flex items-center justify-between px-2">
                        <div>
                            <p className="text-slate-500 dark:text-slate-400 text-base mb-1.5">{lang==='en'?'Welcome back,':'안녕하세요,'}</p>
                            <h2 className="max-[375px]:text-[1.8rem] text-[2.2rem] font-black text-slate-900 dark:text-white whitespace-nowrap tracking-tighter">
                                <span className={group.color}>{mbti}</span> {group.label[lang]}
                            </h2>
                        </div>
                        <div className="w-14 h-14 rounded-full bg-slate-100 dark:bg-slate-800 border-2 border-slate-200 dark:border-slate-700 flex items-center justify-center text-3xl shadow-lg">🥃</div>
                    </div>
```

이어서 다음 블록을 찾는다:

```jsx
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

다음으로 바꾼다:

```jsx
                        <div className="pb-4">
                            <h3 className="font-bold text-slate-600 dark:text-slate-300 mb-4 px-2 text-lg">{lang==='en'?"Today's Daily Pick":"오늘의 데일리 픽"}</h3>
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
                        </div>
```

- [ ] **Step 4: `ProfileTab`의 나머지 클래스 변환 (아바타, 타이틀)**

다음 블록을 찾는다:

```jsx
                <div className="space-y-8 animate-in fade-in pt-[calc(env(safe-area-inset-top,48px)+24px)] flex-1 flex flex-col">
                    <h2 className="text-3xl font-extrabold text-white mb-8 px-2">{lang === 'en' ? 'Profile' : '프로필'}</h2>
                    
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
```

다음으로 바꾼다:

```jsx
                <div className="space-y-8 animate-in fade-in pt-[calc(env(safe-area-inset-top,48px)+24px)] flex-1 flex flex-col">
                    <h2 className="text-3xl font-extrabold text-slate-900 dark:text-white mb-8 px-2">{lang === 'en' ? 'Profile' : '프로필'}</h2>
                    
                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 lg:gap-8 lg:items-start flex-1">
                        <GlassCard className="p-10 flex flex-col items-center text-center">
                            <div className="w-24 h-24 bg-slate-100 dark:bg-slate-800 rounded-full flex items-center justify-center mb-6 border-2 border-slate-200 dark:border-slate-700 text-4xl">
                                👤
                            </div>
                            <h3 className={`text-4xl font-black mb-2 ${group?.color || 'text-slate-900 dark:text-white'}`}>{mbti}</h3>
                            <p className="text-sm font-bold uppercase tracking-widest text-slate-500 dark:text-slate-400">
                                {group ? group.label[lang] : 'Unknown'}
                            </p>
                        </GlassCard>
```

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: HomeTab/ProfileTab/LoadingScreen/FlavorBars 라이트모드 대응"
```

---

### Task 6: dark: 클래스 변환 — CurateTab 전체 + ShareCard 패널

**Files:**
- Modify: `index.html` (`CurateTab`의 4개 스텝, `ShareCard`의 패널 마크업 — canvas 드로잉 로직 제외)

**Interfaces:**
- Consumes: Task 1 CSS 변수
- Produces: 없음

- [ ] **Step 1: cStep 1 (무드 선택) 헤더/카드 클래스 변환**

다음 블록을 찾는다:

```jsx
                    <header className="px-2">
                        <h2 className="max-[375px]:text-[1.1rem] text-[1.35rem] font-extrabold text-white leading-tight whitespace-nowrap tracking-tighter">{t.step2_title}</h2>
                        <p className="text-slate-400 text-base mt-2">{t.step2_sub}</p>
                    </header>
                    <div className="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-4 pb-6 flex-1">
                        {MOODS.map(m => (
                            <button key={m.id} onClick={() => setMoods(p => p.includes(m.id) ? p.filter(id=>id!==m.id) : p.length<3 && !MOODS.find(x=>x.id===m.id).conflicts.some(c=>p.includes(c)) ? [...p, m.id] : p)}
                                className={`rounded-[24px] border-2 flex flex-col items-center justify-center text-center p-4 transition-all ${moods.includes(m.id) ? 'border-[#c8962a] bg-[#c8962a]/20 shadow-lg' : 'border-slate-700 bg-slate-800'}`}>
                                <div className="text-[2.8rem] mb-2 drop-shadow-md">{m.emoji}</div>
                                <div className="w-full flex-1 flex items-center justify-center"><span className={`text-[12.5px] font-bold leading-snug break-keep ${moods.includes(m.id)?'text-[#eac266]':'text-slate-300'}`}>{m.expr[lang]}</span></div>
                            </button>
                        ))}
                    </div>
```

다음으로 바꾼다:

```jsx
                    <header className="px-2">
                        <h2 className="max-[375px]:text-[1.1rem] text-[1.35rem] font-extrabold text-slate-900 dark:text-white leading-tight whitespace-nowrap tracking-tighter">{t.step2_title}</h2>
                        <p className="text-slate-500 dark:text-slate-400 text-base mt-2">{t.step2_sub}</p>
                    </header>
                    <div className="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-4 pb-6 flex-1">
                        {MOODS.map(m => (
                            <button key={m.id} onClick={() => setMoods(p => p.includes(m.id) ? p.filter(id=>id!==m.id) : p.length<3 && !MOODS.find(x=>x.id===m.id).conflicts.some(c=>p.includes(c)) ? [...p, m.id] : p)}
                                className={`rounded-[24px] border-2 flex flex-col items-center justify-center text-center p-4 transition-all ${moods.includes(m.id) ? 'border-[#c8962a] bg-[#c8962a]/20 shadow-lg' : 'border-slate-200 dark:border-slate-700 bg-slate-100 dark:bg-slate-800'}`}>
                                <div className="text-[2.8rem] mb-2 drop-shadow-md">{m.emoji}</div>
                                <div className="w-full flex-1 flex items-center justify-center"><span className={`text-[12.5px] font-bold leading-snug break-keep ${moods.includes(m.id)?'text-[#eac266]':'text-slate-600 dark:text-slate-300'}`}>{m.expr[lang]}</span></div>
                            </button>
                        ))}
                    </div>
```

- [ ] **Step 2: cStep 2 (블렌딩 결과) 헤더/카드/버튼 클래스 변환**

다음 블록을 찾는다:

```jsx
                        <header className="text-center">
                            <h2 className="max-[375px]:text-[1.4rem] text-[1.75rem] font-extrabold text-white whitespace-nowrap tracking-tighter">{t.step3_title}</h2>
                            <p className="text-slate-400 text-base mt-2">{t.step3_sub}</p>
                        </header>
                        <div className="flex-1 flex flex-col justify-center">
                            <GlassCard className="flex flex-col items-center gap-5 p-10 w-full mt-8">
                                <div className="text-[6rem] drop-shadow-2xl mb-4">{bM.emoji}</div>
                                <span className="text-4xl font-black text-[#c8962a] text-center">{bM.label}</span>
                                <span className="text-base text-slate-300 text-center font-medium mt-3 leading-relaxed">{bM.desc}</span>
                                <div className="mt-6 pt-6 border-t-2 border-white/10 w-full text-center"><span className="text-lg font-extrabold text-[#eac266]">{bM.invite}</span></div>
                            </GlassCard>
                        </div>
                        <div className="flex gap-4 mt-10 sticky bottom-0 bg-[var(--bg)] pt-4">
                            <button onClick={()=>setCStep(1)} className="flex-1 py-5 border-2 border-slate-700 rounded-[20px] text-slate-300 font-bold text-lg hover:bg-slate-800">뒤로</button>
                            <button onClick={()=>setCStep(3)} className="flex-1 py-5 bg-[#c8962a] rounded-[20px] text-white font-bold text-lg shadow-xl shadow-[#c8962a]/30 hover:bg-[#b08020]">계속하기</button>
                        </div>
```

다음으로 바꾼다:

```jsx
                        <header className="text-center">
                            <h2 className="max-[375px]:text-[1.4rem] text-[1.75rem] font-extrabold text-slate-900 dark:text-white whitespace-nowrap tracking-tighter">{t.step3_title}</h2>
                            <p className="text-slate-500 dark:text-slate-400 text-base mt-2">{t.step3_sub}</p>
                        </header>
                        <div className="flex-1 flex flex-col justify-center">
                            <GlassCard className="flex flex-col items-center gap-5 p-10 w-full mt-8">
                                <div className="text-[6rem] drop-shadow-2xl mb-4">{bM.emoji}</div>
                                <span className="text-4xl font-black text-[#c8962a] text-center">{bM.label}</span>
                                <span className="text-base text-slate-600 dark:text-slate-300 text-center font-medium mt-3 leading-relaxed">{bM.desc}</span>
                                <div className="mt-6 pt-6 border-t-2 border-black/10 dark:border-white/10 w-full text-center"><span className="text-lg font-extrabold text-[#eac266]">{bM.invite}</span></div>
                            </GlassCard>
                        </div>
                        <div className="flex gap-4 mt-10 sticky bottom-0 bg-[var(--bg)] pt-4">
                            <button onClick={()=>setCStep(1)} className="flex-1 py-5 border-2 border-slate-200 dark:border-slate-700 rounded-[20px] text-slate-600 dark:text-slate-300 font-bold text-lg hover:bg-slate-100 dark:hover:bg-slate-800">뒤로</button>
                            <button onClick={()=>setCStep(3)} className="flex-1 py-5 bg-[#c8962a] rounded-[20px] text-white font-bold text-lg shadow-xl shadow-[#c8962a]/30 hover:bg-[#b08020]">계속하기</button>
                        </div>
```

- [ ] **Step 3: cStep 3 (가격대) 헤더/카드/버튼 클래스 변환**

다음 블록을 찾는다:

```jsx
                    <header className="px-2"><h2 className="max-[375px]:text-[1.4rem] text-[1.75rem] font-extrabold text-white whitespace-nowrap tracking-tighter">{t.step4_title}</h2></header>
                    <div className="grid gap-4 lg:grid-cols-3 flex-1">
                        {PRICE_RANGES.map(r => (
                            <button key={r.id} onClick={()=>setPrice(r.id)} className={`p-6 rounded-[24px] border-2 flex items-center justify-between transition-all ${price===r.id?'border-[#c8962a] bg-[#c8962a]/20 shadow-md':'border-slate-700 bg-slate-800'}`}>
                                <div className="flex items-center gap-4">
                                    <div className={`p-3 rounded-full ${price===r.id?'bg-[#c8962a]/50 text-[#fcedc2]':'bg-slate-700 text-slate-400'}`}><Icon name="credit-card" className="w-6 h-6"/></div>
                                    <span className={`font-bold text-xl ${price===r.id?'text-[#eac266]':'text-slate-400'}`}>{r.label[lang]}</span>
                                </div>
                                {price===r.id && <Icon name="check" className="text-[#c8962a] w-7 h-7"/>}
                            </button>
                        ))}
                    </div>
                    <div className="flex gap-4 mt-10 sticky bottom-0 bg-[var(--bg)] pt-4">
                        <button onClick={()=>setCStep(2)} className="flex-1 py-5 border-2 border-slate-700 rounded-[20px] text-slate-300 font-bold text-lg hover:bg-slate-800">뒤로</button>
                        <button onClick={run} className="flex-[2] py-5 bg-[#c8962a] rounded-[20px] text-white font-bold text-lg shadow-xl shadow-[#c8962a]/30 hover:bg-[#b08020]">{t.run_btn}</button>
                    </div>
```

다음으로 바꾼다:

```jsx
                    <header className="px-2"><h2 className="max-[375px]:text-[1.4rem] text-[1.75rem] font-extrabold text-slate-900 dark:text-white whitespace-nowrap tracking-tighter">{t.step4_title}</h2></header>
                    <div className="grid gap-4 lg:grid-cols-3 flex-1">
                        {PRICE_RANGES.map(r => (
                            <button key={r.id} onClick={()=>setPrice(r.id)} className={`p-6 rounded-[24px] border-2 flex items-center justify-between transition-all ${price===r.id?'border-[#c8962a] bg-[#c8962a]/20 shadow-md':'border-slate-200 dark:border-slate-700 bg-slate-100 dark:bg-slate-800'}`}>
                                <div className="flex items-center gap-4">
                                    <div className={`p-3 rounded-full ${price===r.id?'bg-[#c8962a]/50 text-[#fcedc2]':'bg-slate-200 dark:bg-slate-700 text-slate-500 dark:text-slate-400'}`}><Icon name="credit-card" className="w-6 h-6"/></div>
                                    <span className={`font-bold text-xl ${price===r.id?'text-[#eac266]':'text-slate-500 dark:text-slate-400'}`}>{r.label[lang]}</span>
                                </div>
                                {price===r.id && <Icon name="check" className="text-[#c8962a] w-7 h-7"/>}
                            </button>
                        ))}
                    </div>
                    <div className="flex gap-4 mt-10 sticky bottom-0 bg-[var(--bg)] pt-4">
                        <button onClick={()=>setCStep(2)} className="flex-1 py-5 border-2 border-slate-200 dark:border-slate-700 rounded-[20px] text-slate-600 dark:text-slate-300 font-bold text-lg hover:bg-slate-100 dark:hover:bg-slate-800">뒤로</button>
                        <button onClick={run} className="flex-[2] py-5 bg-[#c8962a] rounded-[20px] text-white font-bold text-lg shadow-xl shadow-[#c8962a]/30 hover:bg-[#b08020]">{t.run_btn}</button>
                    </div>
```

- [ ] **Step 4: cStep 4 (결과) 텍스트/태그/구매버튼 클래스 변환**

다음 블록을 찾는다:

```jsx
                    <div className="text-center relative pt-4">
                        <div className="flex justify-center min-h-[180px]"><PourAnimation color={result.whiskey.color} /></div>
                        <h2 className="text-[2.2rem] font-black mt-6 text-white px-4 leading-tight">{lang==='ko'?result.whiskey.name:result.whiskey.nameEn}</h2>
                        <p className="text-slate-400 font-bold mt-2 text-base">{result.whiskey.style} · <span className="text-[#eac266]">{result.whiskey.abv}%</span></p>
                    </div>
                    <div className="flex justify-center flex-wrap gap-2.5 px-4 pb-2">
                        {result.whiskey.tags.map(tag => <span key={tag} className="px-4 py-1.5 bg-white/5 border border-white/10 text-[#c8962a] rounded-full text-xs font-bold uppercase tracking-widest">#{tag}</span>)}
                    </div>
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
                    <div className="pt-2 pb-6">
                        <button 
                            onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent(result.whiskey.name + ' 위스키')}`, '_blank')}
                            className="w-full py-4 rounded-[20px] bg-slate-800 border-2 border-slate-700 text-[#c8962a] font-bold text-lg hover:bg-slate-700 hover:border-[#c8962a]/50 transition-all flex items-center justify-center gap-2 shadow-lg"
                        >
                            <Icon name="shopping-bag" className="w-5 h-5" />
                            {t.buy_btn}
                        </button>
                    </div>
```

다음으로 바꾼다:

```jsx
                    <div className="text-center relative pt-4">
                        <div className="flex justify-center min-h-[180px]"><PourAnimation color={result.whiskey.color} theme={theme} /></div>
                        <h2 className="text-[2.2rem] font-black mt-6 text-slate-900 dark:text-white px-4 leading-tight">{lang==='ko'?result.whiskey.name:result.whiskey.nameEn}</h2>
                        <p className="text-slate-500 dark:text-slate-400 font-bold mt-2 text-base">{result.whiskey.style} · <span className="text-[#eac266]">{result.whiskey.abv}%</span></p>
                    </div>
                    <div className="flex justify-center flex-wrap gap-2.5 px-4 pb-2">
                        {result.whiskey.tags.map(tag => <span key={tag} className="px-4 py-1.5 bg-black/5 dark:bg-white/5 border border-black/10 dark:border-white/10 text-[#c8962a] rounded-full text-xs font-bold uppercase tracking-widest">#{tag}</span>)}
                    </div>
                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 lg:gap-8 lg:items-start">
                        <GlassCard className="p-8">
                            <h3 className="text-sm font-bold text-[#c8962a] mb-4 uppercase tracking-widest flex items-center gap-2"><Icon name="book-open" className="w-5 h-5"/> Tasting Notes</h3>
                            <p className="text-base text-slate-700 dark:text-slate-200 leading-relaxed font-medium">{lang==='ko'?result.whiskey.desc:result.whiskey.descEn}</p>
                        </GlassCard>
                        <GlassCard className="p-8">
                            <h3 className="text-sm font-bold text-[#c8962a] mb-6 uppercase tracking-widest flex items-center gap-2"><Icon name="bar-chart-2" className="w-5 h-5"/> Flavor Profile</h3>
                            <FlavorBars flavor={result.whiskey.flavor} lang={lang} />
                        </GlassCard>
                    </div>
                    <div className="pt-2 pb-6">
                        <button 
                            onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent(result.whiskey.name + ' 위스키')}`, '_blank')}
                            className="w-full py-4 rounded-[20px] bg-slate-100 dark:bg-slate-800 border-2 border-slate-200 dark:border-slate-700 text-[#c8962a] font-bold text-lg hover:bg-slate-200 dark:hover:bg-slate-700 hover:border-[#c8962a]/50 transition-all flex items-center justify-center gap-2 shadow-lg"
                        >
                            <Icon name="shopping-bag" className="w-5 h-5" />
                            {t.buy_btn}
                        </button>
                    </div>
```

주의: 이 스텝에서 `<PourAnimation color={result.whiskey.color} theme={theme} />`로 `theme` prop을 넘긴다. `CurateTab`은 `theme`을 prop으로 받아야 하므로 이어서 다음 스텝에서 시그니처를 수정한다.

- [ ] **Step 5: `CurateTab` 시그니처에 `theme` prop 추가, `App`의 호출부 갱신**

다음 줄을 찾는다:

```jsx
        const CurateTab = ({ mbti, lang, bio }) => {
```

다음으로 바꾼다:

```jsx
        const CurateTab = ({ mbti, lang, bio, theme }) => {
```

이어서 다음 줄을 찾는다:

```jsx
                                {activeTab === 'curate' && <CurateTab mbti={mbti} lang={lang} bio={calculateBiorhythm(mbti)} />}
```

다음으로 바꾼다:

```jsx
                                {activeTab === 'curate' && <CurateTab mbti={mbti} lang={lang} bio={calculateBiorhythm(mbti)} theme={theme} />}
```

- [ ] **Step 6: `ShareCard` 패널 마크업 변환 (canvas 드로잉 로직 제외)**

다음 블록을 찾는다:

```jsx
                <div className="fixed inset-0 z-[1000] bg-black/80 backdrop-blur-md flex flex-col items-center justify-center p-6 animate-in fade-in" onClick={onClose}>
                    <div className="bg-slate-900 border border-slate-700 rounded-[32px] p-6 w-full max-w-sm flex flex-col gap-5 shadow-2xl" onClick={e=>e.stopPropagation()}>
                        <div className="flex justify-between items-center text-white"><span className="font-bold text-lg">Share Card</span><button onClick={onClose} className="text-slate-400 text-3xl">&times;</button></div>
                        <div className="rounded-2xl overflow-hidden border border-slate-700 shadow-inner"><canvas ref={canvasRef} className="w-full h-auto block" /></div>
                        <button onClick={copy} className={`py-5 rounded-2xl font-bold text-lg transition-all ${copied ? 'bg-green-500/20 text-green-400 border border-green-500/30' : 'bg-[#c8962a] text-white shadow-xl shadow-[#c8962a]/30'}`}>{copied ? '✓ 클립보드에 복사됨' : '📋 이미지 복사 및 저장'}</button>
                    </div>
                </div>
```

다음으로 바꾼다(오버레이 `bg-black/80`은 그대로 유지 — 딤 처리는 테마 무관):

```jsx
                <div className="fixed inset-0 z-[1000] bg-black/80 backdrop-blur-md flex flex-col items-center justify-center p-6 animate-in fade-in" onClick={onClose}>
                    <div className="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-[32px] p-6 w-full max-w-sm flex flex-col gap-5 shadow-2xl" onClick={e=>e.stopPropagation()}>
                        <div className="flex justify-between items-center text-slate-900 dark:text-white"><span className="font-bold text-lg">Share Card</span><button onClick={onClose} className="text-slate-500 dark:text-slate-400 text-3xl">&times;</button></div>
                        <div className="rounded-2xl overflow-hidden border border-slate-200 dark:border-slate-700 shadow-inner"><canvas ref={canvasRef} className="w-full h-auto block" /></div>
                        <button onClick={copy} className={`py-5 rounded-2xl font-bold text-lg transition-all ${copied ? 'bg-green-500/20 text-green-400 border border-green-500/30' : 'bg-[#c8962a] text-white shadow-xl shadow-[#c8962a]/30'}`}>{copied ? '✓ 클립보드에 복사됨' : '📋 이미지 복사 및 저장'}</button>
                    </div>
                </div>
```

- [ ] **Step 7: 커밋**

```bash
git add index.html
git commit -m "feat: CurateTab 4단계 및 ShareCard 패널 라이트모드 대응"
```

---

### Task 7: WaveChart 테마 대응 SVG 색상

**Files:**
- Modify: `index.html` (`WaveChart` 컴포넌트, `HomeTab`의 호출부, `App`의 `HomeTab` 호출부)

**Interfaces:**
- Consumes: Task 3의 `App`의 `theme` state.
- Produces: `WaveChart`는 `theme` prop을 받는다(`'light'|'dark'`).

- [ ] **Step 1: `WaveChart` 컴포넌트에 `theme` prop과 색상 분기 추가**

다음 블록을 찾는다:

```jsx
        const WaveChart = ({ bio, lang }) => {
            const cycles = {
                physical: { v: bio.physical/100 * 2 - 1, color:'#0d9488', ko:'신체', en:'Physical' },
                emotional: { v: bio.emotional/100 * 2 - 1, color:'#f59e0b', ko:'감성', en:'Emotional' },
                intellectual: { v: bio.intellectual/100 * 2 - 1, color:'#3b82f6', ko:'지성', en:'Intellectual' }
            };
            const W = 320, H = 80, DAYS = 21;
            const x = (i) => (i / (DAYS-1)) * W;
            const y = (v) => H/2 - v * (H/2 - 8);
            const wavePath = (vTarget) => {
                const pts = Array.from({length:DAYS}, (_,i) => [x(i), y(Math.sin((i/(DAYS-1) * Math.PI * 4) + Math.asin(vTarget)))]);
                return 'M ' + pts.map(p=>p.join(',')).join(' L ');
            };

            return (
                <div>
                    <div className="bg-black/40 rounded-3xl p-5 mb-5 border border-white/5 relative overflow-hidden">
                        <div className="text-[11px] text-slate-500 mb-3 flex justify-between relative z-10">
                            <span>← 14일 전</span><span className="text-[#c8962a] font-bold">오늘</span><span>7일 후 →</span>
                        </div>
                        <svg width="100%" viewBox={`0 0 ${W} ${H}`} preserveAspectRatio="none">
                            <line x1="0" y1={H/2} x2={W} y2={H/2} stroke="rgba(255,255,255,0.1)" strokeWidth="1"/>
                            <line x1={x(14)} y1="0" x2={x(14)} y2={H} stroke="rgba(255,255,255,0.2)" strokeWidth="1.5" strokeDasharray="4,3"/>
                            {Object.entries(cycles).map(([k,c]) => <path key={k} d={wavePath(c.v)} fill="none" stroke={c.color} strokeWidth="2.5" opacity="0.85"/>)}
                            {Object.entries(cycles).map(([k,c]) => <circle key={`dot-${k}`} cx={x(14)} cy={y(c.v)} r="4.5" fill={c.color} stroke="#1e293b" strokeWidth="2"/>)}
                        </svg>
                    </div>
                    <div className="flex gap-3">
                        {Object.entries(cycles).map(([k,c]) => (
                            <div key={k} className="flex-1 p-3 rounded-2xl text-center border border-white/5 bg-white/5">
                                <div style={{ color: c.color }} className="text-[11px] font-bold mb-1.5">{lang==='en'?c.en:c.ko}</div>
                                <div className="text-2xl font-black text-slate-100 playfair">{Math.round((c.v + 1) / 2 * 100)}</div>
                            </div>
                        ))}
                    </div>
                </div>
            );
        };
```

다음으로 바꾼다:

```jsx
        const WaveChart = ({ bio, lang, theme }) => {
            const isLight = theme === 'light';
            const gridColor = isLight ? 'rgba(0,0,0,0.1)' : 'rgba(255,255,255,0.1)';
            const todayLineColor = isLight ? 'rgba(0,0,0,0.25)' : 'rgba(255,255,255,0.2)';
            const dotStroke = isLight ? '#f7f5f2' : '#1e293b';
            const cycles = {
                physical: { v: bio.physical/100 * 2 - 1, color:'#0d9488', ko:'신체', en:'Physical' },
                emotional: { v: bio.emotional/100 * 2 - 1, color:'#f59e0b', ko:'감성', en:'Emotional' },
                intellectual: { v: bio.intellectual/100 * 2 - 1, color:'#3b82f6', ko:'지성', en:'Intellectual' }
            };
            const W = 320, H = 80, DAYS = 21;
            const x = (i) => (i / (DAYS-1)) * W;
            const y = (v) => H/2 - v * (H/2 - 8);
            const wavePath = (vTarget) => {
                const pts = Array.from({length:DAYS}, (_,i) => [x(i), y(Math.sin((i/(DAYS-1) * Math.PI * 4) + Math.asin(vTarget)))]);
                return 'M ' + pts.map(p=>p.join(',')).join(' L ');
            };

            return (
                <div>
                    <div className="bg-black/5 dark:bg-black/40 rounded-3xl p-5 mb-5 border border-black/5 dark:border-white/5 relative overflow-hidden">
                        <div className="text-[11px] text-slate-400 dark:text-slate-500 mb-3 flex justify-between relative z-10">
                            <span>← 14일 전</span><span className="text-[#c8962a] font-bold">오늘</span><span>7일 후 →</span>
                        </div>
                        <svg width="100%" viewBox={`0 0 ${W} ${H}`} preserveAspectRatio="none">
                            <line x1="0" y1={H/2} x2={W} y2={H/2} stroke={gridColor} strokeWidth="1"/>
                            <line x1={x(14)} y1="0" x2={x(14)} y2={H} stroke={todayLineColor} strokeWidth="1.5" strokeDasharray="4,3"/>
                            {Object.entries(cycles).map(([k,c]) => <path key={k} d={wavePath(c.v)} fill="none" stroke={c.color} strokeWidth="2.5" opacity="0.85"/>)}
                            {Object.entries(cycles).map(([k,c]) => <circle key={`dot-${k}`} cx={x(14)} cy={y(c.v)} r="4.5" fill={c.color} stroke={dotStroke} strokeWidth="2"/>)}
                        </svg>
                    </div>
                    <div className="flex gap-3">
                        {Object.entries(cycles).map(([k,c]) => (
                            <div key={k} className="flex-1 p-3 rounded-2xl text-center border border-black/5 dark:border-white/5 bg-black/5 dark:bg-white/5">
                                <div style={{ color: c.color }} className="text-[11px] font-bold mb-1.5">{lang==='en'?c.en:c.ko}</div>
                                <div className="text-2xl font-black text-slate-800 dark:text-slate-100 playfair">{Math.round((c.v + 1) / 2 * 100)}</div>
                            </div>
                        ))}
                    </div>
                </div>
            );
        };
```

- [ ] **Step 2: `HomeTab`에서 `WaveChart` 호출 시 `theme` 전달, `HomeTab` 시그니처 갱신**

다음 줄을 찾는다:

```jsx
        const HomeTab = ({ mbti, lang }) => {
```

다음으로 바꾼다:

```jsx
        const HomeTab = ({ mbti, lang, theme }) => {
```

이어서 다음 줄을 찾는다:

```jsx
                            <WaveChart bio={bio} lang={lang} />
```

다음으로 바꾼다:

```jsx
                            <WaveChart bio={bio} lang={lang} theme={theme} />
```

- [ ] **Step 3: `App`에서 `HomeTab` 호출 시 `theme` 전달**

다음 줄을 찾는다:

```jsx
                                {activeTab === 'home' && <HomeTab mbti={mbti} lang={lang} />}
```

다음으로 바꾼다:

```jsx
                                {activeTab === 'home' && <HomeTab mbti={mbti} lang={lang} theme={theme} />}
```

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: WaveChart 라이트/다크 SVG 색상 대응"
```

---

### Task 8: PourAnimation 테마 대응 SVG 색상

**Files:**
- Modify: `index.html` (`PourAnimation` 컴포넌트 — `CurateTab` 호출부는 Task 6 Step 4에서 이미 `theme` prop을 넘기도록 변경됨)

**Interfaces:**
- Consumes: Task 6에서 이미 연결된 `<PourAnimation color={result.whiskey.color} theme={theme} />` 호출.
- Produces: 없음 (leaf 변경)

- [ ] **Step 1: `PourAnimation`에 `theme` prop과 색상 분기 추가**

다음 블록을 찾는다:

```jsx
        const PourAnimation = ({ color, onComplete }) => {
            const [fill, setFill] = useState(0);
            const [stream, setStream] = useState(true);
            const rafRef = useRef(null);

            useEffect(() => {
                let start = null;
                const step = (ts) => {
                    if (!start) start = ts;
                    const p = Math.min((ts - start) / 1400, 1);
                    setFill((1 - Math.pow(1 - p, 3)) * 62);
                    if (p < 1) rafRef.current = requestAnimationFrame(step);
                    else { setStream(false); setTimeout(() => onComplete?.(), 400); }
                };
                rafRef.current = requestAnimationFrame(step);
                return () => cancelAnimationFrame(rafRef.current);
            }, [color]);

            const liquidY = 93 - (fill / 100) * 75;
            return (
                <div className="relative inline-block mt-6 scale-110">
                    <svg width="180" height="150" viewBox="0 0 180 150" fill="none">
                        <defs>
                            <linearGradient id="glassBodyGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                                <stop offset="0%" stopColor="white" stopOpacity="0.04"/><stop offset="38%" stopColor="white" stopOpacity="0.15"/><stop offset="100%" stopColor="white" stopOpacity="0.02"/>
                            </linearGradient>
                            <clipPath id="glassClip"><path d="M25 14 L22 93 Q22 103 35 103 L145 103 Q158 103 158 93 L155 14 Z"/></clipPath>
                            <radialGradient id="iceG2" cx="40%" cy="35%" r="55%"><stop offset="0%" stopColor="rgba(255,255,255,0.78)"/><stop offset="100%" stopColor="rgba(200,235,255,0.22)"/></radialGradient>
                        </defs>
                        {stream && (
                            <g>
                                <path d={`M122 18 Q124 ${liquidY*0.7} 128 ${liquidY}`} stroke={color} strokeWidth="5" strokeOpacity="0.7" strokeLinecap="round" fill="none" style={{ filter:`drop-shadow(0 0 4px ${color}aa)` }} />
                                {fill > 5 && (
                                    <g style={{ animation:'splash 0.3s ease infinite' }}>
                                        <circle cx="126" cy={liquidY-3} r="2.5" fill={color} opacity="0.6"/><circle cx="133" cy={liquidY-5} r="1.5" fill={color} opacity="0.4"/>
                                    </g>
                                )}
                            </g>
                        )}
                        <path d="M25 14 L22 93 Q22 103 35 103 L145 103 Q158 103 158 93 L155 14 Z" fill="url(#glassBodyGrad)" stroke="rgba(255,255,255,0.2)" strokeWidth="1.5"/>
                        <g clipPath="url(#glassClip)">
                            <rect x="22" y={liquidY} width="136" height={93-liquidY} fill={color} opacity="0.7" />
                            {fill > 3 && <ellipse cx="90" cy={liquidY} rx="62" ry="3" fill={color} opacity="0.4" style={{ animation:'wave 1.8s ease-in-out infinite' }} />}
                        </g>
                        {fill > 30 && (
                            <g clipPath="url(#glassClip)" style={{ opacity: Math.min((fill-30)/20, 1) }}>
                                <rect x="66" y="68" width="46" height="30" rx="6" fill="url(#iceG2)" opacity="0.6"/>
                                <rect x="66" y="68" width="46" height="30" rx="6" fill="none" stroke="rgba(255,255,255,0.3)" strokeWidth="0.75"/>
                            </g>
                        )}
                        <rect x="25" y="14" width="130" height="8" rx="4" fill="rgba(255,255,255,0.1)"/>
                    </svg>
                    <div style={{ position:'absolute', inset:-16, background:`radial-gradient(circle at 50% 75%, ${color}30 0%, transparent 60%)`, pointerEvents:'none', opacity: fill / 62 }}/>
                </div>
            );
        };
```

다음으로 바꾼다:

```jsx
        const PourAnimation = ({ color, onComplete, theme }) => {
            const [fill, setFill] = useState(0);
            const [stream, setStream] = useState(true);
            const rafRef = useRef(null);
            const isLight = theme === 'light';
            const glassStroke = isLight ? 'rgba(0,0,0,0.15)' : 'rgba(255,255,255,0.2)';
            const glassGrad = isLight
                ? [['0%','black',0.02], ['38%','black',0.08], ['100%','black',0.01]]
                : [['0%','white',0.04], ['38%','white',0.15], ['100%','white',0.02]];
            const rimHighlight = isLight ? 'rgba(0,0,0,0.06)' : 'rgba(255,255,255,0.1)';

            useEffect(() => {
                let start = null;
                const step = (ts) => {
                    if (!start) start = ts;
                    const p = Math.min((ts - start) / 1400, 1);
                    setFill((1 - Math.pow(1 - p, 3)) * 62);
                    if (p < 1) rafRef.current = requestAnimationFrame(step);
                    else { setStream(false); setTimeout(() => onComplete?.(), 400); }
                };
                rafRef.current = requestAnimationFrame(step);
                return () => cancelAnimationFrame(rafRef.current);
            }, [color]);

            const liquidY = 93 - (fill / 100) * 75;
            return (
                <div className="relative inline-block mt-6 scale-110">
                    <svg width="180" height="150" viewBox="0 0 180 150" fill="none">
                        <defs>
                            <linearGradient id="glassBodyGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                                {glassGrad.map(([offset, stopColor, stopOpacity]) => <stop key={offset} offset={offset} stopColor={stopColor} stopOpacity={stopOpacity}/>)}
                            </linearGradient>
                            <clipPath id="glassClip"><path d="M25 14 L22 93 Q22 103 35 103 L145 103 Q158 103 158 93 L155 14 Z"/></clipPath>
                            <radialGradient id="iceG2" cx="40%" cy="35%" r="55%"><stop offset="0%" stopColor="rgba(255,255,255,0.78)"/><stop offset="100%" stopColor="rgba(200,235,255,0.22)"/></radialGradient>
                        </defs>
                        {stream && (
                            <g>
                                <path d={`M122 18 Q124 ${liquidY*0.7} 128 ${liquidY}`} stroke={color} strokeWidth="5" strokeOpacity="0.7" strokeLinecap="round" fill="none" style={{ filter:`drop-shadow(0 0 4px ${color}aa)` }} />
                                {fill > 5 && (
                                    <g style={{ animation:'splash 0.3s ease infinite' }}>
                                        <circle cx="126" cy={liquidY-3} r="2.5" fill={color} opacity="0.6"/><circle cx="133" cy={liquidY-5} r="1.5" fill={color} opacity="0.4"/>
                                    </g>
                                )}
                            </g>
                        )}
                        <path d="M25 14 L22 93 Q22 103 35 103 L145 103 Q158 103 158 93 L155 14 Z" fill="url(#glassBodyGrad)" stroke={glassStroke} strokeWidth="1.5"/>
                        <g clipPath="url(#glassClip)">
                            <rect x="22" y={liquidY} width="136" height={93-liquidY} fill={color} opacity="0.7" />
                            {fill > 3 && <ellipse cx="90" cy={liquidY} rx="62" ry="3" fill={color} opacity="0.4" style={{ animation:'wave 1.8s ease-in-out infinite' }} />}
                        </g>
                        {fill > 30 && (
                            <g clipPath="url(#glassClip)" style={{ opacity: Math.min((fill-30)/20, 1) }}>
                                <rect x="66" y="68" width="46" height="30" rx="6" fill="url(#iceG2)" opacity="0.6"/>
                                <rect x="66" y="68" width="46" height="30" rx="6" fill="none" stroke="rgba(255,255,255,0.3)" strokeWidth="0.75"/>
                            </g>
                        )}
                        <rect x="25" y="14" width="130" height="8" rx="4" fill={rimHighlight}/>
                    </svg>
                    <div style={{ position:'absolute', inset:-16, background:`radial-gradient(circle at 50% 75%, ${color}30 0%, transparent 60%)`, pointerEvents:'none', opacity: fill / 62 }}/>
                </div>
            );
        };
```

- [ ] **Step 2: 커밋**

```bash
git add index.html
git commit -m "feat: PourAnimation 라이트/다크 SVG 색상 대응"
```

---

### Task 9: 무드 12개로 확장

**Files:**
- Modify: `index.html` (`MOODS` 배열)

**Interfaces:**
- Consumes: 없음
- Produces: `MOODS` 배열에 `id: 'nostalgia'`, `id: 'pride'` 2개 항목 추가(총 12개). `CurateTab.run()`의 스코어링 로직은 `flavorProfile` 태그가 `SPIRITS_DB`에 이미 존재하는 태그만 사용하므로 그대로 작동한다.

- [ ] **Step 1: `MOODS` 배열 마지막 항목 뒤에 2개 추가**

다음 줄을 찾는다(배열의 마지막 항목):

```jsx
            { id: 'boredom', label: { ko: '무료함', en: 'Boredom' }, desc: { ko: "평범하고 심심한 하루네요. 무료함을 기분 좋게 날려버릴 한 잔 어떠세요?", en: "Dull day. Drink to blow away boredom?" }, expr: { ko: "모든 게 귀찮고 의미 없이 시간만 흘러간 따분한 하루", en: "A dull, annoying day" }, emoji: "🥱", conflicts: ['joy', 'delight', 'anticipation', 'surprise', 'fear', 'anger'], flavorProfile: ['spicy', 'unique', 'cask_strength', 'citrus', 'peppery', 'fresh', 'bold'] }
        ];
```

다음으로 바꾼다:

```jsx
            { id: 'boredom', label: { ko: '무료함', en: 'Boredom' }, desc: { ko: "평범하고 심심한 하루네요. 무료함을 기분 좋게 날려버릴 한 잔 어떠세요?", en: "Dull day. Drink to blow away boredom?" }, expr: { ko: "모든 게 귀찮고 의미 없이 시간만 흘러간 따분한 하루", en: "A dull, annoying day" }, emoji: "🥱", conflicts: ['joy', 'delight', 'anticipation', 'surprise', 'fear', 'anger'], flavorProfile: ['spicy', 'unique', 'cask_strength', 'citrus', 'peppery', 'fresh', 'bold'] },
            { id: 'nostalgia', label: { ko: '그리움', en: 'Nostalgia' }, desc: { ko: "지난 추억이 그리워지는 날이에요. 옛 생각에 잠겨 한 잔 어떠세요?", en: "A day of missing the past. A drink to reminisce?" }, expr: { ko: "옛 생각이 자꾸 나서 마음이 아련해지는 하루", en: "A wistful day full of old memories" }, emoji: "🍂", conflicts: ['anticipation', 'surprise'], flavorProfile: ['oak', 'sherry', 'rich', 'classic', 'nutty'] },
            { id: 'pride', label: { ko: '뿌듯함', en: 'Pride' }, desc: { ko: "스스로가 뿌듯한 날이에요. 자축하는 의미로 한 잔 어떠세요?", en: "A day of self-pride. A drink to celebrate yourself?" }, expr: { ko: "해낸 일에 스스로가 대견하고 뿌듯한 하루", en: "A day I'm proud of what I accomplished" }, emoji: "😎", conflicts: ['sadness', 'fear', 'disgust'], flavorProfile: ['bold', 'oak', 'cask_strength', 'rich', 'complex'] }
        ];
```

- [ ] **Step 2: 구조적 확인**

`grep -c "id: '" index.html` 등으로 `MOODS` 배열 항목 수를 세어 12개인지 확인하거나, `node -e` 없이 grep으로 새 id 2개(`nostalgia`, `pride`)가 정확히 한 번씩 나타나는지 확인한다.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: 무드 2종(그리움, 뿌듯함) 추가해 12개로 확장"
```

---

### Task 10: 전체 회귀 확인 (라이트/다크 × 모바일/데스크톱, 언어/무드 검증)

**Files:**
- 변경 없음 (검증 전용 태스크)

**Interfaces:**
- Consumes: Task 1~9의 모든 변경사항
- Produces: 없음

- [ ] **Step 1: 언어 변경 UI 위치 확인**

`preview_snapshot`/`preview_eval`로 온보딩 화면과 데스크톱(1280px) 상단 네비에 EN/KO 버튼이 더 이상 없는지 확인. 프로필 탭에는 여전히 언어 변경 버튼이 있고 정상 동작하는지 확인.

- [ ] **Step 2: 시스템 라이트 선호 시 기본 테마 확인**

`preview_resize`의 `colorScheme: 'light'` 옵션으로 라이트 선호를 에뮬레이트하고, `localStorage.clear()` 후 새로고침 → 스플래시/온보딩이 라이트로 뜨는지 `preview_inspect`(`.app-container`의 `background-color`, `body`의 computed `background-color`)로 확인.

- [ ] **Step 3: 프로필에서 테마 토글 동작 확인**

MBTI 설정 후 프로필 탭 진입 → 테마 변경 버튼 클릭 → `document.documentElement.classList.contains('dark')`가 토글되는지, 홈/큐레이션 탭으로 이동해 전체 화면 색상이 바뀌는지 `preview_eval`/`preview_screenshot`으로 확인. 새로고침 후에도 선택한 테마가 유지되는지(`localStorage.whiskey_theme`) 확인.

- [ ] **Step 4: 라이트 모드에서 화면별 대비 확인 (스크린샷)**

라이트 모드 상태에서 온보딩 → 퀴즈 → 홈 → 큐레이션(무드 선택 화면, 결과 화면) → 프로필까지 각 화면을 `preview_screenshot`으로 확인해 텍스트가 배경에 묻히는 곳이 없는지 육안 확인. 특히 `WaveChart`(홈)와 `PourAnimation`(큐레이션 결과)이 라이트 배경에서도 선/유리잔 형태가 보이는지 확인.

- [ ] **Step 5: 공유 카드가 라이트 모드에서도 다크 톤 유지하는지 확인**

라이트 모드에서 큐레이션 결과 화면 → 공유하기 클릭 → `preview_screenshot`으로 캔버스 이미지가 여전히 다크 브랜드 톤(짙은 배경, 밝은 텍스트)인지 확인.

- [ ] **Step 6: 무드 12개 노출 및 스코어링 확인**

큐레이션 1단계에서 무드 카드가 12개(데스크톱 3행×4열, 모바일 6행×2열) 모두 보이는지 `preview_eval`로 `document.querySelectorAll` 개수 확인. '그리움' 또는 '뿌듯함'을 선택해 2단계 블렌딩 화면과 4단계 결과 화면까지 에러 없이 도달하는지 확인(`preview_console_logs`로 콘솔 에러 없음 확인).

- [ ] **Step 7: 다크 모드 + 데스크톱 레이아웃 회귀 확인**

`colorScheme: 'dark'`로 되돌리고 375px/1280px 양쪽에서 스크린샷을 찍어 이전 데스크톱 레이아웃 작업(그리드, 상단 네비) 결과와 시각적으로 달라진 점이 없는지(테마 관련 색상 변경 외) 확인.

- [ ] **Step 8: 문제 발견 시 처리**

발견된 문제는 해당 태스크로 돌아가 수정 후 그 태스크의 커밋을 새로 추가한다. 이 태스크 자체는 코드 변경이 없으므로 문제가 없다면 커밋 없이 종료한다.
