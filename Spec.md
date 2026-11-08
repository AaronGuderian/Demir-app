# SPEC.md · Praxis Dr. Aylin Demir

Quelle: Meeting 1 und Meeting 2 (insgesamt 20 Kundenantworten). Verweise wie `M1/N1` bezeichnen Meeting 1, Nachricht 1. IDs sind vorgeschlagene Schlüssel, keine bestätigten Feldnamen.

## Entitäten

| Entität | Attribute |
|---|---|
| Patient | `patient_id`, `versicherungsart`, `identifikationsnummer`, `geburtsdatum`, `hausaerztin` → Ärztin, `online_gesperrt` |
| Ärztin | `arzt_id`, `name`, `qualifikation`, `sprechzeiten` |
| MFA | `mfa_id`, `name`, `berechtigungsrolle` |
| Termintyp | `termintyp_id`, `bezeichnung` (Beratung, Vorsorge, Impfung, Akut, Blutabnahme), `unterart` (Standard-/Reiseimpfung), `standarddauer_min` |
| Terminslot | `slot_id`, `aerztin` → Ärztin, `beginn`, `dauer_min`, `status` (reserviert/frei/belegt), `buchungskanal` (online/MFA), `reservierungsgrund` |
| Termin | `termin_id`, `patient` → Patient, `termintyp` → Termintyp, `beginn`, `status` (gebucht/zu prüfen/abgesagt) |
| Abwesenheit / Schließung | `abwesenheit_id`, `beginn`, `ende`, `grund` (Urlaub/Krankheit/Schließtag), `geltungsbereich` (einzelne Ärztin/gesamte Praxis) |
| Wiederholungsrezept | `rezept_id`, `patient` → Patient, `medikament`, `intervall`, `status` (beantragt/aktiv/ersetzt), `letzte_freigabe`, `freigegeben_durch` → Ärztin |

Hinweis: Rezeptanträge und Änderungsprotokolle sind fachlich erforderlich; ob sie eigenständige Entitäten werden, ist noch offen (M1/N7–N9).

## Beziehungen

- Ärztin 1:n Patient als zugeordnete Hausärztin (bei Neupatienten ggf. noch ohne Zuordnung; M1/N2, N7).
- Patient 1:n Termin (M1/N2).
- Ärztin 1:n Termin als behandelnde Ärztin (M1/N2, N5).
- Termintyp 1:n Termin (M1/N2, N10).
- Ärztin n:m Termintyp: Mehrere zulässige Behandlungen je Ärztin; Standardimpfungen bei allen drei, Reiseimpfungen nur bei Dr. Demir (M1/N3; M2/N3).
- Ärztin 1:n Terminslot; jeder Slot ist einer Ärztin zugeordnet (M2/N4).
- Termin 1:n Terminslot: Ein Termin kann bei Bedarf einen Doppelslot belegen; jeder Slot ist höchstens einem Termin zugeordnet (M1/N10; Modellannahme zur Slot-Abbildung).
- Ärztin n:m Abwesenheit/Schließung: Eine Abwesenheit betrifft eine oder bei Praxisschließung alle Ärztinnen (M1/N3–N4; Modellannahme).
- Patient 1:n Wiederholungsrezept (M1/N7–N8).

## Business Rules

- **BR1 – Buchungsberechtigung:** Stammpatienten dürfen online buchen. Bei fehlgeschlagener Identifikation (Patienten-/Versichertennummer und Geburtsdatum) wird auf telefonische Buchung verwiesen; Neupatienten buchen Ersttermine telefonisch. Nach zweimaligem unentschuldigtem Nichterscheinen innerhalb von zwölf Monaten erfolgt eine schriftliche Erinnerung, nach dem dritten wird die Online-Buchung gesperrt (M1/N2, N12; M2/N6, N8).
- **BR2 – Termin- und Ärztezuordnung:** Stammpatienten werden grundsätzlich bei ihrer Hausärztin gebucht; Vertretung richtet sich nach Verfügbarkeit, Dringlichkeit und Patientenwunsch. Reiseimpfungen erfolgen nur bei Dr. Demir, Standardimpfungen bei allen drei Ärztinnen. Standarddauern: Beratung 15 Min., Vorsorge 20–30 Min., Impfung 10 Min., Blutabnahme 5–10 Min.; komplexe Beratungen können einen Doppelslot erhalten (M1/N2–N3, N6, N10; M2/N3).
- **BR3 – Reservierte Termine:** Je vier praxisweite Akutslots am Vormittag und Nachmittag sind ausschließlich durch MFAs buchbar. Montags werden zusätzlich zwei Slots je Ärztin für Wochenend-Dringlichkeiten reserviert. MFAs können ungenutzte Reservierungen situativ freigeben, die Slots bleiben jedoch ausschließlich über die MFA buchbar – nicht online (M1/N10; M2/N4–N5).
- **BR4 – Verfügbarkeit und Buchungskonflikte:** Urlaub sperrt Slots der betroffenen Ärztin; vollständige Praxisschließung sperrt alle. Bei kurzfristigem Ausfall werden betroffene Termine als „zu prüfen“ markiert. Ein Slot darf nur einmal vergeben werden: Die erste erfolgreich bestätigte Buchung gilt; der zweite Buchungsversuch erhält eine Rückmeldung und Alternativtermine (M1/N4–N6; M2/N2, N7).
- **BR5 – Wiederholungsrezepte:** MFAs dürfen Anträge erfassen, aber keine ärztlichen Freigaben oder medizinischen Änderungen vornehmen. Berechtigte Ärztinnen dürfen freigeben und ändern, einschließlich vertretender Ärztinnen im Vertretungszeitraum. Ersetzte Einträge bleiben als inaktiv erkennbar; Änderungen protokollieren Ärztin, Zeitpunkt, Grund und Vertretungskontext (M1/N7–N9; M2/N2).

## Widersprüche

- **W1:** „Impfungen nur bei mir“ (M1/N2) versus Standardimpfungen auch bei Kolleginnen (M1/N3) und erneute pauschale Prototyp-Aussage „nur bei mir“ (M2/N2). **Auflösung (M2/N3):** Der Termintyp Impfung unterscheidet Standardimpfungen (alle drei Ärztinnen) und Reiseimpfungen (nur Dr. Demir); diese Regel gilt auch im Prototyp.
- **W2:** Routinetermine können bei Abwesenheit von Kolleginnen übernommen werden (M1/N3) versus Routinetermine werden bei kurzfristigem Ausfall verschoben (M1/N5). **Auflösung (M1/N6):** Keine pauschale Regel; die MFA berücksichtigt Dringlichkeit, freie Kapazität, Kenntnis des Patienten und dessen Wunsch. Übernahme oder Verschiebung sind möglich.

## Prioritäten

1. Online-Terminbuchung mit korrekter Ärztinnen-/Termintypzuordnung und Vermeidung von Doppelbuchungen (**v1**).
2. Akutslots für Online-Buchungen sperren (**v1**).
3. Urlaube und Praxisschließungen berücksichtigen (**v1**).
4. Wiederholungsrezept-Workflow mit ärztlicher Freigabe (**v1**).
5. Identifikation bestehender Patienten vor Online-Buchung (**v1**).  
   *Reihenfolge gemäß M1/N12.*

**Prototyp – drei Abnahmeszenarien (M2/N2):**

1. Stammpatient bucht online einen zulässigen Beratungstermin.
2. MFA erfasst Rezeptanfrage → Ärztin gibt frei → Abholung möglich; demonstrierter Ablauf in unter zwei Minuten.
3. MFA erfasst kurzfristigen Ausfall mit zwei Klicks; betroffene Termine werden als „zu prüfen“ markiert.

**Später:** Terminartspezifischer Online-Fragebogen nach Buchung; Antworten liegen vor dem Termin vor (M1/N11). Erinnerungen per SMS/E-Mail nur mit ausdrücklicher Einwilligung. Flexible Slotlängen sind ausbaubar; die Ausfall-Arbeitsliste wurde später ausdrücklich für den Prototyp gefordert (M1/N12; M2/N2). Videosprechstunden sind derzeit nicht vorgesehen (M1/N11).

## Offene Fragen

- Welche konkret nutzbaren Schnittstellen bietet das Praxisverwaltungssystem **Tomedo** für die geforderte Echtzeit-Synchronisierung? Die technische Eignung ist noch nicht überprüft (M2/N7–N8).
- Wie wird die Schnittstellenintegration technisch und datenschutzkonform umgesetzt (M2/N7–N8)?
- Wie wird die Sicherheit und DSGVO-Konformität der Identifikation über Nummer und Geburtsdatum bewertet? Dies ist keine echte Zwei-Faktor-Authentifizierung (M2/N6).
- Gilt der Zeitraum von zwölf Monaten auch für den dritten No-Show-Vorfall, und wie lange bleibt die Online-Buchung gesperrt (M2/N8)?
- Wie werden die zeitlich bereits verstrichenen Montagsslots in der Praxis behandelt, wenn eine MFA sie erst nach Beginn des Vormittags freigibt (M2/N5)?
