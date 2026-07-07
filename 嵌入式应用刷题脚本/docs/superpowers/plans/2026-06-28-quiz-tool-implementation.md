# 期末刷题小工具 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单 HTML 文件的大学生期末刷题工具，支持单选/填空/判断三种题型，随机抽题，自动错题本，深色主题。

**Architecture:** 纯前端单页应用，所有逻辑在单个 `index.html` 中。使用 `localStorage` 持久化错题记录。页面通过 JavaScript 控制 4 个视图（设置页、答题页、结果页、错题本）的显示切换。

**Tech Stack:** 原生 HTML5 + CSS3 + JavaScript (ES6+)，零外部依赖。

## 全局约束

- 所有代码在单文件 `index.html` 中，CSS/JS 内嵌
- 深色主题：背景 `#1a1a2e`，卡片 `#16213e`，强调色 `#0f3460`/`#e94560`
- 正确反馈 #2ecc71，错误反馈 #e74c3c
- 移动端自适应
- localStorage 键名：`wrongQuestions`
- 填空题模糊匹配：去掉首尾空格后比较用户输入是否在 answer 数组中
- 判断题 answer 为 "对" 或 "错"

---

### Task 1: HTML 骨架 + CSS 样式

**Files:**
- Create: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 产出：完整 HTML 结构（4 个视图的容器 div）+ 深色主题 CSS

- [ ] **Step 1: 创建 HTML 结构 + 内置样式表**

写完整的 HTML 骨架，包含 4 个视图容器：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>期末刷题小工具</title>
<style>
/* 全部样式写在这里 */
</style>
</head>
<body>
  <!-- 设置页 -->
  <div id="pageSetup" class="page active">
    <div class="container">
      <h1 id="subjectTitle"></h1>
      <p id="questionCount"></p>
      <div class="input-group">
        <label>本次抽题数量：</label>
        <input type="number" id="numInput" min="1" value="10">
      </div>
      <button id="btnStart" class="btn btn-primary">开始答题</button>
      <button id="btnWrongBook" class="btn btn-secondary">错题本</button>
      <button id="btnClearWrong" class="btn btn-danger">清空错题</button>
    </div>
  </div>

  <!-- 答题页 -->
  <div id="pageQuiz" class="page">
    <div class="container">
      <div class="progress-bar"><div id="progressFill"></div></div>
      <div class="quiz-header">
        <span id="progressText">第 1/10 题</span>
        <span id="scoreDisplay">正确率: 0%</span>
      </div>
      <div id="questionCard" class="card">
        <span id="typeTag" class="tag"></span>
        <p id="questionText"></p>
        <div id="optionsArea"></div>
      </div>
      <div class="nav-buttons">
        <button id="btnPrev" class="btn">上一题</button>
        <button id="btnSubmit" class="btn btn-primary">提交答案</button>
        <button id="btnNext" class="btn">下一题</button>
      </div>
      <div id="feedbackArea" class="feedback"></div>
      <div id="explainArea" class="explain" style="display:none;"></div>
    </div>
  </div>

  <!-- 结果页 -->
  <div id="pageResult" class="page">
    <div class="container result-container">
      <h2 id="resultScore" class="result-score"></h2>
      <p id="resultRate" class="result-rate"></p>
      <div id="resultList"></div>
      <div class="result-actions">
        <button id="btnRetry" class="btn btn-primary">再来一轮</button>
        <button id="btnGoWrong" class="btn btn-secondary">错题本</button>
      </div>
    </div>
  </div>

  <!-- 错题本 -->
  <div id="pageWrong" class="page">
    <div class="container">
      <h2>错题本</h2>
      <div id="wrongList"></div>
      <div class="wrong-actions">
        <button id="btnRetryWrong" class="btn btn-primary">重做错题</button>
        <button id="btnClearWrongConfirm" class="btn btn-danger">清空错题本</button>
        <button id="btnBackSetup2" class="btn">返回首页</button>
      </div>
    </div>
  </div>

  <script>
  // 所有 JS 写在这里
  </script>
</body>
</html>
```

- [ ] **Step 2: 写完整 CSS 样式**

```css
* {
  margin: 0; padding: 0; box-sizing: border-box;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background: #1a1a2e;
  color: #eee;
  min-height: 100vh;
  display: flex; justify-content: center; align-items: flex-start;
  padding: 20px;
}
.container {
  max-width: 700px; width: 100%;
}
.page {
  display: none;
}
.page.active {
  display: block;
}

/* 卡片 */
.card {
  background: #16213e;
  border-radius: 12px;
  padding: 24px;
  margin: 16px 0;
}

/* 按钮通用 */
.btn {
  padding: 10px 20px;
  border: none; border-radius: 8px;
  cursor: pointer;
  font-size: 15px;
  transition: all 0.2s;
  color: #fff;
  background: #0f3460;
}
.btn:hover {
  opacity: 0.85;
}
.btn-primary {
  background: #e94560;
}
.btn-secondary {
  background: #0f3460;
}
.btn-danger {
  background: #c0392b;
}

/* 进度条 */
.progress-bar {
  height: 6px; background: #333; border-radius: 3px; margin-bottom: 12px;
}
.progress-fill {
  height: 100%; background: #e94560; border-radius: 3px; transition: width 0.3s;
}

/* 设置页 */
#pageSetup .container > * {
  margin-bottom: 16px;
}
.input-group {
  display: flex; align-items: center; gap: 12px;
}
.input-group input {
  width: 80px; padding: 8px; border-radius: 6px; border: 1px solid #555;
  background: #16213e; color: #fff; font-size: 16px; text-align: center;
}
.btn-group {
  display: flex; gap: 12px; flex-wrap: wrap;
}

/* 答题页 */
.quiz-header {
  display: flex; justify-content: space-between; margin-bottom: 8px;
  font-size: 14px; color: #aaa;
}
.tag {
  display: inline-block; padding: 2px 10px; border-radius: 4px;
  font-size: 12px; background: #0f3460; margin-bottom: 12px;
}
#questionText {
  font-size: 17px; line-height: 1.6; margin-bottom: 16px;
}

/* 选项 */
.option {
  display: block; width: 100%;
  padding: 12px 16px; margin-bottom: 8px;
  background: #1a1a2e; border: 2px solid #333; border-radius: 8px;
  cursor: pointer; transition: all 0.2s;
  text-align: left; font-size: 15px; color: #eee;
}
.option:hover {
  border-color: #e94560;
}
.option.selected {
  border-color: #e94560; background: rgba(233,69,96,0.15);
}
.option.correct {
  border-color: #2ecc71; background: rgba(46,204,113,0.15);
}
.option.wrong {
  border-color: #e74c3c; background: rgba(231,76,60,0.15);
}

/* 判断按钮组 */
.judge-buttons {
  display: flex; gap: 16px; margin-top: 12px;
}
.btn-judge {
  flex: 1; padding: 16px; font-size: 18px; font-weight: bold;
  border: 2px solid #333; border-radius: 8px; cursor: pointer;
  background: #1a1a2e; color: #eee; transition: all 0.2s;
}
.btn-judge.selected {
  border-color: #e94560; background: rgba(233,69,96,0.15);
}
.btn-judge.correct {
  border-color: #2ecc71; background: rgba(46,204,113,0.15);
}
.btn-judge.wrong {
  border-color: #e74c3c; background: rgba(231,76,60,0.15);
}

/* 填空输入 */
#fillInput {
  width: 100%; padding: 12px; border-radius: 8px; border: 2px solid #333;
  background: #1a1a2e; color: #eee; font-size: 16px; margin-top: 12px;
}
#fillInput:focus {
  border-color: #e94560; outline: none;
}

/* 导航按钮 */
.nav-buttons {
  display: flex; gap: 12px; margin-top: 16px; flex-wrap: wrap;
}
.nav-buttons .btn {
  flex: 1; min-width: 80px;
}

/* 反馈 */
.feedback {
  margin-top: 12px; padding: 12px; border-radius: 8px; display: none;
}
.feedback.show { display: block; }
.feedback.correct { background: rgba(46,204,113,0.15); color: #2ecc71; }
.feedback.wrong { background: rgba(231,76,60,0.15); color: #e74c3c; }

/* 解析 */
.explain {
  margin-top: 12px; padding: 12px; border-radius: 8px;
  background: rgba(15,52,96,0.5); color: #aab; line-height: 1.6;
}

/* 结果页 */
.result-container { text-align: center; }
.result-score { font-size: 36px; color: #e94560; margin: 24px 0 8px; }
.result-rate { font-size: 18px; color: #aaa; margin-bottom: 24px; }
.result-item {
  background: #16213e; border-radius: 8px; padding: 12px 16px;
  margin-bottom: 8px; text-align: left;
}
.result-item .ri-header {
  display: flex; justify-content: space-between; align-items: center;
  cursor: pointer;
}
.result-item .ri-icon { font-size: 18px; }
.result-item .ri-question { flex: 1; margin: 0 12px; }
.result-item .ri-detail { display: none; margin-top: 8px; color: #aaa; font-size: 14px; }
.result-item .ri-detail.show { display: block; }

/* 错题本 */
#wrongList .wrong-item {
  background: #16213e; border-radius: 8px; padding: 12px 16px;
  margin-bottom: 8px;
}
.wrong-actions {
  display: flex; gap: 12px; margin-top: 16px; flex-wrap: wrap;
}
.wrong-actions .btn { flex: 1; }
```

- [ ] **Step 3: 验证 HTML+CSS 无语法错误**

用浏览器打开文件，确认四页容器都定义正确，深色主题显示正常。

---

### Task 2: 题库数据 + 核心状态管理

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html` (在 `<script>` 中添加数据)

**Interfaces:**
- 产出：全局变量 `QUESTIONS`（题库数组）、`STATE`（运行时状态对象）
- 后续任务通过 `QUESTIONS` 读取题目数据，通过 `STATE` 读取/修改当前会话状态

- [ ] **Step 1: 在 `<script>` 顶部添加题库数据和状态管理**

```javascript
// ==================== 题库数据 ====================
// 使用说明：按以下格式添加题目，然后修改 const CURRENT_SUBJECT
const CURRENT_SUBJECT = "示例科目";

const QUESTIONS = [
  // ---- 示例：单选题 ----
  {
    id: 1,
    type: "单选",
    question: "光的折射定律是指？",
    options: ["反射角等于入射角", "折射光线与入射光线分居法线两侧", "光速不变原理", "以上都不是"],
    answer: "B",
    explain: "根据斯涅耳定律（折射定律），折射光线与入射光线分居法线两侧，入射角与折射角的正弦值之比为常数。"
  },
  // ---- 示例：判断题 ----
  {
    id: 2,
    type: "判断",
    question: "光在真空中的传播速度约为 3×10⁸ m/s。",
    answer: "对",
    explain: "光在真空中速度恒定为 c ≈ 3.0×10⁸ m/s。"
  },
  // ---- 示例：填空题 ----
  {
    id: 3,
    type: "填空",
    question: "牛顿第___定律阐述了力与加速度的关系。",
    answer: ["二", "2", "第二"],
    explain: "牛顿第二定律：F = ma，即物体加速度与所受合外力成正比，与质量成反比。"
  }
];

// ==================== 运行状态 ====================
let STATE = {
  shuffled: [],       // 当前轮抽取的题目（打乱顺序）
  currentIndex: 0,    // 当前正在做的题号（0-based）
  answers: {},        // { index(0-based): userAnswer }
  submitted: {},      // { index: true } 已提交的
  results: {},        // { index: true/false } 是否正确
  totalCorrect: 0,
  totalAnswered: 0,
};

function shuffleArray(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
```

- [ ] **Step 2: 添加 localStorage 错题操作函数**

```javascript
// ==================== 错题本 ====================
function getWrongQuestions() {
  try {
    return JSON.parse(localStorage.getItem('wrongQuestions')) || [];
  } catch { return []; }
}

function saveWrongQuestion(questionId, userAnswer) {
  const wrongs = getWrongQuestions();
  // 不重复记录
  if (!wrongs.some(w => w.questionId === questionId)) {
    wrongs.push({ questionId, userAnswer, timestamp: Date.now() });
    localStorage.setItem('wrongQuestions', JSON.stringify(wrongs));
  }
}

function removeWrongQuestion(questionId) {
  const wrongs = getWrongQuestions().filter(w => w.questionId !== questionId);
  localStorage.setItem('wrongQuestions', JSON.stringify(wrongs));
}

function clearAllWrongQuestions() {
  localStorage.removeItem('wrongQuestions');
}

function getWrongIds() {
  return getWrongQuestions().map(w => w.questionId);
}
```

- [ ] **Step 3: 添加视图切换函数**

```javascript
// ==================== 页面切换 ====================
function showPage(pageId) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(pageId).classList.add('active');
}
```

---

### Task 3: 设置页逻辑

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 消费：`CURRENT_SUBJECT`, `QUESTIONS`, `showPage()`, `getWrongQuestions()`
- 产出：在 `DOMContentLoaded` 中初始化设置页

- [ ] **Step 1: 添加设置页初始化函数**

```javascript
// ==================== 设置页 ====================
function initSetupPage() {
  document.getElementById('subjectTitle').textContent = `📚 ${CURRENT_SUBJECT}`;
  document.getElementById('questionCount').textContent = `题库共 ${QUESTIONS.length} 题`;
  document.getElementById('numInput').value = Math.min(10, QUESTIONS.length);
  updateWrongBadge();
}

function updateWrongBadge() {
  const count = getWrongQuestions().length;
  const btn = document.getElementById('btnWrongBook');
  btn.textContent = count > 0 ? `错题本 (${count})` : '错题本';
}
```

- [ ] **Step 2: 绑定设置页按钮事件**

```javascript
// 开始答题
document.getElementById('btnStart').addEventListener('click', function() {
  const num = parseInt(document.getElementById('numInput').value) || 10;
  const count = Math.min(Math.max(1, num), QUESTIONS.length);
  startQuiz(count);
});

// 错题本
document.getElementById('btnWrongBook').addEventListener('click', function() {
  renderWrongBook();
  showPage('pageWrong');
});

// 清空错题
document.getElementById('btnClearWrong').addEventListener('click', function() {
  if (confirm('确定要清空所有错题记录吗？')) {
    clearAllWrongQuestions();
    updateWrongBadge();
  }
});
```

- [ ] **Step 3: 实现 `startQuiz()` 函数**

```javascript
// ==================== 开始答题 ====================
function startQuiz(count) {
  STATE = {
    shuffled: shuffleArray(QUESTIONS).slice(0, count),
    currentIndex: 0,
    answers: {},
    submitted: {},
    results: {},
    totalCorrect: 0,
    totalAnswered: 0,
  };
  renderQuestion(0);
  showPage('pageQuiz');
}
```

---

### Task 4: 答题页面渲染

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 消费：`STATE`, `showPage()`
- 产出：`renderQuestion(index)` 渲染指定索引的题目

- [ ] **Step 1: 实现 `renderQuestion()` 函数**

```javascript
// ==================== 答题渲染 ====================
function renderQuestion(index) {
  STATE.currentIndex = index;
  const q = STATE.shuffled[index];
  const total = STATE.shuffled.length;
  const answered = Object.keys(STATE.submitted).length;
  const correct = STATE.totalCorrect;
  const rate = answered > 0 ? Math.round(correct / answered * 100) : 0;

  // 进度条
  document.getElementById('progressFill').style.width = `${(index / total) * 100}%`;
  document.getElementById('progressText').textContent = `第 ${index + 1}/${total} 题`;
  document.getElementById('scoreDisplay').textContent = `正确率: ${rate}%`;

  // 题目卡片
  const card = document.getElementById('questionCard');
  const typeMap = { '单选': '🔘 单选题', '判断': '⭕ 判断题', '填空': '📝 填空题' };
  document.getElementById('typeTag').textContent = typeMap[q.type] || q.type;
  document.getElementById('questionText').textContent = q.question;

  const optArea = document.getElementById('optionsArea');
  optArea.innerHTML = '';

  if (q.type === '单选') {
    const labels = ['A', 'B', 'C', 'D'];
    q.options.forEach((opt, i) => {
      const btn = document.createElement('button');
      btn.className = 'option';
      btn.textContent = `${labels[i]}. ${opt}`;
      btn.dataset.value = labels[i];
      // 还原已选状态
      if (STATE.answers[index] === labels[i]) btn.classList.add('selected');
      if (STATE.submitted[index]) {
        if (labels[i] === q.answer) btn.classList.add('correct');
        else if (STATE.answers[index] === labels[i] && STATE.answers[index] !== q.answer) btn.classList.add('wrong');
      }
      btn.onclick = () => selectOption(index, labels[i]);
      optArea.appendChild(btn);
    });
  } else if (q.type === '判断') {
    const div = document.createElement('div');
    div.className = 'judge-buttons';
    ['对', '错'].forEach(val => {
      const btn = document.createElement('button');
      btn.className = 'btn-judge';
      btn.textContent = val;
      btn.dataset.value = val;
      if (STATE.answers[index] === val) btn.classList.add('selected');
      if (STATE.submitted[index]) {
        if (val === q.answer) btn.classList.add('correct');
        else if (STATE.answers[index] === val) btn.classList.add('wrong');
      }
      btn.onclick = () => selectOption(index, val);
      div.appendChild(btn);
    });
    optArea.appendChild(div);
  } else if (q.type === '填空') {
    const input = document.createElement('input');
    input.type = 'text';
    input.id = 'fillInput';
    input.placeholder = '请输入答案...';
    if (STATE.answers[index]) input.value = STATE.answers[index];
    input.oninput = (e) => { STATE.answers[index] = e.target.value.trim(); };
    if (STATE.submitted[index]) input.disabled = true;
    optArea.appendChild(input);
  }

  // 反馈
  const fb = document.getElementById('feedbackArea');
  if (STATE.submitted[index]) {
    const isCorrect = STATE.results[index];
    fb.className = `feedback show ${isCorrect ? 'correct' : 'wrong'}`;
    fb.textContent = isCorrect ? '✅ 回答正确！' : `❌ 回答错误，正确答案是：${q.answer}`;
  } else {
    fb.className = 'feedback';
  }

  // 解析
  const exp = document.getElementById('explainArea');
  if (STATE.submitted[index]) {
    exp.style.display = 'block';
    exp.innerHTML = `<strong>📖 解析：</strong>${q.explain || '暂无解析'}`;
    // 检查是否错题
    const wrongIds = getWrongIds();
    if (wrongIds.includes(q.id) && !STATE.results[index]) {
      exp.innerHTML += '<br><span style="color:#e74c3c;">❗ 已在错题本中</span>';
    }
  } else {
    exp.style.display = 'none';
  }

  // 按钮状态
  document.getElementById('btnPrev').style.display = index > 0 ? 'inline-block' : 'none';
  document.getElementById('btnNext').textContent = index < total - 1 ? '下一题' : '交卷';
  document.getElementById('btnSubmit').disabled = STATE.submitted[index];
  if (q.type === '填空' && STATE.submitted[index]) {
    document.getElementById('fillInput').disabled = true;
  }
}
```

- [ ] **Step 2: 实现选择函数**

```javascript
function selectOption(index, value) {
  if (STATE.submitted[index]) return; // 已提交不可改
  STATE.answers[index] = value;
  renderQuestion(STATE.currentIndex);
}
```

---

### Task 5: 提交答案与导航逻辑

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 消费：`STATE`, `renderQuestion()`, `saveWrongQuestion()`, `showPage()`
- 产出：提交答案、上一题/下一题导航、交卷功能

- [ ] **Step 1: 实现 `submitAnswer()` 函数**

```javascript
// ==================== 提交答案 ====================
function submitAnswer() {
  const idx = STATE.currentIndex;
  if (STATE.submitted[idx]) return;

  const q = STATE.shuffled[idx];
  const userAns = STATE.answers[idx];

  if (!userAns && userAns !== '') {
    alert('请先作答再提交！');
    return;
  }
  // 填空题允许空字符串不算作答
  if (q.type === '填空' && userAns === '') {
    alert('请输入答案！');
    return;
  }

  let isCorrect = false;
  if (q.type === '单选' || q.type === '判断') {
    isCorrect = userAns === q.answer;
  } else if (q.type === '填空') {
    // 模糊匹配：去掉首尾空格后比较
    const normalized = userAns.trim();
    isCorrect = q.answer.some(a => a.trim() === normalized);
  }

  STATE.submitted[idx] = true;
  STATE.results[idx] = isCorrect;
  STATE.totalAnswered++;
  if (isCorrect) STATE.totalCorrect++;
  else saveWrongQuestion(q.id, userAns);

  renderQuestion(idx);
}
```

- [ ] **Step 2: 绑定导航按钮事件**

```javascript
// 提交答案
document.getElementById('btnSubmit').addEventListener('click', submitAnswer);

// 上一题
document.getElementById('btnPrev').addEventListener('click', function() {
  if (STATE.currentIndex > 0) renderQuestion(STATE.currentIndex - 1);
});

// 下一题/交卷
document.getElementById('btnNext').addEventListener('click', function() {
  const total = STATE.shuffled.length;
  if (STATE.currentIndex < total - 1) {
    renderQuestion(STATE.currentIndex + 1);
  } else {
    showResults();
    showPage('pageResult');
  }
});
```

---

### Task 6: 结果页

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 消费：`STATE`, `showPage()`
- 产出：`showResults()` 在结果页渲染

- [ ] **Step 1: 实现 `showResults()` 函数

```javascript
// ==================== 结果页 ====================
function showResults() {
  const total = STATE.shuffled.length;
  const correct = STATE.totalCorrect;
  const rate = total > 0 ? Math.round(correct / total * 100) : 0;

  document.getElementById('resultScore').textContent = `答对 ${correct}/${total} 题`;
  document.getElementById('resultRate').textContent = `正确率 ${rate}%`;

  const list = document.getElementById('resultList');
  list.innerHTML = '';
  STATE.shuffled.forEach((q, i) => {
    const userAns = STATE.answers[i] || '未作答';
    const isCorrect = STATE.submitted[i] ? STATE.results[i] : false;
    const icon = STATE.submitted[i] ? (isCorrect ? '✅' : '❌') : '⏭️';
    const div = document.createElement('div');
    div.className = 'result-item';
    div.innerHTML = `
      <div class="ri-header" onclick="this.nextElementSibling.classList.toggle('show')">
        <span class="ri-icon">${icon}</span>
        <span class="ri-question">${q.question.length > 30 ? q.question.slice(0, 30) + '...' : q.question}</span>
        <span style="color:#aaa;font-size:12px;">▼</span>
      </div>
      <div class="ri-detail">
        <p><strong>你的答案：</strong>${userAns}</p>
        <p><strong>正确答案：</strong>${Array.isArray(q.answer) ? q.answer.join(' / ') : q.answer}</p>
        <p><strong>解析：</strong>${q.explain || '暂无'}</p>
      </div>
    `;
    list.appendChild(div);
  });
}
```

- [ ] **Step 2: 绑定结果页按钮事件**

```javascript
document.getElementById('btnRetry').addEventListener('click', function() {
  initSetupPage();
  showPage('pageSetup');
});

document.getElementById('btnGoWrong').addEventListener('click', function() {
  renderWrongBook();
  showPage('pageWrong');
});
```

---

### Task 7: 错题本

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

**Interfaces:**
- 消费：`QUESTIONS`, `getWrongQuestions()`, `clearAllWrongQuestions()`, `startQuiz()`, `showPage()`
- 产出：`renderWrongBook()`

- [ ] **Step 1: 实现 `renderWrongBook()`**

```javascript
// ==================== 错题本 ====================
function renderWrongBook() {
  const wrongs = getWrongQuestions();
  const list = document.getElementById('wrongList');
  list.innerHTML = '';

  if (wrongs.length === 0) {
    list.innerHTML = '<p style="color:#666;text-align:center;padding:40px;">🎉 暂无错题记录</p>';
    return;
  }

  wrongs.forEach(w => {
    const q = QUESTIONS.find(ques => ques.id === w.questionId);
    if (!q) return;
    const div = document.createElement('div');
    div.className = 'wrong-item';
    div.innerHTML = `
      <div style="display:flex;justify-content:space-between;align-items:center;cursor:pointer;"
           onclick="this.nextElementSibling.classList.toggle('show')">
        <span>❌ ${q.question.length > 40 ? q.question.slice(0,40)+'...' : q.question}</span>
        <span style="color:#aaa;font-size:12px;">▼</span>
      </div>
      <div class="ri-detail">
        <p><strong>你的答案：</strong>${w.userAnswer || '未作答'}</p>
        <p><strong>正确答案：</strong>${Array.isArray(q.answer) ? q.answer.join(' / ') : q.answer}</p>
        <p><strong>解析：</strong>${q.explain || '暂无'}</p>
        <button class="btn btn-danger" style="margin-top:8px;font-size:13px;padding:4px 12px;"
                onclick="removeWrongQuestion(${q.id});renderWrongBook();">移出错题本</button>
      </div>
    `;
    list.appendChild(div);
  });
}
```

- [ ] **Step 2: 绑定错题本按钮事件**

```javascript
document.getElementById('btnRetryWrong').addEventListener('click', function() {
  const wrongs = getWrongQuestions();
  if (wrongs.length === 0) {
    alert('没有错题可重做！');
    return;
  }
  // 只抽错题中对应的题目
  const wrongIds = wrongs.map(w => w.questionId);
  const wrongQuestions = QUESTIONS.filter(q => wrongIds.includes(q.id));
  STATE = {
    shuffled: shuffleArray(wrongQuestions),
    currentIndex: 0,
    answers: {},
    submitted: {},
    results: {},
    totalCorrect: 0,
    totalAnswered: 0,
  };
  if (STATE.shuffled.length === 0) {
    alert('没有找到对应题目！');
    return;
  }
  renderQuestion(0);
  showPage('pageQuiz');
});

document.getElementById('btnClearWrongConfirm').addEventListener('click', function() {
  if (confirm('确定清空所有错题记录吗？')) {
    clearAllWrongQuestions();
    renderWrongBook();
    updateWrongBadge();
  }
});

document.getElementById('btnBackSetup2').addEventListener('click', function() {
  initSetupPage();
  showPage('pageSetup');
});
```

---

### Task 8: 入口初始化 + 正确率更新修复

**Files:**
- Modify: `C:\Users\zhangbuyun\Desktop\刷题脚本\index.html`

- [ ] **Step 1: 添加 DOMContentLoaded 初始化**

```javascript
// ==================== 入口 ====================
document.addEventListener('DOMContentLoaded', function() {
  initSetupPage();
  showPage('pageSetup');
});
```

- [ ] **Step 2: 修复正确率实时更新**

需确保在 `renderQuestion` 中正确计算已答题正确率（使用 `STATE.totalAnswered` 和 `STATE.totalCorrect`，而不是当前题目索引），这个已在 Task 4 中实现。

- [ ] **Step 3: 最终验证**

打开 `index.html` 浏览器进行手工测试：
1. 设置页显示正确题库信息
2. 输入抽题数量 → 开始答题
3. 三种题型是否正常渲染和作答
4. 提交答案 → 对错反馈、解析显示
5. 导航上一题/下一题/交卷
6. 结果页分数和正确率正确
7. 错题本新增/查看/清空
8. 重做错题功能
9. 返回首页再来一轮
