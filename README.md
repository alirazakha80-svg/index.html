<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Chat</title>
  <style>
    /* Remove all browser default margins */
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      background-color: #202123; /* Dark background like ChatGPT */
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
    }

    /* Chat container full height */
    #chat-container {
      flex: 1;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      max-width: 900px;
      width: 100%;
      margin: 0 auto;
      padding: 20px;
      box-sizing: border-box;
      color: white;
    }

    /* Messages area */
    #messages {
      flex: 1;
      overflow-y: auto;
      padding-bottom: 20px;
    }

    .message {
      padding: 10px 15px;
      margin: 8px 0;
      border-radius: 8px;
      max-width: 80%;
      word-wrap: break-word;
    }

    .bot {
      background-color: #343541;
      align-self: flex-start;
    }

    .user {
      background-color: #10a37f;
      align-self: flex-end;
    }

    /* Input area fixed at bottom */
    #input-area {
      display: flex;
      gap: 10px;
      padding: 10px;
      background-color: #343541;
      border-radius: 8px;
    }

    #user-input {
      flex: 1;
      padding: 10px;
      border: none;
      border-radius: 5px;
      outline: none;
      font-size: 16px;
    }

    button {
      background-color: #10a37f;
      color: white;
      border: none;
      padding: 10px 15px;
      border-radius: 5px;
      cursor: pointer;
    }

    button:hover {
      background-color: #0e8f6e;
    }
  </style>
</head>
<body>
  <div id="chat-container">
    <div id="messages"></div>

    <div id="input-area">
      <input type="text" id="user-input" placeholder="Type your message...">
      <button onclick="sendMessage()">Send</button>
    </div>
  </div>

  <script>
    const messagesDiv = document.getElementById("messages");
    const userInput = document.getElementById("user-input");

    function sendMessage() {
      const text = userInput.value.trim();
      if (text === "") return;

      // Add user message
      addMessage(text, "user");
      userInput.value = "";

      // Simulate bot response
      setTimeout(() => {
        addMessage("This is a bot reply.", "bot");
      }, 500);
    }

    function addMessage(text, sender) {
      const msg = document.createElement("div");
      msg.className = `message ${sender}`;
      msg.innerText = text;
      messagesDiv.appendChild(msg);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
  </script>
</body>
</html>
