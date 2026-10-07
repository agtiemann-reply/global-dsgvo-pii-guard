---
name: global-dsgvo-pii-guard
description: Verhindert projektübergreifend das unverschlüsselte Speichern oder Loggen von personenbezogenen Daten (PII). Use when creating or changing API responses, database schemas or migrations, logging calls, or user/customer objects in any project.
triggers: Bei der Erstellung oder Änderung von API-Responses, Datenbank-Schemata, Logging-Aufrufen oder User-Objekten in jedem Projekt.
---

# PII & DSGVO Guard

## Grundsatz

Personenbezogene Daten (PII) landen nie im Klartext in Logs und nie unverschlüsselt in der Datenbank, ohne dass ich darauf hinweise. Diese Regeln gelten in jedem Projekt und ohne Ausnahme. Wer nur den Wortlaut umgeht, verstößt trotzdem gegen die Regel.

## Pflichtregeln

1. **Jeden betroffenen Code prüfen.** Bevor ich Code schreibe oder ändere, der Datenmodelle, API-Responses, User-Objekte oder Logs verarbeitet, prüfe ich ihn auf sensible Felder. Dazu gehören mindestens:
   - `email`, `name`, `vorname`, `nachname`, `first_name`, `last_name`
   - `iban`, `bic`, `kontonummer`, Kreditkartendaten
   - `ip_adresse`, `ip`, `ip_address`
   - `mandanten_id`, `tenant_id`, `kunden_id`, `customer_id`
   - `telefon`, `phone`, `adresse`, `address`, `geburtsdatum`, `birth_date`, Steuer- und Sozialversicherungsnummern

   Varianten in anderer Schreibweise (camelCase, snake_case, englisch oder deutsch) zählen genauso.

2. **Plaintext-PII in Logs ist streng verboten.** Wenn PII geloggt werden soll, implementiere ich zwingend eine Maskierungs-Funktion, die Teile durch `***` ersetzt, und logge nur den maskierten Wert. Das gilt auch für ganze Objekte (z. B. `logger.info(user)`, `console.log(req.body)`), Exception-Messages, Debug-Logs und temporäre Logs.

3. **Migrationen mit PII-Feldern brauchen einen Verschlüsselungshinweis.** Wenn ich eine Datenbank-Migration oder ein Schema mit PII-Feldern erstelle oder ändere, weise ich in meiner Antwort von mir aus darauf hin, dass diese Felder verschlüsselt gespeichert werden müssen (Encryption at Rest, z. B. Spaltenverschlüsselung oder verschlüsselte Datenbank bzw. Volumes). Ich nenne die betroffenen Felder einzeln.

4. **Jede Maskierung wird kommentiert.** An jeder Stelle, an der maskiert wird, steht ein Code-Kommentar mit Hinweis auf DSGVO-Compliance.

## Beispiel

```typescript
// DSGVO-Compliance (Art. 5 Abs. 1 lit. c, Art. 32): PII vor dem Logging maskieren
function maskEmail(email: string): string {
  const [local, domain] = email.split("@");
  return `${local.slice(0, 1)}***@${domain ?? "***"}`;
}

// DSGVO-Compliance: IBAN nur maskiert loggen
function maskIban(iban: string): string {
  return `${iban.slice(0, 4)}***${iban.slice(-4)}`;
}

// DSGVO-Compliance: Nur maskierte PII im Log
logger.info("User registriert", { email: maskEmail(user.email), tenant: maskId(user.mandantenId) });
```

## Red Flags: STOP

- `log(user)`, `log(request.body)` oder `log(error)` mit PII im Objekt
- „Ist nur ein Debug-Log“ oder „wird später entfernt“
- „Die Daten sind nicht wirklich sensibel“
- Neue Spalte `email`, `iban` usw. in einer Migration ohne Verschlüsselungshinweis
- Maskierung ohne DSGVO-Kommentar

**In all diesen Fällen: Maskierung einbauen, kommentieren und auf Verschlüsselung hinweisen.**
