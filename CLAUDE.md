# Calcolatore — guida per Claude

PWA in un solo file (`index.html`, ~840 righe, circa 20k token) che converte alimenti a parità di calorie e macro, con piano pasti, database alimenti e lista della spesa. Il backend è un Google Apps Script (`EXEC` in `index.html`). Non ci sono build, test né dipendenze.

## Regola n.1 per risparmiare token: non leggere `index.html` per intero

Prima trova la sezione con `grep -n`, poi leggi solo l'intervallo che serve (`Read` con offset e limit, oppure `sed -n A,Bp`).

Sezioni (marcate da `/* ---------- nome ---------- */`; cercale con `grep -n "^/\* ----------" index.html`):

| Marcatore | Contenuto | Funzioni principali |
|---|---|---|
| (riga ~15) `<style>` | CSS, ~200 righe | — |
| `icons` | SVG `IC`, `ico()` | — |
| `api` | chiamate al backend e coda offline | `call`, `flush`, `api`, `WRITES` |
| `state` | stato `S`, `LS`, gruppi | `grp`, `thumb`, `GROUPS`, `HUE` |
| `loading` | caricamento dei dati | `load`, `banner`, `toast` |
| `sheet` | scheda dal basso, selettore | `openSheet`, `closeSheet`, `picker` |
| `converter` | tab Converti | `calcConv`, `viewConv`, `bindConv` |
| `foods` | tab Alimenti | `viewCibi`, `bindCibi`, `openFood` |
| `new food` | nuovo alimento, barcode, Open Food Facts | `openNuovo`, `scansiona`, `cercaNome`, `salvaNuovo` |
| `plan` | tab Piano pasti | `viewPiano`, `bindPiano`, `propri`, `pianoTot` |
| `shopping` | tab Spesa | `viewSpesa`, `bindSpesa` |
| `shell` | tab, render, tasto Indietro | `setTab`, `render` |

Convenzioni: ogni schermata ha la coppia `viewX()` (restituisce HTML) e `bindX(root)` (collega gli eventi). Usa `esc()` per il testo dinamico, `nf()` per i numeri e `api()` per le scritture (che passano dalla coda offline).

## Checklist per ogni modifica

1. Modifica `index.html` con `Edit` mirati, senza riscrivere il file.
2. **Aggiorna sempre la versione della cache** in `sw.js` (`const V = "calc-vN"` → N+1), altrimenti il telefono continua a mostrare la versione vecchia.
3. Controlla la sintassi senza aprire il browser: `node -e "new Function(require('fs').readFileSync('index.html','utf8').split('<script>')[1].split('</script>')[0])"`.
4. Scrivi il messaggio di commit in italiano, una riga, nello stile "Sezione: cosa cambia".

## Quanto impegno e quale agente usare

Scegli il livello più basso che basta. Sali di livello solo se il risultato non è buono.

| Tipo di richiesta | Esempi | Come procedere |
|---|---|---|
| **Banale** | testo, colore, etichetta, ordine di voci | Lavora direttamente: `grep`, poi un `Edit` mirato e il bump di `sw.js`. Niente agenti, niente piani. |
| **Media** | nuovo filtro, nuovo campo, bug circoscritto a una sezione | Lavora direttamente, leggendo solo la sezione interessata (più `state` e `api` se servono). |
| **Ricerca** | "dove viene calcolato X?", "quali funzioni usano Y?" | Agente `esploratore` (Haiku, sola lettura), che restituisce solo righe e conclusioni. |
| **Esecuzione meccanica** | rinominare ovunque, applicare la stessa modifica in più punti, già specificata | Agente `esecutore` (Sonnet) con istruzioni precise. |
| **Complessa** | nuova schermata, logica che tocca più sezioni, refactoring, bug difficile | Modello principale con ragionamento alto; per il disegno può servire l'agente `Plan`. Leggi le sezioni coinvolte una volta sola. |

Regole generali:
- Non rileggere un file appena modificato: `Edit` fallisce da solo se qualcosa non va.
- Non lanciare agenti per compiti che bastano 1–3 comandi: ogni agente parte da zero e deve ricostruire il contesto.
- Quando un agente serve, dagli il marcatore della sezione e i nomi delle funzioni presi da questa tabella, così non deve cercarli.
- Risposte brevi: riassumi cosa è cambiato, senza incollare il codice già scritto nel file.
