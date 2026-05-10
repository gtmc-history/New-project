# tooltip-system-snippet.md — 툴팁 시스템 코드 조각

## 목적

어려운 역사 용어 옆에 짧은 설명을 붙일 때 사용하는 코드 조각이다.
학생이 용어를 모르면 게임 흐름이 끊기므로, 별도 검색 없이 그 자리에서 뜻을 확인할 수 있게 한다.

> ⚠️ 이 시스템은 **왕의 결재함 v3**를 레퍼런스 구현체로 확정했다.
> 다른 방식으로 임의 구현하지 말고 이 코드를 그대로 복사해서 쓸 것.

---

## 툴팁을 달아야 하는 용어 기준

다음에 해당하면 반드시 툴팁을 추가한다.

- 한자어로 된 제도·관직명 (예: 음서, 제술과, 의정부, 6조 직계제)
- 고등학생에게 생소할 수 있는 역사 개념어 (예: 풍흉, 삼복, 사병, 외척)
- 게임 내 핵심 행동어 중 한자어 (예: 재가, 환부, 윤허, 반려)
- 게임 안내 문구에 등장하는 시대 용어 (예: 전하, 호조, 병조)

툴팁은 용어 이해를 돕는 용도로만 쓴다. **정답을 바로 알려주는 용도로 쓰지 않는다.**

---

## 설명 작성 기준

- 형식: `한글명(한자) — 한 줄 설명`
- 길이: **40자 이내** (말풍선 크기 기준)
- 어조: 사전체, 개조식 (예: "~하는 제도", "~을 뜻함")
- 한자 병기 시 독음 확인 필수
- 학생이 모를 만한 말은 쉬운 말로 풀어 쓴다

**예시**
```
'음서':   '음서(蔭敍) — 고위 관리 자손에게 시험 없이 관직을 주는 제도.'
'제술과': '제술과(製述科) — 고려 과거 시험 중 문관을 뽑는 최고 시험.'
'재가':   '재가(裁可) — 왕이 의정부의 의결을 그대로 승인하는 것.'
'환부':   '환부(還付) — 왕이 안건을 의정부에 돌려보내 다시 심의하게 하는 것.'
'서경':   '서경(署經) — 관리 임명 전 삼사가 동의 여부를 확인하는 절차.'
```

---

## CSS

`game-html-template.md`의 CSS 안에 그대로 포함되어 있다. 별도로 추가할 필요 없이 템플릿을 복사하면 된다.

```css
/* 툴팁 대상 텍스트 */
.tt {
  border-bottom: 1.5px dashed var(--gold); /* 금색 점선 밑줄 */
  cursor: pointer;
  position: relative;
  white-space: nowrap;
  color: inherit;
  transition: border-color 0.15s;
}
.tt:hover { border-bottom-color: var(--red); }

/* 말풍선 */
#tt-bubble {
  position: fixed;
  z-index: 9000;
  max-width: 240px;
  background: var(--ink);
  border: 1px solid rgba(184,134,11,0.6);
  border-radius: 4px; /* 8px 이상 금지 */
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
```

---

## HTML

말풍선 요소를 `<body>` 끝에 한 번만 넣는다.

```html
<div id="tt-bubble">
  <div class="tb-word"></div>
  <div class="tb-desc"></div>
</div>
```

---

## JavaScript

### 1단계 — TIPS 딕셔너리 작성

게임마다 필요한 용어를 여기에 추가한다.

```js
const TIPS = {
  '음서':   '음서(蔭敍) — 고위 관리 자손에게 시험 없이 관직을 주는 제도.',
  '제술과': '제술과(製述科) — 고려 과거 시험 중 문관을 뽑는 최고 시험.',
  '재가':   '재가(裁可) — 왕이 의정부의 의결을 그대로 승인하는 것.',
  // 게임별 용어를 여기에 추가
};
```

### 2단계 — tt() 헬퍼 함수

HTML 문자열 안에서 툴팁 span을 만들어 주는 함수다.

```js
function tt(word, tipKey) {
  const tip = TIPS[tipKey || word] || '';
  if (!tip) return word;
  return `<span class="tt" data-tip="${encodeURIComponent(tip)}" data-word="${word}">${word}</span>`;
}
```

> ⚠️ **파싱 오류 주의**: JS 템플릿 리터럴 `${}` 표현식 **안에** `tt()`를 넣으면 오류가 생긴다.
> `innerHTML`에 할당하거나 HTML에 직접 `<span class="tt" ...>` 형태로 써야 한다.

**올바른 사용법**
```js
// ✅ innerHTML에 할당
el.innerHTML = `당신은 ${tt('음서')}를 활용할 수 있습니다.`;

// ✅ HTML에 직접 삽입
// <span class="tt" data-tip="..." data-word="음서">음서</span>
```

**잘못된 사용법**
```js
// ❌ 템플릿 리터럴 ${} 안에 span 태그 직접 삽입 — 파싱 오류 발생
const html = `선택: <span class="tt" data-tip="${TIPS['음서']}" data-word="음서">음서</span>`;
//                                              ^^^^^^^^^^^^^ 따옴표 충돌
```

### 3단계 — 이벤트 리스너 (수정 금지)

```js
const bubble = document.getElementById('tt-bubble');
let bubbleTimer = null;

document.addEventListener('click', function(e) {
  const el = e.target.closest('.tt');
  if (el) {
    e.stopPropagation();
    bubble.querySelector('.tb-word').textContent = el.dataset.word;
    bubble.querySelector('.tb-desc').textContent = decodeURIComponent(el.dataset.tip);

    // 위치 계산:
    // 반드시 top:-9999px로 화면 밖에 먼저 렌더링한 뒤 offsetHeight를 측정한다.
    // show 추가 전에 측정해야 정확한 높이를 얻을 수 있다.
    const rect = el.getBoundingClientRect();
    const bw = 240;
    const left = Math.max(8, Math.min(rect.left + rect.width/2 - bw/2, window.innerWidth - bw - 8));
    bubble.style.width = bw + 'px';
    bubble.style.left  = left + 'px';
    bubble.style.top   = '-9999px';
    bubble.classList.add('show');
    const bubH = bubble.offsetHeight;
    // 위쪽 공간이 부족하면 아래에, 충분하면 위에 표시
    bubble.style.top = (rect.top - bubH - 14 < 8)
      ? (rect.bottom + 10) + 'px'
      : (rect.top - bubH - 14) + 'px';

    if (bubbleTimer) clearTimeout(bubbleTimer);
    bubbleTimer = setTimeout(() => bubble.classList.remove('show'), 4000);
  } else {
    bubble.classList.remove('show');
  }
});
```

---

## 전체 동작 흐름 요약

```
학생이 금색 점선 밑줄 단어를 탭
  → tt() 헬퍼가 만든 span의 data-word, data-tip 읽기
  → 말풍선을 top:-9999px에 렌더링해서 높이 측정
  → 위/아래 공간 판단 후 위치 확정
  → opacity 트랜지션으로 자연스럽게 등장
  → 4초 후 자동으로 사라짐
  → 다른 곳 탭하면 즉시 사라짐
```

---

## 빠르게 추가하는 방법 (체크리스트)

새 게임에 툴팁을 추가할 때:

- [ ] `TIPS` 딕셔너리에 이 게임에서 쓸 용어 추가
- [ ] 용어 설명이 40자 이내인지 확인
- [ ] HTML 본문에서 툴팁이 필요한 단어를 `tt()` 또는 `<span class="tt">` 로 감쌈
- [ ] `${}` 안에 `<span>` 직접 삽입하지 않았는지 확인
- [ ] `<body>` 끝에 `#tt-bubble` 요소가 있는지 확인
- [ ] 이벤트 리스너가 한 번만 등록되었는지 확인
- [ ] 툴팁 설명 내용의 역사적 정확성 검토
