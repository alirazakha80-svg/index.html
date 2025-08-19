/**
 * Intellio — single-file Node app
 * Login/Signup + Free & Pro plans + Lemon Squeezy checkout (bank payouts)
 * Chat backend supports OpenAI/OpenRouter/DeepSeek. SQLite for users & quotas.
 * -------------------------------------------------------------------------
 * How to run locally:
 *   1) npm init -y
 *   2) npm i express express-session bcryptjs better-sqlite3 axios
 *   3) Create a .env file (see bottom of this file for sample)
 *   4) node server.js
 *
 * Deploy:
 *   • Push to GitHub ➜ Deploy on Render/Railway/Fly (GitHub Pages cannot host Node backends).
 *   • On your host, add the same environment variables from your .env.
 *
 * Lemon Squeezy Payouts:
 *   • Lemon Squeezy supports bank payouts in many countries (including Pakistan).
 *   • Create a recurring product (your “Pro” plan). Add a webhook with a Signing Secret.
 *   • This app verifies the webhook and upgrades the user to Pro automatically.
 */

const express = require("express");
const session = require("express-session");
const bcrypt = require("bcryptjs");
const Database = require("better-sqlite3");
const axios = require("axios");
const crypto = require("crypto");
const path = require("path");

// ---------- ENV ----------
require("dotenv").config();
const {
  PORT = 3000,
  SESSION_SECRET = "change-me",
  BASE_URL = "http://localhost:3000",

  // AI providers (pick one by setting its key)
  OPENAI_API_KEY = "",
  OPENAI_MODEL = "gpt-4o-mini",
  OPENROUTER_API_KEY = "",
  OPENROUTER_MODEL = "openai/gpt-4o-mini",
  DEEPSEEK_API_KEY = "",
  DEEPSEEK_MODEL = "deepseek-chat",

  // Free plan limit per day
  FREE_DAILY_MESSAGES = "30",

  // Lemon Squeezy (bank payouts)
  // Example store subdomain: if your checkout link starts with
  // https://mystore.lemonsqueezy.com/..., then LEMONSQUEEZY_STORE="mystore"
  LEMONSQUEEZY_STORE = "",
  // The Variant ID for your subscription product (visible in your buy link)
  LEMONSQUEEZY_VARIANT_ID = "", // e.g. d1f120fd-5adc-4fff-8915-cc...
  // Webhook Signing Secret that YOU set when creating your webhook in LS
  LEMONSQUEEZY_WEBHOOK_SECRET = "",
} = process.env;

const FREE_LIMIT = parseInt(FREE_DAILY_MESSAGES, 10) || 30;

// ---------- DB ----------
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

// Body parsers: JSON for normal routes
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Sessions
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
  db.prepare("INSERT OR IGNORE INTO quotas (user_id, day, count) VALUES (?, ?, 0)").run(userId, day);
  return db.prepare("SELECT * FROM quotas WHERE user_id = ? AND day = ?").get(userId, day);
}
function incrementQuota(userId) {
  const day = todayKey();
  db.prepare("UPDATE quotas SET count = count + 1 WHERE user_id = ? AND day = ?").run(userId, day);
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

// ---------- Minimal UI ----------
const layout = (title, body, opts = {}) => `<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>${title}</title>
<script src="https://cdn.tailwindcss.com"></script>
<link rel="icon" href="data:,">
<style>
  html,body{height:100%}
  .vh{height:100vh;height:100dvh}
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
  const info = db.prepare("INSERT INTO users (email, passhash) VALUES (?, ?)").run(email.toLowerCase(), passhash);
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
  if (!bcrypt.compareSync(password, user.passhash)) return res.status(400).send("Invalid credentials");
  req.session.userId = user.id;
  res.redirect("/app");
});

app.get("/logout", (req, res) => {
  req.session.destroy(() => res.redirect("/"));
});

app.get("/account", requireAuth, (req, res) => {
  const user = userById(req.session.userId);
  const quota = upsertQuota(user.id);
  const hasLS = !!(LEMONSQUEEZY_STORE && LEMONSQUEEZY_VARIANT_ID);
  const body = `
  <div class="max-w-2xl mx-auto px-4 py-10 space-y-6">
    <h2 class="text-2xl font-semibold">Account</h2>
    <div class="border rounded-2xl bg-white p-4">
      <div class="flex justify-between"><span>Email</span><span class="font-mono">${user.email}</span></div>
      <div class="flex justify-between mt-2"><span>Plan</span><span class="font-semibold capitalize">${user.plan}</span></div>
      <div class="flex justify-between mt-2"><span>Today usage</span><span>${quota.count}/${user.plan==='pro' ? '∞' : FREE_LIMIT}</span></div>
    </div>
    ${
      user.plan === "pro"
        ? `<div class="text-sm text-green-700">You're Pro. Thank you! 🎉</div>`
        : hasLS
          ? `<script src="https://app.lemonsqueezy.com/js/lemon.js" defer></script>
             <a 
               class="inline-block px-4 py-2 rounded-xl bg-black text-white"
               href="https://${LEMONSQUEEZY_STORE}.lemonsqueezy.com/checkout/buy/${LEMONSQUEEZY_VARIANT_ID}?checkout[custom][user_id]=${user.id}&checkout[email]=${encodeURIComponent(user.email)}">
               Upgrade to Pro
             </a>
             <p class="text-xs text-gray-500">After payment, you'll be upgraded automatically within a few seconds.</p>`
          : `<div class="text-sm text-red-600">Payments not configured. Set LEMONSQUEEZY_* env vars.</div>`
    }
    <a href="/app" class="inline-block underline">Back to chat</a>
  </div>`;
  res.send(layout("Account", body, { user }));
});

// ---------- Chat UI (mobile/desktop) ----------
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

// ---------- Chat API ----------
app.post("/api/chat", requireAuth, async (req, res) => {
  const user = userById(req.session.userId);
  const { content } = req.body || {};
  if (!content) return res.status(400).json({ error: "Missing content" });

  // Free-tier check
  if (user.plan !== "pro") {
    const q = upsertQuota(user.id);
    if (q.count >= FREE_LIMIT) {
      return res.status(429).send(`Daily free limit reached (${FREE_LIMIT}). Visit Account to upgrade.`);
    }
    incrementQuota(user.id);
  }

  // Store user message
  db.prepare("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)").run(user.id, "user", content);

  try {
    const reply = await callAnyProvider(content);
    db.prepare("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)").run(user.id, "assistant", reply);
    res.json({ reply });
  } catch (e) {
    console.error(e);
    res.status(500).json({ error: "AI provider error" });
  }
});

async function callAnyProvider(prompt) {
  // OpenAI
  if (OPENAI_API_KEY) {
    const r = await axios.post(
      "https://api.openai.com/v1/chat/completions",
      { model: OPENAI_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${OPENAI_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  // OpenRouter
  if (OPENROUTER_API_KEY) {
    const r = await axios.post(
      "https://openrouter.ai/api/v1/chat/completions",
      { model: OPENROUTER_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${OPENROUTER_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  // DeepSeek
  if (DEEPSEEK_API_KEY) {
    const r = await axios.post(
      "https://api.deepseek.com/chat/completions",
      { model: DEEPSEEK_MODEL, messages: [{ role: "user", content: prompt }] },
      { headers: { Authorization: `Bearer ${DEEPSEEK_API_KEY}` } }
    );
    return r.data.choices?.[0]?.message?.content?.trim() || "…";
  }
  // Fallback
  return "Configure an AI key in environment variables. Echo: " + prompt;
}

// ---------- Lemon Squeezy Webhook ----------
// Use a raw body parser just for this route so we can verify signatures.
app.post("/webhooks/lemonsqueezy", express.raw({ type: "application/json" }), (req, res) => {
  try {
    const signature = req.get("X-Signature") || "";
    const hmac = crypto.createHmac("sha256", LEMONSQUEEZY_WEBHOOK_SECRET);
    const digest = Buffer.from(hmac.update(req.body).digest("hex"), "utf8");
    const sigBuf = Buffer.from(signature, "utf8");

    if (!LEMONSQUEEZY_WEBHOOK_SECRET || digest.length !== sigBuf.length || !crypto.timingSafeEqual(digest, sigBuf)) {
      return res.status(400).send("Invalid signature");
    }

    const eventName = req.get("X-Event-Name") || ""; // e.g. subscription_created
    const payload = JSON.parse(req.body.toString("utf8"));

    // Lemon Squeezy puts our custom checkout data here
    const custom = payload?.meta?.custom_data || {};
    const userId = Number(custom.user_id);

    // Guard: we only care about our specific Variant ID
    const variantId = payload?.data?.attributes?.variant_id || payload?.data?.relationships?.variant?.data?.id;

    if (userId && String(variantId).includes(String(LEMONSQUEEZY_VARIANT_ID))) {
      if (eventName === "subscription_created" || eventName === "order_created" || eventName === "subscription_payment_success") {
        setUserPlan(userId, "pro");
        console.log(`Upgraded user ${userId} to pro via ${eventName}`);
      }

      if (eventName === "subscription_cancelled" || eventName === "subscription_expired") {
        setUserPlan(userId, "free");
        console.log(`Downgraded user ${userId} to free via ${eventName}`);
      }
    }

    res.sendStatus(200);
  } catch (e) {
    console.error("Webhook error:", e);
    res.sendStatus(200); // avoid retries during development
  }
});

// ---------- Start ----------
app.listen(PORT, () => {
  console.log(`✅ Running on ${BASE_URL.replace(/\/$/, "") || `http://localhost:${PORT}`}`);
});

/**
 * ==================== .env (example) ====================
 * PORT=3000
 * SESSION_SECRET=change-this
 * BASE_URL=http://localhost:3000
 *
 * # Choose one provider (set its API key)
 * OPENAI_API_KEY=
 * OPENAI_MODEL=gpt-4o-mini
 * # OR
 * OPENROUTER_API_KEY=
 * OPENROUTER_MODEL=openai/gpt-4o-mini
 * # OR
 * DEEPSEEK_API_KEY=
 * DEEPSEEK_MODEL=deepseek-chat
 *
 * # Free messages per day for Free plan
 * FREE_DAILY_MESSAGES=30
 *
 * # Lemon Squeezy (bank payouts)
 * LEMONSQUEEZY_STORE=your-store-subdomain
 * LEMONSQUEEZY_VARIANT_ID=your-variant-id
 * LEMONSQUEEZY_WEBHOOK_SECRET=make-a-random-string
 * ========================================================
 */
