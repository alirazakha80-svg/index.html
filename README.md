<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Ali Raza - AI Chatbot</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      background: #fff;
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
    }

    #chat-container {
      flex: 1;
      height: 100%;
      width: 100%;
    }

    iframe,
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container,
    .vf-chat__content {
      width: 100% !important;
      height: 100% !important;
      margin: 0 !important;
      border-radius: 0 !important;
    }

    footer {
      text-align: center;
      padding: 10px;
      font-size: 14px;
      color: #555;
      background-color: #f1f1f1;
    }

    @media (max-width: 768px) {
      html, body {
        height: 100%;
      }
      #chat-container {
        height: calc(100% - 40px); /* leave space for footer */
      }
    }
  </style>
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>

  <div id="chat-container"></div>

  <footer>Powered by Ali Raza</footer>

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
