# Fase corrente

## Fase

**Fase 0 — Fondamenti degli LLM**

## Obiettivo

Costruire un modello mentale corretto di ciò che accade tra l’invio di una richiesta e la generazione della risposta di un LLM.

Applicare subito questo modello mentale a `invoice-local`, distinguendo:

- ciò che il modello può interpretare o proporre;
- ciò che il codice deve validare, calcolare e applicare;
- ciò che deve essere conservato come stato autoritativo;
- ciò che richiede approvazione dell’utente.

## Risultato finale

Alla fine della fase devo saper:

- spiegare il flusso `testo → messaggi → token → predizione → sampling → risposta`;
- descrivere context window, temperature, variabilità e allucinazioni;
- distinguere prompt, structured output, tool calling, RAG, embeddings e fine-tuning;
- riconoscere che un output plausibile non è necessariamente corretto;
- indicare quali output del modello devono essere validati;
- classificare correttamente le responsabilità di modello, codice, database e utente in `invoice-local`;
- eseguire e documentare piccoli esperimenti ripetibili, senza integrare ancora un modello nell’applicazione.

## Concetti essenziali

| Concetto | Che cosa devo capire | Implicazione per `invoice-local` |
|---|---|---|
| Token e tokenizzazione | Il modello elabora token, non direttamente parole o caratteri. | Testi, codici, numeri e JSON possono consumare token in modo diverso. |
| Next-token prediction | La risposta viene costruita scegliendo un token successivo alla volta. | Il modello genera una continuazione plausibile, non verifica automaticamente i fatti. |
| Messaggi e ruoli | Istruzioni, richiesta e cronologia formano il contesto inviato al modello. | I ruoli organizzano il prompt, ma non sostituiscono controlli e permessi nel codice. |
| Context window | Il modello considera una quantità limitata di token durante la generazione. | Il contesto è temporaneo e non deve contenere l’unica copia dello stato reale. |
| Sampling e temperature | I parametri di generazione influenzano la scelta tra token possibili. | Una temperature bassa riduce la variabilità, ma non garantisce correttezza o determinismo assoluto. |
| Allucinazioni | Il modello può produrre dettagli plausibili ma falsi o non supportati. | Cliente, prodotto, aliquota e identificatori devono essere verificati su fonti autoritative. |
| Embeddings | Rappresentano contenuti come vettori utili al confronto semantico. | Non sostituiscono identificatori esatti, regole di business o autorizzazioni. |
| Prompt | È l’insieme di istruzioni e dati forniti al modello. | Migliora il comportamento medio, ma non rende affidabile l’output. |
| Structured output | Richiede una forma esplicita, per esempio JSON. | La struttura deve essere validata e i valori devono essere controllati. |
| Tool calling | Il modello propone una funzione e i relativi argomenti. | Il codice decide se la chiamata è valida, autorizzata ed eseguibile. |
| RAG | Recupera documenti esterni e li inserisce nel contesto. | Fornisce conoscenza, ma non elimina automaticamente gli errori. |
| Fine-tuning | Modifica il comportamento del modello tramite esempi di addestramento. | Non serve per calcoli deterministici, dati aggiornati o regole applicative. |

### Confini permanenti

```text
Il modello interpreta e propone.
Il codice valida, calcola e applica le regole.
Il database o il codice conservano lo stato autoritativo.
L’utente approva le azioni sensibili.
```

Ogni output del modello deve essere considerato non affidabile finché il codice non ne ha verificato struttura, valori, riferimenti e coerenza con lo stato reale.

## Ordine di lavoro

1. **Ricostruire il flusso di generazione**
   - **Studiare:** tokenizzazione, inferenza autoregressiva e sampling.
   - **Provare:** descrivere cosa accade con la richiesta “crea una fattura per Mario Rossi”.
   - **Implementare:** disegnare nelle note un diagramma del flusso richiesta-risposta.
   - **Verificare:** spiegare ogni passaggio senza dire genericamente che “il modello capisce”.

2. **Osservare token e context window**
   - **Studiare:** differenza tra caratteri, parole, token e limite di contesto.
   - **Provare:** confrontare una frase, un codice fiscale, un importo e un frammento JSON con un tokenizer compatibile con il modello scelto.
   - **Implementare:** registrare esempi e conteggi in una tabella.
   - **Verificare:** spiegare perché la cronologia non è memoria permanente e perché il contesto ha un limite.

3. **Studiare messaggi, ruoli e prompt**
   - **Studiare:** istruzioni, richiesta utente, esempi e cronologia.
   - **Provare:** inviare la stessa richiesta con e senza contesto sul dominio di `invoice-local`.
   - **Implementare:** conservare i due prompt e annotare le differenze osservate.
   - **Verificare:** separare le informazioni fornite nell’input da quelle inferite o inventate.

4. **Misurare variabilità e allucinazioni**
   - **Studiare:** sampling, temperature e generazione di contenuti non supportati.
   - **Provare:** ripetere lo stesso prompt più volte e interrogare il modello su un cliente inesistente.
   - **Implementare:** registrare modello, parametri, prompt, risposte e conclusioni.
   - **Verificare:** distinguere stabilità della forma, stabilità del contenuto e correttezza.

5. **Confrontare testo libero e dati strutturati**
   - **Studiare:** differenza tra validità sintattica e validità di business.
   - **Provare:** chiedere gli stessi dati prima in prosa e poi in JSON.
   - **Implementare:** classificare errori di formato, dati mancanti, valori inventati e contraddizioni.
   - **Verificare:** spiegare perché JSON valido può comunque contenere dati errati.

6. **Distinguere le tecniche LLM**
   - **Studiare:** scopo di prompt, structured output, tool calling, RAG, embeddings e fine-tuning.
   - **Provare:** associare ogni tecnica a un problema concreto e indicare almeno un problema che non risolve.
   - **Implementare:** creare una tabella decisionale minima per `invoice-local`.
   - **Verificare:** motivare perché calcoli, permessi e regole di business appartengono al codice.

7. **Definire i confini di invoice-local**
   - **Studiare:** proposta del modello, validazione, stato autoritativo e approvazione umana.
   - **Provare:** classificare almeno dieci azioni del futuro flusso di fatturazione.
   - **Implementare:** creare una matrice con colonne `azione`, `modello`, `codice`, `stato autoritativo`, `approvazione`, `validazione`.
   - **Verificare:** ogni azione deve avere un responsabile chiaro e una fonte autoritativa esplicita.

8. **Consolidare la fase**
   - **Studiare:** soltanto i concetti che non risultano ancora spiegabili.
   - **Provare:** esporre l’intero flusso senza consultare le note.
   - **Implementare:** correggere diagramma, glossario e matrice in base agli errori emersi.
   - **Verificare:** completare la checklist finale con esempi riferiti a `invoice-local`.

## Esperimenti

### Esperimento 1 — Stesso prompt, più esecuzioni

- **Domanda:** la stessa richiesta produce sempre la stessa risposta?
- **Procedura:** eseguire almeno cinque volte lo stesso prompt con lo stesso modello e gli stessi parametri disponibili.
- **Risultato da osservare:** variazioni lessicali, omissioni, aggiunte e differenze nei dati proposti.

### Esperimento 2 — Effetto della temperature

- **Domanda:** come cambia la generazione al variare della temperature?
- **Procedura:** se il parametro è disponibile, ripetere lo stesso prompt con una temperature bassa e una più alta, lasciando invariato il resto.
- **Risultato da osservare:** la variabilità può cambiare, ma la correttezza non è garantita in nessuna configurazione.

### Esperimento 3 — Dati mancanti

- **Domanda:** il modello chiede chiarimenti oppure completa i dati assenti?
- **Procedura:** richiedere una fattura senza specificare cliente, quantità o altro dato essenziale e vietare esplicitamente supposizioni.
- **Risultato da osservare:** richiesta di chiarimento, placeholder, omissione oppure invenzione di dati.

### Esperimento 4 — Cliente inesistente

- **Domanda:** il modello riconosce di non avere accesso all’anagrafica reale?
- **Procedura:** fornire un nome inventato e chiedere codice cliente, indirizzo e partita IVA senza offrire alcuna fonte.
- **Risultato da osservare:** eventuali dettagli fabbricati dimostrano che il modello non può essere la fonte autoritativa.

### Esperimento 5 — Testo libero contro JSON

- **Domanda:** una risposta strutturata è automaticamente affidabile?
- **Procedura:** chiedere gli stessi dati prima in prosa e poi in JSON, lasciando un dato ambiguo o assente.
- **Risultato da osservare:** il JSON può essere valido nella forma ma errato, incompleto o contraddittorio nel contenuto.

### Esperimento 6 — Calcolo del totale

- **Domanda:** un risultato numerico corretto rende il modello adatto ai calcoli monetari autoritativi?
- **Procedura:** chiedere il totale di alcune righe fattura e confrontarlo con un calcolo deterministico separato.
- **Risultato da osservare:** anche quando coincide, il calcolo deve restare nel codice per essere ripetibile, testabile e governato da regole esplicite.

## Implementazione in invoice-local

In questa fase non integrare ancora un modello nel codice applicativo.

Realizzare soltanto materiale di studio e decisioni architetturali:

1. Creare una nota, per esempio:

   ```text
   docs/learning/fase-0-fondamenti-llm.md
   ```

   Deve contenere:

   - diagramma del flusso richiesta-risposta;
   - glossario essenziale;
   - differenza tra contesto e stato autoritativo;
   - differenza tra forma valida e contenuto valido;
   - sintesi delle tecniche LLM della fase.

2. Creare un registro, per esempio:

   ```text
   experiments/fase-0/README.md
   ```

   Per ogni esperimento registrare:

   - modello e versione, quando disponibili;
   - parametri disponibili;
   - prompt esatto;
   - risposta o estratto rilevante;
   - osservazione;
   - conclusione.

3. Inserire nella documentazione la matrice delle responsabilità di `invoice-local`.

Non generare automaticamente tutta questa documentazione: deve essere costruita e corretta durante lo studio.

## Test necessari

### Test normali

- Spiegare il flusso completo da testo a risposta.
- Distinguere token, parola, carattere e context window.
- Spiegare sampling, temperature e allucinazioni.
- Associare correttamente ogni tecnica LLM al proprio scopo.
- Classificare almeno dieci responsabilità tra modello, codice, database e utente.
- Ripetere un esperimento conservando prompt, modello e parametri.

### Test negativi ed errori da simulare

- risposta vuota o interrotta;
- campo essenziale omesso;
- cliente o prodotto inventato;
- JSON valido con valori non validi;
- due campi contraddittori;
- importo ambiguo;
- risposta che tratta la cronologia come stato reale;
- risposta che presenta un’inferenza come fatto verificato;
- risposta diversa a parità apparente di richiesta;
- contesto insufficiente o contenente un’informazione errata.

In questa fase è sufficiente riconoscere, classificare e documentare gli errori. La gestione tecnica verrà implementata nelle fasi successive.

## Errori comuni

- Confondere una risposta plausibile con una risposta verificata.
- Pensare che il modello funzioni come un database o un motore di regole.
- Credere che temperature zero garantisca correttezza o determinismo assoluto.
- Trattare la context window come memoria permanente.
- Pensare che un prompt dettagliato sostituisca la validazione.
- Considerare JSON valido equivalente a dati di business validi.
- Pensare che una tool call proposta sia già autorizzata o già eseguita.
- Credere che il RAG impedisca automaticamente le allucinazioni.
- Usare embeddings per uguaglianze esatte, permessi o regole.
- Usare il fine-tuning per dati aggiornati o calcoli deterministici.
- Affidare al modello importi, IVA, arrotondamenti o stato reale.
- Cambiare più variabili nello stesso esperimento e trarre conclusioni non verificabili.

## Cose da non fare ancora

- Non collegare ancora `invoice-local` a Ollama o ad altri provider.
- Non creare ancora un servizio FastAPI per il modello.
- Non introdurre framework agentici o librerie di orchestrazione.
- Non implementare ancora JSON Schema o modelli Pydantic completi.
- Non costruire ancora il motore deterministico della fattura.
- Non creare tool che leggono o modificano dati.
- Non aggiungere un database vettoriale o una pipeline RAG.
- Non introdurre memoria persistente, eval, tracing o approval workflow.
- Non fare fine-tuning.
- Non eseguire azioni sensibili sulla base del solo output del modello.

## Criteri di completamento

- [ ] So spiegare il percorso completo da richiesta a risposta generata.
- [ ] So spiegare next-token prediction senza descrivere il modello come un database.
- [ ] So distinguere token, context window, sampling e temperature.
- [ ] So spiegare perché una risposta plausibile può essere falsa.
- [ ] So distinguere prompt, structured output, tool calling, RAG, embeddings e fine-tuning.
- [ ] Ho completato e documentato almeno cinque esperimenti.
- [ ] Ogni esperimento conserva prompt, modello, parametri, risultato e conclusione.
- [ ] Ho creato il diagramma del flusso richiesta-risposta.
- [ ] Ho creato il glossario essenziale.
- [ ] Ho creato la matrice delle responsabilità di `invoice-local`.
- [ ] Per ogni output del modello so indicare la validazione necessaria.
- [ ] So indicare dove deve risiedere lo stato autoritativo.
- [ ] So spiegare perché i calcoli monetari appartengono al codice.
- [ ] Non ho introdotto dipendenze o funzionalità delle fasi successive.
- [ ] Riesco a spiegare i concetti essenziali senza consultare le note.

## Stato

- [ ] Studio completato
- [ ] Esperimenti completati
- [ ] Implementazione completata
- [ ] Test completati
- [ ] Casi negativi verificati
- [ ] Concetti spiegabili senza consultare le note
- [ ] Fase completata

## Note di lavoro

