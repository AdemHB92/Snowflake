# LinkedIn Snowflake Project

## Introduction
Ce projet vise à modéliser et charger une base de données LinkedIn dans Snowflake. Il exploite des fichiers CSV et JSON stockés dans un bucket S3 public. L’objectif est d’obtenir une base de données relationnelle robuste permettant l'analyse des offres d’emploi, des entreprises, des salaires, des compétences, et des industries.

### Membres de l'équipe
- [Votre nom ici]
- [Autres membres le cas échéant]

## Objectifs du projet
- Créer une base de données complète à partir de fichiers sources structurés (CSV et JSON).
- Utiliser les fonctionnalités de Snowflake comme les *stages*, les *file formats* et les *variant types* pour un traitement de données semi-structurées.
- Charger et nettoyer les données efficacement pour une utilisation future en BI ou Data Science.

## Prérequis
Avant de commencer, assurez-vous d'avoir :
- Un compte Snowflake actif
- Accès à un rôle avec les droits nécessaires pour créer bases, tables et stages
- SQL Editor de Snowflake ou Snowsight

## Étapes de mise en place

### 1. Création de la base de données
```sql
CREATE DATABASE Linkedin;
```

### 2. Création du stage externe
```sql
CREATE OR REPLACE STAGE linkedin_stage
  URL = 's3://snowflake-lab-bucket/'
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);
```

### 3. Vérification du contenu du stage
```sql
LIST @linkedin_stage;
```

### 4. Création des tables
```sql
CREATE TABLE Jobs_posting (...);
CREATE TABLE Benefits (...);
CREATE TABLE Companies (...);
CREATE TABLE Company_specialities (...);
CREATE TABLE Employee_counts (...);
CREATE TABLE industries (...);
CREATE TABLE Job_Industries (...);
CREATE TABLE Company_industries (...);
CREATE TABLE skills (...);
CREATE TABLE Job_Skills (...);
CREATE TABLE Salaries (...);
```

(Définir chaque table comme dans le script original.)

### 5. Définition des formats de fichiers
```sql
CREATE or replace FILE FORMAT csv TYPE = 'CSV' ...;
CREATE or replace FILE FORMAT csv_job_posting TYPE = 'CSV' FIELD_DELIMITER = ';' ...;
CREATE OR REPLACE FILE FORMAT json_format TYPE = 'JSON' STRIP_OUTER_ARRAY = TRUE;
```

### 6. Chargement des données CSV dans les tables
```sql
COPY INTO linkedin.public.jobs_posting FROM @linkedin_stage/job_postings.csv FILE_FORMAT = csv_job_posting;
COPY INTO linkedin.public.benefits FROM @linkedin_stage/benefits.csv FILE_FORMAT = csv;
...
```

### 7. Traitement des fichiers JSON avec table tampon
```sql
CREATE OR REPLACE TABLE json_buffer (v VARIANT);

TRUNCATE TABLE json_buffer;

COPY INTO json_buffer FROM @linkedin_stage/industries.json FILE_FORMAT = json_format;
INSERT INTO industries (...) SELECT v:... FROM json_buffer;

... (répéter pour chaque JSON)
```

## Structure de la base (résumé)
| Table | Description |
|-------|-------------|
| `Jobs_posting` | Offres d’emploi |
| `Companies` | Informations sur les entreprises |
| `Salaries` | Données salariales |
| `Skills` | Compétences |
| `Industries` | Secteurs d’activité |
| `Job_Skills`, `Job_Industries` | Relations entre offres, compétences, et industries |

## Recommandations
- Utilisez des vues pour regrouper et analyser les données de manière plus accessible.
- Pensez à automatiser les étapes de chargement avec des *Snowflake Tasks* si récurrentes.
