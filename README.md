# Tool Magazzino

Sette strumenti web per elaborare gli estratti e i documenti del magazzino. Sono pagine HTML autonome: l'elaborazione avviene interamente nel browser, nessun file viene inviato a un server.

| File | Cosa fa |
|---|---|
| `index.html` | Home con i sette strumenti; Spacchetta borderò sta nella sezione *Obsoleti*, chiusa in fondo |
| `giacenze.html` | **Aggregatore giacenze** — un file di giacenza UdC diventa un riepilogo per articolo |
| `rotture.html` | **Report rotture / Lost & Found** — confronto tra due file, con delta e totali |
| `inventario.html` | **Inventario** — lista di spunta con barcode + foglio di controllo corridoi |
| `verifica-inventario.html` | **Verifica inventario** — confronto fisico/logico dell'export di inventario (Stock o Picking), con scheda Riconta stampabile |
| `bordero.html` | **Spacchetta borderò** — divide i PDF scansionati in un file per viaggio, con OCR |
| `etichette.html` | **Etichette packing list** — dalla giacenza UdC una etichetta A4 per ogni bancale |
| `confronto-cliente.html` | **Confronto giacenza cliente** — la nostra giacenza contro il tabulato inventario o la giacenza Bollicine del cliente |

Ogni tool porta la propria versione e la data di aggiornamento nel piè di pagina; lo storico è in `CHANGELOG.md`.

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

- Diviso per **area** (colonna *Area* della giacenza) e, dentro ogni area, fra zona picking e zona stoccaggio: corridoi con SX a sinistra e DX a destra, colonna *Spuntatore* da riempire a mano e casella nera dove il lato non esiste.
- Le aree senza corridoi e campate (BOMBOLE, COSTOSI, C04, P01, RIS…) stanno in fondo, in *Altre aree*: una riga per area e zona, con il numero di ubicazioni.
- Riquadro **Aree del foglio di controllo** per scegliere quali aree stampare. Le aree di ingresso e uscita (tutte quelle che iniziano per I o U: I01, U09_B, UCORSIA502…) partono escluse, perché a inventario devono essere vuote; se nel file hanno delle righe compare un avviso con l'elenco. In fondo al foglio, l'elenco delle aree escluse.
- Campo *Nome inventario* compilabile prima di stampare; usa tutte le righe del file, senza i filtri della lista (come la macro).

La stampa esce in A4 verticale, senza l'indirizzo della pagina e la data che il browser aggiunge nei margini. Per il PDF: *Stampa → Salva come PDF*, lasciando attiva la stampa degli sfondi.

## Verifica inventario

- Si carica l'export dell'attività di inventario (fisico e logico sulla stessa riga) e si sceglie il tipo: **Stock** o **Picking**.
- Per ogni ubicazione contata confronta **ubicazione**, **articolo** e **quantità in colli**; in **Stock** controlla anche l'**SSCC** (presente, di 18 cifre, non duplicato; se il file avesse un SSCC logico lo confronta). In **Picking** l'SSCC non si verifica.
- Opzioni, spente di default: *Mostra e confronta anche i pezzi* (solo Picking) e *…anche i lotti*. Filtro opzionale su picking (piano 010–019) o stoccaggio.
- Riepilogo con ubicazioni pari, difettose e non contate, difetti per tipo, delta fisico − logico (eccedenze, ammanchi) e delta per articolo.
- **Stampa scheda controllo**: foglio A4 "Scheda Riconta" con il nome inventario e le sole ubicazioni difettose (ubicazione, articolo, difetto, delta). **Esporta in Excel** salva le righe visibili con riepilogo e delta per articolo.

## Spacchetta borderò

Porting nel browser di `split_bordero.py`: pdf.js rende le pagine, Tesseract.js fa l'OCR della sola testata, pdf-lib scrive i PDF di output.

- Riconosce la testata *BORDERO' DI CARICO*, legge **Viaggio** e **Azienda**, e apre un pacchetto per ogni coppia; le pagine successive (DDT, mail, fogli a mano) finiscono nel pacchetto aperto.
- Solo i borderò del **magazzino di partenza** (default 3980 / SETTALA) aprono un pacchetto: quelli di altri magazzini valgono come allegati. Con `any` si accetta qualsiasi magazzino; se il magazzino non è leggibile si sceglie se trattarlo come borderò (default, con avviso) o come allegato.
- Il riconoscimento è tollerante a maiuscole e minuscole: l'OCR legge spesso `viaggio` o `azienda` in minuscolo e la pagina andrebbe persa. Se la testata ha i marcatori del borderò ma manca un numero, la pagina viene comunque riletta con le varianti prima di essere declassata ad allegato.
- Sulle pagine borderò la testata viene riletta con sei varianti di dpi, segmentazione e micro-rotazione e i numeri vengono decisi a maggioranza; un numero di viaggio isolato che differisce di una cifra da uno molto più frequente viene corretto.
- **La prima pagina di ogni PDF è sempre un borderò di partenza**: se l'OCR non la riconosce è un errore di lettura, non un allegato, quindi il pacchetto viene aperto lo stesso. I dati letti si tengono, quelli mancanti diventano `AZIENDA DA VERIFICARE` / `VIAGGIO DA VERIFICARE` nel nome del file, con un avviso in evidenza e la riga marcata `BORDERO_FORZATO` nel CSV. La casella si può togliere per tornare al comportamento dello script (pagine iniziali in `_NON_ASSEGNATE.pdf`).
- Ogni pacchetto esce come `<AZIENDA> - <VIAGGIO>.pdf`; le pagine precedenti al primo borderò finiscono in `_NON_ASSEGNATE.pdf`. Download singoli o ZIP unico, più la mappa CSV pagina → pacchetto.
- Come nello script, ogni PDF di input fa storia a sé; la casella *Unisci pacchetti fra file diversi* accorpa lo stesso viaggio trovato in scansioni diverse.
- L'OCR gira su più processi in parallelo (fino a quattro, secondo i core della macchina) e le sei varianti della rilettura partono insieme: su un PC normale l'analisi va circa tre volte più veloce della versione a un processo solo. La casella *Analisi rapida* salta la rilettura sulle testate già lette in modo completo (sul file di prova: 71 chiamate OCR invece di 119, stesso risultato).
- Il **livello testo OCR** è opzionale e spento di default: rende i PDF ricercabili scrivendo il testo trasparente sopra la scansione originale, ma richiede qualche secondo per pagina.

Differenze rispetto allo script Python: niente `ocrmypdf` (il livello testo usa sempre il metodo overlay) e niente deskew/rotazione automatica delle pagine. Il primo avvio scarica i dati lingua di Tesseract (circa 15 MB) da un CDN esterno: se la rete aziendale blocca i CDN il pannello di diagnostica lo segnala subito.

## Etichette packing list

- Si trascina l'estratto **Giacenza UdC**: esce una etichetta A4 (layout di `MODELLO.xlsx`) per ogni coppia **Area + Versione**, dove la versione è il numero bancale.
- Motivazione e codice dipendono dall'area, da una tabella modificabile a video (RITIRI COCA, ROTTURE 262, ROTTURE, BL5, BL3 precaricate); le aree senza motivazione vengono segnalate, con un pulsante per aggiungerle.
- Una riga per articolo con le quantità sommate; oltre 27 articoli le righe si abbassano per restare in una sola pagina.
- Data, codice plant e numero verbale compilabili prima di stampare; stampa o PDF dal browser, senza URL e data a piè di pagina.

## Confronto giacenza cliente

Si caricano la nostra **Giacenza UdC** e un file del cliente; il tool riconosce da solo quale dei due confronti fare, e lo dichiara in alto e nei file scaricati.

- **1 · INVENTARIO** — con il *tabulato inventario* del cliente (`s_codart`, `art_um`, `s_qtaxc`, `giacenza`, `conta click`, `diff`). Si scarica **il suo stesso file** con la colonna *conta click* compilata: viene modificato direttamente, quindi foglio, bordi, colonna colorata, filtro e formule `diff` restano come li ha mandati. In fondo si aggiungono, in rosso chiaro, gli articoli che abbiamo solo noi nel magazzino a cui si riferisce il tabulato (default 01 - VENDIBILE), con giacenza cliente 0; per ciascuno si sceglie a video l'U.M., che decide colli o pezzi e finisce in `art_um` (senza scelta: colli).
- **2 · BILANCIAMENTO CLICK - BOLLICINE** — con la *giacenza Bollicine* (anche `.xls`, una riga per articolo e magazzino `kmag`). Il confronto è per articolo e magazzino del cliente (01 - VENDIBILE, AS, NV) sulla colonna `giac`; si scarica un Excel nuovo con differenza, filtro automatico, righe rosse dove Bollicine ha 0 e noi abbiamo merce, e un secondo foglio con le impostazioni usate.

Regole, modificabili a video (si riparte dai valori predefiniti a ogni apertura):

- **Stato contabile → magazzino del cliente:** DISPONIBILE → 01 - VENDIBILE, SHORT → AS, tutti gli altri → NV.
- **U.M. del cliente → conteggio:** CT, CF, CS, PZ, PC a **colli** (*Quantità Stoccata*); BT, FS, BO, PL a **pezzi** (*Pezzi Stoccati*). Le U.M. sconosciute vanno a colli, con un avviso.
- Abbinamento sul campo `Articolo` della giacenza (non `Codice Articolo`), ignorando maiuscole e spazi. Differenza = cliente − nostra.
- Avvisi quando i pezzi per collo del cliente non coincidono con i nostri per un articolo contato a colli: di solito vuol dire U.M. da cambiare.

## Pubblicazione su GitHub Pages

1. Su GitHub crea un repository nuovo, ad esempio `tool-magazzino`.
2. Carica gli otto file HTML nella root (i `.md` sono facoltativi): **Add file → Upload files → Commit changes**.
3. **Settings → Pages** → *Source*: **Deploy from a branch**, branch **main**, cartella **/ (root)** → **Save**.
4. Dopo circa un minuto è online su `https://<tuo-utente>.github.io/tool-magazzino/` — la home si apre da sola, i sette tool sono raggiungibili dalla barra in alto.

Da riga di comando:

```bash
git init
git add index.html giacenze.html rotture.html inventario.html verifica-inventario.html bordero.html etichette.html confronto-cliente.html README.md
git commit -m "Tool magazzino"
git branch -M main
git remote add origin https://github.com/<tuo-utente>/tool-magazzino.git
git push -u origin main
```

Per aggiornare un tool basta sostituire il file corrispondente: Pages si ridispiega da solo.

## Note tecniche

- Ogni pagina è autonoma (CSS incluso), quindi funziona anche aprendola con doppio clic dal PC o inviandola singolarmente a un collega. I link della barra in alto funzionano solo se i file stanno nella stessa cartella.
- Le librerie (SheetJS per Excel, pdf.js / pdf-lib / JSZip / Tesseract.js per i PDF, JSZip anche per scrivere gli Excel del confronto cliente) arrivano da cdnjs e jsDelivr: al primo caricamento serve internet.
- Tema chiaro e scuro automatici, in base alle impostazioni del sistema.
