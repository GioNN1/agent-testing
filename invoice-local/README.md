# invoice-local

Progetto didattico locale per imparare a costruire applicazioni basate su LLM attraverso la preparazione di bozze di fatture simulate.

Il percorso di studio associato si trova nel repository affiancato:

```text
llm-agents-study
```

## Principio architetturale

```text
Il modello interpreta il linguaggio naturale.

Il codice valida, calcola e applica le regole.

L’utente approva le azioni sensibili.
```

L’output del modello non viene mai considerato automaticamente corretto o autoritativo.

## Obiettivi

Il progetto verrà sviluppato progressivamente per studiare:

* integrazione con un modello locale;
* output strutturato;
* workflow deterministici;
* tool calling;
* RAG;
* eval;
* osservabilità;
* approvazione umana;
* agenti controllati.

Nel README verranno documentate soltanto le funzionalità realmente implementate.

## Non-goals

`invoice-local` non è un software fiscale reale.

Non prevede:

* invio reale allo SdI;
* conformità fiscale completa;
* firma digitale;
* conservazione sostitutiva;
* integrazione con sistemi contabili reali;
* utilizzo con dati personali o aziendali reali.

Fatture, clienti, prodotti, PDF, XML e regole fiscali sono simulazioni didattiche.

## Stack

```text
Python
FastAPI
Ollama
Qwen
Pydantic
SQLite
pytest
```

Lo stack verrà aggiornato solo quando nuove tecnologie saranno effettivamente introdotte.

## Stato del progetto

In sviluppo.

Fase corrente: vedere `llm-agents-study/CURRENT.md`.

## Installazione

Da completare dopo il bootstrap del progetto.

## Avvio

Da completare dopo il bootstrap del progetto.

## Test

Da completare dopo l’introduzione dei primi test automatici.
