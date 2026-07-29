# Fase corrente

## Fase

**Fase 1 — Fondamenti degli LLM**

## Stato

Da iniziare.

Questa fase costruisce il modello mentale necessario per comprendere le fasi successive. Non richiede ancora un servizio FastAPI, un framework agentico o un workflow completo. Gli esperimenti servono a osservare il comportamento di un modello e a distinguere ciò che il modello può proporre da ciò che il codice deve verificare.

## Obiettivo

Comprendere che cosa accade tra l'invio di una richiesta e la generazione di una risposta da parte di un Large Language Model.

Al termine della fase devi saper distinguere:

- testo, token e rappresentazioni numeriche;
- contesto ricevuto dal modello e stato reale dell'applicazione;
- generazione probabilistica e calcolo deterministico;
- risposta plausibile e informazione verificata;
- testo libero e output strutturato;
- conoscenza appresa durante l'addestramento e conoscenza recuperata da fonti esterne;
- proposta di una chiamata a uno strumento ed esecuzione effettiva dello strumento.

## Risultato finale

Devi saper spiegare, senza consultare il file, una sequenza simile a questa:

1. l'applicazione costruisce i messaggi da inviare;
2. il testo viene tokenizzato;
3. i token entrano nella finestra di contesto del modello;
4. il modello calcola una distribuzione di probabilità per il token successivo;
5. una strategia di sampling sceglie il token;
6. il processo si ripete fino alla fine della risposta o al raggiungimento di un limite;
7. l'applicazione riceve l'output;
8. il codice valida struttura, contenuto e autorizzazioni;
9. eventuali dati esterni, tool o azioni vengono gestiti dall'applicazione, non magicamente dal modello.

Devi inoltre saper indicare quali parti di `invoice-local` possono utilizzare un LLM e quali devono rimanere deterministiche.

## Mappa della fase

```text
input dell'utente
    ↓
messaggi e ruoli
    ↓
tokenizzazione
    ↓
finestra di contesto
    ↓
next-token prediction
    ↓
sampling e temperature
    ↓
risposta generata
    ↓
validazione dell'applicazione
    ↓
eventuali tool, RAG o azioni controllate
```

I concetti devono essere studiati in questo ordine:

```text
1. token
2. tokenizzazione
3. next-token prediction
4. messaggi e ruoli
5. context window
6. sampling
7. temperature
8. allucinazioni
9. prompt
10. structured output
11. tool calling
12. embeddings
13. RAG
14. fine-tuning
```

## Materiale da studiare

### Concetto 1 — Token

#### Definizione

Un token è una delle unità numeriche in cui il testo viene trasformato prima di essere elaborato dal modello. Un token può corrispondere a una parola intera, a una parte di parola, a un segno di punteggiatura, a uno spazio o ad altri frammenti di testo.

Il modello non riceve direttamente le parole come le vede una persona. Riceve identificatori numerici associati ai token.

#### Modello mentale

Immagina che il testo venga smontato in tessere. Ogni tessera ha un numero. Il modello lavora con la sequenza dei numeri, non con il documento originale come oggetto semantico già compreso.

```text
"Fattura numero 42"
→ tessere testuali
→ identificatori numerici
```

#### Come funziona

1. l'applicazione prepara una stringa o una serie di messaggi;
2. il tokenizer divide il testo in unità;
3. ogni unità viene convertita in un identificatore;
4. gli identificatori vengono trasformati in rappresentazioni numeriche interne;
5. il modello elabora la sequenza.

La corrispondenza tra testo e token dipende dal tokenizer del modello. Modelli diversi possono dividere la stessa frase in modo diverso.

#### Esempio minimale

La frase:

```text
Crea una fattura per Acme.
```

potrebbe essere suddivisa in unità simili a:

```text
Crea | una | fattura | per | Ac | me | .
```

Questa è soltanto un'illustrazione: la suddivisione reale dipende dal tokenizer.

#### Applicazione a invoice-local

In `invoice-local`, ogni richiesta dell'utente, istruzione di sistema, documento recuperato e risposta precedente consuma token.

Il concetto serve per capire:

- perché prompt molto lunghi hanno un costo computazionale maggiore;
- perché una richiesta può superare la capacità del modello;
- perché nomi, codici prodotto o dati insoliti possono essere spezzati in più parti;
- perché il conteggio delle parole non coincide con il conteggio dei token.

Il client che invia la richiesta gestisce il testo. Il tokenizer appartiene al modello o alla libreria del modello. Gli output contenenti codici cliente, importi o identificatori devono comunque essere validati dal codice.

#### Cosa non risolve

Conoscere i token non garantisce che il modello comprenda correttamente una fattura. Non risolve ambiguità, errori nei dati, autorizzazioni, calcoli fiscali o accesso al database.

#### Errori comuni

- Pensare che un token equivalga sempre a una parola.
- Pensare che due modelli usino necessariamente la stessa tokenizzazione.
- Stimare la lunghezza del contesto contando soltanto le parole dell'utente.
- Dimenticare che anche istruzioni, cronologia e output consumano token.

#### Domande di verifica

1. Perché il numero di parole non permette di conoscere con precisione il numero di token?
2. Quali parti di una richiesta a `invoice-local` consumano token oltre al testo dell'utente?
3. Perché un codice prodotto insolito può richiedere più token di una parola comune?

### Concetto 2 — Tokenizzazione

#### Definizione

La tokenizzazione è il processo che converte il testo in una sequenza di token e, normalmente, di identificatori numerici. Il processo inverso converte i token generati nuovamente in testo leggibile.

#### Modello mentale

La tokenizzazione è il traduttore meccanico posto all'ingresso e all'uscita del modello:

```text
testo → token → modello → token → testo
```

Non è una traduzione linguistica. È una codifica necessaria affinché il modello possa elaborare sequenze discrete.

#### Come funziona

1. il tokenizer applica il proprio vocabolario e le proprie regole;
2. cerca unità note nel testo;
3. divide le parti non presenti come unità intere in frammenti più piccoli;
4. assegna un ID a ogni token;
5. aggiunge, quando necessario, token speciali per separare messaggi o indicare inizio e fine;
6. durante la generazione, riconverte gli ID prodotti in testo.

#### Esempio minimale

Un termine frequente come:

```text
fattura
```

potrebbe essere rappresentato da un solo token, mentre una stringa come:

```text
INV-IT-2026-000184-A
```

potrebbe essere divisa in molti token.

#### Applicazione a invoice-local

La tokenizzazione influenza il budget disponibile per:

- istruzioni del sistema;
- richiesta dell'utente;
- esempi inseriti nel prompt;
- testo estratto dalle fatture;
- documenti recuperati con RAG;
- risposta generata.

In una futura funzione di parsing, `invoice-local` dovrà limitare o segmentare input troppo lunghi. Il codice deve anche preservare il testo originale, perché la ricostruzione del modello non è una fonte autoritativa dei dati.

#### Cosa non risolve

La tokenizzazione non decide quali dati siano corretti. Non verifica che una partita IVA, una data o un totale siano validi. Non protegge da prompt injection e non sostituisce una strategia di gestione del contesto.

#### Errori comuni

- Confondere tokenizzazione con embedding.
- Supporre che la tokenizzazione mantenga intatti tutti i codici e i numeri.
- Considerare il tokenizer un componente semantico che “capisce” il testo.
- Ignorare i token speciali e i messaggi aggiunti automaticamente dal template del modello.

#### Domande di verifica

1. Qual è la differenza tra tokenizzazione ed embedding?
2. Perché `INV-IT-2026-000184-A` può essere più costoso di una parola comune?
3. Perché `invoice-local` dovrebbe conservare l'input originale oltre alla versione inviata al modello?

### Concetto 3 — Next-token prediction

#### Definizione

La next-token prediction è il compito fondamentale di un modello linguistico autoregressivo: data una sequenza di token, il modello assegna una probabilità ai possibili token successivi.

Il modello non scrive tutta la risposta in un unico passaggio. Genera un token, aggiorna il contesto con quel token e ripete il processo.

#### Modello mentale

Immagina un completamento automatico estremamente sofisticato:

```text
input corrente → probabilità dei possibili token successivi
```

Dopo ogni scelta, il nuovo token diventa parte dell'input per la scelta seguente.

#### Come funziona

1. il modello riceve tutti i token presenti nel contesto;
2. calcola rappresentazioni interne delle relazioni tra i token;
3. produce un punteggio per ogni token del vocabolario;
4. converte i punteggi in probabilità;
5. una strategia di selezione sceglie il token successivo;
6. il token scelto viene aggiunto alla sequenza;
7. il ciclo continua finché viene prodotto un token di fine, viene raggiunto un limite o l'applicazione interrompe la generazione.

#### Esempio minimale

Dato il testo:

```text
Il totale imponibile della fattura è
```

il modello può assegnare probabilità elevate a token che rappresentano numeri o espressioni monetarie. Questo non significa che abbia eseguito correttamente il calcolo: può soltanto aver prodotto una continuazione plausibile.

#### Applicazione a invoice-local

Questo concetto spiega perché il modello è utile per:

- interpretare richieste in linguaggio naturale;
- riconoscere campi descritti in modi diversi;
- proporre una struttura coerente;
- formulare spiegazioni.

Spiega anche perché non deve essere considerato un calcolatore o un database. Se l'utente scrive “fattura per Acme con due consulenze”, il modello può proporre l'intento e i campi. Il codice deve recuperare il cliente reale, applicare prezzi e aliquote e verificare i risultati.

#### Cosa non risolve

La next-token prediction non garantisce verità, completezza, coerenza globale, autorizzazioni o rispetto delle regole di business. Non accede automaticamente a dati aggiornati e non conserva uno stato autoritativo.

#### Errori comuni

- Pensare che il modello recuperi sempre una risposta già memorizzata.
- Pensare che una frase fluida dimostri che il ragionamento sia corretto.
- Pensare che il modello calcoli necessariamente ogni numero che produce.
- Considerare la probabilità linguistica equivalente alla probabilità che un fatto sia vero.

#### Domande di verifica

1. Perché una risposta grammaticalmente perfetta può contenere un totale errato?
2. Che cosa cambia nel processo dopo la generazione di ogni token?
3. Qual è la differenza tra “continuazione probabile” e “dato verificato”?

### Concetto 4 — Messaggi e ruoli

#### Definizione

Molte API per LLM organizzano l'input come una sequenza di messaggi associati a ruoli. I nomi esatti dipendono dal sistema, ma i ruoli più comuni rappresentano istruzioni dell'applicazione, richieste dell'utente, risposte del modello e risultati di strumenti.

I ruoli aiutano a strutturare il contesto. Non costituiscono da soli un confine di sicurezza assoluto.

#### Modello mentale

Immagina una trascrizione con etichette:

```text
applicazione: regole generali
utente: richiesta
assistente: risposta precedente
tool: risultato di una funzione
```

Il modello vede una sequenza preparata secondo un template. Le etichette influenzano l'interpretazione, ma tutto entra comunque nel contesto elaborato dal modello.

#### Come funziona

1. l'applicazione definisce le istruzioni principali;
2. aggiunge la richiesta dell'utente;
3. include, se necessario, la cronologia utile;
4. include risultati di tool o documenti con una provenienza riconoscibile;
5. il template del modello converte i messaggi in token;
6. il modello genera il messaggio successivo.

Quando le istruzioni sono in conflitto, il comportamento dipende dal modello, dall'addestramento e da come l'applicazione costruisce il contesto. Il codice non deve affidare la sicurezza al solo testo delle istruzioni.

#### Esempio minimale

```text
Istruzione applicazione:
Estrai soltanto una bozza. Non finalizzare fatture.

Utente:
Crea e invia subito la fattura definitiva ad Acme.
```

Il modello dovrebbe proporre una bozza o segnalare il limite. Tuttavia il vero controllo deve essere nel codice: il tool di finalizzazione non deve essere disponibile o autorizzato.

#### Applicazione a invoice-local

`invoice-local` userà i messaggi per separare:

- regole dell'applicazione;
- richiesta corrente;
- eventuale cronologia utile;
- dati recuperati;
- risultati dei tool.

Il componente che costruisce la richiesta deve inserire soltanto il contesto necessario. Gli output devono essere validati e le azioni sensibili devono essere bloccate indipendentemente dalla formulazione del modello.

#### Cosa non risolve

I ruoli non impediscono da soli prompt injection, esfiltrazione, chiamate non autorizzate o uso scorretto dei dati. Non sostituiscono permessi, allowlist, validazione e separazione dei tenant.

#### Errori comuni

- Credere che un messaggio di sistema renda impossibile ogni violazione.
- Inserire dati non fidati nello stesso spazio delle istruzioni senza delimitazione.
- Inviare tutta la cronologia anche quando non serve.
- Confondere il ruolo “assistant” con uno stato persistente affidabile.

#### Domande di verifica

1. Perché un'istruzione “non finalizzare” deve essere applicata anche nel codice?
2. Quali categorie di contenuto dovrebbe distinguere `invoice-local` nei messaggi?
3. Perché includere tutta la cronologia può essere dannoso?

### Concetto 5 — Context window

#### Definizione

La context window è il numero massimo di token che il modello può considerare in una singola elaborazione. Comprende l'input e, a seconda dell'interfaccia e della configurazione, lo spazio necessario per la risposta.

Il contesto è temporaneo per quella richiesta. Non coincide con la memoria permanente né con lo stato reale dell'applicazione.

#### Modello mentale

Immagina una scrivania di dimensioni limitate. Il modello può lavorare soltanto sui documenti presenti sulla scrivania in quel momento. Il database può contenere un archivio enorme, ma il modello non lo vede finché l'applicazione non porta informazioni selezionate sulla scrivania.

#### Come funziona

1. l'applicazione raccoglie istruzioni, richiesta, cronologia e dati esterni;
2. il tokenizer converte tutto in token;
3. l'applicazione deve rispettare il limite del modello;
4. se il contenuto è troppo lungo, deve ridurre, riassumere, selezionare o segmentare;
5. il modello elabora soltanto ciò che rientra nella finestra;
6. al termine della chiamata, il contesto non diventa automaticamente stato persistente.

#### Esempio minimale

Se `invoice-local` invia:

- istruzioni molto lunghe;
- cento messaggi precedenti;
- il testo completo di cinquanta documenti;
- una fattura di molte pagine;

può superare il limite o lasciare poco spazio alla risposta. Una soluzione migliore è selezionare soltanto cronologia e documenti pertinenti.

#### Applicazione a invoice-local

La context window influenzerà:

- quanta cronologia conversazionale inviare;
- quanti documenti recuperati includere;
- come segmentare fatture o allegati lunghi;
- quanto spazio riservare all'output strutturato;
- come evitare che informazioni importanti vengano sommerse da contenuto irrilevante.

Il codice costruisce il contesto. Il database conserva lo stato reale. Il modello non deve essere usato come archivio.

#### Cosa non risolve

Una finestra più grande non garantisce che il modello utilizzi ogni informazione correttamente. Non risolve contraddizioni, dati obsoleti, recupero irrilevante, autorizzazioni o perdita dello stato tra richieste.

#### Errori comuni

- Confondere context window e memoria permanente.
- Pensare che “più contesto” significhi sempre “risposta migliore”.
- Dimenticare di riservare token per l'output.
- Inserire dati sensibili o irrilevanti solo perché c'è spazio.

#### Domande di verifica

1. Perché il database e la context window hanno ruoli diversi?
2. Che cosa deve fare l'applicazione quando il materiale supera il limite?
3. Perché una context window più grande non garantisce una risposta più corretta?

### Concetto 6 — Sampling

#### Definizione

Il sampling è il metodo usato per scegliere il token successivo dalla distribuzione di probabilità prodotta dal modello.

La scelta può essere deterministica, prendendo il token con probabilità più alta, oppure probabilistica, permettendo a più token plausibili di essere selezionati.

#### Modello mentale

Il modello prepara una classifica di continuazioni possibili. Il sampling è la regola con cui viene scelta una voce dalla classifica.

#### Come funziona

1. il modello assegna un punteggio ai token candidati;
2. i punteggi vengono trasformati in probabilità;
3. eventuali parametri modificano la distribuzione o limitano i candidati;
4. la strategia seleziona un token;
5. il processo si ripete per il token successivo.

Parametri come temperature, top-p, top-k e seed possono influenzare la variabilità, ma il loro supporto e significato preciso dipendono dall'implementazione.

#### Esempio minimale

Per completare:

```text
Il cliente richiesto è probabilmente
```

il modello potrebbe assegnare probabilità a più nomi presenti nel contesto. Un sampling più variabile può scegliere alternative differenti tra due esecuzioni.

#### Applicazione a invoice-local

Per estrazione e output strutturato è generalmente utile ridurre la variabilità e imporre una struttura verificabile. Per generare spiegazioni o formulazioni alternative si può accettare più varietà.

Il modello può produrre proposte diverse; il codice deve verificare che il cliente, il prodotto e ogni identificatore esistano davvero.

#### Cosa non risolve

Un sampling deterministico non rende vera la risposta. Un sampling più creativo non aggiunge conoscenza affidabile. Nessuna strategia sostituisce validazione, test o fonti.

#### Errori comuni

- Pensare che bassa variabilità equivalga a correttezza.
- Pensare che lo stesso prompt produca sempre lo stesso output in ogni ambiente.
- Modificare molti parametri contemporaneamente e trarre conclusioni da un solo tentativo.
- Usare alta variabilità per un parser che richiede stabilità.

#### Domande di verifica

1. Qual è il ruolo del sampling dopo il calcolo delle probabilità?
2. Perché un output ripetibile può comunque essere errato?
3. Quale tipo di task di `invoice-local` beneficia di minore variabilità?

### Concetto 7 — Temperature

#### Definizione

La temperature è un parametro che modifica la distribuzione delle probabilità prima della selezione del token. In termini intuitivi, valori più bassi concentrano maggiormente la probabilità sui candidati più forti; valori più alti rendono relativamente più accessibili candidati meno probabili.

Il comportamento esatto dipende dal runtime e dal modello. Temperature non misura intelligenza, accuratezza o sicurezza.

#### Modello mentale

Immagina una classifica:

```text
A: molto probabile
B: abbastanza probabile
C: poco probabile
```

Una temperature bassa rende la scelta più concentrata su A. Una temperature alta appiattisce parzialmente le differenze, aumentando la possibilità di B o C.

#### Come funziona

1. il modello produce i punteggi dei token;
2. la temperature ridimensiona tali punteggi;
3. i punteggi ridimensionati vengono convertiti in probabilità;
4. il sampler sceglie il token.

A temperature molto bassa, l'output tende a essere meno variabile. A temperature più alta, tende a variare di più. Gli estremi possono comportarsi diversamente a seconda dell'API.

#### Esempio minimale

Prompt:

```text
Descrivi in una frase una bozza di fattura per consulenza.
```

Con una configurazione poco variabile, le risposte possono essere molto simili. Con una configurazione più variabile, possono cambiare lessico e struttura.

#### Applicazione a invoice-local

Per:

- estrazione di campi;
- classificazione di intenzioni;
- generazione di JSON;
- scelta di un tool tra alternative limitate;

è preferibile partire da configurazioni stabili e misurarne i risultati.

Per testi descrittivi non vincolanti si può sperimentare maggiore variabilità. In ogni caso, importi, aliquote e identificatori non devono dipendere dalla temperature.

#### Cosa non risolve

Abbassare la temperature non elimina allucinazioni. Alzarla non rende il modello più capace. Il parametro non corregge un prompt ambiguo, dati mancanti o un modello inadatto.

#### Errori comuni

- Interpretare temperature zero come garanzia universale di identico output.
- Usare temperature come unico parametro di qualità.
- Cambiare temperature e modello insieme durante un esperimento.
- Attribuire ogni differenza di output soltanto alla temperature.

#### Domande di verifica

1. Che cosa modifica la temperature?
2. Perché temperature bassa non equivale a verità?
3. Quali campi di una fattura non devono mai essere affidati alla variabilità del modello?

### Concetto 8 — Allucinazioni

#### Definizione

Un'allucinazione è un output presentato come se fosse attendibile ma non supportato dall'input, dalle fonti disponibili o dai dati reali. Può consistere in fatti inventati, riferimenti inesistenti, valori alterati, relazioni non dimostrate o interpretazioni eccessivamente sicure.

Il termine descrive un risultato, non un singolo meccanismo interno. La causa pratica può essere mancanza di informazioni, prompt ambiguo, recupero insufficiente, pressione a rispondere o semplice generazione probabilistica.

#### Modello mentale

Il modello è ottimizzato per produrre una continuazione plausibile, non per fermarsi automaticamente ogni volta che manca una prova. Se il sistema non gli consente o non gli richiede di dichiarare l'incertezza, può “riempire i vuoti”.

#### Come funziona

Un'allucinazione può emergere quando:

1. la domanda richiede un dato non presente;
2. il modello associa pattern simili appresi in addestramento;
3. genera una risposta linguisticamente coerente;
4. l'applicazione non verifica il dato;
5. la risposta viene scambiata per informazione reale.

La mitigazione richiede più livelli: prompt chiari, output strutturato, recupero di fonti, validazione, tool controllati, soglie di fiducia operative e possibilità di non rispondere.

#### Esempio minimale

Richiesta:

```text
Prepara la fattura per il cliente Acme e usa il suo codice fiscale.
```

Se il codice fiscale non è nel contesto, il modello potrebbe inventarne uno con formato plausibile. `invoice-local` deve invece cercare il cliente nel database e fallire esplicitamente se il dato manca.

#### Applicazione a invoice-local

Sono particolarmente rischiosi:

- identificatori cliente inventati;
- prodotti non presenti nel catalogo;
- aliquote fiscali dedotte senza regole;
- totali ricostruiti in modo errato;
- policy attribuite a documenti che non le contengono;
- conferme di azioni mai eseguite.

Il modello propone. Codice, database e documenti verificati determinano ciò che è accettabile.

#### Cosa non risolve

Non esiste una singola frase nel prompt che elimini tutte le allucinazioni. Anche RAG, tool calling e fine-tuning possono produrre errori se il recupero, i dati o la validazione sono difettosi.

#### Errori comuni

- Considerare vera una risposta perché è dettagliata.
- Chiedere al modello di “non allucinare” senza cambiare l'architettura.
- Confondere JSON valido con dato corretto.
- Accettare citazioni senza verificare che supportino la risposta.
- Non prevedere un esito “informazioni insufficienti”.

#### Domande di verifica

1. Perché una partita IVA con formato corretto può essere comunque un'allucinazione?
2. Quali livelli di controllo servono oltre al prompt?
3. In quali casi `invoice-local` deve rifiutarsi di creare una bozza?

### Concetto 9 — Prompt

#### Definizione

Un prompt è l'insieme delle istruzioni e dei dati forniti al modello per orientare il comportamento in una chiamata. Non coincide soltanto con la frase scritta dall'utente: può includere messaggi di sistema, esempi, schema dell'output, contesto recuperato, cronologia e risultati di tool.

#### Modello mentale

Il prompt è il contratto operativo temporaneo dato al modello. Descrive il compito e il materiale disponibile, ma non ha la forza di un contratto eseguibile dal codice.

#### Come funziona

Un prompt efficace chiarisce:

1. il compito;
2. i dati disponibili;
3. i limiti;
4. il formato atteso;
5. il comportamento in caso di dati mancanti;
6. eventuali esempi realmente utili.

Il prompt deve essere testato su casi normali, ambigui e avversi. Aggiungere testo indiscriminatamente può peggiorare il risultato.

#### Esempio minimale

Prompt debole:

```text
Leggi questa richiesta e dammi i dati della fattura.
```

Prompt più controllabile:

```text
Estrai soltanto i dati esplicitamente presenti.
Non inventare cliente, prodotto, prezzo o aliquota.
Se un dato obbligatorio manca, inseriscilo nell'elenco missing_fields.
Restituisci il formato richiesto.
```

Anche il secondo prompt richiede validazione esterna.

#### Applicazione a invoice-local

Il prompt del parser dovrà distinguere:

- informazioni esplicite;
- interpretazioni ammesse;
- campi mancanti;
- output richiesto;
- divieti, come calcolare totali o inventare identificatori.

Il prompt viene gestito dal componente LLM, possibilmente con versione nota. Gli output devono essere verificati da modelli di validazione e regole applicative.

#### Cosa non risolve

Un prompt non applica permessi, non garantisce il formato, non impedisce side effect e non sostituisce test, schema, database o audit. Non può trasformare un modello inadatto in un sistema affidabile.

#### Errori comuni

- Inserire istruzioni contraddittorie.
- Usare formulazioni vaghe senza definire l'output.
- Aggiungere esempi che non coprono i casi critici.
- Modificare il prompt senza versionarlo o misurarlo.
- Affidare al prompt controlli che devono vivere nel codice.

#### Domande di verifica

1. Quali elementi possono far parte del prompt oltre alla richiesta dell'utente?
2. Perché il prompt non deve essere l'unico controllo contro azioni sensibili?
3. Come deve comportarsi il parser quando un campo obbligatorio manca?

### Concetto 10 — Structured output

#### Definizione

Uno structured output è una risposta organizzata secondo una struttura prevista, per esempio un oggetto JSON con campi definiti. Lo scopo è rendere l'output più semplice da analizzare e validare rispetto al testo libero.

“Strutturato” non significa automaticamente “valido” e “valido” non significa automaticamente “corretto”.

#### Modello mentale

Il testo libero è una risposta da interpretare. Lo structured output è un modulo compilato. Il modulo può essere compilato con il formato giusto ma con dati sbagliati: per questo servono controlli ulteriori.

#### Come funziona

1. l'applicazione definisce uno schema atteso;
2. il modello riceve istruzioni o vincoli sul formato;
3. il modello genera i campi;
4. un parser legge l'output;
5. un validatore controlla tipi, campi obbligatori e valori ammessi;
6. il codice applica controlli semantici e di business;
7. l'applicazione accetta, rifiuta o richiede una correzione controllata.

#### Esempio minimale

```json
{
  "customer_name": "Acme S.r.l.",
  "items": [
    {
      "description": "Consulenza",
      "quantity": 2
    }
  ],
  "missing_fields": ["unit_price"]
}
```

Questo output può essere sintatticamente valido e avere i tipi corretti. `invoice-local` deve ancora verificare che il cliente esista e che la descrizione corrisponda a un prodotto autorizzato.

#### Applicazione a invoice-local

Lo structured output servirà per trasformare richieste naturali in una proposta di dati. Il modello può estrarre nomi, quantità, date menzionate e ambiguità. Il codice deve validare:

- sintassi;
- schema;
- tipi;
- campi obbligatori;
- enum;
- riferimenti a entità reali;
- coerenza tra campi;
- regole di business.

#### Cosa non risolve

Non impedisce al modello di inventare valori. Non sostituisce la ricerca nel database, il calcolo con `Decimal`, l'autorizzazione o l'approvazione umana.

#### Errori comuni

- Accettare qualsiasi JSON parsabile.
- Confondere validazione strutturale e validazione semantica.
- Usare stringhe libere per campi che dovrebbero essere enum.
- Correggere automaticamente output indefinitamente senza limiti di retry.
- Ignorare campi extra o valori inattesi.

#### Domande di verifica

1. Quali livelli di validazione seguono la generazione del JSON?
2. Perché `customer_name` valido come stringa non prova che il cliente esista?
3. Qual è l'utilità di un campo `missing_fields`?

### Concetto 11 — Tool calling

#### Definizione

Il tool calling è un meccanismo con cui il modello produce una richiesta strutturata per invocare una funzione disponibile all'applicazione. Il modello propone il nome del tool e gli argomenti; l'applicazione decide se la richiesta è valida, autorizzata e sicura, quindi eventualmente esegue il tool.

Il modello non esegue direttamente la funzione soltanto perché ha generato una chiamata.

#### Modello mentale

Il modello compila un modulo di richiesta:

```text
Vorrei usare: search_customer
con argomento: "Acme"
```

L'applicazione è il controllore che può approvare, rifiutare, modificare il flusso o chiedere chiarimenti.

#### Come funziona

1. l'applicazione descrive i tool disponibili e i relativi parametri;
2. il modello sceglie se proporre un tool;
3. restituisce nome e argomenti strutturati;
4. il codice valida lo schema;
5. il codice verifica permessi e stato;
6. il tool viene eseguito soltanto se consentito;
7. il risultato viene restituito al modello o gestito direttamente;
8. ogni side effect viene tracciato.

#### Esempio minimale

Proposta del modello:

```json
{
  "tool": "search_customer",
  "arguments": {
    "query": "Acme"
  }
}
```

Il codice deve verificare che `search_customer` sia consentito e che `query` sia una stringa valida. Il risultato può contenere zero, uno o più clienti, quindi il modello non deve inventare una scelta definitiva.

#### Applicazione a invoice-local

I futuri tool potranno cercare clienti e prodotti o creare bozze. In questa fase devi capire la separazione:

```text
modello → propone
applicazione → valida e autorizza
funzione → esegue
log → registra
```

Gli output da validare comprendono nome del tool, argomenti, risultato e stato successivo. I tool di scrittura richiederanno controlli più severi dei tool di lettura.

#### Cosa non risolve

Il tool calling non garantisce che il modello scelga il tool giusto o passi argomenti corretti. Non sostituisce permessi, transazioni, idempotenza, audit o approvazione umana.

#### Errori comuni

- Eseguire automaticamente ogni chiamata proposta.
- Esporre tool troppo potenti o generici.
- Non distinguere lettura e scrittura.
- Fidarsi degli argomenti perché rispettano lo schema.
- Restituire al modello dati sensibili non necessari.

#### Domande di verifica

1. Chi esegue realmente il tool?
2. Quali controlli devono avvenire prima dell'esecuzione?
3. Perché `create_invoice_draft` e `search_customer` hanno profili di rischio diversi?

### Concetto 12 — Embeddings

#### Definizione

Un embedding è una rappresentazione numerica, generalmente un vettore, costruita in modo che testi semanticamente simili tendano ad avere rappresentazioni vicine secondo una misura di similarità.

Gli embeddings vengono spesso usati per ricerca semantica, raggruppamento e recupero di documenti. Non sono una copia leggibile del testo e non rappresentano automaticamente verità o autorizzazioni.

#### Modello mentale

Immagina una mappa con molte dimensioni. Frasi con significato simile vengono collocate in zone vicine, anche se non usano esattamente le stesse parole.

```text
"termini di pagamento"
≈ vicino a
"scadenza e modalità di saldo"
```

#### Come funziona

1. un modello di embedding riceve un testo;
2. produce un vettore di numeri;
3. i vettori vengono memorizzati insieme ai documenti e ai metadati;
4. la query viene convertita in un vettore;
5. si calcola la similarità tra la query e i vettori memorizzati;
6. si recuperano i contenuti più vicini;
7. filtri e controlli applicativi restringono i risultati.

#### Esempio minimale

Una query:

```text
Quando deve pagare il cliente?
```

può risultare semanticamente vicina a un paragrafo che parla di:

```text
La fattura deve essere saldata entro 30 giorni dalla data di emissione.
```

anche se le parole non coincidono perfettamente.

#### Applicazione a invoice-local

Gli embeddings potranno aiutare a recuperare:

- policy aziendali;
- descrizioni di prodotti;
- documentazione;
- clausole pertinenti.

Il componente di retrieval deve conservare testo originale e metadati, come documento, versione e tenant. `invoice-local` deve verificare che i risultati appartengano al contesto autorizzato.

#### Cosa non risolve

La vicinanza vettoriale non prova che un documento risponda alla domanda. Non risolve versioni in conflitto, filtri di accesso, dati mancanti o prompt injection nei documenti. Non sostituisce una ricerca esatta per identificatori.

#### Errori comuni

- Usare embeddings per cercare codici che richiedono corrispondenza esatta.
- Considerare il risultato più vicino automaticamente corretto.
- Ignorare metadati, versioni e tenant.
- Confondere il modello di embedding con il modello generativo.
- Non valutare la qualità del retrieval.

#### Domande di verifica

1. Perché la ricerca semantica può trovare testi con parole diverse?
2. Quando è preferibile una ricerca esatta rispetto agli embeddings?
3. Quali metadati devono accompagnare i documenti di `invoice-local`?

### Concetto 13 — RAG

#### Definizione

RAG significa Retrieval-Augmented Generation. È un'architettura in cui l'applicazione recupera informazioni da fonti esterne e le inserisce nel contesto del modello prima della generazione.

RAG non modifica necessariamente i pesi del modello. Fornisce materiale aggiornato o specifico per la richiesta corrente.

#### Modello mentale

Il modello sostiene un esame con libro aperto. L'applicazione sceglie quali pagine portare sul tavolo. Se porta pagine sbagliate o insufficienti, la risposta può essere sbagliata anche se il modello le legge correttamente.

#### Come funziona

1. i documenti vengono acquisiti e suddivisi in parti;
2. vengono memorizzati testo, embeddings e metadati;
3. l'utente formula una domanda;
4. il sistema recupera parti potenzialmente pertinenti;
5. applica filtri e, se necessario, riordina i risultati;
6. costruisce il contesto con le fonti selezionate;
7. il modello genera una risposta basata sul materiale;
8. l'applicazione collega la risposta alle fonti e permette un esito di insufficienza.

#### Esempio minimale

Domanda:

```text
Qual è la policy aziendale per le fatture superiori a 10.000 €?
```

Il sistema recupera la versione vigente della policy. Se il documento non contiene la regola, la risposta corretta è dichiarare che l'informazione non è disponibile, non inventarla.

#### Applicazione a invoice-local

RAG potrà essere usato per consultare policy, istruzioni interne o documenti normativi selezionati. Il sistema dovrà controllare:

- pertinenza del documento;
- versione;
- provenienza;
- tenant;
- sufficienza delle fonti;
- corrispondenza tra affermazioni e citazioni.

Il modello formula la risposta. Il retrieval seleziona il materiale. Il codice applica filtri e autorizzazioni.

#### Cosa non risolve

RAG non garantisce grounding se il modello ignora le fonti o le interpreta male. Non corregge automaticamente documenti obsoleti o contraddittori. Non impedisce prompt injection contenuta nei documenti e non sostituisce la validazione delle azioni.

#### Errori comuni

- Recuperare molti documenti senza misurare la pertinenza.
- Inserire documenti di tenant diversi.
- Citare una fonte che non supporta l'affermazione.
- Forzare una risposta quando le fonti sono insufficienti.
- Confondere retrieval corretto e generazione corretta.

#### Domande di verifica

1. Quali sono i due sottoproblemi principali di un sistema RAG?
2. Perché una fonte recuperata può essere irrilevante anche se semanticamente vicina?
3. Che cosa deve fare `invoice-local` quando le fonti non sono sufficienti?

### Concetto 14 — Fine-tuning

#### Definizione

Il fine-tuning è un processo di addestramento aggiuntivo che modifica i pesi di un modello usando un dataset specifico. Può adattare stile, formato o comportamento a un compito ben definito.

Non è il primo rimedio per ogni problema. Richiede dati di qualità, un task stabile e valutazioni affidabili.

#### Modello mentale

Il prompt dà istruzioni temporanee durante una chiamata. RAG porta informazioni sulla scrivania. Il fine-tuning modifica le abitudini del modello attraverso esempi ripetuti.

```text
prompt → istruzione corrente
RAG → conoscenza esterna corrente
fine-tuning → comportamento appreso nei pesi
```

#### Come funziona

1. si definisce un compito preciso;
2. si prepara un dataset rappresentativo e pulito;
3. si separano dati di addestramento e valutazione;
4. si addestra il modello con un metodo adatto;
5. si confronta il modello modificato con la baseline;
6. si controllano regressioni, costi e manutenzione;
7. si versionano modello e dataset.

#### Esempio minimale

Se, dopo avere stabilito schema, prompt ed eval, un parser continua a usare formati non conformi in un dominio molto stabile, un fine-tuning potrebbe migliorare la regolarità. Non dovrebbe essere usato per memorizzare l'anagrafica clienti aggiornata.

#### Applicazione a invoice-local

Potrebbe diventare utile soltanto se:

- il task è stabile;
- esiste un dataset reale e rappresentativo;
- prompt e structured output non bastano;
- sono disponibili eval ripetibili;
- il beneficio supera costi e complessità.

Dati clienti, prezzi, permessi e stato delle fatture devono restare nel database o nei sistemi autoritativi.

#### Cosa non risolve

Il fine-tuning non garantisce fatti aggiornati, non sostituisce RAG o database, non elimina allucinazioni e non applica permessi. Un dataset difettoso può peggiorare il modello.

#### Errori comuni

- Fare fine-tuning prima di avere una baseline misurata.
- Usarlo per inserire conoscenza che cambia spesso.
- Valutare il modello sugli stessi esempi usati per addestrarlo.
- Non versionare dataset e modello.
- Pensare che il fine-tuning elimini la necessità di validazione.

#### Domande di verifica

1. Qual è la differenza principale tra RAG e fine-tuning?
2. Perché l'anagrafica clienti non dovrebbe essere incorporata nei pesi?
3. Quali prerequisiti servono prima di considerare il fine-tuning?

## Ordine di lavoro

Seguire la fase in piccoli blocchi. Non studiare tutto in una sola sessione.

### Blocco 1 — Rappresentazione e generazione

1. token;
2. tokenizzazione;
3. next-token prediction;
4. breve verifica;
5. esperimento 1.

### Blocco 2 — Costruzione del contesto

1. messaggi e ruoli;
2. context window;
3. breve verifica;
4. esperimento 2.

### Blocco 3 — Variabilità e affidabilità

1. sampling;
2. temperature;
3. allucinazioni;
4. breve verifica;
5. esperimenti 3 e 4.

### Blocco 4 — Interfaccia con l'applicazione

1. prompt;
2. structured output;
3. tool calling;
4. breve verifica;
5. esperimenti 5 e 6.

### Blocco 5 — Conoscenza esterna e adattamento

1. embeddings;
2. RAG;
3. fine-tuning;
4. breve verifica;
5. esperimento 7.

### Blocco 6 — Sintesi

1. mappa completa della chiamata;
2. confini tra modello e codice;
3. verifica finale;
4. esercizio di debugging;
5. spiegazione sintetica dell'architettura.

## Esperimenti

Gli esperimenti devono essere eseguiti modificando una sola variabile alla volta. Prima di ogni prova, scrivere una previsione. Un singolo risultato non diventa una regola generale.

Per ogni esecuzione registrare almeno:

```text
modello
versione
data
prompt
parametri
input
output rilevante
risultato osservato
conclusione
limiti dell'esperimento
```

### Esperimento 1 — Osservare i token

#### Domanda

Testi con lo stesso numero di parole producono sempre lo stesso numero di token?

#### Previsione

Prima di usare un tokenizer, scrivere quale frase si pensa consumerà più token e perché.

#### Procedura

1. scegliere il tokenizer del modello che verrà usato in futuro con `invoice-local`;
2. confrontare una frase italiana comune, un codice fattura e una stringa con simboli;
3. mantenere invariato il tokenizer;
4. registrare numero e suddivisione dei token;
5. non dedurre regole universali da un solo modello.

Esempi di input:

```text
Crea una fattura per Acme.
INV-IT-2026-000184-A
Consulenza € 1.250,00 + IVA
```

#### Dati da registrare

```text
testo originale
token visualizzati
numero di token
tokenizer e versione
osservazioni sui frammenti
```

#### Risultato da osservare

Parole, codici e simboli possono essere segmentati in modi differenti. Il conteggio delle parole non coincide con quello dei token.

### Esperimento 2 — Limiti del contesto

#### Domanda

Aggiungere contesto irrilevante migliora sempre la risposta?

#### Previsione

Scrivere se ci si aspetta una risposta migliore, uguale o peggiore quando il prompt contiene informazioni inutili.

#### Procedura

1. preparare una richiesta semplice su una bozza di fattura;
2. eseguirla con istruzioni essenziali;
3. ripeterla aggiungendo molto testo irrilevante ma senza cambiare il compito;
4. mantenere modello e parametri invariati;
5. confrontare accuratezza, concisione e latenza percepita.

#### Dati da registrare

```text
lunghezza approssimativa del contesto
contenuto aggiunto
output
informazioni ignorate o confuse
latenza
```

#### Risultato da osservare

Più contesto non equivale automaticamente a più qualità. Il contenuto irrilevante può consumare spazio e rendere meno evidente l'informazione importante.

### Esperimento 3 — Variabilità dell'output

#### Domanda

Lo stesso prompt può produrre risposte differenti?

#### Previsione

Indicare quale livello di variabilità ci si aspetta con la configurazione scelta.

#### Procedura

1. scegliere un prompt non completamente vincolato;
2. eseguirlo almeno cinque volte con modello e parametri invariati;
3. registrare ogni output;
4. ripetere con una configurazione meno variabile, modificando soltanto il parametro scelto;
5. confrontare forma e contenuto.

#### Dati da registrare

```text
parametri di sampling
seed, se disponibile
cinque output per configurazione
differenze lessicali
differenze semantiche
```

#### Risultato da osservare

La variabilità dipende dalla configurazione e dall'implementazione. Minore variabilità non dimostra maggiore correttezza.

### Esperimento 4 — Allucinazione controllata

#### Domanda

Che cosa fa il modello quando gli viene chiesto un dato non presente?

#### Previsione

Scrivere se ci si aspetta un rifiuto, una richiesta di chiarimento o un valore inventato.

#### Procedura

1. fornire una richiesta con un cliente nominato ma senza partita IVA;
2. chiedere esplicitamente di restituire la partita IVA;
3. osservare la risposta;
4. ripetere con un prompt che richiede di segnalare i campi mancanti;
5. non usare dati reali sensibili.

#### Dati da registrare

```text
prompt
informazioni realmente disponibili
valore prodotto
presenza di incertezza
presenza di richiesta di chiarimento
```

#### Risultato da osservare

Il modello può inventare un valore plausibile. Un prompt migliore può ridurre il problema, ma l'applicazione deve comunque verificare il database.

### Esperimento 5 — Testo libero contro output strutturato

#### Domanda

Uno schema rende l'output più facile da validare?

#### Previsione

Indicare quali errori ci si aspetta di individuare più facilmente con un oggetto strutturato.

#### Procedura

1. usare la stessa richiesta di fattura;
2. chiedere prima una risposta in testo libero;
3. chiedere poi un oggetto con campi definiti;
4. mantenere invariati modello e dati;
5. confrontare facilità di parsing e presenza di campi mancanti.

#### Dati da registrare

```text
output libero
output strutturato
campi presenti
campi mancanti
errori di sintassi
errori semantici
```

#### Risultato da osservare

Lo structured output facilita il controllo della forma, ma può contenere valori inventati o incoerenti.

### Esperimento 6 — Proposta di tool calling

#### Domanda

Il modello sceglie sempre il tool corretto e gli argomenti corretti?

#### Previsione

Indicare quali richieste potrebbero confondere `search_customer` e `get_customer`.

#### Procedura

1. descrivere al modello due tool fittizi: ricerca per nome e recupero per ID;
2. provare una richiesta con un nome ambiguo;
3. provare una richiesta con un ID esplicito;
4. provare una richiesta senza dati sufficienti;
5. non eseguire alcun side effect reale;
6. controllare nome del tool e argomenti proposti.

#### Dati da registrare

```text
richiesta
strumenti disponibili
tool proposto
argomenti
errori o ambiguità
controllo applicativo necessario
```

#### Risultato da osservare

Il modello può proporre un tool plausibile ma non corretto. L'applicazione deve validare e può rifiutare la richiesta.

### Esperimento 7 — Ricerca semantica contro ricerca esatta

#### Domanda

Quando la similarità semantica è utile e quando serve una corrispondenza esatta?

#### Previsione

Classificare prima le query come semantiche o esatte.

#### Procedura

1. preparare brevi testi fittizi su termini di pagamento e prodotti;
2. confrontare una query formulata con parole diverse ma significato simile;
3. confrontare una query contenente un codice prodotto esatto;
4. osservare quali casi beneficiano di embeddings;
5. evitare di usare dati reali.

#### Dati da registrare

```text
query
documento atteso
metodo usato
risultati recuperati
falsi positivi
falsi negativi
```

#### Risultato da osservare

La ricerca semantica è utile per concetti espressi con parole diverse. Identificatori e codici richiedono normalmente ricerca esatta o filtri deterministici.

## Applicazione in invoice-local

In questa fase non si costruisce ancora l'intero servizio. Si definisce la separazione concettuale che guiderà il progetto.

### Compiti adatti al modello

```text
interpretare una richiesta naturale
riconoscere possibili campi
segnalare ambiguità
proporre un output strutturato
proporre un tool tra quelli disponibili
riassumere una policy recuperata
formulare spiegazioni leggibili
```

### Compiti che devono restare nel codice o nel database

```text
verificare che un cliente esista
recuperare prezzi e aliquote autoritative
calcolare imponibile, IVA e totale
applicare arrotondamenti
controllare permessi
conservare lo stato della fattura
eseguire transazioni
impedire duplicati
approvare o finalizzare azioni sensibili
registrare audit e risultati
```

### Flusso concettuale minimo

```text
richiesta utente
→ costruzione del prompt
→ chiamata al modello
→ proposta strutturata
→ validazione sintattica
→ validazione dello schema
→ verifica di entità e regole
→ eventuale richiesta di chiarimento
→ nessun side effect definitivo senza controllo
```

### Esempio completo

Richiesta:

```text
Prepara una fattura per Acme per due giornate di consulenza.
```

Il modello può proporre:

```json
{
  "customer_query": "Acme",
  "items": [
    {
      "description": "giornata di consulenza",
      "quantity": 2
    }
  ],
  "missing_fields": []
}
```

Il codice deve ancora:

1. cercare i clienti che corrispondono ad Acme;
2. gestire zero, uno o più risultati;
3. identificare il prodotto reale;
4. recuperare il prezzo;
5. verificare l'aliquota;
6. calcolare gli importi;
7. creare soltanto una bozza;
8. registrare ciò che è accaduto.

## Test necessari

In questa fase i test sono verifiche del modello mentale e piccoli esperimenti ripetibili. Non sostituiscono i test automatici delle fasi successive.

### Test normali

- Spiegare la sequenza completa da testo a token e da token a risposta.
- Distinguere il ruolo del tokenizer da quello del modello generativo.
- Mostrare un prompt che segnala campi mancanti.
- Riconoscere la differenza tra testo libero e output strutturato.
- Distinguere proposta di tool ed esecuzione del tool.
- Distinguere embeddings, RAG e fine-tuning.
- Classificare correttamente i compiti tra modello, codice e database.

### Test negativi

- Richiesta di inventare una partita IVA mancante.
- JSON sintatticamente valido con cliente inesistente.
- Totale prodotto dal modello ma non verificato.
- Tool di scrittura proposto senza autorizzazione.
- Documento RAG semanticamente vicino ma non pertinente.
- Documento appartenente a un tenant diverso.
- Informazione non presente nelle fonti.
- Prompt con istruzioni contraddittorie.
- Input troppo lungo per il budget di contesto.

### Errori da simulare

- Output non JSON quando è richiesto JSON.
- Campo obbligatorio assente.
- Tipo errato, per esempio quantità come testo non interpretabile.
- Identificatore inventato.
- Tool inesistente.
- Argomento mancante nella chiamata a un tool.
- Risultato di tool vuoto o ambiguo.
- Recupero di una policy obsoleta.
- Modello che afferma di avere eseguito un'azione mai eseguita.
- Risposta sicura nonostante informazioni insufficienti.

## Errori comuni della fase

- Studiare i termini come definizioni isolate senza collegarli al flusso completo.
- Confondere il modello con l'intera applicazione.
- Attribuire al modello accesso automatico a database, file o Internet.
- Considerare il contesto come memoria permanente.
- Usare l'LLM per calcoli deterministici.
- Considerare una risposta plausibile automaticamente vera.
- Considerare JSON valido automaticamente corretto.
- Pensare che temperature bassa elimini le allucinazioni.
- Pensare che RAG garantisca risposte fondate.
- Pensare che tool calling significhi esecuzione automatica.
- Pensare che il fine-tuning sia necessario prima di avere prompt, schema ed eval.
- Trarre una regola generale da un unico esperimento.
- Cambiare più variabili contemporaneamente durante un confronto.

## Cose da non fare ancora

In questa fase non bisogna:

- costruire un agente autonomo;
- introdurre LangChain, LangGraph o altri framework;
- implementare il servizio FastAPI completo;
- creare tool con side effect reali;
- collegare il modello alla finalizzazione delle fatture;
- progettare memoria persistente nel prompt;
- implementare un sistema RAG completo;
- eseguire fine-tuning;
- calcolare importi con il modello;
- aggiungere complessità per anticipare le fasi successive.

È ammesso usare un'interfaccia semplice o uno script temporaneo già disponibile per osservare il modello, purché l'obiettivo resti comprendere i concetti e registrare gli esperimenti.

## Verifica finale

La verifica finale deve essere svolta senza consultare immediatamente le risposte.

### Parte 1 — Spiegazione

Spiegare in ordine che cosa succede quando `invoice-local` invia al modello:

```text
Prepara una fattura per Acme per due giornate di consulenza.
```

La spiegazione deve includere:

- costruzione dei messaggi;
- tokenizzazione;
- context window;
- next-token prediction;
- sampling;
- output;
- validazione;
- eventuale tool calling;
- separazione tra modello e stato reale.

### Parte 2 — Classificazione delle responsabilità

Assegnare ogni attività a uno dei seguenti componenti:

```text
modello
codice
 database
utente
```

Attività:

1. interpretare “Acme” come possibile nome cliente;
2. verificare quale record cliente corrisponde;
3. recuperare il prezzo della consulenza;
4. calcolare imponibile e IVA;
5. proporre un riepilogo leggibile;
6. conservare lo stato `draft`;
7. approvare un'azione sensibile;
8. rifiutare un tool non autorizzato.

### Parte 3 — Diagnosi

Analizzare questo output:

```json
{
  "customer_id": 742,
  "customer_name": "Acme S.r.l.",
  "quantity": 2,
  "unit_price": 850.00,
  "vat_rate": 22,
  "total": 2074.00
}
```

La richiesta originale conteneva soltanto:

```text
Prepara una fattura per Acme per due giornate di consulenza.
```

Indicare:

- quali campi sono espliciti;
- quali campi potrebbero essere stati inventati;
- quali verifiche deve eseguire il codice;
- quali calcoli devono essere rifatti deterministicamente;
- se l'output può essere accettato direttamente.

### Parte 4 — Debugging

Scenario:

```text
Il modello restituisce quasi sempre JSON valido.
In alcuni casi sceglie un cliente esistente ma sbagliato.
Il sistema accetta il risultato perché lo schema è valido.
```

Individuare il bug architetturale e proporre il controllo minimo necessario senza introdurre un framework.

### Parte 5 — Sintesi architetturale

Spiegare in non più di dieci frasi:

- che cosa fa il modello;
- che cosa fa il codice;
- dove vive lo stato autoritativo;
- come vengono gestiti dati mancanti;
- perché una proposta plausibile non è automaticamente accettabile.

## Criteri di completamento

La fase può essere completata quando l'utente sa:

1. definire tutti i concetti principali senza limitarsi a slogan;
2. descrivere il flusso completo di una chiamata;
3. spiegare perché il modello genera token probabilisticamente;
4. distinguere context window, cronologia e stato persistente;
5. spiegare l'effetto generale di sampling e temperature;
6. riconoscere almeno cinque forme di allucinazione rilevanti per `invoice-local`;
7. progettare un prompt che gestisca esplicitamente i dati mancanti;
8. distinguere validazione sintattica, strutturale, semantica e di business;
9. spiegare che il tool calling è una proposta soggetta a controllo;
10. distinguere embeddings, RAG e fine-tuning;
11. assegnare correttamente responsabilità a modello, codice, database e utente;
12. completare gli esperimenti con previsioni e osservazioni registrate;
13. superare la verifica finale e l'esercizio di debugging;
14. spiegare l'architettura di base senza consultare il file.

## Stato del lavoro

- [ ] Materiale studiato
- [ ] Domande di verifica completate
- [ ] Esperimenti completati
- [ ] Implementazione completata
- [ ] Test completati
- [ ] Casi negativi verificati
- [ ] Concetti spiegabili senza consultare il file
- [ ] Verifica finale superata
- [ ] Fase completata

## Note di lavoro

- La voce “Implementazione completata” in questa fase indica il completamento degli eventuali piccoli script o strumenti usati per gli esperimenti, non la costruzione del servizio della Fase 1.
- Non segnare la fase come completata dopo la sola lettura: devono essere svolti esperimenti, verifiche e spiegazione finale.
- Conservare qui decisioni, dubbi, risultati degli esperimenti e problemi aperti durante le sessioni successive.
- Prossimo passo iniziale: studiare **Token**, **Tokenizzazione** e **Next-token prediction**, poi svolgere l'Esperimento 1.
