<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>車牌查詢</title>
<style>
* { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
:root {
  --bg: #0b0f1a; --card: #141c2e; --btn: #1e2a40;
  --cyan: #00e5ff; --orange: #ff6b35; --red: #ff3355;
  --text: #dde6f0; --dim: #556070;
}
body {
  background: var(--bg); color: var(--text);
  font-family: -apple-system, 'Noto Sans TC', sans-serif;
  min-height: 100vh; display: flex; flex-direction: column;
  align-items: center; padding: 20px 16px 40px; gap: 12px;
}
h1 { font-size: 1.3rem; color: var(--cyan); letter-spacing: 0.1em; text-align: center; }

/* Display */
.display-wrap {
  width: 100%; max-width: 400px; background: var(--card);
  border: 2px solid rgba(0,229,255,0.3); border-radius: 16px;
  padding: 14px 16px; display: flex; align-items: center; gap: 10px; min-height: 68px;
}
#display {
  flex: 1; font-size: 1.9rem; font-family: 'Courier New', monospace;
  letter-spacing: 0.15em; color: var(--cyan);
  background: none; border: none; outline: none;
  min-width: 0; width: 100%; caret-color: var(--cyan);
}

/* Mic */
#micBtn {
  width: 100%; max-width: 400px; height: 90px;
  background: var(--card); border: 2px solid var(--cyan);
  border-radius: 18px; color: var(--cyan); font-size: 2.2rem; cursor: pointer;
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 4px;
  position: relative; overflow: hidden;
}
#micBtn.listening { border-color: var(--red); color: var(--red); background: rgba(255,51,85,0.08); animation: pulse-border 1s infinite; }
@keyframes pulse-border { 0%,100% { box-shadow: 0 0 0 0 rgba(255,51,85,0.4); } 50% { box-shadow: 0 0 0 12px rgba(255,51,85,0); } }

/* Numpad */
#numpad {
  width: 100%; max-width: 400px;
  display: grid; grid-template-columns: repeat(3,1fr); gap: 8px;
  padding: 10px 0;
}
.nk { background: var(--btn); border: none; border-radius: 12px; color: var(--text); font-size: 1.5rem; height: 54px; cursor: pointer; }
#btnZero { grid-column: 1/3; }
#btnDel  { background: rgba(255,107,53,0.12); color: var(--orange); }

/* Overlay */
#overlay {
  position: fixed; inset: 0; z-index: 50; background: var(--bg);
  display: flex; flex-direction: column; transform: translateY(100%);
  transition: transform 0.3s cubic-bezier(.4,0,.2,1); pointer-events: none;
}
#overlay.open { transform: translateY(0); pointer-events: all; }
#overlayHeader { background: var(--card); flex-shrink: 0; }
#overlayHeaderTop { display: flex; align-items: center; justify-content: space-between; padding: 14px 16px 8px; }
#btnBack { background: rgba(0,229,255,0.1); border: 1.5px solid var(--cyan); border-radius: 10px; color: var(--cyan); padding: 10px 18px; cursor: pointer; }
#ringCircle { fill: none; stroke: var(--orange); stroke-width: 3.5; stroke-dasharray: 110; stroke-dashoffset: 0; }
#overlayTitle { font-family: 'Courier New', monospace; font-size: 2.2rem; color: var(--cyan); padding: 4px 16px 14px; text-align: center; }
#overlayBody { flex: 1; overflow-y: auto; padding: 12px 16px; display: flex; flex-direction: column; gap: 10px; }

.rcard { background: var(--card); border: 1px solid rgba(0,229,255,0.18); border-radius: 14px; padding: 12px 18px; display: flex; align-items: center; justify-content: space-between; gap: 10px; }
.rplate { font-size: 2rem; color: var(--cyan); font-family: 'Courier New', monospace; }
.rhouse { font-size: 1.8rem; color: var(--orange); }
.btn-del-item { background: var(--red); color: white; border: none; border-radius: 6px; padding: 8px 12px; font-size: 0.9rem; }

/* Database Button */
#dbBtn { margin-top: 10px; background: none; border: 1px solid var(--dim); color: var(--dim); padding: 8px 16px; border-radius: 8px; font-size: 0.8rem; cursor: pointer; }

/* Custom Input Modal */
#customModal {
  position: fixed; inset: 0; z-index: 100; background: rgba(0,0,0,0.85);
  display: none; align-items: center; justify-content: center; padding: 20px;
}
.modal-content {
  background: var(--card); width: 100%; max-width: 320px; border-radius: 16px;
  padding: 20px; border: 1px solid var(--cyan); box-shadow: 0 10px 30px rgba(0,0,0,0.5);
}
.modal-title { color: var(--cyan); margin-bottom: 15px; font-size: 1.1rem; font-weight: bold; }
#modalInput {
  width: 100%; background: var(--bg); border: 1px solid var(--dim); color: #fff;
  padding: 12px; border-radius: 8px; font-size: 1.4rem; margin-bottom: 20px; outline: none;
}
.modal-btns { display: flex; gap: 10px; }
.m-btn { flex: 1; padding: 12px; border: none; border-radius: 8px; font-size: 1rem; cursor: pointer; font-weight: bold; }
#modalCancel { background: var(--btn); color: var(--text); }
#modalConfirm { background: var(--cyan); color: var(--bg); }
</style>
</head>
<body>

<h1>🚗 車牌查詢</h1>

<div class="display-wrap">
  <input type="text" id="display" placeholder="點擊或語音輸入…" autocomplete="off">
  <button id="btnX" style="background:none; border:none; color:var(--dim); font-size:1.2rem; cursor:pointer;">✕</button>
</div>

<button id="micBtn">
  <span id="micIcon">🎤</span>
  <span class="mic-label" id="micLabel">開啟自動錄音</span>
</button>

<div id="numpad">
  <button class="nk" data-v="1">1</button><button class="nk" data-v="2">2</button><button class="nk" data-v="3">3</button>
  <button class="nk" data-v="4">4</button><button class="nk" data-v="5">5</button><button class="nk" data-v="6">6</button>
  <button class="nk" data-v="7">7</button><button class="nk" data-v="8">8</button><button class="nk" data-v="9">9</button>
  <button class="nk" id="btnZero" data-v="0">0</button><button class="nk" id="btnDel">⌫</button>
</div>

<button id="dbBtn">📋 管理資料庫</button>

<div id="overlay">
  <div id="overlayHeader">
    <div id="overlayHeaderTop">
      <button id="btnBack">← 返回</button>
      <div style="display:flex; align-items:center; gap:10px;">
        <div id="overlayCount" style="color:var(--dim)"></div>
        <div style="position:relative; width:48px; height:48px; display:flex; align-items:center; justify-content:center;">
          <svg width="48" height="48" style="transform: rotate(-90deg); position:absolute;"><circle id="ringCircle" cx="24" cy="24" r="17.5"/></svg>
          <span id="ringNum" style="color:var(--orange); font-weight:bold;">5</span>
        </div>
      </div>
    </div>
    <div id="overlayTitle"></div>
  </div>
  <div id="overlayBody"></div>
</div>

<div id="customModal">
  <div class="modal-content">
    <div class="modal-title" id="modalTitle">標題</div>
    <input type="text" id="modalInput" autocomplete="off">
    <div class="modal-btns">
      <button id="modalCancel" class="m-btn">取消</button>
      <button id="modalConfirm" class="m-btn">確定</button>
    </div>
  </div>
</div>

<script>
var RAW = [[46,'WF8093'],[46,'SRS'],[30,'JK9839'],[52,'XC7111'],[52,'YV5674'],[52,'VF8333'],[1,'BW773'],[1,'BZ809'],[1,'XF9940'],[2,'LC2'],[2,'AT833'],[2,'FT850'],[2,'RS6399'],[2,'FE6993'],[2,'UL8653'],[3,'EK211'],[3,'ZF1244'],[3,'GJ1878'],[3,'WS2089'],[3,'MU2187'],[3,'BL2810'],[3,'WG6212'],[3,'SE7443'],[3,'MV8639'],[4,'RW861'],[4,'EH983'],[4,'DB1283'],[4,'WD4173'],[4,'SK6299'],[4,'HU7545'],[5,'CR488'],[5,'YC1022'],[5,'PA2919'],[5,'NT8002'],[5,'HU8518'],[5,'HH9283'],[6,'DP183'],[6,'GP1213'],[6,'CS8606'],[6,'DC8886'],[7,'PJ341'],[7,'RR3922'],[7,'PX5570'],[7,'YE6475'],[7,'NY8069'],[7,'UR8737'],[8,'TZ2728'],[8,'SL6777'],[9,'MC143'],[9,'GX190'],[9,'AV2040'],[9,'MC2171'],[9,'NU2770'],[9,'RJ3183'],[9,'WN8179'],[9,'UU9633'],[10,'NE3766'],[10,'NJ5046'],[10,'TN5528'],[11,'DF112'],[11,'CR213'],[11,'RX1643'],[11,'YE3687'],[11,'SE9151'],[12,'GB2212'],[12,'GJ6222'],[12,'DW7707'],[12,'SU8306'],[13,'HA1012'],[13,'WP1052'],[13,'UU6818'],[14,'WB1949'],[14,'JL2232'],[14,'EZ8393'],[15,'EE6'],[15,'EE7'],[15,'VT1314'],[15,'VP3044'],[15,'YL5134'],[15,'UA6435'],[16,'JS360'],[16,'KV1212'],[16,'YL2329'],[17,'JC77'],[17,'JC111'],[17,'WD367'],[17,'JC777'],[17,'AX813'],[17,'BD1228'],[17,'KL1980'],[17,'DD2829'],[17,'MU3811'],[17,'VE4310'],[17,'TK5066'],[17,'XB6500'],[17,'NR8022'],[17,'TP8165'],[17,'GK9383'],[18,'YR6680'],[18,'NY6913'],[18,'YJ8930'],[18,'ML9728'],[19,'XH139'],[19,'JY1398'],[19,'AC1399'],[19,'JZ3383'],[19,'KE9983'],[20,'HW123'],[20,'ME128'],[20,'ET288'],[20,'GZ936'],[20,'JX1318'],[20,'SF1322'],[20,'YY6308'],[20,'TK9683'],[21,'MS603'],[21,'LL729'],[21,'TT763'],[21,'DD882'],[21,'WE1078'],[21,'AL1107'],[21,'MR1121'],[21,'ZA1838'],[21,'MY1968'],[21,'WU2766'],[21,'TL5970'],[22,'AC1123'],[22,'MT1816'],[22,'MK1907'],[22,'SB4468'],[22,'YP9052'],[25,'PH82'],[25,'AC331'],[25,'UP2925'],[25,'MM5969'],[25,'AB7878'],[26,'JW166'],[26,'JM518'],[26,'JH666'],[26,'MH7879'],[28,'CB998'],[29,'VK150'],[29,'KV887'],[29,'CA3222'],[30,'WM3839'],[30,'SH7419'],[30,'PH8908'],[31,'BB92'],[31,'LL138'],[31,'RR281'],[31,'LL316'],[31,'AD8799'],[32,'AS127'],[32,'YM2127'],[32,'AB2211'],[32,'ML2382'],[32,'ZJ3244'],[32,'KE3382'],[32,'KY6884'],[32,'YF8029'],[32,'BC8969'],[32,'ZA9336'],[33,'SJ9818'],[34,'FC398'],[34,'FN1128'],[34,'XA1506'],[34,'XS3547'],[34,'WJ5255'],[34,'RM6228'],[34,'XF9967'],[35,'WE2697'],[35,'WJ3661'],[35,'NP7293'],[36,'YJ478'],[36,'UP3340'],[36,'XS5756'],[36,'JS7888'],[36,'NP8198'],[36,'XD8521'],[36,'VU9342'],[37,'TT2295'],[37,'CA2722'],[37,'VM5651'],[37,'UH5788'],[37,'TG6359'],[37,'WE7469'],[37,'TU8802'],[38,'DK1298'],[38,'EL1777'],[38,'XD3105'],[38,'ZD5659'],[38,'UW5901'],[38,'PL8009'],[38,'RY8230'],[39,'DN91'],[39,'DB238'],[39,'FU307'],[40,'SV2218'],[40,'XA6119'],[40,'TJ6340'],[41,'FN250'],[41,'KR326'],[41,'DH730'],[41,'GP1812'],[42,'LD205'],[42,'YG1440'],[42,'RC2272'],[42,'PV2777'],[42,'VY3255'],[42,'KG3268'],[42,'TD4532'],[42,'DW8388'],[42,'VR8506'],[42,'DB8563'],[42,'DG8813'],[42,'CR8888'],[43,'GC3805'],[43,'EF6337'],[43,'TF6686'],[44,'HN733'],[44,'NK2820'],[45,'MY33'],[45,'GC2622'],[45,'YW3980'],[45,'VH4489'],[45,'BK6000'],[45,'RS8587'],[46,'BG162'],[46,'EC660'],[46,'UB1008'],[46,'UG8745'],[47,'EH8765'],[47,'JW9006'],[48,'JL6780'],[49,'MAC168'],[49,'UH621'],[49,'XC4793'],[49,'YM5071'],[49,'VX5753'],[50,'GK8939'],[51,'EK889'],[51,'PW1044'],[52,'BG136'],[52,'ED735'],[52,'SH2816'],[52,'YH4475'],[52,'SW5430'],[52,'SU9219'],[53,'EW132'],[54,'VH1824'],[54,'KT2271'],[54,'YL9895'],[55,'MS530'],[55,'PG5248'],[55,'SK7287'],[56,'EP286'],[56,'EM1127'],[56,'VD1601'],[56,'LW4833'],[56,'GF5188'],[56,'UP8623'],[57,'AC411'],[57,'HU2290'],[57,'ME5823'],[57,'AK9008'],[58,'EX106'],[58,'WC3672'],[58,'TS4887'],[58,'VD9416'],['麗華','WU1115'],['GOV','XF2436'],['GOV','VY3652'],['GOV','XB4739'],['GOV','VY5623'],['GOV','XR6422'],['POLICE','WB5463'],[38,'NFY2'],[24,'GK112'],[41,'MN250'],[21,'VC316'],[34,'FP398'],[41,'VA621'],[22,'PW813'],[22,'YK887'],[21,'TZ936'],[28,'GB998'],[21,'MR1107'],[41,'BZ1123'],[21,'TT1127'],[13,'VV1228'],[43,'WX2770'],[32,'RH3382'],[30,'WN3839'],[42,'XR8506'],[46,'VC8745'],[48,'CB8813'],[24,'JE112'],[52,'JB136'],[41,'WN250'],[40,'MONK'],['H20','WZ9667'],['H25','PH82'],['H6','PP8018'],['H32','RH3382'],['H38','RC748'],['H39','RS3878'],['H26','RF8470'],['H42','SY8283'],['H3','SF1244'],['H27','SU7532'],['H35','SP1036'],['H35','SHAO'],['H7','SU9219'],['H46','SRS'],['H30','SS427'],['H3','SL8706'],['H32','RH3382'],['H38','RC748'],['H26','RF8470'],['H37','TU8802'],['H43','TD4532'],['H21','TS3505'],['H58','TR7857'],['H25','VP2925'],['H36','VU9342'],['H21','VC316'],['H46','VG8745'],['H47','VL3255'],['H35','UR4530'],['H50','UP6301'],['H6','UA1006'],['H3','WV8639'],['H35','WC6487'],['H17','WD367'],['H3','WG1165'],['H34','XF9967'],['H39','D5502'],['H39','XD5502'],[52,'XC7111'],[49,'XM9271'],[9,'XA8185'],[30,'YP6495'],[52,'YH4475'],[32,'YF8029'],[40,'YX2378'],[13,'YW7141'],[47,'YL3255'],[42,'YF6608'],[21,'ZA1838'],[32,'ZA9336'],[3,'ZF1244'],[32,'ZJ3244'],[45,'ZU3798'],[38,'ZD5659'],[24,'ZD8344'],[45,'ZU3798'],[47,'ZY7403'],[37,'ZU5856'],[21,'FF720'],[30,'FU1995'],[6,'FU1613']];

var storageKey = 'CUSTOM_PLATE_DATA';
var savedData = JSON.parse(localStorage.getItem(storageKey) || "[]");
var DATA = RAW.concat(savedData);

var elDisplay = document.getElementById('display');
var elOverlay = document.getElementById('overlay');
var elMicBtn = document.getElementById('micBtn');
var elOvBody = document.getElementById('overlayBody');
var isListening = false;
var autoCloseTimer = null;
var ringTickTimer = null;

var SR = window.SpeechRecognition || window.webkitSpeechRecognition;
var recognizer = null;
if(SR) {
  recognizer = new SR();
  recognizer.lang = 'zh-HK';
  recognizer.onstart = () => { isListening = true; document.getElementById('micBtn').classList.add('listening'); };
  recognizer.onend = () => { isListening = false; document.getElementById('micBtn').classList.remove('listening'); };
  recognizer.onresult = (e) => {
    var res = e.results[0][0].transcript.replace(/[^A-Za-z0-9]/g,'').toUpperCase();
    elDisplay.value = res;
    doSearch(res);
  };
}

function doSearch(q) {
  q = (q || elDisplay.value).trim().toUpperCase();
  if(!q) return;
  var hits = DATA.filter(r => String(r[1]).toUpperCase().includes(q));
  openOverlay(q, hits);
}

function openOverlay(q, hits) {
  stopRing();
  document.getElementById('overlayTitle').textContent = q;
  document.getElementById('overlayCount').textContent = hits.length ? '共 '+hits.length+' 筆' : '查無結果';
  elOvBody.innerHTML = '';

  if(hits.length === 0) {
    var d = document.createElement('div');
    d.className = 'rnone';
    d.style.padding = "20px"; d.style.textAlign = "center"; d.style.color = "var(--dim)";
    d.textContent = '查無「'+q+'」相關車牌';
    elOvBody.appendChild(d);
    setTimeout(() => { addNewRecord(q); }, 500);
  } else {
    hits.forEach(r => {
      var c = document.createElement('div');
      c.className = 'rcard';
      c.innerHTML = `<div class="rhouse">${r[0]}</div><div class="rplate">${r[1]}</div>`;
      elOvBody.appendChild(c);
    });
    elOverlay.classList.add('open');
    startRing(5, () => { closeOverlay(); });
  }
}

// ══════════════════════════════════════
// 修正處：新增記錄 (清空內容 + 強制聚焦 + 語音連動)
// ══════════════════════════════════════

function showCustomInput(title, defaultValue, isNumeric = false) {
  return new Promise((resolve) => {
    const modal = document.getElementById('customModal');
    const input = document.getElementById('modalInput');
    const titleEl = document.getElementById('modalTitle');
    
    titleEl.textContent = title;
    input.value = defaultValue;
    
    if(isNumeric) {
        input.inputMode = "numeric"; input.type = "tel";
    } else {
        input.inputMode = "text"; input.type = "text";
    }
    
    modal.style.display = 'flex';

    // 強制聚焦並觸發語音
    setTimeout(() => {
      input.focus();
      if (recognizer) {
        // 先暫停主畫面識別，重新掛載 Modal 專用邏輯
        try { recognizer.stop(); } catch(e){}
        const oldResult = recognizer.onresult;
        recognizer.onresult = (e) => {
          let res = e.results[0][0].transcript;
          res = isNumeric ? res.replace(/[^0-9]/g,'') : res.replace(/[^A-Za-z0-9]/g,'').toUpperCase();
          input.value = res;
          // 語音識別成功後短暫停頓自動送出
          setTimeout(() => {
            recognizer.onresult = oldResult;
            modal.style.display = 'none';
            resolve(input.value.trim());
          }, 600);
        };
        recognizer.start();
      }
    }, 200);

    document.getElementById('modalConfirm').onclick = () => { 
      if(recognizer) try { recognizer.stop(); } catch(e){}
      modal.style.display = 'none'; 
      resolve(input.value.trim()); 
    };
    document.getElementById('modalCancel').onclick = () => { 
      if(recognizer) try { recognizer.stop(); } catch(e){}
      modal.style.display = 'none'; 
      resolve(null); 
    };
  });
}

async function addNewRecord(detectedPlate) {
  stopRing(); 
  
  const plate = await showCustomInput("請輸入正確車牌：", "", false);
  if (!plate) { closeOverlay(); startMic(); return; }
  
  const house = await showCustomInput("請輸入對應的「屋號」：", "", true);
  if (!house) { closeOverlay(); startMic(); return; }

  var newItem = [house, plate.toUpperCase()];
  savedData.push(newItem);
  localStorage.setItem(storageKey, JSON.stringify(savedData));
  DATA = RAW.concat(savedData);
  
  alert("已成功登記");
  closeOverlay();
  doSearch(newItem[1]);
}

// ══════════════════════════════════════
// 其餘功能保持不變
// ══════════════════════════════════════

document.getElementById('dbBtn').addEventListener('click', () => {
  stopRing();
  document.getElementById('overlayTitle').textContent = "資料庫管理";
  document.getElementById('overlayCount').textContent = `共 ${savedData.length} 筆資料`;
  elOvBody.innerHTML = '';
  if(savedData.length === 0) {
    elOvBody.innerHTML = '<div class="rnone" style="padding:20px; text-align:center; color:var(--dim);">目前無自定義資料</div>';
  } else {
    savedData.forEach((r, idx) => {
      var c = document.createElement('div');
      c.className = 'rcard';
      c.innerHTML = `
        <div style="display:flex; flex-direction:column;">
          <span style="font-size:0.75rem; color:var(--dim)">屋號 ${r[0]}</span>
          <span class="rplate" style="font-size:1.6rem;">${r[1]}</span>
        </div>
        <button class="btn-del-item" onclick="deleteRecord(${idx})">刪除</button>`;
      elOvBody.appendChild(c);
    });
  }
  elOverlay.classList.add('open');
});

function deleteRecord(index) {
  if(confirm("確定刪除此筆資料？")) {
    savedData.splice(index, 1);
    localStorage.setItem(storageKey, JSON.stringify(savedData));
    DATA = RAW.concat(savedData);
    document.getElementById('dbBtn').click();
  }
}

function startMic() { if(recognizer && !isListening) recognizer.start(); }
function stopMic() { if(recognizer && isListening) recognizer.stop(); }
function closeOverlay() { stopRing(); elOverlay.classList.remove('open'); elDisplay.value = ''; }

function startRing(secs, done) {
  var rem = secs;
  document.getElementById('ringNum').textContent = rem;
  var circle = document.getElementById('ringCircle');
  circle.style.transition = 'none'; circle.style.strokeDashoffset = '0';
  setTimeout(() => { circle.style.transition = `stroke-dashoffset ${secs}s linear`; circle.style.strokeDashoffset = '110'; }, 50);
  ringTickTimer = setInterval(() => { rem--; document.getElementById('ringNum').textContent = Math.max(0, rem); }, 1000);
  autoCloseTimer = setTimeout(done, secs * 1000);
}
function stopRing() { if(autoCloseTimer) clearTimeout(autoCloseTimer); if(ringTickTimer) clearInterval(ringTickTimer); }

window.onload = () => { setTimeout(() => { startMic(); }, 800); };
elMicBtn.addEventListener('click', () => isListening ? stopMic() : startMic());
document.getElementById('btnBack').addEventListener('click', closeOverlay);
document.getElementById('btnX').addEventListener('click', () => { elDisplay.value = ''; elDisplay.focus(); });

document.getElementById('numpad').addEventListener('click', (e) => {
  var b = e.target.closest('[data-v]');
  if(b) { elDisplay.value += b.getAttribute('data-v'); if(elDisplay.value.length >= 3) doSearch(); }
});

document.getElementById('btnDel').addEventListener('click', () => {
  elDisplay.value = elDisplay.value.slice(0, -1);
  if(elDisplay.value.length >= 3) doSearch();
});
</script>
</body>
</html>
