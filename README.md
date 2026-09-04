# kings-garage
from pathlib import Path
import zipfile

html = r'''<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>KING'S GARAGE 👑</title>
<style>
  :root { --bg:#0b0d12; --card:#151922; --line:#2a3140; --text:#f4f6fb; --muted:#aeb7c8; --accent:#5b9dff; }
  *{box-sizing:border-box}
  body{margin:0;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;background:var(--bg);color:var(--text)}
  header{padding:32px 20px 24px;background:linear-gradient(135deg,#111827,#0b0d12);border-bottom:1px solid var(--line)}
  h1{margin:0;font-size:30px}.sub{color:var(--muted);margin-top:8px}
  nav{display:flex;gap:8px;overflow:auto;padding:14px 16px;position:sticky;top:0;background:#0b0d12;border-bottom:1px solid var(--line)}
  nav button{border:1px solid var(--line);background:var(--card);color:var(--text);padding:10px 14px;border-radius:12px;white-space:nowrap}
  main{max-width:900px;margin:auto;padding:18px}
  section{display:none}.active{display:block}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:14px}
  .card{background:var(--card);border:1px solid var(--line);border-radius:18px;padding:18px;margin-bottom:16px}
  .stat{font-size:28px;font-weight:800;margin-top:8px}
  input,select{width:100%;padding:13px;border-radius:12px;border:1px solid var(--line);background:#0e1219;color:var(--text);margin:7px 0}
  button.primary{background:var(--accent);color:#06111f;border:0;padding:13px 18px;border-radius:12px;font-weight:800;width:100%}
  .row{display:flex;justify-content:space-between;gap:12px;padding:12px 0;border-bottom:1px solid var(--line)}
  .row:last-child{border-bottom:0}
  .small{font-size:13px;color:var(--muted)}
  .danger{background:#3b1820;color:#ffb7c3;border:0;padding:7px 10px;border-radius:9px}
  .empty{text-align:center;color:var(--muted);padding:25px}
</style>
</head>
<body>
<header>
  <h1>👑 KING'S GARAGE</h1>
  <div class="sub">My Car • My Build • My Expenses</div>
</header>

<nav>
  <button onclick="show('home')">🏠 Home</button>
  <button onclick="show('garage')">🔧 Garage</button>
  <button onclick="show('fuel')">⛽ Fuel</button>
  <button onclick="show('expenses')">💸 Expenses</button>
</nav>

<main>
<section id="home" class="active">
  <div class="card">
    <h2>🚗 รถของฉัน</h2>
    <input id="carName" placeholder="เช่น Honda Civic FE" oninput="saveProfile()">
    <input id="carYear" placeholder="ปีรถ เช่น 2024" oninput="saveProfile()">
  </div>
  <div class="grid">
    <div class="card"><div class="small">ค่าใช้จ่ายทั้งหมด</div><div class="stat" id="totalExpense">0 ฿</div></div>
    <div class="card"><div class="small">ของแต่ง</div><div class="stat" id="modsCount">0 ชิ้น</div></div>
    <div class="card"><div class="small">ประวัติเติมน้ำมัน</div><div class="stat" id="fuelCount">0 ครั้ง</div></div>
  </div>
  <div class="card"><h2 id="welcome">พร้อมสร้างตำนาน 👑</h2><p class="small">ข้อมูลทั้งหมดถูกเก็บไว้ในเบราว์เซอร์เครื่องนี้</p></div>
</section>

<section id="garage">
  <div class="card">
    <h2>🔧 เพิ่มของแต่ง</h2>
    <input id="modName" placeholder="เช่น ล้อ TE37">
    <input id="modPrice" type="number" placeholder="ราคา (บาท)">
    <button class="primary" onclick="addMod()">เพิ่มเข้าการ์ด</button>
  </div>
  <div class="card"><h2>รายการของแต่ง</h2><div id="modsList"></div></div>
</section>

<section id="fuel">
  <div class="card">
    <h2>⛽ บันทึกการเติมน้ำมัน</h2>
    <input id="fuelKm" type="number" placeholder="ระยะทางที่วิ่ง (กม.)">
    <input id="fuelLiters" type="number" step="0.01" placeholder="ปริมาณน้ำมัน (ลิตร)">
    <input id="fuelCost" type="number" placeholder="ค่าใช้จ่าย (บาท)">
    <button class="primary" onclick="addFuel()">บันทึก</button>
  </div>
  <div class="card"><h2>ประวัติ & อัตราสิ้นเปลือง</h2><div id="fuelList"></div></div>
</section>

<section id="expenses">
  <div class="card">
    <h2>💸 เพิ่มค่าใช้จ่าย</h2>
    <input id="expenseName" placeholder="เช่น เปลี่ยนน้ำมันเครื่อง">
    <input id="expenseAmount" type="number" placeholder="จำนวนเงิน (บาท)">
    <select id="expenseType"><option>🔧 ซ่อมบำรุง</option><option>✨ ของแต่ง</option><option>🛞 ยาง/ล้อ</option><option>📄 อื่น ๆ</option></select>
    <button class="primary" onclick="addExpense()">เพิ่มรายการ</button>
  </div>
  <div class="card"><h2>รายการค่าใช้จ่าย</h2><div id="expenseList"></div></div>
</section>
</main>

<script>
const $=id=>document.getElementById(id);
let mods=JSON.parse(localStorage.getItem('kg_mods')||'[]');
let fuel=JSON.parse(localStorage.getItem('kg_fuel')||'[]');
let expenses=JSON.parse(localStorage.getItem('kg_expenses')||'[]');

function saveAll(){localStorage.setItem('kg_mods',JSON.stringify(mods));localStorage.setItem('kg_fuel',JSON.stringify(fuel));localStorage.setItem('kg_expenses',JSON.stringify(expenses));render();}
function saveProfile(){localStorage.setItem('kg_car',$('carName').value);localStorage.setItem('kg_year',$('carYear').value);render();}
function show(id){document.querySelectorAll('section').forEach(x=>x.classList.remove('active'));$(id).classList.add('active');}
function fmt(n){return Number(n||0).toLocaleString('th-TH')+' ฿'}
function list(el,arr,fn){$(el).innerHTML=arr.length?arr.map(fn).join(''):'<div class="empty">ยังไม่มีข้อมูล</div>'}
function addMod(){if(!$('modName').value)return;mods.unshift({n:$('modName').value,p:+$('modPrice').value||0});$('modName').value='';$('modPrice').value='';saveAll();}
function addFuel(){let km=+$('fuelKm').value,l=+$('fuelLiters').value,c=+$('fuelCost').value;if(!km||!l)return;fuel.unshift({km,l,c});$('fuelKm').value=$('fuelLiters').value=$('fuelCost').value='';saveAll();}
function addExpense(){if(!$('expenseName').value||!$('expenseAmount').value)return;expenses.unshift({n:$('expenseName').value,a:+$('expenseAmount').value,t:$('expenseType').value});$('expenseName').value=$('expenseAmount').value='';saveAll();}
function del(type,i){if(type==='m')mods.splice(i,1);if(type==='f')fuel.splice(i,1);if(type==='e')expenses.splice(i,1);saveAll();}
function render(){
  const car=localStorage.getItem('kg_car')||'รถของฉัน';
  $('carName').value=localStorage.getItem('kg_car')||'';$('carYear').value=localStorage.getItem('kg_year')||'';
  $('welcome').textContent='👑 '+car;
  $('modsCount').textContent=mods.length+' ชิ้น';$('fuelCount').textContent=fuel.length+' ครั้ง';
  const total=expenses.reduce((s,x)=>s+x.a,0)+mods.reduce((s,x)=>s+x.p,0)+fuel.reduce((s,x)=>s+x.c,0);
  $('totalExpense').textContent=fmt(total);
  list('modsList',mods,(x,i)=>`<div class="row"><span>${x.n}<br><small class="small">${fmt(x.p)}</small></span><button class="danger" onclick="del('m',${i})">ลบ</button></div>`);
  list('fuelList',fuel,(x,i)=>`<div class="row"><span>${x.km} กม. • ${x.l} ลิตร<br><small class="small">${(x.km/x.l).toFixed(2)} km/L • ${fmt(x.c)}</small></span><button class="danger" onclick="del('f',${i})">ลบ</button></div>`);
  list('expenseList',expenses,(x,i)=>`<div class="row"><span>${x.n}<br><small class="small">${x.t} • ${fmt(x.a)}</small></span><button class="danger" onclick="del('e',${i})">ลบ</button></div>`);
}
render();
</script>
</body>
</html>'''

readme = """# KING'S GARAGE 👑

เว็บบันทึกข้อมูลรถแบบง่าย

## ฟีเจอร์
- บันทึกข้อมูลรถ
- บันทึกของแต่ง
- บันทึกการเติมน้ำมันและคำนวณ km/L
- บันทึกค่าใช้จ่าย
- เก็บข้อมูลในเบราว์เซอร์

