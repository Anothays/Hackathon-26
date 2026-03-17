# Project Brief: Carbon Tracker - Hackathon 2026 (Final Specs)

## 1. Vision Produit & Workflow
Application de gestion et de suivi de l'empreinte carbone pour sites physiques.

### Parcours Utilisateur
- **Authentification** : Connexion obligatoire via JWT. Rôles : `ADMIN` et `USER`.
- **Dashboard** : Visualisation des sites et de leurs rapports respectifs.
- **Gestion des Sites (ADMIN)** : Création via un formulaire regroupant :
    - Infos structurelles (superficie, adresse, employés, etc.).
    - Parkings (saisie par type : Aérien, Sous-sol, Sous-dalle).
    - Matériaux de construction (via recherche API ADEME).
- **Génération automatique** : La validation du formulaire de site génère automatiquement un **Rapport de Construction** initial.
- **Rapports d'Exploitation (ADMIN)** : Création manuelle pour une période donnée en saisissant la consommation énergétique.
- **Gestion Administrative** : CRUD complet sur les utilisateurs et configuration des facteurs d'émission de référence pour les parkings.

## 2. Architecture des Données (PostgreSQL)

### Entités
- **Users** : `id`, `email`, `password`, `role` (ADMIN/USER).
- **Sites** : `id`, `name`, `address`, `city`, `total_surface_m2`, `total_employees`, `construction_year`.
- **ParkingType** : Table de configuration ADMIN. Contient les 3 types (`AÉRIEN`, `SOUS-SOL`, `SOUS-DALLE`) avec leurs noms et facteurs d'émission (`default_fe`) modifiables.
- **SiteParking** : Relation entre `Site` et `ParkingType` avec le champ `spots_count`.
- **Reports** :
    - `id`
    - `site_id` (FK, **NULLABLE**) : Si un site est supprimé, le rapport est conservé mais marqué comme orphelin.
    - `type` (CONSTRUCTION / EXPLOITATION).
    - `total_co2e` : Somme calculée et stockée pour affichage rapide.
    - `start_date` / `end_date` : Pour les rapports d'exploitation.
- **ReportLine** : Lignes de détails (matériaux ou énergie). Stocke le `label`, la `quantity`, l' `unit` et surtout le `fe_value_at_time` (snapshot du facteur d'émission au moment du calcul).

## 3. Intégration API ADEME (Base Empreinte)
L'autocomplete des matériaux et des énergies doit interroger l'API ADEME :
`https://data.ademe.fr/data-fair/api/v1/datasets/base-carboner/lines`

**Contrainte de filtrage (Paramètre `qs`)** :
Pour garantir la qualité des données, chaque requête doit inclure :
`qs=Statut_de_l'élément:"Valide générique" AND Type_de_l'élément:"Facteur d'émission"`

## 4. Logique de Calcul
- **Indicateur Phare** : L'empreinte CO2 équivalente doit être l'unité centrale de l'UI.
- **KPI Surface** : Utiliser `total_surface_m2` du site comme dénominateur pour obtenir le ratio $kgCO_{2}e/m^{2}$.
- **Règle Parking** :
    - Ratio de surface : **25 m² par place** (incluant circulation).
    - Calcul : `spots_count * 25 * FE_parking_type`.

## 5. Cas d'Usage de Référence (Rennes)
- **Site** : 11 771 m², 1 800 employés.
- **Parkings** : 83 aériens, 184 sous-sol, 41 sous-dalle.
- **Energie** : 1 840 MWh (à saisir pour le rapport d'exploitation).