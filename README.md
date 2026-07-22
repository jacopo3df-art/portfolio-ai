# Portfolio — Prompt Engineering & Meta-Prompting

Raccolta di esercizi pratici di prompt engineering e meta-prompting, costruiti seguendo il [tutorial ufficiale di prompt engineering di Anthropic](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview) e applicati a dati reali.

**Nota sullo scope.** Questo non è un repository di codice — è un repository di metodo. Contiene i prompt che ho scritto, iterato e testato, con le decisioni documentate passo per passo. È un tassello di un percorso più ampio verso lo sviluppo software, che sto costruendo in parallelo.

## Contenuto

### `CASO_STUDIO.md`
Caso studio di prompt engineering: un prompt di estrazione dati per analisi finanziaria iterato da una versione grezza a una di livello produzione, applicato a comunicati stampa reali di NVIDIA ed Enel. Include tre esperimenti documentati — un gap di dominio individuato e corretto, un confronto controllato con/senza ragionamento esplicito, un test anti-allucinazione progettato — e un limite residuo dichiarato onestamente, non nascosto.

### `Meta-prompt 1 — Research Builder.md`
Un meta-prompt che genera prompt di ricerca web strutturati e rigorosi: obiettivo, sotto-domande, ambito, metodo evidence-first, citazioni obbligatorie.

### `Meta-prompt 2 — Claude Code Builder.md`
Un meta-prompt che genera specifiche di sviluppo software strutturate: stack, architettura, ambito del task, "prima il piano poi il codice", criteri di accettazione.

### `Meta-prompt 3 — Analista Finanziario.md`
Un meta-prompt che genera prompt di ricerca finanziaria con ragionamento esplicito, regole anti-allucinazione e formato di output strutturato. Usa un portafoglio di esempio generico a scopo puramente illustrativo.

## Cos'è un meta-prompt

Un meta-prompt è un prompt che, invece di eseguire direttamente un compito, genera un altro prompt più rigoroso per quel compito. L'idea di fondo: invece di riscrivere ogni volta un prompt da zero, si costruisce e si affina uno strumento riutilizzabile. Se il risultato finale non convince, si corregge il meta-prompt — non il singolo output — così ogni miglioramento vale per tutti gli usi futuri.

## Metodo applicato

Ogni asset in questo repository applica gli stessi principi, tratti dal tutorial ufficiale Anthropic: ruolo esplicito, separazione dati/istruzioni con tag XML, esempi (few-shot), ragionamento step-by-step prima della risposta, formato di output vincolato, regole esplicite anti-allucinazione.

---

Jacopo Frasconi · [LinkedIn](https://www.linkedin.com/in/jacopofrasconi-b185aa3b0)
