<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Intellio-Style Chat</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      background-color: #f5f5f5;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      padding: 20px;
    }

    .chat-wrapper {
      background-color: white;
      border-radius: 12px;
      width: 100%;
      max-width: 900px;
      height: 90vh;
      display: flex;
      flex-direction: column;
      box-shadow: 0 4px 20px rgba(0,0,0,0.1);
      overflow: hidden;
    }

    /* Header */
    .chat-header {
      background-color: #6e6e6e;
      color: white;
      padding: 15px;
      font-weight: bold;
      font-size: 18px;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    /* Messages area */
    .messages {
      flex: 1;
      padding: 20px;
      overflow-y: auto;
    }

    .message {
      max-width: 70%;
      padding: 12px 16px;
      border-radius: 20px;
      margin-bottom: 12px;
      font-size: 15px;
      line-height: 1.4;
    }

    .bot {
      background-color: #f0f0f0;
      align-self: flex-start;
    }

    .user {
      background-color: #6e6e6e;
      color: white;
      align-self: flex-end;
    }

    /* Input area */
    .input-area {
      padding: 12px;
      display: flex;
      gap: 10px;
      border-top: 1px solid #ddd;
    }

    .input-area input {
      flex: 1;
      padding: 12px;
      border: 1px solid #ccc;
      border-radius: 20px;
      outline: none;
      font-size: 14px;
    }

    .input-ar
