# LinkedIn Snowflake Project

## Introduction
Ce projet vise à modéliser et charger une base de données LinkedIn dans Snowflake. Il exploite des fichiers CSV et JSON stockés dans un bucket S3 public. L’objectif est d’obtenir une base de données relationnelle robuste permettant l'analyse des offres d’emploi, des entreprises, des salaires, des compétences, et des industries.

### Membres de l'équipe
- Adem HAJ BOUBAKER
- Meriam MAALEJ
- Tasnime RAHALI

## 🎯 Objectif du projet
Ce projet a pour but de créer une base de données relationnelle LinkedIn à partir de données brutes (CSV & JSON) stockées dans un bucket S3 public, puis de les exploiter via des requêtes analytiques.

## 🛠️ Étapes de mise en œuvre détaillées

### 1. 🏗️ Création de la base de données
```sql
CREATE DATABASE Linkedin;
```
**Pourquoi ?** Cela crée un conteneur logique dans Snowflake pour stocker toutes les tables et objets du projet.

![image](https://github.com/user-attachments/assets/fca793be-b7a4-4304-adbd-a82ef92530ae)

---

### 2. 📦 Création du Stage externe (connexion à S3)
```sql
CREATE OR REPLACE STAGE linkedin_stage
URL = 's3://snowflake-lab-bucket/'
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);
```
**Pourquoi ?** Le stage permet d'accéder directement aux fichiers stockés sur S3 sans les importer localement.
![image](https://github.com/user-attachments/assets/5082d943-beff-412e-88c1-b7928c86a8b5)

---

### 3. 📂 Vérification du contenu du stage
```sql
LIST @linkedin_stage;
```
📸 
**Pourquoi ?** Vérifie que les fichiers sont bien disponibles depuis S3 avant chargement.
![image](https://github.com/user-attachments/assets/b40568b5-1838-41ca-8af6-368f33283035)

---

### 4. 🧱 Création des tables
Création de toutes les tables nécessaires à la modélisation des données LinkedIn (Jobs, Companies, Skills, Industries, etc.).

```sql
-- Créer la table job_posting
CREATE TABLE Jobs_posting (
    job_ID VARCHAR PRIMARY KEY,
    job VARCHAR,
    location VARCHAR,
    company_id VARCHAR,
    company_name VARCHAR,
    work_type VARCHAR,
    full_time_remote VARCHAR,
    no_of_employ VARCHAR,
    no_of_application varchar,
    posted_day_ago varchar,
    alumni varchar,
    Hiring_person VARCHAR,
    linkedin_followers varchar,
    hiring_person_link VARCHAR,
    job_details TEXT
);

```
![image](https://github.com/user-attachments/assets/cc0b6b2d-d10c-49b4-bad8-03fb08a24c97)

```sql
-- Créer la table benefits
  CREATE or replace TABLE Benefits (
    job_id VARCHAR REFERENCES Jobs_posting(job_ID),
    inferred BOOLEAN,
    type VARCHAR
);
```
![image](https://github.com/user-attachments/assets/88b42abd-bb2d-4893-af30-9d4bb5f6d3b1)
```sql
--Créer la table companies
CREATE TABLE Companies (
    company_id VARCHAR PRIMARY KEY,
    name VARCHAR,
    description TEXT,
    company_size INTEGER,
    country VARCHAR,
    state VARCHAR,
    city VARCHAR,
    zip_code VARCHAR,
    address VARCHAR,
    url VARCHAR
);
```
![image](https://github.com/user-attachments/assets/9dbd9bf0-4796-4da3-9d48-af10552c51cd)

```sql
--créer la table company_spacialities
CREATE TABLE Company_specialities (
    company_id VARCHAR REFERENCES Companies(company_id),
    speciality VARCHAR,
    PRIMARY KEY (company_id)
);
```
![image](https://github.com/user-attachments/assets/14bc61b7-be7a-4772-871f-17a88771e34e)

```sql
--créer la table industries
CREATE TABLE industries (
  industry_id varchar PRIMARY KEY,
  industry_name TEXT
); 
```
![image](https://github.com/user-attachments/assets/575eb3a7-209a-4a3d-888c-ed55c3ff9fa4)

```sql
--créer la table company_industries
CREATE TABLE Company_industries (
    company_id VARCHAR REFERENCES Companies(company_id),
    industry VARCHAR REFERENCES Industries(industry_id),
    PRIMARY KEY (company_id)
);
```
![image](https://github.com/user-attachments/assets/d6c27b90-e9cc-4a54-8f90-5329fc27457d)

```sql
--créer la table Employee_counts
CREATE TABLE Employee_counts (
    company_id VARCHAR REFERENCES Companies(company_id),
    employee_count INTEGER,
    follower_count INTEGER,
    time_recorded BIGINT
);

```
![image](https://github.com/user-attachments/assets/0ce96a9b-d6fc-4b75-9b96-b9c48e879b13)

```sql
-- créer la table skills
CREATE TABLE skills (
  skill_abr varchar PRIMARY KEY,
  skill_name TEXT
);
```
![image](https://github.com/user-attachments/assets/4544bcb1-e0b2-482b-862a-e75e235cf2e3)

```sql
--créer la table Job_Skills
CREATE TABLE Job_Skills (
    job_id VARCHAR REFERENCES Jobs_posting(job_ID),
    skill_abr VARCHAR REFERENCES skills(skill_abr),
    PRIMARY KEY (job_id)
);

```
![image](https://github.com/user-attachments/assets/49f232d0-6714-4428-a7ac-63a172570a11)

```sql
--créer la table Job_Industries
CREATE TABLE Job_Industries (
    job_id VARCHAR REFERENCES Jobs_posting(job_ID),
    industry_id varchar REFERENCES Industries(industry_id),
    PRIMARY KEY (job_id)
);
```
![image](https://github.com/user-attachments/assets/d965ce15-e4cc-4058-81fa-4bde4a3dc910)
```sql
--créer la table Salaries 
CREATE TABLE Salaries (
    salary_id VARCHAR PRIMARY KEY,
    job_id VARCHAR REFERENCES Jobs_posting(job_ID),
    max_salary FLOAT,
    med_salary FLOAT,
    min_salary FLOAT,
    pay_period VARCHAR,
    currency VARCHAR,
    compensation_type VARCHAR
);

```
![image](https://github.com/user-attachments/assets/899a9e4c-4389-4dfb-b592-adfbf10fa1fc)

**Pourquoi ?** Cela structure les données en entités relationnelles avec des clés primaires et étrangères assurant l'intégrité.

---

### 5. 🗂️ Création des formats de fichiers

#### Format CSV standard (séparateur virgule)
```sql
CREATE or replace FILE FORMAT csv 
TYPE = 'CSV' 
FIELD_DELIMITER = ',' 
RECORD_DELIMITER = '\n' 
SKIP_HEADER = 1
field_optionally_enclosed_by = '"'
null_if = ('');
```
![image](https://github.com/user-attachments/assets/25873e17-2d75-4482-9af8-f59cfbeeb6b4)

#### Format spécifique pour `job_postings.csv` (séparateur `;`)
```sql
CREATE or replace FILE FORMAT csv_job_posting 
TYPE = 'CSV' 
FIELD_DELIMITER = ';' 
RECORD_DELIMITER = '\n' 
SKIP_HEADER = 1
field_optionally_enclosed_by = '"'
null_if = ('');
```
![image](https://github.com/user-attachments/assets/1fdeb1c6-cc6c-4d4a-89e9-873d7b6b90a2)

**Pourquoi ?** Adapter le format au séparateur exact du fichier permet une lecture correcte. Le fichier `job_postings.csv` est séparé par `;`.

#### Format JSON avec tableau racine
```sql
CREATE OR REPLACE FILE FORMAT json_format
TYPE = 'JSON'
STRIP_OUTER_ARRAY = TRUE;
```
![image](https://github.com/user-attachments/assets/fadd5b7d-eba0-4b4c-933d-859df9fb31d8)

**Pourquoi ?** Le JSON contient un tableau racine (`[ {...}, {...} ]`). Cette option permet de lire chaque objet du tableau comme une ligne.
![image](https://github.com/user-attachments/assets/a404e20f-dea9-4f50-b811-c53807ca6be2)

---

### 6. 🔄 Chargement des fichiers CSV dans les tables
```sql
COPY INTO linkedin.public.jobs_posting
FROM @linkedin_stage/job_postings.csv
FILE_FORMAT = csv_job_posting;
```
![image](https://github.com/user-attachments/assets/c69162c2-05f0-4a4f-ad14-29fe0a6d804b)

```sql
COPY INTO linkedin.public.benefits
FROM @linkedin_stage/benefits.csv
FILE_FORMAT = csv;
```
![image](https://github.com/user-attachments/assets/5a81fa78-40c9-4e35-9bc1-afc760d3f291)

```sql
COPY INTO linkedin.public.Employee_counts
FROM @linkedin_stage/employee_counts.csv
FILE_FORMAT  = csv;
```
![image](https://github.com/user-attachments/assets/16da62f6-1895-4df3-8b64-58c20302f645)

```sql
COPY INTO linkedin.public.skills
FROM @linkedin_stage/skills.csv
FILE_FORMAT  = csv;
```
![image](https://github.com/user-attachments/assets/a0f92053-6997-4896-8cc0-2c3ccd32b868)

```sql
COPY INTO linkedin.public.Job_Skills
FROM @linkedin_stage/job_skills.csv
FILE_FORMAT  = csv;

```
![image](https://github.com/user-attachments/assets/baeae821-228d-4ae4-aafa-dea84912cba3)
```sql
COPY INTO linkedin.public.Salaries
FROM @linkedin_stage/salaries.csv
FILE_FORMAT  = csv;
```
![image](https://github.com/user-attachments/assets/ab5961ed-a2ae-4788-ab86-bcc9cca1ef6d)

**Pourquoi ?** Cette commande insère les données dans les tables Snowflake à partir du stage.

---

### 7. 🧪 Utilisation de la table tampon JSON

#### Création de la table tampon
```sql
CREATE OR REPLACE TABLE json_buffer (v VARIANT);
```
![image](https://github.com/user-attachments/assets/83ce4c75-2bd3-4f3a-9f61-0a781bd6baea)
**Pourquoi ?** Permet de parser les fichiers JSON via `VARIANT` pour ensuite les insérer proprement.
#### Chargement des JSONs
```sql
COPY INTO json_buffer FROM @linkedin_stage/industries.json  FILE_FORMAT = json_format;
```
![image](https://github.com/user-attachments/assets/65859f99-3bb0-488d-8dd4-33cb028891de)

#### Verfication de chargement
```sql
select * from json_buffer;
```
![image](https://github.com/user-attachments/assets/58351da0-e74b-415e-805c-ffe6bc7b58ec)
#### Insérer dans la table industries
```sql
INSERT INTO industries (industry_id, industry_name)
SELECT 
  v:industry_id::string,
  NULLIF(v:industry_name::string, '') AS industry_name
FROM json_buffer;
```
![image](https://github.com/user-attachments/assets/19a9266c-3fa1-4242-bf05-b31a98062213)
#### Verfication de insertion
```sql
select * from json_buffer;
```
![image](https://github.com/user-attachments/assets/5678b112-8ff6-447c-ab47-d7265442eb15)

⚠️ À chaque nouvelle insertion :
```sql
TRUNCATE TABLE json_buffer;
```
![image](https://github.com/user-attachments/assets/6f56c90e-f68f-41da-981d-859a5f5eb214)

**Pourquoi ?** Vide la table tampon pour éviter la contamination avec les données précédentes.
#### Chargement de table companies
```sql
COPY INTO json_buffer FROM @linkedin_stage/companies.json FILE_FORMAT = json_format;
```
![image](https://github.com/user-attachments/assets/02328de6-a876-4475-9553-8c0b5f733ff9)
#### Verfication de chargement
```sql
select * from json_buffer ;
```
![image](https://github.com/user-attachments/assets/f9314692-4653-49f0-ab71-b2f535db3887)
#### Insérer dans la table companies
```sql
INSERT INTO companies (
  company_id, name, description, company_size,
  country, state, city, zip_code, address, url
)
SELECT 
  v:company_id::string,
  v:name::string,
  NULLIF(v:description::string, '') as description,
  v:company_size::int,
  v:country::string,
  NULLIF(v:state::string, '') as state,
  v:city::string,
  NULLIF(v:zip_code::string, '') as zip_code,
  NULLIF(v:address::string, '') as address,
  v:url::string
FROM json_buffer;

```
![image](https://github.com/user-attachments/assets/ddc58840-2685-438e-b4ae-4473494a374d)

#### Afficher la table companies
```sql
select * from companies;
```
![image](https://github.com/user-attachments/assets/8a953c93-e8e2-4c74-b7ab-5da7373d88aa)

#### Vider la table tampon
```sql
TRUNCATE TABLE json_buffer;
```
![image](https://github.com/user-attachments/assets/d71e82e7-2482-4350-a2d8-096d330c011f)

#### Chargement et insertion de company_specialities
```sql
COPY INTO json_buffer
FROM @linkedin_stage/company_specialities.json
FILE_FORMAT = json_format;
```
![image](https://github.com/user-attachments/assets/61fa8f6b-8c34-43d5-8a23-63cce4462d2d)

```sql
insert into company_specialities(company_id,speciality) 
select
  v:company_id::string,
  v:speciality::string
FROM json_buffer;
```
![image](https://github.com/user-attachments/assets/34c7d385-3f79-4122-bacb-fb802fa18236)

#### Afficher la table company_specialities
```sql
select * from company_specialities;
```
![image](https://github.com/user-attachments/assets/4fb7ffe2-fd58-4d4e-977a-b152df6b2173)

#### Vider la table tampon
```
TRUNCATE TABLE json_buffer;
```
![image](https://github.com/user-attachments/assets/d71e82e7-2482-4350-a2d8-096d330c011f)


#### Chargement et insertion de company_industries
```sql
COPY INTO json_buffer
FROM @linkedin_stage/company_industries.json
FILE_FORMAT = json_format;

```
![image](https://github.com/user-attachments/assets/a2151869-63c5-4db2-b526-3057bd175f49)

---
```sql
insert into  company_industries(company_id, industry)
SELECT 
  v:company_id::string,
  v:industry::string
FROM json_buffer;
```
![image](https://github.com/user-attachments/assets/2cb9e19d-0476-423c-82b6-cd28f4ce1c3c)
#### Afficher  la table company_industries
```sql
select * from company_industries;
```
![image](https://github.com/user-attachments/assets/612cda0a-ff36-4b1a-8de8-7903695e10f8)
#### Vider la table tampon
```sql
TRUNCATE TABLE json_buffer;
```
![image](https://github.com/user-attachments/assets/ef7e7cc0-31a2-4f38-8fa2-6b3f1f4f81a0)
#### Chargement et insertion de job_industries
```sql
COPY INTO json_buffer
FROM @linkedin_stage/job_industries.json
FILE_FORMAT = json_format;
```
![image](https://github.com/user-attachments/assets/2f51bb2a-75d0-4334-bf53-a9e8faca3f5f)
####
```sql
insert into job_industries(job_id,industry_id)
SELECT 
  v:job_id::string,
  v:industry_id::string
FROM json_buffer;
```
![image](https://github.com/user-attachments/assets/458a39d9-c608-43a3-a218-e1c51248dacf)

## 📈 Requêtes analytiques finales

### 🔝 Top 10 des jobs les plus publiés par industrie
```sql
SELECT j.job AS job_title, i.industry_name, COUNT(*) AS nb_offres
FROM jobs_posting j
JOIN job_industries ji ON j.job_ID = ji.job_id
JOIN industries i ON ji.industry_id = i.industry_id
GROUP BY j.job, i.industry_name
ORDER BY nb_offres DESC
LIMIT 10;
```
**Pourquoi ?** Pour avoir le top 10 des jobs les plus publiés par industrié 
![image](https://github.com/user-attachments/assets/569a97a1-6658-489e-a1c5-38d7177134a5)
Le probléme qui il ya aucune intersection entre les tables 
```sql
SELECT COUNT(*) FROM jobs_posting j
JOIN job_industries ji ON j.job_ID = ji.job_id;
```
![image](https://github.com/user-attachments/assets/86c2d2d4-e726-4492-9eeb-d4d601cece75)

---

### 🏢 Répartition des offres par taille d’entreprise
```sql
SELECT c.company_size, COUNT(*) AS nb_jobs
FROM Jobs_posting j
JOIN Companies c ON j.company_id = c.company_id
GROUP BY c.company_size
ORDER BY c.company_size;
```
![image](https://github.com/user-attachments/assets/9dde2024-0baf-4419-9d56-b26ac6c5f60a)

---

### 🏠 Répartition des offres par type de présence (Remote / On-site)
```sql
SELECT 
    work_type,
    COUNT(*) AS nb_jobs
FROM jobs_posting
GROUP BY work_type
ORDER BY nb_jobs DESC;
```
![image](https://github.com/user-attachments/assets/4cf800c0-fc6c-4727-ba15-81e2bcf6b651)

---

### ⏱️ Répartition par type d’emploi (Temps plein, partiel, stage)
```sql
SELECT 
    CASE
        -- Si ça contient "full-time" mais pas d'autres types
        WHEN full_time_remote ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%internship%' 
             AND full_time_remote NOT ILIKE '%part-time%' 
             
        THEN 'Full-time'

        -- Si ça contient uniquement "part-time"
        WHEN full_time_remote ILIKE '%part-time%' 
             AND full_time_remote NOT ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%internship%' 
             
        THEN 'Part-time'

        -- Si ça contient uniquement "internship"
        WHEN full_time_remote ILIKE '%internship%' 
             AND full_time_remote NOT ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%part-time%' 
        THEN 'Internship'

        -- Tous les autres cas (mélanges, erreurs, contrat, temporaire, etc.)
        ELSE 'Autre/Non précisé'
    END AS type_emploi,
    COUNT(*) AS nb_jobs
FROM jobs_posting
GROUP BY type_emploi
ORDER BY COUNT(*) DESC;
```
**Pourquoi ?**
J’utilise ILIKE avec le symbole % (wildcard) pour rechercher de manière flexible des mots-clés comme "full-time", "part-time" ou "internship", peu importe leur position dans la chaîne (début, milieu ou fin).

Exemples de valeurs dans la colonne full_time_remote :

- Full-time · Entry level
- Part-time · Mid-Senior level
- Full-time · Executive
- Full-time · Internship

Certaines lignes peuvent contenir plusieurs types comme :

Full-time Internship Dans ce cas, on ne doit pas catégoriser cette offre comme "Full-time", car ce n’est pas un vrai CDI à temps plein mais un stage.
![image](https://github.com/user-attachments/assets/f59af5af-86ae-499e-b85e-f770daebff83)

---
### Streamlit
```python
# Import des packages
import streamlit as st
from snowflake.snowpark.context import get_active_session
import pandas as pd

# Récupération de la session active Snowflake
session = get_active_session()

# Titre de l'app
st.title("📊 Analyse des Offres d’Emploi ")

# --- 1. Répartition par taille d’entreprise ---
st.subheader("🏢 Offres par taille d’entreprise")

query1 = """
SELECT 
    CASE
        -- Si ça contient "full-time" mais pas d'autres types
        WHEN full_time_remote ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%internship%' 
             AND full_time_remote NOT ILIKE '%part-time%' 
             
        THEN 'Full-time'

        -- Si ça contient uniquement "part-time"
        WHEN full_time_remote ILIKE '%part-time%' 
             AND full_time_remote NOT ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%internship%' 
             
        THEN 'Part-time'

        -- Si ça contient uniquement "internship"
        WHEN full_time_remote ILIKE '%internship%' 
             AND full_time_remote NOT ILIKE '%full-time%' 
             AND full_time_remote NOT ILIKE '%part-time%' 
        THEN 'Internship'

        -- Tous les autres cas (mélanges, erreurs, contrat, temporaire, etc.)
        ELSE 'Autre/Non précisé'
    END AS type_emploi,
    COUNT(*) AS nb_jobs
FROM jobs_posting
GROUP BY type_emploi
ORDER BY COUNT(*) DESC;
"""
df_size = session.sql(query1).to_pandas
```
![image](https://github.com/user-attachments/assets/4b75a311-a68e-4986-933c-eaa349bf3c85)
![image](https://github.com/user-attachments/assets/6327fe76-6783-4a07-89a1-e43478bc3ae5)

![image](https://github.com/user-attachments/assets/29282b0b-ac79-4308-94c6-1f9d63c911f6)

![image](https://github.com/user-attachments/assets/b0e96273-a8ba-44c4-90f7-2016d8939411)

---
