<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no"/>
<meta name="theme-color" content="#0a0f1a"/>
<meta name="apple-mobile-web-app-capable" content="yes"/>
<meta name="apple-mobile-web-app-title" content="Readwell"/>
<title>Readwell · Reading Fluency Tracker</title>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Noto+Sans+Telugu:wght@400;600&family=Noto+Sans+Devanagari:wght@400;600&display=swap" rel="stylesheet"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
*{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#0a0f1a;--surface:#111827;--surface2:#1a2235;--border:#1e2d45;--border2:#263550;
  --text:#e2eaf6;--text2:#7a95b8;--text3:#3d5a7a;
  --accent:#3b8beb;--accent2:#1a5fba;--green:#22c55e;--amber:#f59e0b;--red:#ef4444;
  --radius:14px;--radius-sm:10px;
}
html,body{height:100%;background:var(--bg);color:var(--text);font-family:'Inter',system-ui,sans-serif;-webkit-font-smoothing:antialiased;overscroll-behavior:none}
button{-webkit-tap-highlight-color:transparent;font-family:inherit;cursor:pointer;border:none;outline:none}
input,textarea{font-family:inherit;outline:none}
::-webkit-scrollbar{width:3px;height:3px}
::-webkit-scrollbar-thumb{background:var(--border2);border-radius:3px}
.telugu{font-family:'Noto Sans Telugu',sans-serif}
.hindi{font-family:'Noto Sans Devanagari',sans-serif}
.card{background:var(--surface);border-radius:var(--radius);border:1px solid var(--border);padding:16px}
.card2{background:var(--surface2);border-radius:var(--radius-sm);border:1px solid var(--border2);padding:12px}
.inp{width:100%;padding:11px 14px;border-radius:var(--radius-sm);background:rgba(255,255,255,0.05);border:1px solid var(--border2);color:var(--text);font-size:14px;color-scheme:dark}
.inp:focus{border-color:var(--accent)}
.btn-p{width:100%;padding:13px;border-radius:var(--radius-sm);background:linear-gradient(135deg,var(--accent),var(--accent2));color:#fff;font-size:14px;font-weight:700;letter-spacing:.04em;text-transform:uppercase}
.btn-p:disabled{background:var(--border2);color:var(--text3)}
.btn-g{background:rgba(255,255,255,0.04);border:1px solid var(--border2);border-radius:var(--radius-sm);padding:8px 14px;color:var(--text2);font-size:13px}
.btn-d{background:rgba(239,68,68,0.08);border:1px solid rgba(239,68,68,0.25);border-radius:var(--radius-sm);padding:8px 12px;color:var(--red);font-size:13px}
.lbl{font-size:10px;letter-spacing:.12em;text-transform:uppercase;color:var(--text3);margin-bottom:6px}
.pill{display:inline-flex;align-items:center;padding:3px 10px;border-radius:20px;font-size:11px;font-weight:600}
.page{max-width:560px;margin:0 auto;padding:0 16px 48px}
.toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%);background:#0d2218;border:1px solid rgba(34,197,94,.3);border-radius:24px;padding:10px 24px;font-size:13px;color:#4ade80;z-index:9000;white-space:nowrap}
.overlay{position:fixed;inset:0;background:rgba(0,0,0,.85);display:flex;align-items:center;justify-content:center;z-index:8000;padding:20px}
.stat-row{display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-bottom:12px}
.stat-box{background:var(--surface2);border-radius:var(--radius-sm);border:1px solid var(--border);padding:12px 8px;text-align:center}
.pbar{height:4px;background:var(--border);border-radius:2px;overflow:hidden;margin-top:6px}
.pbar-f{height:100%;border-radius:2px}
.sec-hd{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
.sec-hd span{font-size:10px;letter-spacing:.1em;text-transform:uppercase;color:var(--text3)}
.class-card{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius);padding:14px;margin-bottom:10px}
.class-card.locked{opacity:.5;pointer-events:none}
.ring-wrap{display:inline-flex;align-items:center;justify-content:center;border-radius:50%}
.ring-in{border-radius:50%;background:var(--bg);display:flex;flex-direction:column;align-items:center;justify-content:center}
</style>
</head>
<body>
<div id="app"></div>
<script>
const SUPA_URL="https://bkoofkedsbeylquxgdiu.supabase.co";
const SUPA_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImJrb29ma2Vkc2JleWxxdXhnZGl1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzczNDI2MTgsImV4cCI6MjA5MjkxODYxOH0.ZMB0Cj9jsHnf93D1AT6lQmLBL7rU9Mko0KJhAhHHMmE";
const supa=supabase.createClient(SUPA_URL,SUPA_KEY,{auth:{persistSession:true,storageKey:"rw-v3",storage:window.localStorage,autoRefreshToken:true,detectSessionInUrl:false}});

// Keep-alive: ping DB every 5 days so free tier never pauses
(async()=>{const k="rw-ping",l=parseInt(localStorage.getItem(k)||"0");if(Date.now()-l>432000000){await supa.from('schools').select('id').limit(1).catch(()=>{});localStorage.setItem(k,Date.now());}})();

const MAX_T=30,MAX_S=60,TIMER=60;
const LANGS={
  telugu:{label:"తెలుగు",tag:"TE",color:"#f59e0b",cls:"telugu",passages:[
    {level:"Beginner",title:"అమ్మ",words:18,text:"అమ్మ నాకు అన్నం పెడుతుంది. నేను బడికి వెళ్తాను. బడిలో చదువుతాను. అమ్మ నన్ను ప్రేమిస్తుంది. నేను అమ్మను ప్రేమిస్తాను. మేము కలిసి ఉంటాము."},
    {level:"Intermediate",title:"మన గ్రామం",words:38,text:"మన గ్రామం చాలా అందంగా ఉంటుంది. చుట్టూ పచ్చని చెట్లు ఉంటాయి. నదిలో స్వచ్ఛమైన నీళ్ళు ప్రవహిస్తాయి. రైతులు పొలాల్లో కష్టపడి పని చేస్తారు. పిల్లలు ఆడుకుంటూ సంతోషంగా ఉంటారు. సాయంత్రం పూట పక్షులు గూళ్ళకు తిరిగి వస్తాయి."},
    {level:"Advanced",title:"జ్ఞానం",words:42,text:"చదువు మనకు జ్ఞానాన్ని ఇస్తుంది. మంచి పుస్తకాలు చదవడం వలన మన మనసు విశాలమవుతుంది. విద్య ద్వారా మనం జీవితంలో ముందుకు సాగవచ్చు. తల్లిదండ్రులు పిల్లల చదువు కోసం కష్టపడతారు. మంచి ఉపాధ్యాయులు విద్యార్థులను సరైన దారిలో నడిపిస్తారు."}
  ]},
  hindi:{label:"हिन्दी",tag:"HI",color:"#22c55e",cls:"hindi",passages:[
    {level:"Beginner",title:"मेरा घर",words:32,text:"मेरा घर बहुत सुंदर है। घर में माँ, पिताजी और मैं रहते हैं। माँ खाना बनाती है। पिताजी काम पर जाते हैं। मैं स्कूल जाता हूँ। हम सब मिलकर खुश रहते हैं।"},
    {level:"Intermediate",title:"वर्षा ऋतु",words:38,text:"वर्षा ऋतु में काले बादल आकाश में छा जाते हैं। बिजली चमकती है और बादल गरजते हैं। मेंढक टर्र-टर्र करने लगते हैं। किसानों के खेत हरे-भरे हो जाते हैं। बच्चे बारिश में खेलकर बहुत खुश होते हैं।"},
    {level:"Advanced",title:"परिश्रम का महत्व",words:42,text:"परिश्रम सफलता की कुंजी है। जो व्यक्ति मेहनत करता है, वह जीवन में अवश्य आगे बढ़ता है। महान वैज्ञानिक और विद्वान अपनी कड़ी मेहनत से ही प्रसिद्ध हुए। विद्यार्थियों को चाहिए कि वे अपनी पढ़ाई में पूरा ध्यान लगाएँ।"}
  ]},
  english:{label:"English",tag:"EN",color:"#3b82f6",cls:"",passages:[
    {level:"Beginner",title:"My Family",words:36,text:"I have a big family. My mother cooks food for us. My father goes to work every day. My sister and I go to school. We play together in the evening. We love our family very much."},
    {level:"Intermediate",title:"The Forest",words:48,text:"The forest is home to many animals and birds. Tall trees provide shade and fresh air. Deer roam freely through the green meadows. Monkeys swing from branch to branch high above. The sound of a flowing stream fills the air. Every creature depends on the forest to survive."},
    {level:"Advanced",title:"The Importance of Education",words:50,text:"Education is the foundation of a prosperous society. It empowers individuals to think critically and make informed decisions. Through learning, we discover new ideas and broaden our understanding of the world. Teachers play an essential role in shaping young minds. Every child deserves access to quality education regardless of background."}
  ]}
};
const wc=w=>w>=80?"#22c55e":w>=50?"#f59e0b":"#ef4444";
const wl=w=>w>=80?"Proficient":w>=50?"Developing":"Needs Support";
const today=()=>new Date().toISOString().slice(0,10);
const fmtDate=iso=>{if(!iso)return"";const[y,m,d]=iso.split("-");return`${d}/${m}/${y}`;};

let S={
  screen:"loading",user:null,profile:null,school:null,
  allClasses:[],classes:[],students:[],records:[],
  selClass:null,selStudent:null,
  lang:null,passage:null,assDate:today(),
  timer:TIMER,running:false,started:false,words:0,errs:0,timerInterval:null,
  toast:null,toastTimer:null,confirm:null,
  editClass:null,editStudent:null,
  rptFrom:"",rptTo:"",rptLang:"all",hFrom:"",hTo:"",bulkText:"",
  authTab:"login",authForm:{},authError:"",authBusy:false,online:navigator.onLine
};
const set=p=>{Object.assign(S,p);render();};
const showToast=m=>{clearTimeout(S.toastTimer);set({toast:m,toastTimer:setTimeout(()=>set({toast:null}),3000)});};

async function loadProfile(uid){
  const{data}=await supa.from('teachers').select('*').eq('id',uid).single();
  if(!data)return null;
  S.profile=data;
  const{data:sch}=await supa.from('schools').select('*').eq('id',data.school_id).single();
  S.school=sch;return data;
}
async function loadAllClasses(){
  if(!S.profile)return;
  const{data}=await supa.from('classes').select('*').eq('school_id',S.school.id).order('created_at');
  S.allClasses=data||[];S.classes=(data||[]).filter(c=>c.teacher_id===S.profile.id);
}
async function loadStudents(cid){const{data}=await supa.from('students').select('*').eq('class_id',cid).order('roll_no');S.students=data||[];}
async function loadRecords(cid){const{data}=await supa.from('records').select('*').eq('class_id',cid).order('iso_date',{ascending:false});S.records=data||[];}

supa.auth.onAuthStateChange(async(ev,sess)=>{
  if(sess?.user){S.user=sess.user;const p=await loadProfile(sess.user.id);if(p){await loadAllClasses();set({screen:"home"});}else set({screen:"auth"});}
  else{S.user=null;S.profile=null;S.school=null;set({screen:"auth"});}
});

async function doRegSchool(f){
  set({authBusy:true,authError:""});
  try{
    const{data,error}=await supa.auth.signUp({email:f.email,password:f.password,options:{data:{name:f.name}}});
    if(error)throw error;
    const uid=data.user.id,sid="sch-"+Date.now()+"-"+Math.random().toString(36).slice(2,7);
    await supa.from('schools').insert({id:sid,name:f.schoolName,district:f.district||"",state:f.state||"",admin_uid:uid,teacher_count:1});
    await supa.from('teachers').insert({id:uid,school_id:sid,name:f.name,email:f.email,role:"admin"});
    S.profile={id:uid,school_id:sid,name:f.name,email:f.email,role:"admin"};S.school={id:sid,name:f.schoolName};
    await loadAllClasses();set({screen:"home",authBusy:false});
  }catch(e){set({authBusy:false,authError:e.message||"Registration failed."});}
}
async function doRegTeacher(f){
  set({authBusy:true,authError:""});
  try{
    const{data:sch}=await supa.from('schools').select('*').eq('id',f.schoolCode).single();
    if(!sch){set({authBusy:false,authError:"School code not found."});return;}
    if((sch.teacher_count||0)>=MAX_T){set({authBusy:false,authError:"School reached 30 teacher limit."});return;}
    const{data,error}=await supa.auth.signUp({email:f.email,password:f.password,options:{data:{name:f.name}}});
    if(error)throw error;
    const uid=data.user.id;
    await supa.from('teachers').insert({id:uid,school_id:f.schoolCode,name:f.name,email:f.email,role:"teacher"});
    await supa.from('schools').update({teacher_count:(sch.teacher_count||1)+1}).eq('id',f.schoolCode);
    S.profile={id:uid,school_id:f.schoolCode,name:f.name,email:f.email,role:"teacher"};S.school=sch;
    await loadAllClasses();set({screen:"home",authBusy:false});
  }catch(e){set({authBusy:false,authError:e.message||"Registration failed."});}
}
async function doLogin(f){
  set({authBusy:true,authError:""});
  const{error}=await supa.auth.signInWithPassword({email:f.email,password:f.password});
  if(error)set({authBusy:false,authError:"Incorrect email or password."});else set({authBusy:false});
}
async function doLogout(){await supa.auth.signOut();}
async function doResetPw(email){await supa.auth.resetPasswordForEmail(email,{redirectTo:location.origin});showToast("Reset email sent!");}

async function addClass(n,g,s){
  const id="cls-"+Date.now()+"-"+Math.random().toString(36).slice(2,7);
  await supa.from('classes').insert({id,school_id:S.school.id,teacher_id:S.profile.id,teacher_name:S.profile.name,name:n,grade:g,section:s});
  await loadAllClasses();set({screen:"home"});
}
async function updateClass(id,n,g,s){
  await supa.from('classes').update({name:n,grade:g,section:s}).eq('id',id);
  await loadAllClasses();if(S.selClass?.id===id)S.selClass={...S.selClass,name:n,grade:g,section:s};
  set({screen:"classDetail",editClass:null});
}
async function deleteClass(id){
  await supa.from('records').delete().eq('class_id',id);
  await supa.from('students').delete().eq('class_id',id);
  await supa.from('classes').delete().eq('id',id);
  await loadAllClasses();set({screen:"home"});
}
async function addStudentsBulk(names){
  const ex=S.students.length;
  const rows=names.slice(0,MAX_S-ex).map((name,i)=>({id:"stu-"+Date.now()+"-"+i,school_id:S.school.id,class_id:S.selClass.id,name,roll_no:ex+i+1}));
  await supa.from('students').insert(rows);await loadStudents(S.selClass.id);
  set({screen:"classDetail",bulkText:""});showToast(`✅ Added ${rows.length} students`);
}
async function updateStudent(id,name,rn){await supa.from('students').update({name,roll_no:rn}).eq('id',id);await loadStudents(S.selClass.id);set({editStudent:null});}
async function deleteStudent(id){
  await supa.from('records').delete().eq('student_id',id);await supa.from('students').delete().eq('id',id);
  await loadStudents(S.selClass.id);await loadRecords(S.selClass.id);showToast("Student removed.");
}
async function saveRecord(words,errs){
  const w=Math.max(0,words-errs),acc=words>0?Math.round(((words-errs)/words)*100):0,L=LANGS[S.lang];
  await supa.from('records').insert({id:"rec-"+Date.now()+"-"+Math.random().toString(36).slice(2,7),
    school_id:S.school.id,class_id:S.selClass.id,teacher_id:S.profile.id,
    student_id:S.selStudent.id,student_name:S.selStudent.name,roll_no:S.selStudent.roll_no,
    language:S.lang,lang_label:L.label,level:S.passage.level,title:S.passage.title,
    words_read:words,errors:errs,wcpm:w,accuracy:acc,iso_date:S.assDate,date:fmtDate(S.assDate),
    time:new Date().toLocaleTimeString("en-IN",{hour:"2-digit",minute:"2-digit"})});
  await loadRecords(S.selClass.id);
}
function exportExcel(){
  const wb=XLSX.utils.book_new(),rows=[["Date","Student","Roll","Language","Level","Words","Errors","WCPM","Accuracy","Status"]];
  S.records.forEach(r=>rows.push([r.date,r.student_name,r.roll_no,r.lang_label,r.level,r.words_read,r.errors,r.wcpm,r.accuracy+"%",wl(r.wcpm)]));
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(rows),"Records");
  const a=document.createElement("a");a.href=URL.createObjectURL(new Blob([XLSX.write(wb,{bookType:"xlsx",type:"array"})],{type:"application/octet-stream"}));
  a.download=`${S.selClass?.name||"class"}-records.xlsx`;a.click();
}
function printReport(){
  const cls=S.selClass,recs=filtRecs(),avg=recs.length?Math.round(recs.reduce((a,r)=>a+r.wcpm,0)/recs.length):0;
  const html=`<!DOCTYPE html><html><head><meta charset="UTF-8"><title>${cls.name}</title>
  <style>body{font-family:Arial;padding:24px}table{width:100%;border-collapse:collapse;margin-top:16px;font-size:12px}
  th{background:#1a5fba;color:#fff;padding:8px;text-align:left}td{padding:7px 8px;border-bottom:1px solid #eee}
  @media print{@page{size:landscape}}</style></head><body>
  <h2>📚 ${cls.name} — Reading Fluency Report</h2><p>${S.school?.name} · Avg WCPM: ${avg}</p>
  <table><tr><th>Date</th><th>Student</th><th>Language</th><th>Level</th><th>WCPM</th><th>Accuracy</th><th>Status</th></tr>
  ${recs.map(r=>`<tr><td>${r.date}</td><td>${r.student_name}</td><td>${r.lang_label}</td><td>${r.level}</td><td><b style="color:${wc(r.wcpm)}">${r.wcpm}</b></td><td>${r.accuracy}%</td><td>${wl(r.wcpm)}</td></tr>`).join("")}
  </table><scr`+`ipt>window.onload=()=>window.print()<\/scr`+`ipt></body></html>`;
  const w=window.open("","_blank");if(w){w.document.write(html);w.document.close();}
}
function clsStats(recs){
  if(!recs?.length)return null;
  const avg=Math.round(recs.reduce((a,r)=>a+r.wcpm,0)/recs.length);
  return{avg,prof:recs.filter(r=>r.wcpm>=80).length,dev:recs.filter(r=>r.wcpm>=50&&r.wcpm<80).length,supp:recs.filter(r=>r.wcpm<50).length,total:recs.length};
}
function filtRecs(){return S.records.filter(r=>(!S.rptFrom||r.iso_date>=S.rptFrom)&&(!S.rptTo||r.iso_date<=S.rptTo)&&(S.rptLang==="all"||r.language===S.rptLang));}
function stuRecs(sid){return S.records.filter(r=>r.student_id===sid&&(!S.hFrom||r.iso_date>=S.hFrom)&&(!S.hTo||r.iso_date<=S.hTo)).sort((a,b)=>a.iso_date.localeCompare(b.iso_date));}
function dateWise(recs){const bd={};recs.forEach(r=>{if(!bd[r.iso_date])bd[r.iso_date]=[];bd[r.iso_date].push(r.wcpm);});return Object.keys(bd).sort().map(d=>({date:d,avg:Math.round(bd[d].reduce((a,b)=>a+b,0)/bd[d].length),count:bd[d].length}));}
function nextStu(){const i=S.students.findIndex(s=>s.id===S.selStudent?.id);return S.students[i+1]||null;}
function startTimer(){S.started=true;S.running=true;S.timerInterval=setInterval(()=>{S.timer--;if(S.timer<=0){clearInterval(S.timerInterval);S.running=false;set({screen:"results"});}else render();},1000);}
function stopTimer(){clearInterval(S.timerInterval);S.running=false;set({screen:"results"});}
function resetAss(){clearInterval(S.timerInterval);Object.assign(S,{timer:TIMER,running:false,started:false,words:0,errs:0,lang:null,passage:null,assDate:today()});}

function render(){
  const app=document.getElementById("app");app.innerHTML="";
  if(S.toast){const t=document.createElement("div");t.className="toast";t.textContent=S.toast;app.appendChild(t);}
  if(S.confirm){
    const{title,sub,fn}=S.confirm,d=document.createElement("div");d.className="overlay";
    d.innerHTML=`<div class="card" style="max-width:300px;width:100%;text-align:center;padding:24px">
      <div style="font-size:32px;margin-bottom:10px">⚠️</div>
      <div style="font-size:15px;font-weight:700;margin-bottom:6px">${title}</div>
      ${sub?`<div style="font-size:13px;color:var(--text2);margin-bottom:12px">${sub}</div>`:""}
      <div style="display:flex;gap:10px;margin-top:14px">
        <button id="cfY" class="btn-d" style="flex:1;padding:12px">Delete</button>
        <button id="cfN" class="btn-g" style="flex:1;padding:12px">Cancel</button></div></div>`;
    app.appendChild(d);
    document.getElementById("cfY").onclick=()=>{fn();set({confirm:null});};
    document.getElementById("cfN").onclick=()=>set({confirm:null});
  }
  const wrap=document.createElement("div");wrap.style.minHeight="100vh";app.appendChild(wrap);
  const con=document.createElement("div");con.className="page";wrap.appendChild(con);
  const sc=S.screen;
  if(sc==="loading")return con.innerHTML=`<div style="display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;gap:12px;color:var(--text3)"><div style="font-size:40px">📖</div><div>Loading…</div></div>`;
  if(sc==="auth")return renderAuth(con);
  renderBar(con);
  if(sc==="home")renderHome(con);
  else if(sc==="newClass"||sc==="editClass")renderClassForm(con);
  else if(sc==="addStudents")renderAddStudents(con);
  else if(sc==="classDetail")renderClassDetail(con);
  else if(sc==="setup")renderSetup(con);
  else if(sc==="reading")renderReading(con);
  else if(sc==="results")renderResults(con);
  else if(sc==="history")renderHistory(con);
  else if(sc==="report")renderReport(con);
  else if(sc==="admin")renderAdmin(con);
}

function renderBar(c){
  const h=document.createElement("div");h.style.cssText="text-align:center;padding:20px 0 14px";
  h.innerHTML=`<div style="font-size:26px;margin-bottom:3px">📖</div><h1 style="font-size:17px;font-weight:800;letter-spacing:.15em;text-transform:uppercase">Readwell</h1><p style="font-size:11px;color:var(--text3);margin-top:2px">1-MINUTE ORAL READING ASSESSMENT</p>`;
  if(S.screen!=="home"){const b=document.createElement("button");b.className="btn-g";b.style.cssText="margin-top:10px;font-size:12px;padding:5px 14px";b.textContent="← Home";b.onclick=()=>{resetAss();set({screen:"home"});};h.appendChild(b);}
  c.appendChild(h);
}

function renderAuth(c){
  const tabs=["login","newSchool","joinSchool"],tlbls={login:"Sign In",newSchool:"New School",joinSchool:"Join School"};
  c.innerHTML=`<div style="padding:32px 0 20px;text-align:center"><div style="font-size:48px;margin-bottom:10px">📖</div>
    <h1 style="font-size:24px;font-weight:800">Readwell</h1>
    <p style="font-size:12px;color:var(--text3);margin-top:4px">Telugu · Hindi · English · Multi-School</p></div>
  <div style="display:flex;background:var(--surface2);border-radius:12px;padding:4px;margin-bottom:20px" id="aTabs"></div>
  <div class="card" id="aForm"></div>`;
  const tb=c.querySelector("#aTabs");
  tabs.forEach(t=>{const b=document.createElement("button");b.style.cssText=`flex:1;padding:9px 4px;border-radius:10px;border:none;font-size:12px;font-weight:600;background:${S.authTab===t?"var(--accent)":"transparent"};color:${S.authTab===t?"#fff":"var(--text3)"};transition:all .2s`;b.textContent=tlbls[t];b.onclick=()=>set({authTab:t,authError:""});tb.appendChild(b);});
  const form=c.querySelector("#aForm"),f=S.authForm,sv=(k,v)=>{S.authForm[k]=v;};
  let html="";
  if(S.authTab==="login"){
    html=`<div style="margin-bottom:12px"><div class="lbl">Email</div><input id="fe" type="email" value="${f.email||""}" placeholder="teacher@school.com" class="inp"></div>
    <div style="margin-bottom:4px"><div class="lbl">Password</div><input id="fp" type="password" value="${f.password||""}" placeholder="••••••" class="inp"></div>
    <div style="text-align:right;margin:6px 0 14px"><button id="fgot" style="background:none;border:none;color:var(--accent);font-size:12px;cursor:pointer">Forgot password?</button></div>`;
  }else if(S.authTab==="newSchool"){
    html=`<div style="margin-bottom:12px"><div class="lbl">School Name *</div><input id="fsc" value="${f.schoolName||""}" placeholder="ZP High School, Nandyal" class="inp"></div>
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:12px">
    <div><div class="lbl">District</div><input id="fd" value="${f.district||""}" placeholder="Nizamabad" class="inp"></div>
    <div><div class="lbl">State</div><input id="fst" value="${f.state||""}" placeholder="Telangana" class="inp"></div></div>
    <div style="margin-bottom:12px"><div class="lbl">Your Name *</div><input id="fn" value="${f.name||""}" placeholder="Vinuthna" class="inp"></div>
    <div style="margin-bottom:12px"><div class="lbl">Email *</div><input id="fe" type="email" value="${f.email||""}" placeholder="principal@school.com" class="inp"></div>
    <div style="margin-bottom:12px"><div class="lbl">Password *</div><input id="fp" type="password" value="${f.password||""}" placeholder="Min 6 characters" class="inp"></div>`;
  }else{
    html=`<div style="background:rgba(59,139,235,.08);border:1px solid rgba(59,139,235,.2);border-radius:10px;padding:12px;margin-bottom:14px;font-size:12px;color:#7db8f8">📌 Ask your school principal for the <strong>School Code</strong>.</div>
    <div style="margin-bottom:12px"><div class="lbl">School Code *</div><input id="fcode" value="${f.schoolCode||""}" placeholder="sch-12345-abc" class="inp"></div>
    <div style="margin-bottom:12px"><div class="lbl">Your Name *</div><input id="fn" value="${f.name||""}" placeholder="Ravi Kumar" class="inp"></div>
    <div style="margin-bottom:12px"><div class="lbl">Email *</div><input id="fe" type="email" value="${f.email||""}" placeholder="teacher@gmail.com" class="inp"></div>
    <div style="margin-bottom:12px"><div class="lbl">Password *</div><input id="fp" type="password" value="${f.password||""}" placeholder="Min 6 characters" class="inp"></div>`;
  }
  if(S.authError)html+=`<div style="color:var(--red);font-size:13px;margin-bottom:12px;background:rgba(239,68,68,.1);border-radius:8px;padding:8px 12px">${S.authError}</div>`;
  html+=`<button id="aSub" class="btn-p" ${S.authBusy?"disabled":""}>${S.authBusy?"Please wait…":S.authTab==="login"?"Sign In":S.authTab==="newSchool"?"Register School":"Join School"}</button>`;
  form.innerHTML=html;
  form.querySelectorAll("input").forEach(inp=>{inp.addEventListener("input",e=>{const m={fe:"email",fp:"password",fn:"name",fsc:"schoolName",fd:"district",fst:"state",fcode:"schoolCode"};sv(m[inp.id]||inp.id,e.target.value);});});
  form.querySelector("#aSub").onclick=()=>{const ef=S.authForm;if(S.authTab==="login")doLogin(ef);else if(S.authTab==="newSchool")doRegSchool(ef);else doRegTeacher(ef);};
  const fg=form.querySelector("#fgot");if(fg)fg.onclick=()=>{if(S.authForm.email)doResetPw(S.authForm.email);else showToast("Enter your email first");};
}

function renderHome(c){
  const{profile,school,online,classes,allClasses}=S;
  if(!online){const o=document.createElement("div");o.style.cssText="background:#2a1500;border:1px solid rgba(245,158,11,.3);border-radius:8px;padding:7px 14px;font-size:12px;color:var(--amber);margin-bottom:10px;text-align:center";o.textContent="⚠ You are offline";c.appendChild(o);}

  // Profile card
  const info=document.createElement("div");info.className="card";info.style.marginBottom="14px";
  info.innerHTML=`<div style="display:flex;align-items:center;gap:12px">
    <div style="width:44px;height:44px;border-radius:50%;background:var(--accent);display:flex;align-items:center;justify-content:center;font-size:20px;font-weight:800;flex-shrink:0;color:#fff">${(profile?.name||"?")[0].toUpperCase()}</div>
    <div style="flex:1;min-width:0">
      <div style="font-size:15px;font-weight:700">${profile?.name}</div>
      <div style="font-size:12px;color:var(--text2)">${school?.name}</div>
      <div style="font-size:11px;color:var(--text3);margin-top:1px">${profile?.role==="admin"?"🔑 Admin":"Teacher"} · ${classes.length} class${classes.length!==1?"es":""}</div>
    </div>
    <div style="display:flex;flex-direction:column;gap:6px">
      ${profile?.role==="admin"?`<button id="adminBtn" style="font-size:11px;padding:5px 10px;border-radius:8px;background:rgba(245,158,11,.1);border:1px solid rgba(245,158,11,.25);color:var(--amber)">Admin</button>`:""}
      <button id="logoutBtn" style="font-size:11px;padding:5px 10px;border-radius:8px;background:rgba(239,68,68,.08);border:1px solid rgba(239,68,68,.2);color:var(--red)">Logout</button>
    </div></div>
  <div style="margin-top:12px;padding:8px 12px;background:var(--surface2);border-radius:8px;font-size:12px;color:var(--text3)">
    School Code: <strong style="color:var(--accent);user-select:all">${school?.id}</strong>
    <span style="float:right;font-size:10px">Share with teachers</span></div>`;
  c.appendChild(info);

  // My classes section
  const bar=document.createElement("div");bar.className="sec-hd";
  bar.innerHTML=`<span>My Classes (${classes.length})</span>`;
  const nb=document.createElement("button");nb.className="btn-g";nb.style.cssText="font-size:12px;color:var(--accent);border-color:rgba(59,139,235,.3)";nb.textContent="+ New Class";nb.onclick=()=>set({screen:"newClass",editClass:null});
  bar.appendChild(nb);c.appendChild(bar);

  if(!classes.length){
    const e=document.createElement("div");e.className="card";e.style.cssText="text-align:center;padding:32px;color:var(--text3)";
    e.innerHTML=`<div style="font-size:32px;margin-bottom:8px">📚</div><div>No classes yet.</div><div style="font-size:12px;margin-top:4px">Tap "+ New Class" to get started.</div>`;c.appendChild(e);
  }

  classes.forEach(cls=>{
    const card=document.createElement("div");card.className="class-card";
    card.innerHTML=`<div style="display:flex;justify-content:space-between;align-items:flex-start">
      <div style="flex:1;min-width:0;cursor:pointer" class="cc-open">
        <div style="font-size:16px;font-weight:700">${cls.name}</div>
        <div style="font-size:12px;color:var(--text2);margin-top:2px">${cls.grade?`Grade ${cls.grade}`:""}${cls.grade&&cls.section?" · ":""}${cls.section?`Section ${cls.section}`:""}</div>
      </div>
      <div style="display:flex;gap:6px;margin-left:8px">
        <button class="cc-open btn-g" style="font-size:12px;padding:6px 12px">Open</button>
        <button class="cc-del btn-d" style="font-size:12px;padding:6px 10px">🗑</button>
      </div></div>`;
    card.querySelectorAll(".cc-open").forEach(el=>el.onclick=async()=>{S.selClass=cls;await loadStudents(cls.id);await loadRecords(cls.id);set({screen:"classDetail"});});
    card.querySelector(".cc-del").onclick=e=>{e.stopPropagation();set({confirm:{title:`Delete "${cls.name}"?`,sub:"All students and records will be deleted.",fn:()=>deleteClass(cls.id)}});};
    c.appendChild(card);
  });

  // Other teachers' locked classes
  const others=allClasses.filter(cl=>cl.teacher_id!==profile?.id);
  if(others.length>0){
    const ob=document.createElement("div");ob.className="sec-hd";ob.style.marginTop="18px";
    ob.innerHTML=`<span>Other Classes</span><span style="color:var(--text3);font-size:11px">${others.length} locked</span>`;c.appendChild(ob);
    others.forEach(cls=>{
      const card=document.createElement("div");card.className="class-card locked";
      card.innerHTML=`<div style="display:flex;justify-content:space-between;align-items:center">
        <div><div style="font-size:15px;font-weight:700">${cls.name}</div>
        <div style="font-size:12px;color:var(--text3)">${cls.grade?`Grade ${cls.grade}`:""}${cls.grade&&cls.section?" · ":""}${cls.section?`Section ${cls.section}`:""} · ${cls.teacher_name||"Teacher"}</div></div>
        <div style="font-size:18px">🔒</div></div>`;
      c.appendChild(card);
    });
  }

  document.getElementById("adminBtn")?.addEventListener("click",()=>set({screen:"admin"}));
  document.getElementById("logoutBtn")?.addEventListener("click",doLogout);
}

function renderClassForm(c){
  const isE=S.screen==="editClass",ec=S.editClass;
  const d=document.createElement("div");d.className="card";
  d.innerHTML=`<h2 style="font-size:15px;font-weight:700;margin-bottom:16px">${isE?"EDIT CLASS":"NEW CLASS"}</h2>
  ${["Class Name *","Grade","Section"].map((l,i)=>`<div style="margin-bottom:12px"><div class="lbl">${l}</div>
    <input id="cf${i}" value="${isE?(i===0?ec.name:i===1?ec.grade||"":ec.section||""):(i===0?S.authForm.className||"":"")}"
    placeholder="${["e.g. Class 5A","e.g. 5","e.g. A"][i]}" class="inp"></div>`).join("")}
  <div style="display:flex;gap:10px;margin-top:4px">
    <button id="cfS" class="btn-p">${isE?"Save Changes":"Create Class"}</button>
    <button id="cfC" class="btn-g" style="padding:13px 18px">Cancel</button></div>`;
  c.appendChild(d);
  document.getElementById("cfC").onclick=()=>set({screen:isE?"classDetail":"home"});
  document.getElementById("cfS").onclick=()=>{
    const n=document.getElementById("cf0").value.trim(),g=document.getElementById("cf1").value.trim(),s=document.getElementById("cf2").value.trim();
    if(!n)return;if(isE)updateClass(ec.id,n,g,s);else addClass(n,g,s);
  };
}

function renderClassDetail(c){
  const cls=S.selClass,stats=clsStats(S.records);
  const hd=document.createElement("div");hd.style.cssText="display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:12px;flex-wrap:wrap;gap:8px";
  hd.innerHTML=`<div><div style="font-size:19px;font-weight:800">${cls.name}</div>
    <div style="font-size:12px;color:var(--text2)">${cls.grade?`Grade ${cls.grade}`:""}${cls.grade&&cls.section?" · ":""}${cls.section?`Section ${cls.section}`:""} · ${S.students.length}/${MAX_S} students</div></div>
  <div style="display:flex;gap:6px;flex-wrap:wrap">
    <button id="rptBtn" class="btn-g" style="font-size:12px;padding:7px 12px">📊 Report</button>
    <button id="edtClsBtn" class="btn-g" style="font-size:12px;padding:7px 12px">✏️ Edit</button>
    <button id="addStuBtn" class="btn-g" style="font-size:12px;padding:7px 12px;color:var(--green);border-color:rgba(34,197,94,.25)">+ Students</button>
  </div>`;c.appendChild(hd);

  if(stats){
    const sb=document.createElement("div");sb.className="stat-row";
    [["Proficient",stats.prof,"#22c55e"],["Developing",stats.dev,"#f59e0b"],["Needs Help",stats.supp,"#ef4444"]].forEach(([l,n,col])=>{
      sb.innerHTML+=`<div class="stat-box"><div style="font-size:26px;font-weight:800;color:${col};line-height:1">${n}</div><div style="font-size:9px;letter-spacing:.08em;text-transform:uppercase;color:var(--text3);margin-top:3px">${l}</div></div>`;
    });c.appendChild(sb);
    if(stats.total>0){
      const pct=Math.round((stats.prof/stats.total)*100);
      const pb=document.createElement("div");pb.className="card";pb.style.marginBottom="12px";
      pb.innerHTML=`<div style="display:flex;justify-content:space-between;font-size:12px;margin-bottom:6px">
        <span style="color:var(--text2)">Class Progress</span><span style="font-weight:700;color:${wc(stats.avg)}">${stats.avg} avg WCPM</span></div>
        <div class="pbar"><div class="pbar-f" style="width:${pct}%;background:${wc(stats.avg)}"></div></div>
        <div style="font-size:11px;color:var(--text3);margin-top:5px">${stats.total} assessment${stats.total!==1?"s":""} · ${pct}% proficient</div>`;
      c.appendChild(pb);
    }
  }

  if(!S.students.length){
    const e=document.createElement("div");e.className="card";e.style.cssText="text-align:center;color:var(--text3);padding:28px";
    e.innerHTML=`<div style="font-size:28px;margin-bottom:8px">👥</div>No students yet. Tap '+ Students'.`;c.appendChild(e);
  }

  S.students.forEach(s=>{
    const sR=S.records.filter(r=>r.student_id===s.id).sort((a,b)=>b.iso_date.localeCompare(a.iso_date)),lat=sR[0];
    const card=document.createElement("div");card.className="card";card.style.marginBottom="8px";
    card.innerHTML=`<div style="display:flex;align-items:center;gap:10px">
      <div style="width:32px;height:32px;border-radius:50%;background:var(--surface2);border:1px solid var(--border2);display:flex;align-items:center;justify-content:center;font-size:11px;color:var(--text3);flex-shrink:0;font-weight:700">${s.roll_no}</div>
      <div style="flex:1;min-width:0">
        <div style="font-size:14px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${s.name}</div>
        <div style="font-size:11px;color:var(--text3)">${lat?`${lat.lang_label} · <strong style="color:${wc(lat.wcpm)}">${lat.wcpm} WCPM</strong> · ${lat.date}`:"Not yet assessed"}</div>
      </div>
      ${lat?`<div style="width:8px;height:8px;border-radius:50%;background:${wc(lat.wcpm)};flex-shrink:0"></div>`:""}
      <div style="display:flex;gap:5px;flex-shrink:0">
        <button class="aBtn" data-id="${s.id}" style="background:rgba(59,139,235,.1);border:1px solid rgba(59,139,235,.2);border-radius:8px;padding:6px 9px;font-size:13px;color:var(--accent)">▶</button>
        <button class="hBtn" data-id="${s.id}" style="background:var(--surface2);border:1px solid var(--border2);border-radius:8px;padding:6px 9px;font-size:13px">📋</button>
        <button class="eBtn" data-id="${s.id}" style="background:var(--surface2);border:1px solid var(--border2);border-radius:8px;padding:6px 9px;font-size:13px">✏️</button>
        <button class="dBtn" data-id="${s.id}" style="background:rgba(239,68,68,.08);border:1px solid rgba(239,68,68,.2);border-radius:8px;padding:6px 9px;font-size:13px;color:var(--red)">✕</button>
      </div></div>`;
    c.appendChild(card);
  });

  c.querySelectorAll(".aBtn").forEach(b=>b.onclick=()=>{S.selStudent=S.students.find(s=>s.id===b.dataset.id);resetAss();set({screen:"setup"});});
  c.querySelectorAll(".hBtn").forEach(b=>b.onclick=()=>{S.selStudent=S.students.find(s=>s.id===b.dataset.id);S.hFrom="";S.hTo="";set({screen:"history"});});
  c.querySelectorAll(".eBtn").forEach(b=>b.onclick=()=>{
    S.editStudent=S.students.find(s=>s.id===b.dataset.id);
    const d=document.createElement("div");d.className="overlay";
    d.innerHTML=`<div class="card" style="max-width:320px;width:100%;padding:24px">
      <h3 style="margin-bottom:14px;font-size:15px;font-weight:700">Edit Student</h3>
      <div style="margin-bottom:12px"><div class="lbl">Name</div><input id="esN" value="${S.editStudent.name}" class="inp"></div>
      <div style="margin-bottom:16px"><div class="lbl">Roll No</div><input id="esR" value="${S.editStudent.roll_no}" type="number" class="inp"></div>
      <div style="display:flex;gap:10px">
        <button id="esSv" class="btn-p" style="background:linear-gradient(135deg,#15803d,#22c55e)">Save</button>
        <button id="esCn" class="btn-g" style="flex:1;padding:13px;text-align:center">Cancel</button></div></div>`;
    document.getElementById("app").appendChild(d);
    document.getElementById("esCn").onclick=()=>d.remove();
    document.getElementById("esSv").onclick=()=>{d.remove();updateStudent(S.editStudent.id,document.getElementById("esN").value.trim(),parseInt(document.getElementById("esR").value)||S.editStudent.roll_no);};
  });
  c.querySelectorAll(".dBtn").forEach(b=>b.onclick=()=>{const s=S.students.find(s=>s.id===b.dataset.id);set({confirm:{title:`Delete "${s.name}"?`,sub:"Records will also be deleted.",fn:()=>deleteStudent(s.id)}});});
  document.getElementById("rptBtn")?.addEventListener("click",()=>set({screen:"report",rptFrom:"",rptTo:"",rptLang:"all"}));
  document.getElementById("edtClsBtn")?.addEventListener("click",()=>set({screen:"editClass",editClass:cls}));
  document.getElementById("addStuBtn")?.addEventListener("click",()=>set({screen:"addStudents",bulkText:""}));
}

function renderAddStudents(c){
  const slots=MAX_S-S.students.length;
  const d=document.createElement("div");d.className="card";
  d.innerHTML=`<h2 style="font-size:15px;font-weight:700;margin-bottom:6px">ADD STUDENTS</h2>
    <p style="font-size:12px;color:var(--text2);margin-bottom:10px">${slots} slots remaining · One name per line</p>
    <textarea id="bTA" rows="10" placeholder="Aarav Kumar&#10;Diya Sharma&#10;Rohan Patel&#10;..." class="inp" style="resize:vertical;line-height:1.7">${S.bulkText}</textarea>
    <div style="font-size:11px;color:var(--text3);margin:6px 0 10px" id="bCnt">0 names</div>
    <div style="display:flex;gap:10px"><button id="bAdd" class="btn-p">Add Students</button><button id="bSkip" class="btn-g" style="padding:14px">Skip</button></div>`;
  c.appendChild(d);
  const ta=document.getElementById("bTA"),cnt=document.getElementById("bCnt");
  ta.addEventListener("input",e=>{S.bulkText=e.target.value;cnt.textContent=e.target.value.split("\n").filter(l=>l.trim()).length+" names";});
  document.getElementById("bSkip").onclick=()=>set({screen:"classDetail"});
  document.getElementById("bAdd").onclick=()=>{const names=S.bulkText.trim().split("\n").map(l=>{const m=l.match(/^\d+[.)\s]+(.+)$/);return(m?m[1]:l).trim();}).filter(Boolean);if(!names.length)return;addStudentsBulk(names);};
}

function renderSetup(c){
  const s=S.selStudent;
  c.innerHTML=`<div class="card" style="margin-bottom:12px;text-align:center">
    <div style="font-size:18px;font-weight:800">${s.name}</div>
    <div style="font-size:12px;color:var(--text2)">Roll #${s.roll_no} · ${S.selClass?.name}</div></div>
  <div class="card" style="margin-bottom:12px">
    <div class="lbl">📅 Assessment Date</div>
    <input type="date" id="aDate" value="${S.assDate}" max="${today()}" class="inp">
    ${S.assDate!==today()?`<div style="font-size:11px;color:var(--amber);margin-top:6px">⚠ Back-dated: ${fmtDate(S.assDate)}</div>`:""}
  </div>
  <div style="margin-bottom:12px"><div class="lbl" style="margin-bottom:8px">Select Language</div>
  <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px">
  ${Object.entries(LANGS).map(([k,v])=>`<button data-lang="${k}" class="lBtn" style="padding:12px 6px;border-radius:12px;border:${S.lang===k?`2px solid ${v.color}`:"1px solid var(--border2)"};background:${S.lang===k?v.color+"18":"var(--surface2)"};color:${S.lang===k?v.color:"var(--text3)"}">
    <div style="font-size:15px;font-weight:700" class="${v.cls}">${v.label}</div><div style="font-size:10px;margin-top:2px">${v.tag}</div></button>`).join("")}</div></div>
  ${S.lang?`<div style="margin-bottom:14px"><div class="lbl" style="margin-bottom:8px">Select Level</div>
  ${LANGS[S.lang].passages.map((p,i)=>`<button data-idx="${i}" class="lvBtn" style="display:block;width:100%;margin-bottom:8px;padding:12px 14px;border-radius:10px;text-align:left;border:${JSON.stringify(S.passage)===JSON.stringify(p)?`2px solid ${LANGS[S.lang].color}`:"1px solid var(--border2)"};background:${JSON.stringify(S.passage)===JSON.stringify(p)?LANGS[S.lang].color+"18":"var(--surface2)"};color:var(--text)">
    <span style="font-weight:700;color:${LANGS[S.lang].color}">${p.level}</span>
    <span style="color:var(--text3);font-size:12px;margin-left:8px">"${p.title}"</span>
    <span style="float:right;font-size:11px;color:var(--text3)">${p.words} words</span></button>`).join("")}</div>`:""}
  <button id="startBtn" class="btn-p" ${!S.lang||!S.passage?"disabled":""}
    style="${S.lang&&S.passage?`background:linear-gradient(135deg,${LANGS[S.lang].color},${LANGS[S.lang].color}88)`:""}"
    >Start 1-Minute Assessment →</button>`;
  c.querySelector("#aDate")?.addEventListener("change",e=>set({assDate:e.target.value}));
  c.querySelectorAll(".lBtn").forEach(b=>b.onclick=()=>set({lang:b.dataset.lang,passage:null}));
  c.querySelectorAll(".lvBtn").forEach(b=>b.onclick=()=>set({passage:LANGS[S.lang].passages[parseInt(b.dataset.idx)]}));
  document.getElementById("startBtn").onclick=()=>{if(S.lang&&S.passage)set({screen:"reading"});};
}

function renderReading(c){
  const L=LANGS[S.lang],deg=(S.timer/60)*360,tC=S.timer<=10?"#ef4444":L.color;
  c.innerHTML=`<div style="text-align:center;margin:10px 0">
    <div class="ring-wrap" style="width:100px;height:100px;background:conic-gradient(${tC} ${deg}deg,var(--border) 0deg);box-shadow:${S.timer<=10&&S.running?`0 0 28px ${tC}66`:"none"}">
      <div class="ring-in" style="width:76px;height:76px">
        <div style="font-size:30px;font-weight:800;color:${tC};line-height:1">${S.timer}</div>
        <div style="font-size:9px;color:var(--text3);letter-spacing:1px">SEC</div></div></div>
    <div style="font-size:12px;color:var(--text2);margin-top:8px">${S.selStudent?.name} · ${L.label} · ${S.passage?.level}</div>
    <div style="font-size:11px;color:var(--text3);margin-top:2px">📅 ${fmtDate(S.assDate)}</div></div>
  <div class="card" style="margin-bottom:10px;border-color:${tC}33">
    <div style="font-size:11px;color:${L.color};margin-bottom:6px;font-weight:600">${S.passage?.title}</div>
    <p style="font-size:${S.lang==="english"?18:22}px;line-height:2.2;color:var(--text)" class="${L.cls}">${S.passage?.text}</p></div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px">
  ${[["WORDS READ",S.words,L.color],["ERRORS",S.errs,"#ef4444"]].map(([lbl,val,col],i)=>`
    <div class="card" style="text-align:center;padding:14px"><div class="lbl">${lbl}</div>
      <div style="display:flex;align-items:center;justify-content:center;gap:12px;margin-top:6px">
        <button data-t="${i===0?"w":"e"}" data-o="m" style="width:34px;height:34px;border-radius:50%;background:var(--surface2);border:1px solid var(--border2);color:var(--text2);font-size:22px;line-height:1">−</button>
        <span style="font-size:32px;font-weight:800;color:${col};min-width:40px">${val}</span>
        <button data-t="${i===0?"w":"e"}" data-o="p" style="width:34px;height:34px;border-radius:50%;background:${col}18;border:1px solid ${col}44;color:${col};font-size:22px;line-height:1">+</button>
      </div></div>`).join("")}</div>
  <div style="text-align:center;font-size:12px;color:var(--text3);margin-bottom:10px">Live WCPM: <strong style="color:${wc(Math.max(0,S.words-S.errs))};font-size:18px">${Math.max(0,S.words-S.errs)||"—"}</strong></div>
  ${!S.started?`<button id="stBtn" class="btn-p" style="background:linear-gradient(135deg,${L.color},${L.color}88)">▶ Start Timer</button>`
  :`<button id="spBtn" class="btn-p" style="background:linear-gradient(135deg,#b91c1c,#ef4444)">■ Stop & Save</button>`}`;
  c.querySelectorAll("button[data-t]").forEach(b=>b.onclick=()=>{const t=b.dataset.t,o=b.dataset.o;if(t==="w")S.words=Math.max(0,S.words+(o==="p"?1:-1));else S.errs=Math.max(0,S.errs+(o==="p"?1:-1));render();});
  document.getElementById("stBtn")?.addEventListener("click",startTimer);
  document.getElementById("spBtn")?.addEventListener("click",stopTimer);
}

function renderResults(c){
  const w=Math.max(0,S.words-S.errs),acc=S.words>0?Math.round(((S.words-S.errs)/S.words)*100):0,L=LANGS[S.lang],ns=nextStu();
  c.innerHTML=`<div class="card" style="border:2px solid ${wc(w)}33;text-align:center;margin-bottom:12px;padding:28px">
    <div style="font-size:10px;color:var(--text3);letter-spacing:.15em;margin-bottom:6px">ASSESSMENT COMPLETE</div>
    <div style="font-size:13px;color:${L.color};margin-bottom:4px">${S.selStudent?.name} · ${L.label} · ${S.passage?.level}</div>
    <div style="font-size:12px;color:var(--text3);margin-bottom:18px">📅 ${fmtDate(S.assDate)}</div>
    <div style="font-size:64px;font-weight:800;color:${wc(w)};line-height:1">${w}</div>
    <div style="font-size:11px;color:var(--text3);letter-spacing:.15em;margin-bottom:10px">WCPM</div>
    <span class="pill" style="background:${wc(w)}18;color:${wc(w)};border:1px solid ${wc(w)}33">${wl(w)}</span>
    <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-top:18px">
    ${[["Words",S.words,"#8aaccc"],["Errors",S.errs,"#ef4444"],["Accuracy",acc+"%","#22c55e"]].map(([l,v,col])=>`
      <div class="card2" style="text-align:center"><div style="font-size:22px;font-weight:800;color:${col}">${v}</div><div style="font-size:10px;color:var(--text3);margin-top:2px">${l}</div></div>`).join("")}
    </div></div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px">
    <button id="svBtn" class="btn-p" style="background:linear-gradient(135deg,#15803d,#22c55e)">💾 Save</button>
    <button id="dsBtn" class="btn-g" style="width:100%;padding:13px;text-align:center">✕ Discard</button></div>
  ${ns?`<button id="nxBtn" class="btn-p">Next: ${ns.name} (Roll #${ns.roll_no}) →</button>`:""}`;
  document.getElementById("svBtn").onclick=async()=>{await saveRecord(S.words,S.errs);resetAss();set({screen:"classDetail"});};
  document.getElementById("dsBtn").onclick=()=>{resetAss();set({screen:"classDetail"});};
  document.getElementById("nxBtn")?.addEventListener("click",async()=>{await saveRecord(S.words,S.errs);S.selStudent=ns;resetAss();set({screen:"setup"});});
}

function renderHistory(c){
  const s=S.selStudent,recs=stuRecs(s.id),sp=recs.map(r=>r.wcpm),first=recs[0],last=recs[recs.length-1],chg=recs.length>=2?last.wcpm-first.wcpm:null;
  c.innerHTML=`<div class="card" style="margin-bottom:12px">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div><div style="font-size:17px;font-weight:800">${s.name}</div>
      <div style="font-size:12px;color:var(--text2)">Roll #${s.roll_no} · ${S.selClass?.name}</div></div>
      <button id="aAgain" class="btn-g" style="font-size:12px;padding:7px 12px;color:var(--accent);border-color:rgba(59,139,235,.3)">▶ Assess</button></div>
    ${sp.length>=2?`<div style="margin-top:12px;padding-top:12px;border-top:1px solid var(--border)"><div class="lbl" style="margin-bottom:6px">WCPM Progress</div>
      <div style="display:flex;gap:16px;font-size:13px">
        <span>Start: <strong style="color:${wc(first.wcpm)}">${first.wcpm}</strong></span>
        <span>Latest: <strong style="color:${wc(last.wcpm)}">${last.wcpm}</strong></span>
        <span>Change: <strong style="color:${chg>=0?"#22c55e":"#ef4444"}">${chg>0?"+":""}${chg}</strong></span></div></div>`:""}
  </div>
  <div class="card" style="margin-bottom:10px">
    <div class="lbl" style="margin-bottom:8px">📅 Filter by Date</div>
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px">
      <div><div style="font-size:10px;color:var(--text3);margin-bottom:3px">FROM</div><input type="date" id="hFr" value="${S.hFrom}" class="inp" style="padding:8px 10px;font-size:13px"></div>
      <div><div style="font-size:10px;color:var(--text3);margin-bottom:3px">TO</div><input type="date" id="hTo" value="${S.hTo}" class="inp" style="padding:8px 10px;font-size:13px"></div></div>
    ${S.hFrom||S.hTo?`<button id="hClr" class="btn-g" style="margin-top:8px;font-size:12px;padding:5px 12px">✕ Clear</button>`:""}
  </div>
  ${!recs.length?`<div class="card" style="text-align:center;color:var(--text3);padding:24px">No records${S.hFrom||S.hTo?" in this range":""} yet.</div>`
  :[...recs].reverse().map(r=>`<div class="card" style="margin-bottom:8px;border-color:${wc(r.wcpm)}22">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div style="display:flex;align-items:baseline;gap:8px">
        <span style="font-size:28px;font-weight:800;color:${wc(r.wcpm)}">${r.wcpm}</span>
        <span style="font-size:11px;color:var(--text3)">wcpm</span>
        <span class="pill" style="background:${wc(r.wcpm)}18;color:${wc(r.wcpm)};border:1px solid ${wc(r.wcpm)}22">${wl(r.wcpm)}</span></div>
      <div style="text-align:right"><div style="font-size:13px;font-weight:600;color:var(--accent)">📅 ${r.date}</div>
      <div style="font-size:11px;color:var(--text3)">${r.time}</div></div></div>
    <div style="font-size:12px;color:var(--text3);margin-top:5px">${r.lang_label} · ${r.level} · ${r.accuracy}% acc · ${r.errors} err</div></div>`).join("")}`;
  document.getElementById("aAgain").onclick=()=>{resetAss();set({screen:"setup"});};
  document.getElementById("hFr")?.addEventListener("change",e=>set({hFrom:e.target.value}));
  document.getElementById("hTo")?.addEventListener("change",e=>set({hTo:e.target.value}));
  document.getElementById("hClr")?.addEventListener("click",()=>set({hFrom:"",hTo:""}));
}

function renderReport(c){
  const cls=S.selClass,recs=filtRecs(),stats=clsStats(recs),dw=dateWise(recs),mA=dw.length?Math.max(...dw.map(d=>d.avg)):1;
  c.innerHTML=`<div class="card" style="margin-bottom:12px">
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:8px;margin-bottom:12px">
      <div><div style="font-size:17px;font-weight:800">${cls.name} — Report</div>
      <div style="font-size:12px;color:var(--text2)">${S.school?.name} · ${S.profile?.name}</div></div>
      <div style="display:flex;gap:6px">
        <button id="xlBtn" class="btn-g" style="font-size:12px;padding:7px 12px">📥 Excel</button>
        <button id="prBtn" class="btn-g" style="font-size:12px;padding:7px 12px">🖨 Print</button></div></div>
    <div class="lbl" style="margin-bottom:8px">📅 Filter</div>
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:8px">
      <div><div style="font-size:10px;color:var(--text3);margin-bottom:3px">FROM</div><input type="date" id="rFr" value="${S.rptFrom}" class="inp" style="padding:8px 10px;font-size:13px"></div>
      <div><div style="font-size:10px;color:var(--text3);margin-bottom:3px">TO</div><input type="date" id="rTo" value="${S.rptTo}" class="inp" style="padding:8px 10px;font-size:13px"></div></div>
    <div style="display:flex;gap:6px;margin-bottom:8px">
    ${[["all","All"],["telugu","తె"],["hindi","हि"],["english","EN"]].map(([k,l])=>`<button data-lang="${k}" class="lFBtn" style="flex:1;padding:7px 4px;border-radius:8px;border:${S.rptLang===k?"2px solid var(--accent)":"1px solid var(--border2)"};background:${S.rptLang===k?"rgba(59,139,235,.15)":"var(--surface2)"};color:${S.rptLang===k?"var(--accent)":"var(--text3)"};font-size:13px">${l}</button>`).join("")}</div>
    ${S.rptFrom||S.rptTo||S.rptLang!=="all"?`<button id="rClr" class="btn-g" style="font-size:12px;padding:5px 12px;margin-bottom:8px">✕ Clear</button>`:""}
    ${stats?`<div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-top:10px">
    ${[["Sessions",stats.total,"#8aaccc"],["Avg WCPM",stats.avg,wc(stats.avg)],["Proficient",stats.prof,"#22c55e"],["Need Help",stats.supp,"#ef4444"]].map(([l,v,col])=>`
      <div class="card2" style="text-align:center"><div style="font-size:20px;font-weight:800;color:${col}">${v}</div><div style="font-size:9px;color:var(--text3);margin-top:2px">${l}</div></div>`).join("")}</div>`
    :`<div style="font-size:13px;color:var(--text3);text-align:center;padding:10px">No records match.</div>`}</div>
  ${dw.length?`<div class="card" style="margin-bottom:12px">
    <div class="lbl" style="margin-bottom:10px">📊 Avg WCPM by Date</div>
    <div style="overflow-x:auto"><div style="display:flex;align-items:flex-end;gap:8px;min-width:${dw.length*52}px;padding-bottom:4px">
    ${dw.map(d=>`<div style="display:flex;flex-direction:column;align-items:center;flex:0 0 44px">
      <div style="font-size:10px;font-weight:700;color:${wc(d.avg)};margin-bottom:2px">${d.avg}</div>
      <div style="width:28px;height:${Math.max(6,Math.round((d.avg/mA)*68))}px;background:${wc(d.avg)};border-radius:4px 4px 0 0"></div>
      <div style="font-size:8px;color:var(--text3);margin-top:3px;text-align:center;line-height:1.3">${fmtDate(d.date).slice(0,5)}<br><span>${d.count}s</span></div></div>`).join("")}
    </div></div></div>`:""}
  <div class="lbl" style="margin-bottom:8px">Student Progress</div>
  ${S.students.map(s=>{
    const sr=recs.filter(r=>r.student_id===s.id).sort((a,b)=>a.iso_date.localeCompare(b.iso_date)),lat=sr[sr.length-1],chg=sr.length>=2?sr[sr.length-1].wcpm-sr[0].wcpm:null;
    return`<div class="card" style="margin-bottom:7px"><div style="display:flex;align-items:center;gap:10px">
      <div style="width:28px;height:28px;border-radius:50%;background:var(--surface2);border:1px solid var(--border2);display:flex;align-items:center;justify-content:center;font-size:11px;color:var(--text3);flex-shrink:0;font-weight:700">${s.roll_no}</div>
      <div style="flex:1;min-width:0"><div style="font-size:13px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${s.name}</div>
      <div style="font-size:11px;color:var(--text3)">${sr.length} session${sr.length!==1?"s":""}</div></div>
      ${lat?`<div style="text-align:center;min-width:44px"><div style="font-size:20px;font-weight:800;color:${wc(lat.wcpm)}">${lat.wcpm}</div><div style="font-size:9px;color:var(--text3)">WCPM</div></div>`:`<div style="font-size:11px;color:var(--text3)">—</div>`}</div>
      ${chg!==null?`<div style="font-size:11px;color:var(--text3);margin-top:5px;padding-left:38px">Change: <strong style="color:${chg>=0?"#22c55e":"#ef4444"}">${chg>0?"+":""}${chg} WCPM</strong></div>`:""}
    </div>`;
  }).join("")}`;
  document.getElementById("rFr")?.addEventListener("change",e=>set({rptFrom:e.target.value}));
  document.getElementById("rTo")?.addEventListener("change",e=>set({rptTo:e.target.value}));
  document.getElementById("rClr")?.addEventListener("click",()=>set({rptFrom:"",rptTo:"",rptLang:"all"}));
  c.querySelectorAll(".lFBtn").forEach(b=>b.onclick=()=>set({rptLang:b.dataset.lang}));
  document.getElementById("xlBtn")?.addEventListener("click",exportExcel);
  document.getElementById("prBtn")?.addEventListener("click",printReport);
}

function renderAdmin(c){
  c.innerHTML=`<div class="card" style="margin-bottom:14px">
    <div style="font-size:17px;font-weight:800">🔑 Admin Panel</div>
    <div style="font-size:12px;color:var(--text2);margin-top:2px">${S.school?.name}</div>
    <div style="margin-top:10px;padding:10px 12px;background:var(--surface2);border-radius:8px">
      <div class="lbl">School Code — share with teachers</div>
      <div style="font-size:14px;font-weight:700;color:var(--accent);word-break:break-all;user-select:all;margin-top:4px">${S.school?.id}</div></div></div>
  <div id="tList"></div>`;
  supa.from('teachers').select('*').eq('school_id',S.school?.id).then(({data})=>{
    const tl=document.getElementById("tList");if(!tl)return;
    if(!data?.length){tl.innerHTML=`<div class="card" style="text-align:center;color:var(--text3);padding:24px">No teachers yet.</div>`;return;}
    data.forEach(t=>{
      const card=document.createElement("div");card.className="card";card.style.cssText="margin-bottom:8px;display:flex;align-items:center;gap:10px";
      card.innerHTML=`<div style="flex:1"><div style="font-size:14px;font-weight:600">${t.name}${t.id===S.profile?.id?` <span style="font-size:10px;color:var(--accent)">(you)</span>`:""}</div>
        <div style="font-size:11px;color:var(--text2)">${t.email} · ${t.role}</div></div>
      ${t.id!==S.profile?.id?`
        <button data-email="${t.email}" data-tid="${t.id}" class="rPW btn-g" style="font-size:11px;padding:5px 10px">🔑 Reset PW</button>
        <button data-tid="${t.id}" data-name="${t.name}" class="remT btn-d" style="font-size:11px;padding:5px 10px">Remove</button>`:""}`;
      tl.appendChild(card);
    });
    document.querySelectorAll(".rPW").forEach(b=>b.onclick=async()=>{await doResetPw(b.dataset.email);});
    document.querySelectorAll(".remT").forEach(b=>b.onclick=()=>{set({confirm:{title:`Remove ${b.dataset.name}?`,sub:"Their classes and data remain.",fn:async()=>{await supa.from('teachers').delete().eq('id',b.dataset.tid);render();}}});});
  });
}

window.addEventListener("online",()=>set({online:true}));
window.addEventListener("offline",()=>set({online:false}));
render();
supa.auth.getSession().then(({data:{session}})=>{if(!session)set({screen:"auth"});});
</script>
</body>
</html>
