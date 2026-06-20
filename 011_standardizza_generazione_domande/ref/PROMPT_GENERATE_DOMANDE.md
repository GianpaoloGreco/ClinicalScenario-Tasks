# Prompt: Generazione Domande (solo domande, catalog gia' pronto)

Questo prompt va inviato a Claude Opus DOPO che il catalog e' stato estratto.
Riceve i decision points e gli assi dal catalog e genera SOLO le domande.
Prompt snello = meno token di output = piu' veloce.

## Come usarlo

1. Leggere il catalog salvato nel DB (prodotto dallo step di normalizzazione)
2. Costruire `decisionsStr` e `axesStr` dal catalog
3. Sostituire `{questionCount}`, `{decisionsStr}`, `{axesStr}` nel prompt
4. Inviare a Claude Opus
5. Parsare il JSON di risposta

## Costruzione delle variabili

```typescript
const axes = catalog.preferenceAxes || [];
const axesStr = axes
  .map(a => `${a.id}: ${a.negPole} <-> ${a.posPole}`)
  .join("; ");

const decisionsStr = catalog.decisionPoints
  .map(d => `- ${d.decision} (${d.decisionType}, evidence: ${d.evidenceStrength}, equipoise: ${d.equipoise})`)
  .join("\n");
```

## Il prompt

```
Genera {questionCount} domande cliniche a scenario basate su questi punti di decisione.

PUNTI DI DECISIONE:
{decisionsStr}

ASSI DI PREFERENZA: {axesStr}

REGOLE:
- Ogni domanda: uno scenario clinico concreto (paziente, dati) + domanda decisionale
- Se evidence strong/moderate e equipoise=false: questionClass="competence", una risposta corretta con isCorrect=true
- Se evidence weak/absent o equipoise=true: questionClass="preference", NESSUNA risposta corretta, ogni risposta ha preferenceAxis e loading (-1 a +1)
- 4 risposte per domanda
- Difficolta' miste: easy, medium, hard

JSON output:
{"questions":[{"questionClass":"competence|preference","difficulty":"easy|medium|hard","stem":"scenario","question":"domanda","answers":[{"text":"...","isCorrect":true}]}]}

Per preference: answers=[{"text":"...","preferenceAxis":"id_asse","loading":0.8}] (NO isCorrect)
Rispondi SOLO JSON.
```

## Output atteso

```json
{
  "questions": [
    {
      "questionClass": "competence",
      "difficulty": "medium",
      "stem": "Paziente di 8 anni, post-intervento ortopedico, VAS 7/10, peso 28kg, funzionalita' renale nella norma.",
      "question": "Come gestisci l'analgesia post-operatoria?",
      "answers": [
        { "text": "Ossicodone orale 0.1mg/kg ogni 4-6h", "isCorrect": true },
        { "text": "Tramadolo ev 2mg/kg", "isCorrect": false },
        { "text": "Solo paracetamolo 15mg/kg", "isCorrect": false },
        { "text": "Morfina sottocute 0.2mg/kg", "isCorrect": false }
      ]
    },
    {
      "questionClass": "preference",
      "difficulty": "hard",
      "stem": "Adolescente 14 anni, frattura composta femore, buon compenso emodinamico, genitori ansiosi.",
      "question": "Quale strategia analgesica adotti nelle prime 24h?",
      "answers": [
        { "text": "Protocollo multimodale aggressivo (oppioide + FANS + paracetamolo + blocco nervoso)", "preferenceAxis": "approccio_analgesico", "loading": 0.8 },
        { "text": "Minimo intervento farmacologico, paracetamolo e rivalutazione a 2h", "preferenceAxis": "approccio_analgesico", "loading": -0.7 },
        { "text": "Oppioide a rilascio prolungato per copertura stabile", "preferenceAxis": "timing_intervento", "loading": 0.6 },
        { "text": "FANS + paracetamolo, step-up solo se VAS > 6", "preferenceAxis": "timing_intervento", "loading": -0.5 }
      ]
    }
  ]
}
```

## Post-processing (campi da aggiungere lato codice)

Claude produce solo i campi essenziali. Dopo il parsing, aggiungere:

```typescript
for (const q of questions) {
  q.specialty = q.specialty || catalog.specialty || "";
  q.topicLabel = q.topicLabel || catalog.mainTopic || "";
  q.decisionType = q.decisionType || "terapeutica";
  q.evidenceStrength = q.evidenceStrength || "moderate";
  q.equipoise = q.equipoise ?? (q.questionClass === "preference");
  q.lang = q.lang || "it";
  q.entity = q.entity || "";
}
```

## Regole fondamentali

1. Il `preferenceAxis` nelle risposte DEVE corrispondere a un `id` degli assi nel catalog
2. Per preference: loading va da -1 (polo negativo) a +1 (polo positivo)
3. Per preference: per ogni asse usato in una domanda servono risposte con loading di segno opposto
4. Per competence: esattamente UNA risposta con `isCorrect: true`
5. NON mescolare `isCorrect` e `preferenceAxis` nella stessa domanda
