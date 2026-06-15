# Gara Forio 2026 — Trofeo Open Ischia Big Game

App web (single-file, client-side) per la gara del 21 giugno 2026.

**Live:** https://marinovinc.github.io/GaraForio2026/

## Funzioni
- Mappa clorofilla satellitare (Copernicus Marine / NASA PACE) + SST + fondale EMODnet.
- **Interpretazione biologica**: finestra foraggio 0,20–0,50 mg/m³, rilevamento fronti
  (gradiente ≥ 0,20 mg/m³) e hot-spot per grandi pelagici.
- Campo gara, hotspot, secche, percorso ottimale animato.
- Isobate ≥200 m, vento e correnti orarie (Open-Meteo) sincronizzate con l'orologio di gara.

## Uso su cellulare
Aprire il link in Safari/Chrome → "Aggiungi a Home". Richiede connessione dati
(le sorgenti satellitari/meteo sono live). Per l'uso al largo senza segnale è
prevista la versione PWA con snapshot offline (vedi progetto IschiaFishing).

## Note
- Tutto client-side: nessun server, nessuna credenziale nel codice.
- Cartella `images/`: icone standard Leaflet (controllo layer + marker).
