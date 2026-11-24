# 原型验证模板补充

按 [mz-workflow 的原型阶段](../SKILL.md) 决定是否制作原型。将示例中的状态、事件和验收条件替换为项目需求。

## 逻辑原型模板（详细版）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>逻辑原型：低保申请超时自动退回</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: #f5f5f5;
      padding: 20px;
    }
    .container {
      max-width: 1200px;
      margin: 0 auto;
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
    }
    .panel {
      background: white;
      border-radius: 8px;
      padding: 20px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    h1 { font-size: 24px; margin-bottom: 20px; color: #333; }
    h2 { font-size: 18px; margin-bottom: 15px; color: #555; border-bottom: 2px solid #007bff; padding-bottom: 5px; }

    .state-display {
      text-align: center;
      padding: 30px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      border-radius: 8px;
      margin-bottom: 20px;
    }
    .current-state {
      font-size: 32px;
      font-weight: bold;
      color: white;
      margin-bottom: 10px;
    }
    .state-meta {
      color: rgba(255,255,255,0.9);
      font-size: 14px;
    }

    .actions {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 10px;
      margin-bottom: 20px;
    }
    button {
      padding: 12px 20px;
      border: none;
      border-radius: 6px;
      font-size: 14px;
      font-weight: 500;
      cursor: pointer;
      transition: all 0.2s;
    }
    button:hover { transform: translateY(-2px); box-shadow: 0 4px 8px rgba(0,0,0,0.15); }
    button:active { transform: translateY(0); }
    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
      transform: none !important;
    }
    .btn-primary { background: #007bff; color: white; }
    .btn-success { background: #28a745; color: white; }
    .btn-danger { background: #dc3545; color: white; }
    .btn-warning { background: #ffc107; color: #333; }
    .btn-secondary { background: #6c757d; color: white; }

    .history {
      max-height: 400px;
      overflow-y: auto;
    }
    .history-item {
      padding: 10px;
      border-left: 3px solid #007bff;
      margin-bottom: 10px;
      background: #f8f9fa;
      border-radius: 4px;
    }
    .history-time { font-size: 12px; color: #666; }
    .history-transition { font-weight: 500; color: #333; margin: 5px 0; }
    .history-reason { font-size: 13px; color: #666; }

    .context {
      background: #fff3cd;
      border: 1px solid #ffc107;
      border-radius: 4px;
      padding: 15px;
      margin-bottom: 20px;
    }
    .context-item {
      display: flex;
      justify-content: space-between;
      padding: 5px 0;
      border-bottom: 1px solid rgba(0,0,0,0.1);
    }
    .context-item:last-child { border-bottom: none; }
    .context-label { font-weight: 500; color: #856404; }
    .context-value { color: #333; }

    .timer {
      text-align: center;
      padding: 15px;
      background: #dc3545;
      color: white;
      border-radius: 6px;
      margin-bottom: 20px;
      font-size: 18px;
      font-weight: bold;
    }
    .timer.safe { background: #28a745; }
    .timer.warning { background: #ffc107; color: #333; }
  </style>
</head>
<body>
  <div class="container">
    <!-- 左侧：状态机 -->
    <div class="panel">
      <h1>低保申请状态机原型</h1>

      <div class="state-display">
        <div class="current-state" id="current-state">待提交</div>
        <div class="state-meta" id="state-meta">初始状态</div>
      </div>

      <div class="timer" id="timer">
        无超时计时
      </div>

      <div class="context" id="context">
        <div class="context-item">
          <span class="context-label">申请 ID</span>
          <span class="context-value" id="app-id">LA-20261119-001</span>
        </div>
        <div class="context-item">
          <span class="context-label">提交时间</span>
          <span class="context-value" id="submit-time">—</span>
        </div>
        <div class="context-item">
          <span class="context-label">审核人</span>
          <span class="context-value" id="reviewer">—</span>
        </div>
        <div class="context-item">
          <span class="context-label">超时退回次数</span>
          <span class="context-value" id="timeout-count">0</span>
        </div>
      </div>

      <h2>可用操作</h2>
      <div class="actions" id="actions"></div>
    </div>

    <!-- 右侧：历史记录 -->
    <div class="panel">
      <h2>状态流转历史</h2>
      <div class="history" id="history"></div>
    </div>
  </div>

  <script>
    // 状态机定义
    const STATE_MACHINE = {
      '待提交': {
        actions: [
          { name: '提交申请', nextState: '待审核', color: 'primary' },
        ],
      },
      '待审核': {
        actions: [
          { name: '开始审核', nextState: '审核中', color: 'primary' },
          { name: '退回修改', nextState: '待提交', color: 'warning' },
        ],
        timeout: 5 * 24 * 60 * 60 * 1000, // 5 天
        timeoutAction: { name: '超时自动退回', nextState: '超时退回', color: 'danger' },
      },
      '审核中': {
        actions: [
          { name: '审核通过', nextState: '已通过', color: 'success' },
          { name: '审核拒绝', nextState: '已拒绝', color: 'danger' },
          { name: '退回修改', nextState: '待提交', color: 'warning' },
        ],
        timeout: 3 * 24 * 60 * 60 * 1000, // 3 天
        timeoutAction: { name: '超时自动退回', nextState: '超时退回', color: 'danger' },
      },
      '超时退回': {
        actions: [
          { name: '重新提交', nextState: '待审核', color: 'primary' },
        ],
      },
      '已通过': {
        actions: [],
        isFinal: true,
      },
      '已拒绝': {
        actions: [],
        isFinal: true,
      },
    };

    // 状态
    let currentState = '待提交';
    let history = [];
    let context = {
      appId: 'LA-20261119-001',
      submitTime: null,
      reviewer: null,
      timeoutCount: 0,
    };
    let timeoutTimer = null;
    let timeoutStart = null;

    // 状态转换
    function transition(action, reason = '') {
      const from = currentState;
      const to = action.nextState;

      history.push({
        timestamp: new Date(),
        from,
        to,
        action: action.name,
        reason,
      });

      currentState = to;

      // 更新上下文
      if (to === '待审核' && from === '待提交') {
        context.submitTime = new Date().toLocaleString('zh-CN');
      }
      if (to === '审核中') {
        context.reviewer = '张三（经办人）';
      }
      if (to === '超时退回') {
        context.timeoutCount++;
      }

      render();
      startTimeout();
    }

    // 开始超时计时
    function startTimeout() {
      clearTimeout(timeoutTimer);
      timeoutStart = null;

      const stateConfig = STATE_MACHINE[currentState];
      if (stateConfig.timeout) {
        timeoutStart = Date.now();
        const timeoutDuration = stateConfig.timeout;

        // 快速模拟：1 秒 = 1 天
        const simulatedDuration = (timeoutDuration / (24 * 60 * 60 * 1000)) * 1000;

        timeoutTimer = setTimeout(() => {
          transition(
            stateConfig.timeoutAction,
            `超时 ${timeoutDuration / (24 * 60 * 60 * 1000)} 天未处理`
          );
        }, simulatedDuration);

        updateTimer();
      }
    }

    // 更新计时器显示
    function updateTimer() {
      const timerEl = document.getElementById('timer');
      const stateConfig = STATE_MACHINE[currentState];

      if (!stateConfig.timeout || !timeoutStart) {
        timerEl.textContent = '无超时计时';
        timerEl.className = 'timer';
        return;
      }

      const elapsed = Date.now() - timeoutStart;
      const total = (stateConfig.timeout / (24 * 60 * 60 * 1000)) * 1000; // 模拟时间
      const remaining = Math.max(0, total - elapsed);
      const days = Math.ceil(remaining / 1000); // 1 秒 = 1 天

      timerEl.textContent = `距离超时还有 ${days} 天`;

      if (remaining < total * 0.3) {
        timerEl.className = 'timer';
      } else if (remaining < total * 0.6) {
        timerEl.className = 'timer warning';
      } else {
        timerEl.className = 'timer safe';
      }

      if (remaining > 0) {
        setTimeout(updateTimer, 100);
      }
    }

    // 渲染
    function render() {
      const stateConfig = STATE_MACHINE[currentState];

      // 当前状态
      document.getElementById('current-state').textContent = currentState;
      document.getElementById('state-meta').textContent = stateConfig.isFinal
        ? '终态'
        : `可执行 ${stateConfig.actions.length} 个操作`;

      // 上下文
      document.getElementById('app-id').textContent = context.appId;
      document.getElementById('submit-time').textContent = context.submitTime || '—';
      document.getElementById('reviewer').textContent = context.reviewer || '—';
      document.getElementById('timeout-count').textContent = context.timeoutCount;

      // 操作按钮
      const actionsEl = document.getElementById('actions');
      actionsEl.innerHTML = '';

      stateConfig.actions.forEach(action => {
        const btn = document.createElement('button');
        btn.className = `btn-${action.color}`;
        btn.textContent = action.name;
        btn.onclick = () => transition(action);
        actionsEl.appendChild(btn);
      });

      if (stateConfig.actions.length === 0) {
        const p = document.createElement('p');
        p.textContent = '无可用操作（终态）';
        p.style.color = '#666';
        actionsEl.appendChild(p);
      }

      // 历史记录
      const historyEl = document.getElementById('history');
      historyEl.innerHTML = history
        .slice()
        .reverse()
        .map(h => `
          <div class="history-item">
            <div class="history-time">${h.timestamp.toLocaleTimeString('zh-CN')}</div>
            <div class="history-transition">${h.from} → ${h.to}</div>
            <div class="history-reason">操作：${h.action}${h.reason ? ` (${h.reason})` : ''}</div>
          </div>
        `)
        .join('');
    }

    // 初始化
    render();
  </script>
</body>
</html>
```

## UI 原型模板（picker + 3 个变体）

参考 `prototype` 技能的 `PICKER.md`，生成：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>UI 原型：低保申请列表</title>
  <style>
    /* Picker 样式（参考 prototype 技能的 PICKER.md） */
    .proto-picker {
      position: fixed;
      bottom: 24px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 9999;
      display: flex;
      align-items: center;
      gap: 2px;
      padding: 4px;
      border-radius: 999px;
      background: rgba(10, 10, 10, 0.82);
      backdrop-filter: blur(12px);
      font-family: system-ui;
      font-size: 13px;
    }
    .proto-picker-item {
      padding: 0 12px;
      height: 28px;
      border: 0;
      border-radius: 999px;
      background: transparent;
      color: rgba(255, 255, 255, 0.55);
      cursor: pointer;
      transition: color 150ms ease-out;
    }
    .proto-picker-item[data-active] { color: #fff; }
    .proto-picker-highlight {
      position: absolute;
      top: 4px;
      left: 0;
      height: 28px;
      border-radius: 999px;
      background: rgba(255, 255, 255, 0.12);
      transition: transform 250ms cubic-bezier(0.23, 1, 0.32, 1), width 250ms cubic-bezier(0.23, 1, 0.32, 1);
    }

    /* 变体容器 */
    #stage { min-height: 600px; padding: 20px; background: #f5f5f5; }

    /* 变体 1：简洁风格 */
    .variant-clean { /* ... */ }

    /* 变体 2：密集风格 */
    .variant-dense { /* ... */ }

    /* 变体 3：卡片风格 */
    .variant-card { /* ... */ }
  </style>
</head>
<body>
  <div id="stage"></div>

  <nav class="proto-picker">
    <span class="proto-picker-highlight"></span>
    <button class="proto-picker-item" data-active data-variant="clean">简洁</button>
    <button class="proto-picker-item" data-variant="dense">密集</button>
    <button class="proto-picker-item" data-variant="card">卡片</button>
  </nav>

  <script>
    const variants = {
      clean: () => `<div class="variant-clean">简洁风格的列表...</div>`,
      dense: () => `<div class="variant-dense">密集风格的列表...</div>`,
      card: () => `<div class="variant-card">卡片风格的列表...</div>`,
    };

    let current = 'clean';

    function render() {
      document.getElementById('stage').innerHTML = variants[current]();
    }

    document.querySelectorAll('.proto-picker-item').forEach(btn => {
      btn.addEventListener('click', () => {
        current = btn.dataset.variant;
        document.querySelectorAll('.proto-picker-item').forEach(b => {
          b.toggleAttribute('data-active', b === btn);
        });
        render();
      });
    });

    render();
  </script>
</body>
</html>
```
