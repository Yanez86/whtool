# CLAUDE.md — istruzioni per lavorare su questo progetto

Sito statico di strumenti per il magazzino, pubblicato su GitHub Pages. Nessun backend, nessuna build,
nessun package manager: si modificano i file HTML e si caricano sul repository.

## Struttura

| File | Contenuto |
|---|---|
| `index.html` | Home con le schede dei tool; quelli obsoleti (oggi solo il Borderò) in `<details class="obsoleti">`, chiuso |
| `giacenze.html` | Aggregatore giacenze (Excel → riepilogo per articolo) |
| `rotture.html` | Report rotture / Lost & Found (due Excel a confronto) |
| `inventario.html` | Lista di spunta con barcode + foglio di controllo per area + report conta (stampa A4) |
| `verifica-inventario.html` | Confronto fisico/logico dell'export di inventario (Stock o Picking), scheda Riconta |
| `bordero.html` | Spacchetta borderò: PDF scansionati → un PDF per viaggio, con OCR |
| `etichette.html` | Etichette packing list: una etichetta A4 per Area + Versione (bancale) |
| `confronto-cliente.html` | Confronto giacenza cliente: 1 · INVENTARIO (tabulato) e 2 · BILANCIAMENTO CLICK - BOLLICINE |
| `CHANGELOG.md` | Storico delle versioni dei singoli tool |

## Regole di questo repository

1. **Ogni pagina è autonoma.** CSS e JavaScript stanno dentro il file, nessun `.css` o `.js` esterno.
   Serve perché l'utente spedisce spesso una singola pagina a un collega o la apre con doppio clic.
   Il prezzo è che il blocco `<style>` è duplicato: quando si cambia lo stile va replicato su tutte le pagine.
2. **La barra di navigazione in alto è identica ovunque**, con `class="active"` sulla voce corrente.
   Aggiungendo un tool va aggiornata su tutte le pagine più la scheda nella home.
3. **Ogni tool ha la propria versione.** Sta nel piè di pagina `<footer class="ver noprint">` della pagina
   e nella scheda della home (`div.vertool`): quando si modifica un tool vanno aggiornati **entrambi**, più
   una riga in `CHANGELOG.md`. Numerazione semplice: correzioni e ritocchi alzano il decimale, un tool nuovo
   parte da 1.0. La data è quella dell'aggiornamento, in formato gg/mm/aaaa.
4. **Librerie solo da cdnjs.cloudflare.com e cdn.jsdelivr.net, con versione fissata.**
   SheetJS 0.18.5, pdf.js 3.11.174, pdf-lib 1.17.1, JSZip 3.10.1, Tesseract.js 5.
   Chi carica librerie deve gestire il caso "non caricata" con un messaggio esplicito, non un errore muto.
5. **Tutto gira nel browser.** Nessun file dell'utente deve uscire dal PC: è un requisito, non un dettaglio.
6. **Interfaccia e commenti in italiano.** I nomi delle funzioni possono restare in inglese.
7. **Niente caratteri di controllo nei sorgenti** (una volta ci è finito dentro un NUL come separatore di chiave:
   rende il file "binario" per git e per gli editor). Separatori: usare `|`.
8. **Tema chiaro/scuro** via `prefers-color-scheme` sulle variabili CSS in `:root`. I documenti da stampare
   restano invece sempre bianchi con testo nero, con colori scritti espliciti.

## Regole di dominio da non reinterpretare

Vengono da macro VBA e script Python già in uso: il comportamento deve restare identico salvo richiesta esplicita.

- **Lato dalla campata:** `(campata / 10)` intero, pari → `DX`, dispari → `SX`.
- **Lista inventario:** filtro Picking vero e piano 010–019; aree scelte a video, all'apertura solo quelle R seguite da
  un numero (`/^R\d/i`: R01…R04; non RESO, RIS, ROTTURE), tutte se il file non ha la colonna Area; ordinamento Scaffale → Lato → Campata → Piano
  con chiave `%06d{lato}%06d%03d`; un banner e un cambio pagina a ogni cambio di Scaffale+Lato.
  **Una riga per ubicazione + articolo** (non una per riga di giacenza come la macro): le righe uguali si accorpano
  tenendo la prima nell'ordine del file, articoli diversi nella stessa ubicazione restano separati (richiesta
  dell'utente, 27/11/2026). Nei test la lista va confrontata con quella della macro/1.0 senza i doppioni.
  Le righe fuori scaffale (senza scaffale o senza campata) delle aree spuntate vengono dopo, un blocco *Area X* a pagina
  nuova per area, ordinate per ubicazione e articolo, con il solo filtro Picking (richiesta dell'utente, 27/11/2026).
  Senza colonna Area nel file restano fuori, come nella macro.
- **Foglio di controllo:** usa tutte le righe del file, non i filtri della lista; diviso per `Area`, e dentro ogni area
  picking e stoccaggio separati; casella nera dove quel lato non esiste. Le righe senza scaffale o campata non si
  perdono: vanno in *Altre aree*, una riga per area e zona. Aree di ingresso/uscita = **tutte** quelle che iniziano
  per `I` o `U` (`/^[IU]/i`, comprese `UCORSIA502` e simili, confermato dall'utente): escluse in partenza, con avviso
  a video che le elenca se hanno righe; si possono rimettere dal riquadro Aree.
- **Report conta:** aree scelte a video, **nessuna** all'apertura; una riga per `Ubicazione Corrente` + `Articolo` con
  colli (`Quantità Stoccata`) e pezzi (`Pezzi Stoccati`) sommati; una pagina nuova per area, con NOME INVENTARIO a
  sinistra e SPUNTATORE a destra. Caselle Colli e Pezzi spente di default: senza spunta è una conta alla cieca e il
  titolo lo dice. Ordine: ubicazioni a scaffale con la chiave della lista, poi le altre per codice.
- **Borderò:** solo quelli del magazzino di partenza (default 3980 / SETTALA) aprono un pacchetto; quelli di
  altri magazzini sono allegati. Il magazzino si legge **solo da ciò che sta sopra la riga "Viaggio:"**,
  altrimenti si prende per errore il "Mag. Dest.". Ogni PDF di input fa storia a sé, salvo l'opzione di unione.
- **La prima pagina di ogni PDF è sempre un borderò di partenza** (regola dichiarata dall'utente): se non viene
  riconosciuta è un errore di OCR, non un allegato. Va aperto comunque il pacchetto, tenendo i dati letti e
  mettendo `AZIENDA DA VERIFICARE` / `VIAGGIO DA VERIFICARE` al posto di quelli mancanti, con avviso a video.
  Non silenziare mai questo caso: un nome sbagliato si corregge, un pacchetto perso no.
- **Aggregatore giacenze:** intestazione riconosciuta automaticamente (nei file reali è alla riga 4);
  `Articolo` è la prima colonna con quel nome esatto, non `Codice Articolo`.
- **Confronto giacenza cliente:** due confronti con nome fisso, mostrato in alto e nei file scaricati:
  **1 · INVENTARIO** (tabulato inventario del cliente) e **2 · BILANCIAMENTO CLICK - BOLLICINE** (giacenza Bollicine).
  - Il "magazzino" del cliente è il nostro stato contabile: DISPONIBILE → `01` (01 - VENDIBILE), SHORT → `AS`,
    tutto il resto → `NV`. Tabella modificabile a video, valori predefiniti a ogni apertura.
  - U.M. del cliente → conteggio: CT, CF, CS, PZ, PC a **colli** (`Quantità Stoccata`); BT, FS, BO, PL a **pezzi**
    (`Pezzi Stoccati`); U.M. sconosciuta → colli con avviso. Misurato sui file reali del 26/11/2026: BT a pezzi perché
    il cliente conta le bottiglie (`s_qtaxc`/`qxc` = 1 contro 6 per collo da noi); PZ e PC a colli perché i
    confezionati (es. P5776, 10 pezzi per collo) il cliente li conta a colli.
  - Abbinamento su `Articolo` della giacenza contro `s_codart` / `kart`, ignorando maiuscole e spazi.
    Differenza = cliente − nostra (come la `diff` del tabulato). Righe rosse: cliente 0, noi con merce.
  - INVENTARIO, formato reale ("tabulato conta inventario", dal 28/11/2026): `.xls` con `s_codart`, `s_desart`,
    `art_um`, `s_qtaxc`, `conta ` (con lo spazio). Niente giacenza del cliente: si compila `conta` = nostra quantità
    nel magazzino del tabulato (default 01), senza differenze. Si restituisce in `.xlsx` (unico modo per colorare
    le righe) ricopiando foglio, intestazioni esatte, larghezze lette con `cellStyles:true` e grafica Arial 10 con bordi.
  - INVENTARIO, formato precedente (`giacenza`, `conta click`, `diff`): ancora accettato; se è `.xlsx` si restituisce
    **il file del cliente** modificato con JSZip, con confronto e formula `diff`.
  - In entrambi i formati, in fondo, in rosso chiaro `FFFFC7CE`, solo gli articoli che abbiamo noi e lui no,
    **con merce nel magazzino del tabulato**; `art_um` scelta a video riga per riga (vuota = colli).
  - BILANCIAMENTO: confronto per articolo **e** magazzino sulla colonna `giac` (non `dispo`: sui file reali
    coincidono 750 righe su 1.112 con `giac`, 578 con `dispo`). Solo le righe con merce da almeno una parte.

## Trappole già incontrate (non ripeterle)

- **Regex OCR sensibili alle maiuscole.** L'OCR legge spesso `viaggio` o `azienda` in minuscolo, specie con
  virgolette storte davanti alla parola: le regex di viaggio e azienda devono avere il flag `i`.
  È il bug che faceva perdere una pagina su cinquanta.
- **Non ridurre l'area della testata per andare più veloce.** Misurato: passando da `0.28` a `0.22` di altezza
  si perdono azienda e viaggio su alcune pagine, perché tesseract segmenta peggio con poche righe.
- **`tessedit_do_invert=0` non fa guadagnare tempo** e perde qualche lettura. Lasciarlo attivo.
- **Per accelerare l'OCR si parallelizza**, non si abbassa la qualità: pool di worker Tesseract (max 4) e
  le sei varianti di rilettura lanciate insieme.
- **pdf-lib non supporta `horizontalScale` in `drawText`.** Per il testo invisibile si riduce la dimensione
  del font finché la parola rientra nella sua larghezza.
- **Stampa senza URL e data del browser:** si ottiene con `@page{margin:0}` e i margini dentro il contenuto (come
  in Etichette). Se il documento continua su più pagine (lista inventario) il padding non basta, perché vale solo in
  cima e in fondo: si avvolge in `table.pagina` con una riga vuota in `thead` e `tfoot`, che il browser ripete su ogni
  pagina. Effetto collaterale: dentro `table.pagina` un selettore discendente come `tbody tr` prende anche le righe di
  `thead` delle tabelle interne; nei test usare `:scope > tbody > tr`.
- **localStorage e simili non servono**: nessuno stato va salvato tra una sessione e l'altra.
- **SheetJS 0.18.5 (gratuito) non scrive stili**: riscrivendo un file perde bordi, colori e larghezze, e non sa
  colorare le righe. Per restituire il file del cliente si modifica il suo XML con JSZip (celle, righe nuove,
  stile clonato con riempimento rosso, `dimension`, `autoFilter`, nome `_xlnm._FilterDatabase`); si toglie
  `calcChain.xml` e si mette `fullCalcOnLoad`. Gli Excel nuovi si scrivono a mano, sempre con JSZip.
- **"Bad uncompressed size" in console** leggendo la Giacenza UdC: è SheetJS che segnala gli zip del WMS
  (con data descriptor), il file viene letto bene. Non è un errore della pagina; nei test va tenuto separato.

## Come si verifica una modifica

Non ci sono test automatici nel repository, ma prima di consegnare una modifica va verificata davvero,
con Playwright e Chromium headless su `file:///…`, intercettando i CDN così da isolare la logica:

- **Logica pura** (parsing, aggregazione, costruzione pacchetti): chiamare le funzioni globali dentro
  `page.evaluate` con dati realistici e confrontare con un risultato calcolato in modo indipendente.
- **Aggregatore giacenze:** l'aggregazione va confrontata con lo stesso calcolo fatto in pandas.
- **Inventario:** il file `Inventario_Macro.xlsm` contiene già l'output della macro su 454 righe reali;
  si ricostruisce da lì una giacenza sorgente e si verifica che righe, ordine e banner coincidano.
  Foglio di controllo e report conta si rifanno in pandas (raggruppamento per area / ubicazione + articolo) e si
  confrontano riga per riga; le stampe si generano con `displayHeaderFooter: true` per controllare che l'URL non esca.
- **Borderò:** i testi OCR reali delle pagine di prova si conservano in un JSON e si rigioca l'analisi
  sostituendo `renderHeader` e `ocrText`; il risultato atteso sul PDF di prova è 8 pacchetti,
  pagine borderò 1, 10, 14, 22, 29, 30, 35, 42, nessuna pagina non assegnata.
- **Confronto giacenza cliente:** rifare in pandas i due confronti con le stesse regole e confrontare riga per riga.
  Con i file di prova del 26/11/2026 (tabulato Settala, Giacenza UdC, Bollicine; non sono nel repository):
  INVENTARIO 1.090 righe del tabulato + 119 aggiunte; BILANCIAMENTO 1.157 righe, magazzino 01 750 pari su 1.112,
  AS 5 su 5, NV 20 su 40. Con il tabulato conta del 27/11/2026 e la stessa giacenza: 1.389 righe, 1.064 con nostra
  merce, 325 a 0 (324 articoli assenti, C17AN solo in AS), 4 aggiunte (H11CC, H12BY, H12BZ, H12CB).
  Gli Excel scaricati vanno riaperti (openpyxl: valori, formule, colori, filtro) e
  convertiti in PDF con LibreOffice per guardarli.
- **Stampa:** generare il PDF con `page.pdf()` e guardarlo davvero, convertendolo in immagine.
- In ogni test raccogliere `pageerror` e gli errori di console: devono essere zero.

L'OCR vero in browser non è verificabile in ambiente senza rete: in quel caso dirlo apertamente
invece di dare per funzionante ciò che non è stato eseguito.
