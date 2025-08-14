<!DOCTYPE html>
<html lang="en">
<head>
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
    }

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
</head>
<body>
  <div id="loader">Loading chatbot...</div>
  <div id="chat" aria-label="Intellio chat" role="application"></div>
  <div id="footer">Powered by Ali Raza</div>

  <script>
    (function () {
      function boot() {
        if (!window.voiceflow?.chat?.load) return;
        window.voiceflow.chat.load({
          verify: { projectID: "689c3b1e9d300c90a54798bf" },
          url: "https://general-runtime.voiceflow.com",
          versionID: "production",
          render: { mode: "embedded", target: document.getElementById("chat") },
          autostart: true,
          voice: { url: "https://runtime-api.voiceflow.com" }
        });
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
<style>
/* Remove the sender name/header from messages */
.vf-chat__message-sender {
  display: none !important;
}

/* Optional: tighten spacing after removing name */
.vf-chat__message {
  margin-top: 4px !important;
}
</style>
</body>
</html>
