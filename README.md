<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Intellio - AI Chat</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f5f5f5; margin: 0; padding: 0; }
    #chat-container { max-width: 600px; margin: 40px auto; background: #fff; border-radius: 10px; padding: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
    .message { padding: 10px; margin: 10px 0; border-radius: 6px; }
    .user { background: #007bff; color: white; text-align: right; }
    .bot { background: #f1f1f1; text-align: left; }
    #input-container { display: flex; }
    #user-input { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 6px; }
    #send-btn { margin-left: 10px; padding: 10px 20px; border: none; background: #007bff; color: white; border-radius: 6px; cursor: pointer; }
  </style>
</head>
<body>
  <div id="chat-container">
    <h2>🤖 Intellio</h2>
    <div id="chat-box"></div>
    <div id="input-container">
      <input type="text" id="user-input" placeholder="Type a message..." />
      <button id="send-btn">Send</button>
    </div>
    <p id="plan-status">🆓 Free Plan (5 messages left)</p>
    <button onclick="upgradePlan()">Upgrade to Pro 🚀</button>
  </div>

  <script>
    let freeMessages = 5;

    async function sendMessage() {
      const input = document.getElementById("user-input");
      const message = input.value;
      if (!message) return;

      const chatBox = document.getElementById("chat-box");

      // show user message
      chatBox.innerHTML += `<div class="message user">${message}</div>`;
      input.value = "";

      if (freeMessages <= 0) {
        chatBox.innerHTML += `<div class="message bot">⚠️ Free plan limit reached. Upgrade to Pro 🚀</div>`;
        return;
      }

      freeMessages--;
      document.getElementById("plan-status").innerText = `🆓 Free Plan (${freeMessages} messages left)`;

      // send to backend
      let res = await fetch("http://localhost:5000/chat", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ message })
      });

      let data = await res.json();
      chatBox.innerHTML += `<div class="message bot">${data.reply}</div>`;
    }

    document.getElementById("send-btn").addEventListener("click", sendMessage);

    function upgradePlan() {
      alert("💳 Upgrade system coming soon! We’ll add Stripe/PayPal.");
    }
  </script>
</body>
</html>
