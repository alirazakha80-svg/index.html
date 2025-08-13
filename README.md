<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Fullscreen Chat</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      height: 100vh;
      background-color: #f5f5f5;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .chat-wrapper {
      width: 100%;
      height: 100%;
      background-color: white;
      display: flex;
      flex-direction: column;
      border-radius: 0;
      box-shadow: none;
    }

    .chat-header {
      background-color: #555;
      color: white;
      padding: 10px;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    .chat-messages {
      flex: 1;
      padding: 20px;
      overflow-y: auto;
    }

    .message {
      max-width: 70%;
      margin-bottom: 10px;
      padding: 10px 14px;
      border-radius: 8px;
    }

    .bot {
      background-color: #e5e5e5;
      align-self: flex-start;
    }

    .user {
      background-color: #4a90e2;
      color: white;
      align-self: flex-end;
    }

    .chat-input {
      display: flex;
      padding: 10px;
      border-top: 1px solid #ccc;
    }

    .chat-input input {
      flex: 1;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 6px;
      outline: none;
    }

    .chat-input button {
      margin-left: 8px;
      padding: 10px 16px;
      background-color: #4a90e2;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <div class="chat-wrapper">
    <div class="chat-header">
      <span>Intellio</span>
      <div>
        🔇 ⟳
      </div>
    </div>
    <div class="chat-messages" id="chatMessages">
      <div class="message bot">Hi! How can I help you today?</div>
      <div class="message user">I want this chat to be fullscreen.</div>
    </div>
    <div class="chat-input">
      <input type="text" id="userInput" placeholder="Type a message...">
      <button onclick="sendMessage()">Send</button>
    </div>
  </div>

  <script>
    function sendMessage() {
      const input = document.getElementById('userInput');
      const chatMessages = document.getElementById('chatMessages');

      if (input.value.trim() === '') return;

      // Add user message
      const userMsg = document.createElement('div');
      userMsg.classList.add('message', 'user');
      userMsg.textContent = input.value;
      chatMessages.appendChild(userMsg);

      // Auto bot reply
      setTimeout(() => {
        const botMsg = document.createElement('div');
        botMsg.classList.add('message', 'bot');
        botMsg.textContent = "This is a bot reply.";
        chatMessages.appendChild(botMsg);
        chatMessages.scrollTop = chatMessages.scrollHeight;
      }, 500);

      input.value = '';
      chatMessages.scrollTop = chatMessages.scrollHeight;
    }
  </script>

</body>
</html>
