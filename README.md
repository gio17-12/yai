# yai — folder-based agent harness

## Cos'è

**yai** è un sistema a cartelle per orchestrare agenti AI. Il mondo dell'agente è suddiviso in 3 aree, consultabili sia dall'agente che dall'operatore umano:

| Cartella (in `main/`) | Scopo |
|---|---|
| `tasks/` | cose da fare |
| `memory/` | memoria persistente |
| `objects/` | artefatti prodotti |

L'agente opera all'interno di `main/`. Quando parte, legge `main/start.md` che gli dice di eseguire `bot/tools/start/run` per ottenere il contesto.

## Aggiungere un nuovo tool

I tool sono eseguibili che l'agente può lanciare per compiere azioni. Ogni tool vive in una propria sottocartella dentro `main/bot/tools/`.

### Struttura minima

```
main/bot/tools/<nome-tool>/
├── README.md     ← obbligatorio, con sezione ## Descrizione
└── run           ← obbligatorio, eseguibile (Python, bash, ...)
```

### Passi

1. Crea la cartella:

```bash
mkdir -p main/bot/tools/<nome-tool>
```

2. Crea `README.md` con almeno una sezione `## Descrizione`. Il sistema usa questa sezione per generare l'indice dei tool.

```markdown
## Descrizione
<descrizione breve di cosa fa il tool>

## Come usarlo
<istruzioni>
```

3. Crea `run` — uno script eseguibile in qualsiasi linguaggio (Python, bash, Node.js...):

```bash
#!/usr/bin/env python3
# il tuo codice qui
```

4. Rendi eseguibile:

```bash
chmod +x main/bot/tools/<nome-tool>/run
```

### Indice automatico

Quando qualcuno esegue `bot/tools/start/run`, lo script scansiona tutte le sottocartelle di `bot/tools/`, legge la sezione `## Descrizione` di ogni `README.md`, e rigenera `main/bot/tools/README.md` con l'elenco completo.

### Template (opzionali)

Se il tool produce output formattato, puoi aggiungere una cartella `templates/` con file `.md` contenenti placeholder `{{VAR}}` che lo script sostituisce a runtime.

## Struttura completa del repo

```
yai/
├── CONTRIBUTING.md    ← descrizione del paradigma (leggere per primo)
├── README.md          ← questo file
├── LICENSE
├── .gitignore
└── main/
    ├── start.md       ← punto d'ingresso per l'agente
    ├── memory/        ← memoria persistente
    ├── tasks/         ← cose da fare
    ├── objects/       ← artefatti
    └── bot/
        └── tools/     ← strumenti eseguibili dall'agente
            ├── start/ ← tool di bootstrap sessione
            └── ...    ← altri tool
```