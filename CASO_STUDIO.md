# Caso studio — Prompt engineering applicato all'estrazione di dati finanziari

*Sessione di pratica strutturata su Claude, seguendo i 9 capitoli del tutorial ufficiale Anthropic di prompt engineering, applicati a comunicati stampa finanziari reali (NVIDIA, Enel) invece che a esempi didattici generici.*

> **Scope, in chiaro.** Questo è un esercizio di pratica di una singola sessione, non un sistema in produzione né un deliverable per un cliente. I testi di input sono sintesi di fonti giornalistiche (Soldionline, Websim), non i comunicati ufficiali integrali. Il lavoro è di sviluppo assistito da AI: io ho progettato gli esperimenti, scritto e iterato i prompt, individuato gli errori e validato gli output; il testo è prodotto dal modello. Le aziende del settore energy/finance sono state scelte perché sono gli stessi settori dei clienti enterprise annunciati da Anthropic per la sede di Milano (es. Enel, Generali).

---

## Obiettivo

Trasformare un comunicato stampa finanziario in dati strutturati (JSON) **accurati, riproducibili e onesti sui propri limiti**, costruendo il prompt non in un colpo solo ma per iterazioni motivate — partendo da una richiesta vaga e arrivando a un prompt di livello "produzione", documentando il perché di ogni modifica.

## Metodo: la progressione del prompt (v1 → v2 → v3)

**v1 — grezzo.** Una riga: *"Analizza l'ultimo comunicato ed estrai le informazioni chiave in JSON."* Funziona solo grazie alla capacità di base del modello: riferimento ambiguo ("l'ultimo"), schema deciso da Claude e non da me, nessuna variabile riutilizzabile, nessuna regola sui dati mancanti. Due esecuzioni su testi diversi producono JSON non comparabili.

**v2 — strutturato.** Variabile `{{PRESS_RELEASE}}` separata dalle istruzioni (cap. 4), schema fisso a 9 campi nominati (cap. 5), scratchpad di ragionamento esplicito (cap. 6), regola `null` per i dati mancanti, esempio few-shot prodotto dalla funzione "Genera prompt" del Workbench (cap. 7), tag `<json_output>` per il parsing a valle.

**v3 — produzione.** Aggiunto un ruolo esplicito nel system prompt (cap. 3, "Applied AI Consultant" ancorato allo scenario reale), il campo consensus, il campo strutturale dei dati mancanti, le note metodologiche, e una regola anti-allucinazione rinforzata che vieta anche l'inferenza per analogia.

## Cosa ho scoperto (i tre esperimenti che contano)

### 1. Un gap di dominio reale: il consensus
Testando la v2 su Enel ho notato che lo schema **non aveva un campo per il valore atteso/consensus degli analisti**. In finanza è un dato cruciale: il mercato reagisce al *beat/miss* sulle stime, non al valore assoluto. Il comunicato conteneva sia l'EBITDA reale (6.003 mln) sia l'atteso (5.948 mln), ma l'output li schiacciava in una variazione percentuale, perdendo l'informazione più importante. Ho aggiunto i campi `valore_atteso_consensus` e `scostamento_da_consensus_percentuale` e ho rivalidato. È iterazione guidata da un errore concreto, non da teoria.

### 2. L'effetto dello scratchpad, misurato (cap. 6)
Ho impostato un confronto controllato: stesso modello, **stesso identico input**, stessa domanda ("i risultati Enel Q1 sono migliori o peggiori delle attese?"). Unica variabile: la presenza di uno spazio di ragionamento esplicito prima della risposta.

- **Senza scratchpad** → giudizio *"IN LINEA"*, ma incoerente con la sua stessa motivazione, che descriveva entrambe le metriche come superiori alle attese.
- **Con scratchpad** → giudizio *"SOPRA le attese"*, coerente coi dati; in più ha distinto correttamente un dato senza consensus (ricavi) da uno valutabile, e ha pesato il debito netto e la guidance confermata come fattori separati.

Stesso modello, stessi dati: la qualità del giudizio è cambiata in modo verificabile **solo per effetto della struttura del prompt**. Questo dimostra di capire *perché* la tecnica funziona, non solo di saperla applicare.

### 3. Affidabilità progettata, non sperata (cap. 8)
Ho costruito un test-trabocchetto: ho chiesto il fatturato Enel nel mercato spagnolo, **dato assente dal testo**. Claude ha rifiutato di inventare, ha spiegato la causa strutturale (livello di aggregazione del documento) e ha indicato dove reperire il dato reale. Ho poi reso questo comportamento un requisito **strutturale** del prompt v3, col campo `dati_non_disponibili_nel_testo`, invece di affidarlo a una domanda caso per caso.

## Il prompt finale (v3)

```text
SYSTEM:
Sei un Applied AI Consultant che supporta team Investor Relations e analisti
finanziari di aziende enterprise nei settori regolamentati (finance, energy,
life sciences). Trasforma comunicati stampa finanziari in dati strutturati,
accurati e pronti per sistemi aziendali a valle. Precisione e onestà sui
limiti dei dati disponibili sono più importanti della completezza apparente.

USER:
Analizza il seguente comunicato e produci un'estrazione strutturata.
<press_release>{{PRESS_RELEASE}}</press_release>

Prima dell'output, usa <analisi> per: (1) identificare ogni metrica chiave;
(2) per ciascuna, verificare se il testo riporta un valore atteso/consensus e
calcolare lo scostamento; (3) segnalare esplicitamente quali informazioni
standard NON sono nel testo, invece di ometterle.

Poi fornisci <json_output> con schema fisso { nome_azienda, data_comunicato,
periodo_finanziario, metriche[ {nome, valore, valuta, valore_atteso_consensus,
scostamento_da_consensus_percentuale, variazione_yoy_percentuale} ],
guidance_futura, annunci_strategici[], citazioni_executive,
dati_non_disponibili_nel_testo[], note_metodologiche }.

Usa null per qualunque campo non determinabile. Non inferire o stimare valori
non esplicitamente presenti, nemmeno per analogia.
```

## Il limite che resta aperto (onestà)

Nell'output finale, il campo `variazione_yoy_percentuale` dell'EBITDA è stato popolato con `0.0`, mentre il testo diceva solo *"sostanzialmente stabile su base annua"* — un'affermazione qualitativa, non un numero. Claude ha numerizzato un giudizio qualitativo, violando lievemente la propria regola anti-inferenza. Correzione individuata: aggiungere una regola che vieti esplicitamente di trasformare affermazioni qualitative in numeri, o un controllo umano finale su questo tipo di conversione. **Lo lascio scritto perché un limite documentato vale più di un risultato presentato come perfetto.**

## Cosa dimostra questo lavoro

Progettazione iterativa di prompt motivata da errori reali; separazione dati/istruzioni e output strutturato per l'integrazione a valle; uso del ragionamento esplicito come leva di qualità *misurata*; affidabilità progettata contro le allucinazioni; e calibrazione onesta sui limiti. Il tutto applicato a un dominio reale e verificabile, non a esempi-giocattolo.
