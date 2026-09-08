# Lernprojekt: Pen & Paper Charakterverwaltung — Java Fullstack

**Stack:** Spring Boot (Backend, pro Regelwerk) · Angular (ein gemeinsames Frontend) · PostgreSQL · Docker · Kubernetes (k3s)
**Ziel:** Java auf Industriestandard-Niveau lernen, inkl. Cloud-native Deployment — **und** ein langfristig
weiter wachsendes, echtes Pen-&-Paper-Tool aufbauen.
**Zuletzt aktualisiert:** während Phase 4 (Referenzdaten), mit neuer Zwei-Spuren-Struktur

---

## Grundsatzentscheidung: Zwei getrennte Spuren

Das Projekt hat sich von einem reinen Lernvorhaben zu einem Werkzeug entwickelt, an dem langfristig
weitergearbeitet werden soll — inklusive vollständiger Charaktererstellung, Inventar mit Attributs-Boni,
Feats, und NPC-Verwaltung für Spielleiter. Um das ursprüngliche Kernlernziel (Java/Spring/Kubernetes)
nicht in einem reinen Dateneingabe-Projekt zu verlieren, wird bewusst zwischen zwei Dingen unterschieden:

- **Mechanismus/Architektur** (wie wirken Boni aus verschiedenen Quellen auf einen Charakter, wie ist
  das Datenmodell strukturiert) — das lohnt sich, **früh** durchdacht anzulegen, da nachträgliche Änderungen
  teuer sind.
- **Inhalte** (jedes einzelne Feat, jeder Zauber, jeder Ausrüstungsgegenstand) — reine Dateneingabe, ohne
  Auswirkung auf die Architektur, kann **beliebig lange nach und nach** ergänzt werden.

**Konsequenz:** Die Roadmap besteht aus zwei Spuren:

1. **Spur 1 (Phasen 0–9):** die ursprüngliche, lernzielorientierte Roadmap mit definiertem Ende
   (Backend, Frontend, Docker, Kubernetes, CI/CD) — bewusst mit **minimaler, aber korrekter**
   Regeltiefe (Kern-Attributsystem, Point Buy/Standard Array, grundlegende Referenzdaten).
2. **Spur 2 (Phase 10, neu):** ein bewusst **nie abgeschlossenes** Content- und Regeltiefe-Vorhaben,
   das nach Spur 1 (oder parallel dazu) beliebig lange weiterläuft — Feats mit echten Effekten, Zauber,
   Inventar mit Attributs-Boni, vollständige Rassen-Merkmale, NPC-Unterstützung, Multiclassing.

---

## Architektur-Update: Multi-Regelwerk-Fähigkeit

Backend als eigenständiger Service pro Regelwerk (`dnd-backend`, künftig `pathfinder-backend`,
`dsa5-backend`), ein gemeinsames Angular-Frontend (`pnp-character-manager-frontend`). Details siehe
vorherige Roadmap-Version / README des Meta-Repos `pnp-character-manager`.

---

## Phase 0 — Vorbereitung ✅ abgeschlossen
## Phase 1 — Backend-Grundlagen (`dnd-backend`) ✅ abgeschlossen
📄 `Phase1_Backend-Grundlagen.pdf`
## Phase 2 — Authentifizierung & Autorisierung (`dnd-backend`) ✅ abgeschlossen
📄 `Phase2_Authentifizierung-Autorisierung.pdf`

---

## Phase 3 — Frontend-Grundlagen (`pnp-character-manager-frontend`) ⏳ in Arbeit

1–4 ✅ (Projektstruktur, HttpClient/CORS, Auth-Flow, Liste & Detailansicht)
5. ⏳ Charakter-Erstellung — Basisversion fertig, wird nach Phase 4 mit echten Referenzdaten erweitert
6–8. offen (Fehlerbehandlung, Styling-Feinschliff, Internationalisierung)

---

## Phase 4 — DnD-5e-Erstellungsregeln (`dnd-backend` + Frontend) ⏳ in Arbeit

**Bewusst minimal gehaltene Regeltiefe** (Spur 1), volle Inhaltstiefe folgt in Phase 10 (Spur 2).

### Schritt 1 — Referenzdaten: Rassen, Klassen, Backgrounds ⏳ in Arbeit
- Entities `RaceDefinition`, `CharacterClassDefinition`, `BackgroundDefinition` — bewusst mit
  `...Definition`-Suffix zur Vermeidung von Java-Namenskonflikten (insbesondere `Class`)
- Eigenes Unter-Package `character/referencedata/`, da mit sieben+ neuen Klassen ein eigener
  fachlicher Bereich entsteht
- `BackgroundDefinition` nutzt `@ElementCollection` + `@Enumerated(EnumType.STRING)` für die drei
  berechtigten Attribute (neues JPA-Konzept, wichtige Falle: `EnumType.ORDINAL` wäre fragil bei
  späteren Enum-Änderungen)
- **Umfangs-Entscheidung:** Referenzdaten enthalten Name + Beschreibung + eine einfache,
  rein informative Traits-Liste (`@ElementCollection<String>`, z. B. "Dunkelsicht 18m") — bewusst
  **ohne** mechanische Auswertung dieser Traits im Code. Volle mechanische Umsetzung (Feats mit
  echten Effekten, Klassenmerkmale pro Level) ist explizit Phase 10, nicht hier.
- Seed-Daten via Flyway-Migration (`V4__create_reference_data_tables.sql`), auf Basis der
  offiziellen 2024-PHB-Backgrounds (Acolyte, Criminal, Soldier, Sailor, Entertainer, Noble, Guard,
  Farmer) mit korrekt recherchierten Attribut-Zuordnungen
- Öffentlicher, lesender `ReferenceDataController` (`/api/reference-data/races|classes|backgrounds`),
  in `SecurityConfig` als `permitAll()` markiert

### Schritt 2 — Erstellungsmethode & Point-Buy-Validierung
- Feld `generationMethod` (Enum: `POINT_BUY`, `STANDARD_ARRAY`)
- Eigene Validierungs-Annotation mit `ConstraintValidator`-Implementierung (neues Java-Konzept,
  cross-field Validation über einfaches `@Min`/`@Max` hinaus)

### Schritt 3 — Background-Boni anwenden
- Wichtige Modellierungsentscheidung, die zugleich der erste Schritt Richtung des in der
  Grundsatzentscheidung beschriebenen "berechneter Attributswert aus mehreren Quellen"-Gedankens ist:
  Basiswert und Background-Bonus getrennt speichern, nicht nur das Endergebnis

### Schritt 4 — Frontend: mehrstufiger Erstellungs-Assistent
### Schritt 5 — Tests & Dokumentation

**Nach Abschluss:** Rückkehr zu Phase 3, Schritt 5, mit den echten Referenzdaten statt Freitext.

---

## Phase 5 — Containerisierung (Docker)
## Phase 6 — Kubernetes-Grundlagen (ThinkPad-Server)
## Phase 7 — CI/CD
## Phase 8 — Vertiefung / Ausbau (technisch, z. B. Helm, Monitoring, Auth-Feinschliff)
## Phase 9 — Weitere Regelwerke (`pathfinder-backend`, `dsa5-backend`)

*(Details unverändert gegenüber vorheriger Roadmap-Version, siehe `pnp-character-manager`-Repo.)*

---

## Phase 10 — Content & Regeltiefe (neu, Spur 2, bewusst fortlaufend/nie abgeschlossen)

**Charakter dieser Phase, anders als alle vorherigen:** Kein festes Ende, kein "Schritt 5 = fertig".
Wird kontinuierlich erweitert, auch parallel zu oder nach den technischen Phasen 5–9, ganz nach
verfügbarer Zeit und Lust. Betrifft potenziell jedes künftige Regelwerk-Backend gleichermaßen.

### Geplante Ausbaubereiche (unsortiert, keine feste Reihenfolge)

- **Feats mit echten mechanischen Effekten** — eigene `Feat`-Entity, Verknüpfung zu Background/Klasse,
  tatsächliche Auswirkung auf Charakterwerte statt reiner Anzeige
- **Inventar & Ausrüstung** — Gegenstände, die Attributs- oder andere Boni gewähren (das konkrete
  Beispiel "+1 auf Geschick durch ein Item"); erfordert die Weiterentwicklung des Attributsystems von
  "einzelner gespeicherter Wert" zu "berechneter Wert aus Basis + mehreren Bonus-Quellen"
- **Zauber/Spellcasting** — Zauberlisten pro Klasse, bekannte/vorbereitete Zauber, Zauberplätze
- **Vollständige Rassen-/Klassenmerkmale** — alle Klassenmerkmale pro Level (1–20), vollständige
  Rassen-Traits mit tatsächlicher Wirkung (nicht nur Anzeige-Text)
- **Multiclassing**
- **NPC-Unterstützung für Spielleiter** — vermutlich **eigenes, leichteres Modell** statt
  Wiederverwendung von `PlayerCharacter`, da NPCs typischerweise ohne vollständiges
  Charaktererstellungs-Regelwerk (Point Buy etc.) schnell erstellt werden; eigene Entscheidung,
  wenn dieser Bereich ansteht
- **Charakter-Level-Aufstieg als eigener Ablauf**, nicht nur ein Zahlenfeld

**Ausdrücklich kein Ziel:** eine Spielumgebung zum Auswürfeln von Kämpfen oder anderen Aktionen — der
Fokus bleibt auf der **Abbildung** eines vollständigen Charakterblatts, nicht auf Spielmechanik-Simulation.

---

## Prinzip hinter der Reihenfolge

Jede Phase in Spur 1 ist einzeln lauffähig und testbar, bevor die nächste beginnt. Architektonisch
bedeutsame Erkenntnisse (wie die Notwendigkeit eines "Basiswert + Boni aus mehreren Quellen"-Konzepts)
werden dokumentiert und bewusst vorgedacht, sobald sie erkannt werden — aber nur so weit umgesetzt, wie
es die aktuelle Phase tatsächlich braucht. Volle Inhaltstiefe ist explizit Aufgabe von Spur 2 (Phase 10)
und hat kein Ende.
