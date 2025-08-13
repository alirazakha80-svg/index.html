<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ChatGPT-Style Fullscreen Chat</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      height: 100vh;
      display: flex;
      background-color: #202123; /* ChatGPT dark background */
      color: white;
      font-family: Arial, sans-serif;
    }

    /* Sidebar */
    .sidebar {
      width: 260px;
      background-color: #171717;
      display: flex;
      flex-direction: column;
      padding: 20px;
      border-right: 1px solid #2a2a2a;
    }

    .sidebar h2 {
      margin-bottom: 20px;
      font-size: 18px;
    }

    /* Main Chat Area */
    .chat-container {
      flex: 1;
      display: flex;
      flex-direction: column;
      height: 100vh;
    }

    .messages {
      flex: 1;
      padding: 20px;
      overflow-y: auto;
    }

    .message {
      max-width: 80%;
      margin-bottom: 12px;
      padding: 12px 16px;
      border-radius: 8px;
    }

    .bot {
      background-color: #444654;
      align-self: flex-start;
    }

    .user {
      background-color: #10a37f;
      align-self: flex-end;
    }

    /* Input area */
    .input-area {
      padding: 16px;
      background-color: #343541;
      display: flex;
      gap: 10px;
    }

    .input-area input {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 6px;
      outline: none;
      font-size: 16px;
    }

    .input-area button {
      padding: 12px 20px;
      background-color: #10a37f;
      border: none;
      border-radius: 6px;
      color: white;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <!-- Sidebar -->
  <div class="sidebar">
    <h2>ChatGPT</h2>
    <p>New chat</p>
    <p>Search chats</p>
    <p>Library</p>
  </div>

  <!-- Chat Area -->
  <div class="chat-container">
    <div class="messages" id="messages">
      <div class="message bot">Hi! How can I help you today?</div>
      <div class="message user">Hello! I want to build a chat UI.</div>
    </div>
    <div class="input-area">
      <input type="text" placeholder="Type a message..." id="userInput">
      <button onclick="
