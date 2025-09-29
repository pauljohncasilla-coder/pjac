<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>JASS INSTALLATION SERVICES</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<style>
:root {
  --accent:#0f172a;
  --green:#10b981;
  --white:#ffffff;
  --danger:#ef4444;
  --warning:#fbbf24;
  --text:#111827;
}
*{box-sizing:border-box;}
body{
  margin:0;
  font-family:'Inter',sans-serif;
  background: linear-gradient(135deg,#0f172a,#10b981,#ffffff);
  background-size:400% 400%;
  color: var(--text);
  animation: gradientBG 15s ease infinite;
}
@keyframes gradientBG {
  0%{background-position:0% 50%;}
  50%{background-position:100% 50%;}
  100%{background-position:0% 50%;}
}
header{
  background: rgba(255,255,255,0.95);
  padding:16px 24px;
  display:flex;
  justify-content:space-between;
  align-items:center;
  box-shadow:0 12px 32px rgba(0,0,0,0.15);
}
header h1{margin:0;font-size:24px;color:var(--accent);}
nav button{
  margin-left:12px;
  padding:10px 18px;
  border:none;
  border-radius:8px;
  background: var(--accent);
  color:var(--white);
  cursor:pointer;
  font-weight:600;
  transition:0.3s;
}
nav button.active, nav button:hover{background:linear-gradient(90deg,#0f172a,#10b981);}
main{padding:24px;}
.card{
  background: rgba(255,255,255,0.95);
  padding:24px;
  border-radius:16px;
  box-shadow:0 12px 32px rgba(0,0,0,0.15);
  margin-bottom:24px;
  transition:0.3s;
}
.card:hover{transform:translateY(-2px);}
table{width:100%;border-collapse:collapse;}
th,td{padding:12px;border-bottom:1px solid #e5e7eb;text-align:left;}
th{background:#f3f4f6;}
.btn{padding:10px 16px;border-radius:8px;border:none;cursor:pointer;margin-right:6px;font-weight:600;transition:0.3s;}
.btn-primary{background: var(--accent); color:var(--white);}
.btn-primary:hover{background:linear-gradient(90deg,#0f172a,#10b981);}
.btn-success{background:var(--green); color:var(--white);}
.btn-success:hover{background:#059669;}
.btn-warning{background:var(--warning); color:#000;}
.btn-warning:hover{background:#f59e0b;}
.btn-danger{background:var(--danger); color:var(--white);}
.btn-danger:hover{background:#dc2626;}
.modal{position:fixed;inset:0;background:rgba(0,0,0,0.5);display:flex;align-items:center;justify-content:center;}
.modal .card{width:400px;padding:24px;border-radius:16px;}
input, select{padding:10px;border-radius:6px;border:1px solid #cbd5e1;width:100%;margin-bottom:12px;}
.login-container{height:100vh;display:flex;align-items:center;justify-content:center;}
.login-box{background: rgba(255,255,255,0.95); padding:40px; border-radius:16px; box-shadow:0 12px 32px rgba(0,0,0,0.15); width:360px; max-width:90%;}
.login-box h2{text-align:center;color:var(--accent); margin-bottom:24px;}
.toast{
  position:fixed; bottom:20px; right:20px;
  background: rgba(0,0,0,0.85); color:#fff;
  padding:16px 24px; border-radius:12px;
  box-shadow:0 8px 24px rgba(0,0,0,0.2);
  opacity:0; transform: translateY(50px);
  transition:0.5s;
  z-index:1000;
}
.toast.show{opacity:1; transform: translateY(0);}
@media(max-width:600px){main{padding:12px;}header h1{font-size:20px;}nav button{padding:8px 12px;font-size:14px;}}
</style>
</head>
<body>

<header style="display:none;" id="app-header">
  <h1>JASS INSTALLATION SERVICES</h1>
  <nav>
    <button id="nav-dashboard" class="active">Dashboard</button>
    <button id="nav-employees">Employees</button>
    <button id="nav-logs">Logs</button>
    <button id="btn-logout">Logout</button>
  </nav>
</header>

<main id="main-content"></main>
<div id="modal-root"></div>
<div id="toast"></div>

<script>
// ========== Data ==========
let users = [{username:'admin',password:'admin',role:'admin',name:'Administrator'}];
let employees = [
  {id:'E-1001',name:'Paul Santos',department:'IT',position:'Engineer',password:'pass123'},
  {id:'E-1002',name:'Maria Cruz',department:'HR',position:'Officer',password:'pass456'}
];
let logs = [];
let currentUser=null;
let currentEmployee=null;
function formatDate(d){return new Date(d).toLocaleString();}

// ========== Toast ==========
function showToast(msg){
  const t=document.getElementById('toast');
  t.innerText=msg; t.className='toast show';
  setTimeout(()=>{t.className='toast';},3000);
}

// ========== Landing ==========
function renderLanding(){
  document.getElementById('app-header').style.display="none";
  document.getElementById('main-content').innerHTML=`
  <div class="login-container">
    <div class="login-box">
      <h2>Welcome</h2>
      <button class="btn btn-primary" onclick="renderAdminLogin()">Admin Login</button>
      <button class="btn btn-primary" onclick="renderEmployeeLogin()">Employee Login</button>
    </div>
  </div>`;
}

// ========== Admin Login ==========
function renderAdminLogin(){
  document.getElementById('main-content').innerHTML=`
    <div class="login-container">
      <div class="login-box">
        <h2>Admin Login</h2>
        <input type="text" id="login-username" placeholder="Username">
        <input type="password" id="login-password" placeholder="Password">
        <button class="btn btn-primary" onclick="loginAdmin()">Login</button>
        <button class="btn" onclick="renderLanding()">Back</button>
      </div>
    </div>`;
}
function loginAdmin(){
  const u=document.getElementById('login-username').value.trim();
  const p=document.getElementById('login-password').value.trim();
  const match=users.find(x=>x.username===u && x.password===p && x.role==='admin');
  if(match){ currentUser=match; document.getElementById('app-header').style.display="flex"; showView('dashboard'); showToast('Welcome '+match.name);}
  else showToast('Invalid credentials');
}

// ========== Employee Login ==========
function renderEmployeeLogin(){
  document.getElementById('main-content').innerHTML=`
    <div class="login-container">
      <div class="login-box">
        <h2>Employee Login</h2>
        <input type="text" id="emp-id-login" placeholder="Employee ID">
        <input type="password" id="emp-pass-login" placeholder="Password">
        <button class="btn btn-primary" onclick="loginEmployee()">Login</button>
        <button class="btn" onclick="renderLanding()">Back</button>
      </div>
    </div>`;
}
function loginEmployee(){
  const id=document.getElementById('emp-id-login').value.trim();
  const pass=document.getElementById('emp-pass-login').value.trim();
  const emp=employees.find(e=>e.id===id && e.password===pass);
  if(emp){ currentEmployee=emp; renderEmployeePortal(); showToast('Welcome '+emp.name);}
  else showToast('Invalid employee credentials');
}

// ========== Employee Portal ==========
function renderEmployeePortal(){
  const emp=currentEmployee;
  const empLogs=logs.filter(l=>l.empId===emp.id);
  let rows=''; [...empLogs].reverse().forEach(l=>rows+=`<tr><td>${l.time}</td><td>${l.action}</td></tr>`);
  document.getElementById('main-content').innerHTML=`
    <div class="card">
      <h2>Welcome, ${emp.name}</h2>
      <p><strong>ID:</strong> ${emp.id}<br><strong>Dept:</strong> ${emp.department}<br><strong>Position:</strong> ${emp.position}</p>
      <button class="btn btn-success" onclick="employeeAction('Time In')">Time In</button>
      <button class="btn btn-warning" onclick="employeeAction('Time Out')">Time Out</button>
      <button class="btn btn-danger" onclick="logoutEmployee()">Logout</button>
    </div>
    <div class="card">
      <h3>Your Logs</h3>
      <button class="btn btn-success" onclick="exportLogs()">Export Logs</button>
      <table><thead><tr><th>Time</th><th>Action</th></tr></thead><tbody>${rows}</tbody></table>
    </div>`;
}

function employeeAction(action){
  const emp=currentEmployee;
  const empLogs=logs.filter(l=>l.empId===emp.id);
  const last=empLogs.slice(-1)[0];
  if(last && last.action===action){ showToast('Already did '+action+' at '+last.time); return; }
  logs.push({time:formatDate(new Date()), empId:emp.id, action});
  showToast(action+' recorded');
  renderEmployeePortal();
}
function logoutEmployee(){ currentEmployee=null; renderLanding(); }

// ========== Admin Views ==========
function showView(view){
  document.querySelectorAll('nav button').forEach(b=>b.classList.remove('active'));
  if(view!=='logout') document.getElementById('nav-'+view).classList.add('active');
  if(view==='dashboard') renderDashboard();
  else if(view==='employees') renderEmployeeTable();
  else if(view==='logs') renderLogs();
}
function renderDashboard(){
  let deptCounts={}, empCounts={};
  employees.forEach(e=>{deptCounts[e.department]=0; empCounts[e.name]=0;});
  logs.forEach(l=>{if(l.action==='Time In'){const e=employees.find(x=>x.id===l.empId); if(e){deptCounts[e.department]++; empCounts[e.name]++;}}});
  const deptLabels=Object.keys(deptCounts), deptData=Object.values(deptCounts);
  const empLabels=Object.keys(empCounts), empData=Object.values(empCounts);
  document.getElementById('main-content').innerHTML=`
    <div class="card"><h3>Attendance by Department</h3><canvas id="deptChart"></canvas></div>
    <div class="card"><h3>Attendance by Employee</h3><canvas id="empChart"></canvas></div>`;
  new Chart(document.getElementById('deptChart'),{type:'bar',data:{labels:deptLabels,datasets:[{label:'Time In',data:deptData,backgroundColor:'rgba(79,70,229,0.8)'}]}, options:{responsive:true,plugins:{legend:{display:false}}}});
  new Chart(document.getElementById('empChart'),{type:'bar',data:{labels:empLabels,datasets:[{label:'Time In',data:empData,backgroundColor:'rgba(16,185,129,0.8)'}]}, options:{responsive:true,plugins:{legend:{display:false}}}});
}

function renderEmployeeTable(){
  let rows=''; employees.forEach(e=>{
    const empLogs=logs.filter(l=>l.empId===e.id);
    const lastIn=empLogs.filter(l=>l.action==='Time In').slice(-1)[0];
    const lastOut=empLogs.filter(l=>l.action==='Time Out').slice(-1)[0];
    rows+=`<tr>
      <td>${e.id}</td><td>${e.name}</td><td>${e.department}</td><td>${e.position}</td>
      <td>${lastIn?lastIn.time:'-'}</td><td>${lastOut?lastOut.time:'-'}</td>
      <td>
        <button class="btn btn-warning" onclick="openEmployeeModal('edit','${e.id}')">Edit</button>
        <button class="btn btn-danger" onclick="deleteEmployee('${e.id}')">Delete</button>
      </td>
    </tr>`;
  });
  document.getElementById('main-content').innerHTML=`
    <div class="card">
      <h3>Employees <button class="btn btn-success" onclick="openEmployeeModal('add')">Add Employee</button> <button class="btn btn-success" onclick="exportEmployees()">Export Excel</button></h3>
      <table><thead><tr><th>ID</th><th>Name</th><th>Dept</th><th>Position</th><th>Last In</th><th>Last Out</th><th>Actions</th></tr></thead><tbody>${rows}</tbody></table>
    </div>`;
}

function openEmployeeModal(mode,id=''){
  let emp={id:'',name:'',department:'',position:'',password:''};
  if(mode==='edit') emp=employees.find(e=>e.id===id);
  document.getElementById('modal-root').innerHTML=`
    <div class="modal">
      <div class="card">
        <h3>${mode==='add'?'Add':'Edit'} Employee</h3>
        ${mode==='edit'?`<input type="text" id="emp-id" placeholder="ID" value="${emp.id}" disabled>`:''}
        <input type="text" id="emp-name" placeholder="Name" value="${emp.name}">
        <input type="text" id="emp-dept" placeholder="Department" value="${emp.department}">
        <input type="text" id="emp-pos" placeholder="Position" value="${emp.position}">
        <input type="password" id="emp-pass" placeholder="Password" value="${emp.password}">
        <button class="btn btn-primary" onclick="saveEmployee('${mode}','${id}')">Save</button>
        <button class="btn btn-danger" onclick="closeModal()">Cancel</button>
      </div>
    </div>`;
}

function saveEmployee(mode,id){
  const e={
    name:document.getElementById('emp-name').value.trim(),
    department:document.getElementById('emp-dept').value.trim(),
    position:document.getElementById('emp-pos').value.trim(),
    password:document.getElementById('emp-pass').value.trim()
  };
  if(!e.name){showToast('Name is required'); return;}

  if(mode==='add'){
    // Generate unique ID
    let maxId = 1000;
    if(employees.length > 0){
      employees.forEach(emp => {
        const num = parseInt(emp.id.split('-')[1]);
        if(num > maxId) maxId = num;
      });
    }
    e.id = `E-${maxId + 1}`;
    employees.push(e);
  } else {
    const idx=employees.findIndex(x=>x.id===id);
    e.id = id;
    employees[idx]=e;
  }

  closeModal(); renderEmployeeTable(); showToast('Employee saved');
}

function deleteEmployee(id){if(confirm('Delete '+id+'?')){employees=employees.filter(e=>e.id!==id); renderEmployeeTable(); showToast('Employee deleted');}}
function closeModal(){document.getElementById('modal-root').innerHTML='';}

// ========== Logs ==========
function renderLogs(){
  let rows=''; [...logs].reverse().forEach(l=>rows+=`<tr><td>${l.time}</td><td>${l.empId}</td><td>${l.action}</td></tr>`);
  document.getElementById('main-content').innerHTML=`
    <div class="card">
      <h3>Attendance Logs <button class="btn btn-success" onclick="exportLogs()">Export Excel</button></h3>
      <table><thead><tr><th>Time</th><th>Employee ID</th><th>Action</th></tr></thead><tbody>${rows}</tbody></table>
    </div>`;
}

// ========== Excel Export ==========
function exportEmployees(){
  const ws=XLSX.utils.json_to_sheet(employees);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,'Employees');
  XLSX.writeFile(wb,'Employees.xlsx');
}
function exportLogs(){
  const ws=XLSX.utils.json_to_sheet(logs);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,'Logs');
  XLSX.writeFile(wb,'Logs.xlsx');
}

// ========== Nav ==========
document.getElementById('nav-dashboard').onclick=()=>showView('dashboard');
document.getElementById('nav-employees').onclick=()=>showView('employees');
document.getElementById('nav-logs').onclick=()=>showView('logs');
document.getElementById('btn-logout').onclick=()=>{currentUser=null; renderLanding(); showToast('Logged out');};

// ========== Boot ==========
renderLanding();
</script>
</body>
</html>
