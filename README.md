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
    }
    #input-container {
        display: flex;
        gap: 5px;
    }
    #input {
        flex: 1;
        padding: 10px;
        border-radius: 5px;
        border: 1px solid #ccc;
    }
    #send, #voice {
        background: #007bff;
        color: white;
        border: none;
        padding: 10px 15px;
        cursor: pointer;
        border-radius: 5px;
    }
    footer {
        text-align: right;
        padding: 5px;
        font-size: 12px;
        color: #777;
    }
</style>
</head>
<body>

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

// Text-to-speech
function speak(text) {
    const speech = new SpeechSynthesisUtterance(text);
    speech.lang = "en-US";
    window.speechSynthesis.speak(speech);
}
</script>

</body>
</html>
