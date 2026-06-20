# Prompt: Normalizzazione PDF + Estrazione Catalog

Questo prompt va inviato a Claude Opus con il testo estratto dal PDF.
Produce in UNA SOLA chiamata: il Markdown strutturato + il catalog JSON.

## Come usarlo

1. Estrarre il testo per pagina con `pdfjs-dist` (vedi `scripts/extract-pdf-pages.cjs`)
2. Costruire il blocco `textContent` con formato `=== PAG N ===\n{testo}`
3. Sostituire `{pageCount}` e `{textContent}` nel prompt
4. Sostituire `{decisionTypesJson}` con `["terapeutica","timing","complicanza","diagnosi"]`
5. Inviare a Claude Opus (model: `claude-opus-4-20250514`)
6. Splittare la risposta su `---CATALOG---` per ottenere markdown e JSON separati

## Il prompt

```
Analizza questo documento medico ({pageCount} slide) e produci DUE output separati.

TESTO DEL DOCUMENTO:
{textContent}

=== OUTPUT 1: MARKDOWN ===
Normalizza il contenuto in Markdown strutturato. Per ogni pagina: ## Pagina N - [titolo]. Elenchi puntati, tabelle, paragrafi. Mantieni TUTTO il contenuto clinico.

=== OUTPUT 2: CATALOG JSON ===
Dopo il Markdown, su una riga separata scrivi ---CATALOG--- e poi un JSON con:
- mainTopic, specialty, domain
- preferenceAxes: 3-5 trade-off clinici dal contenuto [{id, negPole, posPole}]
- relevantEntities: entita' cliniche rilevanti
- decisionPoints: punti di decisione [{id, decision, decisionType ({decisionTypesJson}), topicLabel, options[], evidenceStrength (strong|moderate|weak|absent), equipoise}]

Formato risposta:
[markdown qui]
---CATALOG---
{json qui}
```

## Output atteso

La risposta di Claude avra' questa struttura:

```
## Pagina 1 - Titolo della slide

- Contenuto strutturato...
- Altro contenuto...

## Pagina 2 - Titolo seconda slide

...

---CATALOG---
{
  "mainTopic": "Gestione del dolore post-operatorio pediatrico",
  "specialty": "anestesiologia",
  "domain": "anestesiologia pediatrica",
  "preferenceAxes": [
    { "id": "approccio_analgesico", "negPole": "minimo intervento", "posPole": "multimodale aggressivo" },
    { "id": "timing_intervento", "negPole": "attesa vigile", "posPole": "intervento precoce" },
    { "id": "uso_oppioidi", "negPole": "oppioide-sparing", "posPole": "oppioide-first" }
  ],
  "relevantEntities": ["ossicodone", "tramadolo", "paracetamolo", "ketorolac"],
  "decisionPoints": [
    {
      "id": "d1",
      "decision": "Scelta dell'oppioide nel post-operatorio pediatrico",
      "decisionType": "terapeutica",
      "topicLabel": "Analgesia post-operatoria pediatrica",
      "options": ["Ossicodone orale", "Tramadolo ev", "Morfina sottocute"],
      "evidenceStrength": "moderate",
      "equipoise": false
    }
  ]
}
```

## Parsing della risposta (pseudocodice)

```typescript
const response = await callClaude(prompt);
const parts = response.split("---CATALOG---");
const markdown = parts[0].trim();
let catalog = null;
if (parts[1]) {
  const jsonStr = extractJson(parts[1].trim()); // trova il primo {...} bilanciato
  catalog = JSON.parse(jsonStr);
}
```

## Note

- Gli assi di preferenza sono DINAMICI: cambiano per ogni PDF. Non hardcodarli.
- Il catalog e' il ponte tra normalizzazione e generazione domande.
- Tempo atteso: ~60-120s con Opus via CLI, ~30-60s con SDK.
