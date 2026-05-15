<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="theme-color" content="#0f1923">
<title>RunStop</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
<style>
:root {
  --bg:#0f1923;--surface:#1a2535;--surface2:#243042;--border:rgba(255,255,255,0.08);
  --accent:#00e5a0;--accent2:#ff5c5c;--accent3:#ffc947;
  --text:#f0f4f8;--text2:#8a9bb0;--text3:#4a5a6e;
  --porta:#a855f7;--coffee:#f97316;--gym:#38bdf8;--constr:#fbbf24;
  --radius:14px;--radius-sm:8px;
}
*{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent;}
html,body{height:100%;overflow:hidden;background:var(--bg);font-family:'Syne',sans-serif;color:var(--text);}
#map{position:fixed;top:0;left:0;width:100%;height:100%;z-index:0;}
.leaflet-tile-pane{filter:brightness(0.82) saturate(0.65) hue-rotate(195deg);}
.leaflet-control-zoom,.leaflet-control-attribution{display:none!important;}

/* TOP BAR */
#topbar{position:absolute;top:0;left:0;right:0;z-index:500;background:linear-gradient(to bottom,rgba(15,25,35,0.97) 70%,transparent);padding:max(env(safe-area-inset-top),12px) 16px 14px;}
#topbar-inner{display:flex;align-items:center;justify-content:space-between;margin-bottom:10px;}
#app-name{font-size:20px;font-weight:800;letter-spacing:-0.5px;color:var(--accent);}
#app-name span{color:var(--text);font-weight:400;}
#status-pill{font-family:'DM Mono',monospace;font-size:11px;color:var(--text2);background:var(--surface);border:1px solid var(--border);padding:5px 10px;border-radius:999px;max-width:210px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;cursor:pointer;}
#layer-chips{display:flex;gap:6px;overflow-x:auto;scrollbar-width:none;-webkit-overflow-scrolling:touch;}
#layer-chips::-webkit-scrollbar{display:none;}
.chip{display:flex;align-items:center;gap:5px;padding:5px 11px;border-radius:999px;border:1px solid var(--border);background:var(--surface);font-size:12px;font-weight:600;color:var(--text2);white-space:nowrap;cursor:pointer;flex-shrink:0;letter-spacing:0.2px;}
.chip.on{border-color:currentColor;}
.chip-dot{width:8px;height:8px;border-radius:50%;background:currentColor;flex-shrink:0;}
.chip[data-layer="coffee"].on{color:var(--coffee);}
.chip[data-layer="gym"].on{color:var(--gym);}
.chip[data-layer="osm"].on{color:var(--constr);}
.chip[data-layer="porta"].on{color:var(--porta);}
.chip[data-layer="routes"].on{color:var(--accent);}

/* MAP CONTROLS */
#map-controls{position:absolute;right:14px;top:50%;transform:translateY(-50%);z-index:400;display:flex;flex-direction:column;gap:8px;}
.map-ctrl-btn{width:44px;height:44px;background:var(--surface);border:1px solid var(--border);border-radius:var(--radius-sm);display:flex;align-items:center;justify-content:center;cursor:pointer;color:var(--text);font-size:20px;font-weight:700;box-shadow:0 2px 8px rgba(0,0,0,0.4);}
.map-ctrl-btn:active{background:var(--surface2);transform:scale(0.94);}
#nearest-btn{color:var(--porta);font-size:16px;}
#locate-btn{color:var(--accent);font-size:18px;}

/* RUN HUD */
#run-hud{position:absolute;top:0;left:0;right:0;z-index:600;background:rgba(15,25,35,0.97);padding:max(env(safe-area-inset-top),14px) 20px 16px;display:none;border-bottom:1px solid var(--border);}
#hud-stats{display:flex;}
.hud-stat{flex:1;text-align:center;}
.hud-val{font-family:'DM Mono',monospace;font-size:28px;font-weight:500;color:var(--accent);line-height:1;}
.hud-lbl{font-size:10px;color:var(--text3);margin-top:3px;letter-spacing:0.8px;text-transform:uppercase;}
.hud-sep{width:1px;background:var(--border);margin:4px 0;}
#hud-btns{display:flex;gap:8px;margin-top:14px;}
#hud-pause,#hud-end{flex:1;padding:10px;border-radius:var(--radius-sm);border:none;font-family:'Syne',sans-serif;font-size:13px;font-weight:700;cursor:pointer;}
#hud-pause{background:var(--surface2);color:var(--text);}
#hud-end{background:var(--accent2);color:#fff;}

/* BUILD HUD */
#build-hud{position:absolute;top:0;left:0;right:0;z-index:600;background:rgba(15,25,35,0.97);padding:max(env(safe-area-inset-top),14px) 20px 14px;display:none;border-bottom:1px solid var(--border);}
#build-top{display:flex;align-items:center;justify-content:space-between;margin-bottom:10px;}
#build-lbl{font-size:13px;font-weight:700;color:var(--accent);letter-spacing:0.5px;text-transform:uppercase;}
#build-stats{display:flex;gap:16px;align-items:baseline;}
#build-dist{font-family:'DM Mono',monospace;font-size:24px;color:var(--text);}
#build-dist em{font-size:13px;color:var(--text2);font-style:normal;margin-left:3px;}
#build-elev{font-family:'DM Mono',monospace;font-size:14px;color:var(--accent3);}
#build-elev em{font-size:11px;color:var(--text3);font-style:normal;margin-left:2px;}
#build-btns{display:flex;gap:8px;}
#b-undo,#b-clear,#b-exit,#b-save{flex:1;padding:10px;border-radius:var(--radius-sm);border:none;font-family:'Syne',sans-serif;font-size:12px;font-weight:700;cursor:pointer;}
#b-undo{background:var(--surface2);color:var(--text2);}
#b-clear{background:var(--surface2);color:var(--accent2);}
#b-exit{background:var(--surface2);border:1px solid var(--accent2);color:var(--accent2);}
#b-save{background:var(--accent);color:var(--bg);}
#build-tip{font-size:11px;color:var(--text3);text-align:center;margin-top:10px;}

/* TOOLBAR */
#toolbar{position:absolute;bottom:0;left:0;right:0;z-index:500;padding:10px 12px max(env(safe-area-inset-bottom),14px);background:linear-gradient(to top,rgba(15,25,35,0.97) 70%,transparent);}
#toolbar-btns{display:flex;gap:7px;align-items:stretch;}
.tool-btn{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:3px;background:var(--surface);border:1px solid var(--border);border-radius:var(--radius-sm);padding:9px 4px;cursor:pointer;flex:1;color:var(--text2);font-size:10px;font-weight:600;letter-spacing:0.3px;text-transform:uppercase;}
.tool-btn svg{width:19px;height:19px;stroke:currentColor;fill:none;stroke-width:1.8;stroke-linecap:round;stroke-linejoin:round;flex-shrink:0;}
.tool-btn:active{transform:scale(0.95);}
#btn-run{flex:1.6;background:var(--accent2);color:#fff;border-color:var(--accent2);border-radius:var(--radius);font-size:11px;font-weight:800;}
#btn-run svg{stroke:#fff;}
#btn-build{flex:1.6;background:var(--accent);color:var(--bg);border-color:var(--accent);border-radius:var(--radius);font-size:11px;font-weight:800;}
#btn-build svg{stroke:var(--bg);}

/* PANELS */
.panel-backdrop{position:absolute;inset:0;z-index:800;background:rgba(0,0,0,0.5);display:none;align-items:flex-end;}
.panel-backdrop.open{display:flex;}
.panel{width:100%;background:var(--surface);border-radius:var(--radius) var(--radius) 0 0;padding:20px 20px max(env(safe-area-inset-bottom),20px);border-top:1px solid var(--border);max-height:85vh;overflow-y:auto;}
.ph{width:36px;height:4px;background:var(--border);border-radius:2px;margin:0 auto 18px;}
.panel h2{font-size:18px;font-weight:800;margin-bottom:4px;}
.panel-sub{font-size:13px;color:var(--text2);margin-bottom:18px;}
.fl{font-size:11px;font-weight:700;color:var(--text2);letter-spacing:0.8px;text-transform:uppercase;margin-bottom:6px;}
select,input[type=text]{width:100%;background:var(--bg);border:1px solid var(--border);border-radius:var(--radius-sm);color:var(--text);font-family:'Syne',sans-serif;font-size:15px;padding:12px 14px;margin-bottom:14px;appearance:none;}
select:focus,input:focus{outline:none;border-color:var(--accent);}
.br{display:flex;gap:10px;}
.btn{flex:1;padding:13px;border-radius:var(--radius-sm);border:none;font-family:'Syne',sans-serif;font-size:14px;font-weight:700;cursor:pointer;}
.btn-ghost{background:var(--surface2);color:var(--text2);}
.btn-accent{background:var(--accent);color:var(--bg);}
.btn-purple{background:var(--porta);color:#fff;}
.full-btn{width:100%;margin-top:8px;}
.summ{font-family:'DM Mono',monospace;font-size:13px;color:var(--accent);margin-bottom:16px;}

/* route list */
.rrow{display:flex;align-items:center;gap:12px;padding:13px 0;border-bottom:1px solid var(--border);}
.rrow:last-child{border-bottom:none;}
.rbar{width:4px;height:36px;border-radius:2px;flex-shrink:0;}
.rinfo{flex:1;min-width:0;}
.rname{font-size:15px;font-weight:700;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}
.rbadge{font-size:10px;color:var(--text3);background:var(--surface2);border-radius:4px;padding:1px 5px;margin-left:5px;font-weight:400;vertical-align:middle;}
.rmeta{font-family:'DM Mono',monospace;font-size:11px;color:var(--text2);margin-top:2px;}
.rbtns{display:flex;gap:6px;flex-shrink:0;}
.rbtns button{background:var(--surface2);border:1px solid var(--border);border-radius:6px;color:var(--text2);font-size:11px;padding:5px 10px;cursor:pointer;font-family:'Syne',sans-serif;font-weight:600;}
.rbtns button.del{color:var(--accent2);border-color:rgba(255,92,92,0.3);}
#routes-empty{font-size:13px;color:var(--text3);text-align:center;padding:24px 0;}

/* source */
.src-opt{display:flex;align-items:center;gap:12px;padding:13px 0;border-bottom:1px solid var(--border);cursor:pointer;}
.src-opt:last-of-type{border-bottom:none;}
.src-radio{width:18px;height:18px;border-radius:50%;border:2px solid var(--text3);flex-shrink:0;display:flex;align-items:center;justify-content:center;}
.src-radio.checked{border-color:var(--accent);}
.src-radio.checked::after{content:'';width:8px;height:8px;border-radius:50%;background:var(--accent);}
.src-name{font-size:15px;font-weight:600;}
.src-note{font-size:12px;color:var(--text2);}

/* nearest panel */
.near-row{display:flex;align-items:center;gap:12px;padding:11px 0;border-bottom:1px solid var(--border);}
.near-row:last-child{border-bottom:none;}
.near-icon{font-size:22px;flex-shrink:0;}
.near-info{flex:1;min-width:0;}
.near-name{font-size:14px;font-weight:700;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}
.near-dist{font-family:'DM Mono',monospace;font-size:11px;color:var(--accent);margin-top:2px;}
.near-meta{font-size:11px;color:var(--text2);}
.near-go{background:var(--surface2);border:1px solid var(--border);border-radius:6px;color:var(--accent);font-size:11px;padding:5px 10px;cursor:pointer;font-family:'Syne',sans-serif;font-weight:600;white-space:nowrap;}

/* elevation mini chart inside popup */
.elev-chart{width:100%;height:60px;margin-top:8px;}

/* leaflet dark popups */
.leaflet-popup-content-wrapper{background:var(--surface)!important;color:var(--text)!important;border-radius:12px!important;border:1px solid var(--border)!important;box-shadow:0 8px 32px rgba(0,0,0,0.5)!important;}
.leaflet-popup-tip{background:var(--surface)!important;}
.leaflet-popup-content{margin:14px 16px!important;font-family:'Syne',sans-serif!important;min-width:200px;}
.ptitle{font-size:15px;font-weight:700;color:var(--text);margin-bottom:3px;}
.pmeta{font-family:'DM Mono',monospace;font-size:11px;color:var(--text2);margin-bottom:8px;}
.pstale{font-size:11px;color:var(--accent3);margin-bottom:8px;}
.pbtns{display:flex;gap:6px;flex-wrap:wrap;}
.pbtns button{font-family:'Syne',sans-serif;font-size:12px;font-weight:600;padding:6px 12px;border-radius:6px;border:1px solid var(--border);background:var(--surface2);color:var(--text2);cursor:pointer;}
.pbtns button.gone{color:var(--accent2);border-color:rgba(255,92,92,0.3);}

body.pin-mode #map,body.build-mode #map{cursor:crosshair;}

/* trail tooltip */
.trail-tip{background:var(--surface)!important;color:var(--accent)!important;border:1px solid var(--border)!important;border-radius:6px!important;font-family:'Syne',sans-serif!important;font-size:12px!important;font-weight:600!important;padding:4px 8px!important;box-shadow:0 2px 8px rgba(0,0,0,0.4)!important;}

#float-hint{position:absolute;bottom:110px;left:50%;transform:translateX(-50%);z-index:600;padding:10px 20px;border-radius:999px;font-size:13px;font-weight:700;display:none;white-space:nowrap;letter-spacing:0.3px;pointer-events:none;}
#float-hint.pin{background:var(--porta);color:#fff;box-shadow:0 4px 20px rgba(168,85,247,0.4);}
#float-hint.build{background:var(--accent);color:var(--bg);box-shadow:0 4px 20px rgba(0,229,160,0.3);}
</style>
</head>
<body>
<div id="map"></div>

<!-- MAP CONTROLS -->
<div id="map-controls">
  <div class="map-ctrl-btn" onclick="map.zoomIn()">+</div>
  <div class="map-ctrl-btn" onclick="map.zoomOut()">−</div>
  <div class="map-ctrl-btn" id="nearest-btn" onclick="findNearest()" title="Nearest bathroom">🚽</div>
  <div class="map-ctrl-btn" id="locate-btn" onclick="locateMe()">◎</div>
</div>

<!-- TOP BAR -->
<div id="topbar">
  <div id="topbar-inner">
    <div id="app-name">Run<span>Stop</span></div>
    <div id="status-pill" onclick="loadBusinesses()" title="Tap to retry">Loading…</div>
  </div>
  <div id="layer-chips">
    <div class="chip on" data-layer="coffee" onclick="toggleLayer('coffee')"><span class="chip-dot"></span>Coffee</div>
    <div class="chip on" data-layer="gym" onclick="toggleLayer('gym')"><span class="chip-dot"></span>Gyms</div>
    <div class="chip on" data-layer="osm" onclick="toggleLayer('osm')"><span class="chip-dot"></span>Construction</div>
    <div class="chip on" data-layer="porta" onclick="toggleLayer('porta')"><span class="chip-dot"></span>My pins</div>
    <div class="chip on" data-layer="routes" onclick="toggleLayer('routes')"><span class="chip-dot"></span>Routes</div>
  </div>
</div>

<!-- RUN HUD -->
<div id="run-hud">
  <div id="hud-stats">
    <div class="hud-stat"><div class="hud-val" id="hud-dist">0.00</div><div class="hud-lbl">Miles</div></div>
    <div class="hud-sep"></div>
    <div class="hud-stat"><div class="hud-val" id="hud-time">0:00</div><div class="hud-lbl">Time</div></div>
    <div class="hud-sep"></div>
    <div class="hud-stat"><div class="hud-val" id="hud-pace">—</div><div class="hud-lbl">Pace/mi</div></div>
  </div>
  <div id="hud-btns">
    <button id="hud-pause" onclick="togglePause()">Pause</button>
    <button id="hud-end" onclick="endRun()">End run</button>
  </div>
</div>

<!-- BUILD HUD -->
<div id="build-hud">
  <div id="build-top">
    <div id="build-lbl">Building route</div>
    <div id="build-stats">
      <div id="build-dist">0.00<em>mi</em></div>
      <div id="build-elev">—<em>ft gain</em></div>
    </div>
  </div>
  <div id="build-btns">
    <button id="b-undo" onclick="bUndo()">↩ Undo</button>
    <button id="b-clear" onclick="bClear()">✕ Clear</button>
    <button id="b-exit" onclick="bExit()">Exit</button>
    <button id="b-save" onclick="bSave()">Save →</button>
  </div>
  <div id="build-tip">Tap to place waypoints · follows roads &amp; trails · drag to adjust</div>
</div>

<!-- TOOLBAR -->
<div id="toolbar">
  <div id="toolbar-btns">
    <div class="tool-btn" onclick="openPanel('add-pin-bd')">
      <svg viewBox="0 0 24 24"><path d="M12 5v14M5 12h14"/></svg>Pin
    </div>
    <div class="tool-btn" id="btn-run" onclick="handleRun()">
      <svg viewBox="0 0 24 24"><path d="M13 4a1 1 0 1 0 2 0 1 1 0 0 0-2 0M6 20l4-6 2 3 2-2 4 5M7 9l2 5 3-2 4-3"/></svg>Start run
    </div>
    <div class="tool-btn" id="btn-build" onclick="handleBuild()">
      <svg viewBox="0 0 24 24"><polyline points="4,20 8,12 14,16 18,8 22,10"/><circle cx="4" cy="20" r="2"/><circle cx="22" cy="10" r="2"/></svg>Build route
    </div>
    <div class="tool-btn" onclick="openRoutesPanel()">
      <svg viewBox="0 0 24 24"><path d="M3 17c3-3 5-5 8-5s5 2 8 2M3 7c3 3 5 5 8 5s5-2 8-2"/></svg>Routes
    </div>
  </div>
</div>

<div id="float-hint"></div>

<!-- ADD PIN -->
<div class="panel-backdrop" id="add-pin-bd" onclick="closeBd(event,'add-pin-bd')">
  <div class="panel">
    <div class="ph"></div><h2>Add a stop</h2>
    <p class="panel-sub">Pin a bathroom you've spotted on your runs.</p>
    <div class="fl">Type</div>
    <select id="pin-type">
      <option value="porta">🚽 Portapotty / construction site</option>
      <option value="public">🏛 Public restroom</option>
      <option value="other">📍 Other stop</option>
    </select>
    <div class="fl">Label (optional)</div>
    <input type="text" id="pin-label" placeholder="e.g. Cedar St construction"/>
    <div class="br">
      <button class="btn btn-ghost" onclick="closePanel('add-pin-bd')">Cancel</button>
      <button class="btn btn-purple" onclick="startPinMode()">Tap map to place</button>
    </div>
  </div>
</div>

<!-- NEAREST BATHROOM -->
<div class="panel-backdrop" id="nearest-bd" onclick="closeBd(event,'nearest-bd')">
  <div class="panel">
    <div class="ph"></div><h2>🚽 Nearest stops</h2>
    <p class="panel-sub">Closest bathroom options from your current location.</p>
    <div id="nearest-list"><div style="color:var(--text3);font-size:13px;text-align:center;padding:20px 0">Locating…</div></div>
    <button class="btn btn-ghost full-btn" onclick="closePanel('nearest-bd')">Close</button>
  </div>
</div>

<!-- SAVE RUN -->
<div class="panel-backdrop" id="save-run-bd">
  <div class="panel">
    <div class="ph"></div><h2>Save this run</h2>
    <div class="summ" id="run-summ"></div>
    <div class="fl">Route name</div>
    <input type="text" id="run-name" placeholder="e.g. River loop"/>
    <div class="br">
      <button class="btn btn-ghost" onclick="discardRun()">Discard</button>
      <button class="btn btn-accent" onclick="saveRun()">Save</button>
    </div>
  </div>
</div>

<!-- SAVE BUILT ROUTE -->
<div class="panel-backdrop" id="save-build-bd">
  <div class="panel">
    <div class="ph"></div><h2>Save built route</h2>
    <div class="summ" id="build-summ"></div>
    <div class="fl">Route name</div>
    <input type="text" id="build-name" placeholder="e.g. Brook Path loop"/>
    <div class="br">
      <button class="btn btn-ghost" onclick="closePanel('save-build-bd')">Keep editing</button>
      <button class="btn btn-accent" onclick="confirmSaveBuild()">Save</button>
    </div>
  </div>
</div>

<!-- ROUTES -->
<div class="panel-backdrop" id="routes-bd" onclick="closeBd(event,'routes-bd')">
  <div class="panel">
    <div class="ph"></div><h2>Saved routes</h2>
    <p class="panel-sub">Recorded runs and planned routes. Tap a route on the map to see elevation.</p>
    <div id="routes-list"></div>
    <div style="margin-top:16px;padding-top:14px;border-top:1px solid var(--border)">
      <div class="fl" style="margin-bottom:10px">📤 Share pins with your group</div>
      <div class="br" style="margin-bottom:8px">
        <button class="btn btn-ghost" onclick="exportPins()">Export pins</button>
        <button class="btn btn-ghost" onclick="openPanel('import-bd');closePanel('routes-bd')">Import pins</button>
      </div>
    </div>
    <button class="btn btn-ghost full-btn" onclick="closePanel('routes-bd')">Close</button>
    <button class="btn btn-ghost full-btn" style="margin-top:6px;font-size:12px;color:var(--text3)" onclick="openSrcPanel()">⚙ Data source</button>
  </div>
</div>

<!-- IMPORT PINS -->
<div class="panel-backdrop" id="import-bd" onclick="closeBd(event,'import-bd')">
  <div class="panel">
    <div class="ph"></div><h2>Import pins</h2>
    <p class="panel-sub">Paste the pin data your friend shared. Duplicates are skipped.</p>
    <div class="fl">Paste pin data here</div>
    <textarea id="import-text" style="width:100%;background:var(--bg);border:1px solid var(--border);border-radius:var(--radius-sm);color:var(--text);font-family:'DM Mono',monospace;font-size:12px;padding:12px;margin-bottom:14px;height:120px;resize:none;line-height:1.5" placeholder='Paste the "RunStop pins:" message here…'></textarea>
    <div id="import-status" style="font-size:12px;color:var(--accent);margin-bottom:12px;min-height:18px"></div>
    <div class="br">
      <button class="btn btn-ghost" onclick="closePanel('import-bd')">Cancel</button>
      <button class="btn btn-accent" onclick="importPins()">Merge pins</button>
    </div>
  </div>
</div>

<!-- SOURCE -->
<div class="panel-backdrop" id="source-bd" onclick="closeBd(event,'source-bd')">
  <div class="panel">
    <div class="ph"></div><h2>Data source</h2>
    <p class="panel-sub">Where to pull cafés and gyms from.</p>
    <div class="src-opt" onclick="setSrc('osm')"><div class="src-radio checked" id="r-osm"></div><div><div class="src-name">OpenStreetMap</div><div class="src-note">Free, no key needed</div></div></div>
    <div class="src-opt" onclick="setSrc('google')"><div class="src-radio" id="r-google"></div><div><div class="src-name">Google Places</div><div class="src-note">More accurate, needs API key</div></div></div>
    <div id="gkey-wrap" style="display:none;margin-top:8px">
      <div class="fl">API key</div>
      <input type="text" id="gkey" placeholder="Paste Google Places key…"/>
    </div>
    <button class="btn btn-accent full-btn" onclick="applySource()">Apply & reload</button>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
const GH_KEY = '698f0c3c-f8f9-4d80-9972-2f0843533e9f';

// ── STORAGE ───────────────────────────────────────────────────────────────────
// iOS Safari blocks localStorage for file:// URLs — fall back to in-memory
const _mem = {};
const ls = {
  get: k => {
    try { const v = localStorage.getItem(k); return v ? JSON.parse(v) : null; }
    catch(e) { return _mem[k] != null ? _mem[k] : null; }
  },
  set: (k,v) => {
    try { localStorage.setItem(k, JSON.stringify(v)); }
    catch(e) { _mem[k] = v; }
  }
};
let pins   = ls.get('rs_pins_v1')   || [];
let routes = ls.get('rs_routes_v1') || [];
let activeSrc = ls.get('rs_src_v1') || 'osm';

// ── MAP ───────────────────────────────────────────────────────────────────────
const CENTER=[42.2968,-71.2924], BBOX='42.26,-71.40,42.37,-71.17';
const map = L.map('map',{zoomControl:false,attributionControl:false}).setView(CENTER,13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:19}).addTo(map);
const lyr={
  coffee:L.layerGroup().addTo(map), gym:L.layerGroup().addTo(map),
  osm:L.layerGroup().addTo(map),    porta:L.layerGroup().addTo(map),
  routes:L.layerGroup().addTo(map), trails:L.layerGroup().addTo(map)
};

// POI store for nearest-bathroom search {lat,lng,name,type,hours}
let allPOI = [];

// ── FALLBACK DATA ─────────────────────────────────────────────────────────────
const FALLBACK_COFFEE=[
  {name:"Quebrada Baking Co.",lat:42.2967,lng:-71.2936,hours:"Mon–Sat 7am–6pm"},
  {name:"Starbucks (Wellesley)",lat:42.2929,lng:-71.2652,hours:"Mon–Fri 5:30am–8pm"},
  {name:"Caffe Nero (Wellesley Hills)",lat:42.3037,lng:-71.2734,hours:"Mon–Fri 6am–7pm"},
  {name:"Coffee Lab (Natick)",lat:42.2834,lng:-71.3473,hours:"Daily 6:30am–6pm"},
  {name:"Dunkin' (Needham)",lat:42.2796,lng:-71.2332,hours:"Daily 5am–9pm"},
  {name:"Starbucks (Newton)",lat:42.3295,lng:-71.2093,hours:"Daily 5:30am–9pm"},
  {name:"Peet's Coffee (Newton)",lat:42.3324,lng:-71.2088,hours:"Mon–Fri 6am–7pm"},
  {name:"Dunkin' (Wellesley)",lat:42.2872,lng:-71.2986,hours:"Daily 5am–10pm"},
];
const FALLBACK_GYMS=[
  {name:"Wellesley College Athletic Center",lat:42.2946,lng:-71.3066,hours:"Members only"},
  {name:"YMCA Greater Boston (Wellesley)",lat:42.2943,lng:-71.2681,hours:"Mon–Fri 5:30am–10pm"},
  {name:"Planet Fitness (Natick)",lat:42.2817,lng:-71.3494,hours:"24 hours"},
  {name:"Lifetime Fitness (Newton)",lat:42.3312,lng:-71.2018,hours:"Daily 5am–11pm"},
  {name:"Anytime Fitness (Needham)",lat:42.2804,lng:-71.2328,hours:"24 hours"},
  {name:"CrossFit Wellesley",lat:42.2975,lng:-71.2921,hours:"By class schedule"},
];

// ── UTILS ─────────────────────────────────────────────────────────────────────
const $=id=>document.getElementById(id);
function setStatus(m){$('status-pill').textContent=m;}
function daysAgo(ts){return Math.floor((Date.now()-ts)/86400000);}
function haversineM(a,b){
  const R=6371000,r=Math.PI/180,dLat=(b[0]-a[0])*r,dLng=(b[1]-a[1])*r;
  const x=Math.sin(dLat/2)**2+Math.cos(a[0]*r)*Math.cos(b[0]*r)*Math.sin(dLng/2)**2;
  return R*2*Math.atan2(Math.sqrt(x),Math.sqrt(1-x));
}
function totalMi(c){let d=0;for(let i=1;i<c.length;i++)d+=haversineM(c[i-1],c[i]);return d/1609.34;}
function fmtTime(ms){const s=Math.floor(ms/1000),m=Math.floor(s/60),h=Math.floor(m/60);return h>0?`${h}:${String(m%60).padStart(2,'0')}:${String(s%60).padStart(2,'0')}`:`${m}:${String(s%60).padStart(2,'0')}`;}
function fmtPace(mi,ms){if(mi<0.01)return'—';const mpm=(ms/1000/60)/mi;return`${Math.floor(mpm)}:${String(Math.round((mpm%1)*60)).padStart(2,'0')}`;}
function fmtFt(m){return Math.round(m*3.28084);}
function makeIcon(bg,emoji,sz=28){
  return L.divIcon({className:'',iconSize:[sz,sz],iconAnchor:[sz/2,sz/2],
    html:`<div style="width:${sz}px;height:${sz}px;background:${bg};border:2.5px solid rgba(255,255,255,0.9);border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:${Math.round(sz*0.48)}px;box-shadow:0 2px 10px rgba(0,0,0,0.5)">${emoji}</div>`});
}

// ── PANELS ────────────────────────────────────────────────────────────────────
function openPanel(id){$(id).classList.add('open');}
function closePanel(id){$(id).classList.remove('open');}
function closeBd(e,id){if(e.target===e.currentTarget)closePanel(id);}
function openSrcPanel(){closePanel('routes-bd');openPanel('source-bd');}

// ── LAYER TOGGLE ──────────────────────────────────────────────────────────────
function toggleLayer(name){
  const chip=document.querySelector(`.chip[data-layer="${name}"]`);
  if(map.hasLayer(lyr[name])){map.removeLayer(lyr[name]);chip.classList.remove('on');}
  else{map.addLayer(lyr[name]);chip.classList.add('on');}
}

// ── LOCATE ────────────────────────────────────────────────────────────────────
function locateMe(silent=false){
  if(!navigator.geolocation){if(!silent)setStatus('Geolocation unavailable.');return;}
  navigator.geolocation.getCurrentPosition(
    p=>{map.setView([p.coords.latitude,p.coords.longitude],15);if(!silent)setStatus('Centered on your location.');},
    ()=>{if(!silent)setStatus('Could not get location — check permissions.');}
  );
}

// ── NEAREST BATHROOM ─────────────────────────────────────────────────────────
function findNearest(){
  openPanel('nearest-bd');
  $('nearest-list').innerHTML='<div style="color:var(--text3);font-size:13px;text-align:center;padding:20px 0">Locating…</div>';
  if(!navigator.geolocation){$('nearest-list').innerHTML='<div style="color:var(--accent2);font-size:13px;padding:12px 0">Geolocation not available.</div>';return;}
  navigator.geolocation.getCurrentPosition(pos=>{
    const here=[pos.coords.latitude,pos.coords.longitude];
    // Collect everything: all POI + all active pins
    const candidates=[];
    allPOI.forEach(p=>candidates.push({lat:p.lat,lng:p.lng,name:p.name,emoji:p.type==='coffee'?'☕':p.type==='gym'?'🏋️':'🚧',meta:p.hours||'',type:p.type}));
    pins.filter(p=>(p.goneVotes||0)<2).forEach(p=>{
      const lbl=p.label||(p.type==='porta'?'Portapotty':p.type==='public'?'Public restroom':'Bathroom stop');
      candidates.push({lat:p.lat,lng:p.lng,name:lbl,emoji:'🚽',meta:'Your pin',type:'pin'});
    });
    if(!candidates.length){$('nearest-list').innerHTML='<div style="color:var(--text3);font-size:13px;padding:12px 0">No stops loaded yet — wait for map data to load.</div>';return;}
    // Sort by distance from current location
    candidates.forEach(c=>c.distM=haversineM(here,[c.lat,c.lng]));
    candidates.sort((a,b)=>a.distM-b.distM);
    const top=candidates.slice(0,8);
    $('nearest-list').innerHTML=top.map((c,i)=>{
      const distStr=c.distM<1000?`${Math.round(c.distM)} m`:`${(c.distM/1609.34).toFixed(1)} mi`;
      return `<div class="near-row">
        <div class="near-icon">${c.emoji}</div>
        <div class="near-info">
          <div class="near-name">${c.name}</div>
          <div class="near-dist">${distStr} away</div>
          <div class="near-meta">${c.meta}</div>
        </div>
        <button class="near-go" onclick="flyToNearest(${c.lat},${c.lng})">Go →</button>
      </div>`;
    }).join('');
  },()=>{$('nearest-list').innerHTML='<div style="color:var(--accent2);font-size:13px;padding:12px 0">Could not get location — check permissions.</div>';});
}
window.flyToNearest=(lat,lng)=>{map.setView([lat,lng],17);closePanel('nearest-bd');};

// ── OSM LOAD ──────────────────────────────────────────────────────────────────
async function loadOSM(q,layer,bg,emoji,poiType){
  try{
    const ctrl=new AbortController();
    const t=setTimeout(()=>ctrl.abort(),12000);
    const r=await fetch(`https://overpass-api.de/api/interpreter?data=${encodeURIComponent(q)}`,{signal:ctrl.signal});
    clearTimeout(t);
    if(!r.ok)return 0;
    const d=await r.json();let n=0;
    d.elements.forEach(el=>{
      const lat=el.lat||el.center?.lat,lng=el.lon||el.center?.lon;
      if(!lat||!lng)return;
      const name=el.tags?.name||el.tags?.amenity||el.tags?.leisure||'Unknown';
      const hrs=el.tags?.opening_hours||'';
      const m=L.marker([lat,lng],{icon:makeIcon(bg,emoji)});
      m.bindPopup(`<div class="ptitle">${name}</div><div class="pmeta">${hrs||'Hours unknown'}</div>`);
      layer.addLayer(m);
      allPOI.push({lat,lng,name,type:poiType,hours:hrs});
      n++;
    });
    return n;
  }catch(e){return 0;}
}

function loadFallback(data,layer,bg,emoji,poiType){
  data.forEach(p=>{
    const m=L.marker([p.lat,p.lng],{icon:makeIcon(bg,emoji)});
    m.bindPopup(`<div class="ptitle">${p.name}</div><div class="pmeta">${p.hours}</div><div class="pmeta" style="color:var(--accent3);margin-top:4px">📌 Curated — verify before your run</div>`);
    layer.addLayer(m);
    allPOI.push({lat:p.lat,lng:p.lng,name:p.name,type:poiType,hours:p.hours});
  });
  return data.length;
}

async function loadBusinesses(){
  lyr.coffee.clearLayers();lyr.gym.clearLayers();lyr.osm.clearLayers();
  allPOI=[];
  setStatus('Fetching data…');
  const [cc,gc,oc]=await Promise.all([
    loadOSM(`[out:json][timeout:12];(node["amenity"="cafe"](${BBOX});way["amenity"="cafe"](${BBOX}););out center body;`,lyr.coffee,'var(--coffee)','☕','coffee'),
    loadOSM(`[out:json][timeout:12];(node["leisure"="fitness_centre"](${BBOX});node["amenity"="gym"](${BBOX});way["leisure"="fitness_centre"](${BBOX}););out center body;`,lyr.gym,'var(--gym)','🏋️','gym'),
    loadOSM(`[out:json][timeout:12];(way["construction"](${BBOX});way["landuse"="construction"](${BBOX}););out center body;`,lyr.osm,'var(--constr)','🚧','constr')
  ]);
  const fc=cc===0?loadFallback(FALLBACK_COFFEE,lyr.coffee,'var(--coffee)','☕','coffee'):0;
  const fg=gc===0?loadFallback(FALLBACK_GYMS,lyr.gym,'var(--gym)','🏋️','gym'):0;
  const pinCount=pins.filter(p=>(p.goneVotes||0)<2).length;
  setStatus(`${cc+fc} cafés · ${gc+fg} gyms · ${oc} sites · ${pinCount} pins${(fc>0||fg>0)?' (tap to retry)':''}`);
}

// ── TRAILS ────────────────────────────────────────────────────────────────────
// Hardcoded Wellesley-area trail network — only the trails you actually run.
// Coordinates traced from official town trail maps.
const WELLESLEY_TRAILS = [
  { name: 'Brook Path', coords: [
    [42.3014,-71.2780],[42.3005,-71.2795],[42.2993,-71.2814],
    [42.2980,-71.2836],[42.2968,-71.2855],[42.2955,-71.2871],
    [42.2943,-71.2886],[42.2931,-71.2901],[42.2921,-71.2914]
  ]},
  { name: 'Cochituate Rail Trail', coords: [
    [42.2944,-71.3101],[42.2950,-71.3201],[42.2955,-71.3298],
    [42.2959,-71.3399],[42.2961,-71.3501],[42.2965,-71.3607],
    [42.2968,-71.3712],[42.2970,-71.3815],[42.2972,-71.3918]
  ]},
  { name: 'Centennial Reservation', coords: [
    [42.2968,-71.2835],[42.2975,-71.2821],[42.2983,-71.2802],
    [42.2993,-71.2788],[42.3005,-71.2784],[42.3014,-71.2789],
    [42.3021,-71.2801],[42.3018,-71.2815],[42.3009,-71.2828],
    [42.2998,-71.2838],[42.2988,-71.2845],[42.2978,-71.2840]
  ]},
  { name: 'Fuller Brook Park', coords: [
    [42.2881,-71.2962],[42.2871,-71.2971],[42.2861,-71.2975],
    [42.2851,-71.2970],[42.2843,-71.2960],[42.2840,-71.2948],
    [42.2845,-71.2937],[42.2855,-71.2930],[42.2866,-71.2931],
    [42.2875,-71.2940],[42.2881,-71.2952]
  ]},
  { name: 'Morses Pond', coords: [
    [42.3060,-71.2638],[42.3055,-71.2652],[42.3046,-71.2661],
    [42.3035,-71.2665],[42.3024,-71.2660],[42.3016,-71.2650],
    [42.3014,-71.2638],[42.3019,-71.2626],[42.3030,-71.2619],
    [42.3042,-71.2621],[42.3053,-71.2628],[42.3060,-71.2638]
  ]},
  { name: 'Rocky Narrows', coords: [
    [42.2654,-71.3210],[42.2648,-71.3198],[42.2640,-71.3183],
    [42.2635,-71.3168],[42.2633,-71.3150],[42.2638,-71.3135],
    [42.2648,-71.3125],[42.2660,-71.3122],[42.2670,-71.3130]
  ]},
  { name: 'Beebe Woods', coords: [
    [42.2724,-71.3098],[42.2715,-71.3085],[42.2708,-71.3070],
    [42.2705,-71.3052],[42.2710,-71.3036],[42.2722,-71.3028],
    [42.2735,-71.3032],[42.2743,-71.3045],[42.2741,-71.3062],
    [42.2732,-71.3075],[42.2724,-71.3083]
  ]},
];

function loadTrails() {
  lyr.trails.clearLayers();
  WELLESLEY_TRAILS.forEach(t => {
    // Dark shadow for contrast
    lyr.trails.addLayer(L.polyline(t.coords, {
      color:'#052e0f', weight:10, opacity:0.45, lineCap:'round', lineJoin:'round'
    }));
    // Bright green trail line
    const pl = L.polyline(t.coords, {
      color:'#39ff8a', weight:5, opacity:1, lineCap:'round', lineJoin:'round'
    });
    pl.bindPopup(`<div class="ptitle">🌲 ${t.name}</div><div class="pmeta">Trail</div>`);
    lyr.trails.addLayer(pl);
  });
}

// ── PINS ──────────────────────────────────────────────────────────────────────
function renderPins(){
  lyr.porta.clearLayers();
  pins.forEach((pin,idx)=>{
    if((pin.goneVotes||0)>=2)return;
    const age=daysAgo(pin.ts),stale=age>=30;
    const m=L.marker([pin.lat,pin.lng],{icon:makeIcon(stale?'#7c3aed':'var(--porta)','🚽',32)});
    const lbl=pin.label||(pin.type==='porta'?'Portapotty':pin.type==='public'?'Public restroom':'Bathroom stop');
    const s=stale?`<div class="pstale">⚠️ ${age} days old — still there?</div>`:'';
    m.bindPopup(`<div class="ptitle">${lbl}</div><div class="pmeta">${age===0?'Added today':'Added '+age+' day'+(age===1?'':'s')+' ago'} · ${pin.goneVotes||0} gone vote${(pin.goneVotes||0)===1?'':'s'}</div>${s}<div class="pbtns"><button class="gone" onclick="voteGone(${idx})">👎 Gone</button><button onclick="rmPin(${idx})">🗑 Remove</button></div>`);
    lyr.porta.addLayer(m);
  });
}
window.voteGone=idx=>{if(!pins[idx])return;pins[idx].goneVotes=(pins[idx].goneVotes||0)+1;ls.set('rs_pins_v1',pins);renderPins();map.closePopup();setStatus((pins[idx].goneVotes||0)>=2?'Pin removed — thanks!':'Gone vote recorded.');};
window.rmPin=idx=>{pins.splice(idx,1);ls.set('rs_pins_v1',pins);renderPins();map.closePopup();};

// ── PIN MODE ──────────────────────────────────────────────────────────────────
let pinMode=false,pType='porta',pLabel='';
function startPinMode(){
  pType=$('pin-type').value;pLabel=$('pin-label').value.trim();
  closePanel('add-pin-bd');pinMode=true;
  document.body.classList.add('pin-mode');
  showHint('📍 Tap map to place pin','pin');
}

// ── GRAPHHOPPER ROUTING + ELEVATION ──────────────────────────────────────────
async function ghRoute(from,to,withElevation=false){
  try{
    const elevParam=withElevation?'&elevation=true':'';
    const url=`https://graphhopper.com/api/1/route?point=${from.lat},${from.lng}&point=${to.lat},${to.lng}&profile=foot&locale=en&calc_points=true&points_encoded=false${elevParam}&key=${GH_KEY}`;
    const ctrl=new AbortController();
    const t=setTimeout(()=>ctrl.abort(),10000);
    const r=await fetch(url,{signal:ctrl.signal});
    clearTimeout(t);
    if(r.ok){
      const d=await r.json();
      if(d.paths&&d.paths[0]&&d.paths[0].points){
        const path=d.paths[0];
        const coords=path.points.coordinates.map(c=>[c[1],c[0]]);
        const elevGainM=withElevation?(path.ascend||0):0;
        if(coords.length>1)return{coords,elevGainM};
      }
    }
  }catch(e){console.warn('GH routing failed:',e.message);}
  return{coords:[[from.lat,from.lng],[to.lat,to.lng]],elevGainM:0};
}

// Fetch elevation profile for a full saved route (sample up to 20 points)
async function fetchElevProfile(coords){
  const sample=[];
  const step=Math.max(1,Math.floor(coords.length/20));
  for(let i=0;i<coords.length;i+=step)sample.push(coords[i]);
  if(sample[sample.length-1]!==coords[coords.length-1])sample.push(coords[coords.length-1]);
  try{
    const pts=sample.map(c=>`point=${c[0]},${c[1]}`).join('&');
    const url=`https://graphhopper.com/api/1/route?${pts}&profile=foot&points_encoded=false&elevation=true&key=${GH_KEY}`;
    const ctrl=new AbortController();
    const t=setTimeout(()=>ctrl.abort(),8000);
    const r=await fetch(url,{signal:ctrl.signal});
    clearTimeout(t);
    if(r.ok){
      const d=await r.json();
      if(d.paths&&d.paths[0]){
        const path=d.paths[0];
        const elevCoords=path.points.coordinates;// [lng,lat,ele]
        const elevs=elevCoords.map(c=>c[2]).filter(e=>e!=null);
        const gain=path.ascend||0;
        return{elevs,gainM:gain};
      }
    }
  }catch(e){}
  return{elevs:[],gainM:0};
}

// Draw a tiny SVG elevation profile
function drawElevChart(elevs){
  if(!elevs||elevs.length<2)return'';
  const w=220,h=55,pad=4;
  const mn=Math.min(...elevs),mx=Math.max(...elevs);
  const range=mx-mn||1;
  const pts=elevs.map((e,i)=>{
    const x=pad+((w-2*pad)*i/(elevs.length-1));
    const y=h-pad-((e-mn)/range)*(h-2*pad);
    return`${x.toFixed(1)},${y.toFixed(1)}`;
  }).join(' ');
  return`<svg viewBox="0 0 ${w} ${h}" style="width:100%;height:60px;margin-top:8px;border-radius:6px;background:rgba(0,0,0,0.2)">
    <polyline points="${pts}" fill="none" stroke="#00e5a0" stroke-width="2" stroke-linejoin="round"/>
    <polyline points="${pad},${h-pad} ${pts} ${w-pad},${h-pad}" fill="rgba(0,229,160,0.12)" stroke="none"/>
    <text x="${pad+2}" y="${h-pad-2}" fill="#8a9bb0" font-size="8" font-family="DM Mono,monospace">${Math.round(mn*3.28)}ft</text>
    <text x="${pad+2}" y="${pad+8}" fill="#8a9bb0" font-size="8" font-family="DM Mono,monospace">${Math.round(mx*3.28)}ft</text>
  </svg>`;
}

// ── BUILD MODE ────────────────────────────────────────────────────────────────
let buildActive=false,bWaypoints=[],bSegments=[],bSegLayers=[],bRouting=false;
let bTotalElevGainM=0;

function handleBuild(){
  if(runState!=='idle'){alert('Finish your run first.');return;}
  buildActive?bExit():bStart();
}

function bStart(){
  buildActive=true;bWaypoints=[];bSegments=[];bSegLayers=[];bRouting=false;bTotalElevGainM=0;
  $('build-hud').style.display='block';
  $('topbar').style.display='none';
  $('toolbar').style.display='none';
  document.body.classList.add('build-mode');
  showHint('Testing routing…','build');
  bTestRouting();
}

async function bTestRouting(){
  const res=await ghRoute({lat:42.2967,lng:-71.2936},{lat:42.2921,lng:-71.2876},false);
  if(res.coords.length>2){
    showHint('✦ Tap to place waypoints','build');
    setStatus('Route builder ready — follows roads & trails');
  }else{
    showHint('⚠️ Routing offline — straight lines only','build');
    setStatus('⚠️ GraphHopper unavailable');
  }
  bUpdateStats();
}

async function bAddWaypoint(ll){
  if(bRouting)return;
  const idx=bWaypoints.length,isFirst=idx===0;
  const sz=isFirst?22:16,bg=isFirst?'#27ae60':'#00e5a0';
  const icon=L.divIcon({className:'',iconSize:[sz,sz],iconAnchor:[sz/2,sz/2],
    html:`<div style="width:${sz}px;height:${sz}px;background:${bg};border:2.5px solid rgba(255,255,255,0.95);border-radius:50%;box-shadow:0 2px 10px rgba(0,0,0,0.5);cursor:grab;display:flex;align-items:center;justify-content:center;font-family:'DM Mono',monospace;font-size:${isFirst?9:8}px;color:rgba(255,255,255,0.9);font-weight:600">${isFirst?'S':idx}</div>`});
  const mk=L.marker([ll.lat,ll.lng],{icon,draggable:true,zIndexOffset:300}).addTo(map);
  mk.on('dragend',async()=>{bWaypoints[idx].ll=mk.getLatLng();await bRerouteAround(idx);bUpdateStats();});
  bWaypoints.push({ll,marker:mk});
  if(idx>0){
    bRouting=true;
    showHint('⟳ Routing…','build');
    const res=await ghRoute(bWaypoints[idx-1].ll,ll,true);
    const pl=L.polyline(res.coords,{color:'#00e5a0',weight:4,opacity:0.9}).addTo(map);
    bSegments.push({coords:res.coords,elevGainM:res.elevGainM});
    bSegLayers.push(pl);
    bRouting=false;
    showHint('✦ Tap to place waypoints','build');
  }
  bUpdateStats();
}

async function bRerouteAround(i){
  bRouting=true;
  if(i>0&&bSegLayers[i-1]){
    map.removeLayer(bSegLayers[i-1]);
    const res=await ghRoute(bWaypoints[i-1].ll,bWaypoints[i].ll,true);
    const pl=L.polyline(res.coords,{color:'#00e5a0',weight:4,opacity:0.9}).addTo(map);
    bSegments[i-1]={coords:res.coords,elevGainM:res.elevGainM};bSegLayers[i-1]=pl;
  }
  if(i<bWaypoints.length-1&&bSegLayers[i]){
    map.removeLayer(bSegLayers[i]);
    const res=await ghRoute(bWaypoints[i].ll,bWaypoints[i+1].ll,true);
    const pl=L.polyline(res.coords,{color:'#00e5a0',weight:4,opacity:0.9}).addTo(map);
    bSegments[i]={coords:res.coords,elevGainM:res.elevGainM};bSegLayers[i]=pl;
  }
  bRouting=false;
}

function bUpdateStats(){
  let totalMiles=0,totalElevM=0;
  bSegments.forEach(s=>{totalMiles+=totalMi(s.coords);totalElevM+=s.elevGainM||0;});
  $('build-dist').innerHTML=`${totalMiles.toFixed(2)}<em>mi</em>`;
  $('build-elev').innerHTML=totalElevM>0?`+${fmtFt(totalElevM)}<em>ft gain</em>`:`—<em>ft gain</em>`;
  bTotalElevGainM=totalElevM;
}

function bUndo(){
  if(!bWaypoints.length)return;
  const last=bWaypoints.pop();
  if(last.marker)map.removeLayer(last.marker);
  if(bSegLayers.length){const pl=bSegLayers.pop();if(pl)map.removeLayer(pl);bSegments.pop();}
  bUpdateStats();
}

function bClear(){
  bWaypoints.forEach(w=>{if(w.marker)map.removeLayer(w.marker);});
  bSegLayers.forEach(pl=>{if(pl)map.removeLayer(pl);});
  bWaypoints=[];bSegments=[];bSegLayers=[];bTotalElevGainM=0;bUpdateStats();
}

function bSave(){
  if(bWaypoints.length<2){setStatus('Add at least 2 waypoints first.');return;}
  const allCoords=bSegments.flatMap(s=>s.coords);
  let totalMiles=0;bSegments.forEach(s=>totalMiles+=totalMi(s.coords));
  const elevFt=bTotalElevGainM>0?` · +${fmtFt(bTotalElevGainM)}ft`:'';
  $('build-summ').textContent=`${totalMiles.toFixed(2)} mi${elevFt} · ${bWaypoints.length} waypoints`;
  $('build-name').value='';
  window._bCoords=allCoords;window._bMi=totalMiles;window._bElevM=bTotalElevGainM;
  openPanel('save-build-bd');
}

function bExit(){
  bClear();buildActive=false;bRouting=false;
  $('build-hud').style.display='none';
  $('topbar').style.display='block';
  $('toolbar').style.display='block';
  document.body.classList.remove('build-mode');
  hideHint();
}

function confirmSaveBuild(){
  const coords=window._bCoords,mi=window._bMi,elevM=window._bElevM||0;
  if(!coords||coords.length<2)return;
  const name=$('build-name').value.trim()||('Route '+new Date().toLocaleDateString('en-US',{month:'short',day:'numeric'}));
  const color=RCOLS[routes.length%RCOLS.length];
  routes.push({id:Date.now(),name,coords,distMi:mi,elevGainM:elevM,durationMs:0,ts:Date.now(),color,type:'built'});
  ls.set('rs_routes_v1',routes);renderRoutes();renderRoutesList();
  closePanel('save-build-bd');bExit();
  setStatus(`"${name}" saved — ${mi.toFixed(2)} mi · +${fmtFt(elevM)}ft`);
}

// ── MAP CLICK DISPATCHER ──────────────────────────────────────────────────────
map.on('click',e=>{
  if(pinMode){
    pins.push({lat:e.latlng.lat,lng:e.latlng.lng,type:pType,label:pLabel,ts:Date.now(),goneVotes:0});
    ls.set('rs_pins_v1',pins);renderPins();pinMode=false;
    document.body.classList.remove('pin-mode');hideHint();
    $('pin-label').value='';
    setStatus('Pin saved! Stays until 2 runners mark it gone.');
    return;
  }
  if(buildActive){bAddWaypoint(e.latlng);return;}
});

// ── RUN TRACKING ──────────────────────────────────────────────────────────────
let runState='idle',runCoords=[],runPoly=null,runDot=null;
let runT0=null,runPausedMs=0,runPauseStart=null,watchId=null,ticker=null,wakeLock=null;
async function reqWake(){if('wakeLock'in navigator){try{wakeLock=await navigator.wakeLock.request('screen');}catch(e){}}}
function freeWake(){if(wakeLock){wakeLock.release();wakeLock=null;}}
function hudUpdate(){
  const paused=runPauseStart?Date.now()-runPauseStart:0;
  const el=Date.now()-runT0-runPausedMs-paused,mi=totalMi(runCoords);
  $('hud-dist').textContent=mi.toFixed(2);
  $('hud-time').textContent=fmtTime(el);
  $('hud-pace').textContent=fmtPace(mi,el);
}
function handleRun(){if(buildActive){alert('Finish route build first.');return;}if(runState==='idle')startRun();else togglePause();}
function startRun(){
  if(!navigator.geolocation){alert('Geolocation not available.');return;}
  runState='running';runCoords=[];runT0=Date.now();runPausedMs=0;runPauseStart=null;
  $('run-hud').style.display='block';$('topbar').style.display='none';$('toolbar').style.display='none';
  runPoly=L.polyline([],{color:'#ff5c5c',weight:5,opacity:0.9}).addTo(map);
  reqWake();
  watchId=navigator.geolocation.watchPosition(pos=>{
    if(runState!=='running')return;
    const pt=[pos.coords.latitude,pos.coords.longitude];
    if(!runCoords.length){runDot=L.circleMarker(pt,{radius:8,color:'var(--accent)',fillColor:'var(--accent)',fillOpacity:1,weight:2.5}).addTo(map);map.setView(pt,16);}
    runCoords.push(pt);runPoly.setLatLngs(runCoords);hudUpdate();
    map.panTo(pt,{animate:true,duration:0.5});
  },err=>setStatus('GPS: '+err.message),{enableHighAccuracy:true,maximumAge:2000,timeout:15000});
  ticker=setInterval(hudUpdate,1000);
}
function togglePause(){
  if(runState==='running'){runState='paused';runPauseStart=Date.now();$('hud-pause').textContent='Resume';$('hud-pause').style.cssText='background:var(--accent);color:var(--bg)';}
  else if(runState==='paused'){runState='running';if(runPauseStart){runPausedMs+=Date.now()-runPauseStart;runPauseStart=null;}$('hud-pause').textContent='Pause';$('hud-pause').style.cssText='';}
}
function endRun(){
  clearInterval(ticker);if(watchId!==null){navigator.geolocation.clearWatch(watchId);watchId=null;}freeWake();
  const paused=runPauseStart?Date.now()-runPauseStart:0;
  const el=Date.now()-runT0-runPausedMs-paused,mi=totalMi(runCoords);
  runState='idle';
  $('run-hud').style.display='none';$('topbar').style.display='block';$('toolbar').style.display='block';
  $('hud-pause').textContent='Pause';$('hud-pause').style.cssText='';
  if(runPoly){map.removeLayer(runPoly);runPoly=null;}
  if(runDot){map.removeLayer(runDot);runDot=null;}
  if(runCoords.length<2){setStatus('Run ended — not enough GPS points.');return;}
  window._rc=runCoords.slice();window._rt=el;
  $('run-summ').textContent=`${mi.toFixed(2)} mi  ·  ${fmtTime(el)}  ·  ${fmtPace(mi,el)}/mi avg`;
  $('run-name').value='';openPanel('save-run-bd');
}
function discardRun(){window._rc=null;closePanel('save-run-bd');setStatus('Run discarded.');}
function saveRun(){
  const c=window._rc,t=window._rt;
  if(!c||c.length<2){closePanel('save-run-bd');return;}
  const name=$('run-name').value.trim()||('Run '+new Date().toLocaleDateString('en-US',{month:'short',day:'numeric'}));
  const mi=totalMi(c),color=RCOLS[routes.length%RCOLS.length];
  routes.push({id:Date.now(),name,coords:c,distMi:mi,elevGainM:0,durationMs:t,ts:Date.now(),color,type:'recorded'});
  ls.set('rs_routes_v1',routes);renderRoutes();renderRoutesList();
  closePanel('save-run-bd');window._rc=null;
  setStatus(`"${name}" saved — ${mi.toFixed(2)} mi.`);
}

// ── ROUTES ────────────────────────────────────────────────────────────────────
const RCOLS=['#00e5a0','#38bdf8','#f97316','#a855f7','#fbbf24','#ff5c5c','#34d399','#818cf8'];

function renderRoutes(){
  lyr.routes.clearLayers();
  routes.forEach(r=>{
    if(!r.coords||r.coords.length<2)return;
    const built=r.type==='built';
    const pl=L.polyline(r.coords,{color:r.color,weight:built?3:4,opacity:0.75,dashArray:built?'10 6':null});
    pl.on('click',()=>showRoutePopup(r,pl));
    L.circleMarker(r.coords[0],{radius:6,color:r.color,fillColor:r.color,fillOpacity:1,weight:2}).addTo(lyr.routes);
    lyr.routes.addLayer(pl);
  });
}

async function showRoutePopup(r,pl){
  const mi=r.distMi.toFixed(2);
  const elevFt=r.elevGainM?`+${fmtFt(r.elevGainM)}ft gain · `:'';
  const meta=r.durationMs?`${mi} mi · ${fmtTime(r.durationMs)} · ${fmtPace(r.distMi,r.durationMs)}/mi`:`${mi} mi · planned`;
  // Show popup with loading state for elevation chart
  const center=pl.getBounds().getCenter();
  const popup=L.popup({maxWidth:260,className:''})
    .setLatLng(center)
    .setContent(`<div class="ptitle">${r.name}</div><div class="pmeta">${meta}</div><div class="pmeta" style="color:var(--accent3)">${elevFt}tap chart to reload</div><div id="elev-loading" style="font-size:11px;color:var(--text3);margin-top:6px">Loading elevation…</div><div class="pbtns" style="margin-top:8px"><button onclick="flyTo(${r.id})">🗺 View</button><button class="gone" onclick="delRoute(${r.id})">🗑 Delete</button></div>`)
    .openOn(map);
  // Fetch elevation profile async and update popup
  const{elevs,gainM}=await fetchElevProfile(r.coords);
  const chart=drawElevChart(elevs);
  const gainStr=gainM?`+${fmtFt(gainM)}ft gain · `:'';
  if(map.hasLayer(popup)){
    popup.setContent(`<div class="ptitle">${r.name}</div><div class="pmeta">${meta}</div><div class="pmeta" style="color:var(--accent3)">${gainStr}elevation profile</div>${chart}<div class="pbtns" style="margin-top:8px"><button onclick="flyTo(${r.id})">🗺 View</button><button class="gone" onclick="delRoute(${r.id})">🗑 Delete</button></div>`);
  }
}

window.flyTo=id=>{const r=routes.find(x=>x.id===id);if(!r)return;map.fitBounds(L.polyline(r.coords).getBounds(),{padding:[40,40]});map.closePopup();};
window.delRoute=id=>{routes=routes.filter(x=>x.id!==id);ls.set('rs_routes_v1',routes);renderRoutes();renderRoutesList();map.closePopup();setStatus('Route deleted.');};
function openRoutesPanel(){renderRoutesList();openPanel('routes-bd');}
function renderRoutesList(){
  const el=$('routes-list');
  if(!routes.length){el.innerHTML='<div id="routes-empty">No routes yet.<br>Record a run or tap Build route.</div>';return;}
  el.innerHTML=routes.map(r=>{
    const badge=`<span class="rbadge">${r.type==='built'?'planned':'recorded'}</span>`;
    const elev=r.elevGainM?` · +${fmtFt(r.elevGainM)}ft`:'';
    const meta=r.durationMs?`${r.distMi.toFixed(2)} mi · ${fmtTime(r.durationMs)} · ${fmtPace(r.distMi,r.durationMs)}/mi`:`${r.distMi.toFixed(2)} mi`;
    return`<div class="rrow"><div class="rbar" style="background:${r.color}"></div><div class="rinfo"><div class="rname">${r.name}${badge}</div><div class="rmeta">${meta}${elev}</div></div><div class="rbtns"><button onclick="flyTo(${r.id});closePanel('routes-bd')">View</button><button class="del" onclick="delRoute(${r.id})">Del</button></div></div>`;
  }).join('');
}

// ── SOURCE ────────────────────────────────────────────────────────────────────
let selSrc=activeSrc;
function setSrc(s){selSrc=s;$('r-osm').className='src-radio'+(s==='osm'?' checked':'');$('r-google').className='src-radio'+(s==='google'?' checked':'');$('gkey-wrap').style.display=s==='google'?'block':'none';}
function applySource(){activeSrc=selSrc;ls.set('rs_src_v1',activeSrc);closePanel('source-bd');loadBusinesses();}

// ── HINT ──────────────────────────────────────────────────────────────────────
function showHint(txt,cls){const h=$('float-hint');h.textContent=txt;h.className=cls;h.style.display='block';}
function hideHint(){$('float-hint').style.display='none';}

// ── SHARE PINS ────────────────────────────────────────────────────────────────
function exportPins(){
  const active=pins.filter(p=>(p.goneVotes||0)<2);
  if(!active.length){setStatus('No active pins to export.');closePanel('routes-bd');return;}
  const payload='RunStop pins:'+btoa(JSON.stringify(active));
  if(navigator.clipboard&&navigator.clipboard.writeText){
    navigator.clipboard.writeText(payload).then(()=>{setStatus(`${active.length} pin${active.length===1?'':'s'} copied — paste in your group chat!`);closePanel('routes-bd');}).catch(()=>promptManualCopy(payload));
  }else{promptManualCopy(payload);}
}
function promptManualCopy(text){
  const ta=document.createElement('textarea');ta.value=text;
  ta.style.cssText='position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);width:80vw;height:100px;z-index:9999;background:var(--surface);color:var(--text);border:1px solid var(--accent);border-radius:8px;padding:10px;font-size:11px;font-family:monospace';
  document.body.appendChild(ta);ta.select();
  const note=document.createElement('div');note.textContent='Copy this text, then paste in your group chat:';
  note.style.cssText='position:fixed;top:calc(50% - 70px);left:50%;transform:translateX(-50%);z-index:9999;font-size:13px;font-weight:700;color:var(--text);background:var(--surface);padding:8px 14px;border-radius:8px;border:1px solid var(--border);white-space:nowrap';
  document.body.appendChild(note);
  const btn=document.createElement('button');btn.textContent='Done';
  btn.style.cssText='position:fixed;top:calc(50% + 60px);left:50%;transform:translateX(-50%);z-index:9999;background:var(--accent);color:var(--bg);border:none;padding:10px 28px;border-radius:8px;font-family:Syne,sans-serif;font-weight:700;font-size:14px;cursor:pointer';
  btn.onclick=()=>{ta.remove();note.remove();btn.remove();};document.body.appendChild(btn);
}
function importPins(){
  const raw=($('import-text').value||'').trim();
  const statusEl=$('import-status');
  if(!raw){statusEl.textContent='Paste something first.';statusEl.style.color='var(--accent2)';return;}
  try{
    const prefix='RunStop pins:';
    if(!raw.startsWith(prefix))throw new Error('Not RunStop data');
    const incoming=JSON.parse(atob(raw.slice(prefix.length)));
    if(!Array.isArray(incoming))throw new Error('Bad format');
    let added=0,skipped=0;
    incoming.forEach(p=>{
      if(!p.lat||!p.lng||!p.ts)return;
      const dup=pins.some(e=>Math.abs(e.lat-p.lat)<0.0002&&Math.abs(e.lng-p.lng)<0.0002&&Math.abs(e.ts-p.ts)<60000);
      if(dup){skipped++;return;}
      pins.push({...p,goneVotes:0});added++;
    });
    ls.set('rs_pins_v1',pins);renderPins();$('import-text').value='';
    if(added>0){statusEl.textContent=`✓ Added ${added} pin${added===1?'':'s'}${skipped?' ('+skipped+' already had)':''}.`;statusEl.style.color='var(--accent)';}
    else{statusEl.textContent=`All ${skipped} already on your map.`;statusEl.style.color='var(--accent3)';}
  }catch(e){statusEl.textContent='Could not read that data.';statusEl.style.color='var(--accent2)';}
}

// ── INIT ──────────────────────────────────────────────────────────────────────
window.onload = function() {
  try {
    renderPins();
    renderRoutes();
    renderRoutesList();
    setSrc(activeSrc);
    loadBusinesses();
    loadTrails();
    locateMe(true);
  } catch(e) {
    document.getElementById('status-pill').textContent = 'Error: ' + e.message;
  }
};
</script>
</body>
</html>
