<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
    <title>DeepSeek 助手</title>
    <style>
        * {
            -webkit-tap-highlight-color: transparent;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: #f5f5f7;
            margin: 0;
            padding: 0;
            padding-bottom: 80px;
        }
        .chat-container {
            max-width: 600px;
            margin: 0 auto;
            background: white;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
        }
        .message {
            margin-bottom: 16px;
            display: flex;
        }
        .user-message {
            justify-content: flex-end;
        }
        .ai-message {
            justify-content: flex-start;
        }
        .bubble {
            max-width: 80%;
            padding: 12px 16px;
            border-radius: 20px;
            word-wrap: break-word;
        }
        .user-message .bubble {
            background: #007aff;
            color: white;
            border-bottom-right-radius: 4px;
        }
        .ai-message .bubble {
            background: #e5e5ea;
            color: black;
            border-bottom-left-radius: 4px;
        }
        .input-area {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: white;
            border-top: 1px solid #e5e5ea;
            padding: 12px;
            display: flex;
            gap: 10px;
            max-width: 600px;
            margin: 0 auto;
        }
        input {
            flex: 1;
            padding: 12px;
            border: 1px solid #e5e5ea;
            border-radius: 25px;
            font-size: 16px;
            background: #f5f5f7;
        }
        button {
            background: #007aff;
            color: white;
            border: none;
            border-radius: 25px;
            padding: 0 20px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
        }
        button:active {
            opacity: 0.7;
        }
        .loading {
            opacity: 0.6;
        }
        @media (prefers-color-scheme: dark) {
            body { background: #000; }
            .chat-container { background: #000; }
            .ai-message .bubble { background: #1c1c1e; color: white; }
            .input-area { background: #000; border-top-color: #1c1c1e; }
            input { background: #1c1c1e; color: white; border-color: #2c2c2e; }
        }
    </style>
</head>
<body>
<div class="chat-container">
    <div class="messages" id="messages">
        <div class="message ai-message">
            <div class="bubble">你好！我是 DeepSeek AI 助手，有什么我可以帮你的吗？</div>
        </div>
    </div>
</div>
<div class="input-area">
    <input type="text" id="userInput" placeholder="输入消息..." autocomplete="off">
    <button id="sendBtn">发送</button>
</div>

<script>
    // 🔧 修改这里：粘贴你的 Cloudflare Worker 地址
    const WORKER_URL = 'https://你的-worker域名.workers.dev';
    
    const messagesDiv = document.getElementById('messages');
    const userInput = document.getElementById('userInput');
    const sendBtn = document.getElementById('sendBtn');
    
    async function sendMessage() {
        const message = userInput.value.trim();
        if (!message) return;
        
        // 显示用户消息
        addMessage(message, 'user');
        userInput.value = '';
        
        // 显示加载状态
        const loadingId = addLoadingMessage();
        
        try {
            const response = await fetch(WORKER_URL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ message: message })
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            const data = await response.json();
            removeLoadingMessage(loadingId);
            
            if (data.error) {
                addMessage(`错误: ${data.error}`, 'ai');
            } else {
                addMessage(data.reply, 'ai');
            }
        } catch (error) {
            removeLoadingMessage(loadingId);
            addMessage(`网络错误: ${error.message}\n请检查 Worker 地址是否正确`, 'ai');
        }
        
        scrollToBottom();
    }
    
    function addMessage(text, type) {
        const messageDiv = document.createElement('div');
        messageDiv.className = `message ${type}-message`;
        const bubble = document.createElement('div');
        bubble.className = 'bubble';
        bubble.textContent = text;
        messageDiv.appendChild(bubble);
        messagesDiv.appendChild(messageDiv);
        scrollToBottom();
    }
    
    function addLoadingMessage() {
        const id = 'loading-' + Date.now();
        const loadingDiv = document.createElement('div');
        loadingDiv.id = id;
        loadingDiv.className = 'message ai-message loading';
        const bubble = document.createElement('div');
        bubble.className = 'bubble';
        bubble.textContent = '正在思考...';
        loadingDiv.appendChild(bubble);
        messagesDiv.appendChild(loadingDiv);
        scrollToBottom();
        return id;
    }
    
    function removeLoadingMessage(id) {
        const loadingMsg = document.getElementById(id);
        if (loadingMsg) loadingMsg.remove();
    }
    
    function scrollToBottom() {
        messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    // 事件监听
    sendBtn.addEventListener('click', sendMessage);
    userInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') sendMessage();
    });
    
    // 自动调整输入框高度
    userInput.addEventListener('input', function() {
        this.style.height = 'auto';
        this.style.height = Math.min(this.scrollHeight, 100) + 'px';
    });
</script>
</body>
</html>
