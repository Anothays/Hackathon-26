# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Contexte du projet

Application fullstack mandatée par **Capgemini** pour calculer l'empreinte carbone de sites physiques du secteur tertiaire. Les organisations doivent pouvoir mesurer l'impact de leurs bâtiments, identifier les sources d'émissions, comparer plusieurs sites et historiser les données.

Calculer deux composantes :
- **Empreinte construction** : matériaux × facteurs d'émission
- **Empreinte exploitation** : consommation énergétique × facteurs d'émission (ADEME)

KPIs produits : CO₂ total, CO₂/m², CO₂/employé.

## Structure du monorepo

| Dossier | Rôle |
|---------|------|
| `api/` | Backend Spring Boot 4 + PostgreSQL + JWT |
| `web/` | Frontend Angular 21 + Tailwind CSS 4 |

Une app mobile React Native est prévue dans le cahier des charges (palier 2) mais n'est pas encore implémentée.

## Paliers de réalisation

- **Palier 1** *(obligatoire)* — Socle technique : saisie d'un site, calcul CO₂ basique, affichage Angular, historique PostgreSQL.
- **Palier 2** — Dashboard Angular complet (KPIs, graphiques, répartition construction/exploitation) + app mobile React Native (JWT, saisie, consultation).
- **Palier 3** *(bonus)* — Comparaison de sites, historisation avancée, export PDF, intégration API ADEME, heatmap.

## Données de référence (Annexes du cahier des charges)

Site exemple Capgemini Rennes :
- Surface : 11 771 m²
- Consommation énergétique : 1 840 MWh (2025)
- Employés : ~1 800 collaborateurs
- Postes de travail : 1 037
- Matériaux : à sourcer en open-data

Facteurs d'émission : source **ADEME** / bases publiques officielles.

## Commandes

### Backend (`api/`)

```bash
./mvnw spring-boot:run           # Démarrage local
./mvnw package                   # Build JAR
docker compose up --build        # Stack complète (API + PostgreSQL + pgAdmin)
psql -U postgres -d hackathon_db -f SQL/schema.sql  # Init schéma
```

Ports : API → `8080`, PostgreSQL → `5432`, pgAdmin → `5050` (hackathon26@sdv.com / hackathon26)

### Frontend (`web/`)

```bash
npm start       # Dev server → http://localhost:4200
npm run build   # Build production
npm run watch   # Build en mode watch
npm test        # Tests Vitest
```

## Architecture

### Backend

Couches : `Controller → Service → Repository → Entity (JPA)`

- JWT requis sur toutes les routes sauf `/api/auth/*`
- DTOs séparés `request/` et `response/` — ne jamais exposer les entités directement
- Schéma PostgreSQL dans `SQL/schema.sql`

Flux de calcul carbone : données site → `CarbonReportService` → `CarbonReport` (total) + `CarbonReportDetail` (par catégorie : matériaux, énergie, parking)

### Frontend

- Routing dans `app.routes.ts` avec lazy loading par feature
- Signaux pour l'état local, `computed()` pour l'état dérivé
- Services HTTP vers `http://localhost:8080/api`
- Dashboard : graphiques dynamiques (répartition construction/exploitation), KPIs, comparaison multi-sites (palier 3)

## Contraintes techniques imposées

- Facteurs d'émission conformes ADEME / normes officielles
- Architecture modulaire permettant l'ajout futur d'une API externe ADEME
- Déployable en local ou via Docker (multi-campus)