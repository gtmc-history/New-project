# game-html-template.md — 게임 HTML 템플릿

## 목적

이 문서는 새 게임을 만들 때 복사해서 쓰는 HTML 구조 템플릿이다.
실제 게임 파일이 아니며, `games/게임폴더명/index.html`을 만들 때 이 구조를 기준으로 시작한다.

---

## 사용 전 필수 확인

- [ ] `게임 제목`을 실제 제목으로 바꿨는가?
- [ ] 학생의 역할과 시대 상황이 첫 화면에 보이는가?
- [ ] GA4 코드가 들어 있는가? (측정 ID `G-G350VXSFGE` 확인)
- [ ] 툴팁 시스템이 포함되어 있는가? (TIPS 딕셔너리에 게임별 용어 추가)
- [ ] 저작권 푸터가 결과 화면에만 들어 있는가?
- [ ] 태블릿에서 버튼 터치 영역이 충분한가? (최소 48×48px)
- [ ] 역사 오류 패턴을 검토했는가? (project-context.md 참고)
- [ ] 시대에 맞는 CSS 색상 변수를 골랐는가?

---

## 기본 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
  <title>게임 제목</title>

  <!-- Google Fonts: 제목용 Noto Serif KR, 본문용 Noto Sans KR -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@400;700&family=Noto+Sans+KR:wght@300;400;500&display=swap" rel="stylesheet">

  <!-- GA4: 모든 게임에 필수 -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-G350VXSFGE"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-G350VXSFGE');
  </script>

  <style>
    /* ─── 기본 3색 변수 ─── */
    :root {
      --ink:   #090b0d;   /* 먹색 — 글자, 깊은 배경 */
      --gold:  #c9a052;   /* 금빛 — 강조, 선택, 보상 */
      --paper: #f5f0e8;   /* 한지색 — 읽기 편한 배경 */
      --gold2: #e8c97a;   /* 금빛 밝은 버전 */
      --red:   #8b1c1c;   /* 붉은 인장 — 경고, hover */
    }

    /*
    ─── 시대별 보조 색상 (해당 시대 변수를 :root에 추가해서 사용) ───

    고려시대:
      --era-accent: #6b9e8a;   청자 비색
      --era-bg:     #1a1208;   먹빛 바탕
      --era-surface:#2d1f0e;   고동색 표면

    조선시대:
      --era-accent: #1a3a5c;   관복 감색
      --era-bg:     #f5f0e8;   백자 유백색
      --era-surface:#ede5d8;   한지 표면

    개항기·일제강점기:
      --era-accent: #003087;   군청색
      --era-bg:     #f0ece4;   낡은 사진 배경
      --era-surface:#e8e0d4;   세피아 표면

    전쟁·침략기:
      --era-accent: #cc3300;   불꽃 적색
      --era-bg:     #0d0d0d;   칠흑
      --era-surface:#1a1a1a;   연기빛 표면
    */

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Noto Sans KR', sans-serif;
      background: var(--paper);
      color: var(--ink);
      min-height: 100dvh;
    }

    /* ─── 레이아웃 ─── */
    #app {
      max-width: 640px;
      margin: 0 auto;
      padding: 24px 16px 60px;
    }

    /* ─── 버튼 기본 (터치 타깃 최소 48px) ─── */
    .btn {
      display: block;
      width: 100%;
      min-height: 48px;
      padding: 12px 16px;
      border: none;
      border-radius: 4px;
      font-family: 'Noto Serif KR', serif;
      font-size: 16px;
      font-weight: 700;
      cursor: pointer;
      transition: transform 0.15s ease, box-shadow 0.15s ease;
    }
    .btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }
    .btn:active {
      transform: translateY(0) scale(0.97);
    }
    .btn-primary {
      background: var(--gold);
      color: var(--ink);
    }
    .btn-outline {
      background: transparent;
      border: 1.5px solid var(--gold);
      color: var(--ink);
    }

    /* ─── 툴팁 시스템 ─── */
    .tt {
      border-bottom: 1.5px dashed var(--gold);
      cursor: pointer;
      position: relative;
      white-space: nowrap;
      color: inherit;
      transition: border-color 0.15s;
    }
    .tt:hover { border-bottom-color: var(--red); }

    #tt-bubble {
      position: fixed;
      z-index: 9000;
      max-width: 240px;
      background: var(--ink);
      border: 1px solid rgba(184,134,11,0.6);
      border-radius: 4px;
      padding: 10px 13px;
      box-shadow: 0 6px 24px rgba(0,0,0,0.6);
      pointer-events: none;
      opacity: 0;
      transform: translateY(4px);
      transition: opacity 0.18s, transform 0.18s;
    }
    #tt-bubble.show { opacity: 1; transform: translateY(0); }
    #tt-bubble .tb-word {
      font-size: 12px; font-weight: 700;
      color: var(--gold2); margin-bottom: 5px;
    }
    #tt-bubble .tb-desc {
      font-size: 12px;
      color: rgba(240,228,192,0.85);
      line-height: 1.7;
    }

    /* ─── 저작권 푸터 (결과 화면에만) ─── */
    .copyright {
      font-size: 12px;
      color: rgba(120,100,80,0.4);
      text-align: center;
      margin-top: 32px;
    }
  </style>
</head>
<body>

<div id="app">

  <!-- 시작 화면 -->
  <section id="scene-intro" aria-label="시작 화면">
    <h1>게임 제목</h1>
    <p>학생의 역할과 역사적 상황을 짧게 설명합니다.<br>당신은 [역할]입니다. [시대·배경].</p>
    <button class="btn btn-primary" onclick="startGame()">시작하기 →</button>
  </section>

  <!-- 선택 장면 (반복 사용) -->
  <section id="scene-choice" aria-label="선택 장면" hidden>
    <h2>당신의 선택은?</h2>
    <p id="choice-desc">상황 설명이 여기 들어갑니다.</p>
    <div id="choice-buttons"></div>
  </section>

  <!-- 결과 화면 -->
  <section id="scene-result" aria-label="결과 화면" hidden>
    <h2>선택의 결과</h2>
    <p id="result-desc">결과와 역사 해설을 보여줍니다.</p>

    <!-- 결과 공유 버튼 -->
    <button class="btn btn-outline" onclick="copyResult()">
      📋 결과 복사하기 (패들렛 공유용)
    </button>

    <button class="btn btn-primary" onclick="restartGame()" style="margin-top:8px">
      다시 하기
    </button>

    <!-- 저작권 푸터: 결과 화면에만 표시 -->
    <p class="copyright">© 찰리쌤 · hischarlie.tistory.com</p>
  </section>

</div>

<!-- 툴팁 말풍선 -->
<div id="tt-bubble"><div class="tb-word"></div><div class="tb-desc"></div></div>

<script>
// ═══════════════════════════════════════
// 툴팁 시스템 (수정하지 말 것)
// ═══════════════════════════════════════

// TIPS 딕셔너리: 이 게임에서 사용할 용어를 여기에 추가
// 형식: '용어': '한글명(한자) — 한 줄 설명 (40자 이내)'
const TIPS = {
  // 예시 (게임에 맞게 교체)
  '음서': '음서(蔭敍) — 고위 관리의 자손에게 시험 없이 관직을 주는 제도.',
  '제술과': '제술과(製述科) — 고려 과거 시험 중 문관 최고 시험.',
  // 게임별 용어를 여기에 추가하세요
};

// ⚠️ 주의: tt()는 HTML 문자열을 반환한다.
//    JS 템플릿 리터럴 ${} 안에 직접 넣으면 파싱 오류가 생긴다.
//    innerHTML에 할당하거나 HTML 안에 직접 쓸 것.
function tt(word, tipKey) {
  const tip = TIPS[tipKey || word] || '';
  if (!tip) return word;
  return `<span class="tt" data-tip="${encodeURIComponent(tip)}" data-word="${word}">${word}</span>`;
}

// 툴팁 이벤트 (수정 금지)
const bubble = document.getElementById('tt-bubble');
let bubbleTimer = null;
document.addEventListener('click', function(e) {
  const el = e.target.closest('.tt');
  if (el) {
    e.stopPropagation();
    bubble.querySelector('.tb-word').textContent = el.dataset.word;
    bubble.querySelector('.tb-desc').textContent = decodeURIComponent(el.dataset.tip);
    const rect = el.getBoundingClientRect();
    const bw = 240;
    const left = Math.max(8, Math.min(rect.left + rect.width/2 - bw/2, window.innerWidth - bw - 8));
    bubble.style.width = bw + 'px';
    bubble.style.left  = left + 'px';
    bubble.style.top   = '-9999px';
    bubble.classList.add('show');
    const bubH = bubble.offsetHeight;
    bubble.style.top = (rect.top - bubH - 14 < 8)
      ? (rect.bottom + 10) + 'px'
      : (rect.top - bubH - 14) + 'px';
    if (bubbleTimer) clearTimeout(bubbleTimer);
    bubbleTimer = setTimeout(() => bubble.classList.remove('show'), 4000);
  } else {
    bubble.classList.remove('show');
  }
});

// ═══════════════════════════════════════
// GA4 이벤트 트래킹 헬퍼
// ═══════════════════════════════════════
// 게임 시작, 선택, 완료 시점에 아래 함수를 호출한다.
// event_name은 영문 snake_case로 작성.

function gaEvent(event_name, params = {}) {
  if (typeof gtag === 'function') {
    gtag('event', event_name, params);
  }
}

// 사용 예시:
// gaEvent('game_start', { game_id: 'ganghwa' });
// gaEvent('choice_made', { scene: 1, choice: 'A' });
// gaEvent('game_complete', { result: 'success' });

// ═══════════════════════════════════════
// 게임 로직 (여기서부터 게임별로 작성)
// ═══════════════════════════════════════

function show(id) {
  ['scene-intro', 'scene-choice', 'scene-result'].forEach(s => {
    document.getElementById(s).hidden = (s !== id);
  });
  window.scrollTo(0, 0);
}

function startGame() {
  gaEvent('game_start');
  // TODO: 게임 로직 작성
  show('scene-choice');
}

function restartGame() {
  // TODO: 상태 초기화
  show('scene-intro');
}

function copyResult() {
  const text = [
    '게임 제목 — 결과',
    '',
    '내 선택: (선택 내용)',
    '결과: (결과 내용)',
    '',
    '© 찰리쌤 · hischarlie.tistory.com',
  ].join('\n');

  if (navigator.clipboard && window.isSecureContext) {
    navigator.clipboard.writeText(text).then(() => alert('복사되었습니다!'));
  } else {
    const ta = document.createElement('textarea');
    ta.value = text;
    ta.style.position = 'fixed'; ta.style.left = '-9999px';
    document.body.appendChild(ta);
    ta.select(); ta.setSelectionRange(0, 99999);
    try { document.execCommand('copy'); alert('복사되었습니다!'); }
    catch(e) { alert('복사 실패. 직접 선택해서 복사해주세요.'); }
    document.body.removeChild(ta);
  }
}
</script>
</body>
</html>
```

---

## 참고: 이 템플릿에 포함된 것

| 항목 | 포함 여부 | 비고 |
|------|-----------|------|
| GA4 코드 | ✅ | 측정 ID `G-G350VXSFGE` |
| GA4 이벤트 트래킹 헬퍼 | ✅ | `gaEvent()` 함수 |
| Google Fonts (Noto Serif/Sans KR) | ✅ | system-ui 금지 |
| 기본 3색 CSS 변수 | ✅ | 금빛·먹색·한지색 |
| 시대별 보조 색상 주석 | ✅ | 골라서 추가 |
| 툴팁 시스템 전체 | ✅ | CSS + JS + 이벤트 |
| 저작권 푸터 | ✅ | 결과 화면에만 |
| 결과 복사 버튼 | ✅ | `navigator.clipboard` |
| 뷰포트 `maximum-scale=1.0` | ✅ | 태블릿 확대 방지 |
| 버튼 최소 48px | ✅ | 터치 타깃 기준 |
