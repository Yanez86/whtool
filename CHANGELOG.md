# Storico versioni

Ogni tool ha la propria versione, scritta nel piè di pagina della sua pagina e nella scheda della home.

## Aggregatore giacenze — 1.2 (26/11/2026)
- 1.2 — il separatore interno della chiave di aggregazione era un carattere di controllo (U+0001): ora è `|`.
  Messaggio esplicito se la libreria Excel non si carica dal CDN, invece di un errore muto al trascinamento.
- 1.1 — grafica unificata con gli altri tool e barra di navigazione comune.
- 1.0 — versione iniziale: aggregazione per articolo, descrizione, stato contabile e data scadenza,
  campi aggiuntivi a scelta, export XLSX e CSV.

## Report rotture / Lost & Found — 1.1 (08/09/2026)
- 1.1 — grafica unificata, barra di navigazione, tema scuro.
- 1.0 — versione iniziale (sviluppata a parte): confronto rotture / found con delta, totali ed export Excel.

## Inventario — 1.3 (27/11/2026)
- 1.3 — nuovo **Report conta**: si scelgono le aree (all'apertura nessuna) e per ciascuna esce, da una pagina nuova,
  l'elenco di ubicazioni e articoli, una riga per ubicazione + articolo con le quantità sommate, e una casella larga per
  la conta. In cima a ogni area *NOME INVENTARIO* e *SPUNTATORE* da compilare; il nome si può scrivere a video una volta
  sola (è lo stesso del foglio di controllo). Le caselle **Colli** e **Pezzi** aggiungono le quantità a sistema
  (`Quantità Stoccata`, `Pezzi Stoccati`); senza spunta è una **conta alla cieca**. Stampa senza URL del browser.
  A video, su schermi stretti, i documenti scorrono dentro il loro riquadro invece di allargare la pagina.
- 1.2 — lista e foglio di controllo si stampano senza l'indirizzo della pagina, la data e il titolo che il browser
  aggiunge nei margini: margine di pagina a zero, con i margini (10 mm sopra e sotto, 8 mm ai lati) spostati nel
  contenuto e ripetuti su ogni pagina. Impaginazione invariata (stesse pagine e stesse righe per pagina).
- 1.1 — il foglio di controllo è diviso **per area** (colonna `Area` della giacenza) e, dentro, fra picking e
  stoccaggio come prima: ogni tabella porta area e zona nel titolo. Le aree senza corridoi e campate (BOMBOLE,
  COSTOSI, C04, P01, RIS…), che prima non comparivano, stanno in fondo in *Altre aree*, una riga per area e zona
  con il numero di ubicazioni. Prima i corridoi con lo stesso numero di aree diverse finivano mescolati.
  Nuovo riquadro **Aree del foglio di controllo** per scegliere le aree da stampare: quelle di ingresso e uscita
  (tutte le aree che iniziano per I o U, comprese le `UCORSIA…`) partono escluse, con un avviso a video che le elenca
  tutte con le loro righe. In fondo al foglio l'elenco delle aree escluse. La lista inventario non cambia.
- 1.0 — sostituisce le macro `GeneraInventario` e `GeneraFoglioControllo`: lista di spunta stampabile con
  barcode Code 39 disegnati, foglio di controllo corridoi, filtri Picking e Piano modificabili.

## Verifica inventario — 1.1 (26/11/2026)
- 1.1 — il badge *Pari* e i messaggi informativi non avevano lo sfondo colorato: usavano due colori
  (`--ok-bg`, `--warn-bg`) mai definiti. Ora sono alias dei colori comuni.
- 1.0 — versione iniziale: confronto fisico/logico dell'export di inventario, con scelta Stock o Picking.
  Controlla ubicazione, SSCC (solo Stock), articolo e quantità in colli; pezzi (solo Picking) e lotti sono
  opzionali e spenti di default. Riepilogo pari/difettose/non contate, delta (eccedenze e ammanchi) per ubicazione
  e per articolo, scheda *Riconta* stampabile con le sole ubicazioni difettose ed export Excel.

## Spacchetta borderò — 1.3 (08/09/2026)
- 1.3 — la prima pagina di ogni PDF apre sempre un pacchetto: se l'OCR non la riconosce si usano i dati letti
  e un segnaposto `DA VERIFICARE` per quelli mancanti, con avviso e marcatura `BORDERO_FORZATO` nel CSV.
- 1.2 — OCR su più processi in parallelo e varianti di rilettura simultanee (circa 3× più veloce);
  casella *Analisi rapida*; l'elaborazione non viene più rallentata quando la scheda è in secondo piano;
  percentuale nel titolo della scheda e avviso se si prova a chiudere durante il lavoro.
- 1.1 — riconoscimento tollerante a maiuscole e minuscole (l'OCR legge spesso `viaggio` in minuscolo e la
  pagina andava persa) e rilettura con varianti anche quando manca un solo numero; avviso in evidenza se
  i dati lingua italiani non si scaricano.
- 1.0 — versione iniziale: porting nel browser di `split_bordero.py`.

## Etichette packing list — 1.1 (07/10/2026)
- 1.1 — stampa senza URL e data in basso: margine di pagina a zero, margini spostati nel contenuto.
- 1.0 — versione iniziale: dalla giacenza UdC una etichetta A4 (layout di `MODELLO.xlsx`) per ogni coppia
  Area + Versione, dove la versione è il numero bancale. Motivazione e codice in base all'area, con tabella
  modificabile (RITIRI COCA, ROTTURE 262, ROTTURE, BL5, BL3 precaricate). Una riga per articolo con quantità
  sommate; oltre 27 articoli le righe si abbassano per restare in una pagina. Stampa o PDF dal browser.

## Confronto giacenza cliente — 1.2 (26/11/2026)
- 1.2 — gli avvisi che riguardano articoli li elencano **tutti**, uno per riga (prima erano tra parentesi in fondo
  al messaggio e si fermavano a dieci). Per i pezzi per collo diversi ogni riga dice codice, descrizione, U.M. e i
  due valori a confronto; il pulsante *Mostra in tabella* e il filtro *Pezzi per collo diversi* mostrano solo quegli
  articoli, con i due valori evidenziati. Stesso elenco per gli articoli doppi nel tabulato (con i numeri di riga)
  e per quelli assenti dall'anagrafica Bollicine (con il magazzino).
- 1.1 — nel bilanciamento la nostra quantità si chiama **Giacenza Click** (prima "Nostra giacenza"), e così
  "Pezzi per collo Click" e "Solo Click"; accanto resta **Giacenza Bollicine**. Nell'Excel scaricato le
  intestazioni vanno a capo: prima "Giacenza Bollicine" restava tagliata a "Giacenza" dalla colonna stretta.
- 1.0 — versione iniziale, con due confronti fra la nostra Giacenza UdC e la giacenza del cliente.
  **1 · INVENTARIO**: il tabulato inventario del cliente torna compilato nella colonna *conta click*, modificato
  direttamente con JSZip così da restare identico (foglio, bordi, colonna colorata, filtro); in fondo, in rosso
  chiaro, gli articoli che abbiamo solo noi nel magazzino del tabulato, con U.M. scelta a video.
  **2 · BILANCIAMENTO CLICK - BOLLICINE**: la giacenza Bollicine (.xls) confrontata per articolo e magazzino
  (01 - VENDIBILE, AS, NV), con export nuovo e foglio delle impostazioni usate.
  Tabelle modificabili stato contabile → magazzino (DISPONIBILE → 01, SHORT → AS, il resto → NV) e
  U.M. → colli o pezzi (CT, CF, CS, PZ, PC a colli; BT, FS, BO, PL a pezzi).

## Home — 1.10 (27/11/2026)
- 1.10 — scheda aggiornata per Inventario 1.3.
- 1.9 — scheda aggiornata per Inventario 1.2.
- 1.8 — scheda aggiornata per Inventario 1.1.
- 1.7 — la scheda di Spacchetta borderò passa nella sezione **Obsoleti**, in fondo alla home, chiusa all'apertura
  (si apre con un clic). La voce "Borderò" resta nella barra in alto.
- 1.6 — scheda e voce di menu per Confronto giacenza cliente; schede aggiornate per Aggregatore giacenze 1.2
  e Verifica inventario 1.1.
- 1.5 — scheda e voce di menu per Etichette packing list.
- 1.4 — scheda e voce di menu per Verifica inventario.

La versione della home segue l'ultimo aggiornamento del sito.
