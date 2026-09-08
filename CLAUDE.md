# CLAUDE.md — istruzioni per lavorare su questo progetto

Sito statico di strumenti per il magazzino, pubblicato su GitHub Pages. Nessun backend, nessuna build,
nessun package manager: si modificano i file HTML e si caricano sul repository.

## Struttura

| File | Contenuto |
|---|---|
| `index.html` | Home con le schede dei tool |
| `giacenze.html` | Aggregatore giacenze (Excel → riepilogo per articolo) |
| `rotture.html` | Report rotture / Lost & Found (due Excel a confronto) |
| `inventario.html` | Lista di spunta con barcode + foglio di controllo corridoi (stampa A4) |
| `bordero.html` | Spacchetta borderò: PDF scansionati → un PDF per viaggio, con OCR |
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
- **Lista inventario:** filtro Picking vero e piano 010–019; ordinamento Scaffale → Lato → Campata → Piano
  con chiave `%06d{lato}%06d%03d`; un banner e un cambio pagina a ogni cambio di Scaffale+Lato.
- **Foglio di controllo:** usa tutte le righe del file, non i filtri della lista; picking e stoccaggio separati;
  casella nera dove quel lato non esiste.
- **Borderò:** solo quelli del magazzino di partenza (default 3980 / SETTALA) aprono un pacchetto; quelli di
  altri magazzini sono allegati. Il magazzino si legge **solo da ciò che sta sopra la riga "Viaggio:"**,
  altrimenti si prende per errore il "Mag. Dest.". Ogni PDF di input fa storia a sé, salvo l'opzione di unione.
- **La prima pagina di ogni PDF è sempre un borderò di partenza** (regola dichiarata dall'utente): se non viene
  riconosciuta è un errore di OCR, non un allegato. Va aperto comunque il pacchetto, tenendo i dati letti e
  mettendo `AZIENDA DA VERIFICARE` / `VIAGGIO DA VERIFICARE` al posto di quelli mancanti, con avviso a video.
  Non silenziare mai questo caso: un nome sbagliato si corregge, un pacchetto perso no.
- **Aggregatore giacenze:** intestazione riconosciuta automaticamente (nei file reali è alla riga 4);
  `Articolo` è la prima colonna con quel nome esatto, non `Codice Articolo`.

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
- **localStorage e simili non servono**: nessuno stato va salvato tra una sessione e l'altra.

## Come si verifica una modifica

Non ci sono test automatici nel repository, ma prima di consegnare una modifica va verificata davvero,
con Playwright e Chromium headless su `file:///…`, intercettando i CDN così da isolare la logica:

- **Logica pura** (parsing, aggregazione, costruzione pacchetti): chiamare le funzioni globali dentro
  `page.evaluate` con dati realistici e confrontare con un risultato calcolato in modo indipendente.
- **Aggregatore giacenze:** l'aggregazione va confrontata con lo stesso calcolo fatto in pandas.
- **Inventario:** il file `Inventario_Macro.xlsm` contiene già l'output della macro su 454 righe reali;
  si ricostruisce da lì una giacenza sorgente e si verifica che righe, ordine e banner coincidano.
- **Borderò:** i testi OCR reali delle pagine di prova si conservano in un JSON e si rigioca l'analisi
  sostituendo `renderHeader` e `ocrText`; il risultato atteso sul PDF di prova è 8 pacchetti,
  pagine borderò 1, 10, 14, 22, 29, 30, 35, 42, nessuna pagina non assegnata.
- **Stampa:** generare il PDF con `page.pdf()` e guardarlo davvero, convertendolo in immagine.
- In ogni test raccogliere `pageerror` e gli errori di console: devono essere zero.

L'OCR vero in browser non è verificabile in ambiente senza rete: in quel caso dirlo apertamente
invece di dare per funzionante ciò che non è stato eseguito.
