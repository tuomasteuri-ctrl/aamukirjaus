[aamukirjaus (1).html](https://github.com/user-attachments/files/28510871/aamukirjaus.1.html)
<!DOCTYPE html>
<html lang="fi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Aamukirjaus">
<title>Aamukirjaus</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif;
    background: #0a0a0a;
    color: #f0f0f0;
    min-height: 100vh;
    padding: env(safe-area-inset-top, 20px) 20px env(safe-area-inset-bottom, 20px);
    padding-top: max(env(safe-area-inset-top), 48px);
  }
  .header { margin-bottom: 32px; }
  .header .day { font-size: 13px; color: #888; letter-spacing: 0.08em; text-transform: uppercase; margin-bottom: 4px; }
  .header .date { font-size: 28px; font-weight: 600; color: #f0f0f0; }
  .card {
    background: #1a1a1a;
    border-radius: 16px;
    padding: 20px;
    margin-bottom: 12px;
  }
  .card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px; }
  .card-label { font-size: 15px; color: #aaa; }
  .card-value { font-size: 32px; font-weight: 600; color: #f0f0f0; line-height: 1; }
  .slider-row { display: flex; align-items: center; gap: 10px; }
  .slider-edge { font-size: 11px; color: #555; width: 16px; text-align: center; }
  input[type=range] {
    flex: 1;
    -webkit-appearance: none;
    height: 4px;
    border-radius: 2px;
    background: #333;
    outline: none;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: #f0f0f0;
    cursor: pointer;
  }
  .status {
    margin-top: 10px;
    font-size: 12px;
    font-weight: 500;
    padding: 3px 10px;
    border-radius: 20px;
    display: inline-block;
  }
  .status.great  { background: #1a3a1a; color: #5dca85; }
  .status.good   { background: #1a2e2a; color: #3dcaa5; }
  .status.ok     { background: #2e2410; color: #ef9f27; }
  .status.weak   { background: #2e1414; color: #e24b4a; }
  .submit {
    width: 100%;
    padding: 18px;
    font-size: 17px;
    font-weight: 600;
    border-radius: 16px;
    border: none;
    background: #f0f0f0;
    color: #0a0a0a;
    cursor: pointer;
    margin-top: 8px;
    letter-spacing: -0.01em;
  }
  .submit:active { opacity: 0.8; transform: scale(0.98); }
  .submit:disabled { background: #333; color: #666; }
  .success {
    display: none;
    background: #1a3a1a;
    border-radius: 16px;
    padding: 20px;
    text-align: center;
    margin-top: 12px;
  }
  .success .icon { font-size: 32px; margin-bottom: 8px; }
  .success p { color: #5dca85; font-size: 15px; line-height: 1.5; }
  .hint { font-size: 12px; color: #555; text-align: center; margin-top: 16px; line-height: 1.6; }
</style>
</head>
<body>

<div class="header">
  <div class="day" id="day-label"></div>
  <div class="date" id="date-label"></div>
</div>

<div class="card">
  <div class="card-header">
    <span class="card-label">Harjoitteluvalmius</span>
    <span class="card-value" id="val-valmius">70</span>
  </div>
  <div class="slider-row">
    <span class="slider-edge">0</span>
    <input type="range" min="0" max="100" value="70" step="1" id="sl-valmius" oninput="update('valmius',this.value)">
    <span class="slider-edge">100</span>
  </div>
  <span class="status" id="pill-valmius"></span>
</div>

<div class="card">
  <div class="card-header">
    <span class="card-label">Unipisteet</span>
    <span class="card-value" id="val-uni">70</span>
  </div>
  <div class="slider-row">
    <span class="slider-edge">0</span>
    <input type="range" min="0" max="100" value="70" step="1" id="sl-uni" oninput="update('uni',this.value)">
    <span class="slider-edge">100</span>
  </div>
  <span class="status" id="pill-uni"></span>
</div>

<div class="card">
  <div class="card-header">
    <span class="card-label">Body Battery</span>
    <span class="card-value" id="val-battery">70</span>
  </div>
  <div class="slider-row">
    <span class="slider-edge">0</span>
    <input type="range" min="0" max="100" value="70" step="1" id="sl-battery" oninput="update('battery',this.value)">
    <span class="slider-edge">100</span>
  </div>
  <span class="status" id="pill-battery"></span>
</div>

<button class="submit" id="submit-btn" onclick="submitData()">Tallenna</button>

<div class="success" id="success">
  <div class="icon">✓</div>
  <p>Tallennettu!<br>Aamuohjelma tulee sähköpostiisi.</p>
</div>

<p class="hint">Täytä heti herättyäsi.<br>Garmin näyttää Body Battery -arvon.</p>

<script>
const SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbw3vFz1aJEpCe7d1PLk91RVOjvZ1vbV1d1tEqLgp3v3jv8cyDDHl3_ArY7HoIyyVEJl/exec';

const days = ['Sunnuntai','Maanantai','Tiistai','Keskiviikko','Torstai','Perjantai','Lauantai'];
const months = ['tammikuuta','helmikuuta','maaliskuuta','huhtikuuta','toukokuuta','kesäkuuta','heinäkuuta','elokuuta','syyskuuta','lokakuuta','marraskuuta','joulukuuta'];
const now = new Date();
document.getElementById('day-label').textContent = days[now.getDay()];
document.getElementById('date-label').textContent = now.getDate() + '. ' + months[now.getMonth()];

function getStatus(v) {
  v = parseInt(v);
  if (v >= 80) return ['Erinomainen', 'great'];
  if (v >= 60) return ['Hyvä', 'good'];
  if (v >= 40) return ['Kohtalainen', 'ok'];
  return ['Heikko', 'weak'];
}

function update(key, val) {
  document.getElementById('val-' + key).textContent = Math.round(val);
  const [label, cls] = getStatus(val);
  const pill = document.getElementById('pill-' + key);
  pill.textContent = label;
  pill.className = 'status ' + cls;
}

['valmius','uni','battery'].forEach(k => update(k, 70));

function pad(n) { return String(n).padStart(2,'0'); }
function getToday() {
  const d = new Date();
  return d.getFullYear() + '-' + pad(d.getMonth()+1) + '-' + pad(d.getDate());
}

function submitData() {
  const btn = document.getElementById('submit-btn');
  btn.disabled = true;
  btn.textContent = 'Tallennetaan...';
  const params = new URLSearchParams({
    pvm: getToday(),
    valmius: document.getElementById('sl-valmius').value,
    uni: document.getElementById('sl-uni').value,
    battery: document.getElementById('sl-battery').value
  });
  fetch(SCRIPT_URL + '?' + params.toString(), { method: 'GET', mode: 'no-cors' })
    .finally(() => {
      document.getElementById('success').style.display = 'block';
      btn.textContent = 'Tallennettu ✓';
    });
}
</script>
</body>
</html>
