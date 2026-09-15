<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Projekt CANDY-AR — Demo</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jsQR/1.4.0/jsQR.min.js"></script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --factory-blue: #0a1628;
    --steel: #1e3a5f;
    --cyan: #00d4ff;
    --green: #00ff88;
    --amber: #ffb800;
    --red: #ff4040;
    --panel: rgba(0,20,50,0.88);
    --glass: rgba(0,212,255,0.10);
  }

  body {
    background: var(--factory-blue);
    font-family: 'Courier New', monospace;
    color: #e0f0ff;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── NAV ── */
  nav {
    display: flex; align-items: center; gap: 14px;
    padding: 12px 20px;
    background: rgba(0,0,0,0.5);
    border-bottom: 1px solid rgba(0,212,255,0.2);
    position: sticky; top: 0; z-index: 100;
  }
  .logo {
    font-size: 13px; font-weight: 700; letter-spacing: 2px;
    color: var(--cyan);
  }
  .logo span { color: #fff; }
  nav .tabs { display: flex; gap: 4px; margin-left: auto; }
  .tab {
    padding: 6px 14px; border-radius: 4px; cursor: pointer;
    font-size: 11px; letter-spacing: 1px; border: 1px solid transparent;
    transition: all .2s;
  }
  .tab.active { background: var(--cyan); color: #000; font-weight: 700; }
  .tab:not(.active) { border-color: rgba(0,212,255,0.3); color: var(--cyan); }
  .tab:not(.active):hover { background: rgba(0,212,255,0.1); }

  /* ── VIEWS ── */
  .view { display: none; }
  .view.active { display: block; }

  /* ══════════════════════════════
     AR VIEW
  ══════════════════════════════ */
  #ar-view {
    display: none;
    flex-direction: column;
    align-items: center;
    min-height: calc(100vh - 50px);
    padding: 20px;
  }
  #ar-view.active { display: flex; }

  .ar-stage {
    position: relative;
    width: 100%; max-width: 700px;
    aspect-ratio: 16/9;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid rgba(0,212,255,0.3);
    background: #000;
  }

  /* Fake camera feed — industrial background */
  .cam-bg {
    position: absolute; inset: 0;
    background:
      linear-gradient(180deg, #0d1f35 0%, #162b44 60%, #0a1628 100%);
    overflow: hidden;
  }

  /* Animated grid floor */
  .cam-bg::before {
    content:'';
    position: absolute; inset: 0;
    background-image:
      linear-gradient(rgba(0,212,255,0.06) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,255,0.06) 1px, transparent 1px);
    background-size: 40px 40px;
    animation: gridScroll 8s linear infinite;
  }
  @keyframes gridScroll {
    from { background-position: 0 0; }
    to   { background-position: 0 40px; }
  }

  /* Machinery illustration via CSS */
  .machine {
    position: absolute; bottom: 0; left: 50%;
    transform: translateX(-50%);
    width: 320px; height: 180px;
    background: linear-gradient(180deg, #1a3a5c 0%, #0e2640 100%);
    border-top: 3px solid #2a5080;
    border-radius: 8px 8px 0 0;
  }
  .machine::before {
    content: 'MODUL CPS-i40 · BONBON-FERT-03';
    position: absolute; top: 12px; left: 50%;
    transform: translateX(-50%);
    font-size: 9px; letter-spacing: 2px; color: rgba(0,212,255,0.5);
    white-space: nowrap;
  }
  .machine::after {
    content:'';
    position: absolute; top: 35px; left: 50%; transform: translateX(-50%);
    width: 260px; height: 4px;
    background: repeating-linear-gradient(90deg, #0e2640 0 8px, rgba(0,212,255,0.3) 8px 16px);
  }
  .conveyor {
    position: absolute; bottom: 28px; left: 10px; right: 10px; height: 12px;
    background: #0e2640;
    border: 1px solid #2a5080; border-radius: 2px;
  }
  .conveyor-belt {
    position: absolute; inset: 2px;
    background: repeating-linear-gradient(90deg, #1e4060 0 20px, #162b44 20px 40px);
    animation: beltMove 1.2s linear infinite;
  }
  @keyframes beltMove { from{background-position:0 0} to{background-position:40px 0} }

  /* QR Marker on machine */
  .marker-wrap {
    position: absolute; top: 55px; left: 50%; transform: translateX(-50%);
    width: 56px; height: 56px;
    border: 2px solid rgba(0,212,255,0.6);
    border-radius: 4px;
    display: flex; align-items: center; justify-content: center;
    background: rgba(255,255,255,0.9);
    cursor: pointer;
  }
  .marker-wrap svg { width: 44px; height: 44px; }

  /* Scanning animation */
  .scan-line {
    position: absolute; left: 0; right: 0; height: 2px;
    background: linear-gradient(90deg, transparent, var(--cyan), transparent);
    top: 0; animation: scanDown 2s ease-in-out infinite;
    pointer-events: none;
  }
  @keyframes scanDown { 0%{top:0;opacity:1} 90%{top:100%;opacity:1} 100%{top:100%;opacity:0} }

  /* Corner brackets */
  .bracket {
    position: absolute; width: 16px; height: 16px;
    border-color: var(--cyan); border-style: solid; border-width: 0;
  }
  .bracket.tl { top:-1px; left:-1px; border-top-width:2px; border-left-width:2px; }
  .bracket.tr { top:-1px; right:-1px; border-top-width:2px; border-right-width:2px; }
  .bracket.bl { bottom:-1px; left:-1px; border-bottom-width:2px; border-left-width:2px; }
  .bracket.br { bottom:-1px; right:-1px; border-bottom-width:2px; border-right-width:2px; }

  /* ── AR OVERLAY PANEL ── */
  .ar-overlay {
    position: absolute; top: 12px; right: 12px;
    width: 210px;
    background: var(--panel);
    border: 1px solid rgba(0,212,255,0.4);
    border-radius: 8px;
    backdrop-filter: blur(8px);
    padding: 10px;
    opacity: 0; pointer-events: none;
    transform: translateY(-6px);
    transition: opacity .4s, transform .4s;
  }
  .ar-overlay.visible { opacity: 1; pointer-events: all; transform: none; }

  .overlay-header {
    font-size: 9px; letter-spacing: 2px; color: var(--cyan);
    border-bottom: 1px solid rgba(0,212,255,0.2);
    padding-bottom: 5px; margin-bottom: 8px;
    display: flex; justify-content: space-between; align-items: center;
  }
  .status-dot {
    width: 7px; height: 7px; border-radius: 50%;
    background: var(--green);
    box-shadow: 0 0 6px var(--green);
    animation: blink 1.5s ease-in-out infinite;
  }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:.3} }

  .sensor-row {
    display: flex; justify-content: space-between; align-items: center;
    padding: 5px 0;
    border-bottom: 1px solid rgba(255,255,255,0.05);
    font-size: 11px;
  }
  .sensor-row:last-child { border-bottom: none; }
  .sensor-label { color: rgba(255,255,255,0.55); font-size: 10px; }
  .sensor-val { font-weight: 700; font-variant-numeric: tabular-nums; }
  .val-green  { color: var(--green); }
  .val-amber  { color: var(--amber); }
  .val-red    { color: var(--red); }
  .val-cyan   { color: var(--cyan); }

  .mini-bar {
    width: 100%; height: 3px; background: rgba(255,255,255,0.1);
    border-radius: 2px; margin-top: 3px;
  }
  .mini-bar-fill { height: 100%; border-radius: 2px; transition: width .6s; }

  /* AR connector line */
  .connector {
    position: absolute; top: 112px; right: 222px;
    width: 60px; height: 1px;
    background: linear-gradient(90deg, transparent, var(--cyan));
    opacity: 0; transition: opacity .4s .3s;
  }
  .connector.visible { opacity: 0.6; }

  /* Scan prompt */
  .scan-prompt {
    position: absolute; bottom: 14px; left: 50%; transform: translateX(-50%);
    font-size: 11px; letter-spacing: 1.5px; color: rgba(0,212,255,0.7);
    background: rgba(0,0,0,0.4); padding: 5px 14px; border-radius: 20px;
    white-space: nowrap; animation: promptPulse 2s ease-in-out infinite;
    pointer-events: none;
  }
  @keyframes promptPulse { 0%,100%{opacity:.7} 50%{opacity:1} }

  /* HUD elements */
  .hud-tl {
    position: absolute; top: 10px; left: 10px;
    font-size: 9px; letter-spacing: 1px; color: rgba(0,212,255,0.4);
    line-height: 1.8;
  }
  .hud-br {
    position: absolute; bottom: 10px; right: 10px;
    font-size: 9px; color: rgba(0,212,255,0.3); text-align: right;
  }

  /* Click-to-scan button */
  .btn-scan {
    margin-top: 14px;
    padding: 10px 28px;
    background: transparent;
    border: 1px solid var(--cyan);
    color: var(--cyan);
    font-family: 'Courier New', monospace;
    font-size: 12px; letter-spacing: 2px;
    border-radius: 4px; cursor: pointer;
    transition: all .2s;
  }
  .btn-scan:hover { background: rgba(0,212,255,0.1); }
  .btn-scan.scanning {
    background: rgba(0,255,136,0.1);
    border-color: var(--green); color: var(--green);
  }

  .ar-info {
    margin-top: 10px; max-width: 700px; width: 100%;
    font-size: 11px; color: rgba(255,255,255,0.4);
    text-align: center; line-height: 1.7;
  }
  .ar-info strong { color: rgba(0,212,255,0.7); }

  /* ══════════════════════════════
     ARCHITECTURE VIEW
  ══════════════════════════════ */
  #arch-view {
    padding: 24px 20px;
    min-height: calc(100vh - 50px);
  }
  #arch-view.active { display: block; }

  .arch-header {
    text-align: center; margin-bottom: 24px;
  }
  .arch-header h2 {
    font-size: 15px; letter-spacing: 3px; color: var(--cyan);
    margin-bottom: 4px;
  }
  .arch-header p { font-size: 11px; color: rgba(255,255,255,0.4); letter-spacing: 1px; }

  /* DIAGRAM */
  .diagram {
    max-width: 900px; margin: 0 auto;
    display: flex; flex-direction: column; gap: 0;
    position: relative;
  }

  .layer {
    display: flex; align-items: stretch; gap: 10px;
    position: relative; padding: 0 10px;
  }

  .layer-label {
    writing-mode: vertical-rl; text-orientation: mixed;
    transform: rotate(180deg);
    font-size: 9px; letter-spacing: 2px;
    color: rgba(255,255,255,0.25);
    padding: 10px 0;
    flex-shrink: 0; width: 18px;
    display: flex; align-items: center; justify-content: center;
  }

  .blocks {
    flex: 1; display: flex; gap: 10px; align-items: center;
    padding: 12px 0;
  }

  .block {
    flex: 1; border-radius: 8px; padding: 12px 10px;
    border: 1px solid;
    position: relative; text-align: center;
    min-height: 80px; display: flex; flex-direction: column;
    align-items: center; justify-content: center; gap: 5px;
  }

  .block-icon { font-size: 22px; line-height: 1; }
  .block-name { font-size: 11px; font-weight: 700; letter-spacing: .5px; }
  .block-sub  { font-size: 9px; color: rgba(255,255,255,0.45); line-height: 1.4; }

  /* Layer color themes */
  .b-physical { background: rgba(30,58,95,0.5); border-color: rgba(30,90,160,0.5); }
  .b-protocol { background: rgba(20,50,30,0.5); border-color: rgba(0,180,80,0.3); }
  .b-gateway  { background: rgba(50,30,80,0.5); border-color: rgba(140,80,220,0.4); }
  .b-cloud    { background: rgba(10,30,60,0.5); border-color: rgba(0,180,255,0.3); }
  .b-client   { background: rgba(50,40,10,0.5); border-color: rgba(220,160,0,0.4); }
  .b-highlight {
    background: rgba(0,212,255,0.12);
    border-color: var(--cyan);
    box-shadow: 0 0 16px rgba(0,212,255,0.15);
  }

  /* Arrow between layers */
  .arrow-row {
    display: flex; justify-content: center;
    padding: 2px 0;
    position: relative;
  }
  .arrow-col {
    flex: 1; display: flex; flex-direction: column; align-items: center;
    padding: 0 10px;
  }
  .arrow-line {
    width: 2px; height: 28px;
    background: linear-gradient(180deg, rgba(0,212,255,0.15), rgba(0,212,255,0.5));
  }
  .arrow-head {
    width: 0; height: 0;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    border-top: 7px solid rgba(0,212,255,0.5);
  }
  .arrow-label {
    position: absolute; left: 50%; transform: translateX(-50%);
    top: 4px;
    font-size: 9px; color: rgba(0,212,255,0.5);
    white-space: nowrap; letter-spacing: 1px;
    background: var(--factory-blue); padding: 1px 6px;
  }

  /* horizontal connector inside layer */
  .h-connector {
    display: flex; align-items: center; gap: 0;
    flex: 0 0 auto; color: rgba(0,212,255,0.4);
    font-size: 11px; padding: 0 2px;
  }

  /* Layer separator line */
  .layer-sep {
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(0,212,255,0.15), transparent);
    margin: 0 30px;
  }

  /* LEGEND */
  .legend {
    max-width: 900px; margin: 20px auto 0;
    display: grid; grid-template-columns: repeat(auto-fit, minmax(200px,1fr));
    gap: 8px;
  }
  .legend-item {
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 6px; padding: 9px 12px;
    font-size: 11px;
  }
  .legend-item strong { color: var(--cyan); display: block; margin-bottom: 2px; font-size: 10px; letter-spacing: 1px; }

  /* NOTES box */
  .notes {
    max-width: 900px; margin: 16px auto 0;
    background: rgba(0,212,255,0.05);
    border: 1px solid rgba(0,212,255,0.2);
    border-radius: 8px; padding: 14px 18px;
    font-size: 11px; line-height: 1.8;
    color: rgba(255,255,255,0.65);
  }
  .notes strong { color: var(--cyan); }

  /* Tag badges */
  .tag {
    display: inline-block; font-size: 8px; padding: 1px 5px;
    border-radius: 3px; vertical-align: middle; margin-left: 3px;
    font-weight: 700; letter-spacing: .5px;
  }
  .tag-now  { background: rgba(0,255,136,.15); color: var(--green); border: 1px solid rgba(0,255,136,.3); }
  .tag-plan { background: rgba(0,212,255,.15); color: var(--cyan);  border: 1px solid rgba(0,212,255,.3); }

</style>
</head>
<body>

<nav>
  <div class="logo">CANDY<span>-AR</span></div>
  <div style="font-size:10px;color:rgba(255,255,255,0.35);letter-spacing:1px">PROJEKT-DEMO · INNOVATIONS-BUDGET-MEETING</div>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('ar')">AR-DEMO</div>
    <div class="tab" onclick="switchTab('arch')">ARCHITEKTUR</div>
  </div>
</nav>

<!-- ══ AR VIEW ══ -->
<div id="ar-view" class="view active">
  <div class="ar-stage">
    <div class="cam-bg"></div>

    <!-- HUD -->
    <div class="hud-tl">
      CANDY-AR v0.1<br>
      MODUL: BONBON-FERT-03<br>
      STATUS: BEREIT
    </div>
    <div class="hud-br">
      DEMO-MODUS<br>STATISCHE DATEN
    </div>

    <!-- Machine -->
    <div class="machine">
      <div class="conveyor"><div class="conveyor-belt"></div></div>

      <!-- QR Marker -->
      <div class="marker-wrap" id="marker" onclick="triggerScan()">
        <div class="bracket tl"></div>
        <div class="bracket tr"></div>
        <div class="bracket bl"></div>
        <div class="bracket br"></div>
        <div class="scan-line"></div>
        <!-- Simple QR-like SVG pattern -->
        <svg viewBox="0 0 44 44" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect x="2" y="2" width="16" height="16" fill="#000" rx="1"/>
          <rect x="5" y="5" width="10" height="10" fill="#fff" rx="0.5"/>
          <rect x="7" y="7" width="6" height="6" fill="#000"/>
          <rect x="26" y="2" width="16" height="16" fill="#000" rx="1"/>
          <rect x="29" y="5" width="10" height="10" fill="#fff" rx="0.5"/>
          <rect x="31" y="7" width="6" height="6" fill="#000"/>
          <rect x="2" y="26" width="16" height="16" fill="#000" rx="1"/>
          <rect x="5" y="29" width="10" height="10" fill="#fff" rx="0.5"/>
          <rect x="7" y="31" width="6" height="6" fill="#000"/>
          <rect x="22" y="20" width="4" height="4" fill="#000"/>
          <rect x="28" y="20" width="4" height="4" fill="#000"/>
          <rect x="34" y="20" width="4" height="4" fill="#000"/>
          <rect x="20" y="24" width="4" height="4" fill="#000"/>
          <rect x="26" y="24" width="4" height="4" fill="#000"/>
          <rect x="32" y="24" width="4" height="4" fill="#000"/>
          <rect x="38" y="24" width="4" height="4" fill="#000"/>
          <rect x="22" y="28" width="4" height="4" fill="#000"/>
          <rect x="30" y="28" width="4" height="4" fill="#000"/>
          <rect x="38" y="28" width="4" height="4" fill="#000"/>
          <rect x="20" y="32" width="4" height="4" fill="#000"/>
          <rect x="28" y="32" width="4" height="4" fill="#000"/>
          <rect x="36" y="32" width="4" height="4" fill="#000"/>
          <rect x="24" y="36" width="4" height="4" fill="#000"/>
          <rect x="32" y="36" width="4" height="4" fill="#000"/>
          <rect x="38" y="36" width="4" height="4" fill="#000"/>
        </svg>
      </div>
    </div>

    <!-- AR Overlay Panel -->
    <div class="ar-overlay" id="arOverlay">
      <div class="overlay-header">
        <span>BONBON-FERT-03</span>
        <div class="status-dot"></div>
      </div>

      <div class="sensor-row">
        <div>
          <div class="sensor-label">Temperatur</div>
          <div class="mini-bar"><div class="mini-bar-fill" id="bar-temp" style="width:78%;background:var(--amber)"></div></div>
        </div>
        <div class="sensor-val val-amber" id="val-temp">156 °C</div>
      </div>

      <div class="sensor-row">
        <div>
          <div class="sensor-label">Bandgeschw.</div>
          <div class="mini-bar"><div class="mini-bar-fill" id="bar-speed" style="width:62%;background:var(--green)"></div></div>
        </div>
        <div class="sensor-val val-green" id="val-speed">1.2 m/s</div>
      </div>

      <div class="sensor-row">
        <div>
          <div class="sensor-label">Füllstand</div>
          <div class="mini-bar"><div class="mini-bar-fill" id="bar-fill" style="width:34%;background:var(--red)"></div></div>
        </div>
        <div class="sensor-val val-red" id="val-fill">34 %</div>
      </div>

      <div class="sensor-row">
        <div>
          <div class="sensor-label">Druck</div>
          <div class="mini-bar"><div class="mini-bar-fill" id="bar-pres" style="width:55%;background:var(--cyan)"></div></div>
        </div>
        <div class="sensor-val val-cyan" id="val-pres">2.1 bar</div>
      </div>

      <div class="sensor-row">
        <div>
          <div class="sensor-label">Fehlercode</div>
        </div>
        <div class="sensor-val val-green">OK</div>
      </div>

      <div style="margin-top:8px;font-size:9px;color:rgba(255,255,255,0.3);text-align:right;letter-spacing:1px">
        DEMO · STATISCHE DATEN
      </div>
    </div>

    <!-- Connector line marker → overlay -->
    <div class="connector" id="connector"></div>

    <div class="scan-prompt" id="scanPrompt">▶ MARKER ANTIPPEN ZUM SCANNEN</div>
  </div>

  <button class="btn-scan" id="btnScan" onclick="triggerScan()">MARKER SCANNEN</button>

  <div class="ar-info">
    <strong>Demo-Modus:</strong> Marker antippen oder Button klicken · Statische Demodaten · Läuft direkt im Browser<br>
    Für Live-Daten: Anbindung an CPS-i40 API via OPC-UA Gateway (siehe Architektur-Tab)
  </div>
</div>

<!-- ══ ARCHITECTURE VIEW ══ -->
<div id="arch-view" class="view">
  <div class="arch-header">
    <h2>SYSTEMARCHITEKTUR — CANDY-AR</h2>
    <p>Datenfluss von der Maschine bis zum Smartphone · CPS-i40-Modulbauweise</p>
  </div>

  <div class="diagram">

    <!-- LAYER 1: Physical -->
    <div class="layer">
      <div class="layer-label">MASCHINE</div>
      <div class="blocks">
        <div class="block b-physical">
          <div class="block-icon">⚙️</div>
          <div class="block-name">CPS-i40 Modul</div>
          <div class="block-sub">Bonbon-Fertigung<br>Sensoren integriert</div>
        </div>
        <div class="h-connector">—</div>
        <div class="block b-physical">
          <div class="block-icon">🌡️</div>
          <div class="block-name">Sensoren</div>
          <div class="block-sub">Temp · Druck · Speed<br>Füllstand · Fehlercodes</div>
        </div>
        <div class="h-connector">—</div>
        <div class="block b-physical">
          <div class="block-icon">🏷️</div>
          <div class="block-name">AR-Marker</div>
          <div class="block-sub">QR-Code / ArUco<br>an der Anlage montiert</div>
        </div>
      </div>
    </div>

    <div class="layer-sep"></div>

    <div class="arrow-row">
      <div style="flex:1;padding:0 28px;display:flex;gap:10px;">
        <div class="arrow-col" style="flex:1">
          <div class="arrow-line"></div>
          <div class="arrow-head"></div>
        </div>
        <div class="arrow-col" style="flex:1">
          <div class="arrow-line"></div>
          <div class="arrow-head"></div>
        </div>
        <div class="arrow-col" style="flex:1">
          <!-- marker is scanned, no data flow upward -->
        </div>
      </div>
      <div class="arrow-label">OPC-UA / MQTT  <span class="tag tag-now">HERSTELLERPROTOKOLL</span></div>
    </div>

    <!-- LAYER 2: Protocol / Edge -->
    <div class="layer">
      <div class="layer-label">EDGE</div>
      <div class="blocks">
        <div class="block b-protocol b-highlight">
          <div class="block-icon">🔌</div>
          <div class="block-name">OPC-UA Server</div>
          <div class="block-sub">Im CPS-i40 eingebaut<br>Datenpunkt-Abgriff hier<span class="tag tag-now">ABGRIFF</span></div>
        </div>
        <div class="h-connector">→</div>
        <div class="block b-gateway">
          <div class="block-icon">🖥️</div>
          <div class="block-name">Edge Gateway</div>
          <div class="block-sub">Lokaler Mini-PC<br>OPC-UA → REST/JSON<br>Normalisierung</div>
        </div>
        <div class="h-connector">→</div>
        <div class="block b-gateway">
          <div class="block-icon">🔒</div>
          <div class="block-name">Auth + TLS</div>
          <div class="block-sub">API-Key / JWT<br>HTTPS verschlüsselt<br>Kein Direktzugriff</div>
        </div>
      </div>
    </div>

    <div class="layer-sep"></div>

    <div class="arrow-row">
      <div style="flex:1;padding:0 28px;display:flex;gap:10px;">
        <div class="arrow-col" style="flex:1"></div>
        <div class="arrow-col" style="flex:1">
          <div class="arrow-line"></div>
          <div class="arrow-head"></div>
        </div>
        <div class="arrow-col" style="flex:1">
          <div class="arrow-line"></div>
          <div class="arrow-head"></div>
        </div>
      </div>
      <div class="arrow-label">HTTPS / REST-API  <span class="tag tag-plan">NÄCHSTER SCHRITT</span></div>
    </div>

    <!-- LAYER 3: Backend -->
    <div class="layer">
      <div class="layer-label">BACKEND</div>
      <div class="blocks">
        <div class="block b-cloud" style="opacity:.3">
          <div class="block-icon">📦</div>
          <div class="block-name">Historische Daten</div>
          <div class="block-sub">optional<br>Datenbank / Logging</div>
        </div>
        <div class="h-connector">—</div>
        <div class="block b-cloud b-highlight">
          <div class="block-icon">⚡</div>
          <div class="block-name">REST-API Server</div>
          <div class="block-sub">Node.js / Python<br>Holt Daten vom Gateway<br>Gibt JSON zurück</div>
        </div>
        <div class="h-connector">—</div>
        <div class="block b-cloud" style="opacity:.3">
          <div class="block-icon">🔔</div>
          <div class="block-name">Alerting</div>
          <div class="block-sub">optional<br>Push-Notification<br>bei Grenzwertübersch.</div>
        </div>
      </div>
    </div>

    <div class="layer-sep"></div>

    <div class="arrow-row">
      <div style="flex:1;padding:0 28px;display:flex;gap:10px;">
        <div class="arrow-col" style="flex:1"></div>
        <div class="arrow-col" style="flex:1">
          <div class="arrow-line"></div>
          <div class="arrow-head"></div>
        </div>
        <div class="arrow-col" style="flex:1"></div>
      </div>
      <div class="arrow-label">JSON via HTTPS  <span class="tag tag-plan">SMARTPHONE-ZUGRIFF</span></div>
    </div>

    <!-- LAYER 4: Client -->
    <div class="layer">
      <div class="layer-label">CLIENT</div>
      <div class="blocks">
        <div class="block b-client">
          <div class="block-icon">📷</div>
          <div class="block-name">Kamera</div>
          <div class="block-sub">Marker erkennen<br>WebRTC / getUserMedia</div>
        </div>
        <div class="h-connector">→</div>
        <div class="block b-client b-highlight">
          <div class="block-icon">📱</div>
          <div class="block-name">Web-AR App</div>
          <div class="block-sub">Browser (kein Install)<br>AR.js / jsQR<br>Overlay rendern</div>
        </div>
        <div class="h-connector">→</div>
        <div class="block b-client">
          <div class="block-icon">👁️</div>
          <div class="block-name">AR-Overlay</div>
          <div class="block-sub">Sensorwerte live<br>direkt auf dem Screen<br>über Marker-Bild</div>
        </div>
      </div>
    </div>

  </div><!-- /diagram -->

  <!-- LEGEND -->
  <div class="legend">
    <div class="legend-item">
      <strong>JETZT — DEMO</strong>
      AR-Mockup mit statischen Daten im Browser. Keine Hardware nötig. Zeigt die UX für das Meeting.
    </div>
    <div class="legend-item">
      <strong>SCHRITT 1</strong>
      OPC-UA Anbindung: Hersteller des CPS-i40 stellt OPC-UA Endpunkt bereit → dort greifen wir Daten ab.
    </div>
    <div class="legend-item">
      <strong>SCHRITT 2</strong>
      Edge Gateway + REST-API: kleiner Mini-PC in der Fertigung übersetzt OPC-UA → JSON per HTTPS.
    </div>
    <div class="legend-item">
      <strong>SCHRITT 3</strong>
      Web-AR App holt Daten per fetch() und rendert sie live über dem Marker-Bild auf dem Smartphone.
    </div>
  </div>

  <div class="notes">
    <strong>Sicherheit:</strong> Alle Verbindungen laufen über HTTPS/TLS. Das CPS-i40 ist nicht direkt aus dem Internet erreichbar — ausschließlich über das interne Edge Gateway mit API-Key-Authentifizierung.<br>
    <strong>Kosten:</strong> Consumer-Smartphone + Mini-PC (ca. 80–150 €) als Gateway + Open-Source-Stack (AR.js, Node.js). Keine proprietäre Industrie-Middleware nötig.<br>
    <strong>Warum OPC-UA?</strong> Der CPS-i40-Standard schreibt OPC-UA vor — das ist der offizielle Datenpunkt-Abgriff den der Hersteller vorgesehen hat. Kein Reverse Engineering nötig.
  </div>

</div><!-- /arch-view -->

<script>
  function switchTab(t) {
    document.querySelectorAll('.tab').forEach((el,i)=>{
      el.classList.toggle('active', (i===0&&t==='ar')||(i===1&&t==='arch'));
    });
    document.getElementById('ar-view').classList.toggle('active', t==='ar');
    document.getElementById('arch-view').classList.toggle('active', t==='arch');
  }

  let scanned = false;

  function triggerScan() {
    if (scanned) { resetScan(); return; }
    scanned = true;

    const btn = document.getElementById('btnScan');
    const prompt = document.getElementById('scanPrompt');
    btn.classList.add('scanning');
    btn.textContent = 'SCANNEN… ▌';

    // Simulate scan delay
    setTimeout(() => {
      btn.textContent = '✓ ERKANNT — TIPPEN ZUM ZURÜCKSETZEN';
      prompt.style.display = 'none';
      document.getElementById('arOverlay').classList.add('visible');
      document.getElementById('connector').classList.add('visible');
      startLiveFlicker();
    }, 900);
  }

  function resetScan() {
    scanned = false;
    clearInterval(flickerInterval);
    document.getElementById('btnScan').classList.remove('scanning');
    document.getElementById('btnScan').textContent = 'MARKER SCANNEN';
    document.getElementById('scanPrompt').style.display = '';
    document.getElementById('arOverlay').classList.remove('visible');
    document.getElementById('connector').classList.remove('visible');
  }

  // Slight random flicker on sensor values to simulate live data
  let flickerInterval;
  const sensors = [
    { id:'val-temp',  base:156, unit:'°C', bar:'bar-temp', max:200, color:'amber', lo:140, hi:180 },
    { id:'val-speed', base:1.2, unit:'m/s', bar:'bar-speed', max:2, color:'green', lo:0.8, hi:1.8 },
    { id:'val-fill',  base:34,  unit:'%',   bar:'bar-fill', max:100, color:'red',  lo:20,  hi:60 },
    { id:'val-pres',  base:2.1, unit:'bar', bar:'bar-pres', max:5,  color:'cyan',  lo:1.5, hi:3.5 },
  ];

  function startLiveFlicker() {
    flickerInterval = setInterval(() => {
      sensors.forEach(s => {
        const delta = (Math.random() - 0.5) * (s.base * 0.03);
        const val = Math.max(s.lo * 0.8, Math.min(s.hi * 1.1, s.base + delta));
        const pct = Math.min(100, (val / s.max) * 100);
        const el = document.getElementById(s.id);
        const bar = document.getElementById(s.bar);
        el.textContent = (Number.isInteger(s.base) ? Math.round(val) : val.toFixed(1)) + ' ' + s.unit;
        bar.style.width = pct + '%';
      });
    }, 1400);
  }
</script>
</body>
</html>
