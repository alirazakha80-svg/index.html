<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Intellio - AI Chatbot</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      background: #fff;
      font-family: Arial, sans-serif;
      overflow: hidden;
    }
    #chat-container {
      height: calc(100vh - 50px);
      width: 100vw;
    }

    /* New Chat button styling */
    #new-chat-btn {
      height: 50px;
      background: #10a37f;
      color: white;
      border: none;
      width: 100%;
      font-size: 16px;
      cursor: pointer;
    }
    #new-chat-btn:hover {
      background: #0e8f6f;
    }

    /* Force Voiceflow widget to use full width */
    iframe, 
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container {
      width: 100% !important;
      height: 100% !important;
      max-width: 100% !important;
      margin: 0 !important;
      border-radius: 0 !important;
    }

    /* Optional: match ChatGPT light mode background */
    .vf-chat__container {
      background-color: #ffffff !important;
    }
  </style>
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>

  <!-- New Chat Button -->
  <button id="new-chat-btn">+ New Chat</button>

  <!-- Chat Window -->
  <div id="chat-container"></div>
  
  <script type="text/javascript">
    function loadChat() {
      window.voiceflow.chat.load({
        verify: { projectID: '689c3b1e9d300c90a54798bf' },
        url: 'https://general-runtime.voiceflow.com',
        versionID: 'production',
        render: { mode: 'embedded', target: document.getElementById("chat-container") },
        autostart: true
      });
    }

    window.addEventListener("DOMContentLoaded", function() {
      var v = document.createElement("script");
      v.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
      v.type = "text/javascript";
      v.async = true;
      v.onload = loadChat;
      document.head.appendChild(v);

      // Handle New Chat button click
      document.getElementById("new-chat-btn").addEventListener("click", function() {
        document.getElementById("chat-container").innerHTML = ""; // Clear chat
        loadChat(); // Reload chat widget
      });
    });
  </script>
</body>
</html>
