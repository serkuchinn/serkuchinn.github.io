<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Мессенджер</title>
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"> </script>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #f5f4f0;
      --surface: #ffffff;
      --surface2: #f0efe9;
      --border: rgba(0,0,0,0.1);
      --border2: rgba(0,0,0,0.18);
      --text: #1a1a18;
      --text2: #6b6b66;
      --text3: #9e9e97;
      --accent: #534AB7;
      --accent-dark: #3C3489;
      --accent-light: #EEEDFE;
      --green: #1D9E75;
      --red: #E24B4A;
      --radius: 12px;
      --radius-sm: 8px;
    }

    @media (prefers-color-scheme: dark) {
      :root {
        --bg: #1a1a18;
        --surface: #242422;
        --surface2: #2e2e2b;
        --border: rgba(255,255,255,0.1);
        --border2: rgba(255,255,255,0.18);
        --text: #f0efe9;
        --text2: #a0a09a;
        --text3: #6b6b66;
        --accent-light: #2a2660;
      }
    }

    html, body {
      height: 100%;
      background: var(--bg);
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      color: var(--text);
    }

    body {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
      min-height: 100vh;
    }

    .app {
      width: 100%;
      max-width: 700px;
      height: calc(100vh - 32px);
      max-height: 700px;
      background: var(--surface);
      border: 0.5px solid var(--border);
      border-radius: var(--radius);
      overflow: hidden;
      display: flex;
      flex-direction: column;
      box-shadow: 0 4px 40px rgba(0,0,0,0.08);
    }

    /* Header */
    .header {
      padding: 14px 18px;
      border-bottom: 0.5px solid var(--border);
      background: var(--surface2);
      display: flex;
      align-items: center;
      gap: 10px;
      flex-shrink: 0;
    }

    .logo {
      width: 32px;
      height: 32px;
      background: var(--accent);
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
    }

    .logo svg { width: 18px; height: 18px; }

    .header-info { flex: 1; min-width: 0; }

    .header-title {
      font-size: 15px;
      font-weight: 600;
      color: var(--text);
    }

    .header-status {
      font-size: 12px;
      color: var(--text3);
      display: flex;
      align-items: center;
      gap: 5px;
    }

    .dot {
      width: 7px;
      height: 7px;
      border-radius: 50%;
      background: var(--text3);
      transition: background 0.3s;
      flex-shrink: 0;
    }
    .dot.on { background: var(--green); }
    .dot.err { background: var(--red); }

    /* Setup panel */
    .setup {
      padding: 10px 18px;
      border-bottom: 0.5px solid var(--border);
      background: var(--surface2);
      display: flex;
      gap: 8px;
      align-items: center;
      flex-shrink: 0;
      flex-wrap: wrap;
    }

    .setup label {
      font-size: 12px;
      color: var(--text2);
      white-space: nowrap;
    }

    .setup input {
      flex: 1;
      min-width: 80px;
      border: 0.5px solid var(--border2);
      border-radius: var(--radius-sm);
      padding: 6px 10px;
      font-size: 13px;
      color: var(--text);
      background: var(--surface);
      outline: none;
      transition: border-color 0.15s;
    }

    .setup input:focus { border-color: var(--accent); }

    .join-btn {
      background: var(--accent);
      color: #fff;
      border: none;
      border-radius: var(--radius-sm);
      padding: 6px 16px;
      font-size: 13px;
      font-weight: 500;
      cursor: pointer;
      white-space: nowrap;
      transition: background 0.15s;
    }
    .join-btn:hover { background: var(--accent-dark); }

    /* Room hint */
    .room-hint {
      padding: 6px 18px;
      font-size: 11px;
      color: var(--text3);
      background: var(--surface2);
      border-bottom: 0.5px solid var(--border);
      display: none;
      align-items: center;
      justify-content: center;
      gap: 6px;
      flex-shrink: 0;
    }
    .room-hint.visible { display: flex; }

    .copy-btn {
      background: none;
      border: none;
      color: var(--accent);
      font-size: 11px;
      cursor: pointer;
      text-decoration: underline;
      padding: 0;
    }

    /* Messages */
    .messages {
      flex: 1;
      overflow-y: auto;
      padding: 16px 18px;
      display: flex;
      flex-direction: column;
      gap: 10px;
      scroll-behavior: smooth;
    }

    .messages::-webkit-scrollbar { width: 4px; }
    .messages::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 2px; }

    .sys-msg {
      text-align: center;
      font-size: 12px;
      color: var(--text3);
      padding: 2px 0;
      user-select: none;
    }

    .msg-group { display: flex; flex-direction: column; }
    .msg-group.me { align-items: flex-end; }
    .msg-group.them { align-items: flex-start; }

    .msg-name {
      font-size: 11px;
      color: var(--text2);
      margin-bottom: 3px;
      padding: 0 4px;
    }

    .bubble {
      max-width: 72%;
      padding: 9px 14px;
      border-radius: 16px;
      font-size: 14px;
      line-height: 1.55;
      word-break: break-word;
    }

    .bubble.me {
      background: var(--accent);
      color: #fff;
      border-bottom-right-radius: 4px;
    }

    .bubble.them {
      background: var(--surface2);
      color: var(--text);
      border-bottom-left-radius: 4px;
      border: 0.5px solid var(--border);
    }

    .msg-time {
      font-size: 11px;
      color: var(--text3);
      margin-top: 3px;
      padding: 0 4px;
    }

    /* Input area */
    .input-area {
      padding: 10px 14px;
      border-top: 0.5px solid var(--border);
      background: var(--surface);
      display: flex;
      gap: 8px;
      align-items: flex-end;
      flex-shrink: 0;
    }

    .msg-input {
      flex: 1;
      border: 0.5px solid var(--border2);
      border-radius: 20px;
      padding: 9px 16px;
      font-size: 14px;
      color: var(--text);
      background: var(--surface2);
      outline: none;
      resize: none;
      min-height: 38px;
      max-height: 120px;
      font-family: inherit;
      line-height: 1.4;
      transition: border-color 0.15s;
    }
    .msg-input:focus { border-color: var(--accent); }
    .msg-input:disabled { opacity: 0.5; cursor: not-allowed; }

    .send-btn {
      width: 38px;
      height: 38px;
      border-radius: 50%;
      background: var(--accent);
      border: none;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
      transition: background 0.15s, transform 0.1s;
    }
    .send-btn:hover { background: var(--accent-dark); }
    .send-btn:active { transform: scale(0.95); }
    .send-btn:disabled { background: var(--surface2); cursor: not-allowed; }

    .send-btn svg {
      width: 16px;
      height: 16px;
      fill: #fff;
    }
    .send-btn:disabled svg { fill: var(--text3); }

    /* Emoji bar */
    .emoji-bar {
      padding: 0 14px 8px;
      display: flex;
      gap: 6px;
      flex-shrink: 0;
    }
    .emoji-btn {
      background: none;
      border: none;
      font-size: 20px;
      cursor: pointer;
      border-radius: 6px;
      padding: 2px 4px;
      transition: background 0.1s;
    }
    .emoji-btn:hover { background: var(--surface2); }

    @media (max-width: 500px) {
      body { padding: 0; }
      .app { max-height: 100vh; height: 100vh; border-radius: 0; border: none; }
    }
  </style>
</head>
<body>

<div class="app">
  <div class="header">
    <div class="logo">
      <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </div>
    <div class="header-info">
      <div class="header-title" id="headerTitle">Мессенджер</div>
      <div class="header-status">
        <div class="dot" id="dot"></div>
        <span id="statusText">Введите имя и ID комнаты</span>
      </div>
    </div>
  </div>

  <div class="setup">
    <label>Имя:</label>
    <input id="nameInput" placeholder="Ваш псевдоним" maxlength="20" style="max-width:160px;" />
    <label>Комната:</label>
    <input id="roomInput" placeholder="ID комнаты" maxlength="40" />
    <button class="join-btn" id="joinBtn" onclick="joinRoom()">Войти</button>
  </div>

  <div class="room-hint" id="roomHint">
    Комната: <strong id="hintRoom"></strong> —
    <button class="copy-btn" onclick="copyRoom()">скопировать ID</button>
    и отправить собеседнику
  </div>

  <div class="messages" id="messages">
    <div class="sys-msg">👋 Добро пожаловать! Введите имя и ID комнаты выше.</div>
    <div class="sys-msg">Оба собеседника должны ввести одинаковый ID комнаты.</div>
  </div>

  <div class="emoji-bar">
    <button class="emoji-btn" onclick="insertEmoji('😊')">😊</button>
    <button class="emoji-btn" onclick="insertEmoji('👍')">👍</button>
    <button class="emoji-btn" onclick="insertEmoji('❤️')">❤️</button>
    <button class="emoji-btn" onclick="insertEmoji('😂')">😂</button>
    <button class="emoji-btn" onclick="insertEmoji('🔥')">🔥</button>
    <button class="emoji-btn" onclick="insertEmoji('👋')">👋</button>
    <button class="emoji-btn" onclick="insertEmoji('🎉')">🎉</button>
    <button class="emoji-btn" onclick="insertEmoji('😢')">😢</button>
  </div>

  <div class="input-area">
    <textarea class="msg-input" id="msgInput" placeholder="Напишите сообщение... (Enter — отправить)" rows="1" disabled></textarea>
    <button class="send-btn" id="sendBtn" disabled onclick="sendMsg()">
      <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
        <path d="M22 2L11 13M22 2L15 22L11 13M22 2L2 9L11 13"/>
      </svg>
    </button>
  </div>
</div>

<script>
  let client = null;
  let currentRoom = null;
  let myName = 'Аноним';
  let myId = Math.random().toString(36).slice(2, 10);

  // Generate random defaults
  const adj = ['Весёлый','Быстрый','Умный','Яркий','Добрый','Храбрый','Мудрый'];
  const nou = ['Кот','Волк','Орёл','Лис','Медведь','Тигр','Сова'];
  document.getElementById('nameInput').value =
    adj[Math.floor(Math.random()*adj.length)] + nou[Math.floor(Math.random()*nou.length)];
  document.getElementById('roomInput').value = Math.random().toString(36).slice(2, 8);

  function ts() {
    const d = new Date();
    return d.getHours().toString().padStart(2,'0') + ':' + d.getMinutes().toString().padStart(2,'0');
  }

  function addSys(text) {
    const el = document.getElementById('messages');
    const d = document.createElement('div');
    d.className = 'sys-msg';
    d.textContent = text;
    el.appendChild(d);
    el.scrollTop = el.scrollHeight;
  }

  function addMsg(name, text, isMe) {
    const el = document.getElementById('messages');
    const group = document.createElement('div');
    group.className = 'msg-group ' + (isMe ? 'me' : 'them');

    if (!isMe) {
      const n = document.createElement('div');
      n.className = 'msg-name';
      n.textContent = name;
      group.appendChild(n);
    }

    const b = document.createElement('div');
    b.className = 'bubble ' + (isMe ? 'me' : 'them');
    b.textContent = text;
    group.appendChild(b);

    const t = document.createElement('div');
    t.className = 'msg-time';
    t.textContent = ts();
    group.appendChild(t);

    el.appendChild(group);
    el.scrollTop = el.scrollHeight;
  }

  function setStatus(state, text) {
    const dot = document.getElementById('dot');
    dot.className = 'dot' + (state === 'on' ? ' on' : state === 'err' ? ' err' : '');
    document.getElementById('statusText').textContent = text;
  }

  function joinRoom() {
    const rawRoom = document.getElementById('roomInput').value.trim().replace(/[^a-zA-Z0-9\-_а-яёА-ЯЁ]/gi, '');
    const name = document.getElementById('nameInput').value.trim() || 'Аноним';

    if (!rawRoom) { alert('Введите ID комнаты!'); return; }

    myName = name;
    currentRoom = 'claudemsg_v1/' + rawRoom;

    document.getElementById('headerTitle').textContent = 'Комната: ' + rawRoom;
    document.getElementById('hintRoom').textContent = rawRoom;
    document.getElementById('roomHint').classList.add('visible');
    setStatus('', 'Подключение...');

    if (client) { try { client.end(true); } catch(e){} client = null; }

    const brokers = [
      'wss://broker.hivemq.com:8884/mqtt',
      'wss://test.mosquitto.org:8081/mqtt',
    ];

    function tryBroker(idx) {
      if (idx >= brokers.length) {
        setStatus('err', 'Ошибка: не удалось подключиться');
        addSys('❌ Не удалось подключиться к серверу. Обновите страницу и попробуйте снова.');
        return;
      }

      setStatus('', 'Подключение... (' + (idx+1) + '/' + brokers.length + ')');

      const c = window.mqtt.connect(brokers[idx], {
        clientId: 'cm_' + myId + '_' + Date.now(),
        reconnectPeriod: 0,
        connectTimeout: 8000,
        keepalive: 30,
        clean: true,
      });

      const timer = setTimeout(() => {
        c.end(true);
        tryBroker(idx + 1);
      }, 9000);

      c.on('connect', () => {
        clearTimeout(timer);
        client = c;
        setStatus('on', 'Подключено • ' + myName);
        c.subscribe(currentRoom + '/msg', { qos: 1 });
        c.subscribe(currentRoom + '/sys', { qos: 0 });

        document.getElementById('msgInput').disabled = false;
        document.getElementById('sendBtn').disabled = false;
        document.getElementById('joinBtn').textContent = 'Сменить';
        document.getElementById('msgInput').focus();

        c.publish(currentRoom + '/sys',
          JSON.stringify({ id: myId, name: myName, action: 'join' }),
          { qos: 0 });

        addSys('✅ Вы вошли как «' + myName + '». Поделитесь ID комнаты с собеседником!');
      });

      c.on('message', (topic, payload) => {
        try {
          const data = JSON.parse(payload.toString());
          if (topic.endsWith('/msg')) {
            if (data.id === myId) return;
            addMsg(data.name, data.text, false);
          } else if (topic.endsWith('/sys')) {
            if (data.id === myId) return;
            if (data.action === 'join') addSys('👤 ' + data.name + ' вошёл(а) в комнату');
            if (data.action === 'leave') addSys('👤 ' + data.name + ' покинул(а) комнату');
          }
        } catch(e) {}
      });

      c.on('error', () => {
        clearTimeout(timer);
        c.end(true);
        setTimeout(() => tryBroker(idx + 1), 300);
      });

      c.on('close', () => {
        if (client === c) {
          setStatus('err', 'Соединение разорвано');
          addSys('⚠️ Соединение прервано. Нажмите «Сменить» чтобы переподключиться.');
          document.getElementById('msgInput').disabled = true;
          document.getElementById('sendBtn').disabled = true;
        }
      });
    }

    tryBroker(0);
  }

  function sendMsg() {
    if (!client || !currentRoom) return;
    const input = document.getElementById('msgInput');
    const text = input.value.trim();
    if (!text) return;

    client.publish(currentRoom + '/msg',
      JSON.stringify({ id: myId, name: myName, text }),
      { qos: 1 });

    addMsg(myName, text, true);
    input.value = '';
    input.style.height = 'auto';
  }

  function copyRoom() {
    const id = document.getElementById('roomInput').value.trim().replace(/[^a-zA-Z0-9\-_а-яёА-ЯЁ]/gi, '');
    navigator.clipboard.writeText(id).then(() => {
      const btn = document.querySelector('.copy-btn');
      const orig = btn.textContent;
      btn.textContent = '✓ скопировано!';
      setTimeout(() => btn.textContent = orig, 2000);
    });
  }

  function insertEmoji(e) {
    const input = document.getElementById('msgInput');
    if (input.disabled) return;
    const start = input.selectionStart;
    const end = input.selectionEnd;
    input.value = input.value.slice(0, start) + e + input.value.slice(end);
    input.selectionStart = input.selectionEnd = start + e.length;
    input.focus();
  }

  document.getElementById('msgInput').addEventListener('keydown', e => {
    if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMsg(); }
  });

  document.getElementById('msgInput').addEventListener('input', function() {
    this.style.height = 'auto';
    this.style.height = Math.min(this.scrollHeight, 120) + 'px';
  });

  document.getElementById('roomInput').addEventListener('keydown', e => {
    if (e.key === 'Enter') joinRoom();
  });

  window.addEventListener('beforeunload', () => {
    if (client && currentRoom) {
      try {
        client.publish(currentRoom + '/sys',
          JSON.stringify({ id: myId, name: myName, action: 'leave' }),
          { qos: 0 });
      } catch(e) {}
    }
  });
</script>
</body>
</html>
