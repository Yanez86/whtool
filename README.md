# Tool Magazzino

Quattro strumenti web per elaborare gli estratti Excel del magazzino. Sono pagine HTML autonome: l'elaborazione avviene interamente nel browser, nessun file viene inviato a un server.

| File | Cosa fa |
|---|---|
| `index.html` | Home con i quattro strumenti |
| `giacenze.html` | **Aggregatore giacenze** — un file di giacenza UdC diventa un riepilogo per articolo |
| `rotture.html` | **Report rotture / Lost & Found** — confronto tra due file, con delta e totali |
| `inventario.html` | **Inventario** — lista di spunta con barcode + foglio di controllo corridoi |
| `bordero.html` | **Spacchetta borderò** — divide i PDF scansionati in un file per viaggio, con OCR |

## Aggregatore giacenze

- Aggregazione di default: **Articolo · Descrizione Articolo · Stato Contabile · Data Scadenza**, con somma di **Quantità Stoccata** e conteggio UdC.
- Qualsiasi altra colonna del file può essere aggiunta come chiave di raggruppamento o come campo da sommare (checkbox con ricerca; i campi selezionati restano in cima alla lista).
- Riconosce da solo la riga di intestazione e le colonne numeriche.
- Export XLSX (filtri automatici, date in dd/mm/yyyy) oppure CSV con separatore `;`.
- Formati letti: `.xlsx`, `.xlsm`, `.xls`, `.csv` — viene elaborato il primo foglio.

## Report rotture / Lost & Found

- Si trascinano i due file insieme: lo strumento capisce da solo quale è la giacenza rotture e quale il lost & found (in caso di dubbio c'è il pulsante *Inverti i due file*).
- Confronto per articolo: rotture, found e delta (il delta è il minore fra i due), con totali e riepilogo articoli presenti in uno solo dei due file.
- Se le intestazioni attese non vengono trovate, ripiega sulle posizioni fisse delle colonne e lo segnala con un avviso.
- Export Excel del report.

## Inventario

Sostituisce le macro `GeneraInventario` e `GeneraFoglioControllo` del file `Inventario_Macro.xlsm`: si trascina l'estratto **Giacenza UdC** e si ottengono due documenti già impaginati, da stampare o salvare in PDF dal browser.

**Lista inventario**

- Filtri: Picking (default *solo VERO*) e intervallo Piano (default 010–019), modificabili a video.
- Lato calcolato dalla campata con la stessa regola della macro: `(campata \ 10)` pari → **DX**, dispari → **SX**.
- Ordinamento Scaffale → Lato → Campata → Piano, con banner *Scaffale N | Lato X* e cambio pagina a ogni blocco, intestazione ripetuta su ogni pagina.
- Barcode **Code 39** disegnati come grafica vettoriale: non serve installare il font *Free 3 of 9 Extended* sul PC che stampa.

**Foglio di controllo**

- Corridoi distinti divisi fra zona picking e zona stoccaggio, SX a sinistra e DX a destra, con colonna *Spuntatore* da riempire a mano e casella nera dove il lato non esiste.
- Campo *Nome inventario* compilabile prima di stampare; usa tutte le righe del file, senza i filtri della lista (come la macro).

La stampa esce in A4 verticale. Per il PDF: *Stampa → Salva come PDF*, lasciando attiva la stampa degli sfondi.

## Spacchetta borderò

Porting nel browser di `split_bordero.py`: pdf.js rende le pagine, Tesseract.js fa l'OCR della sola testata, pdf-lib scrive i PDF di output.

- Riconosce la testata *BORDERO' DI CARICO*, legge **Viaggio** e **Azienda**, e apre un pacchetto per ogni coppia; le pagine successive (DDT, mail, fogli a mano) finiscono nel pacchetto aperto.
- Solo i borderò del **magazzino di partenza** (default 3980 / SETTALA) aprono un pacchetto: quelli di altri magazzini valgono come allegati. Con `any` si accetta qualsiasi magazzino; se il magazzino non è leggibile si sceglie se trattarlo come borderò (default, con avviso) o come allegato.
- Sulle pagine borderò la testata viene riletta con sei varianti di dpi, segmentazione e micro-rotazione e i numeri vengono decisi a maggioranza; un numero di viaggio isolato che differisce di una cifra da uno molto più frequente viene corretto.
- Ogni pacchetto esce come `<AZIENDA> - <VIAGGIO>.pdf`; le pagine precedenti al primo borderò finiscono in `_NON_ASSEGNATE.pdf`. Download singoli o ZIP unico, più la mappa CSV pagina → pacchetto.
- Come nello script, ogni PDF di input fa storia a sé; la casella *Unisci pacchetti fra file diversi* accorpa lo stesso viaggio trovato in scansioni diverse.
- Il **livello testo OCR** è opzionale e spento di default: rende i PDF ricercabili scrivendo il testo trasparente sopra la scansione originale, ma richiede qualche secondo per pagina.

Differenze rispetto allo script Python: niente `ocrmypdf` (il livello testo usa sempre il metodo overlay) e niente deskew/rotazione automatica delle pagine. Il primo avvio scarica i dati lingua di Tesseract (circa 15 MB) da un CDN esterno: se la rete aziendale blocca i CDN il pannello di diagnostica lo segnala subito.

## Pubblicazione su GitHub Pages

1. Su GitHub crea un repository nuovo, ad esempio `tool-magazzino`.
2. Carica i cinque file HTML nella root: **Add file → Upload files → Commit changes**.
3. **Settings → Pages** → *Source*: **Deploy from a branch**, branch **main**, cartella **/ (root)** → **Save**.
4. Dopo circa un minuto è online su `https://<tuo-utente>.github.io/tool-magazzino/` — la home si apre da sola, i quattro tool sono raggiungibili dalla barra in alto.

Da riga di comando:

```bash
git init
git add index.html giacenze.html rotture.html inventario.html bordero.html README.md
git commit -m "Tool magazzino"
git branch -M main
git remote add origin https://github.com/<tuo-utente>/tool-magazzino.git
git push -u origin main
```

Per aggiornare un tool basta sostituire il file corrispondente: Pages si ridispiega da solo.

## Note tecniche

- Ogni pagina è autonoma (CSS incluso), quindi funziona anche aprendola con doppio clic dal PC o inviandola singolarmente a un collega. I link della barra in alto funzionano solo se i file stanno nella stessa cartella.
- Le librerie (SheetJS per Excel, pdf.js / pdf-lib / JSZip / Tesseract.js per i PDF) arrivano da cdnjs e jsDelivr: al primo caricamento serve internet.
- Tema chiaro e scuro automatici, in base alle impostazioni del sistema.
