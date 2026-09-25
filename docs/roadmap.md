# Lernprojekt: Pen & Paper Charakterverwaltung — Java Fullstack

**Stack:** Spring Boot (Backend, pro Regelwerk) · Angular (ein gemeinsames Frontend) · PostgreSQL · Docker · Kubernetes (k3s)
**Ziel:** Java auf Industriestandard-Niveau lernen, inkl. Cloud-native Deployment — **und** ein langfristig
weiter wachsendes, echtes Pen-&-Paper-Tool aufbauen.
**Zuletzt aktualisiert:** nach Abschluss von Phase 6

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

## Phase 5 — Containerisierung (Docker) ✅ abgeschlossen

📄 `Phase5_Containerisierung.pdf`

### Schritt 1 — Backend-Dockerfile (`dnd-backend`) ✅

- Multi-Stage-Build: Stage 1 (`eclipse-temurin:21-jdk`) baut das Jar mit Maven, Stage 2
  (`eclipse-temurin:21-jre`) enthält nur das fertige Artefakt — spart erheblich Image-Größe, da
  JDK/Maven/Quellcode im finalen Image nicht mehr enthalten sind
- Layer-Caching bewusst ausgenutzt: zuerst `.mvn/`, `mvnw`, `pom.xml` kopieren und
  `dependency:go-offline` ausführen, _danach erst_ `src/` kopieren — solange sich nur der Code,
  nicht aber die Abhängigkeiten ändern, bleibt der teure Dependency-Download-Layer im Docker-Cache
  erhalten
- Nicht-root-Nutzer im Runtime-Image (`addgroup`/`adduser` + `USER spring:spring`) als
  Sicherheitsmaßnahme — ein kompromittierter Java-Prozess läuft nicht mit Root-Rechten im Container
- **Nachtrag aus Phase 6:** Der benannte Nutzer `spring:spring` wurde später durch einen numerischen
  Nutzer ersetzt (`appuser`, UID/GID `10001:10001`), um NSS-Namensauflösungsprobleme im
  Kubernetes-/containerd-Kontext zu vermeiden.
- Verifiziert mit eigenständigem `docker run` gegen eine lokal laufende Postgres-Instanz
  (`host.docker.internal`), inkl. erfolgreichem `curl` auf `/api/reference-data/races`

### Schritt 2 — Frontend-Dockerfile (`pnp-character-manager-frontend`) ✅

- Ebenfalls Multi-Stage: Stage 1 (`node:24-alpine`) führt `npm ci` und `npm run build` aus,
  Stage 2 (`nginx:alpine`) übernimmt nur die fertigen, statischen Build-Artefakte
- Nginx dient hier zwei Zwecken gleichzeitig: statischer Webserver für die Angular-App **und**
  Reverse Proxy für `/api/` in Richtung Backend (`proxy_pass http://backend:8080/api/`) — dadurch
  entfällt CORS innerhalb des Compose-Netzwerks vollständig, da der Browser nur noch mit dem
  Frontend-Origin spricht
- Diese Nginx-Proxy-Lösung zahlt sich erst hier aus, weil das Frontend seit Phase 3 bewusst mit
  einem relativen `/api`-Pfad (statt fest codierter `localhost:8080`-URL) arbeitet

### Schritt 3 — Echter Bug entdeckt: Projekt-Namensinkonsistenz ✅

- Beim Docker-Build fiel auf, dass der erzeugte `dist/`-Ordner weiterhin `dnd-character-frontend`
  hieß, obwohl `package.json` bereits umbenannt war — Ursache: `angular.json` führt einen eigenen,
  von `package.json` unabhängigen internen Projekt-Schlüssel
- Bewusste Entscheidung, dies jetzt sauber zu beheben (statt nur den Dockerfile-Pfad anzupassen),
  weil dasselbe Frontend später auch Pathfinder und DSA5 bedienen wird — ein internes „dnd“ im
  Namen wäre irreführend
- Behoben durch Umbenennung des `angular.json`-Projekt-Schlüssels und der beiden
  `buildTarget`-Referenzen unter `serve.configurations` auf `pnp-character-manager-frontend`
- Zusätzlich fehlende Abhängigkeit `@angular/animations` im Produktions-Build entdeckt und behoben
  (vorher nur transitiv vorhanden, für den optimierten Build aber explizit nötig)

### Schritt 4 — Orchestrierung mit Docker Compose (`pnp-character-manager`) ✅

- Ein zentrales `docker-compose.yml` im Meta-Repo startet alle drei Dienste (`postgres`,
  `backend`, `frontend`) gemeinsam; die Backend- und Frontend-Images werden dabei direkt aus den
  Nachbar-Repos gebaut (`build.context: ../dnd-backend` bzw. `../pnp-character-manager-frontend`)
- Startreihenfolge über `healthcheck` (Postgres: `pg_isready`) und `depends_on: condition:
service_healthy` abgesichert — das Backend startet nachweislich erst, wenn die Datenbank
  tatsächlich Verbindungen annimmt, nicht nur, wenn der Postgres-Container existiert
- Internes Networking rein über Docker-Compose-Servicenamen (`postgres`, `backend` als Hostnamen)
  statt über `localhost` oder feste IPs — funktioniert automatisch dank Compose-eigenem DNS
- Geheimnisverwaltung über `.env` (lokal, in `.gitignore`, **niemals committet**) und
  `.env.example` (committet, mit Platzhalter-/Dev-Werten als Vorlage) — bewusste
  Sicherheitsentscheidung, Zugangsdaten und JWT-Secret nicht im Repository zu versionieren

### Schritt 5 — End-to-End-Verifikation ✅

- `docker compose up --build` erfolgreich: alle 6 Flyway-Migrationen liefen gegen eine frische
  Datenbank durch, Healthcheck grün, Backend und Frontend gestartet
- Vollständiger Register → Login → Charakterliste-Ablauf im Browser über den Nginx-Reverse-Proxy
  getestet und funktionsfähig
- Ein während der Verifikation aufgetretener Login-Fehler stellte sich nicht als Bug heraus,
  sondern als eigener Tippfehler bei den Anmeldedaten (E-Mail-Adresse statt Benutzername
  verwendet) — sauber diagnostiziert über die Netzwerk-Analyse im Browser statt vorschneller
  Codeänderungen

**Wichtige Lektion aus dieser Phase:** Der Docker-Build-Prozess deckte zwei reale Inkonsistenzen auf
(Projekt-Namensgebung, fehlende Abhängigkeit), die im normalen Entwicklungsbetrieb (`ng serve`)
nicht sichtbar geworden wären — ein guter Beleg dafür, warum Containerisierung früh im Projekt
sinnvoll ist, statt sie bis kurz vor einem realen Deployment aufzuschieben.

---

## Phase 6 — Kubernetes-Grundlagen (Homeserver) ✅ abgeschlossen

📄 `Phase6_Kubernetes-Grundlagen.pdf` · 📄 `exec-format-error-postmortem.pdf`

### Schritt 1 — Backend-Deployment (`backend.yaml`) ✅

- Deployment + Service (ClusterIP) im Namespace `pnp`
- Zugangsdaten und JWT-Secret über Kubernetes-Secret (`pnp-secrets`), nicht im Manifest selbst
- PersistentVolumeClaim für Postgres (`1Gi`, `ReadWriteOnce`) von Anfang an vorhanden
- Verifiziert über `kubectl port-forward` und `curl` gegen `/api/reference-data/races`

### Schritt 2 — Debugging: `exec format error` ✅

- Backend-Pod ging nach dem Deployment sofort in `CrashLoopBackOff` mit
  `exec /opt/java/openjdk/bin/java: exec format error`
- Architektur-Mismatch, korrupter Image-Push/Pull, BuildKit-Attestations, Layer-Kompression und
  Speicherplatz wurden der Reihe nach ausgeschlossen
- Tatsächliche Ursache: ein dauerhaft korrupter, containerd-interner overlayfs-Snapshot-Cache für
  das Standard-`eclipse-temurin:21-jre`-Basisimage auf dem Homeserver-Node — verursacht durch eine
  Inkompatibilität zwischen der dortigen containerd-Version und Canonicals neuerer
  "Rockcraft"/Chiseled-Ubuntu-Bauweise der Standard-Tags
- Behoben durch Wechsel auf die expliziten `-jammy`-Tag-Varianten (`21-jdk-jammy`, `21-jre-jammy`)
- Zum Nachlesen als eigenes kleines PDF zusammengefasst (`exec-format-error-postmortem.pdf`)

### Schritt 3 — Frontend-Deployment (`frontend.yaml`) ✅

- Frontend-Image gebaut und nach Docker Hub gepusht
- Deployment + Service vom Typ `NodePort` (Port `30080`) — bewusst nur intern im Heimnetz
  erreichbar, keine Exposition ins öffentliche Internet
- Erreichbarkeit im Heimnetz über `http://<homeserver-ip>:30080` verifiziert

### Schritt 4 — CORS zwischen Frontend und Backend ✅

- Login-Versuch über die neue Frontend-Origin schlug zunächst mit `403 Invalid CORS request` fehl
- Behoben durch Ergänzung der `allowedOrigins` in `SecurityConfig.java` um
  `http://<homeserver-ip>:30080`
- Vollständiger Registrierung → Login-Ablauf im Cluster über `curl`/Postman und die Browser-UI
  verifiziert

### Schritt 5 — Liveness- und Readiness-Probes ✅

- Backend: `spring-boot-starter-actuator` ergänzt, `/actuator/health/liveness` und
  `/actuator/health/readiness` über `management.endpoint.health.probes.enabled: true` aktiviert,
  in `SecurityConfig` als `permitAll()` freigegeben
- Postgres: `exec`-Probe mit `pg_isready -U $(POSTGRES_USER)`
- Beobachtung: ein einzelner Postgres-Neustart kurz nach Einführung der Probe (vermutlich
  `initialDelaySeconds` knapp zu kurz für den Postgres-Start), trat bei weiterer Beobachtung nicht
  erneut auf

### Schritt 6 — ConfigMap für nicht-geheime Konfiguration ✅

- `SPRING_DATASOURCE_URL` aus dem Deployment-Manifest in eine eigene `ConfigMap`
  (`backend-config`) ausgelagert, per `configMapKeyRef` referenziert
- Trennt Konfiguration klar von Deployment-Definition und Secrets

### Offen für später (nicht Teil von Phase 6)

- Ingress-Controller / externe Erreichbarkeit — bewusst einer viel späteren Phase vorbehalten
- Helm-Charts, Monitoring (vorgesehen für Phase 8)

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
