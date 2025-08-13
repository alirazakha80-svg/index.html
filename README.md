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
      background: #fff;
      font-family: Arial, sans-serif;
      overflow: hidden;
    }

    #chat-container {
      height: 100vh;
      width: 100vw;
      margin: 0;
      padding: 0;
    }

    /* Make Voiceflow widget take entire viewport */
    iframe,
    .vf-chat,
    .vf-chat--embedded,
    .vf-chat__container,
    .vf-chat__content {
      width: 100% !important;
      height: 100% !important;
      border-radius: 0 !important;
      margin: 0 !important;
      padding: 0 !important;
    }

    /* Remove*
