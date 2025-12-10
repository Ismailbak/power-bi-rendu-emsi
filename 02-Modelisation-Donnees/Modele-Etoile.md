# ⭐ Modèle en Étoile - EMSI BI

## Vue d'Ensemble

Ce document décrit le modèle de données dimensionnel en étoile (Star Schema) utilisé pour le projet Power BI de l'EMSI. Le modèle optimise les performances des requêtes et facilite l'analyse multidimensionnelle.

---

## 1. Architecture du Modèle

```
                        ┌─────────────────┐
                        │  Dim_Calendrier │
                        │   (Date Table)  │
                        └────────┬────────┘
                                 │
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        │                        │                        │
┌───────▼────────┐     ┌────────▼────────┐     ┌────────▼────────┐
│ Dim_Etudiants  │────►│ Fait_Inscriptions│◄────│  Dim_Filieres   │
└────────────────┘     └─────────────────┘     └─────────────────┘
                                │
                                │
                       ┌────────▼────────┐
                       │   Dim_Niveaux   │
                       └─────────────────┘


        ┌─────────────────┐
        │  Dim_Calendrier │
        └────────┬────────┘
                 │
                 │
        ┌────────▼────────┐
        │  Dim_Etudiants  │────┐
        └─────────────────┘    │
                               │
        ┌─────────────────┐    │
        │  Dim_Matieres   │────┼─────►┌──────────────────┐
        └─────────────────┘    │      │ Fait_Absences    │
                               └──────│   (Etudiants)    │
                                      └──────────────────┘


        ┌─────────────────┐
        │  Dim_Calendrier │
        └────────┬────────┘
                 │
                 │
        ┌────────▼────────┐
        │  Dim_Etudiants  │────────────►┌──────────────────┐
        └─────────────────┘             │ Fait_Paiements   │
                                        └──────────────────┘


        ┌─────────────────┐
        │  Dim_Calendrier │
        └────────┬────────┘
                 │
                 │
        ┌────────▼────────┐
        │ Dim_Professeurs │────┐
        └─────────────────┘    │
                               │
        ┌─────────────────┐    │
        │  Dim_Matieres   │────┼─────►┌──────────────────┐
        └─────────────────┘    │      │  Fait_Planning   │
                               │      │  (Enseignement)  │
        ┌─────────────────┐    │      └──────────────────┘
        │  Dim_Filieres   │────┘
        └─────────────────┘


        ┌─────────────────┐
        │  Dim_Calendrier │
        └────────┬────────┘
                 │
                 │
        ┌────────▼────────┐
        │ Dim_Professeurs │────────────►┌──────────────────┐
        └─────────────────┘             │ Fait_Absences    │
                                        │  (Professeurs)   │
                                        └──────────────────┘
```

---

## 2. Tables de Faits (Fact Tables)

### 2.1 Fait_Inscriptions

**Description** : Enregistre chaque inscription d'un étudiant pour une année universitaire donnée.

**Granularité** : Une ligne par inscription (Étudiant × Filière × Niveau × Année)

**Type** : Transaction Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Inscription` | INT | PK | Identifiant unique |
| `ID_Etudiant` | INT | FK | Lien vers Dim_Etudiants |
| `ID_Filiere` | INT | FK | Lien vers Dim_Filieres |
| `ID_Niveau` | INT | FK | Lien vers Dim_Niveaux |
| `Date_Inscription` | DATE | FK | Lien vers Dim_Calendrier |
| `Type_Inscription` | VARCHAR(20) | - | Nouvelle/Réinscription |
| `Montant_Annuel` | DECIMAL(10,2) | Measure | Frais scolarité |
| `Statut_Inscription` | VARCHAR(20) | - | En cours/Validé/Abandon |
| `Annee_Universitaire` | VARCHAR(10) | - | Ex: 2024-2025 |

**Mesures Calculées**
- Nombre d'inscriptions
- Montant total des frais
- Taux de réinscription
- Évolution des inscriptions

---

### 2.2 Fait_Absences_Etudiants

**Description** : Capture chaque absence d'étudiant pour une matière spécifique.

**Granularité** : Une ligne par absence (Étudiant × Matière × Date)

**Type** : Transaction Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Absence` | INT | PK | Identifiant unique |
| `ID_Etudiant` | INT | FK | Lien vers Dim_Etudiants |
| `ID_Matiere` | INT | FK | Lien vers Dim_Matieres |
| `Date_Absence` | DATE | FK | Lien vers Dim_Calendrier |
| `Type_Absence` | VARCHAR(20) | - | Justifiée/Non justifiée |
| `Duree_Heures` | INT | Measure | Nombre d'heures |
| `Justification` | VARCHAR(200) | - | Raison (si justifiée) |

**Mesures Calculées**
- Nombre total d'absences
- Taux d'absentéisme
- Heures d'absence cumulées
- Étudiants à risque

---

### 2.3 Fait_Paiements

**Description** : Enregistre chaque transaction de paiement effectuée par un étudiant.

**Granularité** : Une ligne par paiement (Étudiant × Date × Transaction)

**Type** : Transaction Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Paiement` | INT | PK | Identifiant unique |
| `ID_Etudiant` | INT | FK | Lien vers Dim_Etudiants |
| `Date_Paiement` | DATE | FK | Lien vers Dim_Calendrier |
| `Montant_Paye` | DECIMAL(10,2) | Measure | Montant encaissé |
| `Mode_Paiement` | VARCHAR(20) | - | Espèces/Chèque/Virement |
| `Reference_Paiement` | VARCHAR(50) | - | N° reçu/chèque |
| `Annee_Universitaire` | VARCHAR(10) | - | Année concernée |
| `Trimestre` | VARCHAR(5) | - | T1/T2/T3 |
| `Montant_Attendu` | DECIMAL(10,2) | Measure | Montant facturé |
| `Solde_Restant` | DECIMAL(10,2) | Measure | Reste à payer |

**Mesures Calculées**
- Total encaissé
- Total attendu
- Taux de recouvrement
- Montant impayé
- Délai moyen de paiement

---

### 2.4 Fait_Planning_Enseignement

**Description** : Planification et réalisation des cours par professeur.

**Granularité** : Une ligne par assignation (Professeur × Matière × Filière × Période)

**Type** : Periodic Snapshot Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Planning` | INT | PK | Identifiant unique |
| `ID_Professeur` | INT | FK | Lien vers Dim_Professeurs |
| `ID_Matiere` | INT | FK | Lien vers Dim_Matieres |
| `ID_Filiere` | INT | FK | Lien vers Dim_Filieres |
| `Date_Debut` | DATE | FK | Lien vers Dim_Calendrier |
| `Annee_Universitaire` | VARCHAR(10) | - | Ex: 2024-2025 |
| `Semestre` | VARCHAR(5) | - | S1/S2 |
| `Heures_Prevues` | INT | Measure | Volume horaire planifié |
| `Heures_Realisees` | INT | Measure | Heures effectuées |
| `Taux_Horaire` | DECIMAL(8,2) | Measure | Coût par heure |
| `Jour_Semaine` | VARCHAR(10) | - | Lundi/Mardi... |

**Mesures Calculées**
- Total heures prévues
- Total heures réalisées
- Taux de réalisation
- Coût total enseignement
- Charge par professeur

---

### 2.5 Fait_Absences_Professeurs

**Description** : Enregistre les absences des professeurs.

**Granularité** : Une ligne par absence (Professeur × Date)

**Type** : Transaction Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Absence_Prof` | INT | PK | Identifiant unique |
| `ID_Professeur` | INT | FK | Lien vers Dim_Professeurs |
| `Date_Absence` | DATE | FK | Lien vers Dim_Calendrier |
| `Type_Absence` | VARCHAR(30) | - | Maladie/Congé/Autre |
| `Justifiee` | BIT | - | 0/1 |
| `Cours_Annules` | INT | Measure | Heures perdues |
| `Remplacement` | BIT | - | 0/1 |
| `ID_Remplacant` | INT | FK | Professeur remplaçant |

**Mesures Calculées**
- Nombre d'absences professeurs
- Taux d'absence
- Heures cours annulés
- Taux de remplacement

---

### 2.6 Fait_Resultats

**Description** : Notes et résultats des étudiants par matière.

**Granularité** : Une ligne par résultat (Étudiant × Matière × Session)

**Type** : Transaction Fact Table

| Colonne | Type | Clé | Description |
|---------|------|-----|-------------|
| `ID_Resultat` | INT | PK | Identifiant unique |
| `ID_Etudiant` | INT | FK | Lien vers Dim_Etudiants |
| `ID_Matiere` | INT | FK | Lien vers Dim_Matieres |
| `Date_Examen` | DATE | FK | Lien vers Dim_Calendrier |
| `Note` | DECIMAL(4,2) | Measure | Note obtenue (/20) |
| `Statut` | VARCHAR(20) | - | Admis/Ajourné/Absent |
| `Session` | VARCHAR(20) | - | Normale/Rattrapage |
| `Annee_Universitaire` | VARCHAR(10) | - | Année concernée |

**Mesures Calculées**
- Moyenne générale
- Taux de réussite
- Nombre d'admis
- Nombre de redoublants

---

## 3. Tables de Dimensions (Dimension Tables)

### 3.1 Dim_Calendrier (Date Dimension)

**Type** : Dimension Date (générée)

**Rôle** : Permet l'analyse temporelle (Intelligence Temporelle)

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `Date` | DATE | 2024-01-15 | PK - Date complète |
| `Annee` | INT | 2024 | Année |
| `Mois` | INT | 1 | Numéro du mois (1-12) |
| `Mois_Nom` | VARCHAR(20) | Janvier | Nom du mois |
| `Mois_Court` | VARCHAR(3) | Jan | Abréviation |
| `Trimestre` | VARCHAR(5) | T1 | Trimestre (T1-T4) |
| `Semestre` | INT | 1 | Semestre (1-2) |
| `Semaine_Annee` | INT | 3 | Numéro semaine ISO |
| `Jour_Mois` | INT | 15 | Jour du mois (1-31) |
| `Jour_Annee` | INT | 15 | Jour de l'année (1-366) |
| `Jour_Semaine` | INT | 1 | 1=Lundi, 7=Dimanche |
| `Jour_Semaine_Nom` | VARCHAR(10) | Lundi | Nom du jour |
| `Jour_Semaine_Court` | VARCHAR(3) | Lun | Abréviation |
| `Est_Weekend` | BIT | 0 | 0=Non, 1=Oui |
| `Est_Jour_Ferie` | BIT | 0 | 0=Non, 1=Oui |
| `Annee_Universitaire` | VARCHAR(10) | 2023-2024 | Année académique |
| `Semestre_Univ` | VARCHAR(5) | S1 | Semestre universitaire |
| `Debut_Mois` | DATE | 2024-01-01 | Premier jour du mois |
| `Fin_Mois` | DATE | 2024-01-31 | Dernier jour du mois |

**Hiérarchies**
- Année > Trimestre > Mois > Date
- Année Universitaire > Semestre > Mois > Date
- Année > Semaine > Date

---

### 3.2 Dim_Etudiants

**Type** : SCD Type 1 (écrasement)

**Rôle** : Informations détaillées sur les étudiants

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `ID_Etudiant` | INT | 1001 | PK - Identifiant unique |
| `Code_Massar` | VARCHAR(20) | M123456789 | Code national |
| `Nom` | VARCHAR(100) | ALAMI | Nom de famille |
| `Prenom` | VARCHAR(100) | Ahmed | Prénom |
| `Nom_Complet` | VARCHAR(200) | ALAMI Ahmed | Nom + Prénom |
| `Date_Naissance` | DATE | 2002-05-15 | Date de naissance |
| `Age` | INT | 22 | Âge calculé |
| `Sexe` | VARCHAR(1) | M | M/F |
| `Ville` | VARCHAR(50) | Casablanca | Ville de résidence |
| `Region` | VARCHAR(50) | Grand Casablanca | Région |
| `Telephone` | VARCHAR(20) | 0612345678 | Contact |
| `Email` | VARCHAR(100) | ahmed.a@emsi.ma | Email institutionnel |
| `Date_Premiere_Inscription` | DATE | 2020-09-01 | Première inscription EMSI |
| `Anciennete_Annees` | INT | 4 | Années à l'EMSI |
| `Statut_Actuel` | VARCHAR(20) | Actif | Actif/Diplômé/Abandon |
| `Filiere_Actuelle` | VARCHAR(50) | Génie Informatique | Filière en cours |
| `Niveau_Actuel` | VARCHAR(10) | 4ème année | Niveau actuel |

**Hiérarchies**
- Région > Ville
- Statut > Filière > Niveau

---

### 3.3 Dim_Filieres

**Type** : SCD Type 1

**Rôle** : Programmes d'études proposés

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `ID_Filiere` | INT | 1 | PK - Identifiant unique |
| `Code_Filiere` | VARCHAR(10) | GI | Code court |
| `Nom_Filiere` | VARCHAR(100) | Génie Informatique | Nom complet |
| `Departement` | VARCHAR(50) | Informatique | Département |
| `Cycle` | VARCHAR(20) | Ingénierie | Licence/Master/Ingénierie |
| `Duree_Annees` | INT | 5 | Durée du programme |
| `Capacite_Max` | INT | 120 | Places disponibles |
| `Tarif_Annuel` | DECIMAL(10,2) | 45000.00 | Frais annuels (MAD) |
| `Date_Creation` | DATE | 2010-09-01 | Date création filière |
| `Statut_Filiere` | VARCHAR(20) | Active | Active/Suspendue |
| `Chef_Filiere` | VARCHAR(100) | Pr. BENNANI | Responsable |

**Hiérarchies**
- Département > Cycle > Filière

---

### 3.4 Dim_Niveaux

**Type** : Dimension statique

**Rôle** : Niveaux d'études

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `ID_Niveau` | INT | 1 | PK - Identifiant unique |
| `Code_Niveau` | VARCHAR(10) | 1A | Code court |
| `Nom_Niveau` | VARCHAR(50) | 1ère année | Nom complet |
| `Ordre_Niveau` | INT | 1 | Ordre séquentiel (1-5) |
| `Cycle` | VARCHAR(20) | Préparatoire | Prépa/Cycle Ingénieur |

---

### 3.5 Dim_Matieres

**Type** : SCD Type 1

**Rôle** : Matières enseignées

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `ID_Matiere` | INT | 101 | PK - Identifiant unique |
| `Code_Matiere` | VARCHAR(10) | INFO101 | Code matière |
| `Nom_Matiere` | VARCHAR(100) | Programmation Java | Nom complet |
| `Categorie` | VARCHAR(50) | Informatique | Catégorie |
| `Coefficient` | DECIMAL(3,1) | 3.0 | Coefficient |
| `Volume_Horaire` | INT | 42 | Heures totales |
| `Type_Cours` | VARCHAR(20) | Cours/TD/TP | Type |
| `Semestre` | VARCHAR(5) | S1 | Semestre enseignement |

**Hiérarchies**
- Catégorie > Type > Matière

---

### 3.6 Dim_Professeurs

**Type** : SCD Type 2 (historisation du statut et département)

**Rôle** : Corps enseignant

| Colonne | Type | Exemple | Description |
|---------|------|---------|-------------|
| `ID_Professeur` | INT | 501 | PK - Identifiant unique |
| `Code_Professeur` | VARCHAR(10) | PROF501 | Code interne |
| `Nom_Complet` | VARCHAR(200) | Dr. BENNANI Ahmed | Nom complet |
| `Specialite` | VARCHAR(100) | Réseaux Informatiques | Domaine expertise |
| `Departement` | VARCHAR(50) | Informatique | Département |
| `Statut` | VARCHAR(20) | Permanent | Permanent/Vacataire |
| `Grade` | VARCHAR(50) | Professeur | Grade académique |
| `Date_Embauche` | DATE | 2015-09-01 | Date d'entrée |
| `Anciennete_Annees` | INT | 9 | Années d'ancienneté |
| `Email` | VARCHAR(100) | a.bennani@emsi.ma | Email professionnel |
| `Telephone` | VARCHAR(20) | 0612345678 | Contact |
| `Taux_Horaire` | DECIMAL(8,2) | 250.00 | Salaire horaire (MAD) |
| `Date_Debut_Validite` | DATE | 2015-09-01 | SCD Type 2 - Début |
| `Date_Fin_Validite` | DATE | 9999-12-31 | SCD Type 2 - Fin |
| `Est_Actuel` | BIT | 1 | Version actuelle |

**Hiérarchies**
- Département > Statut > Grade > Professeur

---

## 4. Relations et Cardinalités

### 4.1 Tableau des Relations

| Table Fait | Dimension | Relation | Cardinalité | Direction Filtre |
|------------|-----------|----------|-------------|------------------|
| Fait_Inscriptions | Dim_Etudiants | ID_Etudiant | N:1 | Simple (One → Many) |
| Fait_Inscriptions | Dim_Filieres | ID_Filiere | N:1 | Simple |
| Fait_Inscriptions | Dim_Niveaux | ID_Niveau | N:1 | Simple |
| Fait_Inscriptions | Dim_Calendrier | Date_Inscription | N:1 | Simple |
| Fait_Absences_Etudiants | Dim_Etudiants | ID_Etudiant | N:1 | Simple |
| Fait_Absences_Etudiants | Dim_Matieres | ID_Matiere | N:1 | Simple |
| Fait_Absences_Etudiants | Dim_Calendrier | Date_Absence | N:1 | Simple |
| Fait_Paiements | Dim_Etudiants | ID_Etudiant | N:1 | Simple |
| Fait_Paiements | Dim_Calendrier | Date_Paiement | N:1 | Simple |
| Fait_Planning_Enseignement | Dim_Professeurs | ID_Professeur | N:1 | Simple |
| Fait_Planning_Enseignement | Dim_Matieres | ID_Matiere | N:1 | Simple |
| Fait_Planning_Enseignement | Dim_Filieres | ID_Filiere | N:1 | Simple |
| Fait_Planning_Enseignement | Dim_Calendrier | Date_Debut | N:1 | Simple |
| Fait_Absences_Professeurs | Dim_Professeurs | ID_Professeur | N:1 | Simple |
| Fait_Absences_Professeurs | Dim_Calendrier | Date_Absence | N:1 | Simple |
| Fait_Resultats | Dim_Etudiants | ID_Etudiant | N:1 | Simple |
| Fait_Resultats | Dim_Matieres | ID_Matiere | N:1 | Simple |
| Fait_Resultats | Dim_Calendrier | Date_Examen | N:1 | Simple |

### 4.2 Règles de Filtrage

**Propagation des Filtres**
- Dimensions → Faits : **Toujours activée** (One-way filtering)
- Faits → Dimensions : **Désactivée** (sauf cas spécifiques)
- Dim_Calendrier : **Filtre toutes les tables de faits**

**Cross-Filtering**
- Désactivé entre tables de faits (éviter ambiguïté)
- Activé entre dimensions connectées (drill-down)

---

## 5. Optimisations et Bonnes Pratiques

### 5.1 Clés de Substitution (Surrogate Keys)

✅ **Toutes les dimensions utilisent des clés entières séquentielles**
- Plus performantes que les clés naturelles
- Gèrent facilement les SCD Type 2
- Réduisent la taille du modèle

### 5.2 Colonnes Calculées vs Mesures

**Colonnes Calculées** (évaluées au chargement)
```dax
// Dans Dim_Etudiants
Nom_Complet = Etudiants[Nom] & " " & Etudiants[Prenom]
Age = DATEDIFF(Etudiants[Date_Naissance], TODAY(), YEAR)
```

**Mesures** (évaluées à la demande)
```dax
// Mesures dynamiques
Total_Etudiants = DISTINCTCOUNT(Etudiants[ID_Etudiant])
Taux_Reussite = DIVIDE([Nb_Admis], [Nb_Inscrits], 0)
```

### 5.3 Réduction de la Taille

**Techniques Appliquées**
1. ✅ Supprimer colonnes inutilisées à la source
2. ✅ Types de données optimaux (INT vs BIGINT)
3. ✅ Pas de doublons dans les dimensions
4. ✅ Index sur clés primaires/étrangères
5. ✅ Compression automatique Power BI (VertiPaq)

### 5.4 Performance des Requêtes

**Index Recommandés (Sources SQL)**
```sql
-- Table Inscriptions
CREATE INDEX IX_Inscriptions_Etudiant ON Inscriptions(ID_Etudiant)
CREATE INDEX IX_Inscriptions_Date ON Inscriptions(Date_Inscription)
CREATE INDEX IX_Inscriptions_Filiere ON Inscriptions(ID_Filiere)

-- Table Absences
CREATE INDEX IX_Absences_Etudiant ON Absences_Etudiants(ID_Etudiant)
CREATE INDEX IX_Absences_Date ON Absences_Etudiants(Date_Absence)

-- Table Paiements
CREATE INDEX IX_Paiements_Etudiant ON Paiements(ID_Etudiant)
CREATE INDEX IX_Paiements_Date ON Paiements(Date_Paiement)
```

---

## 6. Gestion de la Temporalité (SCD)

### 6.1 Slowly Changing Dimensions

**Dim_Etudiants : Type 1 (Overwrite)**
- Les modifications écrasent les anciennes valeurs
- Pas d'historique conservé
- Utilisé pour : adresse, téléphone, email

**Dim_Professeurs : Type 2 (History)**
- Conservation de l'historique
- Champs de gestion :
  - `Date_Debut_Validite`
  - `Date_Fin_Validite`
  - `Est_Actuel` (flag booléen)

**Exemple Professeur SCD Type 2**
```
ID | Nom    | Departement  | Statut    | Debut      | Fin        | Actuel
501| BENNANI| Informatique | Vacataire | 2015-09-01 | 2018-08-31 | 0
501| BENNANI| Informatique | Permanent | 2018-09-01 | 9999-12-31 | 1
```

### 6.2 Date Intelligence

**Comparaisons Temporelles**
```dax
// Year-over-Year
YoY_Etudiants = 
VAR CurrentYear = [Total_Etudiants]
VAR PreviousYear = CALCULATE([Total_Etudiants], SAMEPERIODLASTYEAR(Calendrier[Date]))
RETURN DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)

// Month-to-Date
MTD_Encaissements = TOTALMTD([Total_Encaisse], Calendrier[Date])

// Year-to-Date
YTD_Encaissements = TOTALYTD([Total_Encaisse], Calendrier[Date])
```

---

## 7. Règles de Nommage

### 7.1 Conventions

**Tables**
- Faits : `Fait_[Nom]` (ex: `Fait_Inscriptions`)
- Dimensions : `Dim_[Nom]` (ex: `Dim_Etudiants`)
- Bridge : `Bridge_[Nom]` (si nécessaire)

**Colonnes**
- Clés primaires : `ID_[Table]` (ex: `ID_Etudiant`)
- Clés étrangères : `ID_[TableReferencee]`
- Mesures : `[Nom]` sans préfixe de table
- Pas d'espaces ni caractères spéciaux

**Mesures DAX**
- Nom explicite : `Taux_Reussite`, `Total_Encaisse`
- Underscore pour séparation
- Pas d'abréviation cryptique

---

## 8. Validation et Tests

### 8.1 Checklist Qualité

✅ **Intégrité Référentielle**
- Toutes les FK ont une correspondance dans les dimensions
- Pas de valeurs NULL dans les FK
- Test : `LEFT JOIN` avec `IS NULL`

✅ **Unicité des Clés**
- Pas de doublons dans les PK des dimensions
- Test : `GROUP BY HAVING COUNT(*) > 1`

✅ **Cohérence Temporelle**
- Dates valides (pas de dates futures erronées)
- Période couverte cohérente
- Test : `MIN(Date)`, `MAX(Date)`

✅ **Cardinalités**
- Relations Many-to-One respectées
- Test : Vérifier ambiguïtés dans Power BI

### 8.2 Requêtes de Validation

```sql
-- Vérifier doublons étudiants
SELECT Code_Massar, COUNT(*) 
FROM Dim_Etudiants 
GROUP BY Code_Massar 
HAVING COUNT(*) > 1

-- Vérifier orphelins (inscriptions sans étudiant)
SELECT i.*
FROM Fait_Inscriptions i
LEFT JOIN Dim_Etudiants e ON i.ID_Etudiant = e.ID_Etudiant
WHERE e.ID_Etudiant IS NULL

-- Vérifier cohérence montants
SELECT 
    SUM(Montant_Paye) as Total_Paye,
    SUM(Montant_Attendu) as Total_Attendu,
    SUM(Solde_Restant) as Total_Solde
FROM Fait_Paiements
-- Total_Solde devrait égaler Total_Attendu - Total_Paye
```

---

## 9. Évolutions Futures

### 9.1 Améliorations Prévues (Phase 2)

1. **Dimension Géographique**
   - Ajout `Dim_Geographie` (Pays > Région > Ville)
   - Analyse spatiale des étudiants

2. **Fait Agrégé**
   - `Fait_Inscriptions_Mois` (snapshot mensuel)
   - Amélioration performances requêtes longue période

3. **Bridge Tables**
   - Gestion relations Many-to-Many si nécessaire
   - Ex: Étudiants avec plusieurs filières (double diplôme)

4. **Dimension Junk**
   - Regrouper attributs à faible cardinalité
   - Ex: Type_Inscription, Mode_Paiement, Statut

---

## 📅 Informations Document

**Version** : 1.0  
**Date de création** : 10 Décembre 2025  
**Dernière mise à jour** : 10 Décembre 2025  
**Auteur** : Équipe BI EMSI  
**Statut** : ✅ Validé  
**Tables** : 6 Faits + 6 Dimensions = 12 tables

---

*École Marocaine des Sciences de l'Ingénieur - Projet Business Intelligence*
