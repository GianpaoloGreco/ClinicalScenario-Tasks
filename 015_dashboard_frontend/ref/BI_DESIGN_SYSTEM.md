# Design System - Dashboard BI ClinicalScenario

Specifiche visive del report BI. Il developer puo' usare questo documento per ricostruire la dashboard nel suo framework o copiare i componenti React + CSS dal POC.

---

## 1. Design Tokens

### Colori

| Token | Hex | Uso |
|-------|-----|-----|
| `--paper` | `#eef1f1` | Background pagina |
| `--card` | `#ffffff` | Background card |
| `--ink` | `#16302c` | Testo principale |
| `--ink-soft` | `#51635f` | Testo secondario |
| `--ink3` | `#8a9a96` | Label terziarie |
| `--line` | `#e1e8e6` | Bordi |
| `--line-soft` | `#eef2f1` | Bordi leggeri |
| `--cons` | `#0f6b62` | Teal - conservativo/OK/brand |
| `--aggr` | `#bf6b2e` | Arancione - interventista/warning |
| `--mid` | `#9fb0ac` | Grigio - pragmatico/neutro |
| `--warn` | `#b23a48` | Rosso - gap/errore |
| `--ok` | `#2e7d5b` | Verde - allineato |
| `--neutral` | `#6f827d` | Grigio scuro |

### Palette Personas (3 cluster)

| Persona | Colore | Chiave |
|---------|--------|--------|
| Cluster A (conservativo) | `#0f6b62` teal | `a` |
| Cluster B (interventista) | `#bf6b2e` arancione | `b` |
| Cluster C (pragmatico) | `#6f827d` grigio | `c` |

### Tipografia

| Ruolo | Font | Peso | Size |
|-------|------|------|------|
| Display/titoli | Quicksand | 700 | 25-46px |
| Body | Quicksand | 400-600 | 14-16px |
| Monospace/dati | IBM Plex Mono | 500-600 | 10.5-13px |
| Eyebrow/label | IBM Plex Mono | 600 | 10.5-11.5px, uppercase, letter-spacing 0.12em |

### Shadows & Radius

- Card shadow: `0 18px 44px -30px rgba(0, 75, 75, 0.3)`
- Card border-radius: `16px`
- Pill border-radius: `20px`
- Button border-radius: `999px` (pager), `11px` (nav)

---

## 2. Layout

### Shell

```
+---250px---+---------------------------+
|           |                           |
|  Sidebar  |        Main Content       |
|  (rail)   |      max-width: 1000px    |
|  sticky   |      padding: 40px        |
|           |                           |
|           +---------------------------+
|           |     Pager (sticky bottom) |
+-----------+---------------------------+
```

- Sidebar: 250px fissi, sticky top, border-right
- Main: flex grow, max-width 1000px centrato
- Pager: sticky bottom, con frecce prev/next e contatore pagina
- Responsive < 920px: sidebar diventa top bar orizzontale

### Navigazione

Due modalita':
- **Sintesi**: 2 schermate (Guida + Sintesi compatta)
- **Approfondita**: 7 schermate (Guida, Intestazione, Preferenze, Competenza, Crosscut, Profilo, Appendice)

Toggle tra le due modalita' nella sidebar. Navigazione anche da tastiera (frecce, tasti 1-9).

---

## 3. Schermate

### 00 - Guida alla lettura
4 card in griglia 2x2:
1. **Competenza vs Preferenza** - spiega le due famiglie
2. **Gli assi di preferenza** - elenca i primi 3 assi con poli
3. **Profili decisionali** - spiega il clustering k-means
4. **Come leggere i risultati** - legenda pill (gap, da rivedere, ok)

### 01 - Intestazione (Approfondita) / Sintesi
- Eyebrow "ClinicalScenario - Report di profilazione decisionale"
- Titolo h1 grande (nome modulo)
- Meta card: fonti, item, preferenza, rispondenti, tasso risposta
- Warning arancione se dati simulati
- Run tags: run ID, data, conteggio item
- Findings: 3-5 card con tag/big number/testo

### 02 - Mappa Preferenze
- **Gauges**: uno per asse. Track orizzontale con gradiente teal->arancione.
  - Linea zero centrale
  - Barra IQR (interquartile range) semitrasparente
  - Marker mediana (rettangolo nero 3px)
  - Label valore sopra il marker
  - Poli sotto (sinistro teal, destro arancione)
  - Flag "bimodale" se distribuzione bimodale
- **DotMap SVG**: scatter plot 360x300px
  - Asse X = asse piu' disperso, asse Y = secondo piu' disperso
  - Ogni medico e' un cerchio colorato per persona
  - Linee tratteggiate a croce (zero su entrambi gli assi)
  - Click su punto = apre profilo individuale
  - Label sui poli
- **Personas**: card verticali con bordo sinistro colorato
  - Nome persona generato dinamicamente dal cluster center
  - Share (N / totale, %)
  - Blurb descrittivo generato dagli assi dominanti

### 03 - Competenza
- Tabella con colonne: Item, Argomento, % corrette, barra difficolta', Discriminazione, Esito
- Pill colorate per esito: gap (rosso), da rivedere (arancione), ok (verde)
- **Mappa distrattori**: per ogni item competenza
  - Barra stacked orizzontale: segmento corretto (nero) + distrattori (grigio/rosso)
  - Legenda sotto con percentuali

### 04 - Evidenza x Comportamento (Crosscut)
- **Quadrante SVG** 640x392px
  - Asse X = solidita' evidenza (weak -> strong)
  - Asse Y = dispersione/errore (basso -> alto)
  - 4 quadranti: frontiera incertezza (rosso), gap formazione (arancione), consenso de facto (grigio), allineato (verde)
  - Competenza = quadrati, Preferenza = cerchi
  - Colore = quadrante di appartenenza
  - Tooltip hover con dettaglio item

### 05 - Profilo Individuale
- **Roster**: chip selezionabili per ogni medico (#01, #02...) colorati per persona
- **Card profilo**: grid 220px + 1fr
  - Lato sinistro: ID grande, ruolo, persona, coerenza, competenza, dot correctness
  - Lato destro: MiniGauge per ogni asse (track piccola 8px con dot marker)
  - Nota contestuale: coerenza bassa = avviso, alta = conferma profilo

### Appendice
- Lista item: card con codice (C1, P1...), classe (competenza/preferenza), meta, corpo
- Metodo & cautele: lista puntata con spiegazione routing, assi, coerenza, cautele

---

## 4. Componenti React

### Pill
```tsx
<span className="bi-pill gap">gap</span>
<span className="bi-pill ok">allineato</span>
<span className="bi-pill rev">da rivedere</span>
```

### Gauge (asse di preferenza)
Props: `axis: { key, axis, poleL, poleR, mean, median, sd, iqr, bimodal, note }`
- Track con gradiente
- IQR band
- Median marker
- Flag bimodale

### MiniGauge (profilo individuale)
Props: `axis: AxisData, value: number`
- Track piccola 8px
- Dot marker posizionato

### DotMap (scatter)
Props: `doctors[], personas[], selectedId, onSelect, onOpen, xAxis, yAxis`
- SVG con punti cliccabili
- Colori per persona
- Selezione interattiva

### Quadrant (crosscut)
Props: `points: CrosscutPoint[]`
- SVG con 4 quadranti
- Hover per tooltip
- Deconflicting labels sovrapposti

### DistractorMap
Props: `items: CompetenceItemData[]`
- Barre stacked per item
- Legenda con swatch colorati

---

## 5. Responsive

| Breakpoint | Comportamento |
|------------|---------------|
| > 920px | Layout sidebar + main |
| 680-920px | Sidebar -> top bar, nav orizzontale, labels nascosti |
| < 680px | Griglia findings 1 colonna, profilo stacked, distractor bar full-width |

---

## 6. Stampa

Il report supporta export PDF via `window.print()`:
- Viene renderizzato un container `.bi-print-all` con tutte le schermate in sequenza
- Page breaks tra le sezioni
- Card senza shadow, bordi grigi
- SVG ridimensionati
- Elementi interattivi (roster) nascosti
- Formato A4, margini 18mm x 15mm

---

## 7. File sorgente nel POC

| File | Cosa contiene |
|------|---------------|
| `src/app/operator/bi/bi-dashboard.css` | Tutti gli stili (1278 righe, CSS puro) |
| `src/app/operator/bi/components.tsx` | Componenti base (Pill, Gauge, MiniGauge, DotMap, Quadrant, DistractorMap) |
| `src/app/operator/bi/screens.tsx` | Schermate composte (Guide, Header, Preference, Competence, Crosscut, Profile, Appendix, Sintesi) |
| `src/app/operator/bi/page.tsx` | Shell con routing, sidebar, pager, print |
| `src/app/api/bi-report/route.ts` | Endpoint che assembla tutti i dati per il frontend (562 righe) |

Tutti questi file sono copiabili direttamente. Adattare solo gli import del DB.
