<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Входяшки</title>
  <style>
    html, body {
      margin: 0; padding: 0;
      height: 100%;
      font-family: Arial, sans-serif;

      background:
        radial-gradient(circle, #0b1e3d 40%, #000000 100%),
        radial-gradient(circle at 20% 30%, #fff 1px, transparent 2px),
        radial-gradient(circle at 80% 70%, #fff 1px, transparent 2px),
        radial-gradient(circle at 50% 50%, #fff 1px, transparent 2px);
      background-repeat: no-repeat;
      background-size: cover;

      color: #a0e7ff;
      transition: background 0.5s ease, color 0.5s ease;
    }
    #backgroundText, #message {
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      text-align: center;
      white-space: pre-line;
      flex-direction: column;
    }
    #backgroundText {
      font-size: 3.5em; /* 🔥 увеличено с 2.5em */
      z-index: 1;
    }
    #message {
      font-size: 3.5em;
      z-index: 2;
      color: #073763;
      background: linear-gradient(135deg, #e0f0ff, #a0c8ff);
      box-shadow: 0 4px 15px rgba(0, 60, 120, 0.4);
      border-radius: 12px;
      padding: 3rem 4rem;
      display: none;
      user-select: none;
      line-height: 1.2;
    }
    #enableSound {
      position: absolute;
      top: 10px;
      right: 10px;
      z-index: 1000;
      padding: 10px 20px;
      font-size: 1em;
      background: #222;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    /* Таблица */
    #backgroundText table {
      margin: auto;
      font-family: monospace;
      border-collapse: collapse;
      user-select: none;
      color: #a0e7ff;
    }
    #backgroundText td {
      padding: 2px 10px;
      border: 1px solid #555;
      white-space: nowrap;
      color: #a0e7ff;
      text-shadow: none;
      position: relative;
      transition: background-color 0.3s ease;
    }

    /* Заголовки */
    #backgroundText tr:first-child td {
      font-weight: bold;
      color: #00ff88;
      text-shadow: 0 0 12px #0f0, 0 0 20px #0f0;
      animation: pulseGlow 2s infinite alternate;
    }

    /* Медали — без анимаций */
    #backgroundText td:has(> .medal),
    #backgroundText td:contains("🥇"),
    #backgroundText td:contains("🥈"),
    #backgroundText td:contains("🥉") {
      animation: none !important;
      color: #a0e7ff;
      text-shadow: none;
    }

    /* Анимация бегущей подсветки по строкам */
    @keyframes rowHighlight {
      0%   { background-color: transparent; }
      50%  { background-color: rgba(0, 255, 255, 0.3); }
      100% { background-color: transparent; }
    }

    /* Пульс для заголовка */
    @keyframes pulseGlow {
      0% { text-shadow: 0 0 8px #0f0, 0 0 16px #0f0; }
      100% { text-shadow: 0 0 14px #0f0, 0 0 28px #0f0; }
    }
  </style>
</head>
<body>
  <div id="backgroundText">Загрузка...</div>
  <div id="message"></div>
  <button id="enableSound">🔊 Включить звук</button>

  <audio id="notifySound" src="mamont2.mp3"></audio>
  <audio id="sitdownSound" src="sitdown.mp3"></audio>

  <script>
    // === Твои URL'ы ===
    const BASE_URL = 'https://script.google.com/macros/s/AKfycbx8WNR24XB5dT5o2fiaVPGKHWksjnBXWWoaVUNObjzCkawbtIuJ9YkserXSKBG5kSQ/exec';
    // Фон: лист «Балы»!M1:M9 → ?bg=1
    const BG_URL = `${BASE_URL}?bg=1`;
    // Входяшки (лист «Киев днем»): без параметров
    const MESSAGES_URL = BASE_URL;

    const bgDiv = document.getElementById('backgroundText');
    const msgDiv = document.getElementById('message');
    const notifySound = document.getElementById('notifySound');
    const sitdownSound = document.getElementById('sitdownSound');
    const soundBtn = document.getElementById('enableSound');

    let queue = [];
    let showingNotification = false;
    let lastBgIncludedSitdown = false;

    const defaultBackground =
      'radial-gradient(circle, #0b1e3d 40%, #000000 100%),' +
      'radial-gradient(circle at 20% 30%, #fff 1px, transparent 2px),' +
      'radial-gradient(circle at 80% 70%, #fff 1px, transparent 2px),' +
      'radial-gradient(circle at 50% 50%, #fff 1px, transparent 2px)';
    const defaultColor = '#a0e7ff';

    soundBtn.addEventListener('click', () => {
      notifySound.play().then(() => {
        notifySound.pause();
        notifySound.currentTime = 0;
        return sitdownSound.play();
      }).then(() => {
        sitdownSound.pause();
        sitdownSound.currentTime = 0;
        soundBtn.style.display = 'none';
      }).catch(e => {
        alert("Браузер заблокировал звук. Попробуйте ещё раз.");
        console.error(e);
      });
    });

    async function fetchBackground() {
      try {
        const res = await fetch(BG_URL);
        const values = await res.json();

        let html = '<table>';
        values.forEach(row => {
          html += '<tr>';
          if (Array.isArray(row)) {
            let filteredRow = row.slice();
            while (filteredRow.length && filteredRow[filteredRow.length - 1] === "") {
              filteredRow.pop();
            }
            filteredRow.forEach(cell => { html += `<td>${cell}</td>`; });
          } else {
            html += `<td>${row}</td>`;
          }
          html += '</tr>';
        });
        html += '</table>';
        bgDiv.innerHTML = html;

        applyRowHighlightAnimation();

        const flatValues = values.flat().join(' ').toLowerCase();
        const containsSitdown = flatValues.includes('можем садиться');

        if (containsSitdown && !lastBgIncludedSitdown) {
          showSitdown();
        }
        lastBgIncludedSitdown = containsSitdown;
      } catch (e) {
        bgDiv.textContent = 'Ошибка загрузки данных: ' + e.message;
        console.error(e);
      }
    }

    function applyRowHighlightAnimation() {
      const rows = bgDiv.querySelectorAll('table tr');
      rows.forEach((row, index) => {
        row.style.animation = `rowHighlight 3s ease-in-out infinite`;
        row.style.animationDelay = `${index * 0.5}s`;
      });
    }

    function showSitdown() {
      if (showingNotification) return;
      showingNotification = true;
      msgDiv.textContent = 'МОЖЕМ САДИТЬСЯ';
      msgDiv.style.display = 'flex';
      bgDiv.style.display = 'none';

      document.body.style.background = 'linear-gradient(135deg, #e0f0ff, #a0c8ff)';
      document.body.style.color = '#073763';
      sitdownSound.play().catch(console.error);

      setTimeout(() => {
        msgDiv.style.display = 'none';
        bgDiv.style.display = 'flex';
        document.body.style.background = defaultBackground;
        document.body.style.color = defaultColor;
        showingNotification = false;
      }, 20000);

      setTimeout(() => {
        sitdownSound.pause();
        sitdownSound.currentTime = 0;
      }, 5000);
    }

    async function fetchMessages() {
      if (showingNotification || queue.length > 0) return;
      try {
        const res = await fetch(MESSAGES_URL);
        const messages = await res.json();
        const filtered = messages.filter(msg => !msg.toLowerCase().includes('можем садиться'));
        if (filtered.length > 0) {
          queue.push(...filtered);
          showNextMessage();
        }
      } catch (e) {
        console.error('Ошибка получения сообщений:', e);
      }
    }

    function showNextMessage() {
      if (queue.length === 0) return;
      showingNotification = true;

      const message = queue.shift();
      msgDiv.textContent = message;
      msgDiv.style.display = 'flex';
      bgDiv.style.display = 'none';

      document.body.style.background = 'linear-gradient(135deg, #e0f0ff, #a0c8ff)';
      document.body.style.color = '#073763';
      notifySound.play().catch(console.error);

      setTimeout(() => {
        msgDiv.style.display = 'none';
        bgDiv.style.display = 'flex';
        document.body.style.background = defaultBackground;
        document.body.style.color = defaultColor;
        showingNotification = false;
        if (queue.length > 0) showNextMessage();
      }, 20000);

      setTimeout(() => {
        notifySound.pause();
        notifySound.currentTime = 0;
      }, 5000);
    }

    fetchBackground();
    setInterval(fetchBackground, 30000);
    setInterval(fetchMessages, 5000);
  </script>
</body>
</html>
