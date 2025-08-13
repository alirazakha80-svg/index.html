<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
  }

  body {
    display: flex;
    flex-direction: column;
    background-color: #f9f9f9;
  }

  #chat-container {
    flex: 1;
    width: 100%;
    height: 100vh; /* Always fill mobile viewport height */
  }

  /* Force Voiceflow widget iframe to fill the container */
  #chat-container iframe {
    width: 100% !important;
    height: 100% !important;
    border: none;
  }
</style>

</html>
