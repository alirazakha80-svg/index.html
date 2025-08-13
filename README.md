<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Intellio – AI Chatbot</title>
  <meta name="color-scheme" content="light dark" />
  <style>
    /* Full-screen canvas for the chat */
    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; }
    /* use svh to behave better on mobile address bars */
    body { height: 100svh; background: #ffffff; font-family: Arial, Helvetica, sans-serif; overflow: hidden; }
    #chat { position: fixed; inset: 0; }

    /* Stretch the Voiceflow embed to 100% and remove popup styling */
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

    /* Keep the VF header visible (for mic/restart icons) */
    /* Do NOT hide .vf-chat__header */

    /* Make sure messages area can scroll while footer stays at bottom */
    .vf-chat__messages { overscroll-behavior: contain; }
  </style>

  <!-- Preload for faster load -->
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
</head>
<body>
  <div id="chat" aria-label="Intellio chat" role="application"></div>

  <script>
    (function () {
      function boot() {
        if (!window.voiceflow?.chat?.load) return;
        window.voiceflow.chat.load({
          verify:   { projectID: "689c3b1e9d300c90a54798bf" },
          url:      "https://general-runtime.voiceflow.com",
          versionID:"production",
          render:   { mode: "embedded", target: document.getElementById("chat") },
          autostart:true,
          // Keep voice enabled so the MIC appears on desktop
          voice: { url: "https://runtime-api.voiceflow.com" }
        });
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
