<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>StudyMate</title>
<style>
  html,body{margin:0;height:100%;background:#000;color:#fff;font-family:Arial, sans-serif}
  header{display:flex;justify-content:space-between;align-items:center;padding:10px 14px;border-bottom:1px solid #222;background:#000}
  .brand{font-weight:700}
  .auth{display:flex;gap:8px}
  .auth input{background:#111;border:1px solid #333;color:#fff;padding:8px;border-radius:6px}
  .auth button,.pill{background:#111;border:1px solid #333;color:#fff;padding:8px 12px;border-radius:6px;cursor:pointer}
  #notice{display:none;gap:10px;align-items:center;justify-content:center;background:#0a0a0a;border-bottom:1px solid #222;padding:10px}
  #chat{height:calc(100% - 102px)}
  footer{height:40px;display:flex;align-items:center;justify-content:center;border-top:1px solid #222;font-size:14px}
  .hide{display:none}
</style>
</head>
<body>
  <header>
    <div class="brand">StudyMate</div>
    <div class="auth" id="authBox">
      <input id="email" type="email" placeholder="email"/>
      <input id="pass" type="password" placeholder="password"/>
      <button id="login">Log in</button>
      <button id="signup">Sign up</button>
      <button id="logout" class="hide">Log out</button>
    </div>
  </header>

  <div id="notice">
    <span id="planText">Free plan</span>
    <button id="upgrade" class="pill">Upgrade to Pro</button>
  </div>

  <div id="chat"></div>
  <footer>Powered by Voiceflow • Powered by Ali Raza</footer>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>

  <script>
  // TODO: replace with your Firebase config (Firebase Console → Project settings)
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();

  const chatDiv   = document.getElementById('chat');
  const notice    = document.getElementById('notice');
  const planText  = document.getElementById('planText');
  const loginBtn  = document.getElementById('login');
  const signupBtn = document.getElementById('signup');
  const logoutBtn = document.getElementById('logout');
  const emailEl   = document.getElementById('email');
  const passEl    = document.getElementById('pass');
  const upgradeBtn= document.getElementById('upgrade');

  // Auth actions
  loginBtn.onclick = async ()=> {
    await auth.signInWithEmailAndPassword(emailEl.value, passEl.value);
  };
  signupBtn.onclick = async ()=> {
    await auth.createUserWithEmailAndPassword(emailEl.value, passEl.value);
  };
  logoutBtn.onclick = async ()=> auth.signOut();

  // When user logs in/out
  auth.onAuthStateChanged(async (user)=>{
    if (!user) {
      logoutBtn.classList.add('hide');
      loginBtn.classList.remove('hide'); signupBtn.classList.remove('hide');
      emailEl.classList.remove('hide');  passEl.classList.remove('hide');
      planText.textContent = "Please log in to chat.";
      notice.style.display = 'flex';
      chatDiv.innerHTML = ''; // remove bot
      return;
    }
    // UI
    logoutBtn.classList.remove('hide');
    loginBtn.classList.add('hide'); signupBtn.classList.add('hide');
    emailEl.classList.add('hide');  passEl.classList.add('hide');

    // Read plan from Firestore
    const doc = await db.collection('users').doc(user.uid).get();
    const plan = doc.exists ? (doc.data().plan || 'free') : 'free';
    planText.textContent = plan === 'pro' ? "Pro plan" : "Free plan";
    notice.style.display = 'flex';
    upgradeBtn.style.display = plan === 'pro' ? 'none' : 'inline-block';

    // Mount Voiceflow widget (embedded)
    mountVoiceflow();
  });

  async function mountVoiceflow(){
    if (document.getElementById('vf-loader')) return; // already loaded
    const s = document.createElement('script');
    s.id = 'vf-loader';
    s.type = 'text/javascript';
    s.src = 'https://cdn.voiceflow.com/widget-next/bundle.mjs';
    s.onload = function(){
      window.voiceflow.chat.load({
        verify: { projectID: '689c3b1e9d300c90a54798bf' },
        url: 'https://general-runtime.voiceflow.com',
        versionID: 'production',
        render: { mode: 'embedded', target: document.getElementById('chat') },
        theme: { primary:'#000000', secondary:'#1a1a1a', text:'#FFFFFF', font:'Arial, sans-serif' },
        assistant: { title: 'StudyMate', description: 'How can I help you today?' }
      });
    };
    document.body.appendChild(s);
  }

  // Upgrade to Pro (Stripe Checkout via your backend)
  upgradeBtn.onclick = async ()=>{
    const user = auth.currentUser;
    if (!user) return alert('Log in first');
    const idToken = await user.getIdToken();
    const res = await fetch('https://YOUR_BACKEND_URL/checkout', {
      method:'POST',
      headers:{'Content-Type':'application/json','Authorization':'Bearer '+idToken},
      body: JSON.stringify({ priceId: 'YOUR_STRIPE_PRICE_ID' })
    });
    const data = await res.json();
    if (data.url) location.href = data.url;
    else alert('Checkout error');
  };
  </script>
</body>
</html>

