# Istruzioni per generare CURRENT.md

Questo file contiene esclusivamente le istruzioni per creare il file `CURRENT.md` del percorso di studio.

Il progetto pratico associato si chiama:

```text
invoice-local
```

## Input forniti a ChatGPT

Per generare un nuovo `CURRENT.md`, riceverai:

```text
Questo file (README.md)
ROADMAP.md
```

`ROADMAP.md` è la fonte ufficiale per:

* ordine delle fasi;
* fasi completate;
* prima fase ancora da svolgere;
* obiettivi generali del percorso.

Le fasi completate sono indicate con:

```markdown
- [x] Fase completata
```

Le fasi non ancora completate sono indicate con:

```markdown
- [ ] Fase non completata
```

## Compito

Leggi `ROADMAP.md` e individua la prima fase non completata.

Crea il contenuto completo del nuovo file:

```text
CURRENT.md
```

Non iniziare ancora la lezione.

Non farmi domande preliminari.

Non chiedere conferma sulla fase individuata.

Non modificare `ROADMAP.md`.

Non creare altri file.

Non espandere le fasi successive.

Restituisci soltanto il contenuto pronto da copiare in `CURRENT.md`.

Se tutte le fasi risultano completate, comunica semplicemente che la roadmap è terminata.

## Caratteristiche di CURRENT.md

`CURRENT.md` deve essere:

* pratico;
* ordinato;
* sufficientemente dettagliato per guidare lo studio;
* focalizzato su una sola fase;
* collegato a modifiche concrete in `invoice-local`;
* privo di spiegazioni inutilmente lunghe;
* utilizzabile durante più sessioni di studio.

Deve distinguere sempre:

* ciò che deve fare il modello;
* ciò che deve fare il codice;
* dove risiede lo stato autoritativo;
* quali output devono essere validati.

## Struttura obbligatoria

Genera `CURRENT.md` con questa struttura:

```markdown
# Fase corrente

## Fase

Nome e numero della fase.

## Obiettivo

Che cosa devo imparare.

## Risultato finale

Che cosa devo saper spiegare, costruire e testare alla fine della fase.

## Concetti essenziali

Solo i concetti realmente necessari per questa fase.

## Ordine di lavoro

Una sequenza numerata di piccoli passi.

Ogni passo deve indicare:

- cosa studiare;
- cosa provare;
- cosa implementare;
- come verificare il risultato.

## Esperimenti

Piccoli esperimenti utili a comprendere i concetti prima o durante l’implementazione.

Per ogni esperimento indica:

- domanda;
- procedura;
- risultato da osservare.

## Implementazione in invoice-local

Modifiche concrete da realizzare nel progetto.

Non generare automaticamente tutta l’implementazione.

## Test necessari

Elenco dei test normali, negativi e degli errori da simulare.

## Errori comuni

Problemi e incomprensioni tipiche della fase.

## Cose da non fare ancora

Tecnologie, astrazioni e funzionalità appartenenti a fasi successive.

## Criteri di completamento

Checklist verificabile per decidere quando segnare la fase come completata.

## Stato

- [ ] Studio completato
- [ ] Esperimenti completati
- [ ] Implementazione completata
- [ ] Test completati
- [ ] Casi negativi verificati
- [ ] Concetti spiegabili senza consultare le note
- [ ] Fase completata

## Note di lavoro

Spazio inizialmente vuoto per dubbi, errori, risultati e decisioni emersi durante la fase.
```

## Principi permanenti

Durante la generazione del file rispetta queste regole:

```text
Non usare un LLM per calcoli deterministici.

Non usare un agente quando basta un workflow.

Non accettare output non validati.

Non introdurre framework prima che esista un problema concreto.

Non anticipare argomenti appartenenti a fasi successive.

Il modello interpreta e propone.

Il codice valida, calcola e applica le regole.

Il database o il codice conservano lo stato autoritativo.

L’utente approva le azioni sensibili.
```

## Formato della risposta

La risposta deve contenere soltanto il contenuto completo di `CURRENT.md`, racchiuso in un unico blocco Markdown.

Non aggiungere introduzioni, commenti, spiegazioni o domande fuori dal file.
