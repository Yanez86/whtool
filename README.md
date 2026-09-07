# Tool Magazzino

Due strumenti web per elaborare gli estratti Excel del magazzino. Sono pagine HTML autonome: l'elaborazione avviene interamente nel browser, nessun file viene inviato a un server.

| File | Cosa fa |
|---|---|
| `index.html` | Home con i due strumenti |
| `giacenze.html` | **Aggregatore giacenze** — un file di giacenza UdC diventa un riepilogo per articolo |
| `rotture.html` | **Report rotture / Lost & Found** — confronto tra due file, con delta e totali |

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

## Pubblicazione su GitHub Pages

1. Su GitHub crea un repository nuovo, ad esempio `tool-magazzino`.
2. Carica i tre file HTML nella root: **Add file → Upload files → Commit changes**.
3. **Settings → Pages** → *Source*: **Deploy from a branch**, branch **main**, cartella **/ (root)** → **Save**.
4. Dopo circa un minuto è online su `https://<tuo-utente>.github.io/tool-magazzino/` — la home si apre da sola, i due tool sono raggiungibili dalla barra in alto.

Da riga di comando:

```bash
git init
git add index.html giacenze.html rotture.html README.md
git commit -m "Tool magazzino"
git branch -M main
git remote add origin https://github.com/<tuo-utente>/tool-magazzino.git
git push -u origin main
```

Per aggiornare un tool basta sostituire il file corrispondente: Pages si ridispiega da solo.

## Note tecniche

- Ogni pagina è autonoma (CSS incluso), quindi funziona anche aprendola con doppio clic dal PC o inviandola singolarmente a un collega. I link della barra in alto funzionano solo se i tre file stanno nella stessa cartella.
- La libreria Excel (SheetJS 0.18.5) viene caricata da cdnjs: al primo caricamento serve internet. Per una versione completamente offline si può incorporare la libreria nelle pagine.
- Tema chiaro e scuro automatici, in base alle impostazioni del sistema.
