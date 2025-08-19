<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Intellio – Student Chatbot</title>
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  <meta name="theme-color" content="#0b0b0c" />
  <style>
    /* ---- Reset & base ---- */
    *, *::before, *::after { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; padding: 0; }
    body {
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      color: #0b0b0c; background: #f8fafc;
    }

    /* ---- Layout ---- */
    .app {
      min-height: 100dvh; /* mobile-safe full height */
      display: flex; flex-direction: column;
    }
    .topbar {
      height: 56px; display: flex; align-items: center; justify-content: space-between;
      padding: 0 16px; background: #ffffff; border-bottom: 1px solid #e5e7eb;
      position: sticky; top: 0; z-index: 10;
    }
    .brand { display: flex; align-items: center; gap: 10px; font-weight: 600; }
    .dot { width: 20px; height: 20px; border-radius: 999px; background: #0b0b0c; }

    .chat-wrap {
      flex: 1; min-height: 0; /* lets child overflow-y work */
      background: #f1f5f9;
      display: flex;
    }
    #chat-container {
      flex: 1; /* fill all space */
      width: 100%;
      height: 100%;
      display: flex;
      min-height: 0;
    }

    /* Make sure Voiceflow iframe truly fills */
    #chat-container iframe {
      width: 100% !important;
      height: 100% !important;
      border: 0 !important;
      display: block;
    }

    .footer {
      background: #ffffff; border-top: 1px solid #e5e7eb;
      font-size: 12px; color: #64748b; text-align: center; padding: 10px 12px;
    }

    /* Small screens polish */
    @media (max-width: 480px) {
      .topbar { height: 52px; }
      .brand span { font-size: 14px; }
    }
  </style>
</head>
<body>
  <div class="app">
    <!-- Simple top bar like modern chat apps -->
    <header class="topbar">
      <div class="brand">
        <div class="dot"></div>
        <span>Intellio – Student Chatbot</span>
      </div>
      <nav style="font-size:12px; color:#64748b">
        <!-- add links if you want -->
      </nav>
    </header>

    <!-- Full-screen chat area -->
    <main class="chat-wrap">
      <div id="chat-container" aria-label="Chatbot area"></div>
    </main>

    <footer class="footer">
      © <span id="year"></span> Intellio • Powered by Voiceflow
    </footer>
  </div>

  <!-- Voiceflow embed -->
  <script type="module">
    // Footer year
    document.getElementById('year').textContent = new Date().getFullYear();

    // Load Voiceflow widget
    const script = document.createElement("script");
    script.src = "https://cdn.voiceflow.com/widget-next/bundle.mjs";
    script.type = "module";
    script.async = true;
    script.onload = () => {
      window.voiceflow.chat.load({
        verify: { projectID: "689c3b1e9d300c90a54798bf" }, // your project
        url: "https://general-runtime.voiceflow.com",
        versionID: "production",
        render: { mode: "embedded", target: document.getElementById("chat-container") },
        autostart: true
      });
    };
    document.body.appendChild(script);
  </script>
</body>
</html>
