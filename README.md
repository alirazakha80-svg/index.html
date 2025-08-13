<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Intellio Chat</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      background: #f7f7f8;
      font-family: Arial, sans-serif;
    }

    /* Container to center chat */
    #chat-wrapper {
      display: flex;
      justify-content: center;
      align-items: flex-start;
      padding: 20px;
      height: 100%;
      box-sizing: border-box;
    }

    /* Chat box styling like ChatGPT */
    #chat-container {
      background: #fff;
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0,0,0,0.1);
      width: 900px;
      height: calc(100vh - 40px);
      overflow: hidden;
    }
  </style>
</head>
<body>

  <div id="chat-wrapper">
    <div id="chat-container"></div>
  </div>

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
    });
  </script>
</body>
</html>
