<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no"/>
<meta name="theme-color" content="#070d14"/>
<meta name="apple-mobile-web-app-capable" content="yes"/>
<meta name="apple-mobile-web-app-title" content="RFT"/>
<meta name="description" content="Reading Fluency Tracker - Telugu, Hindi, English"/>
<title>Reading Fluency Tracker</title>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+Telugu:wght@400;600&family=Noto+Sans+Devanagari:wght@400;600&display=swap" rel="stylesheet"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
*{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%;background:#070d14;color:#d4e8f0;font-family:'Inter',system-ui,sans-serif;-webkit-font-smoothing:antialiased;overscroll-behavior:none}
button{-webkit-tap-highlight-color:transparent;font-family:inherit;cursor:pointer}
input,textarea,select{font-family:inherit}
::-webkit-scrollbar{width:3px;height:3px}
::-webkit-scrollbar-thumb{background:rgba(255,255,255,0.1);border-radius:3px}
.telugu{font-family:'Noto Sans Telugu',sans-serif}
.hindi{font-family:'Noto Sans Devanagari',sans-serif}
#app{min-height:100vh}
.hidden{display:none!important}
</style>
</head>
<body>
<div id="app"></div>
<script>
// ═══════════════════════════════════════════════
// SUPABASE CONFIG — Vinnu224's Project
// ═══════════════════════════════════════════════
const SUPA_URL = "https://bkoofkedsbeylquxgdiu.supabase.co";
const SUPA_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImJrb29ma2Vkc2JleWxxdXhnZGl1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzczNDI2MTgsImV4cCI6MjA5MjkxODYxOH0.ZMB0Cj9jsHnf93D1AT6lQmLBL7rU9Mko0KJhAhHHMmE";
const supa = supabase.createClient(SUPA_URL, SUPA_KEY, {
  auth: {
    persistSession: true,
    storageKey: "rft-auth",
    storage: window.localStorage,
    autoRefreshToken: true,
    detectSessionInUrl: false
  }
});
// ═══════════════════════════════════════════════
// CONSTANTS
// ═══════════════════════════════════════════════
const MAX_TEACHERS = 30;
const MAX_STUDENTS = 60;
const TIMER_SECS = 60;
const LANGS = {
telugu: {
label:"తెలుగు", tag:"TE", color:"#f59e0b", cls:"telugu",
passages:[
{level:"Beginner", title:"అమ్మ", words:18, text:"అమ్మ నాకు అన్నం పెడుతుంది. నేను బడికి వెళ్తాను. బడిలో చదువుతాను. అమ్మ నన్ను ప్రేమిస్తుంది. నేను అమ్మను ప్రేమిస్తాను. మేము కలిసిఉంటాము."},
{level:"Intermediate",title:"మన గ్రామం", words:38, text:"మన గ్రామం చాలా అందంగా ఉంటుంది. చుట్టూ పచ్చని చెట్లు ఉంటాయి. నదిలో స్వచ్ఛమైన నీళ్ళు పవ్రహిస్తాయి. రైతులు పొలాల్లో కష్టపడిపని చేస్తారు. పిల్లలు ఆడుకుంటూ సంతోషంగా ఉంటారు. సాయంతం్రపూట పక్షులు గూళ్ళకు తిరిగివస్తాయి."},
{level:"Advanced", title:"జ్ఞానం", words:42, text:"చదువు మనకు జ్ఞానాన్ని ఇస్తుంది. మంచి పుస్తకాలు చదవడం వలన మన మనసు విశాలమవుతుంది. విద్య ద్వా రా మనం జీవితంలో ముందుకు సాగవచ్చు . తల్లిదండ్రులు పిల్లల చదువు కోసం కష్టపడతారు. మంచి ఉపాధ్యా యులు విద్యా ర్థులను సరైన దారిలో నడిపిస్తారు."}
]
},
hindi: {
label:"हिन्दी", tag:"HI", color:"#22c55e", cls:"hindi",
passages:[
{level:"Beginner", title:"मेरा घर", words:32, text:"मेरा घर बहुत सुंदर है। घर में माँ, पिताजी और मैं रहते हैं। माँ खाना बनाती है। पिताजी काम पर जाते हैं। मैं स्कूल जाता हूँ। हम सब मिलकर खुश रहते हैं।"},
{level:"Intermediate",title:"वर्षा ऋतु", words:38, text:"वर्षा ऋतु में काले बादल आकाश में छा जाते हैं। बिजली चमकती है और बादल गरजते हैं। मेंढक टर्र-टर्र करने लगते हैं। किसानों के खेत हरे-भरे हो जाते हैं। बच्चे बारिश में खेलकर बहुत खुश होते हैं।"},
{level:"Advanced", title:"परिश्रम का महत्व", words:42, text:"परिश्रम सफलता की कुंजी है। जो व्यक्ति मेहनत करता है, वह जीवन में अवश्य आगे बढ़ता है। महान वैज्ञानिक और विद्वान अपनी कड़ी मेहनत से ही प्रसिद्ध हुए। विद्यार्थियों को चाहिए कि वे अपनी पढ़ाई में पूरा ध्यान लगाएँ।"}
]
},
english: {
label:"English", tag:"EN", color:"#3b82f6", cls:"",
passages:[
{level:"Beginner", title:"My Family", words:36, text:"I have a big family. My mother cooks food for us. My father goes to work every day. My sister and I go to school. We play together in the evening. We love our family very much."},
{level:"Intermediate",title:"The Forest", words:48, text:"The forest is home to many animals and birds. Tall trees provide shade and fresh air. Deer roam freely through the green meadows. Monkeys swing from branch to branch high above. The sound of a flowing stream fills the air. Every creature depends on the forest to survive."},
{level:"Advanced", title:"The Importance of Education", words:50, text:"Education is the foundation of a prosperous society. It empowers individuals to think critically and make informed decisions. Through learning, we discover new ideas and broaden our understanding of the world. Teachers play an essential role in shaping young minds. Every child deserves access to quality education regardless of background."}
]
}
};
const wc = w => w>=80?"#22c55e":w>=50?"#f59e0b":"#ef4444";
const wl = w => w>=80?"Proficient":w>=50?"Developing":"Needs Support";
const today = () => new Date().toISOString().slice(0,10);
const fmtDate = iso => { if(!iso)return""; const[y,m,d]=iso.split("-"); return`${d}/${m}/${y}`; };
// ═══════════════════════════════════════════════
// STATE
// ═══════════════════════════════════════════════
let STATE = {
screen: "loading",
user: null, profile: null, school: null,
classes:[], students:[], records:[],
selClass:null, selStudent:null,
lang:null, passage:null, assDate:today(),
timer:TIMER_SECS, running:false, started:false,
words:0, errs:0,
timerInterval:null,
toast:null, toastTimer:null,
confirm:null,
editClass:null, editStudent:null,
rptFrom:"", rptTo:"", rptLang:"all",
hFrom:"", hTo:"",
bulkText:"", busyBulk:false,
authTab:"login",
authForm:{},
authError:"", authBusy:false,
online:navigator.onLine,
};
function setState(patch){ Object.assign(STATE,patch); render(); }
function showToast(msg){ clearTimeout(STATE.toastTimer); setState({toast:msg, toastTimer:setTimeout(()=>setState({toast:null}),3000)}); }
// ═══════════════════════════════════════════════
// SUPABASE DB HELPERS
// ═══════════════════════════════════════════════
async function dbInit(){
const sql = `
create table if not exists schools (
id text primary key, name text, district text, state text,
admin_uid text, teacher_count int default 1, created_at timestamptz default now()
);
create table if not exists teachers (
id text primary key, school_id text, name text, email text,
role text default 'teacher', last_login timestamptz default now(), created_at timestamptz default now()
);
create table if not exists classes (
id text primary key, school_id text, teacher_id text, teacher_name text,
name text, grade text, section text, created_at timestamptz default now()
);
create table if not exists students (
id text primary key, school_id text, class_id text,
name text, roll_no int, created_at timestamptz default now()
);
create table if not exists records (
id text primary key, school_id text, class_id text, teacher_id text,
student_id text, student_name text, roll_no int,
language text, lang_label text, level text, title text,
words_read int, errors int, wcpm int, accuracy int,
iso_date text, date text, time text, created_at timestamptz default now()
);
`;
await supa.rpc('exec_sql', {sql}).catch(()=>{});
}
async function loadProfile(uid){
const {data} = await supa.from('teachers').select('*').eq('id',uid).single();
if(!data) return null;
STATE.profile = data;
const {data:sch} = await supa.from('schools').select('*').eq('id',data.school_id).single();
STATE.school = sch;
return data;
}
async function loadClasses(){
if(!STATE.profile) return;
const {data} = await supa.from('classes').select('*')
.eq('teacher_id', STATE.profile.id).order('created_at');
STATE.classes = data || [];
}
async function loadStudents(classId){
const {data} = await supa.from('students').select('*')
.eq('class_id', classId).order('roll_no');
STATE.students = data || [];
}
async function loadRecords(classId){
const {data} = await supa.from('records').select('*')
.eq('class_id', classId).order('iso_date', {ascending:false});
STATE.records = data || [];
}
// ═══════════════════════════════════════════════
// AUTH
// ═══════════════════════════════════════════════
supa.auth.onAuthStateChange(async (event, session)=>{
if(session?.user){
STATE.user = session.user;
const p = await loadProfile(session.user.id);
if(p){ await loadClasses(); setState({screen:"home"}); }
else setState({screen:"auth"});
} else {
STATE.user=null; STATE.profile=null; STATE.school=null;
setState({screen:"auth"});
}
});
async function authRegisterSchool(f){
setState({authBusy:true,authError:""});
try{
const {data,error} = await supa.auth.signUp({email:f.email,password:f.password,options:{data:{name:f.name}}});
if(error) throw error;
const uid = data.user.id;
const schoolId = "sch-"+Date.now()+"-"+Math.random().toString(36).slice(2,7);
await supa.from('schools').insert({id:schoolId,name:f.schoolName,district:f.district||"",state:f.state||"",admin_uid:uid,teacher_count:1});
await supa.from('teachers').insert({id:uid,school_id:schoolId,name:f.name,email:f.email,role:"admin"});
STATE.profile = {id:uid,school_id:schoolId,name:f.name,email:f.email,role:"admin"};
STATE.school = {id:schoolId,name:f.schoolName};
await loadClasses();
setState({screen:"home",authBusy:false});
}catch(e){ setState({authBusy:false,authError:e.message||"Registration failed. Try again."}); }
}
async function authRegisterTeacher(f){
setState({authBusy:true,authError:""});
try{
const {data:sch} = await supa.from('schools').select('*').eq('id',f.schoolCode).single();
if(!sch){ setState({authBusy:false,authError:"School code not found. Check with your principal."}); return; }
if((sch.teacher_count||0)>=MAX_TEACHERS){ setState({authBusy:false,authError:"School has reached 30 teacher limit."}); return; }
const {data,error} = await supa.auth.signUp({email:f.email,password:f.password,options:{data:{name:f.name}}});
if(error) throw error;
const uid = data.user.id;
await supa.from('teachers').insert({id:uid,school_id:f.schoolCode,name:f.name,email:f.email,role:"teacher"});
await supa.from('schools').update({teacher_count:(sch.teacher_count||1)+1}).eq('id',f.schoolCode);
STATE.profile={id:uid,school_id:f.schoolCode,name:f.name,email:f.email,role:"teacher"};
STATE.school=sch;
await loadClasses();
setState({screen:"home",authBusy:false});
}catch(e){ setState({authBusy:false,authError:e.message||"Registration failed. Try again."}); }
}
async function authLogin(f){
setState({authBusy:true,authError:""});
const {error} = await supa.auth.signInWithPassword({email:f.email,password:f.password});
if(error) setState({authBusy:false,authError:"Incorrect email or password."});
else setState({authBusy:false});
}
async function authLogout(){ await supa.auth.signOut(); }
async function authResetPassword(email){
await supa.auth.resetPasswordForEmail(email,{redirectTo:location.origin});
showToast("Password reset email sent!");
}
// ═══════════════════════════════════════════════
// CLASS CRUD
// ═══════════════════════════════════════════════
async function addClass(name,grade,section){
const id="cls-"+Date.now()+"-"+Math.random().toString(36).slice(2,7);
await supa.from('classes').insert({id,school_id:STATE.school.id,teacher_id:STATE.profile.id,teacher_name:STATE.profile.name,name,grade,section});
await loadClasses(); setState({screen:"home"});
}
async function updateClass(id,name,grade,section){
await supa.from('classes').update({name,grade,section}).eq('id',id);
await loadClasses();
if(STATE.selClass?.id===id) STATE.selClass={...STATE.selClass,name,grade,section};
setState({screen:"classDetail",editClass:null});
}
async function deleteClass(id){
await supa.from('records').delete().eq('class_id',id);
await supa.from('students').delete().eq('class_id',id);
await supa.from('classes').delete().eq('id',id);
await loadClasses(); setState({screen:"home"});
}
// ═══════════════════════════════════════════════
// STUDENT CRUD
// ═══════════════════════════════════════════════
async function addStudentsBulk(names){
const existing = STATE.students.length;
const rows = names.slice(0,MAX_STUDENTS-existing).map((name,i)=>({
id:"stu-"+Date.now()+"-"+Math.random().toString(36).slice(2,7)+i,
school_id:STATE.school.id, class_id:STATE.selClass.id,
name, roll_no:existing+i+1
}));
await supa.from('students').insert(rows);
await loadStudents(STATE.selClass.id);
setState({screen:"classDetail",bulkText:""});
showToast(`✅ Added ${rows.length} students`);
}
async function updateStudent(id,name,rollNo){
await supa.from('students').update({name,roll_no:rollNo}).eq('id',id);
await loadStudents(STATE.selClass.id);
setState({editStudent:null});
}
async function deleteStudent(id){
await supa.from('records').delete().eq('student_id',id);
await supa.from('students').delete().eq('id',id);
await loadStudents(STATE.selClass.id);
await loadRecords(STATE.selClass.id);
showToast("Student deleted.");
}
// ═══════════════════════════════════════════════
// RECORDS
// ═══════════════════════════════════════════════
async function saveRecord(words,errs){
const w = Math.max(0,words-errs);
const acc = words>0?Math.round(((words-errs)/words)*100):0;
const L = LANGS[STATE.lang];
const rec = {
id:"rec-"+Date.now()+"-"+Math.random().toString(36).slice(2,7),
school_id:STATE.school.id, class_id:STATE.selClass.id,
teacher_id:STATE.profile.id, student_id:STATE.selStudent.id,
student_name:STATE.selStudent.name, roll_no:STATE.selStudent.roll_no,
language:STATE.lang, lang_label:L.label, level:STATE.passage.level,
title:STATE.passage.title,
words_read:words, errors:errs, wcpm:w, accuracy:acc,
iso_date:STATE.assDate, date:fmtDate(STATE.assDate),
time:new Date().toLocaleTimeString("en-IN",{hour:"2-digit",minute:"2-digit"})
};
await supa.from('records').insert(rec);
await loadRecords(STATE.selClass.id);
}
// ═══════════════════════════════════════════════
// EXPORT
// ═══════════════════════════════════════════════
function exportExcel(){
const wb = XLSX.utils.book_new();
const rows = [["Date","Student","Roll","Language","Level","Words","Errors","WCPM","Accuracy","Status"]];
STATE.records.forEach(r=>rows.push([r.date,r.student_name,r.roll_no,r.lang_label,r.level,r.words_read,r.errors,r.wcpm,r.accuracy+"%",wl(r.wcpm)]));
const ws = XLSX.utils.aoa_to_sheet(rows);
XLSX.utils.book_append_sheet(wb,ws,"Records");
const blob = new Blob([XLSX.write(wb,{bookType:"xlsx",type:"array"})],{type:"application/octet-stream"});
const a=document.createElement("a"); a.href=URL.createObjectURL(blob);
a.download=`${STATE.selClass?.name||"class"}-records.xlsx`; a.click();
}
function printReport(){
const cls = STATE.selClass;
const recs = filteredRecords();
const avg = recs.length?Math.round(recs.reduce((a,r)=>a+r.wcpm,0)/recs.length):0;
const html=`<!DOCTYPE html><html><head><meta charset="UTF-8"><title>${cls.name} Report</title>
<style>body{font-family:Arial,sans-serif;padding:24px;color:#111}h1{font-size:20px}h2{font-size:13px;color:#555;margin-top:4px}
table{width:100%;border-collapse:collapse;margin-top:16px;font-size:12px}th{background:#0369a1;color:white;padding:8px;text-align:left}
td{padding:7px 8px;border-bottom:1px solid #eee}tr:nth-child(even){background:#f8fafc}
@media print{@page{size:landscape}}</style></head><body>
<h1>📚 ${cls.name} — Reading Fluency Report</h1>
<h2>${STATE.school?.name} · ${STATE.profile?.name} · Avg WCPM: ${avg}</h2>
<table><tr><th>Date</th><th>Student</th><th>Language</th><th>Level</th><th>WCPM</th><th>Accuracy</th><th>Status</th></tr>
${recs.map(r=>`<tr><td>${r.date}</td><td>${r.student_name}</td><td>${r.lang_label}</td><td>${r.level}</td>
<td><b style="color:${wc(r.wcpm)}">${r.wcpm}</b></td><td>${r.accuracy}%</td><td>${wl(r.wcpm)}</td></tr>`).join("")}
</table><scr'+'ipt>window.onload=()=>window.print()<\/scr'+'ipt></body></html>`;
const w=window.open("","_blank"); if(w){w.document.write(html);w.document.close();}
}
// ═══════════════════════════════════════════════
// DERIVED
// ═══════════════════════════════════════════════
function classStats(recs){
if(!recs?.length) return null;
const avg=Math.round(recs.reduce((a,r)=>a+r.wcpm,0)/recs.length);
return{avg,prof:recs.filter(r=>r.wcpm>=80).length,dev:recs.filter(r=>r.wcpm>=50&&r.wcpm<80).length,supp:recs.filter(r=>r.wcpm<50).length,total:recs.length};
}
function filteredRecords(){
return STATE.records.filter(r=>
(!STATE.rptFrom||r.iso_date>=STATE.rptFrom)&&
(!STATE.rptTo||r.iso_date<=STATE.rptTo)&&
(STATE.rptLang==="all"||r.language===STATE.rptLang)
);
}
function studentRecords(sid){
return STATE.records.filter(r=>r.student_id===sid&&
(!STATE.hFrom||r.iso_date>=STATE.hFrom)&&(!STATE.hTo||r.iso_date<=STATE.hTo)
).sort((a,b)=>a.iso_date.localeCompare(b.iso_date));
}
function dateWise(recs){
const bd={};recs.forEach(r=>{if(!bd[r.iso_date])bd[r.iso_date]=[];bd[r.iso_date].push(r.wcpm);});
return Object.keys(bd).sort().map(d=>({date:d,avg:Math.round(bd[d].reduce((a,b)=>a+b,0)/bd[d].length),count:bd[d].length}));
}
function nextStudent(){
const i=STATE.students.findIndex(s=>s.id===STATE.selStudent?.id);
return STATE.students[i+1]||null;
}
// ═══════════════════════════════════════════════
// TIMER
// ═══════════════════════════════════════════════
function startTimer(){
STATE.started=true; STATE.running=true;
STATE.timerInterval=setInterval(()=>{
STATE.timer--;
if(STATE.timer<=0){
clearInterval(STATE.timerInterval); STATE.running=false;
setState({screen:"results"});
} else render();
},1000);
}
function stopTimer(){
clearInterval(STATE.timerInterval); STATE.running=false;
setState({screen:"results"});
}
function resetAssessment(){
clearInterval(STATE.timerInterval);
Object.assign(STATE,{timer:TIMER_SECS,running:false,started:false,words:0,errs:0,lang:null,passage:null,assDate:today()});
}
// ═══════════════════════════════════════════════
// CSS HELPERS
// ═══════════════════════════════════════════════
const BG="#070d14";
const cs={
card:`background:rgba(255,255,255,0.04);border-radius:14px;padding:16px;border:1px solid rgba(255,255,255,0.07)`,
inp:`width:100%;padding:11px 14px;border-radius:10px;background:rgba(255,255,255,0.06);border:1px solid rgba(255,255,255,0.1);color:#d4e8f0;font-size:14px;outline:none;box-sizing:border-box;color-scheme:dark`,
ghost:`background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.1);border-radius:10px;padding:8px 14px;color:#8aaccc`,
red:`background:rgba(239,68,68,0.08);border:1px solid rgba(239,68,68,0.3);border-radius:10px;padding:8px 14px;color:#ef4444`,
};
function pBtn(c1,c2,disabled=false,full=true){
return `style="${full?"width:100%;":""}padding:13px 18px;border-radius:12px;border:none;background:${disabled?"rgba(255,255,255,0.06)":`linear-gradient(135deg,${c1},${c2})`};color:${disabled?"#445":"#fff"};font-size:14px;font-weight:700;letter-spacing:0.04em;text-transform:uppercase;${disabled?"cursor:default":"cursor:pointer"}"`;
}
// ═══════════════════════════════════════════════
// RENDER
// ═══════════════════════════════════════════════
function render(){
const app=document.getElementById("app");
app.innerHTML="";
if(STATE.toast){
const t=document.createElement("div");
t.style.cssText="position:fixed;bottom:24px;left:50%;transform:translateX(-50%);background:#0f2a1a;border:1px solid #16a34a55;border-radius:24px;padding:10px 24px;font-size:13px;color:#4ade80;z-index:5000;white-space:nowrap;box-shadow:0 8px 32px rgba(0,0,0,0.7)";
t.textContent=STATE.toast; app.appendChild(t);
}
if(STATE.confirm){
const {title,sub,fn}=STATE.confirm;
const d=document.createElement("div");
d.style.cssText="position:fixed;inset:0;background:rgba(0,0,0,0.85);display:flex;align-items:center;justify-content:center;z-index:4000;padding:20px";
d.innerHTML=`<div style="${cs.card};max-width:320px;width:100%;text-align:center;padding:28px">
<div style="font-size:34px;margin-bottom:10px">⚠️</div>
<div style="font-size:15px;font-weight:700;color:#d4e8f0;margin-bottom:6px">${title}</div>
${sub?`<div style="font-size:13px;color:#4a6a8a;margin-bottom:12px">${sub}</div>`:""}
<div style="display:flex;gap:10px;margin-top:16px">
<button id="cfmYes" ${pBtn("#b91c1c","#ef4444")}>Delete</button>
<button id="cfmNo" style="${cs.ghost};flex:1;padding:13px;text-align:center">Cancel</button>
</div></div>`;
app.appendChild(d);
document.getElementById("cfmYes").onclick=()=>{fn();setState({confirm:null});};
document.getElementById("cfmNo").onclick=()=>setState({confirm:null});
}
const wrap=document.createElement("div");
wrap.style.cssText=`min-height:100vh;background:${BG};padding-bottom:40px`;
app.appendChild(wrap);
const con=document.createElement("div");
con.style.cssText="max-width:560px;margin:0 auto;padding:0 16px";
wrap.appendChild(con);
const s=STATE.screen;
if(s==="loading") return renderLoading(con);
if(s==="auth") return renderAuth(con);
renderHeader(con);
if(s==="home") renderHome(con);
else if(s==="newClass"||s==="editClass") renderClassForm(con);
else if(s==="addStudents") renderAddStudents(con);
else if(s==="classDetail") renderClassDetail(con);
else if(s==="setup") renderSetup(con);
else if(s==="reading") renderReading(con);
else if(s==="results") renderResults(con);
else if(s==="history") renderHistory(con);
else if(s==="report") renderReport(con);
else if(s==="admin") renderAdmin(con);
}
function renderLoading(c){
c.innerHTML=`<div style="display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;gap:12px;color:#4a6a8a"><div style="font-size:40px">📖</div><div>Loading…</div></div>`;
}
function renderHeader(c){
const h=document.createElement("div");
h.style.cssText="text-align:center;padding:18px 0 14px";
h.innerHTML=`<div style="font-size:28px">📖</div>
<h1 style="font-size:18px;font-weight:700;letter-spacing:2px;text-transform:uppercase;color:#d4e8f0;margin:4px 0 0">Reading Fluency Tracker</h1>
<p style="font-size:11px;color:#4a6a8a;margin-top:3px">1-MINUTE ORAL READING ASSESSMENT</p>`;
if(STATE.screen!=="home"){
const b=document.createElement("button");
b.style.cssText=`${cs.ghost};margin-top:10px;font-size:12px;padding:5px 14px`;
b.textContent="← Home";
b.onclick=()=>{ resetAssessment(); setState({screen:"home"}); };
h.appendChild(b);
}
c.appendChild(h);
}
// ── AUTH ──────────────────────────────────────
function renderAuth(c){
const tabs=["login","newSchool","joinSchool"];
const tabLabels={"login":"Sign In","newSchool":"New School","joinSchool":"Join School"};
c.innerHTML=`<div style="padding:28px 0 16px;text-align:center"><div style="font-size:44px;margin-bottom:8px">📖</div>
<h1 style="font-size:22px;font-weight:700;color:#d4e8f0;letter-spacing:1px">Reading Fluency Tracker</h1>
<p style="font-size:12px;color:#4a6a8a;margin-top:4px">Telugu · Hindi · English · Multi-School Edition</p></div>
<div style="display:flex;background:rgba(255,255,255,0.04);border-radius:12px;padding:4px;margin-bottom:20px" id="authTabs"></div>
<div style="${cs.card}" id="authForm"></div>`;
const tabBar=c.querySelector("#authTabs");
tabs.forEach(t=>{
const btn=document.createElement("button");
btn.style.cssText=`flex:1;padding:9px 4px;border-radius:10px;border:none;font-size:12px;font-weight:600;background:${STATE.authTab===t?"rgba(3,105,161,0.4)":"transparent"};color:${STATE.authTab===t?"#38bdf8":"#6a8aaa"};transition:all .2s`;
btn.textContent=tabLabels[t];
btn.onclick=()=>setState({authTab:t,authError:""});
tabBar.appendChild(btn);
});
const form=c.querySelector("#authForm");
const f=STATE.authForm;
const set=(k,v)=>{ STATE.authForm[k]=v; };
const err=STATE.authError;
let html="";
if(STATE.authTab==="login"){
html=`<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;text-transform:uppercase;color:#4a6a8a;margin-bottom:7px">EMAIL</div>
<input id="femail" type="email" value="${f.email||""}" placeholder="teacher@school.com" style="${cs.inp}"></div>
<div style="margin-bottom:4px"><div style="font-size:10px;letter-spacing:.12em;text-transform:uppercase;color:#4a6a8a;margin-bottom:7px">PASSWORD</div>
<input id="fpass" type="password" value="${f.password||""}" placeholder="••••••" style="${cs.inp}"></div>
<div style="text-align:right;margin-bottom:14px"><button id="forgotBtn" style="background:none;border:none;color:#38bdf8;font-size:12px;cursor:pointer">Forgot password?</button></div>`;
} else if(STATE.authTab==="newSchool"){
html=`<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;text-transform:uppercase;color:#4a6a8a;margin-bottom:7px">SCHOOL NAME *</div>
<input id="fschool" value="${f.schoolName||""}" placeholder="ZP High School, Nandyal" style="${cs.inp}"></div>
<div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:12px">
<div><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">DISTRICT</div>
<input id="fdistrict" value="${f.district||""}" placeholder="Kurnool" style="${cs.inp}"></div>
<div><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">STATE</div><input id="fstate" value="${f.state||""}" placeholder="Andhra Pradesh" style="${cs.inp}"></div></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">YOUR NAME *</div><input id="fname" value="${f.name||""}" placeholder="Priya Sharma" style="${cs.inp}"></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">EMAIL *</div><input id="femail" type="email" value="${f.email||""}" placeholder="principal@school.com" style="${cs.inp}"></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">PASSWORD *</div><input id="fpass" type="password" value="${f.password||""}" placeholder="Min 6 characters" style="${cs.inp}"></div>`;
} else {
html=`<div style="background:rgba(56,189,248,0.08);border:1px solid #38bdf844;border-radius:10px;padding:12px;margin-bottom:14px;font-size:12px;color:#7dd3fc">
📌 Ask your school principal for the <strong>School Code</strong> to join.</div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">SCHOOL CODE *</div><input id="fcode" value="${f.schoolCode||""}" placeholder="sch-12345-abc" style="${cs.inp}"></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">YOUR NAME *</div><input id="fname" value="${f.name||""}" placeholder="Ravi Kumar" style="${cs.inp}"></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">EMAIL *</div><input id="femail" type="email" value="${f.email||""}" placeholder="teacher@gmail.com" style="${cs.inp}"></div>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">PASSWORD *</div><input id="fpass" type="password" value="${f.password||""}" placeholder="Min 6 characters" style="${cs.inp}"></div>`;
}
if(err) html+=`<div style="color:#ef4444;font-size:13px;margin-bottom:12px;background:rgba(239,68,68,0.1);border-radius:8px;padding:8px 12px">${err}</div>`;
html+=`<button id="authSubmit" ${pBtn("#0369a1","#38bdf8",STATE.authBusy)}>${STATE.authBusy?"Please wait…":STATE.authTab==="login"?"Sign In":STATE.authTab==="newSchool"?"Register School":"Join School"}</button>`;
form.innerHTML=html;
form.querySelectorAll("input").forEach(inp=>{
inp.addEventListener("input",e=>{
const keyMap={femail:"email",fpass:"password",fname:"name",fschool:"schoolName",fdistrict:"district",fstate:"state",fcode:"schoolCode"};
set(keyMap[inp.id]||inp.id, e.target.value);
});
});
const sub=form.querySelector("#authSubmit");
if(sub) sub.onclick=()=>{
const ef=STATE.authForm;
if(STATE.authTab==="login") authLogin(ef);
else if(STATE.authTab==="newSchool") authRegisterSchool(ef);
else authRegisterTeacher(ef);
};
const fg=form.querySelector("#forgotBtn");
if(fg) fg.onclick=()=>{ if(STATE.authForm.email) authResetPassword(STATE.authForm.email); else showToast("Enter your email first"); };
}
// ── HOME ──────────────────────────────────────
function renderHome(c){
const {classes,profile,school,online}=STATE;
if(!online){ const o=document.createElement("div");
o.style.cssText="background:#2a1500;border:1px solid #f59e0b55;border-radius:8px;padding:6px 14px;font-size:12px;color:#f59e0b;margin-bottom:10px;text-align:center"; o.textContent="⚠ You are offline"; c.appendChild(o); }
const info=document.createElement("div");
info.style.cssText=`${cs.card};margin-bottom:14px;text-align:center`;
info.innerHTML=`<div style="font-size:14px;font-weight:600">👤 ${profile?.name}</div>
<div style="font-size:12px;color:#4a6a8a;margin-top:2px">${school?.name}</div>
<div style="margin-top:10px;padding:8px 14px;background:rgba(56,189,248,0.06);border:1px solid rgba(56,189,248,0.15);border-radius:8px;font-size:12px;color:#4a6a8a">
School Code: <strong style="color:#38bdf8;user-select:all">${school?.id}</strong>
<div style="font-size:10px;margin-top:2px">Share with teachers to join your school</div></div>
<div style="display:flex;gap:8px;justify-content:center;margin-top:10px;flex-wrap:wrap">
${profile?.role==="admin"?`<button id="adminBtn" style="${cs.ghost};font-size:12px;padding:5px 12px;color:#f59e0b;border-color:#f59e0b44">🔑 Admin</button>`:""}
<button id="downloadAppBtn" style="${cs.ghost};font-size:12px;padding:5px 12px;color:#22c55e;border-color:#22c55e44">⬇ Download App</button>
<button id="logoutBtn" style="${cs.ghost};font-size:12px;padding:5px 12px;color:#ef4444;border-color:#ef444444">Logout</button></div>`;
c.appendChild(info);
const bar=document.createElement("div");
bar.style.cssText="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px";
bar.innerHTML=`<span style="font-size:10px;color:#4a6a8a;letter-spacing:.1em;text-transform:uppercase">My Classes (${classes.length})</span>`;
const nb=document.createElement("button");
nb.style.cssText=`${cs.ghost};color:#38bdf8;font-size:12px`; nb.textContent="+ New Class";
nb.onclick=()=>setState({screen:"newClass",editClass:null});
bar.appendChild(nb); c.appendChild(bar);
if(!classes.length){
const e=document.createElement("div");
e.style.cssText=`${cs.card};text-align:center;padding:32px;color:#2a4a6a`;
e.textContent="No classes yet. Create your first class."; c.appendChild(e);
}
classes.forEach(cls=>{
const card=document.createElement("div");
card.style.cssText=`${cs.card};margin-bottom:10px`;
card.innerHTML=`<div style="display:flex;justify-content:space-between;align-items:flex-start">
<div style="cursor:pointer;flex:1" class="openClass"><div style="font-size:17px;font-weight:700">${cls.name}</div>
<div style="font-size:12px;color:#4a6a8a;margin-top:2px">${cls.grade?`Grade ${cls.grade}`:""}${cls.grade&&cls.section?" · ":""}${cls.section?`Section ${cls.section}`:""}</div></div>
<div style="display:flex;gap:6px">
<button class="openClassBtn" style="${cs.ghost};font-size:12px;padding:6px 12px">Open</button>
<button class="delClassBtn" style="${cs.red};font-size:12px;padding:6px 10px">🗑</button></div></div>`;
card.querySelector(".openClass").onclick=card.querySelector(".openClassBtn").onclick=async()=>{
STATE.selClass=cls; await loadStudents(cls.id); await loadRecords(cls.id);
setState({screen:"classDetail"});
};
card.querySelector(".delClassBtn").onclick=()=>setState({confirm:{title:`Delete "${cls.name}"?`,sub:"All students and records will be deleted.",fn:()=>deleteClass(cls.id)}});
c.appendChild(card);
});
if(document.getElementById("adminBtn")) document.getElementById("adminBtn").onclick=()=>setState({screen:"admin"});
if(document.getElementById("logoutBtn")) document.getElementById("logoutBtn").onclick=authLogout;
if(document.getElementById("downloadAppBtn")) document.getElementById("downloadAppBtn").onclick=async()=>{ showToast("📥 Downloading…"); try{ const res=await fetch(location.href); const html=await res.text(); const blob=new Blob([html],{type:"text/html"}); const url=URL.createObjectURL(blob); const a=document.createElement("a"); a.href=url; a.download="reading-fluency-tracker.html"; a.click(); setTimeout(()=>URL.revokeObjectURL(url),1000); }catch(e){ showToast("❌ Download failed"); } };
}
// ── CLASS FORM ────────────────────────────────
function renderClassForm(c){
const isEdit=STATE.screen==="editClass";
const ec=STATE.editClass;
const d=document.createElement("div"); d.style.cssText=`${cs.card}`;
d.innerHTML=`<h2 style="font-size:15px;font-weight:700;margin-bottom:16px">${isEdit?"EDIT CLASS":"NEW CLASS"}</h2>
${["Class Name *","Grade","Section"].map((l,i)=>`<div style="margin-bottom:12px">
<div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px;text-transform:uppercase">${l}</div>
<input id="cf${i}" value="${isEdit?(i===0?ec.name:i===1?ec.grade||"":ec.section||""):(i===0?STATE.authForm.className||"":"")}" placeholder="${["e.g. Class 3A","e.g. 3","e.g. A"][i]}" style="${cs.inp}"></div>`).join("")}
<div style="display:flex;gap:10px;margin-top:4px">
<button id="cfSave" ${pBtn("#0369a1","#38bdf8")}>${isEdit?"Save Changes":"Create Class"}</button>
<button id="cfCancel" style="${cs.ghost};padding:14px">Cancel</button></div>`;
c.appendChild(d);
document.getElementById("cfCancel").onclick=()=>setState({screen:isEdit?"classDetail":"home"});
document.getElementById("cfSave").onclick=()=>{
const name=document.getElementById("cf0").value.trim();
const grade=document.getElementById("cf1").value.trim();
const section=document.getElementById("cf2").value.trim();
if(!name) return;
if(isEdit) updateClass(ec.id,name,grade,section);
else addClass(name,grade,section);
};
}
// ── CLASS DETAIL ──────────────────────────────
function renderClassDetail(c){
const cls=STATE.selClass;
const stats=classStats(STATE.records);
c.innerHTML=`<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;flex-wrap:wrap;gap:8px">
<div><div style="font-size:18px;font-weight:700">${cls.name}</div>
<div style="font-size:12px;color:#4a6a8a">${cls.grade?`Grade ${cls.grade}`:""}${cls.grade&&cls.section?" · ":""}${cls.section?`Section ${cls.section}`:""} · ${STATE.students.length}/${MAX_STUDENTS} students</div></div>
<div style="display:flex;gap:6px">
<button id="rptBtn" style="${cs.ghost};font-size:12px;padding:7px 12px">📊 Report</button>
<button id="editClsBtn" style="${cs.ghost};font-size:12px;padding:7px 12px">✏️ Edit</button>
<button id="addStuBtn" style="${cs.ghost};font-size:12px;padding:7px 12px;color:#22c55e;border-color:#22c55e44">+ Students</button></div></div>`;
if(stats){
const statBar=document.createElement("div");
statBar.style.cssText="display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-bottom:14px";
[["Proficient",stats.prof,"#22c55e"],["Developing",stats.dev,"#f59e0b"],["Need Support",stats.supp,"#ef4444"]].forEach(([l,n,col])=>{
statBar.innerHTML+=`<div style="background:${col}12;border-radius:10px;padding:10px 6px;text-align:center"><div style="font-size:22px;font-weight:700;color:${col}">${n}</div><div style="font-size:9px;color:${col}">${l}</div></div>`;
}); c.appendChild(statBar);
}
if(!STATE.students.length){
const e=document.createElement("div");
e.style.cssText=`${cs.card};text-align:center;color:#2a4a6a;padding:24px`;
e.textContent="No students yet. Tap '+ Students' to add."; c.appendChild(e);
}
STATE.students.forEach(s=>{
const sRecs=STATE.records.filter(r=>r.student_id===s.id).sort((a,b)=>b.iso_date.localeCompare(a.iso_date));
const lat=sRecs[0];
const card=document.createElement("div");
card.style.cssText=`${cs.card};margin-bottom:8px`;
card.innerHTML=`<div style="display:flex;align-items:center;gap:10px">
<div style="width:30px;height:30px;border-radius:50%;background:rgba(255,255,255,0.07);display:flex;align-items:center;justify-content:center;font-size:11px;color:#4a6a8a;flex-shrink:0">${s.roll_no}</div>
<div style="flex:1;min-width:0"><div style="font-size:14px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${s.name}</div>
<div style="font-size:11px;color:#4a6a8a">${lat?`${lat.lang_label} · ${lat.wcpm} WCPM · ${lat.date}`:"Not yet assessed"}</div></div>
${lat?`<div style="width:8px;height:8px;border-radius:50%;background:${wc(lat.wcpm)};flex-shrink:0"></div>`:""}
<div style="display:flex;gap:5px;flex-shrink:0">
<button class="assBtn" data-id="${s.id}" style="${cs.ghost};padding:5px 9px;font-size:13px">▶</button>
<button class="hisBtn" data-id="${s.id}" style="${cs.ghost};padding:5px 9px;font-size:13px">📋</button>
<button class="edtBtn" data-id="${s.id}" style="${cs.ghost};padding:5px 9px;font-size:13px">✏️</button>
<button class="delBtn" data-id="${s.id}" style="${cs.red};padding:5px 9px;font-size:13px">✕</button></div></div>`;
c.appendChild(card);
});
c.querySelectorAll(".assBtn").forEach(btn=>btn.onclick=()=>{
STATE.selStudent=STATE.students.find(s=>s.id===btn.dataset.id);
resetAssessment(); setState({screen:"setup"});
});
c.querySelectorAll(".hisBtn").forEach(btn=>btn.onclick=()=>{
STATE.selStudent=STATE.students.find(s=>s.id===btn.dataset.id);
STATE.hFrom=""; STATE.hTo=""; setState({screen:"history"});
});
c.querySelectorAll(".edtBtn").forEach(btn=>btn.onclick=()=>{
STATE.editStudent=STATE.students.find(s=>s.id===btn.dataset.id);
const esDiv=document.createElement("div");
esDiv.style.cssText="position:fixed;inset:0;background:rgba(0,0,0,0.85);display:flex;align-items:center;justify-content:center;z-index:4000;padding:20px";
esDiv.innerHTML=`<div style="${cs.card};max-width:340px;width:100%;padding:24px">
<h3 style="margin-bottom:14px;font-size:15px">Edit Student</h3>
<div style="margin-bottom:12px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">NAME</div><input id="esName" value="${STATE.editStudent.name}" style="${cs.inp}"></div>
<div style="margin-bottom:16px"><div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px">ROLL NO</div><input id="esRoll" value="${STATE.editStudent.roll_no}" type="number" style="${cs.inp}"></div>
<div style="display:flex;gap:10px">
<button id="esSave" ${pBtn("#15803d","#22c55e")}>Save</button>
<button id="esCancel" style="${cs.ghost};flex:1;padding:13px;text-align:center">Cancel</button></div></div>`;
document.getElementById("app").appendChild(esDiv);
document.getElementById("esCancel").onclick=()=>esDiv.remove();
document.getElementById("esSave").onclick=()=>{ esDiv.remove();
updateStudent(STATE.editStudent.id,document.getElementById("esName").value.trim(),parseInt(document.getElementById("esRoll").value)||STATE.editStudent.roll_no); };
});
c.querySelectorAll(".delBtn").forEach(btn=>btn.onclick=()=>{
const s=STATE.students.find(s=>s.id===btn.dataset.id);
setState({confirm:{title:`Delete "${s.name}"?`,sub:"Records will also be deleted.",fn:()=>deleteStudent(s.id)}});
});
const rptBtn=document.getElementById("rptBtn"); if(rptBtn) rptBtn.onclick=()=>setState({screen:"report",rptFrom:"",rptTo:"",rptLang:"all"});
const editClsBtn=document.getElementById("editClsBtn"); if(editClsBtn) editClsBtn.onclick=()=>setState({screen:"editClass",editClass:cls});
const addStuBtn=document.getElementById("addStuBtn"); if(addStuBtn) addStuBtn.onclick=()=>setState({screen:"addStudents",bulkText:""});
}
// ── ADD STUDENTS ──────────────────────────────
function renderAddStudents(c){
const slots=MAX_STUDENTS-STATE.students.length;
const d=document.createElement("div"); d.style.cssText=cs.card;
d.innerHTML=`<h2 style="font-size:15px;font-weight:700;margin-bottom:6px">ADD STUDENTS</h2>
<p style="font-size:12px;color:#4a6a8a;margin-bottom:10px">${slots} slots remaining. One name per line.</p>
<textarea id="bulkTA" rows="10" placeholder="Aarav Kumar&#10;Diya Sharma&#10;Rohan Patel&#10;..." style="${cs.inp};resize:vertical;line-height:1.7">${STATE.bulkText}</textarea>
<div style="font-size:11px;color:#3a5a7a;margin:6px 0 10px" id="bulkCount">0 names entered</div>
<div style="display:flex;gap:10px">
<button id="bulkAdd" ${pBtn("#15803d","#22c55e",STATE.busyBulk)}>${STATE.busyBulk?"Adding…":"Add Students"}</button>
<button id="bulkSkip" style="${cs.ghost};padding:14px">Skip</button></div>`;
c.appendChild(d);
const ta=document.getElementById("bulkTA");
const cnt=document.getElementById("bulkCount");
ta.addEventListener("input",e=>{ STATE.bulkText=e.target.value;
cnt.textContent=e.target.value.split("\n").filter(l=>l.trim()).length+" names entered"; });
document.getElementById("bulkSkip").onclick=()=>setState({screen:"classDetail"});
document.getElementById("bulkAdd").onclick=()=>{
const names=STATE.bulkText.trim().split("\n").map(l=>{const m=l.match(/^\d+[.)\s]+(.+)$/);return(m?m[1]:l).trim();}).filter(Boolean);
if(!names.length) return;
addStudentsBulk(names);
};
}
// ── SETUP ─────────────────────────────────────
function renderSetup(c){
const s=STATE.selStudent;
c.innerHTML=`<div style="${cs.card};margin-bottom:12px;text-align:center">
<div style="font-size:16px;font-weight:700">${s.name}</div>
<div style="font-size:12px;color:#4a6a8a">Roll #${s.roll_no} · ${STATE.selClass?.name}</div></div>
<div style="${cs.card};margin-bottom:12px">
<div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:7px;text-transform:uppercase">📅 Assessment Date</div>
<input type="date" id="assDateInp" value="${STATE.assDate}" max="${today()}" style="${cs.inp}">
${STATE.assDate!==today()?`<div style="font-size:11px;color:#f59e0b;margin-top:6px">⚠ Back-dated: ${fmtDate(STATE.assDate)}</div>`:""}
</div>
<div style="margin-bottom:12px">
<div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:8px;text-transform:uppercase">Select Language</div>
<div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px">
${Object.entries(LANGS).map(([k,v])=>`<button data-lang="${k}" class="langBtn" style="padding:12px 6px;border-radius:12px;border:${STATE.lang===k?`2px solid ${v.color}`:"1px solid rgba(255,255,255,0.08)"};background:${STATE.lang===k?v.color+"18":"rgba(255,255,255,0.04)"};color:${STATE.lang===k?v.color:"#7a9ab0"}">
<div style="font-size:15px;font-weight:700" class="${v.cls}">${v.label}</div><div style="font-size:10px;color:#4a6a8a;margin-top:2px">${v.tag}</div></button>`).join("")}</div></div>
${STATE.lang?`<div style="margin-bottom:14px">
<div style="font-size:10px;letter-spacing:.12em;color:#4a6a8a;margin-bottom:8px;text-transform:uppercase">Select Level</div>
${LANGS[STATE.lang].passages.map((p,i)=>`<button data-idx="${i}" class="lvlBtn" style="display:block;width:100%;margin-bottom:8px;padding:12px 14px;border-radius:10px;border:${JSON.stringify(STATE.passage)===JSON.stringify(p)?`2px solid ${LANGS[STATE.lang].color}`:"1px solid rgba(255,255,255,0.08)"};background:${JSON.stringify(STATE.passage)===JSON.stringify(p)?LANGS[STATE.lang].color+"18":"rgba(255,255,255,0.04)"};color:#d4e8f0;text-align:left">
<span style="font-weight:700;color:${LANGS[STATE.lang].color}">${p.level}</span>
<span style="color:#5a7a9a;font-size:12px;margin-left:8px">"${p.title}"</span>
<span style="float:right;font-size:11px;color:#3a5a7a">${p.words} words</span></button>`).join("")}</div>`:""}
<button id="startAssBtn" ${pBtn(STATE.lang?LANGS[STATE.lang].color:"#334",STATE.lang?LANGS[STATE.lang].color+"aa":"#334",!STATE.lang||!STATE.passage)}>Start 1-Minute Assessment →</button>`;
c.querySelector("#assDateInp")?.addEventListener("change",e=>setState({assDate:e.target.value}));
c.querySelectorAll(".langBtn").forEach(b=>b.onclick=()=>setState({lang:b.dataset.lang,passage:null}));
c.querySelectorAll(".lvlBtn").forEach(b=>b.onclick=()=>setState({passage:LANGS[STATE.lang].passages[parseInt(b.dataset.idx)]}));
document.getElementById("startAssBtn").onclick=()=>{ if(STATE.lang&&STATE.passage) setState({screen:"reading"}); };
}
// ── READING ───────────────────────────────────
function renderReading(c){
const L=LANGS[STATE.lang];
const deg=(STATE.timer/60)*360;
const timerColor=STATE.timer<=10?"#ef4444":L.color;
c.innerHTML=`<div style="text-align:center;margin:10px 0">
<div style="display:inline-flex;align-items:center;justify-content:center;width:96px;height:96px;border-radius:50%;background:conic-gradient(${timerColor} ${deg}deg, rgba(255,255,255,0.06) 0deg);box-shadow:${STATE.timer<=10&&STATE.running?`0 0 24px ${L.color}99`:"none"}">
<div style="width:74px;height:74px;border-radius:50%;background:#070d14;display:flex;flex-direction:column;align-items:center;justify-content:center">
<div style="font-size:28px;font-weight:700;color:${timerColor};line-height:1">${STATE.timer}</div>
<div style="font-size:9px;color:#3a5a7a;letter-spacing:1px">SEC</div></div></div>
<div style="font-size:12px;color:#6a8aaa;margin-top:6px">${STATE.selStudent?.name} · ${L.label} · ${STATE.passage?.level}</div>
<div style="font-size:11px;color:#4a6a8a;margin-top:2px">📅 ${fmtDate(STATE.assDate)}</div></div>
<div style="${cs.card};margin-bottom:10px;border:1px solid ${L.color}44">
<div style="font-size:11px;color:${L.color};margin-bottom:6px">${STATE.passage?.title}</div>
<p style="font-size:${STATE.lang==="english"?18:22}px;line-height:2.1;color:#e4f0f8" class="${L.cls}">${STATE.passage?.text}</p></div>
<div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:8px">
${[["WORDS READ",STATE.words,L.color],["ERRORS",STATE.errs,"#ef4444"]].map(([lbl,val,col],i)=>`
<div style="${cs.card};text-align:center">
<div style="font-size:10px;color:#4a6a8a;letter-spacing:.1em;margin-bottom:5px;text-transform:uppercase">${lbl}</div>
<div style="display:flex;align-items:center;justify-content:center;gap:10px">
<button data-type="${i===0?"w":"e"}" data-op="minus" style="width:32px;height:32px;border-radius:50%;border:1px solid rgba(255,255,255,0.1);background:transparent;color:#6a8aaa;font-size:20px;line-height:1">−</button>
<span style="font-size:30px;font-weight:700;color:${col};min-width:38px">${val}</span>
<button data-type="${i===0?"w":"e"}" data-op="plus" style="width:32px;height:32px;border-radius:50%;border:1px solid ${col};background:${col}18;color:${col};font-size:20px;line-height:1">+</button>
</div></div>`).join("")}</div>
<div style="text-align:center;font-size:12px;color:#4a6a8a;margin-bottom:10px">Live WCPM: <strong style="color:${wc(Math.max(0,STATE.words-STATE.errs))};font-size:16px">${Math.max(0,STATE.words-STATE.errs)||"—"}</strong></div>
${!STATE.started
?`<button id="startTimerBtn" ${pBtn(L.color,L.color+"aa")}>▶ Start Timer</button>`
:`<button id="stopTimerBtn" ${pBtn("#b91c1c","#ef4444")}>■ Stop & Save</button>`}`;
c.querySelectorAll("button[data-type]").forEach(btn=>{
btn.onclick=()=>{
const t=btn.dataset.type, op=btn.dataset.op;
if(t==="w") STATE.words=Math.max(0,STATE.words+(op==="plus"?1:-1));
else STATE.errs =Math.max(0,STATE.errs +(op==="plus"?1:-1));
render();
};
});
var _t=(document.getElementById("startTimerBtn")); if(_t) _t.onclick=startTimer;
var _t=(document.getElementById("stopTimerBtn")); if(_t) _t.onclick=stopTimer;
}
// ── RESULTS ───────────────────────────────────
function renderResults(c){
const w=Math.max(0,STATE.words-STATE.errs);
const acc=STATE.words>0?Math.round(((STATE.words-STATE.errs)/STATE.words)*100):0;
const L=LANGS[STATE.lang];
const ns=nextStudent();
c.innerHTML=`<div style="${cs.card};border:2px solid ${wc(w)}44;text-align:center;margin-bottom:12px;padding:24px">
<div style="font-size:10px;color:#4a6a8a;letter-spacing:.15em;margin-bottom:5px">ASSESSMENT COMPLETE</div>
<div style="font-size:13px;color:${L.color};margin-bottom:4px">${STATE.selStudent?.name} · ${L.label} · ${STATE.passage?.level}</div>
<div style="font-size:12px;color:#4a6a8a;margin-bottom:16px">📅 ${fmtDate(STATE.assDate)}</div>
<div style="font-size:56px;font-weight:700;color:${wc(w)};line-height:1">${w}</div>
<div style="font-size:11px;color:#4a6a8a;letter-spacing:.15em;margin-bottom:8px">WCPM</div>
<span style="font-size:11px;padding:2px 10px;border-radius:20px;background:${wc(w)}22;color:${wc(w)};font-weight:700;border:1px solid ${wc(w)}44">${wl(w)}</span>
<div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-top:16px">
${[["Words Read",STATE.words,"#8aaccc"],["Errors",STATE.errs,"#ef4444"],["Accuracy",acc+"%","#22c55e"]].map(([l,v,col])=>`
<div style="background:rgba(255,255,255,0.04);border-radius:10px;padding:12px">
<div style="font-size:22px;font-weight:700;color:${col}">${v}</div>
<div style="font-size:10px;color:#3a5a7a">${l}</div></div>`).join("")}</div></div>
<div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px">
<button id="saveBtn" ${pBtn("#15803d","#22c55e")}>💾 Save</button>
<button id="discardBtn" style="${cs.ghost};width:100%;padding:13px;text-align:center">✕ Discard</button></div>
${ns?`<button id="nextBtn" ${pBtn("#0369a1","#38bdf8")}>Next: ${ns.name} (Roll #${ns.roll_no}) →</button>`:""}`;
document.getElementById("saveBtn").onclick=async()=>{ await saveRecord(STATE.words,STATE.errs); resetAssessment(); setState({screen:"classDetail"}); };
document.getElementById("discardBtn").onclick=()=>{ resetAssessment(); setState({screen:"classDetail"}); };
var _t=(document.getElementById("nextBtn")); if(_t) _t.onclick=async()=>{ await saveRecord(STATE.words,STATE.errs); STATE.selStudent=ns; resetAssessment(); setState({screen:"setup"}); };
}
// ── HISTORY ───────────────────────────────────
function renderHistory(c){
const s=STATE.selStudent;
const recs=studentRecords(s.id);
const sp=recs.map(r=>r.wcpm);
const first=recs[0],last=recs[recs.length-1];
const change=recs.length>=2?last.wcpm-first.wcpm:null;
c.innerHTML=`<div style="${cs.card};margin-bottom:12px">
<div style="display:flex;justify-content:space-between;align-items:center">
<div><div style="font-size:17px;font-weight:700">${s.name}</div>
<div style="font-size:12px;color:#4a6a8a">Roll #${s.roll_no} · ${STATE.selClass?.name}</div></div>
<button id="assessAgainBtn" style="${cs.ghost};font-size:12px;padding:7px 12px;color:#38bdf8">▶ Assess</button></div>
${sp.length>=2?`<div style="margin-top:12px"><div style="font-size:10px;letter-spacing:.1em;color:#4a6a8a;margin-bottom:4px">WCPM PROGRESS</div>
<div style="display:flex;gap:16px;font-size:12px;margin-top:6px">
<span>Start: <strong style="color:${wc(first.wcpm)}">${first.wcpm}</strong></span>
<span>Latest: <strong style="color:${wc(last.wcpm)}">${last.wcpm}</strong></span>
<span>Change: <strong style="color:${change>=0?"#22c55e":"#ef4444"}">${change>0?"+":""}${change} WCPM</strong></span></div></div>`:""}
</div>
<div style="${cs.card};margin-bottom:10px">
<div style="font-size:10px;letter-spacing:.1em;color:#4a6a8a;margin-bottom:8px;text-transform:uppercase">📅 Filter by Date</div>
<div style="display:grid;grid-template-columns:1fr 1fr;gap:8px">
<div><div style="font-size:10px;color:#4a6a8a;margin-bottom:3px">FROM</div><input type="date" id="hFrom" value="${STATE.hFrom}" style="${cs.inp};padding:8px 10px;font-size:13px"></div>
<div><div style="font-size:10px;color:#4a6a8a;margin-bottom:3px">TO</div><input type="date" id="hTo" value="${STATE.hTo}" style="${cs.inp};padding:8px 10px;font-size:13px"></div></div>
${STATE.hFrom||STATE.hTo?`<button id="hClear" style="${cs.ghost};margin-top:8px;font-size:12px;padding:5px 12px">✕ Clear</button>`:""}
</div>
${!recs.length?`<div style="${cs.card};text-align:center;color:#2a4a6a;padding:24px">No records${STATE.hFrom||STATE.hTo?" in this date range":""} yet.</div>`
:[...recs].reverse().map(r=>`<div style="${cs.card};margin-bottom:8px;border:1px solid ${wc(r.wcpm)}44">
<div style="display:flex;justify-content:space-between;align-items:center">
<div style="display:flex;align-items:baseline;gap:8px">
<span style="font-size:26px;font-weight:700;color:${wc(r.wcpm)}">${r.wcpm}</span>
<span style="font-size:11px;color:#4a6a8a">wcpm</span>
<span style="font-size:11px;padding:2px 10px;border-radius:20px;background:${wc(r.wcpm)}22;color:${wc(r.wcpm)};font-weight:700">${wl(r.wcpm)}</span></div>
<div style="text-align:right"><div style="font-size:13px;font-weight:600;color:#7ab8e8">📅 ${r.date}</div>
<div style="font-size:11px;color:#4a6a8a">${r.time}</div></div></div>
<div style="font-size:12px;color:#5a7a9a;margin-top:6px">${r.lang_label} · ${r.level} · ${r.accuracy}% acc · ${r.errors} err · ${r.words_read} words</div></div>`).join("")}`;
document.getElementById("assessAgainBtn").onclick=()=>{ resetAssessment(); setState({screen:"setup"}); };
document.getElementById("hFrom")?.addEventListener("change",e=>setState({hFrom:e.target.value}));
document.getElementById("hTo")?.addEventListener("change",e=>setState({hTo:e.target.value}));
var _t=(document.getElementById("hClear")); if(_t) _t.onclick=()=>setState({hFrom:"",hTo:""});
}
// ── REPORT ────────────────────────────────────
function renderReport(c){
const cls=STATE.selClass;
const recs=filteredRecords();
const stats=classStats(recs);
const dw=dateWise(recs);
const maxAvg=dw.length?Math.max(...dw.map(d=>d.avg)):1;
c.innerHTML=`<div style="${cs.card};margin-bottom:12px">
<div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:8px;margin-bottom:12px">
<div><div style="font-size:18px;font-weight:700">${cls.name} — Report</div>
<div style="font-size:12px;color:#4a6a8a">${STATE.school?.name} · ${STATE.profile?.name}</div></div>
<div style="display:flex;gap:6px">
<button id="xlsxBtn" style="${cs.ghost};font-size:12px;padding:7px 12px">📥 Excel</button>
<button id="printBtn" style="${cs.ghost};font-size:12px;padding:7px 12px">🖨 Print</button></div></div>
<div style="font-size:10px;letter-spacing:.1em;color:#4a6a8a;margin-bottom:8px;text-transform:uppercase">📅 Filter</div>
<div style="display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:8px">
<div><div style="font-size:10px;color:#4a6a8a;margin-bottom:3px">FROM</div><input type="date" id="rFrom" value="${STATE.rptFrom}" style="${cs.inp};padding:8px 10px;font-size:13px"></div>
<div><div style="font-size:10px;color:#4a6a8a;margin-bottom:3px">TO</div><input type="date" id="rTo" value="${STATE.rptTo}" style="${cs.inp};padding:8px 10px;font-size:13px"></div></div>
<div style="display:flex;gap:6px;margin-bottom:8px">
${[["all","All"],["telugu","తె"],["hindi","हि"],["english","EN"]].map(([k,l])=>`<button data-lang="${k}" class="langFBtn" style="flex:1;padding:7px 4px;border-radius:8px;border:${STATE.rptLang===k?"2px solid #38bdf8":"1px solid rgba(255,255,255,0.1)"};background:${STATE.rptLang===k?"rgba(56,189,248,0.15)":"rgba(255,255,255,0.04)"};color:${STATE.rptLang===k?"#38bdf8":"#5a7a9a"};font-size:13px">${l}</button>`).join("")}
</div>
${STATE.rptFrom||STATE.rptTo||STATE.rptLang!=="all"?`<button id="rClear" style="${cs.ghost};font-size:12px;padding:5px 12px;margin-bottom:8px">✕ Clear Filters</button>`:""}
${stats?`<div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-top:12px">
${[["Sessions",stats.total,"#8aaccc"],["Avg WCPM",stats.avg,wc(stats.avg)],["Proficient",stats.prof,"#22c55e"],["Need Help",stats.supp,"#ef4444"]].map(([l,v,col])=>`
<div style="background:rgba(255,255,255,0.04);border-radius:10px;padding:10px;text-align:center">
<div style="font-size:20px;font-weight:700;color:${col}">${v}</div>
<div style="font-size:9px;color:#3a5a7a">${l}</div></div>`).join("")}</div>`
:`<div style="font-size:13px;color:#3a5a7a;text-align:center;padding:12px">No records match this filter.</div>`}</div>
${dw.length?`<div style="${cs.card};margin-bottom:12px">
<div style="font-size:10px;letter-spacing:.1em;color:#4a6a8a;margin-bottom:10px;text-transform:uppercase">📊 Avg WCPM by Date</div>
<div style="overflow-x:auto"><div style="display:flex;align-items:flex-end;gap:8px;min-width:${dw.length*52}px;padding-bottom:4px">
${dw.map(d=>`<div style="display:flex;flex-direction:column;align-items:center;flex:0 0 44px">
<div style="font-size:10px;font-weight:700;color:${wc(d.avg)};margin-bottom:2px">${d.avg}</div>
<div style="width:28px;height:${Math.max(6,Math.round((d.avg/maxAvg)*68))}px;background:${wc(d.avg)};border-radius:4px 4px 0 0"></div>
<div style="font-size:8px;color:#4a6a8a;margin-top:3px;text-align:center;line-height:1.3">${fmtDate(d.date).slice(0,5)}<br><span style="color:#2a4a6a">${d.count}s</span></div></div>`).join("")}
</div></div></div>`:""}
<div style="font-size:10px;letter-spacing:.1em;color:#4a6a8a;margin-bottom:8px;text-transform:uppercase">Student Progress</div>
${STATE.students.map(s=>{
const sr=recs.filter(r=>r.student_id===s.id).sort((a,b)=>a.iso_date.localeCompare(b.iso_date));
const lat=sr[sr.length-1]; const chg=sr.length>=2?sr[sr.length-1].wcpm-sr[0].wcpm:null;
return`<div style="${cs.card};margin-bottom:7px">
<div style="display:flex;align-items:center;gap:10px">
<div style="width:28px;height:28px;border-radius:50%;background:rgba(255,255,255,0.07);display:flex;align-items:center;justify-content:center;font-size:11px;color:#4a6a8a;flex-shrink:0">${s.roll_no}</div>
<div style="flex:1;min-width:0"><div style="font-size:13px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${s.name}</div>
<div style="font-size:11px;color:#3a5a7a">${sr.length} session${sr.length!==1?"s":""}</div></div>
${lat?`<div style="text-align:center;min-width:40px"><div style="font-size:18px;font-weight:700;color:${wc(lat.wcpm)}">${lat.wcpm}</div><div style="font-size:9px;color:#3a5a7a">WCPM</div></div>`:`<div style="font-size:11px;color:#2a4a6a">—</div>`}
</div>
${chg!==null?`<div style="font-size:11px;color:#4a6a8a;margin-top:5px;padding-left:38px">Change: <strong style="color:${chg>=0?"#22c55e":"#ef4444"}">${chg>0?"+":""}${chg} WCPM</strong></div>`:""}
</div>`;
}).join("")}`;
document.getElementById("rFrom")?.addEventListener("change",e=>setState({rptFrom:e.target.value}));
document.getElementById("rTo")?.addEventListener("change",e=>setState({rptTo:e.target.value}));
var _t=(document.getElementById("rClear")); if(_t) _t.onclick=()=>setState({rptFrom:"",rptTo:"",rptLang:"all"});
c.querySelectorAll(".langFBtn").forEach(b=>b.onclick=()=>setState({rptLang:b.dataset.lang}));
var _t=(document.getElementById("xlsxBtn")); if(_t) _t.onclick=exportExcel;
var _t=(document.getElementById("printBtn")); if(_t) _t.onclick=printReport;
}
// ── ADMIN ─────────────────────────────────────
function renderAdmin(c){
c.innerHTML=`<div style="${cs.card};margin-bottom:14px">
<div style="font-size:18px;font-weight:700">🔑 School Admin</div>
<div style="font-size:12px;color:#4a6a8a;margin-top:2px">${STATE.school?.name}</div>
<div style="margin-top:10px;padding:10px;background:rgba(56,189,248,0.08);border-radius:8px">
<div style="font-size:11px;color:#4a6a8a;margin-bottom:3px">SCHOOL CODE — share with teachers</div>
<div style="font-size:14px;font-weight:700;color:#38bdf8;word-break:break-all;user-select:all">${STATE.school?.id}</div></div></div>
<div id="teacherList"></div>`;
supa.from('teachers').select('*').eq('school_id',STATE.school?.id).then(({data})=>{
const tl=document.getElementById("teacherList");
if(!tl) return;
if(!data?.length){ tl.innerHTML=`<div style="${cs.card};text-align:center;color:#2a4a6a;padding:24px">No teachers yet.</div>`; return; }
data.forEach(t=>{
const card=document.createElement("div");
card.style.cssText=`${cs.card};margin-bottom:8px;display:flex;align-items:center;gap:10px`;
card.innerHTML=`<div style="flex:1"><div style="font-size:14px;font-weight:600">${t.name}${t.id===STATE.profile?.id?` <span style="font-size:10px;color:#38bdf8">(you)</span>`:""}</div>
<div style="font-size:11px;color:#4a6a8a">${t.email} · ${t.role}</div></div>
${t.id!==STATE.profile?.id?`<button data-email="${t.email}" data-tid="${t.id}" class="resetPwBtn" style="${cs.ghost};font-size:11px;padding:5px 10px">🔑 Reset PW</button>
<button data-tid="${t.id}" data-name="${t.name}" class="removeTeacherBtn" style="${cs.red};font-size:11px;padding:5px 10px">Remove</button>`:""}`;
tl.appendChild(card);
});
document.querySelectorAll(".resetPwBtn").forEach(btn=>btn.onclick=async()=>{ await authResetPassword(btn.dataset.email); });
document.querySelectorAll(".removeTeacherBtn").forEach(btn=>btn.onclick=()=>{
setState({confirm:{title:`Remove ${btn.dataset.name}?`,sub:"Their classes and data will remain.",fn:async()=>{ await supa.from('teachers').delete().eq('id',btn.dataset.tid); render(); }}});
});
});
}
// ═══════════════════════════════════════════════
// ONLINE / OFFLINE
// ═══════════════════════════════════════════════
window.addEventListener("online",()=>setState({online:true}));
window.addEventListener("offline",()=>setState({online:false}));
// ═══════════════════════════════════════════════
// BOOT
// ═══════════════════════════════════════════════
render();
supa.auth.getSession().then(({data:{session}})=>{
if(!session) setState({screen:"auth"});
});
</script>
</body>
</html>
