<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Hospital Data Analysis Dashboard</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600;700&display=swap" rel="stylesheet"/>
<style>
  :root {
    --bg:        #050d1a;
    --panel:     #0a1628;
    --panel2:    #0f1e36;
    --border:    #1a2e50;
    --cyan:      #00d4ff;
    --cyan2:     #00ffcc;
    --pink:      #ff3e7f;
    --orange:    #ff8c42;
    --yellow:    #ffd166;
    --purple:    #7b5ea7;
    --green:     #06d6a0;
    --text:      #e8f4ff;
    --muted:     #6b8aaa;
    --font-head: 'Space Mono', monospace;
    --font-body: 'DM Sans', sans-serif;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    font-family: var(--font-body);
    color: var(--text);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── animated grid background ── */
  body::before {
    content: '';
    position: fixed; inset: 0; z-index: 0;
    background-image:
      linear-gradient(var(--border) 1px, transparent 1px),
      linear-gradient(90deg, var(--border) 1px, transparent 1px);
    background-size: 44px 44px;
    opacity: 0.35;
    pointer-events: none;
  }

  body::after {
    content: '';
    position: fixed; inset: 0; z-index: 0;
    background: radial-gradient(ellipse 80% 50% at 50% -20%, rgba(0,212,255,.12) 0%, transparent 60%),
                radial-gradient(ellipse 60% 40% at 90% 80%, rgba(255,62,127,.08) 0%, transparent 60%);
    pointer-events: none;
  }

  /* ── layout ── */
  .wrapper { position: relative; z-index: 1; max-width: 1540px; margin: 0 auto; padding: 0 24px 40px; }

  /* ── header ── */
  header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 28px 0 24px;
    border-bottom: 1px solid var(--border);
    margin-bottom: 32px;
  }
  .logo-area { display: flex; align-items: center; gap: 14px; }
  .logo-icon {
    width: 44px; height: 44px; border-radius: 12px;
    background: linear-gradient(135deg, var(--cyan), var(--cyan2));
    display: flex; align-items: center; justify-content: center;
    font-size: 22px; box-shadow: 0 0 20px rgba(0,212,255,.4);
    animation: pulse-glow 3s ease-in-out infinite;
  }
  @keyframes pulse-glow {
    0%,100% { box-shadow: 0 0 20px rgba(0,212,255,.4); }
    50%      { box-shadow: 0 0 36px rgba(0,212,255,.7); }
  }
  .logo-text { font-family: var(--font-head); font-size: 13px; line-height: 1.5; }
  .logo-text span { display: block; color: var(--cyan); font-size: 10px; letter-spacing: 3px; text-transform: uppercase; }
  .header-meta { display: flex; gap: 24px; align-items: center; }
  .live-badge {
    display: flex; align-items: center; gap: 7px;
    background: rgba(6,214,160,.12); border: 1px solid rgba(6,214,160,.3);
    padding: 6px 14px; border-radius: 99px;
    font-family: var(--font-head); font-size: 11px; color: var(--green);
    letter-spacing: 1px;
  }
  .live-dot {
    width: 8px; height: 8px; background: var(--green); border-radius: 50%;
    animation: blink 1.4s ease-in-out infinite;
  }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:.2} }
  .header-date { font-size: 12px; color: var(--muted); font-family: var(--font-head); }

  /* ── nav tabs ── */
  .nav-tabs {
    display: flex; gap: 4px; margin-bottom: 28px;
    background: var(--panel); border: 1px solid var(--border);
    border-radius: 14px; padding: 5px; width: fit-content;
  }
  .tab {
    padding: 9px 22px; border-radius: 10px; font-size: 13px; font-weight: 600;
    cursor: pointer; transition: all .25s; color: var(--muted); border: none;
    background: transparent; font-family: var(--font-body); letter-spacing: .3px;
  }
  .tab.active { background: var(--panel2); color: var(--cyan); box-shadow: 0 0 14px rgba(0,212,255,.15); }
  .tab:hover:not(.active) { color: var(--text); }

  /* ── KPI strip ── */
  .kpi-grid {
    display: grid; grid-template-columns: repeat(5, 1fr); gap: 14px; margin-bottom: 24px;
  }
  .kpi {
    background: var(--panel); border: 1px solid var(--border); border-radius: 16px;
    padding: 20px 22px; position: relative; overflow: hidden;
    transition: transform .2s, box-shadow .2s;
    animation: fadeUp .5s ease both;
  }
  .kpi:hover { transform: translateY(-3px); box-shadow: 0 8px 32px rgba(0,0,0,.4); }
  .kpi::before {
    content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px;
    border-radius: 16px 16px 0 0;
  }
  .kpi.c1::before { background: linear-gradient(90deg, var(--cyan), var(--cyan2)); }
  .kpi.c2::before { background: linear-gradient(90deg, var(--pink), #ff9a9e); }
  .kpi.c3::before { background: linear-gradient(90deg, var(--orange), var(--yellow)); }
  .kpi.c4::before { background: linear-gradient(90deg, var(--green), #80ed99); }
  .kpi.c5::before { background: linear-gradient(90deg, var(--purple), #c4b5fd); }
  .kpi-label { font-size: 11px; color: var(--muted); text-transform: uppercase; letter-spacing: 1.5px; margin-bottom: 10px; }
  .kpi-value { font-family: var(--font-head); font-size: 28px; font-weight: 700; line-height: 1; margin-bottom: 8px; }
  .kpi.c1 .kpi-value { color: var(--cyan); }
  .kpi.c2 .kpi-value { color: var(--pink); }
  .kpi.c3 .kpi-value { color: var(--orange); }
  .kpi.c4 .kpi-value { color: var(--green); }
  .kpi.c5 .kpi-value { color: var(--purple); }
  .kpi-sub { font-size: 12px; color: var(--muted); display: flex; align-items: center; gap: 5px; }
  .up { color: var(--green); } .down { color: var(--pink); }
  .kpi-icon { position: absolute; right: 18px; top: 18px; font-size: 28px; opacity: .15; }

  /* ── section grid ── */
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 18px; margin-bottom: 18px; }
  .grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 18px; margin-bottom: 18px; }
  .grid-1-2 { display: grid; grid-template-columns: 1fr 2fr; gap: 18px; margin-bottom: 18px; }
  .grid-2-1 { display: grid; grid-template-columns: 2fr 1fr; gap: 18px; margin-bottom: 18px; }
  .span2 { grid-column: span 2; }

  /* ── card ── */
  .card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 18px;
    padding: 22px 24px; position: relative; overflow: hidden;
    animation: fadeUp .6s ease both;
  }
  .card::after {
    content: ''; position: absolute; top: -80px; right: -80px;
    width: 200px; height: 200px; border-radius: 50%;
    background: radial-gradient(var(--cyan), transparent 70%);
    opacity: .03; pointer-events: none;
  }
  .card-head {
    display: flex; align-items: center; justify-content: space-between; margin-bottom: 18px;
  }
  .card-title {
    font-family: var(--font-head); font-size: 13px; color: var(--text); letter-spacing: .5px;
    display: flex; align-items: center; gap: 9px;
  }
  .card-title span { font-size: 16px; }
  .card-badge {
    font-size: 10px; padding: 4px 10px; border-radius: 99px;
    font-family: var(--font-head); letter-spacing: 1px; text-transform: uppercase;
  }
  .badge-cyan   { background: rgba(0,212,255,.1);  color: var(--cyan);   border: 1px solid rgba(0,212,255,.25); }
  .badge-pink   { background: rgba(255,62,127,.1); color: var(--pink);   border: 1px solid rgba(255,62,127,.25); }
  .badge-green  { background: rgba(6,214,160,.1);  color: var(--green);  border: 1px solid rgba(6,214,160,.25); }
  .badge-orange { background: rgba(255,140,66,.1); color: var(--orange); border: 1px solid rgba(255,140,66,.25); }
  canvas { max-height: 270px; }

  /* ── table ── */
  .data-table { width: 100%; border-collapse: collapse; font-size: 13px; }
  .data-table thead th {
    font-family: var(--font-head); font-size: 10px; letter-spacing: 1.5px;
    color: var(--muted); text-transform: uppercase; padding: 10px 14px;
    border-bottom: 1px solid var(--border); text-align: left;
  }
  .data-table tbody tr { border-bottom: 1px solid rgba(26,46,80,.5); transition: background .15s; }
  .data-table tbody tr:hover { background: rgba(0,212,255,.04); }
  .data-table td { padding: 11px 14px; color: var(--text); }
  .dept-dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; margin-right: 8px; }
  .pill {
    display: inline-block; padding: 3px 10px; border-radius: 99px; font-size: 11px; font-weight: 600;
  }
  .pill.high   { background: rgba(255,62,127,.15);  color: var(--pink); }
  .pill.med    { background: rgba(255,140,66,.15);  color: var(--orange); }
  .pill.low    { background: rgba(6,214,160,.15);   color: var(--green); }
  .pill.normal { background: rgba(0,212,255,.15);   color: var(--cyan); }

  /* ── progress bars ── */
  .prog-row { margin-bottom: 14px; }
  .prog-meta { display: flex; justify-content: space-between; font-size: 12px; margin-bottom: 6px; }
  .prog-name { color: var(--text); font-weight: 500; }
  .prog-val  { font-family: var(--font-head); font-size: 12px; }
  .prog-track { height: 6px; background: var(--border); border-radius: 99px; overflow: hidden; }
  .prog-fill  { height: 100%; border-radius: 99px; transition: width 1.5s cubic-bezier(.22,.68,0,1.2); width: 0; }

  /* ── satisfaction circles ── */
  .sat-grid { display: grid; grid-template-columns: repeat(5,1fr); gap: 10px; text-align: center; }
  .sat-item { }
  .sat-ring { position: relative; width: 70px; height: 70px; margin: 0 auto 8px; }
  .sat-ring svg { transform: rotate(-90deg); }
  .sat-ring circle { fill: none; stroke-width: 6; stroke-linecap: round; }
  .sat-ring .track { stroke: var(--border); }
  .sat-ring .fill  { stroke-dashoffset: 176; transition: stroke-dashoffset 1.5s cubic-bezier(.22,.68,0,1.2); }
  .sat-pct { position: absolute; inset: 0; display: flex; align-items: center; justify-content: center; font-family: var(--font-head); font-size: 14px; font-weight: 700; }
  .sat-label { font-size: 11px; color: var(--muted); }

  /* ── footer ── */
  footer {
    border-top: 1px solid var(--border); margin-top: 12px; padding-top: 20px;
    display: flex; justify-content: space-between; align-items: center;
    font-size: 12px; color: var(--muted);
  }
  footer span { font-family: var(--font-head); font-size: 11px; }
  footer .by { color: var(--cyan); }

  /* ── animations ── */
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(20px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .kpi:nth-child(1) { animation-delay: .05s; }
  .kpi:nth-child(2) { animation-delay: .10s; }
  .kpi:nth-child(3) { animation-delay: .15s; }
  .kpi:nth-child(4) { animation-delay: .20s; }
  .kpi:nth-child(5) { animation-delay: .25s; }

  /* ── scrollbar ── */
  ::-webkit-scrollbar { width: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }

  /* ── responsive ── */
  @media (max-width:1100px) {
    .kpi-grid { grid-template-columns: repeat(3,1fr); }
    .grid-2, .grid-1-2, .grid-2-1 { grid-template-columns: 1fr; }
    .grid-3 { grid-template-columns: 1fr 1fr; }
  }
  @media (max-width:640px) {
    .grid-3 { grid-template-columns: 1fr; }
    .kpi-grid { grid-template-columns: 1fr 1fr; }
    header { flex-direction: column; gap: 14px; }
  }
</style>
</head>
<body>
<div class="wrapper">

  <!-- ── HEADER ── -->
  <header>
    <div class="logo-area">
      <div class="logo-icon">🏥</div>
      <div class="logo-text">
        HOSPITAL ANALYTICS DASHBOARD
        <span>Rajalakshmi Institute of Technology · CSE Dept.</span>
      </div>
    </div>
    <div class="header-meta">
      <div class="live-badge"><div class="live-dot"></div>LIVE DATA · FY 2025</div>
      <div class="header-date" id="clock"></div>
    </div>
  </header>

  <!-- ── NAV ── -->
  <div class="nav-tabs">
    <button class="tab active">Overview</button>
    <button class="tab">Patient Flow</button>
    <button class="tab">Departments</button>
    <button class="tab">Financials</button>
    <button class="tab">Staff</button>
  </div>

  <!-- ── KPI STRIP ── -->
  <div class="kpi-grid">
    <div class="kpi c1">
      <div class="kpi-icon">🏥</div>
      <div class="kpi-label">Total Admissions</div>
      <div class="kpi-value">4,885</div>
      <div class="kpi-sub"><span class="up">↑ 8.2%</span> vs last year</div>
    </div>
    <div class="kpi c2">
      <div class="kpi-icon">🚨</div>
      <div class="kpi-label">Emergency Cases</div>
      <div class="kpi-value">1,100</div>
      <div class="kpi-sub"><span class="up">↑ 12%</span> YoY</div>
    </div>
    <div class="kpi c3">
      <div class="kpi-icon">🛏</div>
      <div class="kpi-label">Avg Bed Occupancy</div>
      <div class="kpi-value">82.4%</div>
      <div class="kpi-sub"><span class="up">↑ 3.1%</span> optimal</div>
    </div>
    <div class="kpi c4">
      <div class="kpi-icon">📅</div>
      <div class="kpi-label">Avg Length of Stay</div>
      <div class="kpi-value">4.4d</div>
      <div class="kpi-sub"><span class="down">↓ 0.3d</span> improved</div>
    </div>
    <div class="kpi c5">
      <div class="kpi-icon">💰</div>
      <div class="kpi-label">Total Revenue</div>
      <div class="kpi-value">₹23.5Cr</div>
      <div class="kpi-sub"><span class="up">↑ 15%</span> growth</div>
    </div>
  </div>

  <!-- ── ROW 1: Admissions line + ICU gauge ── -->
  <div class="grid-2">
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>📈</span>ADMISSIONS vs DISCHARGES</div>
        <div class="card-badge badge-cyan">2025 MONTHLY</div>
      </div>
      <canvas id="admChart"></canvas>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>🚨</span>EMERGENCY TREND</div>
        <div class="card-badge badge-pink">CRITICAL WATCH</div>
      </div>
      <canvas id="emergChart"></canvas>
    </div>
  </div>

  <!-- ── ROW 2: Dept bar + pie charts ── -->
  <div class="grid-2-1">
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>🏢</span>DEPARTMENT PATIENT LOAD</div>
        <div class="card-badge badge-orange">ALL DEPTS</div>
      </div>
      <canvas id="deptChart"></canvas>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>👥</span>AGE DISTRIBUTION</div>
        <div class="card-badge badge-cyan">DEMOGRAPHICS</div>
      </div>
      <canvas id="ageChart"></canvas>
    </div>
  </div>

  <!-- ── ROW 3: Bed Occ + LOS + Diagnosis ── -->
  <div class="grid-3">
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>🛏</span>BED OCCUPANCY %</div>
        <div class="card-badge badge-green">MONTHLY</div>
      </div>
      <canvas id="bedChart"></canvas>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>⏱</span>ICU OCCUPANCY %</div>
        <div class="card-badge badge-pink">CRITICAL</div>
      </div>
      <canvas id="icuChart"></canvas>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>🔬</span>TOP DIAGNOSES</div>
        <div class="card-badge badge-orange">DISTRIBUTION</div>
      </div>
      <canvas id="diagChart"></canvas>
    </div>
  </div>

  <!-- ── ROW 4: Revenue bar + staff grouped bar ── -->
  <div class="grid-2">
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>💹</span>REVENUE BY DEPARTMENT</div>
        <div class="card-badge badge-green">₹ CRORES</div>
      </div>
      <canvas id="revChart"></canvas>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>👨‍⚕️</span>STAFF DISTRIBUTION</div>
        <div class="card-badge badge-cyan">DOCTORS vs NURSES</div>
      </div>
      <canvas id="staffChart"></canvas>
    </div>
  </div>

  <!-- ── ROW 5: Dept table + satisfaction ── -->
  <div class="grid-1-2">
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>😊</span>PATIENT SATISFACTION</div>
        <div class="card-badge badge-green">SURVEY 2025</div>
      </div>
      <div class="sat-grid" id="satGrid"></div>
      <div style="margin-top:20px">
        <div class="prog-row" id="progs"></div>
      </div>
    </div>
    <div class="card">
      <div class="card-head">
        <div class="card-title"><span>📋</span>DEPARTMENTAL SUMMARY</div>
        <div class="card-badge badge-cyan">LIVE TABLE</div>
      </div>
      <table class="data-table">
        <thead>
          <tr>
            <th>Department</th><th>Patients</th><th>Revenue</th><th>Doctors</th><th>Load</th>
          </tr>
        </thead>
        <tbody id="deptTable"></tbody>
      </table>
    </div>
  </div>

  <!-- ── FOOTER ── -->
  <footer>
    <div>
      <strong>Abinayasri. S</strong> · Reg: 2117250020008 · CSE · Rajalakshmi Institute of Technology
    </div>
    <span>Faculty: Dr. H. Anwar Basha · <span class="by">Hospital Data Analysis · 2026</span></span>
  </footer>

</div>

<script>
// ── clock ──
function tick() {
  const d = new Date();
  document.getElementById('clock').textContent =
    d.toLocaleDateString('en-IN',{day:'2-digit',month:'short',year:'numeric'}) + '  ' +
    d.toLocaleTimeString('en-IN',{hour:'2-digit',minute:'2-digit'});
}
tick(); setInterval(tick, 1000);

// ── data ──
const months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
const admissions  = [320,290,350,380,410,370,395,420,400,360,340,450];
const discharges  = [310,280,340,370,400,360,385,410,390,350,330,440];
const emergency   = [85,78,92,88,105,95,100,112,98,90,87,120];
const icuOcc      = [72,68,75,80,85,78,82,88,84,76,73,90];
const bedOcc      = [78,72,80,85,88,82,86,90,87,81,79,92];
const depts       = ['Cardiology','Orthopedics','Pediatrics','Neurology','Oncology','Gen Surgery','Gynecology'];
const deptPts     = [520,380,450,310,290,680,340];
const deptRev     = [4.2,2.8,1.9,3.1,5.6,3.8,2.1];
const deptDoc     = [18,12,15,10,14,22,11];
const deptNurse   = [54,36,45,30,42,66,33];
const ageLbls     = ['0–18','19–35','36–50','51–65','65+'];
const ageCnts     = [15,22,28,20,15];
const diagLbls    = ['Hypertension','Diabetes','Fractures','Cardiac','Cancer','Respiratory','Others'];
const diagCnts    = [18,15,12,14,8,11,22];

// ── chart defaults ──
Chart.defaults.color = '#6b8aaa';
Chart.defaults.font.family = "'DM Sans', sans-serif";
Chart.defaults.font.size = 11;

const gridOpts = {
  color: 'rgba(26,46,80,.6)',
  borderColor: 'rgba(26,46,80,.6)'
};
const tooltipOpts = {
  backgroundColor: '#0a1628',
  borderColor: '#1a2e50',
  borderWidth: 1,
  titleColor: '#00d4ff',
  bodyColor: '#e8f4ff',
  padding: 12,
  cornerRadius: 10
};

// ── 1. Admissions vs Discharges ──
new Chart(document.getElementById('admChart'), {
  type: 'line',
  data: {
    labels: months,
    datasets: [
      {
        label: 'Admissions',
        data: admissions,
        borderColor: '#00d4ff',
        backgroundColor: 'rgba(0,212,255,.08)',
        fill: true, tension: .4, pointRadius: 4,
        pointBackgroundColor: '#00d4ff',
        pointHoverRadius: 7
      },
      {
        label: 'Discharges',
        data: discharges,
        borderColor: '#06d6a0',
        backgroundColor: 'rgba(6,214,160,.06)',
        fill: true, tension: .4, pointRadius: 4,
        pointBackgroundColor: '#06d6a0',
        pointHoverRadius: 7
      }
    ]
  },
  options: {
    responsive: true,
    plugins: { legend: { position: 'top' }, tooltip: tooltipOpts },
    scales: {
      x: { grid: gridOpts },
      y: { grid: gridOpts, beginAtZero: false }
    }
  }
});

// ── 2. Emergency Trend ──
new Chart(document.getElementById('emergChart'), {
  type: 'line',
  data: {
    labels: months,
    datasets: [{
      label: 'Emergency Cases',
      data: emergency,
      borderColor: '#ff3e7f',
      backgroundColor: 'rgba(255,62,127,.1)',
      fill: true, tension: .4, pointRadius: 4,
      pointBackgroundColor: '#ff3e7f',
      borderWidth: 2.5
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false }, tooltip: tooltipOpts,
      annotation: {} },
    scales: {
      x: { grid: gridOpts },
      y: { grid: gridOpts, beginAtZero: false }
    }
  }
});

// ── 3. Department Patient Load ──
const deptColors = ['#00d4ff','#06d6a0','#ff3e7f','#ff8c42','#ffd166','#7b5ea7','#00ffcc'];
new Chart(document.getElementById('deptChart'), {
  type: 'bar',
  data: {
    labels: depts,
    datasets: [{
      label: 'Patients',
      data: deptPts,
      backgroundColor: deptColors.map(c => c + 'cc'),
      borderColor: deptColors,
      borderWidth: 1.5, borderRadius: 7
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false }, tooltip: tooltipOpts },
    scales: {
      x: { grid: { display: false } },
      y: { grid: gridOpts }
    }
  }
});

// ── 4. Age Distribution Doughnut ──
new Chart(document.getElementById('ageChart'), {
  type: 'doughnut',
  data: {
    labels: ageLbls,
    datasets: [{
      data: ageCnts,
      backgroundColor: ['#00d4ff','#06d6a0','#ff8c42','#ff3e7f','#7b5ea7'],
      borderColor: '#050d1a', borderWidth: 3,
      hoverOffset: 10
    }]
  },
  options: {
    responsive: true, cutout: '65%',
    plugins: {
      legend: { position: 'bottom', labels: { padding: 14, boxWidth: 10 } },
      tooltip: tooltipOpts
    }
  }
});

// ── 5. Bed Occupancy ──
new Chart(document.getElementById('bedChart'), {
  type: 'bar',
  data: {
    labels: months,
    datasets: [{
      label: 'Bed Occupancy %',
      data: bedOcc,
      backgroundColor: bedOcc.map(v => v >= 85 ? 'rgba(255,62,127,.75)' : 'rgba(0,212,255,.6)'),
      borderColor: bedOcc.map(v => v >= 85 ? '#ff3e7f' : '#00d4ff'),
      borderWidth: 1.5, borderRadius: 5
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false }, tooltip: tooltipOpts },
    scales: {
      x: { grid: { display: false } },
      y: { grid: gridOpts, min: 60, max: 100,
        ticks: { callback: v => v + '%' } }
    }
  }
});

// ── 6. ICU Occupancy ──
new Chart(document.getElementById('icuChart'), {
  type: 'line',
  data: {
    labels: months,
    datasets: [{
      label: 'ICU %',
      data: icuOcc,
      borderColor: '#ff8c42',
      backgroundColor: 'rgba(255,140,66,.1)',
      fill: true, tension: .4, pointRadius: 4,
      pointBackgroundColor: icuOcc.map(v => v >= 85 ? '#ff3e7f' : '#ff8c42'),
      pointRadius: icuOcc.map(v => v >= 85 ? 7 : 4),
      borderWidth: 2
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false }, tooltip: tooltipOpts },
    scales: {
      x: { grid: gridOpts },
      y: { grid: gridOpts, min: 60, ticks: { callback: v => v + '%' } }
    }
  }
});

// ── 7. Top Diagnoses ──
new Chart(document.getElementById('diagChart'), {
  type: 'polarArea',
  data: {
    labels: diagLbls,
    datasets: [{
      data: diagCnts,
      backgroundColor: ['rgba(255,62,127,.7)','rgba(0,212,255,.7)','rgba(6,214,160,.7)',
        'rgba(255,140,66,.7)','rgba(123,94,167,.7)','rgba(0,255,204,.7)','rgba(255,209,102,.7)'],
      borderColor: '#050d1a', borderWidth: 2
    }]
  },
  options: {
    responsive: true,
    plugins: {
      legend: { position: 'bottom', labels: { padding: 8, boxWidth: 9, font: { size: 9 } } },
      tooltip: tooltipOpts
    },
    scales: { r: { grid: { color: 'rgba(26,46,80,.6)' }, ticks: { display: false } } }
  }
});

// ── 8. Revenue ──
new Chart(document.getElementById('revChart'), {
  type: 'bar',
  data: {
    labels: depts,
    datasets: [{
      label: 'Revenue (Cr ₹)',
      data: deptRev,
      backgroundColor: deptColors.map(c => c + 'b0'),
      borderColor: deptColors,
      borderWidth: 1.5, borderRadius: 8
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false }, tooltip: {
      ...tooltipOpts,
      callbacks: { label: ctx => ' ₹' + ctx.parsed.y + ' Crores' }
    } },
    scales: {
      x: { grid: { display: false } },
      y: { grid: gridOpts, ticks: { callback: v => '₹' + v + 'Cr' } }
    }
  }
});

// ── 9. Staff ──
new Chart(document.getElementById('staffChart'), {
  type: 'bar',
  data: {
    labels: depts,
    datasets: [
      {
        label: 'Doctors',
        data: deptDoc,
        backgroundColor: 'rgba(0,212,255,.7)',
        borderColor: '#00d4ff',
        borderWidth: 1.5, borderRadius: 6
      },
      {
        label: 'Nurses',
        data: deptNurse,
        backgroundColor: 'rgba(0,255,204,.5)',
        borderColor: '#00ffcc',
        borderWidth: 1.5, borderRadius: 6
      }
    ]
  },
  options: {
    responsive: true,
    plugins: { legend: { position: 'top' }, tooltip: tooltipOpts },
    scales: {
      x: { grid: { display: false } },
      y: { grid: gridOpts }
    }
  }
});

// ── Satisfaction rings ──
const satData = [
  { label: 'Excellent', pct: 38, color: '#06d6a0' },
  { label: 'Good',      pct: 32, color: '#00d4ff' },
  { label: 'Average',   pct: 18, color: '#ffd166' },
  { label: 'Poor',      pct:  8, color: '#ff8c42' },
  { label: 'Very Poor', pct:  4, color: '#ff3e7f' },
];
const satGrid = document.getElementById('satGrid');
const R = 28, C = 2 * Math.PI * R;
satData.forEach(s => {
  const offset = C - (s.pct / 100) * C;
  satGrid.innerHTML += `
    <div class="sat-item">
      <div class="sat-ring">
        <svg viewBox="0 0 70 70" width="70" height="70">
          <circle class="track" cx="35" cy="35" r="${R}" stroke="${'rgba(26,46,80,1)'}"/>
          <circle class="fill" cx="35" cy="35" r="${R}"
            stroke="${s.color}"
            stroke-dasharray="${C}"
            stroke-dashoffset="${C}"
            data-offset="${offset}"
            style="transition: stroke-dashoffset 1.5s cubic-bezier(.22,.68,0,1.2);"/>
        </svg>
        <div class="sat-pct" style="color:${s.color}">${s.pct}%</div>
      </div>
      <div class="sat-label">${s.label}</div>
    </div>`;
});

// ── Dept table ──
const loads = ['High','High','High','Med','High','Critical','Med'];
const loadCls= ['high','high','high','med','high','high','med'];
const dotC   = deptColors;
document.getElementById('deptTable').innerHTML = depts.map((d,i) => `
  <tr>
    <td><span class="dept-dot" style="background:${dotC[i]}"></span>${d}</td>
    <td>${deptPts[i].toLocaleString()}</td>
    <td>₹${deptRev[i]}Cr</td>
    <td>${deptDoc[i]}</td>
    <td><span class="pill ${loadCls[i]}">${loads[i]}</span></td>
  </tr>`).join('');

// ── animate rings on scroll ──
function animateOnVisible(entries, obs) {
  entries.forEach(e => {
    if (e.isIntersecting) {
      document.querySelectorAll('.fill').forEach(el => {
        el.style.strokeDashoffset = el.getAttribute('data-offset');
      });
      obs.disconnect();
    }
  });
}
new IntersectionObserver(animateOnVisible, { threshold: .3 })
  .observe(document.getElementById('satGrid'));

// ── tab switch (visual only) ──
document.querySelectorAll('.tab').forEach(t => {
  t.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(x => x.classList.remove('active'));
    t.classList.add('active');
  });
});
</script>
</body>
</html>
