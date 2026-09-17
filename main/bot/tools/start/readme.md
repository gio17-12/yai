## Descrizione
Calcola data/ora correnti e stampa l'albero di `main/` (topologia completa del repo, `bot/` incluso), renderizzati dal template in `templates/output.md`. È il punto di ingresso di ogni sessione: è quello che `start.md`, in root, dice all'assistente di eseguire per primo — per sapere che giorno è, che ora è, e avere sotto gli occhi l'intera struttura del sistema prima di fare qualsiasi altra cosa.

## Come usarlo
<!-- SYNTAX:START -->
Nessun argomento. Esegui: `bot/tools/start/run`
<!-- SYNTAX:END -->

## Esempi
```bash
bot/tools/start/run
```

## Dettagli
Lo script fa due cose in sequenza: costruisce l'albero delle cartelle, poi lo inserisce in un template insieme a data/ora.

**Costruzione dell'albero (`tree()`):** parte dalla cwd e scende ricorsivamente in ogni sottocartella, saltando i nomi presenti in `EXCLUDE` (un set definito in testa a `run`). Gli elementi di ogni livello sono ordinati alfabeticamente con le cartelle prima dei file, e la connettura visiva (`├──`, `└──`, indentazione con `│`) replica l'output di `tree` da shell. Non c'è un limite di profondità: scende finché ci sono sottocartelle.

**Composizione dell'output:** il file `templates/output.md` contiene testo fisso più quattro segnaposto — `{{DATE}}`, `{{WEEKDAY}}`, `{{TIME}}`, `{{TREE}}`. Lo script legge quel file come stringa e fa una sostituzione testuale diretta (niente motore di templating, solo `.replace()` in sequenza) prima di stampare il risultato su stdout.

**Assunzione implicita:** lo script non riceve né calcola il path della root — usa la cwd (`Path(".")`) così com'è al momento dell'esecuzione. Funziona correttamente solo se lanciato da dentro `main/`; non fa alcun controllo per verificarlo.