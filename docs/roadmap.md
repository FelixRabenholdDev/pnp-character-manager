# Lernprojekt: Pen & Paper Charakterverwaltung — Java Fullstack

**Stack:** Spring Boot (Backend, pro Regelwerk) · Angular (ein gemeinsames Frontend) · PostgreSQL · Docker · Kubernetes (k3s)
**Ziel:** Java auf Industriestandard-Niveau lernen, inkl. Cloud-native Deployment
**Zuletzt aktualisiert:** während Phase 3, mit neu eingeplanter Phase 4 (DnD-5e-Erstellungsregeln)

---

## Architektur-Update: Multi-Regelwerk-Fähigkeit

Das Projekt ist von einer reinen D&D-5e-Anwendung zu einem **regelwerkübergreifenden** Charakterverwaltungs-Tool
gewachsen. Geplante weitere Regelwerke: **Pathfinder 1e**, **Das Schwarze Auge 5. Edition**.

**Backend: ein eigenständiger Service pro Regelwerk**, da sich die Regelwerke fachlich zu stark unterscheiden
für ein gemeinsames Datenmodell.

| Repository | Regelwerk | Status |
|---|---|---|
| `dnd-backend` | D&D 5e (2024) | ✅ Phase 1 + 2 abgeschlossen, Phase 4 (neu) geplant |
| `pathfinder-backend` | Pathfinder 1e | geplant, eigenes zukünftiges Repo |
| `dsa5-backend` | DSA 5 | geplant, eigenes zukünftiges Repo |

**Frontend: eine gemeinsame Anwendung** (`pnp-character-manager-frontend`), Ordnerstruktur von Anfang an
verschachtelt nach Regelwerk vorbereitet: `features/characters/dnd5e/`, später `.../pathfinder/`, `.../dsa5/`.

**Kubernetes-Rolle:** Ingress-Routing nach Pfad pro Regelwerk-Service, gemeinsamer, zustandsloser
JWT-Auth-Service — konkret aufgebaut in der Kubernetes-Phase.

---

## Phase 0 — Vorbereitung ✅ abgeschlossen

WSL2 + Ubuntu, Java 21 LTS, Maven, IntelliJ IDEA, Node.js 24 LTS + Angular CLI, Docker Desktop +
WSL2-Integration, Git-Identität, Projektgerüste, GitHub-Anbindung.

---

## Phase 1 — Backend-Grundlagen (`dnd-backend`) ✅ abgeschlossen

Java-Auffrischung · Spring Boot Basics (DI) · REST-API · Persistenz (JPA, Flyway) · Validierung &
Fehlerbehandlung · DnD-5e-Basis-Domänenlogik (Modifikatoren, Proficiency Bonus) · Testing · API-Doku.

📄 `Phase1_Backend-Grundlagen.pdf`

---

## Phase 2 — Authentifizierung & Autorisierung (`dnd-backend`) ✅ abgeschlossen

Spring Security · User-Entity & Passwort-Hashing · JWT · Autorisierung (Besitzer-Zuordnung) · CORS ·
Abschluss: Umstrukturierung in Feature-Packages (`character/`, `auth/`, `config/`).

📄 `Phase2_Authentifizierung-Autorisierung.pdf`

---

## Phase 3 — Frontend-Grundlagen (`pnp-character-manager-frontend`) ⏳ in Arbeit

Angular gegen die fertige `dnd-backend`-API. Angular Material, Standalone Components.

1. ✅ Projektstruktur, Environment-Konfiguration
2. ✅ HttpClient-Anbindung, erster Service, CORS verifiziert (inkl. `127.0.0.1`-vs-`localhost`-Fallstrick)
3. ✅ Auth-Flow — Login, JWT-Interceptor, Route Guard. Dabei gefundener und behobener Bug: abgelaufene
   Tokens ließen das Backend mit `500` statt sauberem `401/403` abstürzen (`JwtAuthenticationFilter` fing
   nur `isTokenValid`, nicht `extractUsername` ab) — Interceptor schickt außerdem bewusst kein Token mehr
   an `/auth/login` bzw. `/auth/register`.
4. ✅ Charakter-Liste (Material Cards) & Detailansicht (`ActivatedRoute`, dynamische Routen-Parameter)
5. ⏳ **Charakter-Erstellung — Basisversion fertig, wird nach Phase 4 (neu) erweitert.** Reactive Forms mit
   verschachtelter `FormGroup` für `stats` stehen; aktuell nur einfache Wertebereichs-Validierung
   (1–30 je Attribut), noch **ohne** echte D&D-5e-Erstellungsregeln (Point Buy/Standard Array,
   Rasse/Klasse als Freitext statt Referenzdaten) — das folgt gezielt in der neuen Phase 4.
6. Fehlerbehandlung & Ladezustände weiter verfeinern
7. Feinschliff Angular-Material-Styling
8. Internationalisierung (DE/EN) — Transloco, Backend-Fehlercodes statt fester Texte

---

## Phase 4 — DnD-5e-Erstellungsregeln (neu, `dnd-backend` + Frontend) ⏳ geplant

**Warum als eigene Phase, nicht einfach "mehr Validierung" in Phase 3:** Diese Anforderung verlangt echte
**Backend-Erweiterungen** (neue Referenzdaten-Entities, neue Geschäftsregel-Validierung), nicht nur
Frontend-Feinschliff. Um unser Grundprinzip ("Backend fertig, dann Frontend dagegen bauen") nicht zu
verletzen, wird das als bewusster, in sich geschlossener Rückschritt zum Backend behandelt, bevor das
Frontend die neuen Möglichkeiten konsumiert.

**Wichtige Regel-Klarstellung (2024-Edition, nicht 2014):** Attributsboni kommen in D&D 5e 2024 vom
**Background**, nicht von der Rasse — das Modell muss das korrekt widerspiegeln, nicht die ältere
2014-Logik übernehmen.

### Schritt 1 — Referenzdaten: Rassen, Klassen, Backgrounds (Backend)
- Neue Entities (Namensgebung mit Bedacht wegen möglicher Java-Namenskonflikte prüfen, z. B.
  `RaceDefinition`, `ClassDefinition`, `BackgroundDefinition`)
- Datenhaltung: feste Referenztabellen, befüllt über eine Flyway-Migration (Seed-Daten), nicht
  Hibernate-generiert
- Neue, öffentlich lesbare Endpoints: `GET /api/races`, `GET /api/classes`, `GET /api/backgrounds`
- `PlayerCharacter` verweist danach auf diese Referenzdaten (Fremdschlüssel) statt auf freien Text

### Schritt 2 — Erstellungsmethode & Point-Buy-Validierung (Backend)
- Neues Feld `generationMethod` (Enum: `POINT_BUY`, `STANDARD_ARRAY`)
- **Neues Java-Konzept: eigene Validierungs-Annotation** (`@ValidAbilityScores` o. ä.) mit einer
  Implementierung von `ConstraintValidator` — Bean Validation über einfaches `@Min`/`@Max` hinaus,
  da hier **mehrere Felder gemeinsam** geprüft werden müssen (Kostentabelle, Gesamtbudget 27 Punkte
  bzw. exakte Übereinstimmung mit `{15,14,13,12,10,8}`)
- Unit-Tests für beide Erstellungsmethoden, inkl. Grenzfällen (genau 27 Punkte, 28 Punkte → Fehler)

### Schritt 3 — Background-Boni anwenden (Backend)
- Modellierungsentscheidung dokumentieren: rohe (gewürfelte/verteilte) Attributwerte plus Background-Bonus
  getrennt speichern, oder nur das bereits finale Ergebnis? (Tendenz: getrennt speichern, für
  Nachvollziehbarkeit und spätere Anzeige "Basis + Bonus" im Frontend)

### Schritt 4 — Frontend: mehrstufiger Erstellungs-Assistent
- Schritt 1: Erstellungsmethode wählen (Point Buy / Standard Array)
- Schritt 2: Rasse/Klasse/Background per Dropdown aus den neuen Referenz-Endpoints (kein Freitext mehr)
- Schritt 3: Attribute passend zur gewählten Methode verteilen (Point-Buy-Zähler mit laufendem
  Restbudget bzw. Standard-Array-Zuordnung statt freier Zahleneingabe)
- Schritt 4: Zusammenfassung & Absenden

### Schritt 5 — Tests & Dokumentation
- Backend: Unit-Tests für Kostentabelle und Array-Prüfung, Integrationstest für die neuen
  Referenz-Endpoints
- Swagger-Dokumentation der neuen Endpoints automatisch mit abgedeckt (springdoc erkennt sie ohne
  Zusatzaufwand, wie in Phase 1 gelernt)

**Nach Abschluss dieser Phase:** Rückkehr zu Phase 3, Schritt 5 (Charakter-Erstellung im Frontend), diesmal
mit den echten Referenzdaten und Regeln statt der aktuellen Freitext-/Freiwert-Basisversion.

---

## Phase 5 — Containerisierung (Docker)

Dockerfile Backend (Multi-Stage), Dockerfile Frontend (Angular-Build → Nginx), `docker-compose.yml`,
saubere Trennung von Umgebungsvariablen/Secrets.

---

## Phase 6 — Kubernetes-Grundlagen (ThinkPad-Server)

k3s, Pods/Deployments/Services/Namespaces, ConfigMaps/Secrets, Ingress (Traefik). Hier entsteht die
tatsächliche Multi-Service-Architektur mit Pfad-Routing pro Regelwerk-Service.

---

## Phase 7 — CI/CD

GitHub Actions: Build + Tests je Push, Docker-Images in Registry, automatisches Deployment.

---

## Phase 8 — Vertiefung / Ausbau

- Helm-Chart, Monitoring (Prometheus/Grafana), Horizontal Pod Autoscaling, Redis-Caching, E2E-Tests
- Auth-Fehlerantwort-Feinschliff (`401` statt `403`, eigener `AuthenticationEntryPoint`)
- Migration auf Angulars neues `animate.enter`/`animate.leave`-System, sobald stabiler (aktuell noch
  `provideAnimationsAsync()`, deprecated seit 20.2, geplante Entfernung erst v23)

---

## Phase 9 — Weitere Regelwerke

`pathfinder-backend`, `dsa5-backend` als eigenständige Projekte mit eigenem Domänenmodell, eigener
Datenbank, eigenem Flyway-Set. Frontend: `features/characters/pathfinder/`, `.../dsa5/`, oberster
Regelwerk-Umschalter. Ingress-Erweiterung. Prüfung, ob Auth wirklich gemeinsam bleiben kann.

---

## Prinzip hinter der Reihenfolge (unverändert)

Jede Phase ist einzeln lauffähig und testbar, bevor die nächste beginnt. Architektonisch bedeutsame
Anforderungen, die während einer laufenden Phase auftauchen (wie die D&D-Erstellungsregeln während der
Frontend-Phase), werden als **eigene, saubere Phase** eingeplant, statt das laufende Backend-vor-Frontend-
Prinzip zu vermischen — auch wenn das bedeutet, kurzzeitig zum Backend zurückzukehren, bevor das Frontend
fortgesetzt wird.
