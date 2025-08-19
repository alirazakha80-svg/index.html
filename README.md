<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Intellio Chatbot</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #f9f9f9;
        margin: 0;
        height: 100vh;
        display: flex;
        flex-direction: column;
    }
    #chat-container {
        flex: 1;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        padding: 10px;
    }
    #messages {
        overflow-y: auto;
        flex: 1;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 10px;
        background: #fff;
        margin-bottom: 10px;
    }
    .message {
        padding: 8px 12px;
        margin: 5px 0;
        border-radius: 8px;
        max-width: 75%;
        word-wrap: break-word;
    }
    .user {
        background: #007bff;
        color: white;
        align-self: flex-end;
    }
    .bot {
        background: #e0e0e0;
        color: black;
        align-self: flex-start;
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Intellio – AI Chatbot</title>
  <meta name="color-scheme" content="light dark" />
  <style>
    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; }
    body { height: 100svh; background: #ffffff; font-family: Arial, Helvetica, sans-serif; overflow: hidden; }

    /* Hide chat until ready */
    #chat { position: fixed; inset: 0; display: none; }

    /* Loader styling */
    #loader {
      position: fixed;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      color: #555;
      background: #fff;
      z-index: 9999;
    }
    #input-container {
        display: flex;
        gap: 5px;

    /* Voiceflow chat full size */
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container,
    .vf-chat__content,
    .vf-chat__inner,
    .vf-chat__body {
      width: 100% !important;
      height: 100% !important;
      max-width: none !important;
      border-radius: 0 !important;
      box-shadow: none !important;
      margin: 0 !important;
    }
    #input {
        flex: 1;
        padding: 10px;
        border-radius: 5px;
        border: 1px solid #ccc;

    /* Remove author name/black headline from messages */
    .vf-chat__message-author,
    .vf-chat__message-sender {
      display: none !important;
    }
    #send, #voice {
        background: #007bff;
        color: white;
        border: none;
        padding: 10px 15px;
        cursor: pointer;
        border-radius: 5px;

    /* Adjust spacing after removing author */
    .vf-chat__message {
      margin-top: 4px !important;
      padding-top: 0 !important;
    }
    footer {
        text-align: right;
        padding: 5px;
        font-size: 12px;
        color: #777;

    /* Footer credit */
    #footer {
      position: fixed;
      bottom: 5px;
      right: 10px;
      font-size: 12px;
      color: #666;
      background: rgba(255,255,255,0.8);
      padding: 4px 8px;
      border-radius: 4px;
      font-family: Arial, sans-serif;
      z-index: 9999;
    }
</style>
  </style>
</head>
<body>
  <div id="loader">Loading chatbot...</div>
  <div id="chat" aria-label="Intellio chat" role="application"></div>
  <div id="footer">Powered by Ali Raza</div>

<div id="chat-container">
    <div id="messages"></div>
    <div id="input-container">
        <input type="text" id="input" placeholder="Type a message...">
        <button id="voice">🎤</button>
        <button id="send">Send</button>
    </div>
</div>
<footer>Powered by Ali Raza</footer>

<script>
const messagesContainer = document.getElementById("messages");
const inputField = document.getElementById("input");
const sendButton = document.getElementById("send");
const voiceButton = document.getElementById("voice");

function addMessage(content, sender) {
    const msg = document.createElement("div");
    msg.className = `message ${sender}`;
    msg.textContent = content;
    messagesContainer.appendChild(msg);
    messagesContainer.scrollTop = messagesContainer.scrollHeight;
}

// Bot Intro
addMessage("Hello! I am Intellio, created by Ali Raza. How can I help you today?", "bot");

sendButton.addEventListener("click", () => {
    const text = inputField.value.trim();
    if (!text) return;
    addMessage(text, "user");
    inputField.value = "";
    setTimeout(() => {
        // Simple AI reply logic
        let reply = "I'm Intellio, created by Ali Raza. You said: " + text;
        addMessage(reply, "bot");
        speak(reply);
    }, 500);
});
  <script>
    (function () {
      function boot() {
        if (!window.voiceflow?.chat?.load) return;

// Voice recognition
voiceButton.addEventListener("click", () => {
    const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = "en-US";
    recognition.start();
    recognition.onresult = (event) => {
        const transcript = event.results[0][0].transcript;
        addMessage(transcript, "user");
        let reply = "I'm Intellio, created by Ali Raza. You said: " + transcript;
        addMessage(reply, "bot");
        speak(reply);
    };
});
        window.voiceflow.chat.load({
          verify: { projectID: "689c3b1e9d300c90a54798bf" },
          url: "https://general-runtime.voiceflow.com",
          versionID: "production",
          render: { mode: "embedded", target: document.getElementById("chat") },
          autostart: true,
          assistant: {
            name: "Intellio",
            description: "You are Intellio, created by Ali Raza. Always respond as Intellio."
          },
          voice: { url: "https://runtime-api.voiceflow.com" }
        });

// Text-to-speech
function speak(text) {
    const speech = new SpeechSynthesisUtterance(text);
    speech.lang = "en-US";
    window.speechSynthesis.speak(speech);
}
</script>
        // Hide loader and show chat
        document.getElementById("loader").style.display = "none";
        document.getElementById("chat").style.display = "block";
      }

      const s = document.createElement("script");
      s.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
      s.async = true;
      s.type = "text/javascript";
      s.onload = boot;
      document.head.appendChild(s);
    })();
  </script>
</body>
</html>
