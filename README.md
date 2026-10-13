<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CTDK Investment - Secure Financial Growth</title>
<style>
:root{--primary:#0f172a;--accent:#0ea5e9;--gold:#f59e0b;--bg:#f8fafc;--card:#ffffff;--text:#1e293b;--muted:#64748b;--border:#e2e8f0}
*{margin:0;padding:0;box-sizing:border-box;font-family:Inter,system-ui,Arial}
body{background:var(--bg);color:var(--text);line-height:1.6}
header{background:var(--card);border-bottom:1px solid var(--border);position:sticky;top:0;z-index:50}
.nav{max-width:1200px;margin:0 auto;display:flex;justify-content:space-between;align-items:center;padding:14px 20px}
.logo{font-weight:800;color:var(--primary);font-size:1.4rem;letter-spacing:-0.5px}.logo span{color:var(--accent)}
nav a{margin-left:18px;text-decoration:none;color:var(--muted);font-weight:600;font-size:.95rem}
.btn{padding:10px 18px;border-radius:8px;border:0;cursor:pointer;font-weight:700}
.btn-primary{background:var(--accent);color:white}.btn-outline{background:transparent;border:1px solid var(--border)}
.hero{max-width:1200px;margin:0 auto;padding:60px 20px;display:grid;grid-template-columns:1.1fr.9fr;gap:30px;align-items:center}
.card{background:var(--card);border:1px solid var(--border);border-radius:16px;padding:22px;box-shadow:0 4px 20px rgba(0,0,0,.05)}
.grid3{display:grid;grid-template-columns:repeat(3,1fr);gap:18px}
@media(max-width:850px){.hero{grid-template-columns:1fr}.grid3{grid-template-columns:1fr}}
.badge{display:inline-block;padding:4px 10px;background:#e0f2fe;color:#0284c7;border-radius:20px;font-size:.8rem;font-weight:700}
input,select{width:100%;padding:12px;border:1px solid var(--border);border-radius:8px;margin-top:6px}
label{font-weight:600;font-size:.9rem;margin-top:14px;display:block}
.section{max-width:1200px;margin:0 auto;padding:40px 20px}
h2{font-size:1.9rem;margin-bottom:12px;color:var(--primary)}
small{color:var(--muted)}
.table{width:100%;border-collapse:collapse;margin-top:12px}.table th,.table td{padding:10px;border-bottom:1px solid var(--border);text-align:left;font-size:.9rem}
.lang-select{padding:8px;border-radius:8px;border:1px solid var(--border)}
</style>
</head>
<body>
<header>
<div class="nav">
<div class="logo">CTDK <span>Investment</span></div>
<div>
<select id="langSwitcher" class="lang-select" onchange="changeLang(this.value)">
<option value="en">English</option><option value="fr">Français</option><option value="es">Español</option><option value="pt">Português</option><option value="ar">العربية</option><option value="yo">Yorùbá</option><option value="ha">Hausa</option><option value="ig">Igbo</option><option value="zh">中文</option>
</select>
<a href="#plans" data-i18n="nav_plans">Plans</a>
<a href="#how" data-i18n="nav_how">How it Works</a>
<a href="#faq" data-i18n="nav_faq">FAQ</a>
<button class="btn btn-outline" onclick="showAuth('login')" data-i18n="login">Login</button>
<button class="btn btn-primary" onclick="showAuth('register')" data-i18n="register">Register</button>
</div>
</div>
</header>

<section class="hero">
<div>
<span class="badge" data-i18n="hero_badge">SECURE • TRANSPARENT • COMPLIANT</span>
<h1 style="font-size:2.6rem;margin:14px 0" data-i18n="hero_title">Build Your Financial Future with CTDK Investment</h1>
<p style="color:var(--muted)" data-i18n="hero_desc">Professional investment management with transparent accounting, verified payments, and real risk disclosure. No fabricated returns.</p>
<div style="margin-top:18px;display:flex;gap:10px">
<button class="btn btn-primary" onclick="showAuth('register')" data-i18n="get_started">Get Started</button>
<a href="#risk" class="btn btn-outline" data-i18n="risk_btn">Risk Disclosure</a>
</div>
<div class="grid3" style="margin-top:24px">
<div class="card"><strong data-i18n="k1">Verified Payments</strong><br><small data-i18n="k1d">Webhook verified</small></div>
<div class="card"><strong data-i18n="k2">Atomic Ledger</strong><br><small data-i18n="k2d">No duplicate credits</small></div>
<div class="card"><strong data-i18n="k3">Audit Logs</strong><br><small data-i18n="k3d">Admin actions tracked</small></div>
</div>
</div>
<div class="card" id="authBox">
<h3 id="authTitle">Create Account</h3>
<label data-i18n="full_name">Full Name</label><input id="fullname" placeholder="John Doe">
<label>Email</label><input id="email" type="email" placeholder="john@example.com">
<label data-i18n="phone">Phone with country code</label><input id="phone" placeholder="+234...">
<label data-i18n="country">Country of Residence</label><select id="country"><option>Nigeria</option><option>Canada</option><option>USA</option><option>UK</option><option>Other</option></select>
<label data-i18n="language_q">Which language do you understand best?</label><select id="prefLang"><option value="en">English</option><option value="fr">French</option><option value="yo">Yoruba</option><option value="ha">Hausa</option><option value="ig">Igbo</option></select>
<label>Password</label><input id="pwd" type="password">
<label data-i18n="confirm_pwd">Confirm Password</label><input id="pwd2" type="password">
<div style="margin-top:12px"><input type="checkbox" id="terms"> <small data-i18n="terms_agree">I accept Terms and Privacy Policy</small></div>
<button class="btn btn-primary" style="width:100%;margin-top:14px" onclick="registerMock()" data-i18n="create_account">Create Account - Send OTP</button>
<small style="display:block;margin-top:8px">TODO Backend: Integrate email/SMS OTP provider. Do not mark verified until verified.</small>
</div>
</section>

<section class="section" id="dashboard" style="display:none">
<h2 data-i18n="dashboard">User Dashboard</h2>
<div class="grid3">
<div class="card"><small data-i18n="wallet">Available Wallet</small><h2 id="walletBal">$0.00</h2></div>
<div class="card"><small data-i18n="invested">Total Invested</small><h2 id="investedBal">$0.00</h2></div>
<div class="card"><small data-i18n="returns">Accrued Returns</small><h2 id="returnsBal">$0.00</h2></div>
</div>
<div class="card" style="margin-top:18px">
<h3 data-i18n="topup">Top Up - Card Payment</h3>
<p><small>Payment Gateway: PENDING CONFIGURATION - Sandbox mode only</small></p>
<input placeholder="Amount e.g. 100" id="topupAmt"><button class="btn btn-primary" style="margin-top:10px" onclick="alert('TODO: Connect Stripe/Paystack - verify webhook server-side, never trust redirect')">Proceed to Secure Checkout</button>
</div>
</section>

<section class="section" id="plans">
<h2 data-i18n="plans_title">Investment Plans</h2>
<p><small data-i18n="plans_note">Plans are database-backed. Returns vary. No guaranteed profits.</small></p>
<div class="grid3" style="margin-top:16px">
<div class="card"><h3>Starter</h3><p><small>Min $50 - Max $499 / 30 days</small></p><p data-i18n="risk_small">Risk: Capital at risk. Returns based on verified performance.</p><button class="btn btn-outline" style="width:100%;margin-top:10px">View Terms</button></div>
<div class="card" style="border:1px solid var(--accent)"><h3>Growth</h3><p><small>Min $500 - Max $4999 / 90 days</small></p><p data-i18n="risk_med">Risk disclosure applies.</p><button class="btn btn-primary" style="width:100%;margin-top:10px">View Terms</button></div>
<div class="card"><h3>Institutional</h3><p><small>Custom - Requires KYC & approval</small></p><button class="btn btn-outline" style="width:100%;margin-top:10px">Contact Compliance</button></div>
</div>
</section>

<section class="section" id="how"><h2 data-i18n="how_title">How It Works</h2><ol style="margin-left:20px"><li data-i18n="how1">Register with language and country</li><li data-i18n="how2">Verify email and phone via OTP</li><li data-i18n="how3">Top up via verified card payment</li><li data-i18n="how4">Choose plan, review risk and fees</li><li data-i18n="how5">Daily return job processes eligible investments at 06:00 UTC with idempotency key</li><li data-i18n="how6">Withdraw with ownership verification</li></ol></section>

<section class="section" id="risk" class="card"><h2>Risk Disclosure</h2><p>Investing involves risk. Past performance does not guarantee future results. You may lose some or all invested capital. CTDK Investment does not promise fixed daily profits without substantiated basis. All returns must be backed by real financial records.</p></section>

<section class="section" id="faq"><h2>FAQ</h2><div class="card"><strong>Is my card PIN needed?</strong><br><small>Never. We use tokenization. We never store PIN or CVV.</small></div></section>

<footer class="section" style="border-top:1px solid var(--border)"><small>© CTDK Investment - Admin dashboard requires separate secure auth, role-based access, audit logs. Environment variables for secrets. Production requires HTTPS, backups, rate limiting.</small></footer>

<script>
const dict={
en:{nav_plans:"Plans",nav_how:"How It Works",nav_faq:"FAQ",login:"Login",register:"Register",hero_badge:"SECURE • TRANSPARENT • COMPLIANT",hero_title:"Build Your Financial Future with CTDK Investment",hero_desc:"Professional investment management with transparent accounting, verified payments, and real risk disclosure. No fabricated returns.",get_started:"Get Started",risk_btn:"Risk Disclosure",k1:"Verified Payments",k1d:"Webhook verified",k2:"Atomic Ledger",k2d:"No duplicate credits",k3:"Audit Logs",k3d:"Admin actions tracked",full_name:"Full Name",phone:"Phone with country code",country:"Country of Residence",language_q:"Which language do you understand best?",confirm_pwd:"Confirm Password",terms_agree:"I accept Terms and Privacy Policy",create_account:"Create Account - Send OTP",dashboard:"User Dashboard",wallet:"Available Wallet",invested:"Total Invested",returns:"Accrued Returns",topup:"Top Up - Card Payment",plans_title:"Investment Plans",plans_note:"Plans are database-backed. Returns vary. No guaranteed profits.",risk_small:"Risk: Capital at risk. Returns based on verified performance.",risk_med:"Risk disclosure applies.",how_title:"How It Works",how1:"Register with language and country",how2:"Verify email and phone via OTP",how3:"Top up via verified card payment",how4:"Choose plan, review risk and fees",how5:"Daily return job processes eligible investments at 06:00 UTC with idempotency key",how6:"Withdraw with ownership verification"},
fr:{nav_plans:"Plans",hero_title:"Construisez votre avenir financier",login:"Connexion",register:"S'inscrire"},
yo:{hero_title:"Kọ ọjọ iwaju owo rẹ pẹlu CTDK",login:"Wọle",register:"Forukọsilẹ"},
ha:{hero_title:"Gina makomar kuɗin ku da CTDK",login:"Shiga",register:"Yi rijista"},
ig:{hero_title:"Wulite Ọdịnihu Ego Gị na CTDK",login:"Banye",register:"Debanye aha"},
ar:{hero_title:"ابنِ مستقبلك المالي مع CTDK",login:"تسجيل الدخول",register:"تسجيل"}
};
function changeLang(l){localStorage.setItem('ctdk_lang',l);applyLang(l)}
function applyLang(l){const d=dict[l]||dict.en;document.querySelectorAll('[data-i18n]').forEach(el=>{const k=el.getAttribute('data-i18n');if(d[k])el.textContent=d[k]});if(l==='ar')document.documentElement.dir='rtl';else document.documentElement.dir='ltr'}
function showAuth(t){document.getElementById('authBox').scrollIntoView({behavior:'smooth'});document.getElementById('authTitle').textContent=t==='login'?'Login':'Create Account'}
function registerMock(){const em=document.getElementById('email').value;if(!em){alert('Enter email');return}alert('MOCK: OTP would be sent via configured provider. Backend TODO.');document.getElementById('dashboard').style.display='block';document.getElementById('dashboard').scrollIntoView()}
applyLang(localStorage.getItem('ctdk_lang')||'en');
</script>
</body>
</html>
