# Roadmap LLM Engineering e Agents

## Obiettivo

Imparare a progettare applicazioni basate su LLM che siano:

```text
utili
comprensibili
controllabili
testabili
osservabili
sicure
riproducibili
manutenibili
```

Il progetto pratico utilizzato durante il percorso è:

```text
invoice-local
```

La domanda centrale è:

```text
Quanto deve decidere il codice
e quanto può decidere il modello?
```

La roadmap descrive soltanto la direzione generale.

La fase corrente viene sviluppata separatamente in:

```text
CURRENT.md
```

---

# Principio generale

Il percorso segue questa progressione:

```text
chiamata LLM
→ output strutturato
→ workflow deterministico
→ tool controllati
→ conoscenza esterna
→ valutazione e osservabilità
→ approvazione umana
→ agente controllato
→ sicurezza e operatività
```

L’autonomia aumenta soltanto quando il sistema è già misurabile e controllabile.

---

# Fase 0 — Fondamenti degli LLM

## Obiettivo

Costruire un modello mentale corretto del funzionamento di un LLM.

## Concetti principali

```text
token
tokenizzazione
next-token prediction
messaggi e ruoli
context window
sampling
temperature
allucinazioni
embeddings
prompt
structured output
tool calling
RAG
fine-tuning
```

## Risultato

Saper spiegare cosa accade tra l’invio di una richiesta e la generazione della risposta.

---

# Fase 1 — Servizio LLM minimale

## Obiettivo

Collegare direttamente `invoice-local` a un modello locale senza utilizzare framework agentici.

## Tecnologie iniziali

```text
Python
FastAPI
Ollama
Qwen
pytest
```

## Risultato

Un servizio che:

```text
si avvia
contatta Ollama
riceve una risposta
gestisce timeout ed errori
produce log comprensibili
possiede test automatici
```

---

# Fase 2 — Output strutturato

## Obiettivo

Trasformare testo naturale in dati validabili.

## Concetti principali

```text
JSON Schema
Pydantic
campi obbligatori
enum
nested model
validation error
retry controllato
normalizzazione
```

## Risultato

Un parser di fatture che restituisce dati strutturati oppure fallisce in modo esplicito e controllato.

## Regola

```text
Il modello propone i dati.
Pydantic verifica la struttura.
Il codice decide se accettarli.
```

---

# Fase 3 — Workflow deterministico

## Obiettivo

Separare interpretazione linguistica e regole di business.

## Concetti principali

```text
service layer
domain model
pure function
Decimal
rounding
state machine
unit test
separation of concerns
```

## Risultato

Un motore fattura utilizzabile e testabile anche senza LLM.

## Regola

Il modello non deve:

```text
calcolare totali
applicare IVA
decidere gli arrotondamenti
convalidare regole di business
```

---

# Fase 4 — Affidabilità e recupero dagli errori

## Obiettivo

Gestire retry, timeout, richieste duplicate, errori parziali e concorrenza.

## Concetti principali

```text
idempotency
timeout
retry
backoff
transazione
deduplicazione
concorrenza
checkpoint
resume
```

## Risultato

Ripetere la stessa richiesta non deve creare due fatture o lasciare dati incoerenti.

---

# Fase 5 — Tool calling controllato

## Obiettivo

Permettere al modello di proporre l’uso di funzioni senza cedergli il controllo dell’applicazione.

## Tool iniziali

```text
search_customer
get_customer
search_product
get_product
create_invoice_draft
validate_invoice
```

## Concetti principali

```text
tool schema
input validation
output validation
permission
read tool
write tool
audit log
side effect
```

## Risultato

Tool espliciti, validati, autorizzati e tracciati.

## Regola

Le azioni sensibili non vengono eseguite automaticamente.

---

# Fase 6 — RAG e conoscenza esterna

## Obiettivo

Rispondere utilizzando documenti esterni e indicando le fonti.

## Concetti principali

```text
document ingestion
chunking
embedding
retrieval
metadata
filter
reranking
context construction
citation
grounded answer
```

## Risultato

Un assistente che consulta policy e documenti e che non risponde quando le fonti non sono sufficienti.

## Test importanti

```text
documento irrilevante
informazione assente
versioni in conflitto
tenant errato
prompt injection nei documenti
```

---

# Fase 7 — Context, stato e memoria

## Obiettivo

Capire dove conservare informazioni e stato durante workflow lunghi.

## Concetti principali

```text
conversation history
working state
authoritative state
checkpoint
summary
semantic memory
user preference
context construction
```

## Risultato

Separare chiaramente:

```text
stato reale dell’applicazione
contesto temporaneo fornito al modello
cronologia della conversazione
conoscenza recuperata
```

## Regola

Lo stato autoritativo vive nel codice o nel database, non nella memoria testuale del modello.

---

# Fase 8 — Evals e osservabilità

## Obiettivo

Misurare la qualità e ricostruire ciò che accade durante ogni esecuzione.

## Concetti principali

```text
golden dataset
regression test
component eval
workflow eval
trace
span
metric
correlation ID
prompt version
model version
latency
failure taxonomy
```

## Risultato

Poter confrontare prompt e modelli e diagnosticare gli errori senza affidarsi alle impressioni.

## Metriche importanti

```text
accuratezza del parser
retrieval corretto
tool corretto
argomenti corretti
azioni non autorizzate
duplicati
latenza
errori
```

---

# Fase 9 — Approvazione umana

## Obiettivo

Richiedere conferma prima delle azioni sensibili.

## Stati di esempio

```text
draft
pending_approval
approved
finalized
```

## Concetti principali

```text
human-in-the-loop
approval gate
diff
confirmation
reversibility
audit
uncertainty
```

## Risultato

L’utente vede:

```text
cosa ha capito il modello
quali dati verranno modificati
quali tool verranno eseguiti
quali effetti avrà l’approvazione
```

Nessuna azione definitiva può essere eseguita senza uno stato valido di approvazione.

---

# Fase 10 — Agent loop controllato

## Obiettivo

Permettere al modello di scegliere alcuni passi successivi entro limiti precisi.

## Task di esempio

```text
Prepara le bozze di fatturazione del mese.
```

## Concetti principali

```text
agent loop
tool orchestration
state
checkpoint
termination condition
budget
planning limitato
failure recovery
controlled autonomy
```

## Limiti obbligatori

```text
numero massimo di tool call
durata massima
tool allowlist
solo creazione di bozze
nessuna finalizzazione automatica
output strutturato
audit completo
```

## Risultato

Un agente che può creare e validare bozze, fermarsi, fallire e riprendere senza compiere azioni definitive.

---

# Fase 11 — Sicurezza

## Obiettivo

Limitare abuso dei tool, accesso ai dati, prompt injection ed esfiltrazione.

## Concetti principali

```text
threat modeling
least privilege
RBAC
tenant isolation
secret management
PII
prompt injection
indirect prompt injection
tool abuse
data exfiltration
audit
```

## Risultato

Ogni rischio importante deve avere:

```text
controllo
test
log
procedura di risposta
```

---

# Fase 12 — Prestazioni e routing dei modelli

## Obiettivo

Scegliere modelli e configurazioni usando misure reali.

## Concetti principali

```text
cold start
time to first token
tokens per second
p50
p95
throughput
RAM
VRAM
concurrency
context size
queue
cache
backpressure
```

## Risultato

Una policy che seleziona il modello in base a:

```text
tipo di task
qualità misurata
latenza
risorse disponibili
```

---

# Fase 13 — Gateway e gestione centralizzata

## Obiettivo

Separare l’applicazione dalla gestione dei modelli e dei prompt.

## Concetti principali

```text
LLM gateway
model registry
prompt registry
routing
quota
rate limit
fallback
policy enforcement
cache
multi-tenant configuration
```

## Risultato

`invoice-local` utilizza un’interfaccia stabile senza conoscere ogni dettaglio del modello sottostante.

Questa fase può essere rimandata se esiste una sola applicazione e un solo modello.

---

# Fase 14 — Framework agentici

## Obiettivo

Confrontare una soluzione costruita direttamente con una basata su framework.

## Possibili strumenti

```text
SDK diretto
implementazione custom
LangGraph
LlamaIndex
OpenAI Agents SDK
LangChain
```

## Esperimento

Implementare lo stesso workflow in due modi:

```text
versione custom
versione con framework
```

## Confronto

```text
quantità di codice
trasparenza
testabilità
debug
tracing
gestione dello stato
lock-in
complessità
```

## Risultato

Saper motivare perché un framework è necessario e quali costi introduce.

---

# Fase 15 — Deployment e operatività

## Obiettivo

Rendere il sistema riproducibile fuori dall’ambiente di sviluppo.

## Concetti principali

```text
Docker
configuration
secret management
database migration
CI
health check
monitoring
backup
restore
resource limit
rollback
kill switch
degraded mode
```

## Risultato

Una nuova macchina può avviare il sistema seguendo esclusivamente il README di `invoice-local`.

Il sistema può continuare a offrire funzionalità essenziali anche quando il modello non è disponibile.

---

# Fase 16 — Progetto finale

## Obiettivo

Integrare tutte le competenze in un flusso end-to-end.

## Il sistema finale deve poter

```text
ricevere una richiesta naturale
estrarre dati strutturati
risolvere cliente e prodotto
creare una bozza
calcolare importi
validare la fattura
modificarla conversazionalmente
consultare policy tramite RAG
richiedere approvazione
preparare bozze mensili
tracciare ogni operazione
eseguire eval
gestire errori e rollback
```

## Demo suggerita

```text
1. richiesta ambigua
2. chiarimento
3. creazione della bozza
4. modifica conversazionale
5. consultazione di una policy
6. validazione
7. anteprima
8. approvazione
9. azione finale simulata
10. visualizzazione della trace
11. esecuzione degli eval
```

---

# Moduli opzionali

Questi argomenti non sono necessari per completare il percorso principale.

## MCP

Studiare dopo aver implementato un tool registry proprietario.

Serve a standardizzare l’esposizione di tool e risorse, ma non sostituisce:

```text
permessi
validazione
audit
sicurezza
```

## Multi-agent

Studiare soltanto quando esistono responsabilità realmente separate e valutabili.

Non creare più agenti soltanto per imitare una struttura aziendale.

## Fine-tuning

Considerare quando:

```text
prompt e RAG non bastano
il task è stabile
esiste un dataset di qualità
esistono eval affidabili
```

## Knowledge graph

Considerare quando le relazioni strutturate tra entità sono più importanti della semplice similarità semantica.

## Computer use

Considerare quando è necessario interagire con applicazioni che non espongono API affidabili.

---

# Regole permanenti

```text
Il modello interpreta.

Il codice applica le regole.

I tool producono effetti limitati.

Il database conserva lo stato autoritativo.

I documenti forniscono conoscenza.

Gli eval misurano la qualità.

Il tracing spiega cosa è successo.

Le policy limitano i rischi.

L’utente mantiene il controllo.
```

---

# Ordine di avanzamento

Non è necessario assegnare una settimana a ogni fase.

Ogni fase dura finché sono soddisfatti i suoi criteri di completamento.

La sequenza di lavoro è sempre:

```text
capire
→ sperimentare
→ implementare
→ testare
→ spiegare
→ avanzare
```
