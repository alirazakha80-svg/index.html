<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1, viewport-fit=cover"
  />
  <meta name="color-scheme" content="light dark" />
  <title>Intellio – AI Chatbot</title>
  <style>
    /* --- Layout reset --- */
    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; }
    body {
      height: 100dvh; /* good on mobile address-bar */
      background: #ffffff;
      font-family: Arial, Helvetica, sans-serif;
      overflow: hidden;
    }

    /* Chat host fills the screen */
    #chat { position: fixed; inset: 0; }

    /* Force Voiceflow widget full-bleed */
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

    /* Safe area for iOS bottom bar */
    .vf-chat__footer,
    .vf-messages__footer {
      padding-bottom: max(12px, env(safe-area-inset-bottom));
    }

    /* Loader (shown until widget is actually ready) */
    #loader {
      position: fixed;
      inset: 0;
      display: grid;
      place-items: center;
      background: #ffffff;
      z-index: 10000;
      font-size: 18px;
      color: #444;
      text-align: center;
      padding: 24px;
    }
    #loader small { display:block; opacity:.7; margin-top:6px; }

    /* Bottom credit centered */
    #credit {
      position: fixed;
      left: 50%;
      transform: translateX(-50%);
      bottom: 8px;
      z-index: 9999;
      font-size: 12px;
      color: #666;
      backgrou
