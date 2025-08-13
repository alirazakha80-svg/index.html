<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Student Chatbot - Full Screen</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      overflow: hidden;
      background-color: #f9f9f9;
      font-family: Arial, sans-serif;
    }

    #chat-container {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="chat-container"></div>

  <script type="text/javascript">
    window.addEventListener("DOMContentLoaded", function() {
      const script = document.createElement("script");
      script.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
      script.type = "module"; // must be module for Voiceflow
      script.async = true;
      script.onload = function() {
        window.voiceflow.chat.load({
          verify: { projectID: '689c3b1e9d300c90a54798bf' },
          url: 'https://general-runtime.voiceflow.com',
          versionID: 'production',
          render: { mode: 'embedded', target: document.getElementById("chat-container") },
          autostart: true
        });
      };
      document.body.appendChild(script);
    });
  </script>
</body>
</html>
