/**
 * Single-file Chat App (Auth + Free/Pro + Stripe + Chat UI)
 * ---------------------------------------------------------
 * How to run:
 *   1) npm init -y
 *   2) npm i express express-session bcryptjs jsonwebtoken better-sqlite3 stripe axios
 *   3) Set env vars (see bottom of file comments)
 *   4) node server.js
 *
 * Deploy: Render/Railway/Fly. (GitHub Pages won't work for backends.)
 */

const express = require("express");
const session = require("express-session");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const Database = require("better-sqlite3");
const Stripe = require("stripe");
const axios = require("axios");
const path = require("path");

// ---------- CONFIG ----------
const {
  PORT = 3000,
  SESSION_SECRET = "dev-secret-change-me",
  JWT_SECRET = "dev-jwt-change-me",
  STRIPE_SECRET_KEY = "",
  STRIPE_PRICE_ID = "", // your recurring price (e.g. price_123)
  BASE_URL = "http://localhost:3000",
  // Choose one provider below (OpenAI, OpenRouter, DeepSeek)
  OPENAI_API_KEY = "",
  OPENAI_MODEL = "gpt-4o-mini",
  OPENROUTER_API_KEY = "",
  OPENROUTER_MODEL = "openai/gpt-4o-mini",
  DEEPSEEK_API_KEY = "",
  DEEPSEEK_MODEL = "deepseek-chat",
  FREE_DAILY_MESSAGES = "30",
} = process.env;

const stripe = STRIPE_SECRET_KEY ? new Stripe(STRIPE_SECRET_KEY) : null;
const FREE_LIMIT = parseInt(FREE_DAILY_MESSAGES, 10) || 30;

// ---------- DB (SQLite, single file) ----------
const db = new Database(path.join(__dirname, "app.db"));
db.pragma("journal_mode = WAL");
db.exec(`
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  passhash TEXT NOT NULL,
  plan TEXT NOT NULL DEFAULT 'free',
  created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE IF NOT EXISTS messages (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  role TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE IF NOT EXISTS quotas (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  day TEXT NOT NULL,
  count INTEGER NOT NULL DEFAULT 0,
  UNIQUE(user_id, day)
);
`);

// ---------- Helpers ----------
const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(
  session({
    secret: SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
  })
);

function isAuthed(req) {
  return !!req.session.userId;
}
function requireAuth(req, res, next) {
  if (!isAuthed(req)) return res.redirect("/login");
  next();
}
function todayKey() {
  const d = new Date();
  return d.toISOString().slice(0, 10);
}
function upsertQuota(userId) {
  const day = todayKey();
  db.prepare(
    "INSERT OR IGNORE INTO quotas (user_id, day, count) VALUES (?, ?, 0)"
  ).run(userId, day);
  return db
    .prepare("SELECT * FROM quotas WHERE user_id = ? AND day = ?")
    .get(userId, day);
}
function incrementQuota(userId) {
  const day = todayKey();
  db.prepare("UPDATE quotas SET count = count + 1 WHERE user_id = ? AND day = ?")
    .run(userId, day);
}
function userById(id) {
  return db.prepare("SELECT * FROM users WHERE id = ?").get(id);
}
function userByEmail(email) {
  return db.prepare("SELECT * FROM users WHERE email = ?").get(email);
}
function setUserPlan(userId, plan) {
  db.prepare("UPDATE users SET plan = ? WHERE id = ?").run(plan, userId);
}

// ---------- Minimal UI (Tailwind CDN) ----------
const layout = (title, body, opts = {}) => `<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>${title}</title>
<script src="https://cdn.tailwindcss.com"></script>
<link rel="icon" href="data:,">
<style>
  html,body{height:100%}
  /* Mobile safe vh */
  .vh { height: 100vh; height: 100dvh; }
</style>
</head>
<body class="bg-gray-50 text-gray-900">
  <div class="min-h-screen flex flex-col">
    <header class="border-b bg-white">
      <div class="max-w-5xl mx-auto px-4 py-3 flex items-center justify-between">
        <div class="flex items-center gap-2">
          <div class="w-7 h-7 rounded-full bg-black/90"></div>
          <a href="/" class="font-semibold">Intellio</a>
        </div>
        <nav class="text-sm">
          ${
            opts.user
              ? `<span class="mr-3">Plan: <span class="font-medium">${opts.user.plan}</span></span>
                 <a class="mr-3 underline" href="/account">Account</a>
                 <a class="underline" href="/logout">Logout</a>`
              : `<a class="mr-3 underline" href="/login">Login</a>
                 <a class="underline" href="/signup">Sign up</a>`
          }
        </nav>
      </div>
    </header>
    <main class="flex-1">${body}</main>
    <footer class="border-t bg-white text-center text-xs py-3">© ${new Date().getFullYear()} Intellio</footer>
  </div>
</body></html>`;

// ---------- Routes ----------
app.get("/", (req, res) => {
  const user = req.session.userId ? userById(req.session.userId) : null;
  const body = `
  <section class="max-w-5xl mx-auto px-4 py-10">
    <div class="grid md:grid-cols-2 gap-8 items-center">
      <div>
        <h1 class="text-3xl md:text-5xl font-bold mb-4">Your AI study partner</h1>
        <p class="text-gray-600 mb-6">Free plan: ${FREE_LIMIT}/day messages. Pro plan: unlimited + faster responses.</p>
        <div class="flex gap-3">
          ${
            user
              ? `<a href="/app" class="px-4 py-2 rounded-xl bg-black text-white">Open Chat</a>`
              : `<a href="/signup" class="px-4 py-2 rounded-xl bg-black text-white">Get Started</a>
                 <a href="/login" class="px-4 py-2 rounded-xl border">Sign in</a>`
          }
        </div>
      </div>
      <div class="rounded-2xl border bg-white p-4">
        <div class="h-72 overflow-hidden rounded-xl bg-gradient-to-br from-gray-100 to-gray-200 flex items-center justify-center">
          <span class="text-gray-500">Chat preview</span>
        </div>
      </div>
    </div>
  </section>`;
  res.send(layout("Intellio", body, { user }));
});

app.get("/signup", (req, res) => {
  if (isAuthed(req)) return res.redirect("/app");
  const body = `
  <div class="max-w-md mx-auto px-4 py-10">
    <h2 class="text-2xl font-semibold mb-6">Create account</h2>
    <form method="post" action="/signup" class="space-y-4">
      <input name="email" type="email" required placeholder="Email" class="w-full border rounded-xl px-4 py-2">
      <input name="password" type="password" required placeholder="Password" class="w-full border rounded-xl px-4 py-2">
      <button class="w-full py-2 rounded-xl bg-black text-white">Sign up</button>
    </form>
    <p class="text-sm mt-3">Already have an account? <a class="underline" href="/login">Login</a></p>
  </div>`;
  res.send(layout("Sign up", body));
});

app.post("/signup", (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) return res.status(400).send("Missing fields");
  if (userByEmail(email)) return res.status(400).send("Email already exists");
  const passhash = bcrypt.hashSync(password, 10);
  const info = db
    .prepare("INSERT INTO users (email, passhash) VALUES (?, ?)")
    .run(email.toLowerCase(), passhash);
  req.session.userId = info.lastInsertRowid;
  res.redirect("/app");
});

app.get("/login", (req, res) => {
  if (isAuthed(req)) return res.redirect("/app");
  const body = `
  <div class="max-w-md mx-auto px-4 py-10">
    <h2 class="text-2xl font-semibold mb-6">Welcome back</h2>
    <form method="post" action="/login" class="space-y-4">
      <input name="email" type="email" required placeholder="Email" class="w-full border rounded-xl px-4 py-2">
      <input name="password" type="password" required placeholder="Password" class="w-full border rounded-xl px-4 py-2">
      <button class="w-full py-2 rounded-xl bg-black text-white">Login</button>
    </form>
    <p class="text-sm mt-3">No account? <a class="underline" href="/signup">Sign up</a></p>
  </div>`;
  res.send(layout("Login", body));
});

app.post("/login", (req, res) => {
  const { email, password } = req.body;
  const user = userByEmail(email?.toLowerCase() || "");
  if (!user) return res.status(400).send("Invalid credentials");
  if (!bcrypt.compareSync(password, user.passhash))
    return res.status(400).send("Invalid credentials");
  req.session.userId = user.id;
  res.redirect("/app");
});

app.get("/logout", (req, res) => {
  req.session.destroy(() => res.redirect("/"));
});

app.get("/account", requireAuth, (req, res) => {
  const user = userById(req.session.userId);
  const quota = upsertQuota(user.id);
  const body = `
  <div class="max-w-2xl mx-auto px-4 py-10 space-y-6">
    <h2 class="text-2xl font-semibold">Account</h2>
    <div class="border rounded-2xl bg-white p-4">
      <div class="flex justify-between"><span>Email</span><span class="font-mono">${user.email}</span></div>
      <div class="flex justify-between mt-2"><span>Plan</span><span class="font-semibold capitalize">${user.plan}</span></div>
      <div class="flex justify-between mt-2"><span>Today usage</span><span>${quota.count}/${user.plan==='pro' ? '∞' : FREE_LIMIT}</span></div>
    </div>
    ${
      user.plan === "pro" || !stripe
        ? ""
        : `<form action="/create-checkout-session" method="POST">
             <button class="px-4 py-2 rounded-xl bg-black text-white">Upgrade to Pro</button>
           </form>`
    }
    <a href="/app" class="inline-block underline">Back to chat</a>
  </div>`;
  res.send(layout("Account", body, { user }));
});

// ---------- Chat UI (mobile/desktop, full screen) ----------
app.get("/app", requireAuth, (req, res) => {
  const user = userById(req.session.userId);
  const body = `
  <div class="h-[100dvh] flex flex-col">
    <div class="px-4 py-3 border-b bg-white flex items-center justify-between">
      <div class="flex items-center gap-2"><div class="w-6 h-6 rounded-full bg-black/90"></div><span class="font-semibold">Intellio</span></div>
      <div class="text-xs">Plan: <span class="font-medium">${user.plan}</span></div>
    </div>

    <div id="chat" class="flex-1 overflow-y-auto bg-gray-50">
      <div id="messages" class="max-w-3xl mx-auto px-4 py-6 space-y-3"></div>
    </div>

    <form id="composer" class="bg-white border-t p-3">
      <div class="max-w-3xl mx-auto flex gap-2">
        <input id="input" class="flex-1 border rounded-xl px-4 py-3" placeholder="Message Intellio…" autocomplete="off" />
        <button class="px-4 py-3 rounded-xl bg-black text-white">Send</button>
      </div>
      <p id="limit" class="max-w-3xl mx-auto px-1 mt-2 text-xs text-gray-500 text-right"></p>
    </form>
  </div>

  <script>
    const elMsgs = document.getElementById('messages');
    const elForm = document.getElementById('composer');
    const elInput = document.getElementById('input');
    const elLimit = document.getElementById('limit');

    async function fetchStatus() {
      const r = await fetch('/status');
      const s = await r.json();
      elLimit.textContent = s.plan === 'pro' ? 'Pro: unlimited' : 'Free: ' + s.count + ' / ${FREE_LIMIT} today';
    }
    function bubble(role, text) {
      const li = document.createElement('div');
      li.className = 'max-w-[85%] ' + (role==='user' ? 'ml-auto bg-black text-white' : 'bg-white') + ' p-3 rounded-2xl shadow-sm';
      li.textContent = text;
      elMsgs.appendChild(li);
      elMsgs.parentElement.scrollTop = elMsgs.parentElement.scrollHeight;
    }
    elForm.addEventListener('submit', async (e) => {
      e.preventDefault();
      const content = elInput.value.trim();
      if (!content) return;
      bubble('user', content);
      elInput.value='';
      const r = await fetch('/api/chat', { method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({content}) });
      if (r.status === 429) { const t = await r.text(); bubble('assistant', t); return; }
      const data = await r.json();
      bubble('assistant', data.reply || 'No reply');
      fetchStatus();
    });
    fetchStatus();
  </script>`;
  res.send(layout("Chat", body, { user }));
});

// small status endpoint for UI
app.get("/status", requireAuth, (req, res) => {
  const user = userById(req.session.userId);
  const q = upsertQuota(user.id);
  res.json({ plan: user.plan, count: q.count, limit: FREE_LIMIT });
});

// ---------- Chat API (rate limits + provider selection) ----------
app.post("/api/chat", requireAuth, async (req, res) => {
  const user = userById(req.session.userId);
  const { content } = req.body || {};
  if (!content) return res.status(400).json({ error: "Missing content" });

  // Free-tier check
  if (user.plan !== "pro") {
    const q = upsertQuota(user.id);
    if (q.count >= FREE_LIMIT) {
      return res
        .status(429)
        .send(`Daily free limit reached (${FREE_LIMIT}). Visit Account to upgrade.`);
    }
    incrementQuota(user.id);
  }

  // Store user message
  db.prepare("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)")
    .run(user.id, "user", content);

  try {
    const reply = await callAnyProvider(content);
    db.prepare("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)")
      .run(user.id, "assistant", reply);
    res.json({ reply });
  } catch (e) {
    console.error(e);
    res.status(500).json({ error: "AI provider error" });
  }
});

// Choose OpenAI / OpenRouter / DeepSeek automatically based on keys present
async function callAnyProvider(prompt) {
  if (OPENAI_API_KEY) {
    const r = await axios.post(
      "https://api.openai.com/v1/chat/completions",
      { model: OPENAI_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${OPENAI_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  if (OPENROUTER_API_KEY) {
    const r = await axios.post(
      "https://openrouter.ai/api/v1/chat/completions",
      { model: OPENROUTER_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${OPENROUTER_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  if (DEEPSEEK_API_KEY) {
    const r = await axios.post(
      "https://api.deepseek.com/chat/completions",
      { model: DEEPSEEK_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${DEEPSEEK_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  // Fallback echo
  return "Configure an AI key in environment variables. Echo: " + prompt;
}

// ---------- Stripe checkout (optional, but ready) ----------
app.post("/create-checkout-session", requireAuth, async (req, res) => {
  try {
    if (!stripe || !STRIPE_PRICE_ID) throw new Error("Stripe not configured");
    const user = userById(req.session.userId);

    const session = await stripe.checkout.sessions.create({
      mode: "subscription",
      line_items: [{ price: STRIPE_PRICE_ID, quantity: 1 }],
      customer_email: user.email,
      success_url: `${BASE_URL}/stripe/success?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${BASE_URL}/account`,
    });
    res.redirect(303, session.url);
  } catch (e) {
    console.error(e);
    res.status(500).send("Stripe is not configured properly.");
  }
});

// Verify payment on return and upgrade user
app.get("/stripe/success", requireAuth, async (req, res) => {
  try {
    if (!stripe) throw new Error("Stripe not configured");
    const { session_id } = req.query;
    const s = await stripe.checkout.sessions.retrieve(session_id);
    if (s.payment_status === "paid") {
      setUserPlan(req.session.userId, "pro");
      return res.redirect("/account");
    }
    res.redirect("/account");
  } catch (e) {
    console.error(e);
    res.redirect("/account");
  }
});

// ---------- Start ----------
app.listen(PORT, () => {
  console.log(`✅ Running on http://localhost:${PORT}`);
});

/**
 * ========= ENV VARIABLES (create .env for local or set on host) =========
 * PORT=3000
 * SESSION_SECRET=change-this
 * JWT_SECRET=change-this-too
 * BASE_URL=https://your-app.onrender.com
 *
 * (Choose one provider below)
 * OPENAI_API_KEY=sk-...
 * OPENAI_MODEL=gpt-4o-mini
 * --OR--
 * OPENROUTER_API_KEY=or-...
 * OPENROUTER_MODEL=openai/gpt-4o-mini
 * --OR--
 * DEEPSEEK_API_KEY=sk-...
 * DEEPSEEK_MODEL=deepseek-chat
 *
 * (Stripe optional but supported)
 * STRIPE_SECRET_KEY=sk_test_...
 * STRIPE_PRICE_ID=price_123
 * FREE_DAILY_MESSAGES=30
 *
 * =========================================================================
 */
