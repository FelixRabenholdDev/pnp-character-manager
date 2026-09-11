# Lernprojekt: Pen & Paper Charakterverwaltung — Java Fullstack

**Stack:** Spring Boot (Backend, pro Regelwerk) · Angular (ein gemeinsames Frontend) · PostgreSQL · Docker · Kubernetes (k3s)
**Ziel:** Java auf Industriestandard-Niveau lernen, inkl. Cloud-native Deployment — **und** ein langfristig
weiter wachsendes, echtes Pen-&-Paper-Tool aufbauen.
**Zuletzt aktualisiert:** nach Abschluss von Phase 4

---

## Grundsatzentscheidung: Zwei getrennte Spuren

- **Spur 1 (Phasen 0–9):** die lernzielorientierte Roadmap mit definiertem Ende (Backend, Frontend,
  Docker, Kubernetes, CI/CD) — bewusst mit minimaler, aber korrekter Regeltiefe.
- **Spur 2 (Phase 10):** ein bewusst nie abgeschlossenes Content- und Regeltiefe-Vorhaben (Feats,
  Zauber, Inventar mit Attributs-Boni, vollständige Rassen-Merkmale, NPC-Unterstützung).

Details: siehe README des Meta-Repos `pnp-character-manager`.

---

## Phase 0 — Vorbereitung ✅ abgeschlossen

## Phase 1 — Backend-Grundlagen (`dnd-backend`) ✅ abgeschlossen

📄 `Phase1_Backend-Grundlagen.pdf`

## Phase 2 — Authentifizierung & Autorisierung (`dnd-backend`) ✅ abgeschlossen

📄 `Phase2_Authentifizierung-Autorisierung.pdf`

---

## Phase 3 — Frontend-Grundlagen (`pnp-character-manager-frontend`) ⏳ in Arbeit

1–4 ✅ (Projektstruktur, HttpClient/CORS, Auth-Flow, Liste & Detailansicht) 5. ✅ Charakter-Erstellung — jetzt vollständig über den in Phase 4 gebauten mehrstufigen
Erstellungs-Assistenten abgedeckt (ersetzt die ursprüngliche, einfache Version)
6–8. offen (Fehlerbehandlung/Ladezustände weiter verfeinern, Styling-Feinschliff, Internationalisierung)

---

## Phase 4 — DnD-5e-Erstellungsregeln (`dnd-backend` + Frontend) ✅ abgeschlossen

### Schritt 1 — Referenzdaten: Rassen, Klassen, Backgrounds ✅

- Entities `RaceDefinition`, `CharacterClassDefinition`, `BackgroundDefinition` in eigenem
  `character/referencedata/`-Package
- `@ElementCollection` (als `Set`, mit `@Enumerated(EnumType.STRING)` bei Enum-Werten, sowie als
  einfache String-Liste für rein informative Traits) — wichtige Falle dabei: `EnumType.ORDINAL`
  wäre fragil bei späteren Enum-Änderungen
- Seed-Daten via Flyway (`V4`), auf Basis recherchierter, korrekter 2024-PHB-Background-Daten
- Öffentlicher, lesender `ReferenceDataController`, in `SecurityConfig` als `permitAll()` markiert

### Schritt 2 — Erstellungsmethode & Point-Buy-Validierung ✅

- Eigenes `character/generation/`-Unter-Package (Package wurde bei 9 Dateien in `character/`
  bewusst weiter unterteilt — Faustregel: eigener Unterordner ab ca. 3–4 fachlich zusammengehörigen
  Klassen)
- Eigene Bean-Validation-Annotation (`@ValidAbilityScoreGeneration`) mit
  `ConstraintValidator`-Implementierung — cross-field Validation über einfaches `@Min`/`@Max` hinaus
- **Erweiterung um eine dritte Methode, ursprünglich nicht eingeplant:** `ROLLED` (4W6, niedrigsten
  Wurf verwerfen) — die unter Spielern tatsächlich gebräuchlichste Methode, nachträglich als
  sinnvoller Ergänzungsfund identifiziert. Server würfelt selbst (nicht der Client), liefert alle
  vier Einzelwürfe transparent zurück; Validierung beschränkt sich auf Plausibilitätsprüfung
  (Werte im für 4W6-Drop-Lowest erreichbaren Bereich 3–18), bewusst ohne kryptografische Bindung
  zwischen Wurf und Einreichung (kein Mehrwert für ein Tool am eigenen Spieltisch)

### Schritt 3 — Background-Boni anwenden ✅

- Zentrale Architekturentscheidung umgesetzt: Basiswert (`baseStats`, aus Point Buy/Standard
  Array/Würfeln) getrennt von angewandtem Bonus gespeichert — `PlayerCharacter.getEffectiveScore(...)`
  berechnet zur Laufzeit `Basiswert + Bonus`, offen für spätere weitere Bonus-Quellen (Items, Feats
  in Phase 10)
- `@ManyToOne`-Beziehungen zu den Referenzdaten ersetzen den ursprünglichen Freitext für
  Rasse/Klasse/Background
- Zweite eigene Validierungs-Annotation (`@ValidBackgroundBonus`) für die +2/+1- bzw.
  +1/+1/+1-Verteilungsregel, inkl. Prüfung gegen die drei berechtigten Attribute des gewählten
  Backgrounds
- `PlayerCharacterResponse` liefert `baseStats` und `effectiveStats` getrennt

### Schritt 4 — Frontend: mehrstufiger Erstellungs-Assistent ✅

- `mat-stepper` mit vier linearen Schritten (Methode → Identität → Attribute → Zusammenfassung)
- Alle drei Erstellungsmethoden abgebildet, inkl. dynamischer Zuordnungs-UI für gewürfelte Werte
  (bereits zugewiesene Würfe werden aus anderen Dropdowns ausgeblendet, mit expliziter
  „Zurücksetzen"-Option)
- Getrennte `FormGroup`s pro Wizard-Schritt statt einem großen Formular

### Schritt 5 — Tests & Dokumentation ✅

- Unit-Tests für beide Custom-Validatoren mittels Mockito (`Answers.RETURNS_DEEP_STUBS` für
  verkettete `ConstraintValidatorContext`-Aufrufe, ganz ohne Spring-Kontext)
- `@RepeatedTest(20)` für den zufallsbasierten Würfel-Mechanismus — Testprinzip bei Zufallslogik:
  Invarianten prüfen (Regeln, die immer gelten müssen), nicht konkrete Werte
- Swagger-Dokumentation automatisch aktuell; ein kleiner, aber echter Bugfix dabei gefunden:
  `/swagger-ui.html` (Redirect-Route) war von der `permitAll()`-Regel nicht erfasst, nur
  `/swagger-ui/**` selbst

**Nach Abschluss:** Phase 3, Schritt 5 gilt durch den neuen Assistenten als erledigt.

---

## Phase 5 — Containerisierung (Docker)

## Phase 6 — Kubernetes-Grundlagen (ThinkPad-Server)

## Phase 7 — CI/CD

## Phase 8 — Vertiefung / Ausbau (technisch, z. B. Helm, Monitoring, Auth-Feinschliff)

## Phase 9 — Weitere Regelwerke (`pathfinder-backend`, `dsa5-backend`)

_(Details unverändert gegenüber vorheriger Roadmap-Version.)_

---

## Phase 10 — Content & Regeltiefe (Spur 2, bewusst fortlaufend/nie abgeschlossen)

Feats mit echten Effekten, Zauber/Spellcasting, Inventar & Ausrüstung mit Attributs-Boni (Erweiterung
des bereits in Phase 4 gelegten "Basiswert + Bonus-Quellen"-Fundaments), vollständige Rassen-/
Klassenmerkmale, Multiclassing, NPC-Unterstützung für Spielleiter (vermutlich eigenes, leichteres
Modell statt `PlayerCharacter`-Wiederverwendung), Charakter-Level-Aufstieg als eigener Ablauf.

Ausdrücklich kein Ziel: eine Spielumgebung zum Auswürfeln von Kämpfen oder anderen Aktionen.

---

## Prinzip hinter der Reihenfolge

Jede Phase in Spur 1 ist einzeln lauffähig und testbar, bevor die nächste beginnt. Architektonisch
bedeutsame Erkenntnisse werden dokumentiert und bewusst vorgedacht, sobald sie erkannt werden, aber
nur so weit umgesetzt, wie es die aktuelle Phase tatsächlich braucht. Ergänzungen, die sich während
der Arbeit als sinnvoll herausstellen (wie die Würfel-Methode), werden aufgenommen, wenn sie den
bestehenden Rahmen nicht sprengen — nicht stur nach ursprünglichem Plan abgearbeitet.
