# DOCUMENTAZIONE TECNICA — App pesca (mirino) + Cartine/Briefing Forio

**Data:** 2026-06-17
**Ambito:** progetti personali pesca (separati dall'ERP). Riferimento per continuare il lavoro.

---

## 1. INFRASTRUTTURA E REPO

| App / Progetto | Dir locale | Repo GitHub | URL Pages |
|---|---|---|---|
| GaraOstia2026 | `D:\Dev\IschiaFishing` | Marinovinc/GaraOstia2026 | https://marinovinc.github.io/GaraOstia2026/ |
| GaraForio2026 | `D:\Dev\_gf_work` (clone) | Marinovinc/GaraForio2026 | https://marinovinc.github.io/GaraForio2026/ |
| ClorofillaLive | `D:\Dev\_cl_work` (clone) | Marinovinc/ClorofillaLive | https://marinovinc.github.io/ClorofillaLive/ |
| Hub cartine/briefing Forio | `D:\Dev\Forio_2026` | (no git; alcuni file pubblicati su GaraForio2026) | — |

- **Tipo app:** HTML single-file + Leaflet inline, PWA (manifest + service worker su GaraOstia).
- **Pubblicazione:** `cp` del file nel clone → `git add/commit/push` → GitHub Pages serve in ~1 min.
- **Identità git cloni:** `Marinovinc` / `marino@unitec.it` (impostata nei cloni `_gf_work` / `_cl_work`).
- **Verifica:** `curl -s -o /dev/null -w "%{http_code}" <url>` (atteso 200) + rendering reale con Playwright.

---

## 2. SORGENTI DATI E COLLEGAMENTI (no database — solo servizi/geodati)

| Fonte | Endpoint / file | Uso |
|---|---|---|
| **EMODnet Bathymetry WMS** | `https://ows.emodnet-bathymetry.eu/wms` layer `emodnet:mean_multicolour` | sfondo batimetria a colori nelle mappe Leaflet |
| **EMODnet Bathymetry WFS** | `https://ows.emodnet-bathymetry.eu/wfs` typeName `emodnet:contours` | isobate reali (200/500/1000/2000 m) — vedi `fetch_isobaths.py` |
| **ESRI World Imagery** | `https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}` | satellite di base |
| **Leaflet** | `https://unpkg.com/leaflet@1.9.4/dist/leaflet.{js,css}` | libreria mappa interattiva |
| **Google Fonts** | `https://fonts.googleapis.com/...` (Fraunces, Public Sans, IBM Plex Mono) | tipografia briefing (incorporabili in base64 per offline) |
| **Copernicus Marine / NASA** | (nelle 3 app PWA) WMTS clorofilla MED_L4 gapfree | layer clorofilla |
| **Waypoint GPS equipaggio** | catture 2017/2019 (file kmz origine; estratti in `_read_waypoints.py`) | punti reali (alalunga/tonno/spada/foraggio) nelle cartine Forio |
| **Vertici campo (Navionics)** | N/E/S/W (vedi §4) | perimetro ufficiale campo gara |

> **NON ci sono database** in questi progetti. I dati sono geodati/servizi web + file locali.

---

## 3. FUNZIONE "MIRINO" (3 app PWA) — metodi

Tutte e tre le app condividono lo stesso modello (file `index.html`):

- **Coordinate = mirino (centro mappa)** in formato nautico:
  ```js
  function fmtDM(v,isLat,dec){ const h=isLat?(v>=0?'N':'S'):(v>=0?'E':'O');
    const a=Math.abs(v),d=Math.floor(a),m=(a-d)*60; return h+' '+d+' '+m.toFixed(dec)+"'"; }
  ```
- **Precisione continua** dai metri/pixel reali (Mercatore, scala isotropa), misurati con `map.distance` su 64px:
  ```js
  function coordDecimals(){
    const c=map.getContainer().getBoundingClientRect(); if(!c.width) return 3;
    const y=c.height/2;
    const res=map.distance(map.containerPointToLatLng([c.width/2-32,y]),
                           map.containerPointToLatLng([c.width/2+32,y]))/64; // m/px
    if(!isFinite(res)||res<=0) return 3;
    return Math.max(1,Math.min(5,Math.ceil(Math.log10(1852/res)))); // 1'=1852 m
  }
  ```
- **Mirino trascinabile** (Pointer Events): grab `.ch-grab` 44px, `map.dragging.disable()` durante il drag, coordinate da `containerPointToLatLng`.
- **Aggiornamento**: `map.on('move zoom zoomend', ...)`.
- Doppio-tap = popup valore (clorofilla); pannello a scomparsa (toggle).

---

## 4. CARTINA `campo_gara_forio_v3.html` — proiezione affine lat/lon → SVG

**Base:** `html_originali/Campo_Isobate_EMODnet_3.html` = **SVG** `viewBox="0 0 1360 1300"`, nord in alto, rilievo+isobate come **immagine base64 incorporata** (self-contained, offline). I 4 vertici del campo sono a coordinate SVG note.

**Vertici (lat, lon → SVG x,y):**
- N: 40.762317, 13.652950 → (753, 117)
- E: 40.585900, 13.831033 → (1238, 751)
- S: 40.463167, 13.600667 → (611, 1191)
- W: 40.636117, 13.418833 → (116, 571)

**Proiezione:** affine ai minimi quadrati (numpy `lstsq`), `x = a·lon + b·lat + c`, `y = d·lon + e·lat + f`. **Errore di ricostruzione sui 4 vertici: max 0.4 px** su 1360 → praticamente esatta (north-up quasi-equirettangolare a questa scala).

**Elementi aggiunti** (SVG `<circle>`/`<polyline>` proiettati): catture reali per specie (alalunga `#39e27a`, tonno `#ff4d4d`, spada `#ff9f1a`, foraggio `#2bd6ff`), percorso di traina (linea gialla tratteggiata) con marker **inizio (verde) / fine (rosso)** — NON "S/E" per non confondere coi vertici S/E del campo — e legenda.

**Catture reali (specie ≠ generica):** 7 alalunga, 2 tonno, 1 spada (avvist.), 1 foraggio. Le ~60 "marche generiche" sono escluse dalla cartina pulita.

---

## 5. `percorso_traina_gara.html` — struttura

- **Mappa Leaflet** full-screen: ArcGIS + EMODnet WMS + isobate (ISOB array, geometria EMODnet WFS) + perimetro campo + **piano 4 spot** (`SPOTS`: RADUNO→R1+R4→Dosso Centro⚡→La Fossa→Canyon) con trasferimenti 21 kn e popup tattici (orario/fondale/target/why) + 71 waypoint (`PTS`).
- **Overlay "Panoramica & spiegazioni"** (iniettato): pulsante `#panoramaBtn` → modale `#ovl` con la **cartina v3 inline** (SVG estratto da `campo_gara_forio_v3.html`) + spiegazioni + card spot generate da `SPOTS`.
- **Toggle pannello** `#panelToggle` ("Comandi"): `#panel.collapsed{transform:translateY(-135%)}`; su mobile (`innerWidth<640`) parte **chiuso**.

---

## 6. SCRIPT E METODI RIUSABILI

- **`fetch_isobaths.py`** (in `D:\Dev\IschiaFishing`): scarica isobate EMODnet WFS (`emodnet:contours`) su bbox, filtra profondità, decima, scrive `isobate.js` (`[lon,lat]`). Riusabile cambiando `BBOX` e `TARGET`.
- **Build cartina v3** (script Python ad-hoc, vedi handover): legge base SVG, calcola affine dai 4 vertici (numpy), inietta catture+percorso+legenda prima di `</svg>`, scrive `campo_gara_forio_v3.html`.
- **Incorporare font** (offline): fetch CSS Google Fonts con UA Chrome → scarica woff2 → base64 → `@font-face` inline al posto dei `<link>`. (Applicato a `Briefing_tattico_Roma_2026_14` e `_Forio_2026_13`: 27 woff2 incorporati, 0 riferimenti Google.)
- **Mappa interattiva → immagine statica**: render Playwright (con internet, attesa tile) → screenshot JPEG → base64 → `<img data:...>` al posto dell'`<iframe>`. (Necessario per offline; usato e poi rollback-ato sul briefing.)
- **Render/verifica**: Playwright headless; per offline testare con `page.route('**/*', abort if http)` e contare le **richieste esterne = 0**.

---

## 7. TROUBLESHOOTING (lezioni della sessione)

- **Link che non funziona sul telefono** → era `file:///D:/...` (path locale PC). Soluzione: **pubblicare su web** (GitHub Pages, https). Tutte le risorse devono essere https o incorporate.
- **Isobate "mancanti"** → in realtà presenti ma **blu su fondale blu**. Per renderle visibili: colori caldi + "casing" scuro, OPPURE usare la base v3 (isobate bianche su rilievo). Diagnosi: contare i punti dentro il bbox campo per profondità.
- **Sideboard copre la mappa su mobile** → aggiungere toggle e far partire il pannello **chiuso** sotto i 640px.
- **FS Windows case-insensitive** → `cp Briefing_Forio_2026.html` sovrascrive `BRIEFING_FORIO_2026.html` (git li vede uguali). Attenzione ai nomi che differiscono solo per maiuscole.
- **Backup**: tenerli finché l'utente non conferma (NON cancellarli su file in iterazione, anche se "verificato").
- **Mappa slippy non è offline** in un singolo file: le tile si scaricano a richiesta. Per offline servono base raster incorporata (SVG v3) o tile pre-cache.

---

## 8. RIFERIMENTI INCROCIATI
- `HANDOVER_SESSIONE_forio_mirino_20260617.md` — narrazione sessione, errori, stato, come continuare, sviluppi futuri.
- `indice_cartine_batimetriche.html` — indice delle cartine batimetriche (base maps).
- Cartine base: `html_originali/Campo_Isobate_EMODnet_3.html` (scelta v3, "Consigliata"), `Campo_Fondale_Profondita.html`, `Campo_Batimetriche_percorsi.html`, `Campo_su_EMODnet_reale.html`.
- `fetch_isobaths.py` (in IschiaFishing) — fetch isobate EMODnet.

*ASD IschiaFishing — documento tecnico. Progetti personali pesca, separati dall'ERP.*
