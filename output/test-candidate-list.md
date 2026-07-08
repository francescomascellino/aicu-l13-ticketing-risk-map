# Test candidate note - L13

Non compilare una suite. Scegli il primo test utile.

## Prompt AI facoltativo

```txt
Dato questo comportamento:
priority + sourceChannel -> urgencyLabel

Proponi 5 rischi testabili.
Per ciascuno indica:
- livello di test piu' piccolo utile;
- controllo falsificabile;
- comando candidato;
- motivo per cui non partiresti da un test browser.

Non proporre una suite completa.
```

## 5 rischi testabili:

### 1. Mappatura errata per combinazioni valide

| Campo | Risposta |
|---|---|
| **Livello** | Unit test — funzione pura |
| **Controllo falsificabile** | `computeUrgency("alta", "telefono")` restituisce `"prioritario"` invece di `"intervento rapido"` |
| **Comando** | `npx jest tests/unit/urgency.test.ts` |
| **Perché non browser** | Trasformazione deterministica 1:1; un test browser testa anche DOM, routing, stato — troppa copertura indiretta per un bug logico |

---

### 2. Combinazione mancante (es. priorità "critica" o canale "sms")

| Campo | Risposta |
|---|---|
| **Livello** | Unit test — table test esaustivo |
| **Controllo falsificabile** | `computeUrgency("critica", "sms")` lancia un `Error` o restituisce `undefined` invece di un fallback |
| **Comando** | `npx jest tests/unit/urgency.test.ts --testNamePattern="fallback"` |
| **Perché non browser** | Il rischio è nel dominio logico (input fuori specifica); un test browser lo catturerebbe come crash generico o Stato impossibile — diagnosi più lenta |

---

### 3. Case-sensitivity su sourceChannel

| Campo | Risposta |
|---|---|
| **Livello** | Unit test — input normalizzato vs non normalizzato |
| **Controllo falsificabile** | `computeUrgency("normale", "Telefono")` e `"telefono"` danno output diversi |
| **Comando** | `npx jest tests/unit/urgency.test.ts --testNamePattern="case"` |
| **Perché non browser** | Il bug emerge solo se il dato arriva da un canale con normalizzazione diversa (es. API esterna). Il test browser userebbe sempre il valore esatto dal form, mascherando il problema |

---

### 4. Regressione dopo refactoring della tabella

| Campo | Risposta |
|---|---|
| **Livello** | Unit test parametrico — tutte le 9 combinazioni tabellate |
| **Controllo falsificabile** | Dopo aver centralizzato la mappatura in un JSON/dict, `("alta","email")` dà `"intervento rapido"` invece di `"prioritario"` |
| **Comando** | `npx jest tests/unit/urgency.test.ts` (full suite) |
| **Perché non browser** | Il regresso è puramente logico; un test browser passerebbe per form → API → DB → render, moltiplicando falsi positivi (flaky test, race condition) senza migliorare la rilevazione del mapping sbagliato |

---

### 5. Doppioni non determinabili a colpo d'occhio: priorità "bassa" è monotona (monitorare su tutti i canali)

| Campo | Risposta |
|---|---|
| **Livello** | Unit test di proprietà / invariante |
| **Controllo falsificabile** | Per ogni `sourceChannel`, `computeUrgency("bassa", sourceChannel)` produce un valore diverso da `computeUrgency("alta", sourceChannel)`, o peggio: per qualche canale "bassa" dà "da gestire" invece di "monitorare" (confusione con priorità "normale") |
| **Comando** | `npx jest tests/unit/urgency.test.ts --testNamePattern="invariant"` |
| **Perché non browser** | È una proprietà algebrica del mapping (ordine parziale non rispettato); testarla via browser significa osservare il colore/tag risultante su pagine diverse, molto più indiretto e lento di un'asserzione su una funzione pura |

#### 3 rischi prioritari selezionati:

| Priorità | Comportamento (errore atteso) | Rischio | Livello | Controllo |
|---|---|---|---|---|
| **1** | `alta`+`telefono` produce un label diverso da **intervento rapido** | L'unica combinazione che produce il label più severo non viene preservata | Unit | `computeUrgency("alta", "telefono") === "intervento rapido"` |
| **2** | `computeUrgency("critica", "sms")` lancia eccezione o restituisce `undefined` o viene accettato | Dato fuori contract non bloccato dalla validazione | API | `computeUrgency("critica", "sms")` genera un `400` + `validation.sourceChannel` |
| **3** | Priorità "bassa" produce label diversi su canali diversi | La monotonia della priorità "bassa" (tutti i canali → monitorare) viene alterata | Unit (property) | `["telefono","chat","email"].every(c => computeUrgency("bassa", c) === "monitorare")` |

## Primo test da scrivere in L14

```txt
Comportamento atteso: Alta + Telefono -> "Intervento Rapido"
Rischio: Il mapping è alterato e l'urgenza del ticket viene falsificata nel caso che produce la label più severa
Controllo: La response contiene il valore del mapping corretto (es. `computeUrgency("alta", "telefono") === "intervento rapido"`)
Comando candidato: npx jest tests/unit/urgency.test.ts
```

## Test da non scrivere ora

```txt
Non scrivo:
Perche':
```
