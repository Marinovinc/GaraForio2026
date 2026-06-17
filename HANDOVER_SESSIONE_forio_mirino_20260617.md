# HANDOVER SESSIONE — App pesca (mirino) + Briefing/Cartine Forio

**Data:** 2026-06-17
**Ambito:** progetti personali pesca (SEPARATI dall'ERP) — 3 app PWA + briefing/cartine gara Forio
**Autore lavoro:** Claude (sessione assistita), revisione utente continua

> Questo documento è **onesto**: confessa gli errori commessi, dice dove siamo arrivati e come continuare.

---

## 1. DUE FILONI DI LAVORO

### Filone A — Funzione "mirino" (Fadenkreuz) su 3 app
App PWA su GitHub Pages (HTML+Leaflet single-file):
- **GaraOstia2026** (dir locale `D:\Dev\IschiaFishing`) → https://marinovinc.github.io/GaraOstia2026/
- **GaraForio2026** (clone `D:\Dev\_gf_work`) → https://marinovinc.github.io/GaraForio2026/
- **ClorofillaLive** (clone `D:\Dev\_cl_work`) → https://marinovinc.github.io/ClorofillaLive/

Aggiunto a tutte e tre (committato e live):
- **Mirino centrale** visibile (croce bianca + pallino magenta) stile BathymetryExplorer.
- **Coordinate del mirino** in formato nautico N/E gradi-minuti, simbolo **⌖ magenta**, box in basso-centro.
- **Mirino trascinabile** dal pallino (area presa 44px) → leggi un punto qualsiasi senza muovere la mappa.
- **Precisione continua**: decimali derivati dai **metri/pixel reali** (`ceil(log10(1852/mpp))`) — niente soglie a gradini.
- **Pannello a scomparsa** (toggle) + coordinate sempre visibili.

Commit chiave (GaraOstia): `e1481a4` → `fb6d1aa` → `fdee75f` → `072ac75` → `110826c` → `df2dacf` → `89230a1`. Equivalenti su GaraForio e ClorofillaLive.

### Filone B — Briefing e cartine gara Forio
Hub locale: `D:\Dev\Forio_2026`. Gara: **Open Big Game Forio d'Ischia, 21 giugno 2026**.

Risultato finale **pubblicato e funzionante sul telefono**:
- 📄 **Briefing equipaggio:** https://marinovinc.github.io/GaraForio2026/BRIEFING_FORIO_2026.html
- 🗺️ **Mappa percorso + panoramica:** https://marinovinc.github.io/GaraForio2026/percorso_traina_gara.html

---

## 2. ERRORI CONFESSATI (onestà brutale)

1. **PDF invece di HTML** per WhatsApp — assunzione mia sul formato, non richiesta. L'utente voleva HTML.
2. **Briefing paralleli sbagliati**: ho creato `BRIEFING_NUOVO_CAMPO_2026` (Ostia) e `BRIEFING_FORIO_2026` (Forio) **senza sapere** che esistevano già i briefing veri dell'utente (`Briefing_tattico_Roma_2026_*`, `Briefing_tattico_Forio_2026_*`, e l'attivo `Briefing_Forio_2026.html`).
3. **Cancellato i backup** di `mappa_punti_reali.html` dopo la verifica, su un file **ancora in iterazione** → al rollback ho dovuto ricostruire (è andata bene solo perché avevo gli array dati intatti nel file). **Regola corretta:** tenere i backup finché l'utente non conferma.
4. **Direzione sbagliata sulla cartina** (zone aggregate + ricolorazione isobate) → "stai sbagliando tutto" → **rollback completo** di `Briefing_Forio_2026.html` (da `.BAK`) e `mappa_punti_reali.html` (edit inversi).
5. **Diagnosi tardiva** sulle isobate "mancanti": NON erano assenti (200m:16, 500m:32, 1000m:108, 2000m:28 punti nel campo) — erano **invisibili** perché blu su fondale blu.
6. **Link `file:///D:/...`** → non funzionano sul telefono (il cellulare non ha il disco D:). Risolto pubblicando su web (https).
7. **FS Windows case-insensitive**: copiando `Briefing_Forio_2026.html` ho **sovrascritto** il vecchio `BRIEFING_FORIO_2026.html` tracciato da git (di fatto un bene: rimpiazza il doc sbagliato).
8. **Stima errata "7 mappe interattive"** — era **1 sola** (un iframe); le altre 6 erano **SVG statici** già offline.
9. Conteggio alalunga: detto 8, erano **7**.

---

## 3. STATO ATTUALE (preciso, file per file)

### Live e funzionanti
| Cosa | URL / file | Stato |
|---|---|---|
| 3 app con mirino | github.io GaraOstia2026 / GaraForio2026 / ClorofillaLive | ✅ live |
| Briefing Forio | github.io/GaraForio2026/BRIEFING_FORIO_2026.html | ✅ live, mappa via URL web |
| Mappa percorso | github.io/GaraForio2026/percorso_traina_gara.html | ✅ live, toggle pannello (mobile parte chiuso) |

### File locali chiave (`D:\Dev\Forio_2026`)
- `campo_gara_forio_v3.html` (392 KB) — **cartina pulita**: base SVG `Campo_Isobate_EMODnet_3` + catture reali + percorso, proiettati con affine (vedi doc tecnico). Self-contained (rilievo+isobate incorporati).
- `percorso_traina_gara.html` (429 KB) — mappa Leaflet 4-spot + **overlay "Panoramica"** (la cartina v3 inline + spiegazioni + card spot) + **toggle "Comandi"**.
- `mappa_punti_reali.html` (30 KB) — **rollback al pristine** (71 punti, isobate blu). Superato da campo_gara_forio_v3.
- `Briefing_Forio_2026.html` (in `Downloads`, 1269 KB) — briefing equipaggio reale; iframe mappa → URL web. Pubblicato come `BRIEFING_FORIO_2026.html`.

### Backup tenuti (da cancellare solo su OK utente)
- `C:\Users\marin\Downloads\Briefing_Forio_2026.html.BAK`
- `D:\Dev\Forio_2026\percorso_traina_gara.html.BAK`

### Loose ends / da decidere
- **Doc paralleli miei** ancora in giro: `BRIEFING_NUOVO_CAMPO_2026` (in repo IschiaFishing, commit `d66e359`) e i `Briefing_tattico_Roma_2026_14` / `Briefing_tattico_Forio_2026_13` che ho reso **offline (font incorporati)** ma **NON pubblicati né usati**. Decidere se tenerli, pubblicarli o eliminarli.
- **Ostia non allineata**: per Forio c'è il briefing+mappa pubblicati; per **Ostia** no equivalente pubblicato sullo stesso schema. Da fare se l'utente vuole.

---

## 4. COME CONTINUARE

1. **Conferma cartina/briefing Forio** dall'utente → poi **cancellare i .BAK**.
2. **Pulizia doc paralleli** (punto loose ends) — chiedere all'utente.
3. **Allineare Ostia** allo stesso schema (cartina v3 + briefing pubblicato), se richiesto.
4. Per modifiche alla cartina: editare **`campo_gara_forio_v3.html`** (o lo script di build) → re-inlinare nell'overlay di `percorso_traina_gara.html` → `cp` in `D:\Dev\_gf_work` → `git commit/push` → verificare URL live.
5. **Workflow obbligatorio**: LEGGERE → BACKUP → MODIFICARE → TESTARE; tenere i backup finché l'utente non conferma; verificare sempre col rendering reale (Playwright) e, per il telefono, sull'**URL live** (non file://).

---

## 5. SVILUPPI FUTURI POSSIBILI

**Cartina / dati**
- Aggiungere alla cartina v3 anche i **4 spot tattici** (R1+R4, Dosso, La Fossa, Canyon) come marker numerati statici, non solo le catture.
- **Isobate dentro il campo** come overlay vettoriale ad alto contrasto sul v3 (oggi il rilievo+isobate sono raster incorporati).
- **Legenda profondità** (scala colore) sulla cartina statica.
- Ricampionare/aggiornare i **waypoint GPS** (oggi 2017/2019) con catture recenti.

**Briefing**
- **Versione + data** e **sezione uso-app** sul briefing pubblicato (erano state fatte, poi rollback — riproporle pulite).
- **Allineare Ostia**: stessa cartina v3 + briefing pubblicato (oggi solo Forio).
- Pulsante "Briefing" dentro le app che apre il documento pubblicato.

**App / mirino**
- Portare il **mirino** anche nella mappa Leaflet `percorso_traina_gara.html` (oggi è solo nelle 3 PWA con modello pane/tile).
- **Offline reale** del briefing per la barca (oggi mappa e font richiedono internet): inlinare font + cartina statica, e per la mappa interattiva valutare cache tile.
- **Export GPX** del percorso di traina dalla pagina percorso.

**Infrastruttura**
- Repo dedicato (es. `Forio2026`) invece di appoggiarsi a GaraForio2026, per separare app-gara e documenti.
- Indice pubblicato (`indice_cartine_batimetriche.html`) come hub linkabile dal telefono.

---

## 6. DOCUMENTI COLLEGATI
- `DOCUMENTAZIONE_TECNICA_forio_mirino_20260617.md` — riferimento tecnico (metodi, dati, proiezione affine, sorgenti, troubleshooting).
- Guida app: `D:\Dev\_gf_work\guida.html` (aggiornata con i link ai documenti).

*ASD IschiaFishing — handover onesto di sessione. Progetti personali pesca, separati dall'ERP.*
