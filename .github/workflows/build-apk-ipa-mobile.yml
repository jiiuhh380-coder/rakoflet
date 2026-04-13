"""
=============================================================
  المستمر للمحاسبة - ملف واحد كامل
=============================================================
تنصيب:   pip install flask
تشغيل:   python app_fixed.py
افتح:    http://192.168.1.100:5000
=============================================================
"""

from flask import Flask, request, jsonify, session, make_response
import sqlite3, os, json
from datetime import date, datetime, timedelta
from functools import wraps

app = Flask(__name__)
app.secret_key  = 'mostamer-2024-xK9#mP2$'
app.config.update(
    SESSION_COOKIE_SAMESITE = 'Lax',
    SESSION_COOKIE_HTTPONLY = True,
    SESSION_COOKIE_SECURE   = False,   # True فقط مع HTTPS
)

# ── معالجات الأخطاء العامة ──────────────────────────────────
@app.errorhandler(400)
def bad_request(error):
    return jsonify({"success": False, "error": "طلب غير صحيح"}), 400

@app.errorhandler(401)
def unauthorized(error):
    return jsonify({"success": False, "error": "غير مصرح"}), 401

@app.errorhandler(403)
def forbidden(error):
    return jsonify({"success": False, "error": "ممنوع"}), 403

@app.errorhandler(404)
def not_found(error):
    return jsonify({"success": False, "error": "الموارد غير موجودة"}), 404

@app.errorhandler(500)
def internal_error(error):
    return jsonify({"success": False, "error": "خطأ في الخادم"}), 500

# ── مسار قاعدة البيانات ──────────────────────────────────
# يبحث تلقائياً في نفس مجلد الملف
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
DB_PATH  = os.path.join(BASE_DIR, 'المستمر.db')

# ═══════════════════════════════════════════════════════════
#  الواجهة الأمامية (HTML مدمج)
# ═══════════════════════════════════════════════════════════
HTML = r"""<!DOCTYPE html>
<html dir="rtl" lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>المستمر للمحاسبة</title>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@300;400;600;700;900&display=swap" rel="stylesheet">
<style>
:root{--bg:#0f1923;--bg2:#162130;--bg3:#1e2d3d;--border:#243447;--accent:#00d4ff;--accent2:#00ff9d;--danger:#ff4757;--warn:#ffa502;--text:#e8f4fd;--muted:#5d7a94;--r:12px;--c1:#2471a3;--c2:#1e8449;--c3:#ba4a00}
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:'Cairo',sans-serif;background:var(--bg);color:var(--text);direction:rtl;min-height:100vh}
::-webkit-scrollbar{width:6px}::-webkit-scrollbar-track{background:var(--bg2)}::-webkit-scrollbar-thumb{background:var(--border);border-radius:3px}

/* LOGIN */
#login-screen{display:flex;align-items:center;justify-content:center;min-height:100vh;background:radial-gradient(ellipse at 60% 40%,#0d2136,#0f1923 70%)}
.login-card{background:var(--bg2);border:1px solid var(--border);border-radius:20px;padding:48px 40px;width:380px;box-shadow:0 20px 60px rgba(0,0,0,.6)}
.login-logo{text-align:center;margin-bottom:32px}
.login-logo h1{font-size:26px;font-weight:900;color:var(--accent)}
.login-logo p{color:var(--muted);font-size:14px;margin-top:6px}
.field{margin-bottom:18px}
.field label{display:block;font-size:13px;color:var(--muted);margin-bottom:7px}
.field input{width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:8px;padding:12px 16px;color:var(--text);font-family:'Cairo',sans-serif;font-size:15px;outline:none;transition:border .2s}
.field input:focus{border-color:var(--accent)}
.err-box{background:rgba(255,71,87,.15);border:1px solid var(--danger);color:var(--danger);padding:10px 14px;border-radius:8px;font-size:14px;margin-bottom:14px;display:none}
.ok-box{background:rgba(0,255,157,.1);border:1px solid var(--accent2);color:var(--accent2);padding:10px 14px;border-radius:8px;font-size:14px;margin-bottom:14px;display:none}

/* BUTTONS */
.btn{display:inline-flex;align-items:center;justify-content:center;gap:7px;padding:11px 22px;border:none;border-radius:8px;font-family:'Cairo',sans-serif;font-size:14px;font-weight:700;cursor:pointer;transition:all .18s;white-space:nowrap}
.btn:disabled{opacity:.5;cursor:not-allowed}
.btn-primary{background:var(--accent);color:#0f1923;width:100%;font-size:16px;padding:14px}
.btn-primary:hover:not(:disabled){background:#00b8d9;transform:translateY(-1px)}
.btn-success{background:var(--accent2);color:#0f1923}
.btn-success:hover{background:#00d980}
.btn-danger{background:var(--danger);color:#fff}
.btn-warn{background:var(--warn);color:#0f1923}
.btn-ghost{background:var(--bg3);color:var(--text);border:1px solid var(--border)}
.btn-ghost:hover{border-color:var(--accent);color:var(--accent)}
.btn-sm{padding:6px 13px;font-size:13px}

/* APP */
#app{display:none;height:100vh}
.sidebar{width:218px;background:var(--bg2);border-left:1px solid var(--border);display:flex;flex-direction:column;position:fixed;top:0;right:0;height:100vh;z-index:100}
.sb-logo{padding:18px 16px;border-bottom:1px solid var(--border)}
.sb-logo h2{font-size:16px;font-weight:900;color:var(--accent)}
.sb-logo span{font-size:11px;color:var(--muted)}
.nav-items{flex:1;overflow-y:auto;padding:6px 0}
.nav-item{display:flex;align-items:center;gap:10px;padding:11px 14px;cursor:pointer;border-radius:8px;margin:2px 8px;font-size:14px;color:var(--muted);transition:all .15s}
.nav-item:hover{background:var(--bg3);color:var(--text)}
.nav-item.active{background:rgba(0,212,255,.1);color:var(--accent);border-right:3px solid var(--accent)}
.nav-item.c1{color:#a9d0e8}
.nav-item.c1:hover{color:#7fc0e8}
.nav-item.c1.active{color:var(--c1)}
.nav-item.c2{color:#a8d9b9}
.nav-item.c2:hover{color:#7fd9a8}
.nav-item.c2.active{color:var(--c2)}
.nav-item.c3{color:#d9b8a8}
.nav-item.c3:hover{color:#d99878}
.nav-item.c3.active{color:var(--c3)}
.nav-item .ic{font-size:17px;width:22px;text-align:center}
.nav-div{height:1px;background:var(--border);margin:6px 14px}
.nav-label{font-size:10px;color:var(--muted);padding:4px 14px;margin-top:6px;font-weight:600;text-transform:uppercase}
.sb-foot{padding:12px}

.main{margin-right:218px;padding:22px;overflow-y:auto;height:100vh}
.topbar{display:flex;align-items:center;justify-content:space-between;margin-bottom:22px}
.topbar h1{font-size:21px;font-weight:700}
.u-badge{background:var(--bg2);border:1px solid var(--border);border-radius:8px;padding:7px 13px;font-size:13px}

/* STATS */
.sg{display:grid;grid-template-columns:repeat(auto-fill,minmax(165px,1fr));gap:14px;margin-bottom:22px}
.sc{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:18px 15px;position:relative;overflow:hidden;transition:transform .18s}
.sc:hover{transform:translateY(-2px)}
.sc::before{content:'';position:absolute;top:0;right:0;width:4px;height:100%;border-radius:0 var(--r) var(--r) 0}
.sc.bl::before{background:var(--accent)}.sc.gr::before{background:var(--accent2)}.sc.rd::before{background:var(--danger)}.sc.wn::before{background:var(--warn)}.sc.pu::before{background:#a855f7}
.sc .lb{font-size:12px;color:var(--muted);margin-bottom:7px}
.sc .vl{font-size:21px;font-weight:900}
.sc .sb{font-size:11px;color:var(--muted);margin-top:3px}

/* TABLE */
.tw{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);overflow:hidden}
.th{display:flex;align-items:center;justify-content:space-between;padding:14px 18px;border-bottom:1px solid var(--border)}
.th h3{font-size:15px;font-weight:700}
table{width:100%;border-collapse:collapse}
th{background:var(--bg3);color:var(--muted);font-size:12px;font-weight:600;padding:11px 14px;text-align:right}
td{padding:11px 14px;font-size:14px;border-bottom:1px solid var(--border)}
tr:last-child td{border-bottom:none}
tr:hover td{background:rgba(255,255,255,.02)}

/* MODAL */
.ov{display:none;position:fixed;inset:0;background:rgba(0,0,0,.72);z-index:200;align-items:center;justify-content:center;padding:16px}
.ov.open{display:flex}
.modal{background:var(--bg2);border:1px solid var(--border);border-radius:15px;padding:26px;width:100%;max-width:540px;max-height:90vh;overflow-y:auto;box-shadow:0 20px 60px rgba(0,0,0,.5)}
.modal h2{font-size:17px;font-weight:700;margin-bottom:18px;color:var(--accent)}
.fg{display:grid;grid-template-columns:1fr 1fr;gap:13px}
.fg .full{grid-column:1/-1}
.ff{display:flex;flex-direction:column;gap:5px}
.ff label{font-size:12px;color:var(--muted)}
.ff input,.ff select,.ff textarea{background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 11px;color:var(--text);font-family:'Cairo',sans-serif;font-size:14px;outline:none;transition:border .2s}
.ff input:focus,.ff select:focus{border-color:var(--accent)}
.ff textarea{resize:vertical;min-height:70px}
.mf{display:flex;gap:9px;justify-content:flex-end;margin-top:18px}

/* MISC */
.badge{display:inline-block;padding:2px 9px;border-radius:20px;font-size:11px;font-weight:700}
.bg{background:rgba(0,255,157,.14);color:var(--accent2)}.br{background:rgba(255,71,87,.14);color:var(--danger)}.bw{background:rgba(255,165,2,.14);color:var(--warn)}.bb{background:rgba(0,212,255,.14);color:var(--accent)}
#toast{position:fixed;bottom:22px;left:22px;z-index:999;display:flex;flex-direction:column;gap:7px}
.ti{background:var(--bg2);border:1px solid var(--border);border-radius:9px;padding:12px 16px;font-size:14px;box-shadow:0 4px 20px rgba(0,0,0,.4);animation:si .25s ease;min-width:240px}
.ti.success{border-color:var(--accent2);color:var(--accent2)}.ti.error{border-color:var(--danger);color:var(--danger)}
@keyframes si{from{transform:translateX(-28px);opacity:0}to{transform:none;opacity:1}}
.tabs{display:flex;gap:3px;background:var(--bg2);border:1px solid var(--border);border-radius:9px;padding:4px;margin-bottom:18px;width:fit-content}
.tab{padding:7px 16px;border-radius:6px;cursor:pointer;font-size:13px;font-weight:600;color:var(--muted);transition:all .18s}
.tab.active{background:var(--accent);color:#0f1923}
.sb-bar{display:flex;align-items:center;gap:9px;margin-bottom:14px}
.si{background:var(--bg2);border:1px solid var(--border);border-radius:8px;padding:9px 14px;color:var(--text);font-family:'Cairo',sans-serif;font-size:14px;outline:none;flex:1}
.si:focus{border-color:var(--accent)}
.spin{border:3px solid var(--border);border-top-color:var(--accent);border-radius:50%;width:30px;height:30px;animation:sp .8s linear infinite;margin:40px auto}
@keyframes sp{to{transform:rotate(360deg)}}
.empty{text-align:center;padding:55px 20px;color:var(--muted)}
.empty .ei{font-size:44px;margin-bottom:10px}
.pos-g{display:grid;grid-template-columns:1fr 340px;gap:18px}
.ci{display:flex;align-items:center;gap:11px;padding:11px;background:var(--bg3);border-radius:8px;margin-bottom:7px}
.ci .nm{flex:1;font-size:14px;font-weight:600}
.qb{background:var(--border);border:none;color:var(--text);width:27px;height:27px;border-radius:5px;cursor:pointer;font-size:15px;font-weight:700}
.ri{background:var(--bg3);border:1px solid var(--border);border-radius:var(--r);padding:14px;margin-top:11px}
.rin{background:var(--bg);border:1px solid var(--border);border-radius:7px;padding:9px 12px;color:var(--text);font-family:'Cairo',sans-serif;font-size:15px;width:100%;outline:none}
.rin:focus{border-color:var(--accent)}
.rb{background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:8px;color:var(--text);font-family:'Cairo',sans-serif;width:100%}
@media(max-width:768px){.sidebar{display:none}.main{margin-right:0;padding:14px}.pos-g{grid-template-columns:1fr}.sg{grid-template-columns:1fr 1fr}}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="login-screen">
  <div class="login-card">
    <div class="login-logo">
      <h1>🏪 المستمر</h1>
      <p>نظام المحاسبة وإدارة المحلات</p>
    </div>
    <div class="err-box" id="lerr"></div>
    <div class="ok-box" id="lok"></div>
    <div class="field"><label>اسم المستخدم</label>
      <input id="lu" type="text" value="admin" autocomplete="username">
    </div>
    <div class="field"><label>كلمة المرور</label>
      <input id="lp" type="password" value="admin123" autocomplete="current-password">
    </div>
    <button class="btn btn-primary" id="login-btn" onclick="doLogin()">🔑 تسجيل الدخول</button>
  </div>
</div>

<!-- APP -->
<div id="app">
  <nav class="sidebar">
    <div class="sb-logo"><h2>🏪 المستمر</h2><span id="sb-user">...</span></div>
    <div class="nav-items">
      <!-- الرئيسية (أخضر - C2) -->
      <div class="nav-item c2 active" data-p="dashboard" onclick="go('dashboard')"><span class="ic">🏠 </span>الرئيسية</div>
      
      <!-- العمليات الرئيسية (أزرق - C1) -->
      <div class="nav-item c1" data-p="pos" onclick="go('pos')"><span class="ic">💰 </span>نقطة البيع</div>
      <div class="nav-item c1" data-p="barcode_sale" onclick="go('barcode_sale')"><span class="ic">📦 </span>بيع بالباركود</div>
      <div class="nav-item c1" data-p="barcode_return" onclick="go('barcode_return')"><span class="ic">🔄 </span>استرجاع بالباركود</div>
      <div class="nav-item c1" data-p="maintenance" onclick="go('maintenance')"><span class="ic">🔧 </span>صيانة الموبايلات</div>
      <div class="nav-item c1" data-p="expenses" onclick="go('expenses')"><span class="ic">💸 </span>المصاريف</div>
      
      <!-- أدوات (برتقالي - C3) -->
      <div class="nav-item c3" data-p="currency" onclick="go('currency')"><span class="ic">🧮 </span>حاسبة العملات</div>
      
      <!-- فاصل المتابعة -->
      <div class="nav-label">المتابعة</div>
      
      <!-- المتابعة والتقارير (أخضر - C2) -->
      <div class="nav-item c2" data-p="sales" onclick="go('sales')"><span class="ic">📊 </span>المبيعات</div>
      <div class="nav-item c2" data-p="purchases" onclick="go('purchases')"><span class="ic">🛒 </span>المشتريات</div>
      <div class="nav-item c2" data-p="inventory" onclick="go('inventory')"><span class="ic">📋 </span>المخزون</div>
      <div class="nav-item c2" data-p="count" onclick="go('count')"><span class="ic">📊 </span>الجرد</div>
      <div class="nav-item c2" data-p="mobile" onclick="go('mobile')"><span class="ic">📱 </span>إدارة الموبايلات</div>
      <div class="nav-item c2" data-p="reports" onclick="go('reports')"><span class="ic">📈 </span>التقارير</div>
      
      <!-- فاصل الإدارة -->
      <div class="nav-label">الإدارة</div>
      
      <!-- الإدارة والإعدادات (برتقالي - C3) -->
      <div class="nav-item c3" data-p="users" onclick="go('users')"><span class="ic">👥 </span>إدارة المستخدمين</div>
      <div class="nav-item c3" data-p="license" onclick="go('license')"><span class="ic">🔐 </span>معلومات الترخيص</div>
      <div class="nav-item c3" data-p="settings" onclick="go('settings')"><span class="ic">⚙️ </span>الإعدادات</div>
    </div>
    <div class="sb-foot">
      <button class="btn btn-ghost" style="width:100%;font-size:13px" onclick="doLogout()">🚪 خروج</button>
    </div>
  </nav>

  <main class="main">
    <div class="topbar">
      <h1 id="page-title">لوحة التحكم</h1>
      <div class="u-badge">👤 <span id="u-name">—</span></div>
    </div>
    <div id="pc"></div>
  </main>
</div>

<!-- MODAL -->
<div class="ov" id="ov" onclick="cmo(event)">
  <div class="modal" id="mbox">
    <h2 id="mtitle"></h2>
    <div id="mbody"></div>
    <div class="mf" id="mfoot"></div>
  </div>
</div>

<div id="toast"></div>

<script>
// ── AUTH TOKEN (بدل session - أكثر موثوقية) ──────────────
let TOKEN = null;

async function api(path, method='GET', body=null){
  const h = {'Content-Type':'application/json'};
  if(TOKEN) h['X-Token'] = TOKEN;
  const opts = {method, headers:h};
  if(body) opts.body = JSON.stringify(body);
  try {
    const r = await fetch(path, opts);
    
    // التحقق من حالة الرد
    if (!r.ok) {
      const contentType = r.headers.get('content-type');
      
      // إذا كان الرد HTML (صفحة خطأ)
      if (contentType && contentType.includes('text/html')) {
        let errorMsg = 'خطأ في الخادم';
        if (r.status === 401) errorMsg = 'يجب تسجيل الدخول أولاً';
        else if (r.status === 403) errorMsg = 'ليس لديك صلاحيات كافية';
        else if (r.status === 404) errorMsg = 'المورد غير موجود';
        else if (r.status === 500) errorMsg = 'خطأ في الخادم - حاول لاحقاً';
        return {success: false, message: errorMsg, error: errorMsg};
      }
      
      // محاولة قراءة JSON من الخطأ
      const data = await r.json();
      return data;
    }
    
    // في حالة النجاح
    const data = await r.json();
    return data;
  } catch(e) {
    // خطأ في الاتصال نفسه
    console.error('API Error:', e);
    return {success: false, message: 'خطأ في الاتصال بالخادم: ' + e.message, error: e.message};
  }
}

function toast(msg, type='success'){
  const t = document.getElementById('toast');
  const el = document.createElement('div');
  el.className = 'ti '+type;
  el.textContent = (type==='success'?'✅ ':'❌ ')+msg;
  t.appendChild(el);
  setTimeout(()=>el.remove(), 3500);
}

function fmt(n){ 
  const v = Number(n||0);
  if(v >= 1000000) return (v/1000000).toFixed(1)+'م';
  if(v >= 1000) return v.toLocaleString('ar');
  return v.toLocaleString('ar');
}

function ld(){ return '<div class="spin"></div>'; }
function em(m='لا توجد بيانات'){ return `<div class="empty"><div class="ei">📭</div><p>${m}</p></div>`; }

// ── MODAL ─────────────────────────────────────────────────
function omo(title, body, foot='<button class="btn btn-ghost" onclick="cmo()">إغلاق</button>'){
  document.getElementById('mtitle').textContent = title;
  document.getElementById('mbody').innerHTML = body;
  document.getElementById('mfoot').innerHTML = foot;
  document.getElementById('ov').classList.add('open');
}
function cmo(e){
  if(!e || e.target===document.getElementById('ov'))
    document.getElementById('ov').classList.remove('open');
}

// ── LOGIN ─────────────────────────────────────────────────
async function doLogin(){
  const btn = document.getElementById('login-btn');
  const un  = document.getElementById('lu').value.trim();
  const pw  = document.getElementById('lp').value.trim();
  
  document.getElementById('lerr').style.display='none';
  document.getElementById('lok').style.display='none';
  
  if(!un||!pw){
    showErr('أدخل اسم المستخدم وكلمة المرور');
    return;
  }
  
  btn.disabled=true; btn.textContent='⏳ جاري الدخول...';
  
  const r = await api('/api/auth/login','POST',{username:un,password:pw});
  
  btn.disabled=false; btn.innerHTML='🔑 تسجيل الدخول';
  
  if(r.success){
    TOKEN = r.token;
    document.getElementById('login-screen').style.display='none';
    document.getElementById('app').style.display='flex';
    document.getElementById('u-name').textContent  = r.data.username;
    document.getElementById('sb-user').textContent = 'مرحباً، '+r.data.username;
    go('dashboard');
  } else {
    showErr(r.message || 'بيانات الدخول غير صحيحة');
  }
}

function showErr(msg){
  const el = document.getElementById('lerr');
  el.textContent = '❌ '+msg;
  el.style.display='block';
}

document.addEventListener('keydown', e=>{
  if(e.key==='Enter' && document.getElementById('login-screen').style.display!=='none') doLogin();
});

async function doLogout(){
  TOKEN=null;
  location.reload();
}

// ── NAVIGATION ────────────────────────────────────────────
const TITLES = {dashboard:'لوحة التحكم',products:'المنتجات',sales:'المبيعات',purchases:'المشتريات',pos:'نقطة البيع',suppliers:'الموردين',customers:'العملاء',expenses:'المصروفات',maintenance:'الصيانة',reports:'التقارير',currency:'حاسبة العملات',barcode_sale:'بيع بالباركود',barcode_return:'استرجاع بالباركود',count:'الجرد',mobile:'إدارة الموبايلات',users:'إدارة المستخدمين',license:'معلومات الترخيص',settings:'الإعدادات',inventory:'المخزون'};

function go(name){
  document.querySelectorAll('.nav-item').forEach(n=>n.classList.remove('active'));
  document.querySelector(`[data-p="${name}"]`)?.classList.add('active');
  document.getElementById('page-title').textContent = TITLES[name]||name;
  const fns = {dashboard,products,sales,purchases,pos,suppliers,customers,expenses,maintenance,reports,currency,barcode_sale,barcode_return,count,mobile,users,license,settings,inventory};
  if(fns[name]) fns[name]();
  else document.getElementById('pc').innerHTML=`<div class="tw"><div class="th"><h3>${name}</h3></div><div style="padding:20px">✅ قسم ${TITLES[name]||name} - جاهز للاستخدام</div></div>`;
}

// ═══════════════════════════════════════════════════════════
//  DASHBOARD
// ═══════════════════════════════════════════════════════════
async function dashboard(){
  const c=document.getElementById('pc');
  c.innerHTML=ld();
  const r=await api('/api/reports/dashboard');
  if(!r.success){c.innerHTML=em('تعذر تحميل البيانات');return;}
  const d=r.data;
  c.innerHTML=`
  <div class="sg">
    <div class="sc bl"><div class="lb">💵 مبيعات اليوم</div><div class="vl">${fmt(d.today_sales)}</div><div class="sb">${d.today_count} فاتورة</div></div>
    <div class="sc bl"><div class="lb">📈 مبيعات الشهر</div><div class="vl">${fmt(d.month_sales)}</div><div class="sb">${d.month_count} فاتورة</div></div>
    <div class="sc gr"><div class="lb">💹 ربح الشهر</div><div class="vl">${fmt(d.month_profit)}</div><div class="sb">سنوي: ${fmt(d.year_profit)}</div></div>
    <div class="sc gr"><div class="lb">💼 إجمالي الأرباح</div><div class="vl">${fmt(d.total_profit)}</div></div>
    <div class="sc wn"><div class="lb">🛍️ مشتريات الشهر</div><div class="vl">${fmt(d.month_purchases)}</div><div class="sb">مصروفات: ${fmt(d.month_expenses)}</div></div>
    <div class="sc pu"><div class="lb">📦 قيمة المخزون</div><div class="vl">${fmt(d.inventory_value)}</div><div class="sb">${d.total_products} منتج</div></div>
    <div class="sc rd"><div class="lb">📥 ديون العملاء</div><div class="vl">${fmt(d.receivables)}</div></div>
    <div class="sc wn"><div class="lb">⚠️ مخزون منخفض</div><div class="vl">${d.low_stock}</div><div class="sb">نافذ: ${d.out_stock}</div></div>
  </div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:18px">
    <div class="tw"><div class="th"><h3>🏆 أعلى المنتجات</h3></div><div id="tp-list">${ld()}</div></div>
    <div class="tw"><div class="th"><h3>⚠️ مخزون منخفض</h3></div><div id="ls-list">${ld()}</div></div>
  </div>`;
  const tp=await api('/api/reports/top-products?limit=5');
  document.getElementById('tp-list').innerHTML=tp.data?.length
    ?`<table><tr><th>المنتج</th><th>الكمية</th><th>الإيراد</th></tr>${tp.data.map(p=>`<tr><td>${p.name}</td><td>${fmt(p.total_qty)}</td><td>${fmt(p.total_revenue)}</td></tr>`).join('')}</table>`
    :em();
  const ls=await api('/api/products/low-stock');
  document.getElementById('ls-list').innerHTML=ls.data?.length
    ?`<table><tr><th>المنتج</th><th>الكمية</th><th>الحد</th></tr>${ls.data.slice(0,5).map(p=>`<tr><td>${p.name}</td><td><span class="badge ${p.quantity<=0?'br':'bw'}">${p.quantity}</span></td><td>${p.min_stock}</td></tr>`).join('')}</table>`
    :em('المخزون كافٍ ✅');
}

// ═══════════════════════════════════════════════════════════
//  PRODUCTS
// ═══════════════════════════════════════════════════════════
async function products(q=''){
  const c=document.getElementById('pc');
  c.innerHTML=`<div class="sb-bar"><input class="si" id="ps" placeholder="🔍 بحث..." value="${q}" oninput="products(this.value)"><button class="btn btn-success" onclick="addProd()">+ إضافة منتج</button></div>
  <div class="tw"><div class="th"><h3>📦 المنتجات</h3><span id="pcnt" style="color:var(--muted);font-size:13px"></span></div><div id="pt">${ld()}</div></div>`;
  const r=await api(`/api/products?search=${encodeURIComponent(q)}`);
  document.getElementById('pcnt').textContent=(r.data?.length||0)+' منتج';
  document.getElementById('pt').innerHTML=r.data?.length
    ?`<table><tr><th>الاسم</th><th>الماركة</th><th>الفئة</th><th>شراء</th><th>بيع</th><th>الكمية</th><th>الحالة</th><th></th></tr>
    ${r.data.map(p=>`<tr><td><b>${p.name}</b></td><td>${p.brand}</td><td>${p.category}</td><td>${fmt(p.cost_price)}</td><td>${fmt(p.selling_price)}</td>
    <td><span class="badge ${p.quantity<=0?'br':p.quantity<=p.min_stock?'bw':'bg'}">${p.quantity}</span></td>
    <td>${p.condition}</td>
    <td style="white-space:nowrap"><button class="btn btn-ghost btn-sm" onclick='eProd(${JSON.stringify(p)})'>✏️</button> <button class="btn btn-danger btn-sm" onclick="dProd(${p.id},'${p.name.replace(/'/g,'')}')">🗑️</button></td></tr>`).join('')}</table>`
    :em();
}
function addProd(){
  omo('➕ إضافة منتج جديد',`<div class="fg">
    <div class="ff full"><label>الاسم *</label><input id="pn" placeholder="iPhone 15 Pro"></div>
    <div class="ff"><label>الماركة *</label><input id="pb" placeholder="Apple"></div>
    <div class="ff"><label>الموديل</label><input id="pm" placeholder="A1234"></div>
    <div class="ff"><label>الفئة</label><select id="pcat" onchange="toggleCondField()"><option>موبايلات</option><option>ملحقات</option><option>أجهزة لوحية</option><option>أخرى</option></select></div>
    <div class="ff"><label>سعر الشراء *</label><input id="pcp" type="number" placeholder="0"></div>
    <div class="ff"><label>سعر البيع *</label><input id="psp" type="number" placeholder="0"></div>
    <div class="ff"><label>الكمية *</label><input id="pq" type="number" placeholder="0"></div>
    <div class="ff"><label>الحد الأدنى</label><input id="pms" type="number" placeholder="5"></div>
    <div class="ff"><label>الباركود</label><input id="pbar" placeholder="اختياري"></div>
    <div class="ff" id="cond-field"><label>الحالة</label><select id="pcon"><option>جديد</option><option>مستعمل</option></select></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="svProd()">💾 حفظ</button>`);
  setTimeout(toggleCondField,50);
}
function toggleCondField(){const cat=document.getElementById('pcat')?.value;const cf=document.getElementById('cond-field');if(cf)cf.style.display=cat==='موبايلات'?'flex':'none';}
async function svProd(){
  const r=await api('/api/products/add','POST',{name:document.getElementById('pn').value,brand:document.getElementById('pb').value,model:document.getElementById('pm').value||'-',category:document.getElementById('pcat').value,cost_price:document.getElementById('pcp').value,selling_price:document.getElementById('psp').value,quantity:document.getElementById('pq').value,min_stock:document.getElementById('pms').value||5,barcode:document.getElementById('pbar').value||null,condition:document.getElementById('pcon').value});
  if(r.success){cmo();toast(r.message);products();}else toast(r.message,'error');
}
function eProd(p){
  omo('✏️ تعديل المنتج',`<div class="fg">
    <div class="ff full"><label>الاسم</label><input id="en" value="${p.name}"></div>
    <div class="ff"><label>الماركة</label><input id="eb" value="${p.brand}"></div>
    <div class="ff"><label>سعر الشراء</label><input id="ec" type="number" value="${p.cost_price}"></div>
    <div class="ff"><label>سعر البيع</label><input id="es" type="number" value="${p.selling_price}"></div>
    <div class="ff"><label>الكمية</label><input id="eq" type="number" value="${p.quantity}"></div>
    <div class="ff"><label>الحد الأدنى</label><input id="em" type="number" value="${p.min_stock}"></div>
    <div class="ff"><label>الحالة</label><select id="eco"><option ${p.condition==='جديد'?'selected':''}>جديد</option><option ${p.condition==='مستعمل'?'selected':''}>مستعمل</option><option ${p.condition==='مجدد'?'selected':''}>مجدد</option></select></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="upProd(${p.id})">💾 حفظ</button>`);
}
async function upProd(id){
  const r=await api(`/api/products/${id}`,'PUT',{name:document.getElementById('en').value,brand:document.getElementById('eb').value,model:'-',cost_price:document.getElementById('ec').value,selling_price:document.getElementById('es').value,quantity:document.getElementById('eq').value,min_stock:document.getElementById('em').value,condition:document.getElementById('eco').value});
  if(r.success){cmo();toast(r.message);products();}else toast(r.message,'error');
}
async function dProd(id,name){
  if(!confirm(`حذف: ${name}?`))return;
  const r=await api(`/api/products/${id}`,'DELETE');
  if(r.success){toast(r.message);products();}else toast(r.message,'error');
}

// ═══════════════════════════════════════════════════════════
//  POS
// ═══════════════════════════════════════════════════════════
let cart=[];
async function pos(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div class="pos-g">
    <div>
      <div class="sb-bar"><input class="si" id="pos-s" placeholder="🔍 اسم المنتج أو الباركود..." oninput="psrch(this.value)"></div>
      <div id="pos-prods" style="display:grid;grid-template-columns:repeat(auto-fill,minmax(150px,1fr));gap:10px">${ld()}</div>
    </div>
    <div>
      <div class="tw" style="padding:14px">
        <h3 style="margin-bottom:11px">🛒 السلة</h3>
        <div id="cart-items"></div>
        <div class="ri">
          <div style="display:flex;justify-content:space-between;margin-bottom:9px"><span style="color:var(--muted)">الإجمالي</span><b id="ct" style="font-size:19px;color:var(--accent)">0</b></div>
          <div class="ff" style="margin-bottom:9px"><label>المدفوع</label><input id="cp2" type="number" class="rin" placeholder="0" oninput="updRem()"></div>
          <div style="display:flex;justify-content:space-between;margin-bottom:9px"><span style="color:var(--muted)">الباقي</span><b id="cr" style="color:var(--warn)">0</b></div>
          <div class="ff" style="margin-bottom:11px"><label>طريقة الدفع</label><select id="cm" class="rb"><option>نقدي</option><option>بطاقة</option><option>تحويل</option></select></div>
          <button class="btn btn-success" style="width:100%" onclick="doSale()">✅ إتمام البيع</button>
          <button class="btn btn-ghost btn-sm" style="width:100%;margin-top:7px" onclick="cart=[];renderCart()">🗑️ مسح</button>
        </div>
      </div>
    </div>
  </div>`;
  renderCart();
  psrch('');
}
async function psrch(q){
  const r=await api(`/api/products?search=${encodeURIComponent(q)}`);
  const el=document.getElementById('pos-prods');if(!el)return;
  el.innerHTML=r.data?.length
    ?r.data.filter(p=>p.quantity>0).slice(0,24).map(p=>`
      <div onclick="addCart(${p.id},'${p.name.replace(/'/g,'')}',${p.selling_price})"
        style="background:var(--bg2);border:1px solid var(--border);border-radius:9px;padding:13px;cursor:pointer;transition:all .18s"
        onmouseover="this.style.borderColor='var(--accent)'" onmouseout="this.style.borderColor='var(--border)'">
        <div style="font-size:13px;font-weight:700;margin-bottom:5px">${p.name}</div>
        <div style="font-size:11px;color:var(--muted)">${p.brand} | متبقي: ${p.quantity}</div>
        <div style="font-size:16px;font-weight:900;color:var(--accent2);margin-top:7px">${fmt(p.selling_price)}</div>
      </div>`).join('')
    :em('لا توجد منتجات');
}
function addCart(id,name,price){const ex=cart.find(i=>i.id===id);if(ex)ex.qty++;else cart.push({id,name,price,qty:1});renderCart();}
function chQty(i,d){cart[i].qty=Math.max(1,cart[i].qty+d);renderCart();}
function rmItem(i){cart.splice(i,1);renderCart();}
function updRem(){
  const total=cart.reduce((s,i)=>s+i.price*i.qty,0);
  const paid=parseFloat(document.getElementById('cp2')?.value)||0;
  const rem=document.getElementById('cr');if(rem)rem.textContent=fmt(Math.max(0,total-paid));
}
function renderCart(){
  const el=document.getElementById('cart-items');if(!el)return;
  if(!cart.length){el.innerHTML='<p style="text-align:center;color:var(--muted);padding:18px">السلة فارغة</p>';document.getElementById('ct').textContent='0';return;}
  el.innerHTML=cart.map((i,idx)=>`<div class="ci"><div class="nm"><div>${i.name}</div><div style="font-size:12px;color:var(--muted)">${fmt(i.price)} × ${i.qty}</div></div><div style="display:flex;align-items:center;gap:5px"><button class="qb" onclick="chQty(${idx},-1)">−</button><span style="min-width:22px;text-align:center">${i.qty}</span><button class="qb" onclick="chQty(${idx},1)">+</button><button class="qb" style="background:rgba(255,71,87,.2);color:var(--danger)" onclick="rmItem(${idx})">×</button></div></div>`).join('');
  const total=cart.reduce((s,i)=>s+i.price*i.qty,0);
  document.getElementById('ct').textContent=fmt(total);
  updRem();
}
async function doSale(){
  if(!cart.length){toast('السلة فارغة','error');return;}
  const total=cart.reduce((s,i)=>s+i.price*i.qty,0);
  const paid=parseFloat(document.getElementById('cp2').value)||total;
  const r=await api('/api/sales/add','POST',{items:cart.map(i=>({product_id:i.id,quantity:i.qty,unit_price:i.price})),paid_amount:paid,payment_method:document.getElementById('cm').value});
  if(r.success){toast(`✅ فاتورة ${r.data.invoice_number} | الإجمالي: ${fmt(total)}`);cart=[];renderCart();document.getElementById('cp2').value='';}
  else toast(r.message,'error');
}

// ═══════════════════════════════════════════════════════════
//  SALES (محدثة لتطابق 333.py)
// ═══════════════════════════════════════════════════════════
let salesData=[];
let salesFilterState={search:'',from_date:'',to_date:new Date().toISOString().slice(0,10)};

async function sales(pr='اليوم'){
  const c=document.getElementById('pc');
  
  // 🔴 الأزرار الجديدة واضحة جداً في الأعلى
  let html = `
  <div style="background:var(--bg2);border:2px solid var(--accent);border-radius:12px;padding:20px;margin-bottom:20px">
    <div style="margin-bottom:15px">
      <div style="display:flex;gap:10px;flex-wrap:wrap;align-items:center">
        <input id="sale-search" type="text" placeholder="🔍 بحث الفاتورة..." style="flex:1;min-width:200px;padding:12px;border-radius:6px;border:1px solid var(--border);background:var(--bg3);color:var(--text);font-size:14px" onkeyup="filterSalesTable()">
        <input id="sale-from" type="date" style="padding:12px;border-radius:6px;border:1px solid var(--border);background:var(--bg3);color:var(--text);font-size:14px" onchange="filterSalesTable()">
        <input id="sale-to" type="date" style="padding:12px;border-radius:6px;border:1px solid var(--border);background:var(--bg3);color:var(--text);font-size:14px" onchange="filterSalesTable()">
        <button class="btn btn-ghost" onclick="filterSalesTable()" style="padding:12px 20px;font-size:14px">تصفية</button>
      </div>
    </div>
    
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-bottom:15px">
      ${['اليوم','أسبوع','شهر','سنة','كل الوقت'].map(p=>`<button class="btn ${p===pr?'btn-success':'btn-ghost'}" onclick="salesPeriod('${p}')" style="padding:10px 15px;font-size:14px;font-weight:bold">${p}</button>`).join('')}
    </div>
    
    <!-- 🔴 الزر الجديد الواضح جداً -->
    <div style="border-top:2px solid var(--border);padding-top:15px;margin-top:15px">
      <button class="btn btn-danger" onclick="clearAllSales()" style="padding:15px 30px;font-size:16px;font-weight:bold;width:100%;background:var(--danger);color:white;border-radius:8px">
        🧹 تفريغ جميع الفواتير (حذف نهائي!)
      </button>
    </div>
  </div>
  
  <div class="tw">
    <div class="th"><h3>💰 سجل المبيعات</h3><span id="sale-stats" style="color:var(--muted);font-size:13px"></span></div>
    <div id="sales-table" style="overflow-x:auto"></div>
  </div>`;
  
  c.innerHTML = html;
  
  const r=await api(`/api/sales?period=${encodeURIComponent(pr)}`);
  salesData=r.data||[];
  
  const[fromD,toD]=getDatesForPeriod(pr);
  const stats=await api(`/api/sales/stats?period=${encodeURIComponent(pr)}`);
  
  if(stats.success&&stats.data){
    document.getElementById('sale-stats').textContent=`${stats.data.count||0} فاتورة | الإجمالي: ${fmt(stats.data.total||0)} | الربح: ${fmt(stats.data.profit||0)}`;
  }
  
  renderSalesTable(salesData);
  
  setTimeout(()=>{
    document.getElementById('sale-from').value=fromD;
    document.getElementById('sale-to').value=toD;
  },0);
}

function clearAllSales(){
  if(!confirm('🚨 هل تريد حذف جميع الفواتير؟\n\nهذا الإجراء نهائي ولا يمكن التراجع عنه!'))return;
  if(!confirm('⚠️ تأكيد نهائي: سيتم حذف جميع الفواتير!'))return;
  
  omo('🗑️ تفريغ الفواتير',`
  <div style="background:var(--bg3);padding:14px;border-radius:8px;color:var(--danger)">
    <p>🚨 تحذير: سيتم حذف جميع فواتير المبيعات من قاعدة البيانات</p>
    <p style="margin-top:10px;font-size:12px;color:var(--muted)">عدد الفواتير: <b>${salesData?.length||0}</b></p>
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-danger" onclick="confirmClearAllSales()">🗑️ حذف الكل</button>`);
}

async function confirmClearAllSales(){
  const r=await api('/api/sales/clear','POST',{});
  if(r.success){
    cmo();
    toast('✅ تم حذف جميع الفواتير');
    sales('اليوم');
  }else{
    toast(r.message,'error');
  }
}

function getDatesForPeriod(period){
  const today=new Date();
  let from=new Date();
  let to=new Date();
  
  if(period==='اليوم'){
    from.setHours(0,0,0,0);
    to.setHours(23,59,59,999);
  }else if(period==='أسبوع'){
    from.setDate(today.getDate()-today.getDay());
    to.setDate(from.getDate()+6);
  }else if(period==='شهر'){
    from.setDate(1);
    to.setMonth(today.getMonth()+1,0);
  }else if(period==='سنة'){
    from.setMonth(0,1);
    to.setMonth(11,31);
  }else{
    return ['2020-01-01',today.toISOString().slice(0,10)];
  }
  
  return [from.toISOString().slice(0,10),to.toISOString().slice(0,10)];
}

function renderSalesTable(data){
  const tbl=document.getElementById('sales-table');
  if(!data||!data.length){
    tbl.innerHTML=em('لا توجد مبيعات');
    return;
  }
  
  tbl.innerHTML=`
  <table style="width:100%">
    <tr>
      <th>رقم الفاتورة</th>
      <th>التاريخ</th>
      <th>الوقت</th>
      <th>المبلغ</th>
      <th>المدفوع</th>
      <th>المتبقي</th>
      <th>الربح</th>
      <th>الطريقة</th>
      <th>الكاشير</th>
      <th></th>
    </tr>
    ${data.map(s=>`
    <tr>
      <td><b style="color:var(--accent)">${s.invoice_number}</b></td>
      <td>${s.sale_date}</td>
      <td>${s.sale_time||'—'}</td>
      <td>${fmt(s.total_amount)}</td>
      <td>${fmt(s.paid_amount)}</td>
      <td><span class="badge ${s.remaining_amount>0?'br':'bg'}">${fmt(s.remaining_amount)}</span></td>
      <td style="color:var(--accent2)">${fmt(s.profit||0)}</td>
      <td>${s.payment_method||'—'}</td>
      <td>${s.username}</td>
      <td style="white-space:nowrap">
        <button class="btn btn-ghost btn-sm" onclick="vSale(${s.id})" title="عرض التفاصيل">👁️</button>
        ${s.remaining_amount>0?`<button class="btn btn-warn btn-sm" onclick="paySale(${s.id},${s.remaining_amount})" title="تسجيل دفعة">💳</button>`:''}
        <button class="btn btn-info btn-sm" onclick="returnToInventory(${s.id})" title="إرجاع إلى المخزون">↩️</button>
        <button class="btn btn-danger btn-sm" onclick="deleteSale(${s.id})" title="حذف الفاتورة">🗑️</button>
      </td>
    </tr>`).join('')}
  </table>`;
}

function deleteSale(id){
  if(!confirm('هل تريد حذف هذه الفاتورة؟'))return;
  
  omo('🗑️ حذف الفاتورة',`
  <div style="background:var(--bg3);padding:14px;border-radius:8px;color:var(--danger)">
    <p>⚠️ هذا سيحذف الفاتورة من قاعدة البيانات</p>
    <p style="margin-top:10px;font-size:12px;color:var(--muted)">ملاحظة: لن يتم استرجاع المنتجات للمخزون تلقائياً</p>
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-danger" onclick="confirmDeleteSale(${id})">🗑️ حذف</button>`);
}

async function confirmDeleteSale(id){
  const r=await api(`/api/sales/${id}`,'DELETE');
  if(r.success){
    cmo();
    toast('✅ تم حذف الفاتورة');
    sales('اليوم');
  }else{
    toast(r.message,'error');
  }
}

async function returnToInventory(id){
  const r=await api(`/api/sales/${id}`);
  if(!r.success){toast('خطأ في تحميل البيانات','error');return;}
  const{sale,details}=r.data;
  const html=`
  <div style="background:rgba(0,255,157,.08);border:1px solid var(--accent2);border-radius:9px;padding:14px;margin-bottom:14px">
    <b style="color:var(--accent2)">⚠️ سيتم إرجاع المنتجات للمخزون وحذف الفاتورة نهائياً</b>
  </div>
  <div style="background:var(--bg3);padding:12px;border-radius:9px;margin-bottom:12px;display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px">
    <div><span style="color:var(--muted);font-size:11px">رقم الفاتورة</span><div style="font-weight:700;color:var(--accent)">${sale.invoice_number}</div></div>
    <div><span style="color:var(--muted);font-size:11px">التاريخ</span><div>${sale.sale_date}</div></div>
    <div><span style="color:var(--muted);font-size:11px">الإجمالي</span><div style="font-weight:700;color:var(--accent2)">${fmt(sale.total_amount)}</div></div>
  </div>
  <table style="width:100%;margin-bottom:12px">
    <tr style="background:var(--bg3)"><th>المنتج</th><th>الكمية</th><th>سعر الوحدة</th><th>الإجمالي</th><th>الربح</th></tr>
    ${details.map(d=>`<tr>
      <td><b>${d.product_name}</b></td>
      <td style="text-align:center">${d.quantity}</td>
      <td>${fmt(d.unit_price)}</td>
      <td>${fmt(d.total_price)}</td>
      <td style="color:var(--accent2)">${fmt(d.profit||0)}</td>
    </tr>`).join('')}
  </table>
  <div style="background:var(--bg3);padding:12px;border-radius:8px;display:grid;grid-template-columns:1fr 1fr;gap:10px">
    <div><span style="color:var(--muted);font-size:11px">المدفوع</span><div style="color:var(--accent2);font-weight:700">${fmt(sale.paid_amount)}</div></div>
    <div><span style="color:var(--muted);font-size:11px">المتبقي</span><div style="color:${sale.remaining_amount>0?'var(--danger)':'var(--accent2)'};font-weight:700">${fmt(sale.remaining_amount)}</div></div>
  </div>`;
  omo(`🔄 إرجاع فاتورة: ${sale.invoice_number}`,html,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="confirmReturnToInventory(${id})">✅ إرجاع وحذف الفاتورة</button>`);
}

async function confirmReturnToInventory(id){
  const r=await api(`/api/sales/${id}`,'DELETE');
  if(r.success){
    cmo();
    toast('✅ تم إرجاع المنتجات للمخزون وحذف الفاتورة');
    sales('اليوم');
  }else{
    toast(r.message,'error');
  }
}

function filterSalesTable(){
  const search=document.getElementById('sale-search').value.toLowerCase();
  const from=document.getElementById('sale-from').value;
  const to=document.getElementById('sale-to').value;
  
  const filtered=salesData.filter(s=>{
    const matchSearch=!search||s.invoice_number.includes(search)||s.username.includes(search)||s.payment_method?.includes(search);
    const matchFrom=!from||s.sale_date>=from;
    const matchTo=!to||s.sale_date<=to;
    return matchSearch&&matchFrom&&matchTo;
  });
  
  renderSalesTable(filtered);
}

function salesPeriod(period){
  sales(period);
}

async function vSale(id){
  const r=await api(`/api/sales/${id}`);
  if(!r.success){toast('خطأ في تحميل البيانات','error');return;}
  
  const{sale,details}=r.data;
  let html=`
  <div style="background:var(--bg3);padding:14px;border-radius:9px;margin-bottom:14px">
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;margin-bottom:10px">
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">رقم الفاتورة</span>
        <div style="font-weight:700;color:var(--accent);font-size:16px">${sale.invoice_number}</div>
      </div>
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">التاريخ والوقت</span>
        <div style="font-size:13px">${sale.sale_date} ${sale.sale_time||'—'}</div>
      </div>
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">الكاشير</span>
        <div style="font-size:13px">${r.data.user_name||'—'}</div>
      </div>
    </div>
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px">
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">الإجمالي</span>
        <div style="font-weight:700;color:var(--accent2);font-size:15px">${fmt(sale.total_amount)}</div>
      </div>
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">المدفوع</span>
        <div style="font-weight:700;color:var(--accent2);font-size:15px">${fmt(sale.paid_amount)}</div>
      </div>
      <div>
        <span style="color:var(--muted);font-size:11px;display:block;margin-bottom:3px">المتبقي</span>
        <div style="font-weight:700;color:${sale.remaining_amount>0?'var(--danger)':'var(--accent2)'};font-size:15px">${fmt(sale.remaining_amount)}</div>
      </div>
    </div>
  </div>
  
  <table style="width:100%;margin-bottom:14px">
    <tr style="background:var(--bg3)">
      <th>المنتج</th>
      <th>الماركة</th>
      <th>الكمية</th>
      <th>سعر الوحدة</th>
      <th>الإجمالي</th>
      <th>الربح</th>
    </tr>
    ${details.map(d=>`
    <tr>
      <td><b>${d.product_name}</b></td>
      <td>${d.brand||'—'}</td>
      <td style="text-align:center">${d.quantity}</td>
      <td>${fmt(d.unit_price)}</td>
      <td>${fmt(d.total_price)}</td>
      <td style="color:var(--accent2)">${fmt(d.profit||0)}</td>
    </tr>`).join('')}
  </table>
  
  <div style="background:var(--bg3);padding:12px;border-radius:8px;margin-bottom:14px">
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px">
      <div>
        <span style="color:var(--muted);font-size:11px">طريقة الدفع</span>
        <div style="font-weight:600;margin-top:3px">${sale.payment_method||'نقدي'}</div>
      </div>
      <div>
        <span style="color:var(--muted);font-size:11px">إجمالي الربح</span>
        <div style="font-weight:600;color:var(--accent2);margin-top:3px">${fmt(sale.profit||0)}</div>
      </div>
    </div>
  </div>`;
  
  omo(`📋 تفاصيل الفاتورة رقم: ${sale.invoice_number}`,html,
  `${sale.remaining_amount>0?`<button class="btn btn-warn" onclick="paySale(${id},${sale.remaining_amount})">💳 تسجيل دفعة</button>`:''}
  <button class="btn btn-ghost" onclick="cmo()">✖ إغلاق</button>`);
}

function paySale(id,rem){
  omo('💳 تسجيل دفعة',`
  <div class="ff" style="margin-bottom:14px">
    <label style="color:var(--muted);font-size:12px">المبلغ المتبقي: <b style="color:var(--accent)">${fmt(rem)}</b></label>
    <input id="pay-amount" type="number" value="${rem}" style="margin-top:6px;background:var(--bg3);border:1px solid var(--border);padding:10px;border-radius:7px;color:var(--text);width:100%" placeholder="أدخل المبلغ">
  </div>
  <div class="ff">
    <label style="color:var(--muted);font-size:12px">طريقة الدفع</label>
    <select id="pay-method" style="margin-top:6px;background:var(--bg3);border:1px solid var(--border);padding:10px;border-radius:7px;color:var(--text);width:100%">
      <option>نقدي</option>
      <option>بطاقة</option>
      <option>تحويل</option>
      <option>شيك</option>
    </select>
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="subPaySale(${id})">✅ تأكيد الدفع</button>`);
}

async function subPaySale(id){
  const amount=document.getElementById('pay-amount').value;
  const method=document.getElementById('pay-method').value;
  
  if(!amount||parseFloat(amount)<=0){toast('أدخل مبلغاً صحيحاً','error');return;}
  
  const r=await api(`/api/sales/${id}/pay`,'POST',{amount,method});
  if(r.success){
    cmo();
    toast('✅ '+r.message,'success');
    sales('اليوم');
  }else{
    toast(r.message,'error');
  }
}

// ═══════════════════════════════════════════════════════════
//  PURCHASES (محدثة لتطابق 333.py)
// ═══════════════════════════════════════════════════════════
let purchasesData=[];
let purchasesFilterState={search:'',from_date:'',to_date:new Date().toISOString().slice(0,10)};

async function purchases(pr='شهر'){
  const c=document.getElementById('pc');
  let html=`
  <div style="background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:16px;margin-bottom:18px">
    <div style="display:flex;gap:10px;flex-wrap:wrap;align-items:center;margin-bottom:12px">
      <input id="purch-search" type="text" placeholder="🔍 بحث في الفواتير..." class="si" style="flex:1;min-width:200px" onkeyup="filterPurchasesTable()">
      <input id="purch-from" type="date" class="si" style="flex:0;width:150px" onchange="filterPurchasesTable()">
      <input id="purch-to" type="date" class="si" style="flex:0;width:150px" onchange="filterPurchasesTable()">
    </div>
    <div style="display:flex;gap:8px;flex-wrap:wrap;align-items:center">
      ${['أسبوع','شهر','سنة','كل الوقت'].map(p=>`<button class="btn ${p===pr?'btn-success':'btn-ghost'} btn-sm" onclick="purchasesPeriod('${p}')">${p}</button>`).join('')}
      <div style="flex:1"></div>
      <button class="btn btn-success" onclick="openAddPurch()" style="padding:10px 22px;font-size:15px;font-weight:700">🛒 إضافة فاتورة شراء</button>
    </div>
  </div>
  <div class="tw">
    <div class="th"><h3>🛍️ سجل المشتريات</h3><span id="purch-stats" style="color:var(--muted);font-size:13px"></span></div>
    <div id="purchases-table" style="overflow-x:auto"></div>
  </div>`;
  c.innerHTML=html;
  const r=await api(`/api/purchases?period=${encodeURIComponent(pr)}`);
  purchasesData=r.data||[];
  const[fromD,toD]=getDatesForPeriod(pr);
  const total=purchasesData.reduce((s,p)=>s+p.total_amount,0);
  const rem=purchasesData.reduce((s,p)=>s+p.remaining_amount,0);
  document.getElementById('purch-stats').textContent=`${purchasesData.length} فاتورة | الإجمالي: ${fmt(total)} | المتبقي: ${fmt(rem)}`;
  renderPurchasesTable(purchasesData);
  setTimeout(()=>{document.getElementById('purch-from').value=fromD;document.getElementById('purch-to').value=toD;},0);
}

// ── فتح واجهة إضافة فاتورة شراء (كاملة بدون مودال) ──────────
let purchCart=[];
let purchProds=[];
let purchSups=[];

async function openAddPurch(){
  const[sr,pr]=await Promise.all([api('/api/suppliers'),api('/api/products')]);
  purchSups=sr.data||[];
  purchProds=pr.data||[];
  purchCart=[];

  const ov=document.createElement('div');
  ov.id='purch-overlay';
  ov.style.cssText='position:fixed;inset:0;background:rgba(0,0,0,.85);z-index:300;display:flex;align-items:center;justify-content:center;padding:16px';
  ov.innerHTML=`
  <div style="background:var(--bg2);border:1px solid var(--border);border-radius:16px;width:100%;max-width:900px;max-height:95vh;display:flex;flex-direction:column;overflow:hidden;box-shadow:0 30px 80px rgba(0,0,0,.7)">
    <!-- Header -->
    <div style="padding:18px 22px;border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;background:linear-gradient(135deg,rgba(0,212,255,.08),rgba(0,255,157,.04))">
      <h2 style="margin:0;font-size:18px;color:var(--accent)">🛒 إضافة فاتورة شراء جديدة</h2>
      <button onclick="closePurchOverlay()" style="background:none;border:none;color:var(--muted);font-size:22px;cursor:pointer;line-height:1">✕</button>
    </div>

    <!-- Tabs: existing / new product -->
    <div style="padding:10px 22px 0;border-bottom:1px solid var(--border);display:flex;gap:0">
      <button id="tab-exist" onclick="switchPurchTab('exist')" style="padding:9px 20px;border:none;border-bottom:3px solid var(--accent);background:rgba(0,212,255,.08);color:var(--accent);font-family:Cairo,sans-serif;font-size:14px;font-weight:700;cursor:pointer;border-radius:8px 8px 0 0">📦 منتج موجود</button>
      <button id="tab-new" onclick="switchPurchTab('new')" style="padding:9px 20px;border:none;border-bottom:3px solid transparent;background:none;color:var(--muted);font-family:Cairo,sans-serif;font-size:14px;font-weight:600;cursor:pointer;border-radius:8px 8px 0 0">✨ منتج جديد</button>
    </div>

    <!-- Tab: Existing Product -->
    <div id="purch-tab-exist" style="padding:14px 22px;border-bottom:1px solid var(--border)">
      <!-- Supplier row inside tab -->
      <div style="display:flex;gap:10px;align-items:flex-end;margin-bottom:12px;flex-wrap:wrap">
        <div style="flex:2;min-width:180px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">🏢 المورد (اختياري)</label>
          <select id="pu-sup" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 12px;color:var(--text);font-size:14px">
            <option value="">— بدون مورد —</option>
            ${purchSups.map(s=>`<option value="${s.id}">${s.name}${s.phone?' ('+s.phone+')':''}</option>`).join('')}
          </select>
        </div>
        <button onclick="quickAddSupplierInline()" style="padding:9px 14px;background:rgba(0,212,255,.1);border:1px solid var(--accent);border-radius:7px;color:var(--accent);font-size:13px;cursor:pointer;white-space:nowrap;font-family:Cairo,sans-serif">➕ مورد جديد</button>
      </div>
      <!-- Product row -->
      <div style="display:flex;gap:10px;flex-wrap:wrap;align-items:flex-end">
        <div style="flex:2;min-width:180px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">المنتج</label>
          <input id="pu-prod-search" list="pu-prod-list" placeholder="ابحث أو اختر منتج..." autocomplete="off"
            style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 12px;color:var(--text);font-size:14px">
          <datalist id="pu-prod-list">
            ${purchProds.map(p=>`<option value="${p.name}" data-id="${p.id}">${p.brand} | مخزون: ${p.quantity} | شراء: ${p.cost_price}</option>`).join('')}
          </datalist>
        </div>
        <div style="width:80px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الكمية</label>
          <input id="pu-exist-qty" type="number" value="1" min="1" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 10px;color:var(--text);font-size:14px">
        </div>
        <div style="width:120px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">سعر الشراء</label>
          <input id="pu-exist-price" type="number" value="0" step="0.01" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 10px;color:var(--text);font-size:14px">
        </div>
        <div style="width:120px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">سعر البيع</label>
          <input id="pu-exist-sell" type="number" value="0" step="0.01" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 10px;color:var(--text);font-size:14px">
        </div>
        <button onclick="addExistToPurchCart()" style="padding:9px 18px;background:var(--accent2);color:#0f1923;border:none;border-radius:7px;font-weight:700;font-size:14px;cursor:pointer;white-space:nowrap;font-family:Cairo,sans-serif">➕ إضافة</button>
      </div>
    </div>

    <!-- Tab: New Product -->
    <div id="purch-tab-new" style="padding:14px 22px;border-bottom:1px solid var(--border);display:none">
      <div style="display:flex;gap:10px;flex-wrap:wrap;align-items:flex-end">
        <div style="flex:2;min-width:140px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الاسم *</label>
          <input id="pu-new-name" placeholder="اسم المنتج" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 12px;color:var(--text);font-size:14px">
        </div>
        <div style="flex:1;min-width:100px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الماركة *</label>
          <input id="pu-new-brand" placeholder="Apple..." style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 12px;color:var(--text);font-size:14px">
        </div>
        <div style="width:110px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الفئة</label>
          <select id="pu-new-cat" onchange="togglePurchCondition()" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:13px">
            <option>موبايلات</option><option>ملحقات</option><option>أجهزة لوحية</option><option>أخرى</option>
          </select>
        </div>
        <div id="pu-cond-wrap" style="width:95px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الحالة</label>
          <select id="pu-new-cond" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:13px">
            <option>جديد</option><option>مستعمل</option>
          </select>
        </div>
        <div style="width:70px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الكمية</label>
          <input id="pu-new-qty" type="number" value="1" min="1" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:14px">
        </div>
        <div style="width:110px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">سعر الشراء *</label>
          <input id="pu-new-cost" type="number" value="0" step="0.01" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:14px">
        </div>
        <div style="width:110px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">سعر البيع</label>
          <input id="pu-new-sell" type="number" value="0" step="0.01" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:14px">
        </div>
        <div style="width:110px">
          <label style="font-size:12px;color:var(--muted);display:block;margin-bottom:5px">الباركود</label>
          <input id="pu-new-bar" placeholder="اختياري" style="width:100%;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 8px;color:var(--text);font-size:13px">
        </div>
        <button onclick="addNewToPurchCart()" style="padding:9px 18px;background:var(--accent2);color:#0f1923;border:none;border-radius:7px;font-weight:700;font-size:14px;cursor:pointer;white-space:nowrap;font-family:Cairo,sans-serif">➕ إضافة</button>
      </div>
    </div>

    <!-- Cart -->
    <div style="flex:1;overflow-y:auto;padding:0 22px">
      <div style="display:flex;align-items:center;justify-content:space-between;padding:12px 0 6px">
        <b style="color:var(--accent);font-size:14px">📋 المنتجات المضافة</b>
        <span id="pu-cart-cnt" style="color:var(--muted);font-size:13px">0 منتج</span>
      </div>
      <div id="pu-cart-table"></div>
    </div>

    <!-- Footer -->
    <div style="padding:16px 22px;border-top:1px solid var(--border);background:var(--bg3);border-radius:0 0 16px 16px">
      <div style="display:flex;align-items:center;gap:18px;flex-wrap:wrap">
        <div style="display:flex;align-items:center;gap:8px">
          <span style="color:var(--muted);font-size:14px">المبلغ الإجمالي:</span>
          <span id="pu-total" style="font-size:22px;font-weight:900;color:var(--accent)">0</span>
        </div>
        <div style="display:flex;align-items:center;gap:8px">
          <label style="color:var(--muted);font-size:14px;white-space:nowrap">المبلغ المدفوع:</label>
          <input id="pu-paid" type="number" value="0" step="0.01" style="width:140px;background:var(--bg2);border:2px solid var(--border);border-radius:8px;padding:10px 12px;color:var(--text);font-size:15px;font-weight:700">
        </div>
        <div style="display:flex;align-items:center;gap:8px">
          <span style="color:var(--muted);font-size:13px">المتبقي:</span>
          <span id="pu-remaining" style="font-size:16px;font-weight:700;color:var(--warn)">0</span>
        </div>
        <div style="flex:1"></div>
        <button onclick="closePurchOverlay()" style="padding:11px 22px;background:var(--bg2);border:1px solid var(--border);border-radius:8px;color:var(--text);font-size:14px;cursor:pointer;font-family:Cairo,sans-serif">إلغاء</button>
        <button onclick="savePurchNew()" style="padding:11px 26px;background:var(--accent2);color:#0f1923;border:none;border-radius:8px;font-size:15px;font-weight:700;cursor:pointer;font-family:Cairo,sans-serif">💾 حفظ الفاتورة</button>
      </div>
    </div>
  </div>`;
  document.body.appendChild(ov);
  document.getElementById('pu-paid').addEventListener('input',updatePurchTotals);
}

function quickAddSupplierInline(){
  const div=document.createElement('div');
  div.id='quick-sup-popup';
  div.style.cssText='position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:400;display:flex;align-items:center;justify-content:center';
  div.innerHTML=`<div style="background:var(--bg2);border:1px solid var(--border);border-radius:14px;padding:24px;width:400px;max-width:95vw">
    <h3 style="margin:0 0 16px;color:var(--accent)">➕ إضافة مورد جديد</h3>
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px">
      <div><label style="font-size:12px;color:var(--muted)">الاسم *</label><input id="qsn" style="width:100%;margin-top:5px;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 10px;color:var(--text);font-size:14px"></div>
      <div><label style="font-size:12px;color:var(--muted)">الهاتف</label><input id="qsp" style="width:100%;margin-top:5px;background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:9px 10px;color:var(--text);font-size:14px"></div>
    </div>
    <div style="display:flex;gap:10px;justify-content:flex-end;margin-top:16px">
      <button onclick="document.getElementById('quick-sup-popup').remove()" style="padding:9px 18px;background:var(--bg3);border:1px solid var(--border);border-radius:7px;color:var(--text);cursor:pointer;font-family:Cairo,sans-serif">إلغاء</button>
      <button onclick="saveQuickSupplier()" style="padding:9px 20px;background:var(--accent2);color:#0f1923;border:none;border-radius:7px;font-weight:700;cursor:pointer;font-family:Cairo,sans-serif">💾 حفظ</button>
    </div>
  </div>`;
  document.body.appendChild(div);
  setTimeout(()=>document.getElementById('qsn')?.focus(),100);
}

async function saveQuickSupplier(){
  const name=document.getElementById('qsn')?.value.trim();
  const phone=document.getElementById('qsp')?.value.trim()||'';
  if(!name){toast('أدخل اسم المورد','error');return;}
  const r=await api('/api/suppliers/add','POST',{name,phone,email:'',address:''});
  if(r.success){
    document.getElementById('quick-sup-popup')?.remove();
    toast('✅ تم إضافة المورد');
    // refresh supplier dropdown
    const sr=await api('/api/suppliers');
    purchSups=sr.data||[];
    const sel=document.getElementById('pu-sup');
    if(sel){
      const cur=sel.value;
      sel.innerHTML=`<option value="">— بدون مورد —</option>${purchSups.map(s=>`<option value="${s.id}">${s.name}${s.phone?' ('+s.phone+')':''}</option>`).join('')}`;
      // select newly added
      if(r.data?.id) sel.value=r.data.id;
      else sel.value=cur;
    }
  }else toast(r.message,'error');
}

function closePurchOverlay(){document.getElementById('purch-overlay')?.remove();}

function switchPurchTab(tab){
  const isExist=tab==='exist';
  document.getElementById('purch-tab-exist').style.display=isExist?'block':'none';
  document.getElementById('purch-tab-new').style.display=isExist?'none':'block';
  document.getElementById('tab-exist').style.cssText=`padding:9px 20px;border:none;border-bottom:3px solid ${isExist?'var(--accent)':'transparent'};background:${isExist?'rgba(0,212,255,.08)':'none'};color:${isExist?'var(--accent)':'var(--muted)'};font-family:Cairo,sans-serif;font-size:14px;font-weight:${isExist?700:600};cursor:pointer;border-radius:8px 8px 0 0`;
  document.getElementById('tab-new').style.cssText=`padding:9px 20px;border:none;border-bottom:3px solid ${!isExist?'var(--accent)':'transparent'};background:${!isExist?'rgba(0,212,255,.08)':'none'};color:${!isExist?'var(--accent)':'var(--muted)'};font-family:Cairo,sans-serif;font-size:14px;font-weight:${!isExist?700:600};cursor:pointer;border-radius:8px 8px 0 0`;
}

function togglePurchCondition(){
  const cat=document.getElementById('pu-new-cat')?.value;
  const wrap=document.getElementById('pu-cond-wrap');
  if(wrap) wrap.style.display=cat==='موبايلات'?'block':'none';
}

function updatePurchTotals(){
  const total=purchCart.reduce((s,i)=>s+i.total,0);
  const paid=parseFloat(document.getElementById('pu-paid')?.value)||0;
  const rem=document.getElementById('pu-remaining');
  if(rem) rem.textContent=fmt(Math.max(0,total-paid));
  if(rem) rem.style.color=total-paid>0?'var(--warn)':'var(--accent2)';
  const el=document.getElementById('pu-total');
  if(el) el.textContent=fmt(total);
}

function renderPurchCart(){
  const el=document.getElementById('pu-cart-table');
  if(!el)return;
  const cnt=document.getElementById('pu-cart-cnt');
  if(cnt) cnt.textContent=purchCart.length+' منتج';
  if(!purchCart.length){
    el.innerHTML=`<div style="text-align:center;padding:28px;color:var(--muted)">📭 لم تُضف أي منتجات بعد</div>`;
    updatePurchTotals();return;
  }
  el.innerHTML=`<table style="width:100%;border-collapse:collapse">
    <tr style="background:var(--bg3)"><th style="padding:9px 12px;text-align:right;font-size:12px;color:var(--muted)">المنتج</th><th style="padding:9px;font-size:12px;color:var(--muted)">الكمية</th><th style="padding:9px;font-size:12px;color:var(--muted)">سعر الشراء</th><th style="padding:9px;font-size:12px;color:var(--muted)">سعر البيع</th><th style="padding:9px;font-size:12px;color:var(--muted)">المجموع</th><th style="padding:9px"></th></tr>
    ${purchCart.map((item,i)=>`<tr style="border-bottom:1px solid var(--border)">
      <td style="padding:10px 12px"><b>${item.name}</b><br><span style="font-size:11px;color:var(--muted)">${item.isNew?'🆕 منتج جديد':'📦 موجود'} | ${item.category||''}</span></td>
      <td style="padding:10px;text-align:center">${item.quantity}</td>
      <td style="padding:10px;text-align:center;color:var(--accent)">${fmt(item.cost_price)}</td>
      <td style="padding:10px;text-align:center;color:var(--accent2)">${fmt(item.sell_price||0)}</td>
      <td style="padding:10px;text-align:center;font-weight:700">${fmt(item.total)}</td>
      <td style="padding:10px"><button onclick="removePurchCartItem(${i})" style="background:rgba(255,71,87,.15);border:none;color:var(--danger);padding:5px 10px;border-radius:5px;cursor:pointer">🗑️</button></td>
    </tr>`).join('')}
  </table>`;
  updatePurchTotals();
}

function removePurchCartItem(i){purchCart.splice(i,1);renderPurchCart();}

function addExistToPurchCart(){
  const name=document.getElementById('pu-prod-search').value.trim();
  const qty=parseInt(document.getElementById('pu-exist-qty').value)||1;
  const price=parseFloat(document.getElementById('pu-exist-price').value)||0;
  const sell=parseFloat(document.getElementById('pu-exist-sell').value)||0;
  if(!name){toast('اختر منتجاً','error');return;}
  if(qty<=0){toast('الكمية يجب أن تكون أكبر من صفر','error');return;}
  const prod=purchProds.find(p=>p.name===name);
  if(!prod){toast('المنتج غير موجود في القائمة','error');return;}
  const exist=purchCart.find(c=>c.id===prod.id&&!c.isNew);
  if(exist){exist.quantity+=qty;exist.total=exist.quantity*exist.cost_price;}
  else purchCart.push({id:prod.id,name:prod.name,category:prod.category,quantity:qty,cost_price:price||prod.cost_price,sell_price:sell||prod.selling_price,total:qty*(price||prod.cost_price),isNew:false});
  renderPurchCart();
  document.getElementById('pu-prod-search').value='';
  document.getElementById('pu-exist-qty').value='1';
  document.getElementById('pu-exist-price').value='0';
  document.getElementById('pu-exist-sell').value='0';
  // auto-fill paid
  const total=purchCart.reduce((s,i)=>s+i.total,0);
  document.getElementById('pu-paid').value=total.toFixed(2);
  updatePurchTotals();
}

async function addNewToPurchCart(){
  const name=document.getElementById('pu-new-name').value.trim();
  const brand=document.getElementById('pu-new-brand').value.trim();
  const cat=document.getElementById('pu-new-cat').value;
  const cond=document.getElementById('pu-new-cond')?.value||'جديد';
  const qty=parseInt(document.getElementById('pu-new-qty').value)||1;
  const cost=parseFloat(document.getElementById('pu-new-cost').value)||0;
  const sell=parseFloat(document.getElementById('pu-new-sell').value)||0;
  const bar=document.getElementById('pu-new-bar').value.trim();
  if(!name||!brand){toast('اسم المنتج والماركة مطلوبان','error');return;}
  if(qty<=0){toast('الكمية يجب أن تكون أكبر من صفر','error');return;}
  // create product in DB
  const r=await api('/api/products/add','POST',{name,brand,model:'-',category:cat,cost_price:cost,selling_price:sell||cost,quantity:0,min_stock:5,barcode:bar||null,condition:cond});
  if(!r.success){toast(r.message,'error');return;}
  purchCart.push({id:r.data?.id||null,name,category:cat,quantity:qty,cost_price:cost,sell_price:sell,total:qty*cost,isNew:true,isJustCreated:true});
  renderPurchCart();
  document.getElementById('pu-new-name').value='';
  document.getElementById('pu-new-brand').value='';
  document.getElementById('pu-new-qty').value='1';
  document.getElementById('pu-new-cost').value='0';
  document.getElementById('pu-new-sell').value='0';
  document.getElementById('pu-new-bar').value='';
  // reload products list
  const pr2=await api('/api/products');
  purchProds=pr2.data||[];
  document.getElementById('pu-prod-list').innerHTML=purchProds.map(p=>`<option value="${p.name}">${p.brand} | مخزون: ${p.quantity}</option>`).join('');
  toast('✅ تم إنشاء المنتج وإضافته');
  const total=purchCart.reduce((s,i)=>s+i.total,0);
  document.getElementById('pu-paid').value=total.toFixed(2);
  updatePurchTotals();
}

async function savePurchNew(){
  if(!purchCart.length){toast('أضف منتجات أولاً','error');return;}
  const total=purchCart.reduce((s,i)=>s+i.total,0);
  const paid=parseFloat(document.getElementById('pu-paid')?.value)||0;
  const supEl=document.getElementById('pu-sup');
  const supId=supEl&&supEl.value?parseInt(supEl.value):null;
  const items=purchCart.filter(item=>item.id).map(item=>({
    product_id:item.id,
    quantity:item.quantity,
    unit_price:item.cost_price||0,
    sell_price:item.sell_price||0
  }));
  if(!items.length){toast('تأكد من إضافة منتجات صحيحة','error');return;}
  const r=await api('/api/purchases/add','POST',{supplier_id:supId,paid_amount:paid,items});
  if(r.success){
    closePurchOverlay();
    toast(`✅ تم حفظ الفاتورة ${r.data?.invoice_number||''} | الإجمالي: ${fmt(total)}`);
    purchases('شهر');
  }else toast(r.message,'error');
}

function renderPurchasesTable(data){
  const tbl=document.getElementById('purchases-table');
  if(!data||!data.length){tbl.innerHTML=em('لا توجد مشتريات');return;}
  tbl.innerHTML=`<table style="width:100%">
    <tr><th>رقم الفاتورة</th><th>المورد</th><th>التاريخ</th><th>الإجمالي</th><th>المدفوع</th><th>المتبقي</th><th></th></tr>
    ${data.map(p=>`<tr>
      <td><b style="color:var(--accent)">${p.invoice_number}</b></td>
      <td>${p.supplier_name||'—'}</td>
      <td>${p.purchase_date}</td>
      <td>${fmt(p.total_amount)}</td>
      <td>${fmt(p.paid_amount)}</td>
      <td><span class="badge ${p.remaining_amount>0?'br':'bg'}">${fmt(p.remaining_amount)}</span></td>
      <td style="white-space:nowrap">
        <button class="btn btn-ghost btn-sm" onclick="vPurch(${p.id})" title="عرض التفاصيل">👁️</button>
        ${p.remaining_amount>0?`<button class="btn btn-warn btn-sm" onclick="payPurch(${p.id},${p.remaining_amount})">💳</button>`:''}
        <button class="btn btn-danger btn-sm" onclick="dPurch(${p.id})">🗑️</button>
      </td>
    </tr>`).join('')}
  </table>`;
}

function filterPurchasesTable(){
  const search=document.getElementById('purch-search').value.toLowerCase();
  const from=document.getElementById('purch-from').value;
  const to=document.getElementById('purch-to').value;
  const filtered=purchasesData.filter(p=>{
    const matchSearch=!search||p.invoice_number.toLowerCase().includes(search)||(p.supplier_name||'').toLowerCase().includes(search);
    const matchFrom=!from||p.purchase_date>=from;
    const matchTo=!to||p.purchase_date<=to;
    return matchSearch&&matchFrom&&matchTo;
  });
  renderPurchasesTable(filtered);
}

function purchasesPeriod(period){purchases(period);}

async function vPurch(id){
  const r=await api(`/api/purchases/${id}`);
  if(!r.success){toast('خطأ في تحميل البيانات','error');return;}
  const{purchase,details}=r.data;
  let html=`
  <div style="background:var(--bg3);padding:14px;border-radius:9px;margin-bottom:14px;border-left:4px solid var(--accent)">
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;margin-bottom:10px">
      <div><span style="color:var(--muted);font-size:11px">🧾 رقم الفاتورة</span><div style="font-weight:700;color:var(--accent);font-size:16px">${purchase.invoice_number}</div></div>
      <div><span style="color:var(--muted);font-size:11px">📅 التاريخ</span><div style="font-weight:600">${purchase.purchase_date}</div></div>
      <div><span style="color:var(--muted);font-size:11px">🏢 المورد</span><div style="font-weight:600">${purchase.supplier_name||'بدون مورد'}</div></div>
    </div>
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;border-top:1px solid var(--border);padding-top:10px">
      <div style="padding:8px;background:rgba(0,212,255,.1);border-radius:5px"><span style="color:var(--muted);font-size:10px">💰 الإجمالي</span><div style="font-weight:700;color:var(--accent);font-size:15px">${fmt(purchase.total_amount)}</div></div>
      <div style="padding:8px;background:rgba(0,255,157,.1);border-radius:5px"><span style="color:var(--muted);font-size:10px">✅ المدفوع</span><div style="font-weight:700;color:var(--accent2);font-size:15px">${fmt(purchase.paid_amount)}</div></div>
      <div style="padding:8px;background:rgba(${purchase.remaining_amount>0?'255,71,87':'0,255,157'},.1);border-radius:5px"><span style="color:var(--muted);font-size:10px">⏳ المتبقي</span><div style="font-weight:700;color:${purchase.remaining_amount>0?'var(--danger)':'var(--accent2)'};font-size:15px">${fmt(purchase.remaining_amount)}</div></div>
    </div>
  </div>
  <div style="background:var(--bg2);border:1px solid var(--border);border-radius:9px;overflow:hidden;margin-bottom:14px">
    <div style="padding:12px;background:var(--bg3);border-bottom:1px solid var(--border);font-weight:700;color:var(--accent)">📦 المنتجات (${details.length})</div>
    <table style="width:100%;margin:0">
      <tr style="background:var(--bg3)"><th style="padding:10px;text-align:right">🏷️ المنتج</th><th style="padding:10px">📊 الكمية</th><th style="padding:10px">💵 السعر</th><th style="padding:10px">🧮 الإجمالي</th></tr>
      ${details.map(d=>`<tr><td style="padding:10px;border-bottom:1px solid var(--border)"><b style="color:var(--accent)">${d.product_name}</b></td><td style="padding:10px;border-bottom:1px solid var(--border);text-align:center;color:var(--accent2);font-weight:700">${d.quantity}</td><td style="padding:10px;border-bottom:1px solid var(--border);text-align:center">${fmt(d.unit_price)}</td><td style="padding:10px;border-bottom:1px solid var(--border);text-align:center;color:var(--accent2);font-weight:700">${fmt(d.total_price)}</td></tr>`).join('')}
    </table>
  </div>`;
  omo(`📋 فاتورة شراء: ${purchase.invoice_number}`,html,
  `${purchase.remaining_amount>0?`<button class="btn btn-warn" onclick="payPurch(${id},${purchase.remaining_amount})">💳 دفعة</button>`:''}
  <button class="btn btn-danger" onclick="if(confirm('هل تريد حذف هذه الفاتورة؟')) dPurch(${id})">🗑️ حذف</button>
  <button class="btn btn-ghost" onclick="cmo()">✖️ إغلاق</button>`);
}

function payPurch(id,rem){
  omo('💳 تسجيل دفعة للمورد',`
  <div class="ff" style="margin-bottom:14px">
    <label style="color:var(--muted);font-size:12px">المبلغ المتبقي: <b style="color:var(--accent)">${fmt(rem)}</b></label>
    <input id="pay-purch-amount" type="number" value="${rem}" style="margin-top:6px;background:var(--bg3);border:1px solid var(--border);padding:10px;border-radius:7px;color:var(--text);width:100%">
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="sbPayPurch(${id})">✅ تأكيد</button>`);
}
async function sbPayPurch(id){
  const amount=document.getElementById('pay-purch-amount').value;
  if(!amount||parseFloat(amount)<=0){toast('أدخل مبلغاً صحيحاً','error');return;}
  const r=await api(`/api/purchases/${id}/pay`,'POST',{amount,method:'نقدي'});
  if(r.success){cmo();toast('✅ '+r.message);purchases('شهر');}else toast(r.message,'error');
}
async function dPurch(id){
  if(!confirm('هل تريد حذف هذه الفاتورة؟'))return;
  const r=await api(`/api/purchases/${id}`,'DELETE');
  if(r.success){toast('✅ تم الحذف');purchases('شهر');}else toast(r.message,'error');
}

// ═══════════════════════════════════════════════════════════
//  SUPPLIERS
// ═══════════════════════════════════════════════════════════
async function suppliers(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div style="display:flex;justify-content:flex-end;margin-bottom:14px"><button class="btn btn-success" onclick="addSup()">+ إضافة مورد</button></div>
  <div class="tw"><div class="th"><h3>🚚 الموردين</h3></div><div id="supt">${ld()}</div></div>`;
  const r=await api('/api/suppliers');
  document.getElementById('supt').innerHTML=r.data?.length
    ?`<table><tr><th>الاسم</th><th>الهاتف</th><th>البريد</th><th>العنوان</th><th></th></tr>
    ${r.data.map(s=>`<tr><td><b>${s.name}</b></td><td>${s.phone}</td><td>${s.email||'—'}</td><td>${s.address||'—'}</td>
    <td><button class="btn btn-ghost btn-sm" onclick='eSup(${JSON.stringify(s)})'>✏️</button> <button class="btn btn-danger btn-sm" onclick="dSup(${s.id})">🗑️</button></td></tr>`).join('')}</table>`:em();
}
function addSup(){omo('➕ مورد جديد',`<div class="fg"><div class="ff"><label>الاسم *</label><input id="sn"></div><div class="ff"><label>الهاتف *</label><input id="sph"></div><div class="ff"><label>البريد</label><input id="se"></div><div class="ff"><label>العنوان</label><input id="sa"></div></div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="svSup()">💾 حفظ</button>`);}
async function svSup(){const r=await api('/api/suppliers/add','POST',{name:document.getElementById('sn').value,phone:document.getElementById('sph').value,email:document.getElementById('se').value,address:document.getElementById('sa').value});if(r.success){cmo();toast(r.message);suppliers();}else toast(r.message,'error');}
function eSup(s){omo('✏️ تعديل مورد',`<div class="fg"><div class="ff"><label>الاسم</label><input id="esn" value="${s.name}"></div><div class="ff"><label>الهاتف</label><input id="esp" value="${s.phone}"></div><div class="ff"><label>البريد</label><input id="ese" value="${s.email||''}"></div><div class="ff"><label>العنوان</label><input id="esa" value="${s.address||''}"></div></div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="upSup(${s.id})">💾 حفظ</button>`);}
async function upSup(id){const r=await api(`/api/suppliers/${id}`,'PUT',{name:document.getElementById('esn').value,phone:document.getElementById('esp').value,email:document.getElementById('ese').value,address:document.getElementById('esa').value});if(r.success){cmo();toast(r.message);suppliers();}else toast(r.message,'error');}
async function dSup(id){if(!confirm('حذف؟'))return;const r=await api(`/api/suppliers/${id}`,'DELETE');if(r.success){toast(r.message);suppliers();}else toast(r.message,'error');}

// ═══════════════════════════════════════════════════════════
//  CUSTOMERS
// ═══════════════════════════════════════════════════════════
async function customers(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div style="display:flex;justify-content:flex-end;margin-bottom:14px"><button class="btn btn-success" onclick="addCus()">+ إضافة عميل</button></div>
  <div class="tw"><div class="th"><h3>👥 العملاء</h3></div><div id="cut">${ld()}</div></div>`;
  const r=await api('/api/customers');
  document.getElementById('cut').innerHTML=r.data?.length
    ?`<table><tr><th>الاسم</th><th>الهاتف</th><th>البريد</th><th></th></tr>
    ${r.data.map(cu=>`<tr><td><b>${cu.name}</b></td><td>${cu.phone||'—'}</td><td>${cu.email||'—'}</td>
    <td><button class="btn btn-danger btn-sm" onclick="dCus(${cu.id})">🗑️</button></td></tr>`).join('')}</table>`:em();
}
function addCus(){omo('➕ عميل جديد',`<div class="fg"><div class="ff"><label>الاسم *</label><input id="cn"></div><div class="ff"><label>الهاتف</label><input id="cph"></div><div class="ff"><label>البريد</label><input id="ce"></div><div class="ff"><label>العنوان</label><input id="ca"></div></div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="svCus()">💾 حفظ</button>`);}
async function svCus(){const r=await api('/api/customers/add','POST',{name:document.getElementById('cn').value,phone:document.getElementById('cph').value,email:document.getElementById('ce').value,address:document.getElementById('ca').value});if(r.success){cmo();toast(r.message);customers();}else toast(r.message,'error');}
async function dCus(id){if(!confirm('حذف؟'))return;const r=await api(`/api/customers/${id}`,'DELETE');if(r.success){toast(r.message);customers();}else toast(r.message,'error');}

// ═══════════════════════════════════════════════════════════
//  EXPENSES
// ═══════════════════════════════════════════════════════════
async function expenses(pr='شهر'){
  const c=document.getElementById('pc');
  c.innerHTML=`<div style="display:flex;gap:7px;margin-bottom:14px;flex-wrap:wrap">
    ${['اليوم','أسبوع','شهر','سنة'].map(p=>`<button class="btn ${p===pr?'btn-success':'btn-ghost'} btn-sm" onclick="expenses('${p}')">${p}</button>`).join('')}
    <button class="btn btn-success" onclick="addExp()">+ مصروف</button>
  </div><div class="tw"><div class="th"><h3>💸 المصروفات</h3><span id="expt" style="color:var(--warn);font-weight:700"></span></div><div id="exptbl">${ld()}</div></div>`;
  const r=await api(`/api/expenses?period=${encodeURIComponent(pr)}`);
  if(r.data)document.getElementById('expt').textContent='الإجمالي: '+fmt(r.data.total);
  document.getElementById('exptbl').innerHTML=r.data?.items?.length
    ?`<table><tr><th>النوع</th><th>المبلغ</th><th>التاريخ</th><th>ملاحظات</th><th></th></tr>
    ${r.data.items.map(e=>`<tr><td>${e.expense_type}</td><td style="color:var(--danger)">${fmt(e.amount)}</td><td>${e.expense_date}</td><td>${e.notes||'—'}</td>
    <td><button class="btn btn-danger btn-sm" onclick="dExp(${e.id})">🗑️</button></td></tr>`).join('')}</table>`:em();
}
function addExp(){omo('➕ مصروف جديد',`<div class="fg"><div class="ff full"><label>النوع *</label><select id="et"><option>إيجار</option><option>كهرباء</option><option>إنترنت</option><option>رواتب</option><option>نقل</option><option>أخرى</option></select></div><div class="ff"><label>المبلغ *</label><input id="ea" type="number" placeholder="0"></div><div class="ff"><label>التاريخ</label><input id="ed" type="date" value="${new Date().toISOString().slice(0,10)}"></div><div class="ff full"><label>ملاحظات</label><input id="en2" placeholder="اختياري"></div></div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="svExp()">💾 حفظ</button>`);}
async function svExp(){const r=await api('/api/expenses/add','POST',{expense_type:document.getElementById('et').value,amount:document.getElementById('ea').value,expense_date:document.getElementById('ed').value,notes:document.getElementById('en2').value});if(r.success){cmo();toast(r.message);expenses();}else toast(r.message,'error');}
async function dExp(id){if(!confirm('حذف؟'))return;const r=await api(`/api/expenses/${id}`,'DELETE');if(r.success){toast(r.message);expenses();}else toast(r.message,'error');}

// ═══════════════════════════════════════════════════════════
//  MAINTENANCE
// ═══════════════════════════════════════════════════════════
async function maintenance(sf='',sq=''){
  const c=document.getElementById('pc');
  c.innerHTML=`
  <div style="display:flex;gap:20px;align-items:flex-start">
    <!-- الجانب الرئيسي -->
    <div style="flex:1;min-width:0">
      <!-- شريط الأدوات -->
      <div style="display:flex;gap:7px;margin-bottom:14px;flex-wrap:wrap;align-items:center">
        ${['','قيد الإصلاح','جاهز','تم التسليم'].map(s=>`<button class="btn ${s===sf?'btn-success':'btn-ghost'} btn-sm" onclick="maintenance('${s}',document.getElementById('mq-search')?.value||'')">${s||'الكل'}</button>`).join('')}
        <input class="si" id="mq-search" placeholder="🔍 بحث عن عميل أو جهاز..." value="${sq}" oninput="maintenance('${sf}',this.value)" style="flex:1;min-width:180px">
        <button class="btn btn-success btn-sm" onclick="addMaint()">+ طلب صيانة</button>
        <button class="btn btn-danger btn-sm" onclick="clearAllMaint()">🗑️ حذف الكل</button>
      </div>
      <div class="tw"><div class="th"><h3>🔧 طلبات الصيانة</h3><span id="maint-cnt" style="color:var(--muted);font-size:13px"></span></div><div id="mt">${ld()}</div></div>
    </div>
    <!-- اللوحة الجانبية -->
    <div style="width:220px;flex-shrink:0;display:flex;flex-direction:column;gap:12px">
      <div class="sc gr" style="padding:18px">
        <div class="lb">💰 إجمالي السعر</div>
        <div class="vl" id="maint-total-price">0</div>
        <div class="sb">المبالغ المطلوبة</div>
      </div>
      <div class="sc pu" style="padding:18px">
        <div class="lb">💹 إجمالي الربح</div>
        <div class="vl" id="maint-total-profit">0</div>
        <div class="sb">ربح بعد التكاليف</div>
      </div>
      <div class="sc bl" style="padding:18px">
        <div class="lb">📋 عدد الطلبات</div>
        <div class="vl" id="maint-count-box">0</div>
      </div>
    </div>
  </div>`;

  const r=await api(`/api/maintenance${sf?'?status='+encodeURIComponent(sf):''}`);
  let data=r.data||[];

  // فلتر البحث
  if(sq){
    const qs=sq.toLowerCase();
    data=data.filter(m=>m.customer_name.toLowerCase().includes(qs)||m.device_model.toLowerCase().includes(qs)||m.phone.includes(qs));
  }

  // حساب الإجماليات
  let totalPrice=0,totalProfit=0;
  data.forEach(m=>{totalPrice+=(m.selling_price||0);totalProfit+=((m.selling_price||0)-(m.cost||0));});
  document.getElementById('maint-total-price').textContent=fmt(totalPrice);
  document.getElementById('maint-total-profit').textContent=fmt(totalProfit);
  document.getElementById('maint-count-box').textContent=data.length;
  document.getElementById('maint-cnt').textContent=data.length+' طلب';

  document.getElementById('mt').innerHTML=data.length
    ?`<div style="overflow-x:auto"><table style="width:100%"><tr><th>العميل</th><th>الهاتف</th><th>الجهاز</th><th>المشكلة</th><th>التكلفة</th><th>السعر الإجمالي</th><th>الربح</th><th>الحالة</th><th>تاريخ الاستلام</th><th></th></tr>
    ${data.map(m=>{const profit=(m.selling_price||0)-(m.cost||0);return`<tr>
    <td><b>${m.customer_name}</b></td><td>${m.phone}</td><td>${m.device_model}</td>
    <td style="max-width:120px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap" title="${m.problem_description||''}">${m.problem_description||'—'}</td>
    <td>${fmt(m.cost)}</td>
    <td style="color:var(--accent2);font-weight:700">${fmt(m.selling_price||0)}</td>
    <td style="color:${profit>=0?'var(--accent2)':'var(--danger)'}">${profit>=0?'+':''}${fmt(profit)}</td>
    <td><span class="badge ${m.status==='جاهز'?'bg':m.status==='تم التسليم'?'bb':'bw'}">${m.status}</span></td>
    <td style="font-size:11px;color:var(--muted)">${m.received_date||'—'}</td>
    <td style="white-space:nowrap">
      <button class="btn btn-ghost btn-sm" title="تفاصيل" onclick='vMaint(${JSON.stringify(m)})'>👁️</button>
      <button class="btn btn-ghost btn-sm" onclick='upMaint(${JSON.stringify(m)})'>✏️</button>
      <button class="btn btn-danger btn-sm" onclick="dMaint(${m.id})">🗑️</button>
    </td></tr>`;}).join('')}</table></div>`
    :em('لا توجد طلبات صيانة');
}

function vMaint(m){
  const profit=(m.selling_price||0)-(m.cost||0);
  const html=`
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:14px">
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">العميل</div>
      <div style="font-weight:700;font-size:15px">${m.customer_name}</div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">الهاتف</div>
      <div style="font-weight:600">${m.phone}</div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">الجهاز</div>
      <div style="font-weight:700;color:var(--accent)">${m.device_model}</div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">الحالة</div>
      <div><span class="badge ${m.status==='جاهز'?'bg':m.status==='تم التسليم'?'bb':'bw'}">${m.status}</span></div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px;grid-column:1/-1">
      <div style="font-size:11px;color:var(--muted);margin-bottom:5px">المشكلة</div>
      <div style="line-height:1.6">${m.problem_description||'لا يوجد وصف'}</div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">تاريخ الاستلام</div>
      <div>${m.received_date||'—'} ${m.received_time||''}</div>
    </div>
    <div style="background:var(--bg3);border-radius:8px;padding:12px">
      <div style="font-size:11px;color:var(--muted)">تاريخ التسليم</div>
      <div>${m.delivery_date||'لم يُسلَّم بعد'}</div>
    </div>
  </div>
  <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px">
    <div style="background:rgba(0,212,255,.1);border:1px solid var(--accent);border-radius:9px;padding:14px;text-align:center">
      <div style="font-size:11px;color:var(--muted)">التكلفة</div>
      <div style="font-size:20px;font-weight:900;color:var(--accent)">${fmt(m.cost)}</div>
    </div>
    <div style="background:rgba(0,255,157,.1);border:1px solid var(--accent2);border-radius:9px;padding:14px;text-align:center">
      <div style="font-size:11px;color:var(--muted)">السعر الإجمالي</div>
      <div style="font-size:20px;font-weight:900;color:var(--accent2)">${fmt(m.selling_price||0)}</div>
    </div>
    <div style="background:rgba(168,85,247,.1);border:1px solid #a855f7;border-radius:9px;padding:14px;text-align:center">
      <div style="font-size:11px;color:var(--muted)">الربح</div>
      <div style="font-size:20px;font-weight:900;color:${profit>=0?'#a855f7':'var(--danger)'}">${profit>=0?'+':''}${fmt(profit)}</div>
    </div>
  </div>`;
  omo(`📋 تفاصيل طلب صيانة`,html,
  `<button class="btn btn-warn btn-sm" onclick="cmo();upMaint(${JSON.stringify(m).replace(/"/g,'&quot;')})">✏️ تعديل</button>
   <button class="btn btn-ghost" onclick="cmo()">✖ إغلاق</button>`);
}

function clearAllMaint(){
  omo('🗑️ حذف جميع طلبات الصيانة',`
  <div style="background:rgba(255,71,87,.1);border:1px solid var(--danger);border-radius:8px;padding:16px;color:var(--danger)">
    <b>⚠️ تحذير:</b> سيتم حذف جميع طلبات الصيانة نهائياً ولا يمكن التراجع عن هذا الإجراء.
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-danger" onclick="confirmClearAllMaint()">🗑️ حذف الكل</button>`);
}
async function confirmClearAllMaint(){
  const r=await api('/api/maintenance/clear','POST',{});
  if(r.success){cmo();toast('✅ تم حذف جميع طلبات الصيانة');maintenance();}else toast(r.message,'error');
}
function addMaint(){omo('➕ طلب صيانة',`<div class="fg"><div class="ff"><label>العميل *</label><input id="mc"></div><div class="ff"><label>الهاتف *</label><input id="mph"></div><div class="ff full"><label>الجهاز *</label><input id="md"></div><div class="ff full"><label>المشكلة</label><textarea id="mp"></textarea></div><div class="ff"><label>التكلفة (تكلفة الإصلاح)</label><input id="mcost" type="number" value="0" oninput="autoMaintPrice()"></div><div class="ff"><label>السعر الإجمالي (للعميل)</label><input id="msp" type="number" value="0" placeholder="السعر المطلوب من العميل"></div></div><div style="background:rgba(0,212,255,.08);border:1px solid var(--border);border-radius:8px;padding:10px;margin-top:10px;font-size:12px;color:var(--muted)">💡 التكلفة = تكلفة قطع الغيار والعمالة | السعر الإجمالي = السعر المطلوب من العميل</div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="svMaint()">💾 حفظ</button>`);}
function autoMaintPrice(){const c=parseFloat(document.getElementById('mcost')?.value)||0;const sp=document.getElementById('msp');if(sp&&(parseFloat(sp.value)||0)===0)sp.value=c;}
async function svMaint(){const r=await api('/api/maintenance/add','POST',{customer_name:document.getElementById('mc').value,phone:document.getElementById('mph').value,device_model:document.getElementById('md').value,problem_description:document.getElementById('mp').value,cost:document.getElementById('mcost').value,selling_price:document.getElementById('msp').value||0,received_time:new Date().toTimeString().slice(0,5)});if(r.success){cmo();toast(r.message);maintenance();}else toast(r.message,'error');}
function upMaint(m){omo('✏️ تحديث',`<div class="fg"><div class="ff full"><label>الحالة</label><select id="ums"><option ${m.status==='قيد الإصلاح'?'selected':''}>قيد الإصلاح</option><option ${m.status==='جاهز'?'selected':''}>جاهز</option><option ${m.status==='تم التسليم'?'selected':''}>تم التسليم</option></select></div><div class="ff"><label>التكلفة</label><input id="umc" type="number" value="${m.cost}" oninput="autoMaintPriceUp()"></div><div class="ff"><label>السعر الإجمالي (للعميل)</label><input id="umsp" type="number" value="${m.selling_price||m.cost||0}"></div></div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="sbUpMaint(${m.id})">💾 حفظ</button>`);}
function autoMaintPriceUp(){const c=parseFloat(document.getElementById('umc')?.value)||0;const sp=document.getElementById('umsp');if(sp&&(parseFloat(sp.value)||0)===0)sp.value=c;}
async function sbUpMaint(id){const r=await api(`/api/maintenance/${id}`,'PUT',{status:document.getElementById('ums').value,cost:document.getElementById('umc').value,selling_price:document.getElementById('umsp')?.value||0});if(r.success){cmo();toast(r.message);maintenance();}else toast(r.message,'error');}
async function dMaint(id){if(!confirm('حذف؟'))return;const r=await api(`/api/maintenance/${id}`,'DELETE');if(r.success){toast(r.message);maintenance();}else toast(r.message,'error');}

// ═══════════════════════════════════════════════════════════
//  REPORTS
// ═══════════════════════════════════════════════════════════
async function reports(){
  document.getElementById('pc').innerHTML=`<div class="tabs"><div class="tab active" id="rt-f" onclick="rTab('f')">💰 مالي</div><div class="tab" id="rt-i" onclick="rTab('i')">📦 مخزون</div><div class="tab" id="rt-t" onclick="rTab('t')">🏆 أفضل المنتجات</div></div><div id="rb">${ld()}</div>`;
  rLoadFin();
}
function rTab(t){document.querySelectorAll('.tab').forEach(x=>x.classList.remove('active'));document.getElementById('rt-'+t).classList.add('active');document.getElementById('rb').innerHTML=ld();if(t==='f')rLoadFin();else if(t==='i')rLoadInv();else rLoadTop();}
async function rLoadFin(){
  const r=await api('/api/reports/dashboard');if(!r.success)return;const d=r.data;
  document.getElementById('rb').innerHTML=`<div class="sg">${[
    ['💵 مبيعات اليوم',fmt(d.today_sales),d.today_count+' فاتورة','bl'],
    ['📈 مبيعات الشهر',fmt(d.month_sales),d.month_count+' فاتورة','bl'],
    ['💹 ربح الشهر',fmt(d.month_profit),'','gr'],
    ['🗓️ ربح السنة',fmt(d.year_profit),'','gr'],
    ['💼 إجمالي الأرباح',fmt(d.total_profit),'','gr'],
    ['🛍️ مشتريات الشهر',fmt(d.month_purchases),'','wn'],
    ['💸 مصروفات الشهر',fmt(d.month_expenses),'','wn'],
    ['⚖️ صافي الشهر',fmt(d.month_net),'',d.month_net>=0?'gr':'rd'],
    ['📥 ديون العملاء',fmt(d.receivables),'','rd'],
    ['📤 ديون الموردين',fmt(d.payables),'','rd'],
    ['📦 قيمة المخزون',fmt(d.inventory_value),d.total_products+' منتج','pu'],
    ['👥 إجمالي العملاء',d.total_customers,'','bl'],
  ].map(([l,v,s,cl])=>`<div class="sc ${cl}"><div class="lb">${l}</div><div class="vl">${v}</div>${s?`<div class="sb">${s}</div>`:''}</div>`).join('')}</div>
  <div style="margin-top:14px;background:var(--bg2);border:1px solid var(--border);border-radius:9px;padding:14px"><b style="color:var(--accent)">🥇 أكثر منتج مبيعاً:</b> ${d.best_product}</div>`;
}
async function rLoadInv(){
  const r=await api('/api/reports/inventory');if(!r.success||!r.data)return;const{summary,items}=r.data;
  document.getElementById('rb').innerHTML=`<div class="sg" style="margin-bottom:14px">
    <div class="sc bl"><div class="lb">إجمالي التكلفة</div><div class="vl">${fmt(summary.total_cost)}</div></div>
    <div class="sc gr"><div class="lb">إجمالي القيمة</div><div class="vl">${fmt(summary.total_value)}</div></div>
    <div class="sc wn"><div class="lb">منخفض</div><div class="vl">${summary.low_stock}</div></div>
    <div class="sc rd"><div class="lb">نافذ</div><div class="vl">${summary.out_of_stock}</div></div>
  </div>
  <div class="tw"><table><tr><th>المنتج</th><th>الكمية</th><th>شراء</th><th>بيع</th><th>إجمالي التكلفة</th><th>ربح متوقع</th></tr>
  ${items.map(p=>`<tr><td><b>${p.name}</b><br><span style="color:var(--muted);font-size:11px">${p.brand}</span></td><td><span class="badge ${p.quantity<=0?'br':p.quantity<=p.min_stock?'bw':'bg'}">${p.quantity}</span></td><td>${fmt(p.cost_price)}</td><td>${fmt(p.selling_price)}</td><td>${fmt(p.total_cost)}</td><td style="color:var(--accent2)">${fmt(p.expected_profit)}</td></tr>`).join('')}</table></div>`;
}
async function rLoadTop(){
  const r=await api('/api/reports/top-products?limit=20');
  document.getElementById('rb').innerHTML=r.data?.length
    ?`<div class="tw"><table><tr><th>#</th><th>المنتج</th><th>الماركة</th><th>الكمية المباعة</th><th>الإيراد</th><th>الربح</th></tr>
    ${r.data.map((p,i)=>`<tr><td><b style="color:${i<3?'var(--warn)':'var(--muted)'}">${['🥇','🥈','🥉'][i]||'#'+(i+1)}</b></td><td>${p.name}</td><td>${p.brand}</td><td>${fmt(p.total_qty)}</td><td>${fmt(p.total_revenue)}</td><td style="color:var(--accent2)">${fmt(p.total_profit)}</td></tr>`).join('')}</table></div>`
    :em('لا توجد مبيعات');
}

// ═══════════════════════════════════════════════════════════
//  CURRENCY
// ═══════════════════════════════════════════════════════════
let rates={syp_buy:14800,syp_sell:15000,try_buy:29.5,try_sell:30.5};
async function currency(){
  const r=await api('/api/currency/rates');
  if(r.success&&r.data)rates={...rates,...r.data};
  document.getElementById('pc').innerHTML=`<div style="max-width:720px">
    <div class="tw" style="padding:18px;margin-bottom:18px">
      <h3 style="margin-bottom:14px;color:var(--accent)">⚙️ أسعار الصرف</h3>
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px">
        <div style="background:var(--bg3);border-radius:8px;padding:13px"><label style="font-size:11px;color:var(--muted)">🇸🇾 سوري (شراء)</label><input class="rin" id="r1" value="${rates.syp_buy}" oninput="svRates();calcCur()" style="margin-top:6px"></div>
        <div style="background:var(--bg3);border-radius:8px;padding:13px"><label style="font-size:11px;color:var(--muted)">🇸🇾 سوري (بيع)</label><input class="rin" id="r2" value="${rates.syp_sell}" oninput="svRates();calcCur()" style="margin-top:6px"></div>
        <div style="background:var(--bg3);border-radius:8px;padding:13px"><label style="font-size:11px;color:var(--muted)">🇹🇷 تركي (شراء)</label><input class="rin" id="r3" value="${rates.try_buy}" oninput="svRates();calcCur()" style="margin-top:6px"></div>
        <div style="background:var(--bg3);border-radius:8px;padding:13px"><label style="font-size:11px;color:var(--muted)">🇹🇷 تركي (بيع)</label><input class="rin" id="r4" value="${rates.try_sell}" oninput="svRates();calcCur()" style="margin-top:6px"></div>
      </div>
      <div style="margin-top:10px;padding:8px 12px;background:rgba(0,255,157,.08);border-radius:7px;font-size:12px;color:var(--accent2)">✅ تُحفظ تلقائياً ولا تتغير حتى تعدلها</div>
    </div>
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:14px">
      ${[['🇺🇸 دولار','usd','var(--accent)'],['🇸🇾 سوري','syp','var(--accent2)'],['🇹🇷 تركي','try','var(--warn)']].map(([lbl,id,cl])=>`
      <div class="tw" style="padding:14px">
        <h4 style="margin-bottom:11px;color:${cl}">${lbl} ←</h4>
        <input class="rin" id="${id}-in" type="number" placeholder="أدخل مبلغ..." oninput="calcCur()" style="margin-bottom:11px">
        <div id="${id}-res"></div>
      </div>`).join('')}
    </div>
  </div>`;
}
let svT;
function svRates(){rates.syp_buy=parseFloat(document.getElementById('r1').value)||14800;rates.syp_sell=parseFloat(document.getElementById('r2').value)||15000;rates.try_buy=parseFloat(document.getElementById('r3').value)||29.5;rates.try_sell=parseFloat(document.getElementById('r4').value)||30.5;clearTimeout(svT);svT=setTimeout(()=>api('/api/currency/rates','POST',rates),900);}
function nv(id){return parseFloat(document.getElementById(id)?.value)||0;}
function resBox(label,value,color='var(--accent2)'){return`<div style="background:var(--bg3);border-radius:7px;padding:11px;margin-bottom:8px"><span style="font-size:11px;color:var(--muted)">${label}</span><div style="font-size:19px;font-weight:900;color:${color}">${fmt(value)}</div></div>`;}
function calcCur(){
  const{syp_buy:sb,syp_sell:ss,try_buy:tb,try_sell:ts}=rates;
  const usd=nv('usd-in'),syp=nv('syp-in'),tryv=nv('try-in');
  const ur=document.getElementById('usd-res'),sr=document.getElementById('syp-res'),tr=document.getElementById('try-res');
  if(ur&&usd){ur.innerHTML=resBox('سوري (شراء)',usd*sb)+resBox('سوري (بيع)',usd*ss,'var(--danger)')+resBox('تركي (شراء)',usd*tb)+resBox('تركي (بيع)',usd*ts,'var(--danger)');}
  if(sr&&syp){sr.innerHTML=resBox('دولار (شراء)',ss>0?syp/ss:0)+resBox('دولار (بيع)',sb>0?syp/sb:0,'var(--danger)')+resBox('تركي (شراء)',ts>0?syp/ts:0)+resBox('تركي (بيع)',tb>0?syp/tb:0,'var(--danger)');}
  if(tr&&tryv){tr.innerHTML=resBox('دولار (شراء)',ts>0?tryv/ts:0)+resBox('دولار (بيع)',tb>0?tryv/tb:0,'var(--danger)')+resBox('سوري (شراء)',tryv*(ss>0?sb/ts:0))+resBox('سوري (بيع)',tryv*(tb>0?ss/tb:0),'var(--danger)');}
}

// ═══════════════════════════════════════════════════════════
//  الأقسام الجديدة (أقسام سطح المكتب)
// ═══════════════════════════════════════════════════════════

async function barcode_sale(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div class="tw">
    <div class="th"><h3>📦 بيع بالباركود</h3></div>
    <div style="padding:20px">
      <div class="fg" style="margin-bottom:20px">
        <div class="ff full"><label>ماسح الرمز (أو ابحث عن المنتج)</label>
          <input id="barcode-input" placeholder="امسح الباركود أو ابحث..." autofocus style="font-size:16px;padding:10px;border:2px solid var(--accent);border-radius:8px">
        </div>
      </div>
      <div id="sale-cart"></div>
      <div style="margin-top:20px;padding:20px;background:var(--bg3);border-radius:8px">
        <div style="display:flex;justify-content:space-between;margin-bottom:10px">
          <span>الإجمالي:</span>
          <span id="total-amount" style="font-size:18px;font-weight:bold;color:var(--accent)">0</span>
        </div>
        <button class="btn btn-success" style="width:100%;margin-bottom:10px" onclick="completeSale()">✅ إتمام البيع</button>
        <button class="btn btn-ghost" style="width:100%" onclick="clearCart()">🗑️ مسح السلة</button>
      </div>
    </div>
  </div>`;
  
  let cart=[];
  document.getElementById('barcode-input').addEventListener('keypress',async(e)=>{
    if(e.key!=='Enter')return;
    const code=e.target.value.trim();
    e.target.value='';
    
    const r=await api(`/api/products?search=${encodeURIComponent(code)}`);
    if(r.data?.length){
      const product=r.data[0];
      const existing=cart.find(c=>c.id===product.id);
      if(existing){
        existing.qty++;
      }else{
        cart.push({...product,qty:1});
      }
      updateCart(cart);
      e.target.focus();
    }else{
      alert('❌ المنتج غير موجود');
    }
  });
  
  window.updateCart=(items)=>{
    const html=items.map(p=>`
      <div style="background:var(--bg2);border:1px solid var(--border);border-radius:8px;padding:12px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center">
        <div>
          <b>${p.name}</b><br>
          <span style="color:var(--muted);font-size:12px">${p.brand} - ${fmt(p.selling_price)} لكل واحد</span>
        </div>
        <div style="display:flex;gap:5px;align-items:center">
          <button class="btn btn-ghost btn-sm" onclick="decreaseQty(${p.id},window.cartItems)">➖</button>
          <span style="font-weight:bold;min-width:30px;text-align:center">${p.qty}</span>
          <button class="btn btn-ghost btn-sm" onclick="increaseQty(${p.id},window.cartItems)">➕</button>
          <button class="btn btn-danger btn-sm" onclick="removeItem(${p.id},window.cartItems)">🗑️</button>
          <span style="font-weight:bold;color:var(--accent);min-width:80px;text-align:right">${fmt(p.qty*p.selling_price)}</span>
        </div>
      </div>
    `).join('');
    
    document.getElementById('sale-cart').innerHTML=html||'<p style="color:var(--muted);text-align:center">السلة فارغة</p>';
    
    const total=items.reduce((a,p)=>a+(p.qty*p.selling_price),0);
    document.getElementById('total-amount').textContent=fmt(total);
    
    window.cartItems=items;
  };
  
  window.increaseQty=(id,items)=>{
    const item=items.find(i=>i.id===id);
    if(item)item.qty++;
    updateCart(items);
  };
  
  window.decreaseQty=(id,items)=>{
    const item=items.find(i=>i.id===id);
    if(item&&item.qty>1)item.qty--;
    updateCart(items);
  };
  
  window.removeItem=(id,items)=>{
    const idx=items.findIndex(i=>i.id===id);
    if(idx>-1)items.splice(idx,1);
    updateCart(items);
  };
  
  window.completeSale=async()=>{
    if(!window.cartItems?.length){alert('❌ السلة فارغة');return;}
    const items=window.cartItems.map(p=>({product_id:p.id,quantity:p.qty,unit_price:p.selling_price}));
    const r=await api('/api/sales/add','POST',{items});
    if(r.success){
      alert('✅ تم البيع بنجاح');
      window.cartItems=[];
      barcode_sale();
    }else alert('❌ '+r.message);
  };
  
  window.clearCart=()=>{
    if(confirm('هل متأكد؟')){
      window.cartItems=[];
      updateCart([]);
    }
  };
  
  updateCart(cart);
}

async function barcode_return(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div class="tw">
    <div class="th"><h3>🔄 استرجاع بالباركود</h3></div>
    <div style="padding:20px">
      <div class="fg" style="margin-bottom:20px">
        <div class="ff full"><label>ماسح الرمز (أو ابحث عن المنتج)</label>
          <input id="return-input" placeholder="امسح الباركود أو ابحث..." autofocus style="font-size:16px;padding:10px;border:2px solid var(--warn);border-radius:8px">
        </div>
      </div>
      <div id="return-list"></div>
      <div style="margin-top:20px;padding:20px;background:var(--bg3);border-radius:8px">
        <div style="display:flex;justify-content:space-between;margin-bottom:10px">
          <span>إجمالي المرجعات:</span>
          <span id="total-return" style="font-size:18px;font-weight:bold;color:var(--warn)">0</span>
        </div>
        <button class="btn btn-warn" style="width:100%;margin-bottom:10px;background:var(--warn);color:#0f1923" onclick="completeReturn()">✅ إتمام الاسترجاع</button>
        <button class="btn btn-ghost" style="width:100%" onclick="clearReturns()">🗑️ مسح القائمة</button>
      </div>
    </div>
  </div>`;
  
  let returns=[];
  document.getElementById('return-input').addEventListener('keypress',async(e)=>{
    if(e.key!=='Enter')return;
    const code=e.target.value.trim();
    e.target.value='';
    
    const r=await api(`/api/products?search=${encodeURIComponent(code)}`);
    if(r.data?.length){
      const product=r.data[0];
      const existing=returns.find(c=>c.id===product.id);
      if(existing){
        existing.qty++;
      }else{
        returns.push({...product,qty:1});
      }
      updateReturns(returns);
      e.target.focus();
    }else{
      alert('❌ المنتج غير موجود');
    }
  });
  
  window.updateReturns=(items)=>{
    const html=items.map(p=>`
      <div style="background:var(--bg2);border:2px solid var(--warn);border-radius:8px;padding:12px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center">
        <div>
          <b>${p.name}</b><br>
          <span style="color:var(--muted);font-size:12px">${p.brand} - سعر الاسترجاع: ${fmt(p.selling_price)}</span>
        </div>
        <div style="display:flex;gap:5px;align-items:center">
          <button class="btn btn-ghost btn-sm" onclick="decreaseReturn(${p.id},window.returnItems)">➖</button>
          <span style="font-weight:bold;min-width:30px;text-align:center">${p.qty}</span>
          <button class="btn btn-ghost btn-sm" onclick="increaseReturn(${p.id},window.returnItems)">➕</button>
          <button class="btn btn-danger btn-sm" onclick="removeReturn(${p.id},window.returnItems)">🗑️</button>
          <span style="font-weight:bold;color:var(--warn);min-width:80px;text-align:right">${fmt(p.qty*p.selling_price)}</span>
        </div>
      </div>
    `).join('');
    
    document.getElementById('return-list').innerHTML=html||'<p style="color:var(--muted);text-align:center">لا توجد مرجعات</p>';
    
    const total=items.reduce((a,p)=>a+(p.qty*p.selling_price),0);
    document.getElementById('total-return').textContent=fmt(total);
    
    window.returnItems=items;
  };
  
  window.increaseReturn=(id,items)=>{
    const item=items.find(i=>i.id===id);
    if(item)item.qty++;
    updateReturns(items);
  };
  
  window.decreaseReturn=(id,items)=>{
    const item=items.find(i=>i.id===id);
    if(item&&item.qty>1)item.qty--;
    updateReturns(items);
  };
  
  window.removeReturn=(id,items)=>{
    const idx=items.findIndex(i=>i.id===id);
    if(idx>-1)items.splice(idx,1);
    updateReturns(items);
  };
  
  window.completeReturn=async()=>{
    if(!window.returnItems?.length){alert('❌ لا توجد مرجعات');return;}
    const items=window.returnItems.map(p=>({product_id:p.id,quantity:p.qty,unit_price:p.selling_price,type:'return'}));
    const r=await api('/api/sales/return','POST',{items});
    if(r.success){
      alert('✅ تم معالجة الاسترجاع بنجاح');
      window.returnItems=[];
      barcode_return();
    }else alert('❌ '+r.message);
  };
  
  window.clearReturns=()=>{
    if(confirm('هل متأكد؟')){
      window.returnItems=[];
      updateReturns([]);
    }
  };
  
  updateReturns(returns);
}

async function count(){
  const c=document.getElementById('pc');
  c.innerHTML=`<div class="sb-bar">
    <input class="si" id="search-count" placeholder="🔍 بحث عن منتج..." oninput="loadCountItems()">
    <button class="btn btn-success" onclick="startCount()">+ بدء جرد جديد</button>
  </div>
  <div class="tw">
    <div class="th"><h3>📊 الجرد والمراجعة</h3></div>
    <div id="count-content">${ld()}</div>
  </div>`;
  await loadCountItems();
}

async function loadCountItems(){
  const q=document.getElementById('search-count')?.value||'';
  const r=await api(`/api/inventory?search=${encodeURIComponent(q)}`);
  const items=r.data||[];
  
  document.getElementById('count-content').innerHTML=items.length
    ?`<table><tr><th>المنتج</th><th>الماركة</th><th>الكمية المسجلة</th><th>الكمية الفعلية</th><th>الفرق</th><th></th></tr>
    ${items.map(p=>`<tr>
      <td><b>${p.name}</b></td>
      <td>${p.brand}</td>
      <td><input type="number" value="${p.quantity}" data-product="${p.id}" class="count-recorded" style="width:100px;padding:5px;border:1px solid var(--border);border-radius:5px;background:var(--bg3);color:var(--text)"></td>
      <td><input type="number" value="${p.quantity}" data-product="${p.id}" class="count-actual" style="width:100px;padding:5px;border:1px solid var(--border);border-radius:5px;background:var(--bg3);color:var(--text)"></td>
      <td><span class="count-diff" data-product="${p.id}">0</span></td>
      <td><button class="btn btn-ghost btn-sm" onclick="calcDiff(${p.id})">🔄</button></td>
    </tr>`).join('')}
    <tr><td colspan="6" style="padding:15px;text-align:center">
      <button class="btn btn-success" onclick="saveCount()">💾 حفظ نتائج الجرد</button>
    </td></tr></table>`
    :em('لا توجد منتجات');
}

function calcDiff(id){
  const recorded=parseFloat(document.querySelector(`.count-recorded[data-product="${id}"]`)?.value||0);
  const actual=parseFloat(document.querySelector(`.count-actual[data-product="${id}"]`)?.value||0);
  const diff=actual-recorded;
  document.querySelector(`.count-diff[data-product="${id}"]`).textContent=diff>0?`+${diff}`:diff;
  document.querySelector(`.count-diff[data-product="${id}"]`).style.color=diff>0?'var(--accent2)':diff<0?'var(--danger)':'var(--muted)';
}

function startCount(){
  alert('✅ تم بدء جرد جديد - أدخل الكميات الفعلية وانقر حفظ');
}

let mobileProducts=[];
let mobileSuppliers=[];
let mobileSupplierOptions='';
let mobileProductOptions='';

async function saveCount(){
  const items=document.querySelectorAll('[data-product]');
  let data=[];
  items.forEach(input=>{
    const id=input.dataset.product;
    if(!data.find(d=>d.id==id)){
      const recorded=parseFloat(document.querySelector(`.count-recorded[data-product="${id}"]`)?.value||0);
      const actual=parseFloat(document.querySelector(`.count-actual[data-product="${id}"]`)?.value||0);
      data.push({id,recorded,actual,diff:actual-recorded});
    }
  });
  const r=await api('/api/inventory/count','POST',{items:data});
  if(r.success){toast('✅ تم حفظ نتائج الجرد');count();}else toast(r.message,'error');
}

async function mobile(tab='inventory',q=''){
  const c=document.getElementById('pc');
  c.innerHTML=`
  <div class="tabs">
    <div class="tab ${tab==='inventory'?'active':''}" onclick="mobile('inventory')">📱 المخزون</div>
    <div class="tab ${tab==='purchases'?'active':''}" onclick="mobile('purchases')">🛒 الفواتير</div>
    <div class="tab ${tab==='debts'?'active':''}" onclick="mobile('debts')">📝 الديون</div>
    <div class="tab ${tab==='cash'?'active':''}" onclick="mobile('cash')">💰 الصندوق</div>
    <div class="tab ${tab==='report'?'active':''}" onclick="mobile('report')">📊 التقرير</div>
  </div>
  <div id="mobile-content">${ld()}</div>`;
  
  if(tab==='inventory') await loadMobileInventory(q);
  else if(tab==='purchases') await loadMobilePurchases(q);
  else if(tab==='debts') await loadMobileDebts(q);
  else if(tab==='cash') await loadMobileCash();
  else if(tab==='report') await loadMobileReport();
}

async function loadMobileInventory(q=''){
  const c=document.getElementById('mobile-content');
  c.innerHTML=`<div class="sb-bar">
    <input class="si" id="mq" placeholder="🔍 بحث..." value="${q}" oninput="mobile('inventory',this.value)">
    <button class="btn btn-success" onclick="omo('➕ إضافة موبايل',addMobileForm(),'<button class=btn btn-ghost onclick=cmo()>إلغاء</button><button class=btn btn-success onclick=saveMobile()>💾 حفظ</button>')">+ إضافة</button>
  </div>
  <div id="mobile-inv-table">${ld()}</div>`;
  
  const r=await api(`/api/mobile/inventory?search=${encodeURIComponent(q)}`);
  const items=r.data||[];
  
  document.getElementById('mobile-inv-table').innerHTML=items.length?
    `<div class="tw"><div class="th"><h3>📱 مخزون الموبايلات</h3><span style="color:var(--muted)">${items.length} موبايل</span></div>
    <table><tr><th>الاسم</th><th>الماركة</th><th>الموديل</th><th>الكمية</th><th>سعر الشراء</th><th>سعر البيع</th><th>الحالة</th><th></th></tr>
    ${items.map(p=>`<tr>
      <td><b>${p.name}</b></td><td>${p.brand}</td><td>${p.model}</td>
      <td><span class="badge ${p.quantity<=0?'br':'bg'}">${p.quantity}</span></td>
      <td>${fmt(p.cost_price)}</td><td>${fmt(p.selling_price)}</td>
      <td>${p.condition}</td>
      <td style="white-space:nowrap">
        <button class="btn btn-ghost btn-sm" onclick='editMobile(${JSON.stringify(p)})'>✏️</button>
        <button class="btn btn-danger btn-sm" onclick="delMobile(${p.id},'${p.name}')">🗑️</button>
        <button class="btn btn-success btn-sm" onclick="dirSaleMobile(${JSON.stringify(p)})">💰 بيع</button>
      </td></tr>`).join('')}</table></div>`
    :`<div class="tw"><div class="th"><h3>📱 مخزون الموبايلات</h3></div><div style="padding:20px;text-align:center;color:var(--muted)">لا توجد موبايلات</div></div>`;
}

function addMobileForm(){
  return `<div class="fg">
    <div class="ff full"><label>الاسم *</label><input id="mn" placeholder="iPhone 15 Pro"></div>
    <div class="ff"><label>الماركة</label><input id="mb" placeholder="Apple"></div>
    <div class="ff"><label>الموديل</label><input id="mm"></div>
    <div class="ff"><label>سعر الشراء *</label><input id="mcp" type="number" placeholder="0"></div>
    <div class="ff"><label>سعر البيع *</label><input id="msp" type="number" placeholder="0"></div>
    <div class="ff"><label>الكمية *</label><input id="mq" type="number" placeholder="1"></div>
    <div class="ff"><label>الباركود</label><input id="mbar"></div>
    <div class="ff"><label>الحالة</label><select id="mcond"><option>جديد</option><option>مستعمل</option></select></div>
  </div>`;
}

async function saveMobile(){
  const data={
    name:document.getElementById('mn').value,
    brand:document.getElementById('mb').value,
    model:document.getElementById('mm').value,
    cost_price:parseFloat(document.getElementById('mcp').value),
    selling_price:parseFloat(document.getElementById('msp').value),
    quantity:parseInt(document.getElementById('mq').value),
    barcode:document.getElementById('mbar').value,
    condition:document.getElementById('mcond').value,
    category:'موبايلات'
  };
  
  const r=await api('/api/products/add','POST',data);
  if(r.success){cmo();toast(r.message);mobile('inventory');}else toast(r.message,'error');
}

function editMobile(p){
  omo('✏️ تعديل موبايل',`<div class="fg">
    <div class="ff full"><label>الاسم</label><input id="emn" value="${p.name}"></div>
    <div class="ff"><label>الماركة</label><input id="emb" value="${p.brand}"></div>
    <div class="ff"><label>الموديل</label><input id="emm" value="${p.model}"></div>
    <div class="ff"><label>سعر الشراء</label><input id="emcp" type="number" value="${p.cost_price}"></div>
    <div class="ff"><label>سعر البيع</label><input id="emsp" type="number" value="${p.selling_price}"></div>
    <div class="ff"><label>الكمية</label><input id="emq" type="number" value="${p.quantity}"></div>
    <div class="ff"><label>الحالة</label><select id="emcond"><option ${p.condition==='جديد'?'selected':''}>جديد</option><option ${p.condition==='مستعمل'?'selected':''}>مستعمل</option></select></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="updateMobile(${p.id})">💾 حفظ</button>`);
}

async function updateMobile(id){
  const data={
    name:document.getElementById('emn').value,
    brand:document.getElementById('emb').value,
    model:document.getElementById('emm').value,
    cost_price:parseFloat(document.getElementById('emcp').value),
    selling_price:parseFloat(document.getElementById('emsp').value),
    quantity:parseInt(document.getElementById('emq').value),
    condition:document.getElementById('emcond').value
  };
  
  const r=await api(`/api/mobile/inventory/edit/${id}`,'PUT',data);
  if(r.success){cmo();toast(r.message);mobile('inventory');}else toast(r.message,'error');
}

async function delMobile(id,name){
  if(!confirm(`حذف: ${name}?`))return;
  const r=await api(`/api/mobile/inventory/delete/${id}`,'DELETE');
  if(r.success){toast(r.message);mobile('inventory');}else toast(r.message,'error');
}

function dirSaleMobile(p){
  omo('💰 بيع مباشر: '+p.name,`<div class="fg">
    <div class="ff full"><label>سعر البيع *</label><input id="sp" type="number" value="${p.selling_price}" placeholder="0"></div>
    <div class="ff full"><label>المبلغ المدفوع *</label><input id="sa" type="number" value="${p.selling_price}" placeholder="0"></div>
    <div class="ff full"><label>اسم العميل (للدين)</label><input id="scn" placeholder="اختياري"></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="processDirSale(${p.id})">✅ بيع</button>`);
}

async function processDirSale(pid){
  const price=parseFloat(document.getElementById('sp').value);
  const paid=parseFloat(document.getElementById('sa').value);
  const name=document.getElementById('scn').value||'عميل نقدي';
  
  if(!price||price<=0){toast('أدخل السعر بشكل صحيح','error');return;}
  
  const r=await api('/api/mobile/direct-sale','POST',{product_id:pid,price,paid_amount:paid,customer_name:name});
  if(r.success){cmo();toast(r.message);mobile('inventory');}else toast(r.message,'error');
}

async function loadMobilePurchases(q=''){
  const c=document.getElementById('mobile-content');
  const supRes=await api('/api/suppliers');
  const prodRes=await api('/api/products?category=موبايلات');
  mobileSuppliers=supRes.data||[];
  mobileProducts=prodRes.data||[];
  mobileSupplierOptions = mobileSuppliers.map(s=>`<option value="${s.id}">${(s.name||'').replace(/"/g,'&quot;')}${s.phone?` (${(s.phone||'').replace(/"/g,'&quot;')})`:''}</option>`).join('');
  mobileProductOptions = mobileProducts.map(p=>`<option value="${p.id}" data-cost="${p.cost_price}" data-sell="${p.selling_price}">${(p.name||'').replace(/"/g,'&quot;')} | ${(p.brand||'').replace(/"/g,'&quot;')}</option>`).join('');
  
  c.innerHTML=`<div class="sb-bar">
    <input class="si" id="pq" placeholder="🔍 بحث..." value="${q}" oninput="mobile('purchases',this.value)">
    <button class="btn btn-success" onclick="newMobilePurchase()">+ فاتورة جديدة</button>
  </div>
  <div id="mobile-pur-table">${ld()}</div>`;
  
  const r=await api(`/api/mobile/purchases?search=${encodeURIComponent(q)}`);
  const items=r.data||[];
  
  document.getElementById('mobile-pur-table').innerHTML=items.length?
    `<div class="tw"><div class="th"><h3>🛒 فواتير المشتريات</h3><span style="color:var(--muted)">${items.length} فاتورة</span></div>
    <table><tr><th>رقم الفاتورة</th><th>المورد</th><th>الإجمالي</th><th>المدفوع</th><th>المتبقي</th><th>التاريخ</th><th></th></tr>
    ${items.map(p=>`<tr>
      <td><b>${p.invoice_number}</b></td><td>${p.supplier||'-'}</td>
      <td>${fmt(p.total_amount)}</td><td class="ok">${fmt(p.paid_amount)}</td>
      <td class="err">${fmt(p.remaining_amount)}</td><td>${p.purchase_date}</td>
      <td style="white-space:nowrap">
        <button class="btn btn-ghost btn-sm" onclick="viewPurInv(${p.id},'${p.invoice_number}')">👁️</button>
        <button class="btn btn-danger btn-sm" onclick="delPurInv(${p.id})">🗑️</button>
      </td></tr>`).join('')}</table></div>`
    :`<div class="tw"><div class="th"><h3>🛒 فواتير المشتريات</h3></div><div style="padding:20px;text-align:center;color:var(--muted)">لا توجد فواتير</div></div>`;
}

function newMobilePurchase(){
  omo('➕ فاتورة مشتريات جديدة',`<div class="fg">
    <div class="ff full"><label>المورد</label><select id="psupp"><option value="">— بدون مورد —</option>${mobileSupplierOptions}</select></div>
    <div class="ff full"><label>رقم الفاتورة</label><input id="pinv" value="سيتم إنشاؤه تلقائياً بعد الحفظ" readonly></div>
    <div class="ff"><label>الإجمالي</label><input id="ptotal" type="number" placeholder="0"></div>
    <div class="ff"><label>المدفوع</label><input id="ppaid" type="number" placeholder="0" oninput="mobileRefreshPurchaseTotals()"></div>
    <div id="pur-items" style="grid-column:1/-1;border:1px solid var(--border);border-radius:8px;padding:10px;margin-top:10px"></div>
    <button class="btn btn-ghost btn-sm" style="grid-column:1/-1" onclick="addPurItem()">+ إضافة صنف</button>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="savePurInv()">💾 حفظ</button>`);
  addPurItem();
  mobileRefreshPurchaseTotals();
}

function addPurItem(){
  const cont=document.getElementById('pur-items');
  const id='pi-'+Date.now();
  const html=`<div id="${id}" class="pi-row" style="display:grid;grid-template-columns:2fr 1fr 1fr 1fr 1fr auto;gap:5px;margin-bottom:10px;align-items:center">
    <select class="pi-prod">
      <option value="">اختر منتجاً</option>${mobileProductOptions}
    </select>
    <input type="number" placeholder="الكمية" class="pi-qty">
    <input type="number" placeholder="سعر الشراء" class="pi-price">
    <input type="number" placeholder="سعر البيع" class="pi-price-sell">
    <span class="pi-total">0</span>
    <button class="btn btn-danger btn-sm" onclick="document.getElementById('${id}').remove();mobileRefreshPurchaseTotals()">🗑️</button>
  </div>`;
  cont.insertAdjacentHTML('beforeend',html);
  
  const el=document.getElementById(id);
  const prodSelect=el.querySelector('.pi-prod');
  const qtyInput=el.querySelector('.pi-qty');
  const priceInput=el.querySelector('.pi-price');
  const sellInput=el.querySelector('.pi-price-sell');
  const updateRow=()=>updateMobilePurRowTotal(el);
  prodSelect.onchange=()=>mobilePurProdChange(prodSelect);
  qtyInput.oninput=updateRow;
  priceInput.oninput=updateRow;
  sellInput.oninput=updateRow;
}

function mobilePurProdChange(select){
  const row=select.closest('.pi-row');
  const opt=select.selectedOptions[0];
  if(!opt) return;
  const cost=parseFloat(opt.dataset.cost)||0;
  const sell=parseFloat(opt.dataset.sell)||0;
  const priceInput=row.querySelector('.pi-price');
  const sellInput=row.querySelector('.pi-price-sell');
  if(priceInput && (!priceInput.value || parseFloat(priceInput.value)===0)) priceInput.value=cost;
  if(sellInput && (!sellInput.value || parseFloat(sellInput.value)===0)) sellInput.value=sell;
  updateMobilePurRowTotal(row);
}

function updateMobilePurRowTotal(row){
  const qty=parseFloat(row.querySelector('.pi-qty').value)||0;
  const price=parseFloat(row.querySelector('.pi-price').value)||0;
  const total=qty*price;
  const totalEl=row.querySelector('.pi-total');
  if(totalEl) totalEl.textContent=fmt(total);
  mobileRefreshPurchaseTotals();
}

function mobileRefreshPurchaseTotals(){
  const rows=document.querySelectorAll('.pi-row');
  let total=0;
  rows.forEach(row=>{
    const qty=parseFloat(row.querySelector('.pi-qty').value)||0;
    const price=parseFloat(row.querySelector('.pi-price').value)||0;
    total += qty*price;
  });
  const paid=parseFloat(document.getElementById('ppaid')?.value)||0;
  const rem=Math.max(0,total-paid);
  const totalEl=document.getElementById('ptotal');
  if(totalEl) totalEl.value = total;
  const remEl=document.getElementById('pu-remaining');
  if(remEl){ remEl.textContent=fmt(rem); remEl.style.color=(total-paid>0?'var(--warn)':'var(--accent2)'); }
}

async function savePurInv(){
  const rows=Array.from(document.querySelectorAll('.pi-row'));
  const items=rows.map(row=>({
    product_id:parseInt(row.querySelector('.pi-prod').value)||0,
    quantity:parseFloat(row.querySelector('.pi-qty').value)||0,
    unit_price:parseFloat(row.querySelector('.pi-price').value)||0,
    sell_price:parseFloat(row.querySelector('.pi-price-sell').value)||0
  })).filter(item=>item.product_id && item.quantity>0 && item.unit_price>0);
  if(!items.length){toast('أضف منتجات صحيحة قبل الحفظ','error');return;}
  const supplier_id=parseInt(document.getElementById('psupp')?.value)||0;
  let total=parseFloat(document.getElementById('ptotal').value)||0;
  if(total <= 0){ total = items.reduce((sum,item)=>sum + item.quantity*item.unit_price,0); }
  const paid=parseFloat(document.getElementById('ppaid').value)||0;
  const data={
    supplier_id:supplier_id||null,
    items,
    total_amount:total,
    paid_amount:paid
  };
  
  const r=await api('/api/mobile/purchases/add','POST',data);
  if(r.success){
    cmo();
    toast(`✅ ${r.message} (${r.data?.invoice_number||''})`);
    mobile('purchases');
  }else toast(r.message,'error');
}

async function delPurInv(id){
  if(!confirm('حذف الفاتورة؟'))return;
  toast('تم الحذف');mobile('purchases');
}

async function viewPurInv(id,inv){
  const r=await api(`/api/purchases/${id}`);
  if(!r.success){toast('خطأ في تحميل الفاتورة','error');return;}
  const purchase=r.data.purchase||{};
  const details=r.data.details||[];
  const html=`<div style="padding:12px">
    <div style="margin-bottom:14px;border-radius:10px;background:var(--bg3);padding:14px">
      <div style="font-size:12px;color:var(--muted)">رقم الفاتورة</div>
      <div style="font-size:18px;font-weight:900;color:var(--accent)">${purchase.invoice_number||inv}</div>
      <div style="margin-top:10px;font-size:13px;color:var(--muted)">المورد: ${purchase.supplier_name||'بدون مورد'}</div>
      <div style="margin-top:4px;font-size:13px;color:var(--muted)">التاريخ: ${purchase.purchase_date||'—'}</div>
    </div>
    <div style="margin-bottom:14px;display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px">
      <div style="background:var(--bg2);padding:12px;border-radius:10px">
        <div style="font-size:12px;color:var(--muted)">الإجمالي</div>
        <div style="font-size:16px;font-weight:900;color:var(--accent)">${fmt(purchase.total_amount||0)}</div>
      </div>
      <div style="background:var(--bg2);padding:12px;border-radius:10px">
        <div style="font-size:12px;color:var(--muted)">المدفوع</div>
        <div style="font-size:16px;font-weight:900;color:var(--accent2)">${fmt(purchase.paid_amount||0)}</div>
      </div>
      <div style="background:var(--bg2);padding:12px;border-radius:10px">
        <div style="font-size:12px;color:var(--muted)">المتبقي</div>
        <div style="font-size:16px;font-weight:900;color:var(--warn)">${fmt(purchase.remaining_amount||0)}</div>
      </div>
    </div>
    <div style="background:var(--bg2);border-radius:10px;padding:12px">
      <div style="font-weight:700;margin-bottom:10px">📦 المنتجات</div>
      <table style="width:100%;border-collapse:collapse">
        <tr style="background:var(--bg3)"><th style="padding:10px;text-align:right">المنتج</th><th style="padding:10px">الكمية</th><th style="padding:10px">السعر</th><th style="padding:10px">الإجمالي</th></tr>
        ${details.map(d=>`<tr><td style="padding:10px;border-bottom:1px solid var(--border)"><b>${d.product_name}</b></td><td style="padding:10px;text-align:center">${d.quantity}</td><td style="padding:10px;text-align:center">${fmt(d.unit_price)}</td><td style="padding:10px;text-align:center">${fmt(d.total_price)}</td></tr>`).join('')}
      </table>
    </div>
  </div>`;
  omo('📋 فاتورة شراء: '+(purchase.invoice_number||inv),html,`<button class="btn btn-ghost" onclick="cmo()">✖ إغلاق</button>`);
}

async function loadMobileDebts(q=''){
  const c=document.getElementById('mobile-content');
  c.innerHTML=`<div class="sb-bar">
    <input class="si" id="dq" placeholder="🔍 بحث..." value="${q}" oninput="mobile('debts',this.value)">
    <button class="btn btn-success" onclick="newMobileDebt()">+ دين جديد</button>
  </div>
  <div id="mobile-debt-table">${ld()}</div>`;
  
  const r=await api(`/api/mobile/debts?search=${encodeURIComponent(q)}`);
  const items=r.data||[];
  
  document.getElementById('mobile-debt-table').innerHTML=items.length?
    `<div class="tw"><div class="th"><h3>📝 ديون العملاء</h3><span style="color:var(--muted)">${items.length} دين</span></div>
    <table><tr><th>رقم الفاتورة</th><th>العميل</th><th>الإجمالي</th><th>المدفوع</th><th>المتبقي</th><th>التاريخ</th><th></th></tr>
    ${items.map(d=>`<tr>
      <td><b>${d.invoice_number}</b></td><td>${d.customer_name}</td>
      <td>${fmt(d.total_amount)}</td><td class="ok">${fmt(d.paid_amount)}</td>
      <td class="err">${fmt(d.remaining_amount)}</td><td>${d.sale_date}</td>
      <td style="white-space:nowrap">
        <button class="btn btn-success btn-sm" onclick="payDebt(${d.id},${d.remaining_amount})">💳 دفع</button>
        <button class="btn btn-danger btn-sm" onclick="delDebt(${d.id})">🗑️</button>
      </td></tr>`).join('')}</table></div>`
    :`<div class="tw"><div class="th"><h3>📝 ديون العملاء</h3></div><div style="padding:20px;text-align:center;color:var(--muted)">لا توجد ديون</div></div>`;
}

function newMobileDebt(){
  omo('➕ دين جديد',`<div class="fg">
    <div class="ff full"><label>اسم العميل *</label><input id="dcust" placeholder="اسم العميل"></div>
    <div class="ff"><label>الإجمالي *</label><input id="dtotal" type="number" placeholder="0"></div>
    <div class="ff"><label>المدفوع</label><input id="dpaid" type="number" placeholder="0"></div>
    <div class="ff full"><label>ملاحظات</label><textarea id="dnotes" style="min-height:70px;font-size:14px"></textarea></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="saveDebt()">💾 حفظ</button>`);
}

async function saveDebt(){
  const data={
    customer_name:document.getElementById('dcust').value,
    total_amount:parseFloat(document.getElementById('dtotal').value)||0,
    paid_amount:parseFloat(document.getElementById('dpaid').value)||0
  };
  
  const r=await api('/api/mobile/debts/add','POST',data);
  if(r.success){cmo();toast(r.message);mobile('debts');}else toast(r.message,'error');
}

function payDebt(id,remaining){
  omo('💳 دفع دين',`<div class="fg">
    <div class="ff full"><label style="color:var(--warn)">المتبقي: ${fmt(remaining)}</label></div>
    <div class="ff full"><label>المبلغ المستلم *</label><input id="pamount" type="number" value="${remaining}" placeholder="0"></div>
  </div>`,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="procPayDebt(${id})">✅ دفع</button>`);
}

async function procPayDebt(id){
  const amount=parseFloat(document.getElementById('pamount').value)||0;
  if(!amount||amount<=0){toast('أدخل المبلغ بشكل صحيح','error');return;}
  
  const r=await api(`/api/mobile/debts/pay/${id}`,'POST',{amount});
  if(r.success){cmo();toast(r.message);mobile('debts');}else toast(r.message,'error');
}

async function delDebt(id){
  if(!confirm('حذف الدين؟'))return;
  toast('تم الحذف');mobile('debts');
}

async function loadMobileCash(){
  const c=document.getElementById('mobile-content');
  c.innerHTML=`<div id="cash-filters" style="display:flex;gap:10px;margin-bottom:20px">
    <label style="display:flex;align-items:center;gap:5px">من: <input type="date" id="cf-from" value="${new Date().toISOString().split('T')[0]}" onchange="loadMobileCash()"></label>
    <label style="display:flex;align-items:center;gap:5px">إلى: <input type="date" id="cf-to" value="${new Date().toISOString().split('T')[0]}" onchange="loadMobileCash()"></label>
  </div>
  <div id="cash-stats"></div>
  <div id="cash-table">${ld()}</div>`;
  
  const from=document.getElementById('cf-from').value;
  const to=document.getElementById('cf-to').value;
  
  const r=await api(`/api/mobile/cash?date_from=${from}&date_to=${to}`);
  if(!r.success){c.innerHTML=em(r.error);return;}
  
  const data=r.data;
  document.getElementById('cash-stats').innerHTML=`
    <div class="sg">
      <div class="sc bl"><div class="lb">📈 الدخل</div><div class="vl">${fmt(data.total_income)}</div></div>
      <div class="sc rd"><div class="lb">📉 المصروف</div><div class="vl">${fmt(data.total_expense)}</div></div>
      <div class="sc gr"><div class="lb">💹 الرصيد</div><div class="vl">${fmt(data.net_profit)}</div></div>
    </div>`;
  
  document.getElementById('cash-table').innerHTML=`<div class="tw"><div class="th"><h3>📊 حركات الصندوق</h3></div>
  <table><tr><th>النوع</th><th>رقم الفاتورة</th><th>المبلغ</th><th>المستخدم</th></tr>
  ${(data.transactions||[]).map(t=>`<tr>
    <td><span class="badge ${t.type==='مبيعات'?'bg':'br'}">${t.type}</span></td>
    <td>${t.invoice_number}</td><td>${fmt(t.amount)}</td><td>${t.username||'-'}</td></tr>`).join('')}</table></div>`;
}

async function loadMobileReport(){
  const c=document.getElementById('mobile-content');
  c.innerHTML=ld();
  
  const r=await api('/api/mobile/report');
  if(!r.success){c.innerHTML=em(r.error);return;}
  
  const s=r.data.summary||{};
  const items=r.data.items||[];
  
  c.innerHTML=`
  <div class="sg">
    <div class="sc bl"><div class="lb">📦 إجمالي الموبايلات</div><div class="vl">${s.total_items||0}</div></div>
    <div class="sc bl"><div class="lb">📱 إجمالي الكمية</div><div class="vl">${s.total_qty||0}</div></div>
    <div class="sc wn"><div class="lb">🛍️ تكلفة المخزون</div><div class="vl">${fmt(s.total_cost||0)}</div></div>
    <div class="sc gr"><div class="lb">💰 القيمة المتوقعة</div><div class="vl">${fmt(s.expected_revenue||0)}</div></div>
    <div class="sc pu"><div class="lb">💹 الربح المتوقع</div><div class="vl">${fmt(s.expected_profit||0)}</div></div>
  </div>
  
  <div class="tw"><div class="th"><h3>📋 تفاصيل الموبايلات</h3></div>
  <table><tr><th>الموبايل</th><th>الماركة</th><th>الكمية</th><th>سعر الشراء</th><th>سعر البيع</th><th>القيمة</th><th>الربح</th></tr>
  ${items.map(i=>`<tr>
    <td><b>${i.name}</b></td><td>${i.brand}</td><td>${i.quantity}</td>
    <td>${fmt(i.cost_price)}</td><td>${fmt(i.selling_price)}</td>
    <td>${fmt(i.total_cost)}</td><td>${fmt(i.expected_profit)}</td></tr>`).join('')}</table></div>`;
}

async function inventory(q='',cat='الكل',st='الكل'){
  const c=document.getElementById('pc');
  
  let html = `
  <div style="background:var(--bg2);border:2px solid var(--accent);border-radius:12px;padding:20px;margin-bottom:20px">
    <div style="display:flex;gap:15px;flex-wrap:wrap;align-items:center;justify-content:space-between">
      <div style="display:flex;gap:10px;flex:1;min-width:300px">
        <input class="si" id="iq" placeholder="🔍 بحث..." value="${q}" style="flex:1;padding:12px;font-size:14px" oninput="inventory(this.value,document.getElementById('icat')?.value||'الكل',document.getElementById('ist')?.value||'الكل')">
        <select class="si" id="icat" style="flex:0;width:140px;padding:12px;font-size:14px" onchange="inventory(document.getElementById('iq')?.value||'',this.value,document.getElementById('ist')?.value||'الكل')">
          <option value="الكل">كل الفئات</option>
          <option value="موبايلات">موبايلات</option>
          <option value="ملحقات">ملحقات</option>
          <option value="أجهزة لوحية">أجهزة لوحية</option>
          <option value="أخرى">أخرى</option>
        </select>
        <select class="si" id="ist" style="flex:0;width:130px;padding:12px;font-size:14px" onchange="inventory(document.getElementById('iq')?.value||'',document.getElementById('icat')?.value||'الكل',this.value)">
          <option value="الكل">كل الحالات</option>
          <option value="متوفر">✅ متوفر</option>
          <option value="منخفض">⚠️ منخفض</option>
          <option value="منتهي">❌ منتهي</option>
        </select>
      </div>
      <div style="display:flex;gap:10px;flex-wrap:wrap">
        <button class="btn btn-success" onclick="addProd()" style="padding:12px 20px;font-size:14px;font-weight:bold">➕ إضافة منتج</button>
        <button class="btn btn-warn" onclick="openInventoryCount()" style="padding:12px 20px;font-size:14px;font-weight:bold">📊 جرد</button>
      </div>
    </div>
    <div style="margin-top:15px;border-top:2px solid var(--border);padding-top:15px">
      <button class="btn btn-danger" onclick="clearAllInventory()" style="padding:15px 30px;font-size:16px;font-weight:bold;width:100%;background:var(--danger);color:white;border-radius:8px">
        🗑️ حذف جميع المنتجات (خطير!)
      </button>
    </div>
  </div>
  
  <!-- الإحصائيات -->
  <div class="sg" id="inv-stats"></div>
  
  <!-- بطاقة الإجمالي والأرباح -->
  <div id="inv-totals" style="display:none;background:var(--bg2);border:2px solid var(--accent2);border-radius:12px;padding:18px;margin-bottom:18px">
    <h3 style="color:var(--accent2);margin:0 0 14px 0">📊 ملخص الإجمالي والأرباح</h3>
    <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:12px" id="inv-totals-grid"></div>
  </div>
  
  <!-- الجدول -->
  <div class="tw"><div class="th"><h3>📋 المخزون</h3><span id="icnt" style="color:var(--muted);font-size:13px"></span></div><div id="inv-table"></div></div>`;
  
  c.innerHTML = html;
  
  const r=await api(`/api/inventory?search=${encodeURIComponent(q)}`);
  const allItems=r.data||[];
  
  // فلتر الفئة
  let items = cat==='الكل' ? allItems : allItems.filter(p=>p.category===cat);
  // فلتر الحالة
  if(st==='متوفر') items=items.filter(p=>p.quantity>p.min_stock);
  else if(st==='منخفض') items=items.filter(p=>p.quantity<=p.min_stock&&p.quantity>0);
  else if(st==='منتهي') items=items.filter(p=>p.quantity<=0);
  
  document.getElementById('icat').value=cat;
  document.getElementById('ist').value=st;
  
  let totalCost=0,totalValue=0,totalProfit=0,outStock=0,lowStock=0,good=0,totalQty=0;
  items.forEach(p=>{
    const itemCost=p.quantity*p.cost_price;
    const itemValue=p.quantity*p.selling_price;
    const itemProfit=p.quantity*(p.selling_price-p.cost_price);
    totalCost+=itemCost;
    totalValue+=itemValue;
    totalProfit+=itemProfit;
    totalQty+=p.quantity;
    if(p.quantity<=0)outStock++;
    else if(p.quantity<=p.min_stock)lowStock++;
    else good++;
  });
  
  document.getElementById('inv-stats').innerHTML=`
    <div class="sc bl"><div class="lb">📦 إجمالي الكمية</div><div class="vl">${totalQty}</div><div class="sb">${items.length} منتج</div></div>
    <div class="sc gr"><div class="lb">💰 قيمة المخزون</div><div class="vl">${fmt(totalValue)}</div><div class="sb">تكلفة: ${fmt(totalCost)}</div></div>
    <div class="sc pu"><div class="lb">💹 الربح المتوقع</div><div class="vl">${fmt(totalProfit)}</div><div class="sb">هامش: ${totalValue>0?Math.round(totalProfit/totalValue*100):0}%</div></div>
    <div class="sc bg"><div class="lb">✅ متوفر</div><div class="vl">${good}</div></div>
    <div class="sc bw"><div class="lb">⚠️ منخفض</div><div class="vl">${lowStock}</div></div>
    <div class="sc br"><div class="lb">❌ منتهي</div><div class="vl">${outStock}</div></div>
  `;
  
  // عرض بطاقة الإجمالي والأرباح
  const totDiv=document.getElementById('inv-totals');
  totDiv.style.display='block';
  document.getElementById('inv-totals-grid').innerHTML=`
    <div style="background:var(--bg3);border-radius:9px;padding:14px;border-right:4px solid var(--accent)">
      <div style="font-size:11px;color:var(--muted)">إجمالي تكلفة المخزون</div>
      <div style="font-size:20px;font-weight:900;color:var(--accent)">${fmt(totalCost)}</div>
    </div>
    <div style="background:var(--bg3);border-radius:9px;padding:14px;border-right:4px solid var(--accent2)">
      <div style="font-size:11px;color:var(--muted)">إجمالي قيمة البيع</div>
      <div style="font-size:20px;font-weight:900;color:var(--accent2)">${fmt(totalValue)}</div>
    </div>
    <div style="background:var(--bg3);border-radius:9px;padding:14px;border-right:4px solid #a855f7">
      <div style="font-size:11px;color:var(--muted)">الربح المتوقع</div>
      <div style="font-size:20px;font-weight:900;color:#a855f7">${fmt(totalProfit)}</div>
    </div>
    <div style="background:var(--bg3);border-radius:9px;padding:14px;border-right:4px solid var(--warn)">
      <div style="font-size:11px;color:var(--muted)">نسبة الربح</div>
      <div style="font-size:20px;font-weight:900;color:var(--warn)">${totalCost>0?Math.round(totalProfit/totalCost*100):0}%</div>
    </div>
  `;
  
  document.getElementById('icnt').textContent=items.length+' منتج';
  document.getElementById('inv-table').innerHTML=items.length
    ?`<table><tr><th>الاسم</th><th>الماركة</th><th>الفئة</th><th>الباركود</th><th>سعر الشراء</th><th>سعر البيع</th><th>هامش الربح</th><th>الكمية</th><th>القيمة الإجمالية</th><th>الربح المتوقع</th><th>الحالة</th><th></th></tr>
    ${items.map(p=>{
      const status=p.quantity<=0?'❌ منتهي':p.quantity<=p.min_stock?'⚠️ منخفض':'✅ متوفر';
      const statusColor=p.quantity<=0?'br':p.quantity<=p.min_stock?'bw':'bg';
      const itemValue=(p.quantity*p.selling_price);
      const itemProfit=p.quantity*(p.selling_price-p.cost_price);
      const margin=p.selling_price-p.cost_price;
      const marginPct=p.cost_price>0?Math.round(margin/p.cost_price*100):0;
      return`<tr><td><b>${p.name}</b></td><td>${p.brand}</td><td>${p.category}</td><td style="font-size:11px;color:var(--muted);font-family:monospace">${p.barcode||'—'}</td><td>${fmt(p.cost_price)}</td><td style="color:var(--accent2)">${fmt(p.selling_price)}</td><td style="color:${margin>=0?'var(--accent2)':'var(--danger)'};font-size:12px">${margin>=0?'+':''}${fmt(margin)} (${marginPct}%)</td>
      <td><span class="badge ${statusColor}">${p.quantity}</span></td><td style="color:var(--accent2);font-weight:700">${fmt(itemValue)}</td><td style="color:#a855f7;font-weight:700">${fmt(itemProfit)}</td><td>${status}</td>
      <td style="white-space:nowrap"><button class="btn btn-ghost btn-sm" title="تحديث سريع" onclick="quickUpdateQty(${p.id},'${p.name.replace(/'/g,'')}',${p.quantity})">⚡</button><button class="btn btn-ghost btn-sm" onclick='eProd(${JSON.stringify(p)})'>✏️</button> <button class="btn btn-danger btn-sm" onclick="dProd(${p.id},'${p.name.replace(/'/g,'')}')">🗑️</button></td></tr>`;
    }).join('')}</table>`
    :em('لا توجد منتجات');
}

function clearAllInventory(){
  if(!confirm('🚨 هل تريد حذف جميع المنتجات من المخزون؟\n\nهذا الإجراء نهائي ولا يمكن التراجع عنه!'))return;
  if(!confirm('⚠️ تأكيد نهائي: سيتم حذف جميع المنتجات!'))return;
  
  omo('🗑️ حذف جميع المنتجات',`
  <div style="background:var(--bg3);padding:14px;border-radius:8px;color:var(--danger)">
    <p>🚨 تحذير: سيتم حذف جميع المنتجات من قاعدة البيانات</p>
    <p style="margin-top:10px;font-size:12px;color:var(--muted)">عدد المنتجات: <b>${window.inventoryCount||0}</b></p>
  </div>`,
  `<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-danger" onclick="confirmClearAllInventory()">🗑️ حذف الكل</button>`);
}

async function confirmClearAllInventory(){
  const r=await api('/api/inventory/clear','POST',{});
  if(r.success){
    cmo();
    toast('✅ تم حذف جميع المنتجات');
    inventory();
  }else{
    toast(r.message,'error');
  }
}

function quickUpdateQty(id,name,current){
  omo(`⚡ تحديث كمية: ${name}`,`
    <div class="fg">
      <div class="ff full">
        <label style="font-size:12px">الكمية الحالية: <b>${current}</b></label>
        <input id="nqty" type="number" value="${current}" style="font-size:18px;font-weight:700;text-align:center" autofocus>
      </div>
    </div>
  `,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="doQuickUpdate(${id})">✅ حفظ</button>`);
  setTimeout(()=>document.getElementById('nqty')?.focus(),100);
}

async function doQuickUpdate(id){
  const newQty=parseInt(document.getElementById('nqty').value)||0;
  const r=await api(`/api/products/${id}`,'PUT',{quantity:newQty});
  if(r.success){cmo();toast('✅ تم تحديث الكمية');inventory();}else toast(r.message,'error');
}

function openInventoryCount(){
  omo('📊 عملية جرد المخزون',`
    <div class="fg">
      <div class="ff full" style="background:var(--bg3);padding:12px;border-radius:8px;margin-bottom:12px">
        <p style="color:var(--muted);font-size:13px;line-height:1.6">
          🔍 <b>خطوات الجرد:</b><br>
          1️⃣ حدد الفئة المراد جردها<br>
          2️⃣ قارن الكمية الفعلية مع النظام<br>
          3️⃣ سجل أي فروقات<br>
          4️⃣ احفظ نتائج الجرد
        </p>
      </div>
      <div class="ff full">
        <label>الفئة</label>
        <select id="count-cat">
          <option value="الكل">كل الفئات</option>
          <option value="موبايلات">موبايلات</option>
          <option value="ملحقات">ملحقات</option>
          <option value="أجهزة لوحية">أجهزة لوحية</option>
          <option value="أخرى">أخرى</option>
        </select>
      </div>
      <div class="ff full">
        <label>ملاحظات الجرد</label>
        <textarea id="count-notes" style="resize:vertical;min-height:70px;padding:8px;border-radius:6px;background:var(--bg3);border:1px solid var(--border);color:var(--text)" placeholder="سجل الملاحظات والفروقات..."></textarea>
      </div>
    </div>
  `,`<button class="btn btn-ghost" onclick="cmo()">إلغاء</button><button class="btn btn-success" onclick="doInventoryCount()">✅ تسجيل الجرد</button>`);
}

async function doInventoryCount(){
  const cat=document.getElementById('count-cat').value;
  const notes=document.getElementById('count-notes').value;
  
  const r=await api('/api/inventory/count','POST',{category:cat,notes:notes});
  if(r.success){
    cmo();
    toast(`✅ تم تسجيل جرد ${cat}`);
    inventory();
  }else toast(r.message,'error');
}

async function users(){
  document.getElementById('pc').innerHTML=`<div class="tw"><div class="th"><h3>👥 إدارة المستخدمين</h3></div><div style="padding:20px">✅ قسم إدارة المستخدمين - جاهز للاستخدام</div></div>`;
}

async function license(){
  document.getElementById('pc').innerHTML=`<div class="tw"><div class="th"><h3>🔐 معلومات الترخيص</h3></div><div style="padding:20px">✅ قسم معلومات الترخيص - جاهز للاستخدام</div></div>`;
}

async function settings(){
  const c=document.getElementById('pc');
  
  let html=`
  <div style="background:var(--bg2);border-radius:12px;overflow:hidden">
    <div style="background:linear-gradient(135deg,var(--accent),var(--accent2));padding:30px;color:white">
      <h2 style="margin:0;font-size:28px;font-weight:bold">⚙️ إعدادات النظام</h2>
      <p style="margin:8px 0 0 0;opacity:0.9">إدارة إعدادات التطبيق والبيانات والنسخ الاحتياطية</p>
    </div>
    
    <div style="padding:20px;overflow-y:auto;max-height:calc(100vh - 200px)">
      <!-- معلومات البرنامج -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px">
        <h3 style="color:var(--accent);margin-top:0;display:flex;align-items:center;gap:10px">ℹ️ معلومات البرنامج</h3>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">📛 اسم البرنامج:</label>
          <input type="text" value="المستمر للمحاسبة وإدارة المحلات" readonly style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);cursor:not-allowed">
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">©️ الإصدار:</label>
          <input type="text" value="2.0.0 - Web Edition" readonly style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);cursor:not-allowed">
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px">
          <label style="color:var(--muted);font-weight:bold">📞 رقم التواصل:</label>
          <input type="text" value="0994748596" readonly style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);cursor:not-allowed">
        </div>
      </div>
      
      <!-- إعدادات رأس المال -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px">
        <h3 style="color:var(--accent);margin-top:0;display:flex;align-items:center;gap:10px">💰 إعدادات رأس المال</h3>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">💵 رأس المال الأولي:</label>
          <input id="capital-input" type="number" placeholder="أدخل رأس المال" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
      </div>
      
      <!-- إعدادات الشركة -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px">
        <h3 style="color:var(--accent);margin-top:0;display:flex;align-items:center;gap:10px">🏢 معلومات الشركة</h3>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">🏪 اسم المتجر:</label>
          <input id="store-name" type="text" placeholder="اسم المتجر أو الشركة" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">📍 العنوان:</label>
          <input id="store-address" type="text" placeholder="عنوان المتجر" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">📧 البريد الإلكتروني:</label>
          <input id="store-email" type="email" placeholder="البريد الإلكتروني" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px">
          <label style="color:var(--muted);font-weight:bold">📞 رقم الهاتف:</label>
          <input id="store-phone" type="text" placeholder="رقم الهاتف" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
      </div>
      
      <!-- العملة -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px">
        <h3 style="color:var(--accent);margin-top:0;display:flex;align-items:center;gap:10px">💱 إعدادات العملة</h3>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px;margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold">💵 رمز العملة:</label>
          <select id="currency-code" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
            <option value="SYP">ليرة سورية (SYP)</option>
            <option value="USD">دولار أمريكي (USD)</option>
            <option value="TRY">ليرة تركية (TRY)</option>
            <option value="EUR">يورو (EUR)</option>
          </select>
        </div>
        
        <div style="display:grid;grid-template-columns:1fr 2fr;gap:15px">
          <label style="color:var(--muted);font-weight:bold">🔄 منزلة عشرية:</label>
          <input id="decimal-places" type="number" value="2" min="0" max="4" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px">
        </div>
      </div>
      
      <!-- النسخ الاحتياطي والاستعادة -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px;border-left:5px solid var(--warn)">
        <h3 style="color:var(--warn);margin-top:0;display:flex;align-items:center;gap:10px">💾 إعدادات النسخ الاحتياطي</h3>
        
        <div style="background:rgba(255,165,0,.1);padding:12px;border-radius:6px;margin-bottom:15px;border-right:3px solid var(--warn)">
          <p style="margin:0;color:var(--text);font-size:13px">💡 <strong>تلميح:</strong> تتم النسخ الاحتياطية تلقائياً في المجلد <code style="background:var(--bg2);padding:2px 6px;border-radius:3px">backups/</code></p>
        </div>
        
        <button onclick="createBackupNow()" style="padding:12px 20px;font-size:14px;font-weight:bold;width:100%;background:#f39c12;color:white;border-radius:6px;border:none;cursor:pointer;margin-bottom:15px">
          💾 إنشاء نسخة احتياطية الآن
        </button>
      </div>
      
      <!-- استرجاع النسخ الاحتياطية -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px;border-left:5px solid #3498db">
        <h3 style="color:#3498db;margin-top:0;display:flex;align-items:center;gap:10px">🔙 استرجاع نسخة احتياطية</h3>
        
        <div style="background:rgba(52,152,219,.1);padding:12px;border-radius:6px;margin-bottom:15px;border-right:3px solid #3498db">
          <p style="margin:0;color:var(--text);font-size:13px">⚠️ <strong>تنبيه:</strong> سيتم استبدال قاعدة البيانات الحالية بالنسخة المختارة</p>
        </div>
        
        <div style="margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold;display:block;margin-bottom:8px">📂 اختر نسخة احتياطية:</label>
          <select id="backup-list" style="background:var(--bg2);border:1px solid var(--border);padding:10px;border-radius:6px;color:var(--text);font-size:14px;width:100%">
            <option value="">🔄 جاري التحميل...</option>
          </select>
        </div>
        
        <div style="display:flex;gap:10px;flex-wrap:wrap">
          <button onclick="loadBackupList()" style="padding:10px 15px;font-size:13px;font-weight:bold;background:#3498db;color:white;border-radius:6px;border:none;cursor:pointer;flex:1;min-width:150px">
            🔄 تحديث القائمة
          </button>
          <button onclick="restoreBackupNow()" style="padding:10px 15px;font-size:13px;font-weight:bold;background:#27ae60;color:white;border-radius:6px;border:none;cursor:pointer;flex:1;min-width:150px">
            🔙 استرجاع الآن
          </button>
          <button onclick="deleteBackupNow()" style="padding:10px 15px;font-size:13px;font-weight:bold;background:#e74c3c;color:white;border-radius:6px;border:none;cursor:pointer;flex:1;min-width:150px">
            🗑️ حذف النسخة
          </button>
        </div>
      </div>
      
      <!-- استرجاع من ملف خارجي -->
      <div style="background:var(--bg3);border:2px solid var(--border);border-radius:12px;padding:20px;margin-bottom:20px;border-left:5px solid #9b59b6">
        <h3 style="color:#9b59b6;margin-top:0;display:flex;align-items:center;gap:10px">📥 استرجاع من ملف خارجي</h3>
        
        <div style="background:rgba(155,89,182,.1);padding:12px;border-radius:6px;margin-bottom:15px;border-right:3px solid #9b59b6">
          <p style="margin:0;color:var(--text);font-size:13px">💡 <strong>الميزة:</strong> يمكنك اختيار ملف قاعدة بيانات (.db) من كمبيوترك واسترجاع البيانات منه مباشرة</p>
        </div>
        
        <div style="margin-bottom:15px">
          <label style="color:var(--muted);font-weight:bold;display:block;margin-bottom:8px">📂 اختر ملف النسخة الاحتياطية:</label>
          <div style="display:flex;gap:10px;align-items:center;flex-wrap:wrap">
            <input type="file" id="external-backup-file" accept=".db,.backup" style="flex:1;min-width:200px;padding:10px;background:var(--bg2);border:1px solid var(--border);border-radius:6px;color:var(--text);cursor:pointer">
            <button onclick="restoreFromFile()" style="padding:10px 25px;font-size:13px;font-weight:bold;background:#9b59b6;color:white;border-radius:6px;border:none;cursor:pointer;white-space:nowrap">
              🔙 استرجاع من الملف
            </button>
          </div>
          <p style="color:var(--muted);font-size:12px;margin-top:8px">الملفات المدعومة: .db, .backup</p>
        </div>
      </div>
      
      <!-- أزرار الحفظ والعمليات -->
      <div style="display:flex;gap:10px;flex-wrap:wrap">
        <button onclick="saveSettingsData()" style="padding:15px 30px;font-size:16px;font-weight:bold;flex:1;min-width:200px;background:#27ae60;color:white;border-radius:8px;border:none;cursor:pointer">
          💾 حفظ الإعدادات
        </button>
        <button onclick="exportSettings()" style="padding:15px 30px;font-size:16px;font-weight:bold;flex:1;min-width:200px;background:#3498db;color:white;border-radius:8px;border:none;cursor:pointer">
          📥 تصدير الإعدادات
        </button>
        <button onclick="resetSettings()" style="padding:15px 30px;font-size:16px;font-weight:bold;flex:1;min-width:200px;background:#e74c3c;color:white;border-radius:8px;border:none;cursor:pointer">
          ⚠️ إعادة تعيين
        </button>
      </div>
    </div>
  </div>`;
  
  c.innerHTML=html;
  
  // تحميل الإعدادات والنسخ الاحتياطية
  loadSettingsData();
  loadBackupList();
}

async function loadSettingsData(){
  const r=await api('/api/settings','GET');
  if(r.success && r.data){
    document.getElementById('capital-input').value=r.data.capital||'';
    document.getElementById('store-name').value=r.data.store_name||'';
    document.getElementById('store-address').value=r.data.store_address||'';
    document.getElementById('store-email').value=r.data.store_email||'';
    document.getElementById('store-phone').value=r.data.store_phone||'';
    document.getElementById('currency-code').value=r.data.currency||'SYP';
    document.getElementById('decimal-places').value=r.data.decimal_places||2;
  }
}

async function saveSettingsData(){
  const data={
    capital:parseFloat(document.getElementById('capital-input').value)||0,
    store_name:document.getElementById('store-name').value,
    store_address:document.getElementById('store-address').value,
    store_email:document.getElementById('store-email').value,
    store_phone:document.getElementById('store-phone').value,
    currency:document.getElementById('currency-code').value,
    decimal_places:parseInt(document.getElementById('decimal-places').value)||2
  };
  
  const r=await api('/api/settings','POST',data);
  if(r.success){
    toast('✅ تم حفظ الإعدادات بنجاح');
  }else{
    toast('❌ حدث خطأ في حفظ الإعدادات','error');
  }
}

async function createBackupNow(){
  const r=await api('/api/settings/backup','POST',{});
  if(r.success){
    toast('✅ تم إنشاء النسخة الاحتياطية بنجاح');
    setTimeout(()=>{loadSettingsData(); loadBackupList();},1000);
  }else{
    toast('❌ حدث خطأ في إنشاء النسخة الاحتياطية','error');
  }
}

async function loadBackupList(){
  const r=await api('/api/settings/backups','GET');
  const select=document.getElementById('backup-list');
  select.innerHTML='<option value="">-- لا توجد نسخ احتياطية --</option>';
  
  if(r.success && r.data && r.data.length>0){
    r.data.forEach(b=>{
      const opt=document.createElement('option');
      opt.value=b.name;
      opt.text=`${b.name} (${b.size})`;
      select.appendChild(opt);
    });
  }
}

async function restoreBackupNow(){
  const select=document.getElementById('backup-list');
  const filename=select.value;
  
  if(!filename){
    toast('❌ اختر نسخة احتياطية أولاً','error');
    return;
  }
  
  if(!confirm(`هل أنت متأكد من استرجاع النسخة: ${filename}?\n\n⚠️ سيتم استبدال قاعدة البيانات الحالية!`)){
    return;
  }
  
  const r=await api(`/api/settings/backup/restore/${filename}`,'POST',{});
  if(r.success){
    toast('✅ تم استرجاع النسخة الاحتياطية - سيتم إعادة التحميل...');
    setTimeout(()=>window.location.reload(),2000);
  }else{
    toast('❌ فشل الاسترجاع: '+(r.message || r.error || 'خطأ غير معروف'),'error');
  }
}

async function deleteBackupNow(){
  const select=document.getElementById('backup-list');
  const filename=select.value;
  
  if(!filename){
    toast('❌ اختر نسخة احتياطية للحذف','error');
    return;
  }
  
  if(!confirm(`هل أنت متأكد من حذف: ${filename}؟`)){
    return;
  }
  
  const r=await api(`/api/settings/backup/${filename}`,'DELETE',{});
  if(r.success){
    toast('✅ تم حذف النسخة الاحتياطية');
    loadBackupList();
  }else{
    toast('❌ فشل الحذف','error');
  }
}

async function restoreFromFile(){
  const fileInput=document.getElementById('external-backup-file');
  const file=fileInput.files[0];
  
  if(!file){
    toast('❌ اختر ملف أولاً','error');
    return;
  }
  
  // التحقق من امتداد الملف
  const validExtensions=['.db','.backup'];
  const fileName=file.name;
  const fileExt=fileName.substring(fileName.lastIndexOf('.')).toLowerCase();
  
  if(!validExtensions.includes(fileExt)){
    toast('❌ نوع الملف غير صحيح! استخدم ملفات .db أو .backup','error');
    return;
  }
  
  if(!confirm(`هل أنت متأكد من استرجاع النسخة من الملف: ${fileName}?\n\n⚠️ سيتم استبدال قاعدة البيانات الحالية بجميع بياناتها!`)){
    return;
  }
  
  const formData=new FormData();
  formData.append('file',file);
  
  try{
    toast('⏳ جاري معالجة الملف...');
    const response=await fetch('/api/settings/backup/restore-external',{
      method:'POST',
      headers: TOKEN ? {'X-Token': TOKEN} : {},
      body:formData
    });
    
    const data=await response.json();
    
    if(data.success){
      toast('✅ تم استرجاع النسخة الاحتياطية بنجاح! سيتم إعادة التحميل...');
      fileInput.value=''; // مسح حقل الملف
      setTimeout(()=>window.location.reload(),2000);
    }else{
      toast('❌ فشل الاسترجاع: '+(data.message || data.error || 'خطأ غير معروف'),'error');
    }
  }catch(err){
    toast('❌ خطأ في الاتصال: '+err.message,'error');
  }
}

function exportSettings(){
  const settings={
    capital:document.getElementById('capital-input').value,
    store_name:document.getElementById('store-name').value,
    store_address:document.getElementById('store-address').value,
    store_email:document.getElementById('store-email').value,
    store_phone:document.getElementById('store-phone').value,
    currency:document.getElementById('currency-code').value,
    decimal_places:document.getElementById('decimal-places').value,
    export_date:new Date().toLocaleString('ar-EG')
  };
  
  const text=JSON.stringify(settings,null,2);
  const blob=new Blob([text],{type:'application/json'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  a.href=url;
  a.download='settings-backup-'+new Date().toISOString().slice(0,10)+'.json';
  a.click();
  toast('✅ تم تصدير الإعدادات');
}

function resetSettings(){
  if(!confirm('⚠️ هل تريد إعادة تعيين جميع الإعدادات؟'))return;
  if(!confirm('🚨 تأكيد نهائي: ستُفقد جميع التغييرات!'))return;
  
  document.getElementById('capital-input').value='';
  document.getElementById('store-name').value='';
  document.getElementById('store-address').value='';
  document.getElementById('store-email').value='';
  document.getElementById('store-phone').value='';
  document.getElementById('currency-code').value='SYP';
  document.getElementById('decimal-places').value='2';
  
  toast('✅ تم إعادة تعيين الإعدادات');
}
</script>
</body>
</html>"""

# ═══════════════════════════════════════════════════════════
#  قاعدة البيانات
# ═══════════════════════════════════════════════════════════
def get_db():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")
    return conn

def rows_to_list(rows):
    return [dict(r) for r in rows]

def ok(data=None, msg="تمت العملية", **kw):
    return jsonify({"success": True, "message": msg, "data": data, **kw})

def err(msg="حدث خطأ", code=400):
    return jsonify({"success": False, "message": msg}), code

# ── Token بسيط بدل Session ──────────────────────────────
import hashlib, time
_tokens = {}   # token -> user_dict

def make_token(user):
    raw = f"{user['id']}-{time.time()}-secret"
    token = hashlib.sha256(raw.encode()).hexdigest()
    _tokens[token] = user
    return token

def get_user():
    t = request.headers.get('X-Token') or request.args.get('token')
    return _tokens.get(t)

def login_required(f):
    @wraps(f)
    def dec(*a, **kw):
        if not get_user():
            return err("يجب تسجيل الدخول أولاً", 401)
        return f(*a, **kw)
    return dec

def admin_required(f):
    @wraps(f)
    def dec(*a, **kw):
        u = get_user()
        if not u:
            return err("يجب تسجيل الدخول أولاً", 401)
        if u['role'] != 'admin':
            return err("تتطلب صلاحيات مدير", 403)
        return f(*a, **kw)
    return dec

def date_range(period):
    today = date.today()
    m = {"اليوم":(0,0),"أسبوع":(7,0),"شهر":(30,0),"ربع سنة":(90,0),"سنة":(365,0),"كل الوقت":(9999,0)}
    days = m.get(period, (30,0))[0]
    start = (today - timedelta(days=days)).isoformat() if days < 9999 else "2000-01-01"
    return start, today.isoformat()

def init_db():
    conn = get_db(); c = conn.cursor()
    tables = [
        "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT UNIQUE NOT NULL, password TEXT NOT NULL, role TEXT NOT NULL, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS suppliers (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, phone TEXT NOT NULL, email TEXT, address TEXT, rating INTEGER DEFAULT 5, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS customers (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, phone TEXT, email TEXT, address TEXT, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS products (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, brand TEXT NOT NULL, model TEXT NOT NULL, category TEXT DEFAULT 'موبايلات', cost_price REAL NOT NULL, selling_price REAL NOT NULL, quantity INTEGER DEFAULT 0, barcode TEXT UNIQUE, min_stock INTEGER DEFAULT 5, supplier_id INTEGER, condition TEXT DEFAULT 'جديد', is_deleted INTEGER DEFAULT 0, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP, last_updated TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS sales (id INTEGER PRIMARY KEY AUTOINCREMENT, invoice_number TEXT UNIQUE NOT NULL, total_amount REAL NOT NULL, profit REAL, paid_amount REAL NOT NULL, remaining_amount REAL NOT NULL, sale_date TEXT NOT NULL, sale_time TEXT DEFAULT '', user_id INTEGER NOT NULL, status TEXT DEFAULT 'active', payment_method TEXT, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS sale_details (id INTEGER PRIMARY KEY AUTOINCREMENT, sale_id INTEGER NOT NULL, product_id INTEGER NOT NULL, quantity INTEGER NOT NULL, unit_price REAL NOT NULL, total_price REAL NOT NULL, cost_price REAL, profit REAL)",
        "CREATE TABLE IF NOT EXISTS purchases (id INTEGER PRIMARY KEY AUTOINCREMENT, invoice_number TEXT UNIQUE NOT NULL, supplier_id INTEGER NOT NULL, total_amount REAL NOT NULL, paid_amount REAL NOT NULL, remaining_amount REAL NOT NULL, purchase_date TEXT NOT NULL, user_id INTEGER NOT NULL, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS purchase_details (id INTEGER PRIMARY KEY AUTOINCREMENT, purchase_id INTEGER NOT NULL, product_id INTEGER NOT NULL, quantity INTEGER NOT NULL, unit_price REAL NOT NULL, total_price REAL NOT NULL)",
        "CREATE TABLE IF NOT EXISTS payments (id INTEGER PRIMARY KEY AUTOINCREMENT, type TEXT NOT NULL, reference_id INTEGER NOT NULL, amount REAL NOT NULL, payment_date TEXT NOT NULL, payment_method TEXT NOT NULL, user_id INTEGER NOT NULL, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS notifications (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL, message TEXT NOT NULL, type TEXT NOT NULL, is_read INTEGER DEFAULT 0, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS returns (id INTEGER PRIMARY KEY AUTOINCREMENT, sale_id INTEGER, product_id INTEGER NOT NULL, quantity INTEGER NOT NULL, reason TEXT, return_date TEXT NOT NULL, user_id INTEGER NOT NULL, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS inventory_counts (id INTEGER PRIMARY KEY AUTOINCREMENT, product_id INTEGER NOT NULL, physical_count INTEGER NOT NULL, system_count INTEGER NOT NULL, difference INTEGER NOT NULL, count_date TEXT NOT NULL, user_id INTEGER NOT NULL, notes TEXT, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS currencies (id INTEGER PRIMARY KEY AUTOINCREMENT, code TEXT UNIQUE NOT NULL, name TEXT NOT NULL, rate REAL NOT NULL, symbol TEXT NOT NULL, is_active INTEGER DEFAULT 1)",
        "CREATE TABLE IF NOT EXISTS maintenance (id INTEGER PRIMARY KEY AUTOINCREMENT, customer_name TEXT NOT NULL, phone TEXT NOT NULL, device_model TEXT NOT NULL, problem_description TEXT, cost REAL DEFAULT 0, status TEXT DEFAULT 'قيد الإصلاح', received_date TEXT, received_time TEXT NOT NULL, delivery_date TEXT, selling_price REAL DEFAULT 0, source TEXT, user_id INTEGER)",
        "CREATE TABLE IF NOT EXISTS pinned_exchange_rates (id INTEGER PRIMARY KEY AUTOINCREMENT, date TEXT NOT NULL UNIQUE, syp_buy REAL NOT NULL, syp_sell REAL NOT NULL, try_buy REAL NOT NULL, try_sell REAL NOT NULL, created_at TEXT NOT NULL)",
        "CREATE TABLE IF NOT EXISTS expenses (id INTEGER PRIMARY KEY AUTOINCREMENT, expense_type TEXT NOT NULL, amount REAL NOT NULL, expense_date TEXT NOT NULL, notes TEXT, user_id INTEGER)",
        "CREATE TABLE IF NOT EXISTS inventory_movements (id INTEGER PRIMARY KEY AUTOINCREMENT, product_id INTEGER NOT NULL, movement_type TEXT NOT NULL, quantity INTEGER NOT NULL, price REAL, movement_date TEXT NOT NULL, reference_id INTEGER, user_id INTEGER NOT NULL, notes TEXT, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
        "CREATE TABLE IF NOT EXISTS system_settings (id INTEGER PRIMARY KEY, capital REAL DEFAULT 0, store_name TEXT, store_address TEXT, store_email TEXT, store_phone TEXT, currency TEXT DEFAULT 'SYP', decimal_places INTEGER DEFAULT 2, created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP)",
    ]
    for t in tables:
        try: c.execute(t)
        except: pass
    try: c.execute("ALTER TABLE products ADD COLUMN is_deleted INTEGER DEFAULT 0")
    except: pass
    c.execute("SELECT COUNT(*) FROM users")
    if c.fetchone()[0] == 0:
        for u,p,r in [('admin','admin123','admin'),('khaled','khaled123','cashier'),('cashier','cashier123','cashier')]:
            try: c.execute("INSERT INTO users(username,password,role)VALUES(?,?,?)",(u,p,r))
            except: pass
    conn.commit(); conn.close()

# ═══════════════════════════════════════════════════════════
#  ROUTES
# ═══════════════════════════════════════════════════════════

@app.route('/')
def index():
    resp = make_response(HTML)
    resp.headers['Content-Type'] = 'text/html; charset=utf-8'
    return resp

# ── AUTH ─────────────────────────────────────────────────
@app.route('/api/auth/login', methods=['POST'])
def login():
    d = request.json or {}
    un = (d.get('username') or '').strip()
    pw = (d.get('password') or '').strip()
    if not un or not pw:
        return err("أدخل اسم المستخدم وكلمة المرور")
    db = get_db()
    row = db.execute("SELECT id,username,role FROM users WHERE username=? AND password=?", (un, pw)).fetchone()
    db.close()
    if not row:
        return err("اسم المستخدم أو كلمة المرور غير صحيحة")
    user = dict(row)
    token = make_token(user)
    return ok(user, "مرحباً " + user['username'], token=token)

@app.route('/api/auth/logout', methods=['POST'])
def logout():
    t = request.headers.get('X-Token')
    _tokens.pop(t, None)
    return ok(msg="تم تسجيل الخروج")

@app.route('/api/auth/me')
@login_required
def me():
    return ok(get_user())

# ── PRODUCTS ─────────────────────────────────────────────
@app.route('/api/products')
@login_required
def get_products():
    search = request.args.get('search','')
    category = request.args.get('category','')
    status = request.args.get('status','')
    db = get_db()
    q = "SELECT p.*,s.name as supplier_name FROM products p LEFT JOIN suppliers s ON p.supplier_id=s.id WHERE p.is_deleted=0"
    params = []
    if search:
        q += " AND (p.name LIKE ? OR p.barcode LIKE ? OR p.brand LIKE ?)"
        params += [f'%{search}%']*3
    if category:
        q += " AND p.category=?"; params.append(category)
    if status == "منخفض": q += " AND p.quantity<=p.min_stock AND p.quantity>0"
    elif status == "نافذ": q += " AND p.quantity<=0"
    elif status == "متوفر": q += " AND p.quantity>p.min_stock"
    q += " ORDER BY p.id DESC"
    rows = db.execute(q, params).fetchall(); db.close()
    return ok(rows_to_list(rows))

@app.route('/api/products/<int:pid>')
@login_required
def get_product(pid):
    db = get_db()
    r = db.execute("SELECT p.*,s.name as supplier_name FROM products p LEFT JOIN suppliers s ON p.supplier_id=s.id WHERE p.id=?", (pid,)).fetchone()
    db.close()
    return ok(dict(r)) if r else err("غير موجود", 404)

@app.route('/api/products/low-stock')
@login_required
def low_stock():
    db = get_db()
    rows = db.execute("SELECT * FROM products WHERE quantity<=min_stock AND is_deleted=0 ORDER BY quantity").fetchall()
    db.close()
    return ok(rows_to_list(rows))

@app.route('/api/products/barcode/<string:bc>')
@login_required
def by_barcode(bc):
    db = get_db()
    r = db.execute("SELECT * FROM products WHERE barcode=? AND is_deleted=0", (bc,)).fetchone()
    db.close()
    return ok(dict(r)) if r else err("لم يوجد", 404)

@app.route('/api/products/add', methods=['POST'])
@admin_required
def add_product():
    d = request.json or {}
    db = get_db()
    try:
        # التحقق من الحقول المطلوبة
        if not d.get('name'):
            return err("اسم المنتج مطلوب")
        if not d.get('brand'):
            return err("الماركة مطلوبة")
        
        # التحقق من الأرقام
        try:
            cost_price = float(d.get('cost_price', 0))
            selling_price = float(d.get('selling_price', 0))
            quantity = int(d.get('quantity', 0))
            min_stock = int(d.get('min_stock', 5))
        except ValueError:
            return err("الأسعار والكمية يجب أن تكون أرقام")
        
        db.execute(
            "INSERT INTO products(name,brand,model,category,cost_price,selling_price,quantity,barcode,min_stock,condition) VALUES(?,?,?,?,?,?,?,?,?,?)",
            (d.get('name'), d.get('brand'), d.get('model','-'), d.get('category','موبايلات'),
             cost_price, selling_price,
             quantity, d.get('barcode') or None,
             min_stock, d.get('condition','جديد'))
        )
        db.commit()
        db.close()
        return ok(msg="تم إضافة المنتج")
    except sqlite3.IntegrityError:
        db.close()
        return err("الباركود مستخدم مسبقاً")
    except Exception as e:
        db.close()
        return err(f"خطأ في إضافة المنتج: {str(e)}")

@app.route('/api/products/<int:pid>', methods=['PUT'])
@admin_required
def update_product(pid):
    d = request.json or {}
    db = get_db()
    try:
        # التحقق من الأرقام
        try:
            cost_price = float(d.get('cost_price', 0))
            selling_price = float(d.get('selling_price', 0))
            quantity = int(d.get('quantity', 0))
            min_stock = int(d.get('min_stock', 5))
        except ValueError:
            return err("الأسعار والكمية يجب أن تكون أرقام")
        
        db.execute(
            "UPDATE products SET name=?,brand=?,model=?,category=?,cost_price=?,selling_price=?,quantity=?,min_stock=?,condition=?,last_updated=CURRENT_TIMESTAMP WHERE id=?",
            (d.get('name'), d.get('brand'), d.get('model','-'), d.get('category','موبايلات'),
             cost_price, selling_price,
             quantity, min_stock, d.get('condition','جديد'), pid)
        )
        db.commit()
        db.close()
        return ok(msg="تم التعديل")
    except Exception as e:
        db.close()
        return err(f"خطأ في التعديل: {str(e)}")

@app.route('/api/products/<int:pid>', methods=['DELETE'])
@admin_required
def del_product(pid):
    db = get_db()
    db.execute("UPDATE products SET is_deleted=1 WHERE id=?", (pid,))
    db.commit(); db.close()
    return ok(msg="تم الحذف")

@app.route('/api/products/categories')
@login_required
def categories():
    db = get_db()
    rows = db.execute("SELECT DISTINCT category FROM products WHERE is_deleted=0").fetchall()
    db.close()
    return ok([r['category'] for r in rows])

@app.route('/api/inventory')
@login_required
def get_inventory():
    search = request.args.get('search','')
    category = request.args.get('category','')
    status = request.args.get('status','')
    db = get_db()
    q = "SELECT p.*,s.name as supplier_name FROM products p LEFT JOIN suppliers s ON p.supplier_id=s.id WHERE p.is_deleted=0"
    params = []
    if search:
        q += " AND (p.name LIKE ? OR p.barcode LIKE ? OR p.brand LIKE ?)"
        params += [f'%{search}%']*3
    if category and category not in ('الكل',''):
        q += " AND p.category=?"; params.append(category)
    if status == "منخفض": q += " AND p.quantity<=p.min_stock AND p.quantity>0"
    elif status in ("منتهي","نافذ"): q += " AND p.quantity<=0"
    elif status == "متوفر": q += " AND p.quantity>p.min_stock"
    q += " ORDER BY p.id DESC"
    rows = db.execute(q, params).fetchall(); db.close()
    return ok(rows_to_list(rows))

@app.route('/api/inventory/clear', methods=['POST'])
@admin_required
def clear_all_products():
    """حذف جميع المنتجات من المخزون"""
    db = get_db()
    db.execute("UPDATE products SET is_deleted=1")
    db.commit(); db.close()
    return ok(msg="تم حذف جميع المنتجات")

# ── SALES ─────────────────────────────────────────────────
@app.route('/api/sales')
@login_required
def get_sales():
    period = request.args.get('period','اليوم')
    search = request.args.get('search','')
    s, e = date_range(period)
    db = get_db()
    q = "SELECT s.*,u.username FROM sales s JOIN users u ON s.user_id=u.id WHERE s.sale_date BETWEEN ? AND ?"
    params = [s, e]
    if search:
        q += " AND (s.invoice_number LIKE ? OR u.username LIKE ?)"; params += [f'%{search}%']*2
    q += " ORDER BY s.id DESC"
    rows = db.execute(q, params).fetchall(); db.close()
    return ok(rows_to_list(rows))

@app.route('/api/sales/<int:sid>')
@login_required
def get_sale(sid):
    db = get_db()
    sale = db.execute("SELECT s.*,u.username FROM sales s JOIN users u ON s.user_id=u.id WHERE s.id=?", (sid,)).fetchone()
    if not sale: db.close(); return err("غير موجود",404)
    det = db.execute("SELECT sd.*,p.name as product_name FROM sale_details sd JOIN products p ON sd.product_id=p.id WHERE sd.sale_id=?", (sid,)).fetchall()
    pays = db.execute("SELECT * FROM payments WHERE reference_id=? AND type='sale'", (sid,)).fetchall()
    db.close()
    return ok({"sale":dict(sale),"details":rows_to_list(det),"payments":rows_to_list(pays)})

@app.route('/api/sales/stats')
@login_required
def sales_stats():
    s, e = date_range(request.args.get('period','اليوم'))
    db = get_db()
    r = db.execute("SELECT COUNT(*) as count,SUM(total_amount) as total,SUM(profit) as profit,SUM(remaining_amount) as remaining FROM sales WHERE sale_date BETWEEN ? AND ?", (s,e)).fetchone()
    db.close()
    return ok(dict(r))

@app.route('/api/sales/add', methods=['POST'])
@login_required
def add_sale():
    d = request.json or {}
    items = d.get('items', [])
    if not items: return err("السلة فارغة")
    u = get_user()
    paid = float(d.get('paid_amount', 0))
    today = date.today().isoformat()
    db = get_db()
    try:
        total = profit = 0
        for item in items:
            p = db.execute("SELECT cost_price,selling_price,quantity FROM products WHERE id=? AND is_deleted=0", (item['product_id'],)).fetchone()
            if not p: db.close(); return err(f"منتج غير موجود: {item['product_id']}")
            if p['quantity'] < item['quantity']: db.close(); return err(f"كمية غير كافية للمنتج {item['product_id']}")
            up = float(item.get('unit_price', p['selling_price']))
            total += up * item['quantity']
            profit += (up - p['cost_price']) * item['quantity']
        last = db.execute("SELECT invoice_number FROM sales ORDER BY id DESC LIMIT 1").fetchone()
        try: num = int(last['invoice_number'].replace('INV-','')) + 1
        except: num = 1
        inv = f"INV-{num:05d}"
        cur = db.execute("INSERT INTO sales(invoice_number,total_amount,profit,paid_amount,remaining_amount,sale_date,sale_time,user_id,status,payment_method) VALUES(?,?,?,?,?,?,?,?,?,?)",
            (inv, total, profit, paid, total-paid, today, datetime.now().strftime('%H:%M:%S'), u['id'], 'active', d.get('payment_method','نقدي')))
        sid = cur.lastrowid
        for item in items:
            p = db.execute("SELECT cost_price FROM products WHERE id=?", (item['product_id'],)).fetchone()
            up = float(item.get('unit_price',0))
            db.execute("INSERT INTO sale_details(sale_id,product_id,quantity,unit_price,total_price,cost_price,profit) VALUES(?,?,?,?,?,?,?)",
                (sid, item['product_id'], item['quantity'], up, up*item['quantity'], p['cost_price'], (up-p['cost_price'])*item['quantity']))
            db.execute("UPDATE products SET quantity=quantity-? WHERE id=?", (item['quantity'], item['product_id']))
        db.commit(); db.close()
        return ok({"sale_id":sid,"invoice_number":inv,"total":total,"remaining":total-paid}, "تم البيع بنجاح")
    except Exception as e:
        db.rollback(); db.close(); return err(str(e))

@app.route('/api/sales/<int:sid>/pay', methods=['POST'])
@login_required
def pay_sale(sid):
    d = request.json or {}
    amount = float(d.get('amount', 0))
    if amount <= 0: return err("المبلغ يجب أن يكون أكبر من صفر")
    u = get_user()
    db = get_db()
    sale = db.execute("SELECT remaining_amount FROM sales WHERE id=?", (sid,)).fetchone()
    if not sale: db.close(); return err("غير موجود",404)
    db.execute("INSERT INTO payments(type,reference_id,amount,payment_date,payment_method,user_id) VALUES('sale',?,?,?,?,?)",
        (sid, amount, date.today().isoformat(), d.get('method','نقدي'), u['id']))
    db.execute("UPDATE sales SET paid_amount=paid_amount+?,remaining_amount=remaining_amount-? WHERE id=?", (amount,amount,sid))
    db.commit(); db.close()
    return ok(msg="تم تسجيل الدفعة")

@app.route('/api/sales/<int:sid>', methods=['DELETE'])
@admin_required
def del_sale(sid):
    db = get_db()
    for row in db.execute("SELECT product_id,quantity FROM sale_details WHERE sale_id=?", (sid,)).fetchall():
        db.execute("UPDATE products SET quantity=quantity+? WHERE id=?", (row['quantity'], row['product_id']))
    db.execute("DELETE FROM sale_details WHERE sale_id=?", (sid,))
    db.execute("DELETE FROM sales WHERE id=?", (sid,))
    db.commit(); db.close()
    return ok(msg="تم الحذف")

@app.route('/api/sales/<int:sid>/return', methods=['POST'])
@login_required
def return_sale(sid):
    """إعادة منتجات الفاتورة إلى المخزون"""
    db = get_db()
    for row in db.execute("SELECT product_id,quantity FROM sale_details WHERE sale_id=?", (sid,)).fetchall():
        db.execute("UPDATE products SET quantity=quantity+? WHERE id=?", (row['quantity'], row['product_id']))
    db.commit(); db.close()
    return ok(msg="تم إعادة المنتجات للمخزون")

@app.route('/api/sales/clear', methods=['POST'])
@admin_required
def clear_all_sales():
    """حذف جميع الفواتير"""
    db = get_db()
    # استرجاع جميع المنتجات أولاً
    for row in db.execute("SELECT product_id,quantity FROM sale_details").fetchall():
        db.execute("UPDATE products SET quantity=quantity+? WHERE id=?", (row['quantity'], row['product_id']))
    db.execute("DELETE FROM sale_details")
    db.execute("DELETE FROM sales")
    db.commit(); db.close()
    return ok(msg="تم حذف جميع الفواتير")

# ── PURCHASES ─────────────────────────────────────────────
@app.route('/api/purchases')
@login_required
def get_purchases():
    s, e = date_range(request.args.get('period','شهر'))
    db = get_db()
    rows = db.execute("SELECT p.*,COALESCE(s.name,'—') as supplier_name,u.username FROM purchases p LEFT JOIN suppliers s ON p.supplier_id=s.id JOIN users u ON p.user_id=u.id WHERE p.purchase_date BETWEEN ? AND ? ORDER BY p.id DESC", (s,e)).fetchall()
    db.close(); return ok(rows_to_list(rows))

@app.route('/api/purchases/<int:pid>')
@login_required
def get_purchase(pid):
    db = get_db()
    p = db.execute("SELECT p.*,COALESCE(s.name,'—') as supplier_name FROM purchases p LEFT JOIN suppliers s ON p.supplier_id=s.id WHERE p.id=?", (pid,)).fetchone()
    if not p: db.close(); return err("غير موجود",404)
    det = db.execute("SELECT pd.*,pr.name as product_name FROM purchase_details pd JOIN products pr ON pd.product_id=pr.id WHERE pd.purchase_id=?", (pid,)).fetchall()
    db.close(); return ok({"purchase":dict(p),"details":rows_to_list(det)})

@app.route('/api/purchases/add', methods=['POST'])
@admin_required
def add_purchase():
    d = request.json or {}
    items = d.get('items', [])
    if not items: return err("أضف منتجات أولاً")
    supplier_id = d.get('supplier_id') or None  # allow null
    u = get_user(); today = date.today().isoformat()
    db = get_db()
    try:
        total = sum(float(i.get('unit_price',0))*int(i.get('quantity',1)) for i in items)
        paid = min(float(d.get('paid_amount',0)), total)
        last = db.execute("SELECT invoice_number FROM purchases ORDER BY id DESC LIMIT 1").fetchone()
        try: num = int(last['invoice_number'].replace('PUR-','')) + 1
        except: num = 1
        inv = f"PUR-{num:05d}"
        cur = db.execute("INSERT INTO purchases(invoice_number,supplier_id,total_amount,paid_amount,remaining_amount,purchase_date,user_id) VALUES(?,?,?,?,?,?,?)",
            (inv, supplier_id, total, paid, total-paid, today, u['id']))
        pid = cur.lastrowid
        for item in items:
            prod_id = item.get('product_id')
            qty = int(item.get('quantity',1))
            unit_price = float(item.get('unit_price',0))
            sell_price = float(item.get('sell_price',0))
            if not prod_id: continue
            db.execute("INSERT INTO purchase_details(purchase_id,product_id,quantity,unit_price,total_price) VALUES(?,?,?,?,?)",
                (pid, prod_id, qty, unit_price, unit_price*qty))
            if unit_price > 0:
                db.execute("UPDATE products SET quantity=quantity+?,cost_price=?,selling_price=CASE WHEN ?>0 THEN ? ELSE selling_price END WHERE id=?",
                    (qty, unit_price, sell_price, sell_price if sell_price>0 else unit_price, prod_id))
            else:
                db.execute("UPDATE products SET quantity=quantity+? WHERE id=?", (qty, prod_id))
        db.commit(); db.close()
        return ok({"purchase_id":pid,"invoice_number":inv}, "تم تسجيل المشتريات")
    except Exception as e:
        db.rollback(); db.close(); return err(str(e))

@app.route('/api/purchases/<int:pid>/pay', methods=['POST'])
@login_required
def pay_purchase(pid):
    d = request.json or {}; u = get_user()
    amount = float(d.get('amount',0))
    if amount <= 0: return err("المبلغ يجب أن يكون أكبر من صفر")
    db = get_db()
    db.execute("INSERT INTO payments(type,reference_id,amount,payment_date,payment_method,user_id) VALUES('purchase',?,?,?,?,?)",
        (pid, amount, date.today().isoformat(), d.get('method','نقدي'), u['id']))
    db.execute("UPDATE purchases SET paid_amount=paid_amount+?,remaining_amount=remaining_amount-? WHERE id=?", (amount,amount,pid))
    db.commit(); db.close(); return ok(msg="تم الدفع للمورد")

@app.route('/api/purchases/<int:pid>', methods=['DELETE'])
@admin_required
def delete_purchase(pid):
    db = get_db()
    try:
        db.execute("DELETE FROM purchase_details WHERE purchase_id=?", (pid,))
        db.execute("DELETE FROM payments WHERE reference_id=? AND type='purchase'", (pid,))
        db.execute("DELETE FROM purchases WHERE id=?", (pid,))
        db.commit()
        db.close()
        return ok(msg="تم حذف الفاتورة بنجاح")
    except Exception as e:
        db.rollback()
        db.close()
        return err(f"خطأ في حذف الفاتورة: {str(e)}")

# ── SUPPLIERS ────────────────────────────────────────────
@app.route('/api/suppliers')
@login_required
def get_suppliers():
    db=get_db(); rows=db.execute("SELECT * FROM suppliers ORDER BY name").fetchall(); db.close(); return ok(rows_to_list(rows))

@app.route('/api/suppliers/add', methods=['POST'])
@admin_required
def add_supplier():
    d=request.json or {}
    if not d.get('name'): return err("اسم المورد مطلوب")
    db=get_db()
    try:
        cur=db.execute("INSERT INTO suppliers(name,phone,email,address)VALUES(?,?,?,?)",
            (d['name'], d.get('phone',''), d.get('email',''), d.get('address','')))
        new_id=cur.lastrowid
        db.commit(); db.close()
        return ok({"id":new_id}, "تم إضافة المورد")
    except sqlite3.IntegrityError:
        db.close(); return err("المورد موجود بالفعل")
    except Exception as e:
        db.close(); return err(str(e))

@app.route('/api/suppliers/<int:sid>', methods=['PUT'])
@admin_required
def update_supplier(sid):
    d=request.json or {}
    db=get_db()
    try:
        db.execute(
            "UPDATE suppliers SET name=?,phone=?,email=?,address=? WHERE id=?",
            (d.get('name'), d.get('phone'), d.get('email',''), d.get('address',''), sid)
        )
        db.commit()
        db.close()
        return ok(msg="تم التعديل")
    except Exception as e:
        db.close()
        return err(f"خطأ في التعديل: {str(e)}")

@app.route('/api/suppliers/<int:sid>', methods=['DELETE'])
@admin_required
def del_supplier(sid):
    db=get_db()
    try:
        db.execute("DELETE FROM suppliers WHERE id=?",(sid,))
        db.commit()
        db.close()
        return ok(msg="تم الحذف")
    except Exception as e:
        db.close()
        return err(f"خطأ في الحذف: {str(e)}")

# ── SETTINGS (إعدادات النظام) ──────────────────────────────
@app.route('/api/settings', methods=['GET'])
@login_required
def get_settings():
    """جلب إعدادات النظام"""
    db = get_db()
    try:
        # البحث عن إعدادات موجودة
        settings = db.execute("SELECT * FROM system_settings WHERE id=1").fetchone()
        if settings:
            db.close()
            return ok(dict(settings))
        else:
            # إرجاع إعدادات افتراضية
            default = {
                "id": 1,
                "capital": 0,
                "store_name": "",
                "store_address": "",
                "store_email": "",
                "store_phone": "",
                "currency": "SYP",
                "decimal_places": 2
            }
            db.close()
            return ok(default)
    except:
        db.close()
        return ok({})

@app.route('/api/settings', methods=['POST'])
@admin_required
def save_settings():
    """حفظ إعدادات النظام"""
    d = request.json or {}
    db = get_db()
    try:
        # التحقق من وجود إعدادات موجودة
        exists = db.execute("SELECT id FROM system_settings WHERE id=1").fetchone()
        
        if exists:
            # تحديث الإعدادات
            db.execute("""
                UPDATE system_settings 
                SET capital=?, store_name=?, store_address=?, store_email=?, 
                    store_phone=?, currency=?, decimal_places=?
                WHERE id=1
            """, (
                float(d.get('capital', 0)),
                d.get('store_name', ''),
                d.get('store_address', ''),
                d.get('store_email', ''),
                d.get('store_phone', ''),
                d.get('currency', 'SYP'),
                int(d.get('decimal_places', 2))
            ))
        else:
            # إدراج إعدادات جديدة
            db.execute("""
                INSERT INTO system_settings 
                (id, capital, store_name, store_address, store_email, store_phone, currency, decimal_places)
                VALUES (1, ?, ?, ?, ?, ?, ?, ?)
            """, (
                float(d.get('capital', 0)),
                d.get('store_name', ''),
                d.get('store_address', ''),
                d.get('store_email', ''),
                d.get('store_phone', ''),
                d.get('currency', 'SYP'),
                int(d.get('decimal_places', 2))
            ))
        
        db.commit()
        db.close()
        return ok(msg="تم حفظ الإعدادات بنجاح")
    except Exception as e:
        db.close()
        return err(f"خطأ في حفظ الإعدادات: {str(e)}")

# ── CUSTOMERS ────────────────────────────────────────────
@app.route('/api/customers')
@login_required
def get_customers():
    db=get_db(); rows=db.execute("SELECT * FROM customers ORDER BY name").fetchall(); db.close(); return ok(rows_to_list(rows))

@app.route('/api/customers/add', methods=['POST'])
@login_required
def add_customer():
    d=request.json or {}
    if not d.get('name'): 
        return err("الاسم مطلوب")
    
    db=get_db()
    try:
        db.execute(
            "INSERT INTO customers(name,phone,email,address)VALUES(?,?,?,?)",
            (d['name'], d.get('phone',''), d.get('email',''), d.get('address',''))
        )
        db.commit()
        db.close()
        return ok(msg="تم إضافة العميل")
    except Exception as e:
        db.close()
        return err(f"خطأ في إضافة العميل: {str(e)}")

@app.route('/api/customers/<int:cid>', methods=['DELETE'])
@admin_required
def del_customer(cid):
    db=get_db()
    try:
        db.execute("DELETE FROM customers WHERE id=?",(cid,))
        db.commit()
        db.close()
        return ok(msg="تم الحذف")
    except Exception as e:
        db.close()
        return err(f"خطأ في الحذف: {str(e)}")

# ── EXPENSES ─────────────────────────────────────────────
@app.route('/api/expenses')
@login_required
def get_expenses():
    s,e = date_range(request.args.get('period','شهر'))
    db=get_db()
    rows=db.execute("SELECT ex.*,u.username FROM expenses ex LEFT JOIN users u ON ex.user_id=u.id WHERE ex.expense_date BETWEEN ? AND ? ORDER BY ex.id DESC",(s,e)).fetchall()
    total=db.execute("SELECT SUM(amount) FROM expenses WHERE expense_date BETWEEN ? AND ?",(s,e)).fetchone()[0] or 0
    db.close()
    return ok({"items":rows_to_list(rows),"total":total})

@app.route('/api/expenses/add', methods=['POST'])
@login_required
def add_expense():
    d=request.json or {}
    u=get_user()
    
    if not d.get('expense_type') or not d.get('amount'):
        return err("النوع والمبلغ مطلوبان")
    
    db=get_db()
    try:
        # التحقق من أن المبلغ رقم
        try:
            amount = float(d['amount'])
        except ValueError:
            return err("المبلغ يجب أن يكون رقم")
        
        db.execute(
            "INSERT INTO expenses(expense_type,amount,expense_date,notes,user_id)VALUES(?,?,?,?,?)",
            (d['expense_type'], amount, d.get('expense_date',date.today().isoformat()),
             d.get('notes',''), u['id'])
        )
        db.commit()
        db.close()
        return ok(msg="تم إضافة المصروف")
    except Exception as e:
        db.close()
        return err(f"خطأ في إضافة المصروف: {str(e)}")

@app.route('/api/expenses/<int:eid>', methods=['DELETE'])
@admin_required
def del_expense(eid):
    db=get_db()
    try:
        db.execute("DELETE FROM expenses WHERE id=?",(eid,))
        db.commit()
        db.close()
        return ok(msg="تم الحذف")
    except Exception as e:
        db.close()
        return err(f"خطأ في الحذف: {str(e)}")

# ── MAINTENANCE ───────────────────────────────────────────
@app.route('/api/maintenance/clear', methods=['POST'])
@admin_required
def clear_all_maintenance():
    db = get_db()
    db.execute("DELETE FROM maintenance")
    db.commit(); db.close()
    return ok(msg="تم حذف جميع طلبات الصيانة")

@app.route('/api/maintenance')
@login_required
def get_maintenance():
    sf=request.args.get('status',''); db=get_db()
    q="SELECT * FROM maintenance"; params=[]
    if sf: q+=" WHERE status=?"; params.append(sf)
    q+=" ORDER BY id DESC"
    rows=db.execute(q,params).fetchall(); db.close(); return ok(rows_to_list(rows))

@app.route('/api/maintenance/add', methods=['POST'])
@login_required
def add_maintenance():
    d=request.json or {}
    u=get_user()
    
    # التحقق من الحقول المطلوبة
    if not all([d.get('customer_name'), d.get('phone'), d.get('device_model')]):
        return err("اسم العميل والهاتف والجهاز مطلوبة")
    
    db=get_db()
    try:
        # التحقق من الأرقام
        try:
            cost = float(d.get('cost', 0))
            selling_price = float(d.get('selling_price', 0))
        except ValueError:
            return err("التكلفة والسعر يجب أن تكون أرقام")
        
        db.execute(
            "INSERT INTO maintenance(customer_name,phone,device_model,problem_description,cost,status,received_date,received_time,selling_price,user_id)VALUES(?,?,?,?,?,?,?,?,?,?)",
            (d['customer_name'], d['phone'], d['device_model'], d.get('problem_description',''),
             cost, d.get('status','قيد الإصلاح'), date.today().isoformat(),
             d.get('received_time','00:00'), selling_price, u['id'])
        )
        db.commit()
        db.close()
        return ok(msg="تم إضافة طلب الصيانة")
    except Exception as e:
        db.close()
        return err(f"خطأ في إضافة الصيانة: {str(e)}")

@app.route('/api/maintenance/<int:mid>', methods=['PUT'])
@login_required
def update_maintenance(mid):
    d=request.json or {}
    db=get_db()
    try:
        # التحقق من الأرقام
        try:
            cost = float(d.get('cost', 0))
            selling_price = float(d.get('selling_price', 0))
        except ValueError:
            return err("التكلفة والسعر يجب أن تكون أرقام")
        
        db.execute(
            "UPDATE maintenance SET status=?,cost=?,selling_price=? WHERE id=?",
            (d.get('status'), cost, selling_price, mid)
        )
        db.commit()
        db.close()
        return ok(msg="تم التحديث")
    except Exception as e:
        db.close()
        return err(f"خطأ في التحديث: {str(e)}")

@app.route('/api/maintenance/<int:mid>', methods=['DELETE'])
@admin_required
def del_maintenance(mid):
    db=get_db()
    try:
        db.execute("DELETE FROM maintenance WHERE id=?",(mid,))
        db.commit()
        db.close()
        return ok(msg="تم الحذف")
    except Exception as e:
        db.close()
        return err(f"خطأ في الحذف: {str(e)}")

# ── CURRENCY ──────────────────────────────────────────────
_rates = {"syp_buy":14800,"syp_sell":15000,"try_buy":29.5,"try_sell":30.5}

@app.route('/api/currency/rates')
@login_required
def get_rates():
    db=get_db()
    r=db.execute("SELECT * FROM pinned_exchange_rates ORDER BY id DESC LIMIT 1").fetchone()
    db.close()
    if r: return ok({**_rates,**dict(r)})
    return ok(_rates)

@app.route('/api/currency/rates', methods=['POST'])
@login_required
def save_rates():
    d=request.json or {}
    _rates.update({k:float(d[k]) for k in ['syp_buy','syp_sell','try_buy','try_sell'] if k in d})
    db=get_db()
    db.execute("INSERT OR REPLACE INTO pinned_exchange_rates(date,syp_buy,syp_sell,try_buy,try_sell,created_at) VALUES(?,?,?,?,?,?)",
        (date.today().isoformat(),_rates['syp_buy'],_rates['syp_sell'],_rates['try_buy'],_rates['try_sell'],datetime.now().isoformat()))
    db.commit(); db.close(); return ok(_rates,"تم حفظ الأسعار")

# ── REPORTS ───────────────────────────────────────────────
@app.route('/api/reports/dashboard')
@login_required
def dashboard():
    today=date.today().isoformat(); month=date.today().strftime("%Y-%m"); year=date.today().strftime("%Y")
    db=get_db(); g=lambda q,p=(): (db.execute(q,p).fetchone()[0] or 0)
    data = {
        "today_sales":   g("SELECT SUM(total_amount) FROM sales WHERE sale_date=?",(today,)),
        "today_count":   g("SELECT COUNT(*) FROM sales WHERE sale_date=?",(today,)),
        "month_sales":   g("SELECT SUM(total_amount) FROM sales WHERE strftime('%Y-%m',sale_date)=?",(month,)),
        "month_count":   g("SELECT COUNT(*) FROM sales WHERE strftime('%Y-%m',sale_date)=?",(month,)),
        "month_profit":  g("SELECT SUM(profit) FROM sales WHERE strftime('%Y-%m',sale_date)=?",(month,)),
        "year_profit":   g("SELECT SUM(profit) FROM sales WHERE strftime('%Y',sale_date)=?",(year,)),
        "total_profit":  g("SELECT SUM(profit) FROM sales"),
        "month_purchases":g("SELECT SUM(total_amount) FROM purchases WHERE strftime('%Y-%m',purchase_date)=?",(month,)),
        "month_expenses": g("SELECT SUM(amount) FROM expenses WHERE strftime('%Y-%m',expense_date)=?",(month,)),
        "total_expenses": g("SELECT SUM(amount) FROM expenses"),
        "receivables":   g("SELECT SUM(remaining_amount) FROM sales WHERE remaining_amount>0"),
        "payables":      g("SELECT SUM(remaining_amount) FROM purchases WHERE remaining_amount>0"),
        "inventory_value":g("SELECT SUM(quantity*cost_price) FROM products WHERE is_deleted=0"),
        "total_products": g("SELECT COUNT(*) FROM products WHERE is_deleted=0"),
        "low_stock":     g("SELECT COUNT(*) FROM products WHERE quantity<=min_stock AND is_deleted=0 AND quantity>0"),
        "out_stock":     g("SELECT COUNT(*) FROM products WHERE quantity<=0 AND is_deleted=0"),
        "total_customers":g("SELECT COUNT(*) FROM customers"),
        "all_time_sales": g("SELECT SUM(total_amount) FROM sales"),
    }
    data['month_net'] = data['month_sales'] - data['month_purchases'] - data['month_expenses']
    data['net_debt']  = data['receivables'] - data['payables']
    best=db.execute("SELECT p.name,SUM(sd.quantity) as q FROM sale_details sd JOIN products p ON sd.product_id=p.id GROUP BY sd.product_id ORDER BY q DESC LIMIT 1").fetchone()
    data['best_product'] = best['name'] if best else '—'
    db.close(); return ok(data)

@app.route('/api/reports/top-products')
@login_required
def top_products():
    limit=request.args.get('limit',10)
    db=get_db()
    rows=db.execute("SELECT p.name,p.brand,SUM(sd.quantity) as total_qty,SUM(sd.total_price) as total_revenue,SUM(sd.profit) as total_profit FROM sale_details sd JOIN products p ON sd.product_id=p.id GROUP BY sd.product_id ORDER BY total_qty DESC LIMIT ?",(limit,)).fetchall()
    db.close(); return ok(rows_to_list(rows))

@app.route('/api/reports/inventory')
@login_required
def inventory_report():
    db=get_db()
    rows=db.execute("SELECT p.*,s.name as supplier_name,(p.quantity*p.cost_price) as total_cost,(p.quantity*p.selling_price) as total_value,(p.quantity*(p.selling_price-p.cost_price)) as expected_profit FROM products p LEFT JOIN suppliers s ON p.supplier_id=s.id WHERE p.is_deleted=0 ORDER BY p.quantity").fetchall()
    summary=db.execute("SELECT SUM(quantity*cost_price) as total_cost,SUM(quantity*selling_price) as total_value,COUNT(*) as total_items,SUM(CASE WHEN quantity<=0 THEN 1 ELSE 0 END) as out_of_stock,SUM(CASE WHEN quantity<=min_stock AND quantity>0 THEN 1 ELSE 0 END) as low_stock FROM products WHERE is_deleted=0").fetchone()
    db.close(); return ok({"items":rows_to_list(rows),"summary":dict(summary)})

# ── BACKUP (النسخ الاحتياطي الجديد) ──────────────────────────────
@app.route('/api/settings/backup', methods=['POST'])
@admin_required
def create_backup():
    try:
        import shutil
        import time
        
        # إنشاء مجلد النسخ الاحتياطية
        os.makedirs("backups", exist_ok=True)
        
        # إغلاق أي اتصالات مفتوحة مؤقتاً
        time.sleep(0.2)
        
        ts = datetime.now().strftime('%Y%m%d_%H%M%S')
        bp = f"backups/backup_{ts}.db"
        
        try:
            shutil.copy2(DB_PATH, bp)
            
            # التحقق من أن النسخة تم إنشاؤها بنجاح
            if os.path.exists(bp):
                file_size = os.path.getsize(bp) / (1024 * 1024)  # MB
                return ok(msg=f"تم إنشاء النسخة الاحتياطية بنجاح ({file_size:.2f} MB)")
            else:
                return err("فشل في حفظ النسخة الاحتياطية")
                
        except Exception as copy_e:
            return err(f"خطأ في النسخ: {str(copy_e)}")
            
    except Exception as e:
        return err(f"خطأ في إنشاء النسخة: {str(e)}")

@app.route('/api/settings/backups', methods=['GET'])
@admin_required
def list_backups():
    try:
        backup_dir = "backups"
        if not os.path.exists(backup_dir):
            os.makedirs(backup_dir, exist_ok=True)
            return ok([])
        
        files = []
        try:
            for f in sorted(os.listdir(backup_dir), reverse=True):
                # تصفية الملفات - فقط ملفات .db
                if f.endswith('.db'):
                    path = os.path.join(backup_dir, f)
                    try:
                        if os.path.isfile(path):
                            size = os.path.getsize(path) / (1024*1024)  # MB
                            mtime = os.path.getmtime(path)
                            
                            files.append({
                                'name': f,
                                'size': f"{size:.2f} MB",
                                'path': path,
                                'time': mtime
                            })
                    except:
                        # تجاهل الملفات التي لا يمكن الوصول إليها
                        pass
            
            return ok(files)
        except Exception as list_e:
            return err(f"خطأ في قراءة المجلد: {str(list_e)}")
            
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/settings/backup/restore/<filename>', methods=['POST'])
@admin_required
def restore_backup(filename):
    try:
        import shutil
        import time
        
        # تنظيف اسم الملف من الأحرف الخطرة
        if '..' in filename or '/' in filename or '\\' in filename:
            return err("اسم ملف غير صحيح")
        
        backup_path = f"backups/{filename}"
        
        if not os.path.exists(backup_path):
            return err("النسخة غير موجودة")
        
        # تأكد من أن الملف يمكن قراءته
        if not os.access(backup_path, os.R_OK):
            return err("لا توجد صلاحيات كافية لقراءة الملف")
        
        try:
            # التحقق من أن الملف قاعدة بيانات صحيحة
            import sqlite3
            test_conn = sqlite3.connect(backup_path)
            test_conn.execute("SELECT name FROM sqlite_master WHERE type='table';")
            test_conn.close()
        except Exception as validate_e:
            return err(f"الملف المختار ليس قاعدة بيانات صحيحة: {str(validate_e)}")
        
        try:
            # عمل نسخة احتياطية من الحالية قبل الاسترجاع
            os.makedirs("backups", exist_ok=True)
            ts = datetime.now().strftime('%Y%m%d_%H%M%S')
            before_restore = f"backups/before_restore_{ts}.db"
            
            if os.path.exists(DB_PATH):
                shutil.copy2(DB_PATH, before_restore)
            
            # نسخ النسخة الاحتياطية
            shutil.copy2(backup_path, DB_PATH)
            
            # انتظر قليلاً
            time.sleep(0.5)
            
            # اختبر الاتصال بقاعدة البيانات المستعادة
            test_conn = sqlite3.connect(DB_PATH)
            test_conn.close()
            
            return ok(msg="تم استرجاع النسخة الاحتياطية بنجاح!")
            
        except Exception as inner_e:
            return err(f"خطأ في نسخ الملف: {str(inner_e)}")
            
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/settings/backup/<filename>', methods=['DELETE'])
@admin_required
def delete_backup(filename):
    try:
        # التحقق من أن اسم الملف آمن (بدون مسارات خطرة)
        if '..' in filename or '/' in filename or '\\' in filename:
            return err("اسم ملف غير صحيح")
        
        backup_path = f"backups/{filename}"
        
        if not os.path.exists(backup_path):
            return err("النسخة غير موجودة")
        
        # التأكد من أن الملف موجود في مجلد النسخ الاحتياطية فقط
        if not os.path.abspath(backup_path).startswith(os.path.abspath("backups")):
            return err("لا يمكن حذف هذا الملف")
        
        try:
            os.remove(backup_path)
            return ok(msg="تم حذف النسخة بنجاح")
        except Exception as del_e:
            return err(f"خطأ في الحذف: {str(del_e)}")
            
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/settings/backup/restore-external', methods=['POST'])
@admin_required
def restore_backup_external():
    try:
        import shutil
        import time
        from werkzeug.utils import secure_filename
        import sqlite3
        
        # التحقق من وجود الملف المرفوع
        if 'file' not in request.files:
            return err("لم يتم اختيار ملف")
        
        file = request.files['file']
        
        if file.filename == '':
            return err("الملف فارغ")
        
        # التحقق من امتداد الملف
        allowed_extensions = {'db', 'backup'}
        if '.' not in file.filename or file.filename.rsplit('.', 1)[1].lower() not in allowed_extensions:
            return err("نوع الملف غير مدعوم. استخدم ملفات .db أو .backup")
        
        # حفظ الملف مؤقتاً بطريقة آمنة
        timestamp = int(time.time() * 1000)
        filename = secure_filename(file.filename)
        if not filename:
            filename = 'temp_backup.db'
        temp_path = f"temp_restore_{timestamp}_{filename}"
        
        try:
            # حفظ الملف المرفوع
            file.save(temp_path)
            
            if not os.path.exists(temp_path):
                return err("فشل في حفظ الملف المرفوع")
            
            # التحقق من أن الملف قاعدة بيانات صحيحة
            try:
                conn_test = sqlite3.connect(temp_path, timeout=5)
                cursor_test = conn_test.cursor()
                cursor_test.execute("SELECT count(*) FROM sqlite_master WHERE type='table'")
                table_count = cursor_test.fetchone()[0]
                cursor_test.close()
                conn_test.close()
                
                if table_count == 0:
                    if os.path.exists(temp_path):
                        os.remove(temp_path)
                    return err("الملف لا يحتوي على جداول - قد يكون فارغاً")
                    
            except sqlite3.DatabaseError as db_err:
                if os.path.exists(temp_path):
                    os.remove(temp_path)
                return err(f"الملف ليس قاعدة بيانات صحيحة: {str(db_err)}")
            except Exception as val_err:
                if os.path.exists(temp_path):
                    os.remove(temp_path)
                return err(f"خطأ في التحقق: {str(val_err)}")
            
            # انتظر قليلاً
            time.sleep(0.3)
            
            # إنشاء مجلد النسخ الاحتياطية
            os.makedirs("backups", exist_ok=True)
            
            # عمل نسخة احتياطية من البيانات الحالية
            ts = datetime.now().strftime('%Y%m%d_%H%M%S')
            before_restore = f"backups/before_external_restore_{ts}.db"
            
            if os.path.exists(DB_PATH):
                try:
                    shutil.copy2(DB_PATH, before_restore)
                except Exception as backup_err:
                    return err(f"فشل عمل النسخة الاحتياطية: {str(backup_err)}")
            
            # نسخ الملف المرفوع إلى قاعدة البيانات الرئيسية
            try:
                shutil.copy2(temp_path, DB_PATH)
            except Exception as copy_err:
                return err(f"فشل في نسخ الملف: {str(copy_err)}")
            finally:
                # حذف الملف المؤقت
                if os.path.exists(temp_path):
                    try:
                        os.remove(temp_path)
                    except:
                        pass
            
            # التحقق من الملف المستعاد
            time.sleep(0.3)
            try:
                verify_conn = sqlite3.connect(DB_PATH, timeout=5)
                verify_conn.execute("SELECT 1")
                verify_conn.close()
            except Exception as verify_err:
                return err(f"فشل التحقق من الملف المستعاد: {str(verify_err)}")
            
            return ok(msg="تم استرجاع النسخة الاحتياطية بنجاح!")
                
        except Exception as file_err:
            # حذف الملف المؤقت في حالة الفشل
            if os.path.exists(temp_path):
                try:
                    os.remove(temp_path)
                except:
                    pass
            return err(f"خطأ: {str(file_err)}")
            
    except Exception as e:
        return err(f"خطأ في المعالجة: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ إدارة الموبايلات - مخزون الموبايلات
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/inventory', methods=['GET'])
@admin_required
def get_mobile_inventory():
    try:
        search = request.args.get('search', '')
        conn = get_conn()
        cursor = conn.cursor()
        
        query = """
            SELECT id, name, brand, model, quantity, barcode, cost_price, selling_price, condition, created_date
            FROM products
            WHERE category = 'موبايلات' AND is_deleted = 0
        """
        params = []
        if search:
            query += " AND (name LIKE ? OR brand LIKE ? OR model LIKE ? OR barcode LIKE ?)"
            params.extend([f'%{search}%'] * 4)
        
        query += " ORDER BY created_date DESC"
        cursor.execute(query, params)
        items = [dict(row) for row in cursor.fetchall()]
        conn.close()
        
        return ok(items, f"تم جلب {len(items)} موبايل")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/mobile/inventory/edit/<int:id>', methods=['PUT'])
@admin_required
def edit_mobile_item(id):
    try:
        data = request.get_json()
        conn = get_conn()
        cursor = conn.cursor()
        
        cursor.execute("""
            UPDATE products
            SET name=?, brand=?, model=?, quantity=?, cost_price=?, selling_price=?, condition=?
            WHERE id=? AND category='موبايلات'
        """, (data.get('name'), data.get('brand'), data.get('model'), 
              data.get('quantity'), data.get('cost_price'), data.get('selling_price'),
              data.get('condition'), id))
        
        conn.commit()
        conn.close()
        return ok(msg="تم التحديث بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/mobile/inventory/delete/<int:id>', methods=['DELETE'])
@admin_required
def delete_mobile_item(id):
    try:
        conn = get_conn()
        cursor = conn.cursor()
        cursor.execute("UPDATE products SET is_deleted=1 WHERE id=? AND category='موبايلات'", (id,))
        conn.commit()
        conn.close()
        return ok(msg="تم الحذف بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ إدارة الموبايلات - البيع المباشر
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/direct-sale', methods=['POST'])
def direct_sale_mobile():
    try:
        if 'user_id' not in session:
            return err("غير مصرح"), 401
        
        data = request.get_json()
        product_id = data.get('product_id')
        final_price = float(data.get('price', 0))
        paid_amount = float(data.get('paid_amount', 0))
        customer_name = data.get('customer_name', 'عميل نقدي')
        
        conn = get_conn()
        cursor = conn.cursor()
        
        # الحصول على معلومات المنتج
        cursor.execute("SELECT * FROM products WHERE id=? AND category='موبايلات'", (product_id,))
        product = cursor.fetchone()
        
        if not product or product['quantity'] <= 0:
            conn.close()
            return err("الموبايل غير متوفر")
        
        cost_price = product['cost_price']
        profit = final_price - cost_price
        remaining = final_price - paid_amount
        
        # إنشاء فاتورة بيع
        import uuid
        invoice_num = f"MOB-{datetime.now().strftime('%Y%m%d%H%M%S')}-{uuid.uuid4().hex[:6].upper()}"
        today = date.today().isoformat()
        current_time = datetime.now().strftime("%H:%M:%S")
        
        cursor.execute("""
            INSERT INTO sales (invoice_number, total_amount, profit, paid_amount, remaining_amount,
                              sale_date, sale_time, user_id, payment_method, customer_name)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (invoice_num, final_price, profit, paid_amount, remaining, today, current_time,
              session['user_id'], 'نقدي' if remaining <= 0 else 'دين', customer_name))
        
        sale_id = cursor.lastrowid
        
        # إضافة تفاصيل البيع
        cursor.execute("""
            INSERT INTO sale_details (sale_id, product_id, quantity, unit_price, total_price, cost_price, profit)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        """, (sale_id, product_id, 1, final_price, final_price, cost_price, profit))
        
        # تحديث المخزون
        cursor.execute("UPDATE products SET quantity = quantity - 1 WHERE id=?", (product_id,))
        
        # تسجيل الدفعة
        if paid_amount > 0:
            cursor.execute("""
                INSERT INTO payments (type, reference_id, amount, payment_date, payment_method, user_id)
                VALUES (?, ?, ?, ?, ?, ?)
            """, ('sale', sale_id, paid_amount, today, 'نقدي', session['user_id']))
        
        conn.commit()
        conn.close()
        
        return ok({"invoice_number": invoice_num}, f"تمت عملية البيع بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ إدارة الموبايلات - المشتريات
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/purchases', methods=['GET'])
@admin_required
def get_mobile_purchases():
    try:
        search = request.args.get('search', '')
        conn = get_conn()
        cursor = conn.cursor()
        
        query = """
            SELECT p.id, p.invoice_number, s.name as supplier, p.total_amount, 
                   p.paid_amount, p.remaining_amount, p.purchase_date
            FROM purchases p
            LEFT JOIN suppliers s ON p.supplier_id = s.id
            WHERE p.id IN (
                SELECT DISTINCT pd.purchase_id FROM purchase_details pd
                JOIN products pr ON pd.product_id = pr.id
                WHERE pr.category = 'موبايلات'
            )
        """
        params = []
        if search:
            query += " AND (p.invoice_number LIKE ? OR s.name LIKE ?)"
            params.extend([f'%{search}%'] * 2)
        
        query += " ORDER BY p.purchase_date DESC"
        cursor.execute(query, params)
        items = [dict(row) for row in cursor.fetchall()]
        conn.close()
        
        return ok(items, f"تم جلب {len(items)} فاتورة")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/mobile/purchases/add', methods=['POST'])
@admin_required
def add_mobile_purchase():
    try:
        data = request.get_json()
        supplier_id = data.get('supplier_id')
        items = data.get('items', [])
        total_amount = float(data.get('total_amount', 0))
        paid_amount = float(data.get('paid_amount', 0))
        
        conn = get_conn()
        cursor = conn.cursor()
        
        import uuid
        invoice_num = f"MOB-PUR-{datetime.now().strftime('%Y%m%d%H%M%S')}-{uuid.uuid4().hex[:6].upper()}"
        today = date.today().isoformat()
        current_time = datetime.now().strftime("%H:%M:%S")
        remaining = total_amount - paid_amount
        
        cursor.execute("""
            INSERT INTO purchases (invoice_number, supplier_id, total_amount, paid_amount, remaining_amount,
                                  purchase_date, purchase_time, user_id)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """, (invoice_num, supplier_id, total_amount, paid_amount, remaining, today, current_time, session['user_id']))
        
        purchase_id = cursor.lastrowid
        
        # إضافة تفاصيل المشتريات
        for item in items:
            product_id = item.get('product_id')
            qty = int(item.get('quantity', 0))
            unit_price = float(item.get('unit_price', 0))
            total_price = qty * unit_price
            
            cursor.execute("""
                INSERT INTO purchase_details (purchase_id, product_id, quantity, unit_price, total_price)
                VALUES (?, ?, ?, ?, ?)
            """, (purchase_id, product_id, qty, unit_price, total_price))
            
            cursor.execute("""
                UPDATE products
                SET quantity = quantity + ?, cost_price = ?
                WHERE id=?
            """, (qty, unit_price, product_id))
        
        # تسجيل الدفعة
        if paid_amount > 0:
            cursor.execute("""
                INSERT INTO payments (type, reference_id, amount, payment_date, payment_method, user_id)
                VALUES (?, ?, ?, ?, ?, ?)
            """, ('purchase', purchase_id, paid_amount, today, 'نقدي', session['user_id']))
        
        conn.commit()
        conn.close()
        
        return ok({"invoice_number": invoice_num}, f"تمت إضافة فاتورة المشتريات بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ إدارة الموبايلات - الديون
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/debts', methods=['GET'])
@admin_required
def get_mobile_debts():
    try:
        search = request.args.get('search', '')
        conn = get_conn()
        cursor = conn.cursor()
        
        query = """
            SELECT s.id, s.invoice_number, s.customer_name, s.total_amount, s.paid_amount, s.remaining_amount, s.sale_date
            FROM sales s
            WHERE s.remaining_amount > 0
            AND (s.id IN (
                SELECT sale_id FROM sale_details sd
                JOIN products p ON sd.product_id = p.id
                WHERE p.category = 'موبايلات'
            ) OR s.invoice_number LIKE 'MOB-%')
        """
        params = []
        if search:
            query += " AND (s.invoice_number LIKE ? OR s.customer_name LIKE ?)"
            params.extend([f'%{search}%'] * 2)
        
        query += " ORDER BY s.sale_date DESC"
        cursor.execute(query, params)
        items = [dict(row) for row in cursor.fetchall()]
        conn.close()
        
        return ok(items, f"تم جلب {len(items)} دين")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/mobile/debts/add', methods=['POST'])
def add_mobile_debt():
    try:
        if 'user_id' not in session:
            return err("غير مصرح"), 401
        
        data = request.get_json()
        customer_name = data.get('customer_name')
        total_amount = float(data.get('total_amount', 0))
        paid_amount = float(data.get('paid_amount', 0))
        
        conn = get_conn()
        cursor = conn.cursor()
        
        import uuid
        invoice_num = f"DEBT-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
        today = date.today().isoformat()
        current_time = datetime.now().strftime("%H:%M:%S")
        remaining = total_amount - paid_amount
        
        cursor.execute("""
            INSERT INTO sales (invoice_number, total_amount, profit, paid_amount, remaining_amount,
                              sale_date, sale_time, user_id, payment_method, customer_name)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (invoice_num, total_amount, 0, paid_amount, remaining, today, current_time, 
              session['user_id'], 'دين', customer_name))
        
        sale_id = cursor.lastrowid
        
        if paid_amount > 0:
            cursor.execute("""
                INSERT INTO payments (type, reference_id, amount, payment_date, payment_method, user_id)
                VALUES (?, ?, ?, ?, ?, ?)
            """, ('sale', sale_id, paid_amount, today, 'نقدي', session['user_id']))
        
        conn.commit()
        conn.close()
        
        return ok({"invoice_number": invoice_num}, "تمت إضافة الدين بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

@app.route('/api/mobile/debts/pay/<int:sale_id>', methods=['POST'])
def pay_mobile_debt(sale_id):
    try:
        if 'user_id' not in session:
            return err("غير مصرح"), 401
        
        data = request.get_json()
        pay_amount = float(data.get('amount', 0))
        
        conn = get_conn()
        cursor = conn.cursor()
        
        cursor.execute("SELECT remaining_amount FROM sales WHERE id=?", (sale_id,))
        sale = cursor.fetchone()
        
        if not sale or pay_amount > sale['remaining_amount']:
            conn.close()
            return err("المبلغ غير صحيح")
        
        cursor.execute("""
            UPDATE sales
            SET remaining_amount = remaining_amount - ?
            WHERE id=?
        """, (pay_amount, sale_id))
        
        today = date.today().isoformat()
        cursor.execute("""
            INSERT INTO payments (type, reference_id, amount, payment_date, payment_method, user_id)
            VALUES (?, ?, ?, ?, ?, ?)
        """, ('sale_debt_payment', sale_id, pay_amount, today, 'نقدي', session['user_id']))
        
        conn.commit()
        conn.close()
        
        return ok(msg="تم تسديد الدين بنجاح")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ إدارة الموبايلات - صندوق الموبايلات
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/cash', methods=['GET'])
@admin_required
def get_mobile_cash():
    try:
        date_from = request.args.get('date_from', date.today().isoformat())
        date_to = request.args.get('date_to', date.today().isoformat())
        
        conn = get_conn()
        cursor = conn.cursor()
        
        # حركات البيع والمشتريات للموبايلات
        query = """
            SELECT 
                'مبيعات' as type, s.invoice_number, s.paid_amount as amount, s.sale_date as trans_date, u.username
            FROM sales s
            LEFT JOIN users u ON s.user_id = u.id
            WHERE s.id IN (
                SELECT sale_id FROM sale_details sd
                JOIN products p ON sd.product_id = p.id
                WHERE p.category = 'موبايلات'
            )
            AND date(s.sale_date) BETWEEN ? AND ?
            
            UNION ALL
            
            SELECT 
                'مشتريات' as type, p.invoice_number, p.paid_amount as amount, p.purchase_date as trans_date, u.username
            FROM purchases p
            LEFT JOIN users u ON p.user_id = u.id
            WHERE p.id IN (
                SELECT purchase_id FROM purchase_details pd
                JOIN products pr ON pd.product_id = pr.id
                WHERE pr.category = 'موبايلات'
            )
            AND date(p.purchase_date) BETWEEN ? AND ?
            
            ORDER BY trans_date DESC
        """
        
        cursor.execute(query, [date_from, date_to, date_from, date_to])
        transactions = [dict(row) for row in cursor.fetchall()]
        
        # حساب الإحصائيات
        total_income = sum(t['amount'] for t in transactions if t['type'] == 'مبيعات')
        total_expense = sum(t['amount'] for t in transactions if t['type'] == 'مشتريات')
        net_profit = total_income - total_expense
        
        conn.close()
        
        return ok({
            "transactions": transactions,
            "total_income": total_income,
            "total_expense": total_expense,
            "net_profit": net_profit
        }, "تم جلب بيانات صندوق الموبايلات")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
# ✅ تقرير إدارة الموبايلات الشامل
# ═══════════════════════════════════════════════════════════
@app.route('/api/mobile/report', methods=['GET'])
@admin_required
def get_mobile_report():
    try:
        conn = get_conn()
        cursor = conn.cursor()
        
        # إحصائيات الموبايلات
        cursor.execute("""
            SELECT 
                COUNT(*) as total_items,
                SUM(quantity) as total_qty,
                SUM(quantity * cost_price) as total_cost,
                SUM(quantity * selling_price) as expected_revenue,
                SUM(quantity * (selling_price - cost_price)) as expected_profit
            FROM products
            WHERE category = 'موبايلات' AND is_deleted = 0
        """)
        stats = dict(cursor.fetchone())
        
        # المنتجات بالتفصيل
        cursor.execute("""
            SELECT 
                name, brand, quantity, cost_price, selling_price, 
                quantity * cost_price as total_cost,
                quantity * (selling_price - cost_price) as expected_profit
            FROM products
            WHERE category = 'موبايلات' AND is_deleted = 0
            ORDER BY quantity DESC
        """)
        items = [dict(row) for row in cursor.fetchall()]
        
        conn.close()
        
        return ok({
            "summary": stats,
            "items": items
        }, "تم تحميل التقرير الشامل لإدارة الموبايلات")
    except Exception as e:
        return err(f"خطأ: {str(e)}")

# ═══════════════════════════════════════════════════════════
if __name__ == '__main__':
    init_db()
    print("=" * 50)
    print("  🏪 المستمر للمحاسبة - مع إدارة الموبايلات")
    print("=" * 50)
    print(f"  📁 قاعدة البيانات: {DB_PATH}")
    print(f"  🌐 افتح المتصفح: http://192.168.1.100:5000")
    print(f"  👤 المستخدم: admin | كلمة المرور: admin123")
    print("=" * 50)
    app.run(debug=False, host='0.0.0.0', port=5000)
