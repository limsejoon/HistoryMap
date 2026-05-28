# 유럽 중세 역사 게임 MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `index.html`을 유럽 중세 역사 퀴즈 게임으로 발전시킨다 — 4개 도시 마커, 우측 사이드 패널(사건 3개 + 퀴즈), 좌상단 점수 배지.

**Architecture:** `index.html` 단일 파일에 레이아웃·스타일·로직을 모두 담고, `events.json`에서 도시 데이터를 fetch로 로드한다. 기존 클릭-마커 기능 제거. 추가 라이브러리 없이 Leaflet + 순수 JS(ES6+)만 사용.

**Tech Stack:** Leaflet 1.9.4, OpenStreetMap tiles, Vanilla JS fetch API

**로컬 서버 실행:** `npx serve .` → `http://localhost:3000`

---

### Task 1: events.json 생성

**Files:**
- Create: `events.json`

- [ ] **Step 1: events.json 파일 생성**

```json
{
  "cities": [
    {
      "id": "constantinople",
      "name": "콘스탄티노플",
      "lat": 41.0082,
      "lng": 28.9784,
      "subtitle": "비잔티움 제국 · 동지중해",
      "events": [
        { "year": 330,  "text": "콘스탄티누스 1세, 비잔티움을 새 로마 제국 수도로 재건." },
        { "year": 1054, "text": "대분열 — 가톨릭과 동방정교회 영구 분리." },
        { "year": 1453, "text": "오스만 술탄 메흐메트 2세, 비잔티움 제국 멸망시킴." }
      ],
      "quiz": {
        "question": "1453년 콘스탄티노플을 정복한 오스만 술탄은 누구인가?",
        "options": ["술레이만 1세", "메흐메트 2세", "셀림 1세", "바예지트 1세"],
        "answer": 1
      }
    },
    {
      "id": "paris",
      "name": "파리",
      "lat": 48.8566,
      "lng": 2.3522,
      "subtitle": "프랑스 왕국 · 서유럽",
      "events": [
        { "year": 987,  "text": "위그 카페 즉위 — 카페 왕조 창건, 파리 왕도 확립." },
        { "year": 1163, "text": "모리스 드 쉴리 주교, 노트르담 대성당 착공." },
        { "year": 1348, "text": "흑사병 파리 강타, 도시 인구 절반 사망." }
      ],
      "quiz": {
        "question": "파리 노트르담 대성당 공사를 시작한 사람은?",
        "options": ["위그 카페", "루이 7세", "모리스 드 쉴리", "필리프 2세"],
        "answer": 2
      }
    },
    {
      "id": "london",
      "name": "런던",
      "lat": 51.5074,
      "lng": -0.1278,
      "subtitle": "잉글랜드 왕국 · 북서유럽",
      "events": [
        { "year": 1066, "text": "헤이스팅스 전투 — 윌리엄 정복왕, 잉글랜드 장악." },
        { "year": 1215, "text": "존 왕, 마그나 카르타 서명 — 왕권 최초 제한." },
        { "year": 1381, "text": "왓 타일러 농민 반란, 런던 진입." }
      ],
      "quiz": {
        "question": "1215년 존 왕이 서명한, 왕권을 제한한 문서는?",
        "options": ["권리장전", "마그나 카르타", "대헌법", "옥스퍼드 조례"],
        "answer": 1
      }
    },
    {
      "id": "rome",
      "name": "로마",
      "lat": 41.9028,
      "lng": 12.4964,
      "subtitle": "교황령 · 이탈리아 반도",
      "events": [
        { "year": 800,  "text": "교황 레오 3세, 성 베드로 대성당에서 샤를마뉴를 황제로 대관." },
        { "year": 1084, "text": "황제 하인리히 4세 로마 점령, 교황 그레고리우스 7세 추방." },
        { "year": 1309, "text": "교황청, 아비뇽으로 이전 — '아비뇽 유수' 시작." }
      ],
      "quiz": {
        "question": "800년 크리스마스에 샤를마뉴를 황제로 대관한 교황은?",
        "options": ["그레고리우스 7세", "클레멘스 5세", "레오 3세", "우르바누스 2세"],
        "answer": 2
      }
    }
  ]
}
```

- [ ] **Step 2: JSON 유효성 확인**

```bash
npx serve .
```

브라우저에서 `http://localhost:3000/events.json` 열기.  
기대값: JSON 데이터가 그대로 출력됨 (오류 없음).

- [ ] **Step 3: 커밋**

```bash
git add events.json
git commit -m "feat: add city data with medieval events and quizzes"
```

---

### Task 2: index.html 레이아웃 재구성

기존 `index.html`을 완전히 교체한다. 기존 클릭-마커 기능 제거, flex 레이아웃 적용, 사이드 패널 HTML 구조·CSS 추가.

**Files:**
- Modify: `index.html` (전체 교체)

- [ ] **Step 1: index.html을 아래 내용으로 교체**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HistoryMap</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { height: 100%; display: flex; }

    #map { flex: 1; height: 100vh; }

    #panel {
      width: 300px;
      height: 100vh;
      background: #1a1a2e;
      border-left: 1px solid #333;
      display: flex;
      flex-direction: column;
      overflow: hidden;
      color: #eee;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    }

    #panel-placeholder {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100%;
      color: #666;
      font-size: 14px;
      text-align: center;
      padding: 20px;
    }

    #panel-content {
      display: none;
      flex-direction: column;
      height: 100%;
    }
    #panel-content.active { display: flex; }

    .panel-header {
      background: #252540;
      padding: 16px;
      border-bottom: 1px solid #333;
      flex-shrink: 0;
    }
    .panel-city-name { font-size: 18px; font-weight: 700; color: #f0c040; }
    .panel-city-sub { font-size: 12px; color: #888; margin-top: 2px; }

    .panel-body { flex: 1; overflow-y: auto; padding: 16px; }

    .section-label {
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 1px;
      color: #888;
      margin-bottom: 12px;
    }

    .event-item {
      display: flex;
      gap: 12px;
      margin-bottom: 14px;
      padding-bottom: 14px;
      border-bottom: 1px solid #2a2a44;
    }
    .event-item:last-child { border-bottom: none; }
    .event-year { font-size: 12px; font-weight: 700; color: #f0c040; min-width: 36px; }
    .event-text { font-size: 13px; color: #ccc; line-height: 1.5; }

    .quiz-box {
      background: #252540;
      border-radius: 8px;
      padding: 16px;
      margin-top: 16px;
    }
    .quiz-label {
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 1px;
      color: #f0c040;
      margin-bottom: 10px;
    }
    .quiz-question { font-size: 14px; color: #eee; margin-bottom: 14px; line-height: 1.5; }
    .quiz-options { display: flex; flex-direction: column; gap: 8px; }

    .quiz-btn {
      background: #1a1a35;
      border: 1px solid #3a3a5c;
      color: #ccc;
      padding: 10px 14px;
      border-radius: 6px;
      text-align: left;
      font-size: 13px;
      cursor: pointer;
      transition: border-color 0.15s, color 0.15s;
    }
    .quiz-btn:hover:not(:disabled) { border-color: #f0c040; color: #fff; }
    .quiz-btn.correct { background: #1e4d2b; border-color: #2ecc71; color: #2ecc71; }
    .quiz-btn.wrong   { background: #4d1e1e; border-color: #e74c3c; color: #e74c3c; }
    .quiz-btn:disabled { cursor: default; opacity: 0.8; }

    .quiz-result { margin-top: 12px; font-size: 13px; text-align: center; font-weight: 600; }
    .quiz-result.correct { color: #2ecc71; }
    .quiz-result.wrong   { color: #e74c3c; }

    #score-badge {
      position: fixed;
      top: 12px;
      left: 12px;
      background: rgba(0, 0, 0, 0.75);
      border: 1px solid #f0c040;
      border-radius: 8px;
      padding: 6px 16px;
      font-size: 14px;
      font-weight: 700;
      color: #f0c040;
      z-index: 1000;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    }
  </style>
</head>
<body>
  <div id="map"></div>

  <div id="panel">
    <div id="panel-placeholder">← 도시를 클릭하세요</div>
    <div id="panel-content">
      <div class="panel-header">
        <div id="city-name" class="panel-city-name"></div>
        <div id="city-sub"  class="panel-city-sub"></div>
      </div>
      <div class="panel-body">
        <div class="section-label">📜 주요 사건</div>
        <div id="events-list"></div>
        <div class="quiz-box">
          <div class="quiz-label">❓ 퀴즈</div>
          <div id="quiz-question" class="quiz-question"></div>
          <div id="quiz-options"  class="quiz-options"></div>
          <div id="quiz-result"   class="quiz-result"></div>
        </div>
      </div>
    </div>
  </div>

  <div id="score-badge">⭐ 점수: <span id="score">0</span></div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const map = L.map('map').setView([48, 18], 4);
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);
  </script>
</body>
</html>
```

- [ ] **Step 2: 브라우저에서 레이아웃 확인**

`http://localhost:3000` 열기 (Task 1의 `npx serve .`가 실행 중이어야 함).  
기대값:
- 지도가 왼쪽에, 어두운 패널이 오른쪽에 보임
- 패널 안에 "← 도시를 클릭하세요" 문구
- 좌상단에 "⭐ 점수: 0" 배지
- 지도 중심이 유럽 (이탈리아·프랑스·독일 주변)

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: restructure layout with side panel and score badge"
```

---

### Task 3: 데이터 로드 + 도시 마커 생성

**Files:**
- Modify: `index.html` — `<script>` 블록에 fetch 로직 추가

- [ ] **Step 1: index.html의 `</script>` 바로 앞 (map 초기화 다음)에 아래 코드 추가**

기존 마지막 `</script>` 직전의 닫는 부분을 아래처럼 교체한다:

```html
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    let score = 0;
    const answered = {};
    let activeMarker = null;

    const map = L.map('map').setView([48, 18], 4);
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    fetch('events.json')
      .then(r => r.json())
      .then(data => {
        data.cities.forEach(city => {
          const marker = L.circleMarker([city.lat, city.lng], {
            radius: 8,
            fillColor: '#f0c040',
            color: '#fff',
            weight: 2,
            fillOpacity: 1
          }).addTo(map);

          marker.bindTooltip(city.name, {
            permanent: true,
            direction: 'top',
            offset: [0, -10]
          });

          marker.on('click', function () {
            if (activeMarker) activeMarker.setStyle({ fillColor: '#f0c040' });
            this.setStyle({ fillColor: '#e74c3c' });
            activeMarker = this;
            renderPanel(city);
          });
        });
      })
      .catch(err => console.error('events.json 로딩 실패:', err));

    function renderPanel(city) {
      // Task 4에서 구현
    }
  </script>
```

- [ ] **Step 2: 브라우저에서 마커 확인**

`http://localhost:3000` 새로고침.  
기대값:
- 지도 위에 금색 원형 마커 4개 (콘스탄티노플·파리·런던·로마)
- 각 마커 위에 도시 이름 툴팁 표시
- 마커 클릭 시 해당 마커가 빨간색으로 바뀜 (패널은 아직 빈 상태)

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: load events.json and place city markers on map"
```

---

### Task 4: 사이드 패널 — 도시 정보 표시

**Files:**
- Modify: `index.html` — `renderPanel` 함수 구현

- [ ] **Step 1: `renderPanel` 함수를 아래 내용으로 교체**

`function renderPanel(city) { // Task 4에서 구현 }` 부분을 아래로 교체:

```js
    function renderPanel(city) {
      document.getElementById('panel-placeholder').style.display = 'none';
      const content = document.getElementById('panel-content');
      content.classList.add('active');

      document.getElementById('city-name').textContent = city.name;
      document.getElementById('city-sub').textContent  = city.subtitle;

      document.getElementById('events-list').innerHTML = city.events.map(e =>
        `<div class="event-item">
          <div class="event-year">${e.year}</div>
          <div class="event-text">${e.text}</div>
        </div>`
      ).join('');

      renderQuiz(city);
    }

    function renderQuiz(city) {
      const prev = answered[city.id];

      document.getElementById('quiz-question').textContent = city.quiz.question;

      const optionsEl = document.getElementById('quiz-options');
      optionsEl.innerHTML = city.quiz.options.map((opt, i) => {
        let cls = '';
        if (prev !== undefined) {
          if (i === city.quiz.answer) cls = 'correct';
          else if (i === prev.chosen) cls = 'wrong';
        }
        const disabled = prev !== undefined ? 'disabled' : '';
        return `<button class="quiz-btn ${cls}" ${disabled} data-index="${i}">${opt}</button>`;
      }).join('');

      const resultEl = document.getElementById('quiz-result');
      if (prev !== undefined) {
        resultEl.textContent  = prev.correct ? '✓ 정답! +1점' : '✗ 오답';
        resultEl.className    = 'quiz-result ' + (prev.correct ? 'correct' : 'wrong');
      } else {
        resultEl.textContent = '';
        resultEl.className   = 'quiz-result';
      }

      optionsEl.querySelectorAll('.quiz-btn').forEach(btn => {
        btn.addEventListener('click', function () {
          const chosen  = parseInt(this.dataset.index);
          const correct = chosen === city.quiz.answer;

          answered[city.id] = { chosen, correct };

          if (correct) {
            score++;
            document.getElementById('score').textContent = score;
          }

          optionsEl.querySelectorAll('.quiz-btn').forEach((b, i) => {
            b.disabled = true;
            if (i === city.quiz.answer) b.classList.add('correct');
            else if (i === chosen)      b.classList.add('wrong');
          });

          resultEl.textContent = correct ? '✓ 정답! +1점' : '✗ 오답';
          resultEl.className   = 'quiz-result ' + (correct ? 'correct' : 'wrong');
        });
      });
    }
```

- [ ] **Step 2: 브라우저에서 전체 흐름 확인**

`http://localhost:3000` 새로고침 후 아래 시나리오 순서대로 테스트:

1. 콘스탄티노플 마커 클릭 → 패널에 도시명·사건 3개·퀴즈 표시 확인
2. 퀴즈에서 **오답** 선택 → 선택 버튼 빨간색, 정답 버튼 초록색, "✗ 오답" 표시
3. 파리 마커 클릭 → 패널이 파리 내용으로 교체됨
4. 파리 퀴즈에서 **정답** 선택 → "✓ 정답! +1점" 표시, 점수 배지 1로 증가
5. 콘스탄티노플 마커 다시 클릭 → 이전 오답 결과 그대로 유지 (버튼 비활성화)
6. 런던·로마 마커 각각 클릭 → 내용 정상 표시

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: implement side panel with events display and quiz logic"
```

---

### Task 5: 배포 확인 및 push

**Files:**
- 없음 (코드 변경 없음)

- [ ] **Step 1: git push**

```bash
git push
```

- [ ] **Step 2: GitHub Pages 활성화 확인 (미설정 시)**

GitHub 저장소 → Settings → Pages → Source: `main` 브랜치, `/ (root)` 폴더 선택 → Save.  
배포 URL: `https://<username>.github.io/HistoryMap/`

- [ ] **Step 3: 배포 환경에서 동일 시나리오 확인**

Task 4의 시나리오를 배포 URL에서 한 번 더 반복.  
기대값: 로컬과 동일하게 동작.
