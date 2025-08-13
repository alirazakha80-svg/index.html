<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Intellio – AI Chatbot</title>
  <meta name="color-scheme" content="light dark" />
  <style>
    /* Page = pure full screen canvas for the chat */
    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; }
    body { background: #ffffff; font-family: Arial, Helvetica, sans-serif; overflow: hidden; }
    #chat { position: fixed; inset: 0; }  /* full screen */

    /* Make the Voiceflow embed truly full screen */
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container,
    .vf-chat__content,
    .vf-chat__inner,
    .vf-chat__body,
    .vf-chat__messages,
    .vf-chat__footer {
      width: 100% !important;
      height: 100% !important;
      max-width: none !important;
      border-radius: 0 !important;
      box-shadow: none !important;
      margin: 0 !important;
    }

    /* Hide the Voiceflow gray header/top bar */
    .vf-chat__header,
    .vf-chat-header,
    .vf-header,
    .vf-app__header {
      display: none !important;
      height: 0 !important;
      padding: 0 !important;
      margin: 0 !important;
      border: 0 !important;
    }

    /* Let the message area scroll inside while page stays fixed */
    .vf-chat__messages { height: calc(100% - 84px) !important; } /* footer/input space safety */
  </style>

  <!-- Preload for snappier load -->
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>
  <div id="chat"></div>

  <script>
    // Load Voiceflow and embed full-screen
    (function () {
      function boot() {
        if (!window.voiceflow?.chat?.load) return;
        window.voiceflow.chat.load({
          verify: { projectID: "689c3b1e9d300c90a54798bf" },
          url: "https://general-runtime.voiceflow.com",
          versionID: "production",
          render: { mode: "embedded", target: document.getElementById("chat") },
          autostart: true
        });
      }

      var s = document.createElement("script");
      s.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
      s.async = true;
      s.type = "text/javascript";
      s.onload = boot;
      document.head.appendChild(s);
    })();
  </script>
</body>
</html>
