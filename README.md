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
      font-family: Arial, sans-serif;
      background: #fff;
    }

    #chat-container {
      height: 100%;
      width: 100%;
    }

    footer {
      text-align: center;
      padding: 10px;
      font-size: 14px;
      color: #555;
      background-color: #f1f1f1;
    }
  </style>
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>
  <div id="chat-container"></div>

  <footer>Powered by Ali Raza</footer>

  <script type="text/javascript">
    window.addEventListener("DOMContentLoaded", function() {
      var v = document.createElement("script");
      v.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
      v.type = "text/javascript";
      v.async = true;
      v.onload = function() {
        window.voiceflow.chat.load({
          verify: { projectID: '689c3b1e9d300c90a54798bf' },
          url: 'https://general-runtime.voiceflow.com',
          versionID: 'production',
          render: { mode: 'embedded', target: document.getElementById("chat-container") },
          autostart: true
        });
      };
      document.head.appendChild(v);
    });
  </script>
</body>
</html>
