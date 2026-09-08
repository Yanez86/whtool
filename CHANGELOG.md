# Storico versioni

Ogni tool ha la propria versione, scritta nel piè di pagina della sua pagina e nella scheda della home.

## Aggregatore giacenze — 1.1 (08/09/2026)
- 1.1 — grafica unificata con gli altri tool e barra di navigazione comune.
- 1.0 — versione iniziale: aggregazione per articolo, descrizione, stato contabile e data scadenza,
  campi aggiuntivi a scelta, export XLSX e CSV.

## Report rotture / Lost & Found — 1.1 (08/09/2026)
- 1.1 — grafica unificata, barra di navigazione, tema scuro.
- 1.0 — versione iniziale (sviluppata a parte): confronto rotture / found con delta, totali ed export Excel.

## Inventario — 1.0 (08/09/2026)
- 1.0 — sostituisce le macro `GeneraInventario` e `GeneraFoglioControllo`: lista di spunta stampabile con
  barcode Code 39 disegnati, foglio di controllo corridoi, filtri Picking e Piano modificabili.

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

## Home — 1.3 (08/09/2026)
La versione della home segue l'ultimo aggiornamento del sito.
