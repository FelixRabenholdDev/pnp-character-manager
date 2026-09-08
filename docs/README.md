# Pen & Paper Charakterverwaltung

Ein Lernprojekt zum Aufbau einer modernen, cloud-nativen Fullstack-Anwendung zur Verwaltung von
Pen-&-Paper-Rollenspielcharakteren — mit dem Ziel, professionelle Java- und Kubernetes-Praxis auf
Industriestandard-Niveau zu erlernen.

## Überblick

Die Anwendung verwaltet Charaktere für mehrere Pen-&-Paper-Regelwerke unter einer gemeinsamen
Oberfläche. Aktuell unterstützt: **D&D 5e (2024)**. Geplant: **Pathfinder 1e**, **Das Schwarze Auge 5**.

## Architektur

Jedes Regelwerk wird durch einen eigenständigen Backend-Service abgebildet, da sich die
Domänenmodelle (Attribute, Fertigkeitssysteme, Formeln) zu stark unterscheiden, um sie sinnvoll in
einem gemeinsamen Datenmodell abzubilden. Ein gemeinsames Angular-Frontend spricht mit allen
Services und lässt Nutzer zwischen Regelwerken wechseln.

Authentifizierung erfolgt zustandslos über JWT — jeder Backend-Service validiert Tokens unabhängig
über ein gemeinsames Secret, ganz ohne Rückfrage bei einem zentralen Auth-Service.

In Kubernetes (geplant) übernimmt ein Ingress-Controller das Routing zu den einzelnen
Regelwerk-Services nach URL-Pfad.

## Repositories

| Repository | Beschreibung | Status |
|---|---|---|
| [`dnd-backend`](https://github.com/FelixRabenholdDev/dnd-backend) | Spring-Boot-API für D&D 5e (2024) | In aktiver Entwicklung |
| [`pnp-character-manager-frontend`](https://github.com/FelixRabenholdDev/pnp-character-manager-frontend) | Gemeinsames Angular-Frontend | In aktiver Entwicklung |
| `pathfinder-backend` | Spring-Boot-API für Pathfinder 1e | Geplant |
| `dsa5-backend` | Spring-Boot-API für DSA 5 | Geplant |

## Tech-Stack

**Backend:** Java 21, Spring Boot 4, Spring Data JPA, Spring Security (JWT), PostgreSQL, Flyway,
JUnit 5, Testcontainers, springdoc-openapi

**Frontend:** Angular 22 (Standalone Components), Angular Material, RxJS, TypeScript

**Infrastruktur (geplant):** Docker, Kubernetes (k3s), GitHub Actions, Helm

## Projektstatus & Roadmap

Eine detaillierte, phasenweise Roadmap mit Lernzielen und aktuellem Fortschritt findet sich unter
[`docs/roadmap.md`](docs/roadmap.md).

**Aktueller Stand:** Backend-Grundlagen und Authentifizierung für D&D 5e abgeschlossen, Frontend-Grundlagen
in Arbeit.

## Lokale Entwicklung

Detaillierte Setup-Anleitungen befinden sich in den jeweiligen Repositories:
- Backend-Setup: siehe README in [`dnd-backend`](https://github.com/FelixRabenholdDev/dnd-backend)
- Frontend-Setup: siehe README in [`pnp-character-manager-frontend`](https://github.com/FelixRabenholdDev/pnp-character-manager-frontend)

## Hintergrund

Dieses Projekt ist als strukturiertes Lernvorhaben entstanden, um praktische Erfahrung mit modernem
Java/Spring-Boot-Backend-Development, Angular-Frontend-Entwicklung und Kubernetes-Deployment auf
einem selbst verwalteten Server zu sammeln — begleitet von schrittweiser, dokumentierter
Weiterentwicklung über mehrere Projektphasen.