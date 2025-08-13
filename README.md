<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Intellio</title>
  <style>
    body, html {
      margin: 0;
      padding: 0;
      height: 100%;
      background: #fff;
      display: flex;
      flex-direction: column;
      font-family: Arial, sans-serif;
    }

    /* Header */
    header {
      background-color: #6b6b6b;
      color: white;
      display: flex;
      align-items: center;
      padding: 10px 20px;
    }
    header img {
      height: 32px;
      margin-right: 10px;
      border-radius: 50%;
    }
    header h1 {
      font-size: 18px;
      margin: 0;
    }

    /* Chat container takes full space below header */
    #chat-container {
      flex: 1;
      height: 100%;
    }

    footer {
      text-align: center;
      padding: 10px;
      font-size: 14px;
      color: #555;
      background-color: #f1f1f1;
    }

    /* Force Voiceflow chat to fill space */
    iframe,
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container,
    .vf-chat__content {
      width: 100% !important;
      height: 100% !important;
      border-radius: 0 !important;
    }
  </style>
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>

  <header>
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Voiceflow_Icon.svg/1024px-Voiceflow_Icon.svg.png" alt="Logo">
    <h1>Intellio</h1>
  </header>

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
