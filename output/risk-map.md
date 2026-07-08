# Mini risk note - L13

Usa questo file solo se ti aiuta a ordinare le idee.

## Nota minima obbligatoria

```txt
Comportamento: Mappatura errata per combinazioni valide (`computeUrgency("alta", "telefono")` restituisce "prioritario" invece di "intervento rapido")
Rischio: Urgenza del ticket falsificata
Controllo: La response contiene il valore del mapping  corretto
Comando candidato: npx jest tests/unit/urgency.test.ts
```

## Rischi possibili

| Rischio | Impatto | Test piccolo utile | Controllo |
| --- | --- | --- | --- |
| mapping `alta + telefono` errato | ticket non trattato come urgente | unit/API | `urgencyLabel === "intervento rapido"` |
| `whatsapp` accettato | dato fuori contract | API validation | `400` + `validation.sourceChannel` |
| label non visibile | operatore non vede priorita' | browser smoke | testo visibile in lista |
| regola spostata nel client | server non autorevole | API | response calcolata dal server |

## Scelta per L14

```txt
Scrivero' prima:
Perche':
```
