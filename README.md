const express = require("express");
const session = require("express-session");
const bcrypt = require("bcryptjs");
const Database = require("better-sqlite3");

const app = express();
const db = new Database("ctdk.sqlite");

const PORT = process.env.PORT || 3000;
const SECRET = process.env.SESSION_SECRET;

if (!SECRET || !process.env.ADMIN_EMAIL || !process.env.ADMIN_PASSWORD) {
  console.error("Set SESSION_SECRET, ADMIN_EMAIL and ADMIN_PASSWORD first.");
  process.exit(1);
}

db.pragma("journal_mode = WAL");

db.exec(`
CREATE TABLE IF NOT EXISTS users (
 id INTEGER PRIMARY KEY AUTOINCREMENT,
 name TEXT NOT NULL,
 email TEXT NOT NULL UNIQUE,
 password TEXT NOT NULL,
 role TEXT NOT NULL DEFAULT 'user',
 status TEXT NOT NULL DEFAULT 'active',
 created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE IF NOT EXISTS requests (
 id INTEGER PRIMARY KEY AUTOINCREMENT,
 user_id INTEGER NOT NULL,
 type TEXT NOT NULL,
 amount REAL NOT NULL,
 status TEXT NOT NULL DEFAULT 'pending',
 created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
`);

async function createAdmin() {
 const email = process.env.ADMIN_EMAIL.toLowerCase().trim();
 const exists = db.prepare("SELECT id FROM users WHERE email=?").get(email);
 if (!exists) {
   const hash = await bcrypt.hash(process.env.ADMIN_PASSWORD, 12);
   db.prepare("INSERT INTO users(name,email,password,role) VALUES(?,?,?,?)")
     .run("Administrator", email, hash, "admin");
 }
}
 
app.use(express.json({ limit: "15kb" }));
app.use(session({
 secret: SECRET,
 resave: false,
 saveUninitialized: false,
 cookie: {
   httpOnly: true,
   sameSite: "lax",
   secure: process.env.NODE_ENV === "production",
   maxAge: 8 * 60 * 60 * 1000
 }
}));

function auth(req, res, next) {
 const user = req.session.userId &&
   db.prepare("SELECT id,name,email,role,status,created_at FROM users WHERE id=?")
     .get(req.session.userId);

 if (!user || user.status !== "active") {
   return res.status(401).json({ error: "Please log in first." });
 }
 req.user = user;
 next();
}

function admin(req, res, next) {
 if (req.user.role !== "admin") {
   return res.status(403).json({ error: "Admin access required." });
 }
 next();
}

const page = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CTDK Investment</title>
<style>
:root{--navy:#0a1730;--green:#1bc58d;--bg:#f3f6fb}
*{box-sizing:border-box}
body{margin:0;font:15px Arial,sans-serif;background:var(--bg);color:#18243a}
header{background:var(--navy);color:white;padding:20px;display:flex;justify-content:space-between;align-items:center}
.brand{font-size:22px;font-weight:bold}.brand span{color:var(--green)}
main{max-width:1000px;margin:auto;padding:22px}
.hero{background:var(--navy);color:white;padding:35px 25px;border-radius:18px;margin:15px 0}
h1{font-size:34px;line-height:1.15}.green{color:var(--green)}
button{background:var(--green);border:0;border-radius:9px;padding:12px 16px;font-weight:bold;cursor:pointer;margin:4px 2px}
.secondary{background:white;border:1px solid #ddd}
input,select{width:100%;padding:12px;border:1px solid #d7deea;border-radius:8px;margin:7px 0 14px}
.panel,.stat{background:white;border:1px solid #e1e7ef;border-radius:14px;padding:20px;margin:14px 0}
.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
.two{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.muted{color:#718096;line-height:1.5}.hidden{display:none!important}
label{font-size:13px;font-weight:bold}
table{width:100%;border-collapse:collapse;min-width:500px}
th,td{text-align:left;border-bottom:1px solid #e5eaf0;padding:12px;font-size:13px}
.tablewrap{overflow:auto}.pill{background:#e0f8ee;color:#08774f;padding:6px 10px;border-radius:30px}
.error{color:#b42318}.notice{background:#fff5d9;padding:13px;border-radius:9px;font-size:13px}
footer{text-align:center;padding:20px;color:#718096;font-size:12px}
@media(max-width:650px){main{padding:12px}.grid,.two{grid-template-columns:1fr}h1{font-size:29px}.hero{padding:25px 18px}}
</style>
</head>
<body>
<header><div class="brand">CTDK <span>Investment</span></div>
<button id="logout" class="hidden" onclick="logout()">Log out</button></header>
<main>
<section id="home" class="hero">
<p class="green">YOUR FINANCIAL SPACE</p>
<h1>Plan your future with <span class="green">confidence.</span></h1>
<p>Manage your account and review transaction requests in one place.</p>
<button onclick="showAuth('register')">Create account</button>
<button class="secondary" onclick="showAuth('login')">Log in</button>
<p class="muted">Investments involve risk. Returns are not guaranteed.</p>
</section>

<section id="auth" class="panel hidden">
<button class="secondary" onclick="goHome()">← Back</button>
<h2 id="authTitle">Create account</h2>
<form id="authForm">
<div id="nameBox"><label>Full name</label><input id="name" minlength="2"></div>
<label>Email address</label><input id="email" type="email" required>
<label>Password (minimum 10 characters)</label><input id="password" type="password" minlength="10" required>
<button id="authButton">Create account</button>
<p id="authError" class="error"></p>
</form>
</section>

<section id="dashboard" class="hidden">
<h1>Welcome, <span class="green" id="userName"></span></h1>
<div id="notice" class="notice"></div>
<div class="grid">
<div class="stat"><small>Account</small><h2>Active</h2></div>
<div class="stat"><small>Pending requests</small><h2 id="pending">0</h2></div>
<div class="stat"><small>Transaction records</small><h2 id="total">0</h2></div>
</div>
<div class="two">
<section class="panel">
<h2>Deposit or withdrawal request</h2>
<p class="muted">This records a request only. It does not move money.</p>
<form id="requestForm">
<label>Request type</label>
<select id="type"><option value="deposit">Deposit inquiry</option><option value="withdrawal">Withdrawal request</option></select>
<label>Amount</label><input id="amount" type="number" min="1" step="0.01" required>
<button>Submit request</button>
</form><p id="requestMessage" class="muted"></p>
</section>
<section class="panel">
<h2>Investment information</h2>
<h3>Conservative</h3><p class="muted">Review the risks and fees before investing.</p>
<h3>Balanced</h3><p class="muted">Investment values can rise or fall.</p>
<h3>Growth</h3><p class="muted">Higher risks may mean greater losses.</p>
</section>
</div>
<section class="panel">
<h2>My transaction history</h2>
<div class="tablewrap"><table><thead><tr><th>Type</th><th>Amount</th><th>Status</th><th>Date</th></tr></thead><tbody id="myRows"></tbody></table></div>
</section>
<section id="adminPanel" class="panel hidden">
<h2>Administrator dashboard</h2>
<h3>Users</h3>
<div class="tablewrap"><table><thead><tr><th>Name</th><th>Email</th><th>Role</th><th>Status</th><th>Action</th></tr></thead><tbody id="userRows"></tbody></table></div>
<h3>Transaction requests</h3>
<div class="tablewrap"><table><thead><tr><th>User</th><th>Type</th><th>Amount</th><th>Status</th><th>Action</th></tr></thead><tbody id="requestRows"></tbody></table></div>
</section>
</section>
</main>
<footer>CTDK Investment · Account and request management starter</footer>
<script>
const $=id=>document.getElementById(id);
let mode="register";

function escapeHTML(s){
 return String(s??"").replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));
}
function money(n){
 return new Intl.NumberFormat("en-US",{style:"currency",currency:"USD"}).format(Number(n)||0);
}
function section(id){
 ["home","auth","dashboard"].forEach(s=>$(s).classList.toggle("hidden",s!==id));
}
function goHome(){section("home")}
function showAuth(m){
 mode=m;
 $("authTitle").textContent=m==="register"?"Create account":"Log in";
 $("authButton").textContent=m==="register"?"Create account":"Log in";
 $("nameBox").classList.toggle("hidden",m!=="register");
 $("name").required=m==="register";
 $("authError").textContent="";
 section("auth");
}
async function api(url,options={}){
 const r=await fetch(url,{...options,headers:{"Content-Type":"application/json",...(options.headers||{})}});
 const d=await r.json();
 if(!r.ok)throw Error(d.error||"Request failed");
 return d;
}
$("authForm").onsubmit=async e=>{
 e.preventDefault();$("authError").textContent="";
 try{
  await api("/api/"+mode,{method:"POST",body:JSON.stringify({
   name:$("name").value,email:$("email").value,password:$("password").value
  })});
  await loadDashboard();
 }catch(err){$("authError").textContent=err.message}
};
$("requestForm").onsubmit=async e=>{
 e.preventDefault();$("requestMessage").textContent="Sending…";
 try{
  const d=await api("/api/requests",{method:"POST",body:JSON.stringify({
   type:$("type").value,amount:$("amount").value
  })});
  $("requestMessage").textContent=d.message;
  $("amount").value="";
  await loadDashboard();
 }catch(err){$("requestMessage").textContent=err.message}
};
async function logout(){
 await fetch("/api/logout",{method:"POST"});
 $("logout").classList.add("hidden");
 $("adminPanel").classList.add("hidden");
 goHome();
}
async function loadDashboard(){
 try{
  const d=await api("/api/me");
  $("logout").classList.remove("hidden");
  $("userName").textContent=d.user.name;
  $("notice").textContent=d.notice;
  $("pending").textContent=d.transactions.filter(t=>t.status==="pending").length;
  $("total").textContent=d.transactions.length;
  $("myRows").innerHTML=d.transactions.map(t=>`<tr><td>${escapeHTML(t.type)}</td><td>${money(t.amount)}</td><td>${escapeHTML(t.status)}</td><td>${escapeHTML(t.created_at)}</td></tr>`).join("")||"<tr><td colspan='4'>No requests yet.</td></tr>";
  section("dashboard");
  $("adminPanel").classList.toggle("hidden",d.user.role!=="admin");
  if(d.user.role==="admin")await loadAdmin();
 }catch(e){$("logout").classList.add("hidden");goHome()}
}
async function loadAdmin(){
 const d=await api("/api/admin");
 $("userRows").innerHTML=d.users.map(u=>`<tr><td>${escapeHTML(u.name)}</td><td>${escapeHTML(u.email)}</td><td>${escapeHTML(u.role)}</td><td>${escapeHTML(u.status)}</td><td>${u.role==="admin"?"Admin":`<button onclick="changeUser(${u.id},'${u.status==="active"?"suspended":"active"}')">${u.status==="active"?"Suspend":"Activate"}</button>`}</td></tr>`).join("");
 $("requestRows").innerHTML=d.requests.map(t=>`<tr><td>${escapeHTML(t.name)}</td><td>${escapeHTML(t.type)}</td><td>${money(t.amount)}</td><td>${escapeHTML(t.status)}</td><td><button onclick="changeRequest(${t.id},'approved')">Approve record</button><button onclick="changeRequest(${t.id},'rejected')">Reject</button></td></tr>`).join("")||"<tr><td colspan='5'>No requests.</td></tr>";
}
async function changeUser(id,status){
 try{await api("/api/admin/users/"+id,{method:"PATCH",body:JSON.stringify({status})});await loadAdmin()}catch(e){alert(e.message)}
}
async function changeRequest(id,status){
 try{await api("/api/admin/requests/"+id,{method:"PATCH",body:JSON.stringify({status})});await loadDashboard()}catch(e){alert(e.message)}
}
loadDashboard();
</script>
</body></html>`;

app.get("/", (req,res) => res.type("html").send(page));

app.post("/api/register", async (req,res) => {
 try {
  const name=String(req.body.name||"").trim();
  const email=String(req.body.email||"").trim().toLowerCase();
  const password=String(req.body.password||"");
  if(name.length<2 || !/^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/.test(email) || password.length<10)
   return res.status(400).json({error:"Enter your name, a valid email, and a password of at least 10 characters."});
  const hash=await bcrypt.hash(password,12);
  const result=db.prepare("INSERT INTO users(name,email,password) VALUES(?,?,?)").run(name,email,hash);
  req.session.userId=result.lastInsertRowid;
  res.json({ok:true});
 } catch(e) {
  res.status(409).json({error:"That email may already be registered."});
 }
});

app.post("/api/login", async (req,res) => {
 const email=String(req.body.email||"").trim().toLowerCase();
 const password=String(req.body.password||"");
 const user=db.prepare("SELECT * FROM users WHERE email=?").get(email);
 if(!user || !(await bcrypt.compare(password,user.password)) || user.status!=="active")
  return res.status(401).json({error:"Email/password is incorrect or the account is disabled."});
 req.session.userId=user.id;
 res.json({ok:true});
});

app.post("/api/logout",(req,res)=>{
 req.session.destroy(()=>res.json({ok:true}));
});

app.get("/api/me",auth,(req,res)=>{
 const transactions=db.prepare(
  "SELECT id,type,amount,status,created_at FROM requests WHERE user_id=? ORDER BY id DESC"
 ).all(req.user.id);
 res.json({
  user:req.user,
  transactions,
  notice:"Requests are recorded for review only. No money is held, transferred, or invested by this starter."
 });
});

app.post("/api/requests",auth,(req,res)=>{
 const type=String(req.body.type||"");
 const amount=Number(req.body.amount);
 if(!["deposit","withdrawal"].includes(type) || !Number.isFinite(amount) || amount<=0)
  return res.status(400).json({error:"Enter a valid request type and amount."});
 db.prepare("INSERT INTO requests(user_id,type,amount) VALUES(?,?,?)").run(req.user.id,type,amount);
 res.json({ok:true,message:"Request recorded. No payment was taken and no funds were transferred."});
});

app.get("/api/admin",auth,admin,(req,res)=>{
 const users=db.prepare("SELECT id,name,email,role,status,created_at FROM users ORDER BY id DESC").all();
 const requests=db.prepare(
  "SELECT r.id,r.type,r.amount,r.status,r.created_at,u.name,u.email FROM requests r JOIN users u ON u.id=r.user_id ORDER BY r.id DESC"
 ).all();
 res.json({users,requests});
});

app.patch("/api/admin/users/:id",auth,admin,(req,res)=>{
 const status=String(req.body.status||"");
 if(!["active","suspended"].includes(status))
  return res.status(400).json({error:"Invalid status."});
 const id=Number(req.params.id);
 if(id===req.user.id && status!=="active")
  return res.status(400).json({error:"You cannot suspend your own admin account."});
 db.prepare("UPDATE users SET status=? WHERE id=?").run(status,id);
 res.json({ok:true});
});

app.patch("/api/admin/requests/:id",auth,admin,(req,res)=>{
 const status=String(req.body.status||"");
 if(!["pending","approved","rejected"].includes(status))
  return res.status(400).json({error:"Invalid status."});
 db.prepare("UPDATE requests SET status=? WHERE id=?").run(status,Number(req.params.id));
 res.json({ok:true,message:"Record updated. This does not move money."});
});

createAdmin().then(()=>{
 app.listen(PORT,()=>console.log("CTDK Investment running on port "+PORT));
});
npm init -y
npm install express express-session bcryptjs better-sqlite3
SESSION_SECRET=a-long-random-secret-value
ADMIN_EMAIL=your-admin-email@example.com
ADMIN_PASSWORD=your-strong-admin-password
node server.js
