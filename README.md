<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Jackson Deliveries - Connected</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:sans-serif}
body{background:#f5f5f7;padding:12px}
.top{display:flex;justify-content:space-between;align-items:center;background:white;border-radius:16px;padding:12px 16px;margin-bottom:16px}
.logo{display:flex;gap:10px;align-items:center}
.logo-icon{width:40px;height:40px;background:#ff6b1a;border-radius:12px;display:flex;align-items:center;justify-content:center;color:white;font-weight:800}
.nav button{border:none;padding:10px 18px;border-radius:24px;font-weight:700}
.active{background:#111;color:white}
.inactive{background:#f0f0f0}
.card{background:white;border-radius:20px;padding:18px;margin-bottom:16px;box-shadow:0 4px 20px rgba(0,0,0,.06)}
.input{width:100%;padding:14px;border:1.5px solid #e5e5e5;border-radius:14px;margin-bottom:12px;font-size:15px}
.btn-black{width:100%;background:#111;color:white;border:none;padding:16px;border-radius:16px;font-weight:800;font-size:16px}
.order-item{background:#fafafa;border-radius:14px;padding:14px;margin-bottom:10px;border-left:4px solid #ff6b1a}
.badge{background:#ffe9d6;color:#ff6b1a;padding:4px 12px;border-radius:20px;font-weight:800;font-size:12px}
.status{font-size:12px;padding:3px 8px;border-radius:10px;font-weight:700}
.NEW{background:#fff3cd} .ASSIGNED{background:#cce5ff} .DELIVERED{background:#d4edda}
.admin-only{display:none}
</style>
</head>
<body>

<div class="top">
<div class="logo"><div class="logo-icon">J</div><div><b>Jackson</b><br><span style="color:#ff6b1a;font-weight:800;font-size:12px">DELIVERIES</span></div></div>
<div class="nav"><button id="btnOrder" class="active" onclick="showOrder()">Order</button><button id="btnAdmin" class="inactive" onclick="showAdmin()">Admin</button></div>
</div>

<div id="orderView">
<div class="card">
<h3 style="margin-bottom:12px">Create New Delivery</h3>
<input id="phone" class="input" placeholder="Phone 07XXXXXXXX">
<div style="display:flex;gap:12px"><input id="pickup" class="input" placeholder="Pickup Location"><input id="dropoff" class="input" placeholder="Dropoff Location"></div>
<textarea id="item" class="input" placeholder="What to deliver? e.g Tecno Spark 20 - UGX 420,000"></textarea>
<button class="btn-black" onclick="placeOrder()">Place Order - Save to Firebase 🚀</button>
<p id="msg" style="text-align:center;font-size:13px;color:#00c853;margin-top:10px;font-weight:700"></p>
</div>
<div class="card"><div style="display:flex;justify-content:space-between;margin-bottom:10px"><h3>Live Orders</h3><span id="liveCount" class="badge">0</span></div><div id="liveList"></div></div>
</div>

<div id="adminView" class="admin-only">
<div class="card"><div style="display:flex;justify-content:space-between;margin-bottom:10px"><h3>ORDERS - ASSIGN RIDER</h3><span id="adminCount" class="badge">0</span></div><div id="adminList"></div></div>
<div class="card"><h3>Rider View - YOUR JOB</h3><div id="riderList" style="margin-top:10px"></div></div>
</div>

<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
<script>
const firebaseConfig={
apiKey:"AIzaSyD0CHgFbz1XwD9t7edWU7VQ0-zXw5vz5Lw",
authDomain:"jackson-deliveries.firebaseapp.com",
databaseURL:"https://jackson-deliveries-default-rtdb.firebaseio.com",
projectId:"jackson-deliveries",
storageBucket:"jackson-deliveries.firebasestorage.app",
messagingSenderId:"464029770729",
appId:"1:464029770729:web:142003a1e109beb872e178"
};
firebase.initializeApp(firebaseConfig);
const db=firebase.database();
const ORDERS_REF="orders";
let orders=[];
function showOrder(){document.getElementById('orderView').style.display='block';document.getElementById('adminView').style.display='none';document.getElementById('btnOrder').className='active';document.getElementById('btnAdmin').className='inactive'}
function showAdmin(){document.getElementById('orderView').style.display='none';document.getElementById('adminView').style.display='block';document.getElementById('btnAdmin').className='active';document.getElementById('btnOrder').className='inactive';render()}
function placeOrder(){
const phone=document.getElementById('phone').value.trim();
const pickup=document.getElementById('pickup').value.trim();
const dropoff=document.getElementById('dropoff').value.trim();
const item=document.getElementById('item').value.trim();
if(!phone||!pickup||!dropoff||!item){document.getElementById('msg').innerText='Fill all fields!';return}
const order={id:Date.now(),phone,pickup,dropoff,item,status:'NEW',rider:'',time:new Date().toLocaleString(),amount:item.match(/UGX\s*[\d,]+/i)?.[0]||'UGX 15,000'};
document.getElementById('msg').innerText='Saving to Firebase...';
db.ref(ORDERS_REF).push(order).then(()=>{
document.getElementById('msg').innerText='✅ Saved! Check APK Admin!';
document.getElementById('phone').value='';document.getElementById('pickup').value='';document.getElementById('dropoff').value='';document.getElementById('item').value='';
setTimeout(()=>document.getElementById('msg').innerText='',3000)
})
}
function render(){
document.getElementById('liveCount').innerText=orders.length;
document.getElementById('adminCount').innerText=orders.length;
const liveList=document.getElementById('liveList');
const adminList=document.getElementById('adminList');
const riderList=document.getElementById('riderList');
if(orders.length==0){liveList.innerHTML='<p style="color:#999;text-align:center">No orders yet</p>';adminList.innerHTML='<p style="color:#999">No orders</p>';riderList.innerHTML='<p style="color:#999">No jobs</p>';return}
liveList.innerHTML=orders.map(o=>`<div class="order-item"><div style="font-size:12px;color:#888">${o.time} • ${o.phone}</div><b>${o.item}</b><div>${o.pickup} → ${o.dropoff}</div><span class="status ${o.status}">${o.status} ${o.rider?'• '+
