<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📝 GitHub Блокнот</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: #0d1117;
            color: #c9d1d9;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .toolbar {
            background: #161b22;
            padding: 12px 20px;
            border-bottom: 1px solid #30363d;
            display: flex;
            align-items: center;
            gap: 12px;
            flex-wrap: wrap;
        }
        .toolbar h1 {
            font-size: 20px;
            color: #f0f6fc;
            margin-right: 20px;
        }
        .toolbar .spacer { flex: 1; }
        .btn {
            padding: 6px 16px;
            border: none;
            border-radius: 6px;
            font-size: 13px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            white-space: nowrap;
        }
        .btn-primary { background: #238636; color: white; }
        .btn-primary:hover { background: #2ea043; }
        .btn-secondary { background: #21262d; color: #c9d1d9; border: 1px solid #30363d; }
        .btn-secondary:hover { background: #30363d; }
        .btn-publish { background: #1f6feb; color: white; }
        .btn-publish:hover { background: #388bfd; }
        .btn-danger { background: #da3633; color: white; }
        .btn-danger:hover { background: #f85149; }
        .btn-success { background: #238636; color: white; }
        .btn-success:hover { background: #2ea043; }
        
        .status {
            font-size: 13px;
            padding: 4px 12px;
            background: #21262d;
            border-radius: 20px;
            border: 1px solid #30363d;
        }
        .status.success { color: #3fb950; border-color: #238636; }
        .status.error { color: #f85149; border-color: #da3633; }
        .status.loading { color: #d29922; border-color: #bb8009; }
        .status.warning { color: #f0883e; border-color: #d29922; }

        .editor-container { flex: 1; padding: 20px; background: #0d1117; }
        textarea {
            width: 100%;
            height: 100%;
            background: #0d1117;
            color: #c9d1d9;
            border: 1px solid #30363d;
            border-radius: 8px;
            padding: 16px;
            font-size: 15px;
            font-family: 'Consolas', 'Courier New', monospace;
            line-height: 1.6;
            resize: none;
            outline: none;
        }
        textarea:focus { border-color: #58a6ff; }
        textarea::placeholder { color: #484f58; }

        .footer {
            background: #161b22;
            padding: 8px 20px;
            border-top: 1px solid #30363d;
            display: flex;
            justify-content: space-between;
            font-size: 12px;
            color: #8b949e;
            flex-wrap: wrap;
        }
        .footer .stats { display: flex; gap: 20px; flex-wrap: wrap; }

        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.8);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal.active { display: flex; }
        .modal-content {
            background: #161b22;
            padding: 30px;
            border-radius: 12px;
            border: 1px solid #30363d;
            width: 500px;
            max-width: 95%;
        }
        .modal-content h2 { margin-bottom: 20px; color: #f0f6fc; }
        .modal-content label {
            display: block;
            margin: 12px 0 4px;
            font-size: 14px;
            color: #8b949e;
        }
        .modal-content input {
            width: 100%;
            padding: 8px 12px;
            background: #0d1117;
            border: 1px solid #30363d;
            color: #c9d1d9;
            border-radius: 6px;
            font-size: 14px;
        }
        .modal-content input:focus {
            border-color: #58a6ff;
            outline: none;
        }
        .modal-content .hint {
            font-size: 12px;
            color: #8b949e;
            margin-top: 4px;
        }
        .modal-buttons {
            margin-top: 20px;
            display: flex;
            gap: 10px;
            justify-content: flex-end;
        }
        .debug-info {
            background: #0d1117;
            padding: 8px 12px;
            border-radius: 6px;
            font-size: 12px;
            color: #8b949e;
            margin-top: 10px;
            border: 1px solid #30363d;
            max-height: 100px;
            overflow-y: auto;
            font-family: monospace;
            white-space: pre-wrap;
        }

        @media (max-width: 700px) {
            .toolbar { flex-direction: column; align-items: stretch; }
            .toolbar h1 { margin-right: 0; text-align: center; }
            .status { text-align: center; }
        }
    </style>
</head>
<body>

    <!-- Модалка настроек -->
    <div class="modal" id="settingsModal">
        <div class="modal-content">
            <h2>⚙️ Настройки GitHub</h2>
            
            <label>🔑 Personal Access Token (classic)</label>
            <input type="password" id="tokenInput" placeholder="ghp_xxxxxxxxxxxxxxxxxxxx">
            <div class="hint">Создайте токен в GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)</div>
            
            <label>📁 Репозиторий</label>
            <input type="text" id="repoInput" placeholder="username/repo">
            <div class="hint">Например: story987/App_gram_ax</div>
            
            <label>📄 Путь к файлу</label>
            <input type="text" id="filepathInput" placeholder="notes/заметки.txt">
            <div class="hint">Файл будет создан автоматически при публикации</div>
            
            <div class="hint" id="connectionStatus" style="margin-top:12px; padding:8px; background:#0d1117; border-radius:6px;">🔍 Введите данные и нажмите "Проверить"</div>
            
            <div class="modal-buttons">
                <button class="btn btn-secondary" onclick="closeSettings()">Закрыть</button>
                <button class="btn btn-secondary" onclick="testConnection()">🔌 Проверить</button>
                <button class="btn btn-primary" onclick="saveSettings()">💾 Сохранить</button>
            </div>
        </div>
    </div>

    <!-- Основной интерфейс -->
    <div class="toolbar">
        <h1>📝 Блокнот</h1>
        <button class="btn btn-secondary" onclick="openSettings()">⚙️ Настройки</button>
        <button class="btn btn-secondary" onclick="loadFromGitHub()">📥 Загрузить</button>
        <button class="btn btn-publish" onclick="publishToGitHub()">🚀 Опубликовать</button>
        <button class="btn btn-secondary" onclick="clearNote()">🗑️ Очистить</button>
        <span class="status" id="statusBar">● Готово</span>
    </div>

    <div class="editor-container">
        <textarea id="editor" placeholder="Введите текст заметки..."></textarea>
    </div>

    <div class="footer">
        <span>📄 <span id="fileName">не сохранено</span></span>
        <div class="stats">
            <span>📏 <span id="charCount">0</span> симв.</span>
            <span>📊 <span id="wordCount">0</span> слов</span>
            <span>⏱️ <span id="lastSaved">—</span></span>
        </div>
    </div>

    <script>
        // ===== Управление настройками =====
        function loadSettings() {
            document.getElementById('tokenInput').value = localStorage.getItem('gh_token') || '';
            document.getElementById('repoInput').value = localStorage.getItem('gh_repo') || '';
            document.getElementById('filepathInput').value = localStorage.getItem('gh_filepath') || 'note.txt';
            updateFileName();
        }

        function openSettings() {
            loadSettings();
            document.getElementById('settingsModal').classList.add('active');
        }

        function closeSettings() {
            document.getElementById('settingsModal').classList.remove('active');
        }

        function saveSettings() {
            const token = document.getElementById('tokenInput').value.trim();
            const repo = document.getElementById('repoInput').value.trim();
            const filepath = document.getElementById('filepathInput').value.trim();
            
            if (!token || !repo || !filepath) {
                setStatus('⚠️ Заполните все поля!', 'error');
                return;
            }
            
            localStorage.setItem('gh_token', token);
            localStorage.setItem('gh_repo', repo);
            localStorage.setItem('gh_filepath', filepath);
            updateFileName();
            setStatus('✅ Настройки сохранены', 'success');
            closeSettings();
            testConnection();
        }

        function getConfig() {
            return {
                token: localStorage.getItem('gh_token'),
                repo: localStorage.getItem('gh_repo'),
                filepath: localStorage.getItem('gh_filepath') || 'note.txt'
            };
        }

        function updateFileName() {
            const path = getConfig().filepath;
            const name = path.split('/').pop() || 'не сохранено';
            document.getElementById('fileName').textContent = name;
        }

        // ===== Тест подключения =====
        async function testConnection() {
            const token = document.getElementById('tokenInput').value.trim() || localStorage.getItem('gh_token');
            const repo = document.getElementById('repoInput').value.trim() || localStorage.getItem('gh_repo');
            
            if (!token || !repo) {
                document.getElementById('connectionStatus').textContent = '❌ Введите токен и репозиторий';
                return;
            }

            document.getElementById('connectionStatus').textContent = '⏳ Проверка подключения...';
            
            try {
                // Проверяем токен
                const userResponse = await fetch('https://api.github.com/user', {
                    headers: {
                        'Authorization': `token ${token}`,
                        'Accept': 'application/vnd.github.v3+json'
                    }
                });

                if (!userResponse.ok) {
                    if (userResponse.status === 401) {
                        throw new Error('Токен недействителен или истёк');
                    }
                    throw new Error(`Ошибка ${userResponse.status}: ${userResponse.statusText}`);
                }

                const userData = await userResponse.json();
                
                // Проверяем репозиторий
                const repoResponse = await fetch(`https://api.github.com/repos/${repo}`, {
                    headers: {
                        'Authorization': `token ${token}`,
                        'Accept': 'application/vnd.github.v3+json'
                    }
                });

                if (!repoResponse.ok) {
                    if (repoResponse.status === 404) {
                        throw new Error(`Репозиторий "${repo}" не найден`);
                    }
                    throw new Error(`Ошибка доступа к репозиторию (${repoResponse.status})`);
                }

                document.getElementById('connectionStatus').innerHTML = `✅ Подключено! Пользователь: <strong>${userData.login}</strong>`;
                setStatus('✅ Подключение к GitHub установлено', 'success');
                
            } catch (error) {
                document.getElementById('connectionStatus').textContent = `❌ ${error.message}`;
                setStatus('❌ Ошибка подключения', 'error');
                console.error('Connection error:', error);
            }
        }

        // ===== Статус бар =====
        function setStatus(text, type = '') {
            const bar = document.getElementById('statusBar');
            bar.textContent = text;
            bar.className = 'status' + (type ? ' ' + type : '');
        }

        // ===== Счётчики =====
        function updateStats() {
            const text = document.getElementById('editor').value;
            document.getElementById('charCount').textContent = text.length;
            const words = text.trim() ? text.trim().split(/\s+/).length : 0;
            document.getElementById('wordCount').textContent = words;
        }

        document.getElementById('editor').addEventListener('input', updateStats);

        // ===== Работа с GitHub =====
        async function loadFromGitHub() {
            const config = getConfig();
            if (!config.token || !config.repo) {
                setStatus('⚠️ Сначала настройте GitHub', 'error');
                openSettings();
                return;
            }

            setStatus('⏳ Загрузка...', 'loading');
            try {
                const url = `https://api.github.com/repos/${config.repo}/contents/${config.filepath}`;
                
                const response = await fetch(url, {
                    headers: {
                        'Authorization': `token ${config.token}`,
                        'Accept': 'application/vnd.github.v3+json'
                    }
                });

                if (response.status === 404) {
                    document.getElementById('editor').value = '';
                    setStatus('ℹ️ Файл не найден. Создайте новый и опубликуйте.', 'warning');
                    updateStats();
                    return;
                }

                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }

                const data = await response.json();
                const content = atob(data.content);
                document.getElementById('editor').value = content;
                updateStats();
                setStatus(`✅ Загружено (SHA: ${data.sha.substring(0, 7)})`, 'success');
                document.getElementById('lastSaved').textContent = new Date().toLocaleTimeString();
                localStorage.setItem('gh_file_sha', data.sha);
                
            } catch (error) {
                setStatus(`❌ ${error.message}`, 'error');
                console.error('Load error:', error);
            }
        }

        async function publishToGitHub() {
            const config = getConfig();
            if (!config.token || !config.repo) {
                setStatus('⚠️ Сначала настройте GitHub', 'error');
                openSettings();
                return;
            }

            const content = document.getElementById('editor').value;
            if (!content.trim()) {
                setStatus('⚠️ Нечего публиковать — заметка пуста', 'error');
                return;
            }

            setStatus('⏳ Публикация...', 'loading');
            try {
                const url = `https://api.github.com/repos/${config.repo}/contents/${config.filepath}`;
                
                // Получаем SHA для обновления
                let sha = localStorage.getItem('gh_file_sha') || null;
                if (!sha) {
                    try {
                        const getResponse = await fetch(url, {
                            headers: { 
                                'Authorization': `token ${config.token}`,
                                'Accept': 'application/vnd.github.v3+json'
                            }
                        });
                        if (getResponse.ok) {
                            const data = await getResponse.json();
                            sha = data.sha;
                        }
                    } catch (e) {}
                }

                const payload = {
                    message: `Обновление заметки ${new Date().toLocaleString()}`,
                    content: btoa(unescape(encodeURIComponent(content))),
                    sha: sha || undefined
                };

                const response = await fetch(url, {
                    method: 'PUT',
                    headers: {
                        'Authorization': `token ${config.token}`,
                        'Content-Type': 'application/json',
                        'Accept': 'application/vnd.github.v3+json'
                    },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    const error = await response.json();
                    throw new Error(error.message || `HTTP ${response.status}`);
                }

                const result = await response.json();
                localStorage.setItem('gh_file_sha', result.content.sha);
                setStatus(`🚀 Опубликовано!`, 'success');
                document.getElementById('lastSaved').textContent = new Date().toLocaleTimeString();
                updateFileName();
                
            } catch (error) {
                setStatus(`❌ ${error.message}`, 'error');
                console.error('Publish error:', error);
            }
        }

        // ===== Очистка =====
        function clearNote() {
            if (document.getElementById('editor').value && !confirm('Очистить заметку?')) return;
            document.getElementById('editor').value = '';
            updateStats();
            setStatus('🗑️ Очищено', '');
        }

        // ===== Автосохранение черновика =====
        function autoSave() {
            const content = document.getElementById('editor').value;
            localStorage.setItem('note_draft', content);
        }

        document.getElementById('editor').addEventListener('input', () => {
            updateStats();
            autoSave();
        });

        function loadDraft() {
            const draft = localStorage.getItem('note_draft');
            if (draft) {
                document.getElementById('editor').value = draft;
                updateStats();
            }
        }

        // ===== Инициализация =====
        loadSettings();
        loadDraft();
        updateFileName();
        updateStats();

        // Автоматическая проверка при загрузке
        if (localStorage.getItem('gh_token') && localStorage.getItem('gh_repo')) {
            setTimeout(() => testConnection(), 1000);
        } else {
            setTimeout(() => {
                setStatus('⚙️ Настройте GitHub для публикации', 'warning');
                openSettings();
            }, 500);
        }

        // ===== Горячие клавиши =====
        document.addEventListener('keydown', (e) => {
            if ((e.ctrlKey || e.metaKey) && e.key === 's') {
                e.preventDefault();
                publishToGitHub();
            }
            if ((e.ctrlKey || e.metaKey) && e.key === 'o') {
                e.preventDefault();
                loadFromGitHub();
            }
        });

        // ===== Определяем, что мы на GitHub Pages =====
        console.log('📍 Блокнот запущен на:', window.location.href);
        if (window.location.hostname.includes('github.io')) {
            console.log('✅ GitHub Pages — CORS не должен мешать');
        }
    </script>
</body>
</html>