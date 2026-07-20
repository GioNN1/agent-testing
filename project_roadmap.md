# TODO: Da revisionare
# Local Agent Testing - Invoices Helper - Project Roadmap

## 1. Scopo del progetto

`Invoices Helper` è un progetto didattico per imparare AI engineering moderno costruendo un assistente locale per la preparazione di bozze di fatture.

L'obiettivo non è costruire subito un software fiscale reale, ma un sistema completo e sicuro che ti faccia praticare:

- SDK diretto verso un LLM locale via Ollama/Qwen;
- output strutturato e validazione schema;
- tool calling;
- workflow deterministici;
- RAG su manuali e policy;
- evals e regression test;
- agent loop controllato;
- observability;
- human approval;
- document generation;
- sicurezza multi-tenant simulata;
- controllo performance/costo anche in locale.

Il progetto parte piccolo e diventa progressivamente più agentico.

---

## 2. Principio guida

La regola architetturale centrale è:

```text
LLM = interpreta, spiega, propone, corregge linguaggio naturale
Codice = calcola, valida, decide regole rigide, genera documenti
Utente = approva azioni sensibili
```

Nel dominio fatturazione questo è fondamentale.

Il modello può capire una richiesta tipo:

```text
Fai una fattura ad Acme per 3 giornate di sviluppo backend,
2 call di consulenza e uno sconto del 10%. Pagamento a 30 giorni.
```

Ma non deve essere lui a decidere in modo libero totali, IVA, validità fiscale o finalizzazione.

Il modello estrae e aiuta. Il codice verifica e calcola.

---

## 3. Scope e non-goals

### In scope

- Creazione di bozze fattura da linguaggio naturale.
- Registry clienti e catalogo prodotti/servizi fittizio.
- Calcolo deterministico di imponibile, IVA, sconti e totale.
- Validazione dati mancanti.
- Generazione PDF mock.
- Generazione XML mock ispirato a FatturaPA, non invio reale.
- RAG su manuali/policy fittizie.
- Agent mode per preparazione bozze di fine mese.
- Human approval prima di ogni azione sensibile.
- Eval harness e osservabilità.

### Out of scope iniziale

- Invio reale a SdI.
- Conservazione sostitutiva.
- Firma digitale.
- Compliance fiscale completa.
- Calcolo fiscale reale per casi complessi.
- Integrazione con sistemi contabili reali.

Nota: in Italia la fattura elettronica reale usa il formato XML FatturaPA e passa dal Sistema di Interscambio. Questo progetto usa solo versioni mock o semplificate, salvo eventuale studio successivo degli XSD ufficiali.

---

## 4. Stack suggerito

### Base locale

- Python 3.11+
- FastAPI
- Ollama con Qwen/Gwen locale
- Pydantic
- SQLite all'inizio, poi Postgres se serve
- SQLModel oppure SQLAlchemy
- Jinja2 per template documenti
- WeasyPrint o ReportLab per PDF
- lxml per XML
- pytest

### RAG opzionale

- Chroma, Qdrant o pgvector
- SentenceTransformers o embedding locali supportati da Ollama
- Markdown come formato documentazione iniziale

### Agent/workflow opzionale avanzato

- Implementazione custom iniziale
- LangGraph più avanti se vuoi workflow stateful e controllabili
- OpenAI Agents SDK solo come studio comparativo o se vuoi provare agent loop gestiti e tracing cloud

---

## 5. Architettura target

```text
                 ┌─────────────────────┐
                 │     Frontend/API     │
                 └──────────┬──────────┘
                            │
                 ┌──────────▼──────────┐
                 │   Invoice Service    │
                 └──────────┬──────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│  LLM Parser   │   │ Business Core │   │ Tool Registry │
│ Ollama/Qwen   │   │ totals/rules  │   │ lookup/render │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                 ┌──────────▼──────────┐
                 │ DB + file storage    │
                 │ customers/invoices   │
                 └──────────┬──────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│ PDF generator │   │ XML generator │   │ RAG index      │
└───────────────┘   └───────────────┘   └───────────────┘
```

---

## 6. Data model minimo

### Customer

```json
{
  "id": "cust_001",
  "tenant_id": "tenant_demo_001",
  "name": "Acme SRL",
  "vat_number": "IT01234567890",
  "tax_code": "01234567890",
  "address": "Via Roma 1, Milano",
  "default_payment_terms_days": 30,
  "email": "admin@acme.example"
}
```

### Product / Service

```json
{
  "id": "prod_backend_api",
  "tenant_id": "tenant_demo_001",
  "code": "CONS_BACKEND_API",
  "description": "Consulenza sviluppo backend API",
  "unit": "hour",
  "unit_price": 80.0,
  "vat_rate": 22.0
}
```

### InvoiceDraft

```json
{
  "id": "inv_draft_001",
  "tenant_id": "tenant_demo_001",
  "customer_id": "cust_001",
  "status": "draft",
  "issue_date": "2026-07-20",
  "payment_terms_days": 30,
  "lines": [],
  "net_total": 0.0,
  "vat_total": 0.0,
  "gross_total": 0.0,
  "validation_errors": [],
  "requires_human_approval": true
}
```

### InvoiceLine

```json
{
  "id": "line_001",
  "description": "Consulenza sviluppo backend API",
  "quantity": 5,
  "unit": "hour",
  "unit_price": 80.0,
  "discount_percent": 0,
  "vat_rate": 22.0,
  "net_amount": 400.0,
  "vat_amount": 88.0,
  "gross_amount": 488.0
}
```

---

## 7. Milestone roadmap

## Milestone 0 — Bootstrap progetto

### Obiettivo

Creare la struttura repository, avviare FastAPI, collegare Ollama e preparare logging minimo.

### Endpoint iniziali

```text
GET /health
GET /models/local
POST /llm/ping
```

### Deliverable

- Repo funzionante.
- Docker Compose opzionale.
- Config `.env`.
- Client Ollama minimale.
- Test base.

### Competenze

- Setup AI service locale.
- Configurazione modello.
- Separazione client LLM / service layer.

---

## Milestone 1 — Natural language invoice parser

### Obiettivo

Dato un testo utente, estrarre una struttura fattura parziale usando output JSON validato.

### Esempio input

```text
Fattura ad Acme 5 ore di consulenza backend a 80 euro l'ora, IVA 22%, pagamento a 30 giorni.
```

### Esempio output

```json
{
  "customer_name": "Acme",
  "invoice_lines": [
    {
      "description": "consulenza backend",
      "quantity": 5,
      "unit": "hour",
      "unit_price": 80.0,
      "vat_rate": 22.0
    }
  ],
  "payment_terms_days": 30,
  "missing_fields": []
}
```

### Endpoint

```text
POST /invoice/parse
```

### Acceptance criteria

- Output sempre validato con Pydantic.
- Se il modello produce JSON invalido, il sistema fallisce in modo controllato.
- Nessun calcolo fiscale affidato al modello.
- Campi mancanti esplicitati.

### Competenze AI

- SDK diretto.
- Structured output.
- Prompt design.
- Schema validation.
- Retry controllato.

---

## Milestone 2 — Motore deterministico fattura

### Obiettivo

Calcolare totali e validare le regole rigide senza LLM.

### Endpoint

```text
POST /invoice/draft
POST /invoice/{id}/validate
```

### Regole minime

- Quantità > 0.
- Prezzo unitario >= 0.
- IVA consentita da lista configurata.
- Cliente obbligatorio.
- Almeno una riga fattura.
- Totali calcolati dal codice.

### Acceptance criteria

- Test unitari su calcolo imponibile, IVA, sconti, totale.
- Differenze di arrotondamento gestite in modo riproducibile.
- Validazioni espresse come codici macchina, non solo testo.

### Competenze

- Separazione AI / business logic.
- Determinismo.
- Testabilità.

---

## Milestone 3 — Customer registry e product catalog

### Obiettivo

Aggiungere clienti e prodotti/servizi locali. Il sistema deve risolvere nomi ambigui senza inventare.

### Endpoint

```text
POST /customers
GET /customers/search?q=
POST /products
GET /products/search?q=
POST /invoice/from-message
```

### Esempio

Input:

```text
Fattura a Mario 4 ore backend e 2 ore deploy.
```

Possibile risposta:

```text
Ho trovato più clienti compatibili con "Mario":
1. Mario Rossi SRL
2. Mario Rossi Consulenze

Scegli il cliente prima di creare la bozza.
```

### Acceptance criteria

- Soglia di confidenza per match cliente/prodotto.
- Ambiguità gestita con richiesta di conferma.
- Nessuna selezione automatica se confidence insufficiente.

### Competenze AI

- Entity resolution.
- Semantic matching.
- Ambiguity handling.
- Human clarification.

---

## Milestone 4 — Tool calling controllato

### Obiettivo

Esporre funzioni interne come tool, ma mantenere l'orchestrazione principale nel codice.

### Tool iniziali

```python
search_customer(name: str)
get_customer(customer_id: str)
search_product(query: str)
create_invoice_draft(data: InvoiceDraftInput)
validate_invoice(invoice_id: str)
render_invoice_pdf(invoice_id: str)
render_invoice_xml(invoice_id: str)
```

### Pattern consigliato

```text
utente scrive richiesta
  ↓
LLM estrae intento e campi
  ↓
codice decide quali tool chiamare
  ↓
codice crea/valida bozza
  ↓
LLM spiega risultato
```

### Acceptance criteria

- Ogni tool ha input/output schema.
- Ogni tool call viene loggata.
- Le write operation sono separate dalle read operation.
- Nessuna finalizzazione senza approval.

### Competenze AI

- Tool design.
- Tool schema.
- Orchestrazione deterministica.
- Auditabilità.

---

## Milestone 5 — PDF e XML mock

### Obiettivo

Generare documenti partendo dalla bozza validata.

### Endpoint

```text
GET /invoice/{id}/pdf
GET /invoice/{id}/xml
```

### XML mock semplificato

Ispirarsi alla struttura FatturaPA, ma mantenere uno schema ridotto:

```text
DatiTrasmissione
CedentePrestatore
CessionarioCommittente
DatiGeneraliDocumento
DettaglioLinee
DatiRiepilogo
```

### Acceptance criteria

- PDF leggibile.
- XML generato dal codice, non dall'LLM.
- XML validabile contro un tuo XSD semplificato.
- Errori spiegabili via validation assistant.

### Competenze

- Document generation.
- JSON → XML mapping.
- Schema validation.
- Separazione formato interno / formato esterno.

---

## Milestone 6 — Validation assistant

### Obiettivo

Il codice valida, l'LLM spiega gli errori in modo utile.

### Input validatore

```json
{
  "valid": false,
  "errors": [
    {
      "field": "customer.vat_number",
      "code": "MISSING_REQUIRED_FIELD",
      "severity": "blocking"
    }
  ]
}
```

### Output assistente

```text
Non posso finalizzare la bozza perché manca la partita IVA del cliente.
Puoi inserirla manualmente oppure scegliere un cliente già registrato.
```

### Endpoint

```text
POST /assistant/explain-validation
```

### Acceptance criteria

- Il modello non inventa errori nuovi.
- Il modello spiega solo errori ricevuti dal validatore.
- Differenza chiara tra warning e blocking errors.

### Competenze AI

- Grounded generation.
- Controlled explanation.
- Safety by design.

---

## Milestone 7 — RAG su manuale e policy

### Obiettivo

Aggiungere documentazione interrogabile.

### Documenti iniziali

```text
docs/policy_pagamenti.md
docs/policy_sconti.md
docs/manuale_fatturazione_mock.md
docs/faq_amministrazione.md
docs/regole_iva_fittizie.md
```

### Domande supportate

```text
Posso applicare uno sconto del 20%?
Quali dati servono per finalizzare una bozza?
Perché questa fattura è bloccata?
Quando una bozza deve andare in revisione?
```

### Pipeline

```text
documenti
  ↓
chunking
  ↓
embedding
  ↓
vector store
  ↓
retrieval
  ↓
risposta con fonti
```

### Endpoint

```text
POST /assistant/ask
```

### Acceptance criteria

- Risposte con fonti/citazioni.
- Se il contesto non basta, il sistema dice che non lo sa.
- Supporto a filtri per tenant/document set.
- Eval retrieval su domande note.

### Competenze AI

- RAG.
- Chunking.
- Metadata.
- Retrieval evaluation.
- Hallucination reduction.

---

## Milestone 8 — Human approval e action safety

### Obiettivo

Introdurre approval gates per azioni sensibili.

### Azioni sensibili

```text
finalizzare fattura
generare XML definitivo
inviare email al cliente
marcare come pagata
cancellare bozza
modificare anagrafica cliente
```

### Stato approvazione

```text
draft → pending_approval → approved → finalized
```

### Endpoint

```text
POST /invoice/{id}/request-approval
POST /invoice/{id}/approve
POST /invoice/{id}/reject
```

### Acceptance criteria

- Nessun tool write sensibile può essere eseguito senza approval.
- Audit log completo.
- Motivo approvazione/rifiuto registrato.
- Test che provano a bypassare approval.

### Competenze AI

- Human-in-the-loop.
- Safe tool calling.
- Permission boundaries.
- Audit trail.

---

## Milestone 9 — Evals

### Obiettivo

Costruire un eval harness per misurare regressioni e qualità.

### Dataset esempio

```json
{
  "input": "Fattura ad Acme 3 ore backend a 80 euro",
  "expected_customer": "Acme SRL",
  "expected_lines_count": 1,
  "expected_net_total": 240.0,
  "should_require_confirmation": true,
  "should_not_finalize": true
}
```

### Metriche

```text
customer corretto
righe corrette
totali corretti
campi mancanti trovati
tool chiamati corretti
nessuna azione sensibile senza approval
risposta comprensibile
fonti RAG corrette
latenza
numero chiamate LLM
```

### Acceptance criteria

- Test automatici per parser.
- Test automatici per calcoli.
- Test per casi ambigui.
- Test per prompt injection e richieste pericolose.
- Report eval generato in Markdown/JSON.

### Competenze AI

- Eval-driven development.
- Regression testing.
- Prompt versioning.
- Model comparison.
- Quality gates.

---

## Milestone 10 — Observability dashboard

### Obiettivo

Sapere cosa è successo in ogni run.

### Log minimo

```json
{
  "run_id": "run_001",
  "tenant_id": "tenant_demo_001",
  "user_id": "user_demo_001",
  "model": "qwen-local",
  "prompt_version": "invoice_parser_v1",
  "input_hash": "...",
  "output_schema_valid": true,
  "tool_calls": ["search_customer", "create_invoice_draft"],
  "duration_ms": 1420,
  "estimated_input_tokens": 620,
  "estimated_output_tokens": 310,
  "approval_required": true,
  "errors": []
}
```

### Dashboard minima

```text
ultime richieste
richieste fallite
tool più usati
campi mancanti più frequenti
tempo medio risposta
bozze create
bozze bloccate
```

### Acceptance criteria

- Ogni richiesta ha un `run_id`.
- Ogni tool call è tracciata.
- Ogni output LLM è associato a prompt version e model.
- Possibile riprodurre/debuggare una run.

### Competenze AI

- LLM tracing.
- Debugging.
- Auditabilità.
- Production readiness.

---

## Milestone 11 — Conversational editing e patch strutturate

### Obiettivo

Permettere all'utente di modificare una bozza in linguaggio naturale.

### Esempio

Utente:

```text
No, togli il deploy e metti 6 ore invece di 4.
```

Output LLM non deve essere una nuova fattura completa, ma una patch:

```json
{
  "operations": [
    {
      "operation": "remove_line",
      "line_match": "deploy"
    },
    {
      "operation": "update_line",
      "line_match": "backend",
      "field": "quantity",
      "value": 6
    }
  ]
}
```

### Acceptance criteria

- LLM genera patch, non stato finale autoritativo.
- Codice applica patch e rivalida.
- In caso di match ambiguo, richiesta chiarimento.

### Competenze AI

- Stateful editing.
- LLM as patch generator.
- Conversation state.
- Validation after modification.

---

## Milestone 12 — Agent mode: chiusura mese

### Obiettivo

Creare un agente read-only/write-limited per preparare bozze di fine mese.

### Prompt esempio

```text
Prepara le bozze di fatturazione per marzo.
```

### Sequenza agentica attesa

```text
1. leggere attività fatturabili
2. raggrupparle per cliente
3. recuperare dati cliente
4. recuperare listino
5. creare bozze
6. validare bozze
7. generare riepilogo
8. segnalare problemi
9. chiedere approvazione
```

### Guardrail

```text
max 20 tool call
nessun invio reale
nessuna finalizzazione senza conferma
solo bozze
max tempo run
max documenti letti
output schema obbligatorio
audit log completo
```

### Endpoint

```text
POST /agent/month-end-invoicing
```

### Acceptance criteria

- L'agente non può bypassare approval.
- L'agente produce un report finale strutturato.
- Tutte le tool call sono tracciate.
- Se fallisce un tool, l'agente degrada in modo controllato.

### Competenze AI

- Agent loop.
- Tool orchestration.
- State management.
- Failure recovery.
- Human approval.

---

## 8. Moduli AI opzionali ad alto valore

## A. Prompt registry

Versiona prompt e template.

```text
invoice_parser_v1
invoice_parser_v2
invoice_validator_explainer_v1
rag_answerer_v1
agent_month_end_v1
```

Scopo:

- capire quale prompt ha prodotto quale output;
- confrontare versioni;
- fare rollback;
- collegare eval a prompt/model.

---

## B. Model router locale

Con Ollama puoi provare più modelli.

```text
modello piccolo → classificazione semplice
modello medio → parsing complesso
modello embedding → ricerca semantica
doppio passaggio → task critici
```

Scopo:

- qualità vs latenza;
- routing per task;
- fallback se un modello fallisce;
- confronto Qwen/Gwen locale con altri modelli.

---

## C. Critic / reviewer model

Dopo la generazione della bozza, un secondo passaggio controlla:

```text
La bozza è coerente con il messaggio originale?
Ci sono righe inventate?
I dati cliente sono ambigui?
Serve conferma?
```

Pattern:

```text
generator → deterministic validator → AI reviewer → human approval
```

---

## D. Hallucination guard e red teaming

Casi da testare:

```text
cliente inesistente
prodotto inesistente
richiesta ambigua
regola non presente nel manuale
utente chiede di saltare conferma
utente chiede di inventare partita IVA
utente tenta prompt injection contro i tool
```

Obiettivo:

- abituare il sistema a dire “non lo so”;
- impedire invenzioni;
- impedire azioni non autorizzate.

---

## E. Synthetic data generation

Usa il modello per generare casi di test fittizi:

```text
100 richieste normali
50 richieste ambigue
30 richieste malformate
20 richieste pericolose
```

Poi usali nell'eval harness.

---

## F. Multi-tenant safety simulata

Crea più tenant fittizi:

```text
tenant_alpha
tenant_beta
tenant_gamma
```

Regole:

- un utente vede solo clienti/fatture del proprio tenant;
- il RAG recupera solo documenti del tenant corretto;
- i tool hanno sempre `tenant_id` obbligatorio;
- gli eval testano tentativi di accesso cross-tenant.

---

## G. Report generation

Report utili:

```text
riepilogo fatture mese
bozze bloccate e motivi
clienti con dati mancanti
previsione incassi simulata
log errori agente
report eval settimanale
```

Output:

- Markdown;
- PDF;
- JSON strutturato.

---

## 9. Repo structure consigliata

```text
invoice-copilot-local/
  app/
    api/
      routes_health.py
      routes_llm.py
      routes_invoice.py
      routes_assistant.py
      routes_agent.py
    core/
      config.py
      logging.py
      security.py
    llm/
      ollama_client.py
      prompts.py
      schemas.py
      parser.py
      reviewer.py
    invoices/
      models.py
      service.py
      calculator.py
      validator.py
      renderer_pdf.py
      renderer_xml.py
    tools/
      registry.py
      customer_tools.py
      product_tools.py
      invoice_tools.py
    rag/
      ingest.py
      chunking.py
      embeddings.py
      retriever.py
      answerer.py
    evals/
      datasets/
      runner.py
      metrics.py
      reports.py
    observability/
      traces.py
      events.py
      dashboard.py
  docs/
    policy_pagamenti.md
    policy_sconti.md
    manuale_fatturazione_mock.md
    faq_amministrazione.md
  tests/
    test_parser.py
    test_calculator.py
    test_validator.py
    test_tools.py
    test_evals.py
  scripts/
    seed_demo_data.py
    run_evals.py
  pyproject.toml
  README.md
```

---

## 10. Ordine di implementazione consigliato

Non partire dall'agente.

Ordine consigliato:

```text
1. Ollama client + FastAPI skeleton
2. Parser JSON strutturato
3. Motore deterministico di calcolo
4. Validatore
5. Registry clienti/prodotti
6. Tool layer controllato
7. PDF/XML mock
8. Validation assistant
9. RAG manuali/policy
10. Evals
11. Observability
12. Human approval
13. Conversational editing
14. Agent mode fine mese
15. Model router / critic / red-team
```

---

## 11. Cosa deve insegnarti il progetto

Alla fine dovresti saper decidere:

```text
Questa parte va fatta con codice deterministico?
Questa parte richiede interpretazione LLM?
Serve retrieval?
Serve tool calling?
Serve agente?
Serve human approval?
Come misuro se funziona?
Come debuggo quando sbaglia?
Come impedisco azioni non sicure?
```

Questo è il vero valore del progetto.

Non impari “a usare LangChain”.

Impari a progettare un sistema AI production-grade.

---

## 12. Riferimenti utili

- [Ollama Structured Outputs](https://docs.ollama.com/capabilities/structured-outputs)
- [Ollama API Reference](https://ollama.readthedocs.io/en/api/)
- [LlamaIndex — Introduction to RAG](https://developers.llamaindex.ai/python/framework/understanding/rag/)
- [OpenAI Agents SDK guide](https://developers.openai.com/api/docs/guides/agents)
- [OpenAI Agents SDK tracing](https://openai.github.io/openai-agents-python/tracing/)
- [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [FatturaPA — formato XML e documentazione ufficiale](https://www.fatturapa.gov.it/it/norme-e-regole/documentazione-fattura-elettronica/formato-fatturapa/)
- [Agenzia delle Entrate — Sistema di Interscambio](https://www.agenziaentrate.gov.it/portale/aree-tematiche/fatturazione-elettronica/guida-fatturazione-elettronica/come-predisporre-inviare-ricevere-fe/cosa-fa-sistema-interscambio-fe)
