/* Chat container full height */
#chat-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
  color: white;
}

/* Messages area */
#messages {
  flex: 1;
  overflow-y: auto;
  padding-bottom: 20px;
}

.message {
  padding: 10px 15px;
  margin: 8px 0;
  border-radius: 8px;
  max-width: 80%;
  word-wrap: break-word;
}

.bot {
  background-color: #343541;
  align-self: flex-start;
}

.user {
  background-color: #10a37f;
  align-self: flex-end;
}

/* Input area fixed at bottom */
#input-area {
  display: flex;
  gap: 10px;
  padding: 10px;
  background-color: #343541;
  border-radius: 8px;
}

#user-input {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 5px;
  outline: none;
  font-size: 16px;
}

button {
  background-color: #10a37f;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0e8f6e;
}
