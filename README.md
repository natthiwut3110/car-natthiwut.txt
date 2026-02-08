<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Blood Exchange Calculator</title>
<style>
:root{--primary:#b91c1c;--primary2:#ef4444;--soft:#fff1f2;--gray:#6b7280}
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Arial;background:linear-gradient(180deg,#fff5f5,#ffe4e6);padding:12px}
.card{background:white;padding:16px;border-radius:18px;max-width:520px;margin:auto;box-shadow:0 14px 34px rgba(185,28,28,.18);border-top:6px solid var(--primary)}
h2{margin:0 0 6px 0;font-size:28px;color:var(--primary)}
label{font-weight:600;font-size:14px;color:#450a0a}
input{width:100%;padding:10px;margin:4px 0 10px;border-radius:10px;border:1px solid #fecaca;font-size:15px}
input:focus{outline:none;border-color:var(--primary);box-shadow:0 0 0 3px rgba(239,68,68,.25)}
.btns{display:flex;gap:6px}
button{flex:1;padding:11px;border:none;border-radius:12px;color:white;font-size:14px;cursor:pointer;transition:.15s}
button:active{transform:scale(.98)}
.calc{background:linear-gradient(135deg,var(--primary),var(--primary2))}
.copy{background:linear-gradient(135deg,#7f1d1d,#991b1b)}
.reset{background:linear-gradient(135deg,#9ca3af,#6b7280)}
.result{margin-top:10px;display:flex;flex-direction:column;gap:3px}
.summary{background:#fff;border:1px solid #fecaca;border-radius:10px;padding:6px 8px;font-size:13px}
.bag{background:#fff9c4;border:1px solid #fde68a;border-radius:12px;padding:6px 8px}
.bag-title{font-weight:700;color:#450a0a;margin-bottom:2px;font-size:18px}
.row{display:flex;gap:6px;align-items:baseline}
.name,.label,.unit{color:#450a0a;font-size:18px;font-weight:700}
.value{color:#b91c1c;font-size:18px;font-weight:800}
.final{margin-top:6px;background:#fff;border:2px solid #b91c1c;border-radius:12px;padding:8px;font-size:14px;color:#450a0a;font-weight:600}
.note{font-size:11px;color:#7f1d1d;margin-top:6px;text-align:center}
textarea{position:absolute;left:-9999px;top:-9999px}
.footer{position:fixed;bottom:4px;left:0;right:0;text-align:center;font-size:13px;color:#7f1d1d;opacity:.9}
</style>
</head>
<body>
<div class="card">
<h2>Blood Exchange</h2>
<label>ปริมาตร whole blood ที่ต้องการ (ml)</label>
<input id="whole" type="number" inputmode="numeric">
<label>ปริมาตร PRC ต่อถุง (ml)</label>
<input id="prc" type="number" inputmode="numeric">
<div class="btns">
<button class="calc" id="calcBtn">คำนวณ</button>
<button id="copyBtn" class="copy" style="display:none">Copy</button>
<button class="reset" id="resetBtn">Reset</button>
</div>
<div class="result" id="output"></div>
<div class="note" id="copyNote" style="display:none">* หากกด Copy ไม่ได้ ให้กดค้างเพื่อคัดลอก</div>
<textarea id="fallback"></textarea>
</div>
<div class="footer">สงวนลิขสิทธิ์ © ธนาคารเลือด โรงพยาบาลสุราษฎร์ธานี</div>
<script>
const TARGET_HCT=45,PRC_HCT=60;let plainTextResult="";
const wholeEl=document.getElementById('whole');
const prcEl=document.getElementById('prc');
const output=document.getElementById('output');
const copyBtn=document.getElementById('copyBtn');
const copyNote=document.getElementById('copyNote');
const fallback=document.getElementById('fallback');

function calculate(){
 const whole=parseFloat(wholeEl.value);
 const prcBagMax=parseFloat(prcEl.value);
 if(!whole||!prcBagMax) return;
 let remainingTotal=whole,bag=1,totalPRC=0,totalFFP=0,html="",text="";
 html+=`<div class='summary'>Whole blood: <b>${whole.toFixed(0)} ml</b></div>`;
 text+=`ปริมาตร whole blood ${whole.toFixed(0)} ml\n`;
 while(remainingTotal>0.5){
  let prcNeeded=remainingTotal*(TARGET_HCT/PRC_HCT);
  let prcUse=Math.min(prcBagMax,prcNeeded);
  let totalFromThisPRC=prcUse*(PRC_HCT/TARGET_HCT);
  let ffpUse=totalFromThisPRC-prcUse;
  html+=`<div class='bag'><div class='bag-title'>Unit ที่ ${bag}</div>
  <div class='row'><span class='name'>PRC</span><span class='label'>ปริมาตร</span><span class='value'>${prcUse.toFixed(0)}</span><span class='unit'>ml</span></div>
  <div class='row'><span class='name'>เติม FFP</span><span class='label'>ปริมาตร</span><span class='value'>${ffpUse.toFixed(0)}</span><span class='unit'>ml</span></div></div>`;
  text+=`Unit ที่ ${bag}\nPRC ปริมาตร ${prcUse.toFixed(0)} ml\nเติม FFP ปริมาตร ${ffpUse.toFixed(0)} ml\n\n`;
  totalPRC+=prcUse;totalFFP+=ffpUse;remainingTotal-=(prcUse+ffpUse);bag++;
 }
 const totalVol=totalPRC+totalFFP;
 html+=`<div class='final'>สรุป: ใช้ PRC ปริมาตร ${totalPRC.toFixed(0)} ml และ FFP ปริมาตร ${totalFFP.toFixed(0)} ml<br>รวมปริมาตรทั้งหมด ${totalVol.toFixed(0)} ml</div>`;
 text+=`สรุป: ใช้ PRC ปริมาตร ${totalPRC.toFixed(0)} ml และ FFP ปริมาตร ${totalFFP.toFixed(0)} ml\nรวมปริมาตรทั้งหมด ${totalVol.toFixed(0)} ml\n`;
 plainTextResult=text;output.innerHTML=html;copyBtn.style.display='block';copyNote.style.display='block';
}

function resetForm(){plainTextResult="";wholeEl.value="";prcEl.value="";output.innerHTML="";copyBtn.style.display='none';copyNote.style.display='none';wholeEl.focus();}
function copyResult(){if(!plainTextResult)return;if(navigator.clipboard&&window.isSecureContext){navigator.clipboard.writeText(plainTextResult).then(()=>alert('คัดลอกแล้ว')).catch(()=>fallbackCopy());}else{fallbackCopy();}}
function fallbackCopy(){fallback.value=plainTextResult;fallback.select();try{document.execCommand('copy');alert('คัดลอกแล้ว');}catch{alert('กดค้างเพื่อคัดลอก');}}

document.getElementById('calcBtn').addEventListener('click',calculate);
document.getElementById('resetBtn').addEventListener('click',resetForm);
copyBtn.addEventListener('click',copyResult);
document.addEventListener('keydown',e=>{if(e.key==='Enter')calculate();});
wholeEl.focus();
</script>
</body>
</html>
