# Mulini & Ponticelli — minigolf

Gioco di minigolf in italiano, tutto in un unico file: `index.html` (HTML, CSS e JavaScript inline, nessuna build, nessuna dipendenza oltre ai Google Fonts). Pubblicato su GitHub Pages: il file deve chiamarsi `index.html` e stare nella radice del repository.

Il gioco deve funzionare bene su PC, smartphone (verticale e orizzontale) e LIM (lavagna interattiva, schermi da 1920×1080 in su). Ogni modifica va verificata su tutti e tre.

## Regole di lavoro

- **Non toccare ciò che non serve.** Il gioco base, le 90 buche e la fisica vanno bene così: cambia solo ciò che la richiesta riguarda.
- Interfaccia e testi in italiano.
- Le modalità Libera e Highlander a 1 giocatore devono restare invariate quando si aggiungono funzioni (Laboratorio, editor, multigiocatore sono separati e opzionali).
- Dopo ogni modifica l'utente ricarica `index.html` su GitHub: ricordaglielo.

## Struttura del file

La prima riga di `index.html` è uno scheletro (`<!doctype html>`, `<meta charset>`, `<meta name=viewport …>`, stile di reset). Non toglierla: senza il meta viewport i telefoni mostrano la pagina da PC rimpicciolita. Il file termina con `</body></html>`.

Dentro, in ordine:
1. `<title>`, link ai Google Fonts (Bagel Fat One, Figtree, DM Mono), `<style>` con i token colore su `:root` (tema unico scuro).
2. HTML: HUD in alto (`.hud`), palco con canvas (`#stage`, `#cv`), barra di tiro in basso (`.dock`), menu (`#ovMenu`), carta punteggi (`#ovCard`), pannello Laboratorio (`aside#lab`), editor (`#edTools`, `aside#edSide`).
3. Un unico `<script>` in una IIFE con sezioni marcate da commenti `// ---------- nome ----------`.

### Sezioni del codice (in ordine)
- **Costanti e helper geometrici**: mondo 1000×600 unità, 100 unità = 1 m. `R()`, `ell()`, `arc()`, `P()` (rombo), `ring()`, `pip()`.
- **THEMES**: 5 ambienti (borgo, montagna, mare, giungla, deserto) con colori e stili di ostacoli, decorazioni, mulini, ostacoli mobili.
- **Circuiti**: `BORGO`, `MONTAGNA`, `MARE`, `GIUNGLA`, `DESERTO` (18 buche ciascuno; le prime 9 più facili) + `CIRCUITS`, che include `mie` (buche dell'utente dall'editor, getter su `mineDefs()`).
- **compile(def, theme)**: trasforma la definizione di una buca in segmenti di collisione, zone, ponti ecc.
- **Audio**: sintetizzato con Web Audio (`tone`, `noise`, `sfx.*`), nessun file audio.
- **Stato**: `cfg` (impostazioni salvate in `localStorage` `mp-cfg2`), `G` (stato di gioco), `PH` (parametri fisici; fissi tranne che nel Laboratorio).
- **Fisica** (`physics(dt)`, passo fisso 1/240 s): attrito di rotolamento, pendenze/colline/rampe, rimbalzi su segmenti e cerchi, mulini e ostacoli mobili, salti, acqua/burroni/sabbie mobili, tubi, cattura in buca.
- **predict()**: colpo guidato; esegue la fisica vera su una copia della pallina con il flag `PRED`.
- **Flusso di gioco**: `shoot`, `afterRest`, `holed`, `nextHole`, `newGame`, `finish`, `gameOver`.
- **Multigiocatore** (stesso dispositivo, 1–6 giocatori): `G.players`, `curP()`, `useTurn`, `nextTurn`, `stashTurn`, classifica (`rankPlayers`, `finishMulti`, `multiOver`). Le palline degli altri sono vere (`cfg.realBalls`, urti con `ballHit`, simulate in `stepOthers` con il flag `OTHER`) oppure segnaposto. Chi tira per primo ruota a ogni buca.
- **Rendering**: livello statico pre-disegnato (`renderStatic`) + disegno dinamico (`draw`). In verticale il campo ruota di 90° (`view.rot`); usare sempre `toScreen`/`toWorld`.
- **Laboratorio**: materiali reali delle superfici (`MATS`), materiali della pallina (`BMATS`), sfera/disco, `applyLab()`.
- **Editor di buche**: strumenti (`TOOLS`), proprietà (`PROPS`), `docToDef()` converte il documento dell'editor in una buca normale. Buche salvate in `localStorage` `mp-myholes` (max 18).
- **Boot**: `start()`.

## Insidie note

- **Nomi di classe CSS in collisione**: le etichette dei gruppi del menu usano la classe `lab`; il pannello Laboratorio e l'editor usano `aside.lab`. Stilare sempre `aside.lab`, mai `.lab` da solo (aveva creato un riquadro grigio che copriva lo schermo del telefono).
- **Layout responsive**:
  - telefono in orizzontale (`orientation:landscape` e `max-height:520px`): HUD e barra di tiro diventano colonne laterali strette;
  - telefono in verticale (`max-width:620px`): HUD su due righe compatte;
  - LIM (`min-width:1600px` e `min-height:900px`): controlli ingranditi con `zoom:1.3`.
- **Test su telefono**: provare sempre una copia con la prima riga dello scheletro (meta viewport), altrimenti il browser simula una pagina da PC e i problemi da telefono non si vedono.
- **Test automatici nel browser**: se la finestra di anteprima è nascosta, `requestAnimationFrame` si ferma. Per simulare partite usare una copia di prova con un hook temporaneo (per esempio `window.__G=G` e una funzione che chiama `update(1/60)` in ciclo) e non lasciarlo mai nel file pubblicato.
- `[hidden]{display:none!important}` è nello stile: nascondere gli elementi con `el.hidden`.
- Le buche dell'editor e le impostazioni restano nel browser (`localStorage`): non sono nel file. Per spostarle si usano "Copia le mie buche" e "Importa" nell'editor.

## Verifica dopo le modifiche

1. Nessun errore in console.
2. Menu, partita, Laboratorio ed editor su: PC, telefono verticale (375×812), telefono orizzontale (812×375), LIM (1920×1080).
3. Se si toccano buche o fisica: ogni buca deve restare risolvibile (partenza e buca dentro il campo, fuori da acqua e burroni).
