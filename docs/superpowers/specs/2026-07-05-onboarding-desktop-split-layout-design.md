# 온보딩 3화면 데스크톱 좌우 분할 레이아웃 설계

## 배경
현재 `Splash`(초기 화면), `Onboarding`(MBTI 테스트 시작/직접 선택 분기), `Quiz`/`DirectSelect`(실제 MBTI 확인) 4개 컴포넌트는 이전 데스크톱 반응형 작업에서 의도적으로 데스크톱에서도 모바일과 동일한 좁은 카드형(`430px`)을 유지하도록 결정했었다(`App`의 `.app-container.is-main` 클래스가 `appState==='main'`일 때만 붙어 데스크톱 전체폭 스타일이 적용되고, 그 외 상태는 계속 좁은 카드형). 이번 작업은 그 결정을 바꿔 데스크톱(≥1024px)에서 이 3화면을 좌우 분할 레이아웃으로 넓게 활용한다. 모바일(<1024px)은 완전히 그대로 유지한다.

## 범위 밖
- `appState==='main'`(홈/큐레이션/프로필, 상단 데스크톱 네비) 레이아웃은 변경하지 않는다.
- `Splash`/`Onboarding`/`Quiz`/`DirectSelect` 개별 컴포넌트의 내부 마크업, 로직, MBTI 스코어링은 변경하지 않는다 — 오직 이들을 감싸는 `App`의 wrapper 구조와 새 CSS만 추가한다.
- 언어 변경 UI는 추가하지 않는다(기존 정책: 프로필에만 존재).

## 구조

### 1. `App`의 return JSX 재구성
현재 `App`은 `appState`에 따라 4개 컴포넌트 중 하나를 `.app-container` 안에 직접 렌더링한다(`index.html:997-1020`). 이를 다음과 같이 재구성한다:

- `appState !== 'main'`일 때: `.onboarding-shell`로 감싸고, 그 안에 (a) `.onboarding-brand-panel`(고정 브랜드 패널), (b) 기존과 동일한 `.app-container`(Splash/Onboarding/Quiz/DirectSelect 렌더링, 내용 변경 없음)를 나란히 둔다.
- `appState === 'main'`일 때: 기존 그대로 `.app-container.is-main`만 렌더링(변경 없음).

### 2. `.onboarding-brand-panel` 내용
Splash의 브랜드 정체성(🥃 이모지, "WHISKY MBTI" 플레이페어 타이틀)과 Onboarding의 카피(`TEXT[lang].onboard_title`, `TEXT[lang].onboard_sub` — 이미 존재하는 텍스트, 새 텍스트 키 추가 없음)를 재사용해 3화면 내내 고정된 패널을 구성한다. `lang`에 따라 텍스트만 바뀐다(토글 버튼 없음).

### 3. CSS
```css
.onboarding-shell {
    display: block;
}
.onboarding-brand-panel {
    display: none;
}

@media (min-width: 1024px) {
    .onboarding-shell {
        display: flex;
        align-items: stretch;
        min-height: 100vh;
        width: 100%;
    }
    .onboarding-brand-panel {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        flex: 1;
        padding: 64px;
        text-align: center;
        background-image: radial-gradient(ellipse at 50% 30%, rgba(200,150,42,0.12) 0%, transparent 60%);
    }
}
```
- 모바일: `.onboarding-shell`은 스타일 없는 `display:block` 래퍼, `.onboarding-brand-panel`은 `display:none` — 기존 `.app-container`만 그대로 보여 현재와 완전히 동일.
- 데스크톱(`min-width:1024px`, 기존 `lg:` 컨벤션과 동일 값): 좌우 flex, 왼쪽은 브랜드 패널(`flex:1`), 오른쪽은 기존 `.app-container`(430px 고정폭, 변경 없음)가 자연스럽게 중앙 정렬되어 나란히 배치된다.

## 컴포넌트 트리 (변경 후)
```jsx
{appState !== 'main' ? (
    <div className="onboarding-shell">
        <div className="onboarding-brand-panel">
            <div className="text-[4rem] mb-6">🥃</div>
            <h1 className="playfair text-4xl font-black text-slate-900 dark:text-white tracking-widest mb-4">WHISKY MBTI</h1>
            <h2 className="text-2xl font-bold text-slate-700 dark:text-slate-200 mb-4">{TEXT[lang].onboard_title}</h2>
            <p className="text-slate-500 dark:text-slate-400 text-lg leading-relaxed whitespace-pre-line max-w-md">{TEXT[lang].onboard_sub}</p>
        </div>
        <div className="app-container">
            {appState === 'splash' && <Splash onFinish={...} />}
            {appState === 'onboard' && <Onboarding ... />}
            {appState === 'quiz' && <Quiz ... />}
            {appState === 'direct' && <DirectSelect ... />}
        </div>
    </div>
) : (
    <div className="app-container is-main">
        {/* 기존 main 상태 렌더링, 변경 없음 */}
    </div>
)}
```

## 테스트 계획
- 자동화 테스트 없음 — 브라우저로 확인
- 모바일(375px): 스플래시/온보딩/퀴즈/직접선택 화면이 기존과 픽셀 단위로 동일하게 보이는지 확인(회귀 없음), `.onboarding-brand-panel`이 보이지 않는지 확인
- 데스크톱(1280px, 1440px): 좌측에 고정 브랜드 패널, 우측에 430px 카드가 나란히 보이는지 확인. 스플래시 → 온보딩 → 퀴즈(또는 직접선택) → 결과(main 진입) 전체 플로우를 진행하며 브랜드 패널이 3화면 내내 유지되고 `main` 진입 시 사라지는지 확인
- 라이트/다크 모드 양쪽에서 브랜드 패널 텍스트/배경 대비 확인
- 한국어/영어 언어 전환 시(프로필에서 변경 후 재방문 등) 브랜드 패널 텍스트가 올바르게 바뀌는지 확인
