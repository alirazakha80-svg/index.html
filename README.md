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
    #chat { position: fixed; inset: 0; }

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

    .vf-chat__messages { overscroll-behavior: contain; }

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

    /* Loader screen */
    #loader {
      position: fixed;
      top: 0; left: 0;
      width: 100%;
      height: 100%;
      background: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      color: #444;
      z-index: 10000;
    }
  </style>
  <link rel="preload" href="https://cdn.voiceflow.com/widget-next/bundle.mjs" as="script">
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
          verify:   { projectID: "689c3b1e9d300c90a54798bf" },
          url:      "https://general-runtime.voiceflow.com",
          versionID:"production",
          render:   { mode: "embedded", target: document.getElementById("chat") },
          autostart:true,
          voice: { url: "https://runtime-api.voiceflow
