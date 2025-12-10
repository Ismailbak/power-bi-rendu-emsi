# 📊 Tables Dimensions et Faits - Structure Détaillée

## Vue d'Ensemble

Ce document détaille la structure complète de chaque table du modèle Power BI, incluant toutes les colonnes, types de données, contraintes, et exemples de données. Ces tables sont directement visibles et utilisables dans Power BI Desktop.

---

## 📅 1. Dim_Calendrier (Date Dimension)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `Date` | Date | dd/mm/yyyy | Clé primaire - Date complète | 15/01/2024 |
| 2 | `Annee` | Nombre Entier | #### | Année (2020-2028) | 2024 |
| 3 | `Mois` | Nombre Entier | ## | Numéro du mois (1-12) | 1 |
| 4 | `Mois_Nom` | Texte | - | Nom complet du mois | Janvier |
| 5 | `Mois_Court` | Texte | - | Abréviation 3 lettres | Jan |
| 6 | `Trimestre` | Texte | T# | Trimestre (T1-T4) | T1 |
| 7 | `Semestre` | Nombre Entier | # | Semestre (1-2) | 1 |
| 8 | `Semaine_Annee` | Nombre Entier | ## | Numéro semaine ISO (1-53) | 3 |
| 9 | `Jour_Mois` | Nombre Entier | ## | Jour du mois (1-31) | 15 |
| 10 | `Jour_Annee` | Nombre Entier | ### | Jour de l'année (1-366) | 15 |
| 11 | `Jour_Semaine` | Nombre Entier | # | Jour semaine (1=Lun, 7=Dim) | 1 |
| 12 | `Jour_Semaine_Nom` | Texte | - | Nom complet du jour | Lundi |
| 13 | `Jour_Semaine_Court` | Texte | - | Abréviation 3 lettres | Lun |
| 14 | `Est_Weekend` | Booléen | Oui/Non | Weekend (Sam-Dim) | Non |
| 15 | `Est_Jour_Ferie` | Booléen | Oui/Non | Jour férié marocain | Non |
| 16 | `Annee_Universitaire` | Texte | ####-#### | Année académique (Sep-Août) | 2023-2024 |
| 17 | `Semestre_Univ` | Texte | S# | Semestre universitaire | S1 |
| 18 | `Debut_Mois` | Date | dd/mm/yyyy | Premier jour du mois | 01/01/2024 |
| 19 | `Fin_Mois` | Date | dd/mm/yyyy | Dernier jour du mois | 31/01/2024 |

### Hiérarchies Créées dans Power BI

**Hiérarchie Calendrier**
```
Calendrier
  └─ Annee
      └─ Trimestre
          └─ Mois_Nom
              └─ Date
```

**Hiérarchie Universitaire**
```
Année Académique
  └─ Annee_Universitaire
      └─ Semestre_Univ
          └─ Mois_Nom
              └─ Date
```

### Exemples de Données (5 lignes)

| Date | Annee | Mois | Mois_Nom | Trimestre | Annee_Universitaire | Est_Weekend |
|------|-------|------|----------|-----------|---------------------|-------------|
| 15/01/2024 | 2024 | 1 | Janvier | T1 | 2023-2024 | Non |
| 16/01/2024 | 2024 | 1 | Janvier | T1 | 2023-2024 | Non |
| 20/01/2024 | 2024 | 1 | Janvier | T1 | 2023-2024 | Oui |
| 01/09/2024 | 2024 | 9 | Septembre | T3 | 2024-2025 | Oui |
| 15/09/2024 | 2024 | 9 | Septembre | T3 | 2024-2025 | Non |

### Configuration Power BI

```dax
// Marquer comme Table Date
Table.AddKey('Dim_Calendrier', "Date", true)

// Dans Options du modèle > Marquer la table comme table de dates
```

---

## 👨‍🎓 2. Dim_Etudiants (Dimension Étudiants)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `ID_Etudiant` | Nombre Entier | #### | Clé primaire - ID unique | 1001 |
| 2 | `Code_Massar` | Texte | - | Code national étudiant | M123456789 |
| 3 | `Nom` | Texte | - | Nom de famille (MAJUSCULES) | ALAMI |
| 4 | `Prenom` | Texte | - | Prénom (Première lettre maj) | Ahmed |
| 5 | `Nom_Complet` | Texte | - | Nom + Prénom | ALAMI Ahmed |
| 6 | `Date_Naissance` | Date | dd/mm/yyyy | Date de naissance | 15/05/2002 |
| 7 | `Age` | Nombre Entier | ## | Âge calculé | 22 |
| 8 | `Sexe` | Texte | - | M ou F | M |
| 9 | `Ville` | Texte | - | Ville de résidence | Casablanca |
| 10 | `Region` | Texte | - | Région administrative | Grand Casablanca |
| 11 | `Telephone` | Texte | - | Numéro téléphone (10 chiffres) | 0612345678 |
| 12 | `Email` | Texte | - | Email institutionnel | ahmed.a@emsi.ma |
| 13 | `Date_Premiere_Inscription` | Date | dd/mm/yyyy | Première inscription EMSI | 01/09/2020 |
| 14 | `Anciennete_Annees` | Nombre Entier | # | Années à l'EMSI | 4 |
| 15 | `Statut_Actuel` | Texte | - | Actif/Diplômé/Abandon | Actif |
| 16 | `Filiere_Actuelle` | Texte | - | Filière en cours | Génie Informatique |
| 17 | `Niveau_Actuel` | Texte | - | Niveau actuel | 4ème année |

### Hiérarchies Créées

**Hiérarchie Géographique**
```
Localisation
  └─ Region
      └─ Ville
```

**Hiérarchie Académique**
```
Parcours Étudiant
  └─ Statut_Actuel
      └─ Filiere_Actuelle
          └─ Niveau_Actuel
```

### Colonnes Calculées DAX

```dax
// Tranche d'Âge
Tranche_Age = 
SWITCH(
    TRUE(),
    'Dim_Etudiants'[Age] < 18, "Moins de 18 ans",
    'Dim_Etudiants'[Age] <= 22, "18-22 ans",
    'Dim_Etudiants'[Age] <= 25, "23-25 ans",
    "Plus de 25 ans"
)

// Initiales
Initiales = 
LEFT('Dim_Etudiants'[Prenom], 1) & ". " & 'Dim_Etudiants'[Nom]
```

### Exemples de Données (5 lignes)

| ID | Code_Massar | Nom_Complet | Age | Ville | Statut_Actuel | Filiere_Actuelle |
|----|-------------|-------------|-----|-------|---------------|------------------|
| 1001 | M123456789 | ALAMI Ahmed | 22 | Casablanca | Actif | Génie Informatique |
| 1002 | M123456790 | BENNANI Fatima | 21 | Rabat | Actif | Génie Civil |
| 1003 | M123456791 | TAZI Hassan | 23 | Fès | Actif | Génie Informatique |
| 1004 | M123456792 | IDRISSI Sara | 25 | Marrakech | Diplômé | Finance |
| 1005 | M123456793 | ALAOUI Mehdi | 20 | Casablanca | Actif | Génie Industriel |

---

## 🎓 3. Dim_Filieres (Dimension Filières)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `ID_Filiere` | Nombre Entier | ## | Clé primaire - ID unique | 1 |
| 2 | `Code_Filiere` | Texte | - | Code abrégé (2-4 lettres) | GI |
| 3 | `Nom_Filiere` | Texte | - | Nom complet filière | Génie Informatique |
| 4 | `Departement` | Texte | - | Département rattachement | Informatique |
| 5 | `Cycle` | Texte | - | Licence/Master/Ingénierie | Ingénierie |
| 6 | `Duree_Annees` | Nombre Entier | # | Durée programme (années) | 5 |
| 7 | `Capacite_Max` | Nombre Entier | ### | Places disponibles | 120 |
| 8 | `Tarif_Annuel` | Devise | # ##0 MAD | Frais annuels | 45 000 |
| 9 | `Date_Creation` | Date | dd/mm/yyyy | Date création filière | 01/09/2010 |
| 10 | `Statut_Filiere` | Texte | - | Active/Suspendue | Active |
| 11 | `Chef_Filiere` | Texte | - | Responsable filière | Pr. BENNANI |

### Hiérarchies Créées

**Hiérarchie Académique**
```
Organisation Académique
  └─ Departement
      └─ Cycle
          └─ Nom_Filiere
```

### Colonnes Calculées DAX

```dax
// Catégorie Tarif
Categorie_Tarif = 
SWITCH(
    TRUE(),
    'Dim_Filieres'[Tarif_Annuel] < 30000, "Économique",
    'Dim_Filieres'[Tarif_Annuel] <= 50000, "Standard",
    "Premium"
)

// Ancienneté Filière
Anciennete_Filiere = 
YEAR(TODAY()) - YEAR('Dim_Filieres'[Date_Creation])
```

### Exemples de Données (5 lignes)

| ID | Code | Nom_Filiere | Departement | Cycle | Tarif_Annuel | Capacite_Max |
|----|------|-------------|-------------|-------|--------------|--------------|
| 1 | GI | Génie Informatique | Informatique | Ingénierie | 45 000 | 120 |
| 2 | GC | Génie Civil | BTP | Ingénierie | 42 000 | 80 |
| 3 | GIN | Génie Industriel | Industrie | Ingénierie | 43 000 | 100 |
| 4 | FIN | Finance | Gestion | Master | 38 000 | 60 |
| 5 | MKT | Marketing | Gestion | Master | 36 000 | 50 |

---

## 📚 4. Dim_Niveaux (Dimension Niveaux)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `ID_Niveau` | Nombre Entier | # | Clé primaire | 1 |
| 2 | `Code_Niveau` | Texte | - | Code court | 1A |
| 3 | `Nom_Niveau` | Texte | - | Nom complet | 1ère année |
| 4 | `Ordre_Niveau` | Nombre Entier | # | Ordre séquentiel (1-5) | 1 |
| 5 | `Cycle` | Texte | - | Préparatoire/Cycle Ingénieur | Préparatoire |

### Exemples de Données (Toutes les lignes)

| ID | Code | Nom_Niveau | Ordre | Cycle |
|----|------|------------|-------|-------|
| 1 | 1A | 1ère année | 1 | Préparatoire |
| 2 | 2A | 2ème année | 2 | Préparatoire |
| 3 | 3A | 3ème année | 3 | Cycle Ingénieur |
| 4 | 4A | 4ème année | 4 | Cycle Ingénieur |
| 5 | 5A | 5ème année | 5 | Cycle Ingénieur |

---

## 📖 5. Dim_Matieres (Dimension Matières)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `ID_Matiere` | Nombre Entier | ### | Clé primaire | 101 |
| 2 | `Code_Matiere` | Texte | - | Code matière (alphanumérique) | INFO101 |
| 3 | `Nom_Matiere` | Texte | - | Nom complet | Programmation Java |
| 4 | `Categorie` | Texte | - | Domaine (Info, Math, Gestion...) | Informatique |
| 5 | `Coefficient` | Décimal | #.0 | Coefficient matière | 3.0 |
| 6 | `Volume_Horaire` | Nombre Entier | ## | Heures totales | 42 |
| 7 | `Type_Cours` | Texte | - | Cours/TD/TP | Cours |
| 8 | `Semestre` | Texte | S# | Semestre enseignement | S1 |

### Hiérarchies Créées

**Hiérarchie Matières**
```
Matières
  └─ Categorie
      └─ Type_Cours
          └─ Nom_Matiere
```

### Exemples de Données (5 lignes)

| ID | Code | Nom_Matiere | Categorie | Coeff | Volume_Horaire | Type |
|----|------|-------------|-----------|-------|----------------|------|
| 101 | INFO101 | Programmation Java | Informatique | 3.0 | 42 | Cours |
| 102 | INFO102 | Base de Données | Informatique | 3.5 | 48 | Cours |
| 103 | MATH101 | Algèbre Linéaire | Mathématiques | 2.5 | 36 | Cours |
| 104 | INFO103 | TP Java | Informatique | 2.0 | 24 | TP |
| 105 | MGT101 | Management | Gestion | 2.0 | 30 | Cours |

---

## 👨‍🏫 6. Dim_Professeurs (Dimension Professeurs)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Description | Exemple |
|---|-------------|--------------|--------|-------------|---------|
| 1 | `ID_Professeur` | Nombre Entier | ### | Clé primaire | 501 |
| 2 | `Code_Professeur` | Texte | - | Code interne | PROF501 |
| 3 | `Nom_Complet` | Texte | - | Nom complet avec titre | Dr. BENNANI Ahmed |
| 4 | `Specialite` | Texte | - | Domaine expertise | Réseaux Informatiques |
| 5 | `Departement` | Texte | - | Département affectation | Informatique |
| 6 | `Statut` | Texte | - | Permanent/Vacataire | Permanent |
| 7 | `Grade` | Texte | - | Grade académique | Professeur |
| 8 | `Date_Embauche` | Date | dd/mm/yyyy | Date d'entrée | 01/09/2015 |
| 9 | `Anciennete_Annees` | Nombre Entier | ## | Années ancienneté | 9 |
| 10 | `Email` | Texte | - | Email professionnel | a.bennani@emsi.ma |
| 11 | `Telephone` | Texte | - | Téléphone | 0612345678 |
| 12 | `Taux_Horaire` | Devise | ### MAD | Salaire horaire | 250 |
| 13 | `Date_Debut_Validite` | Date | dd/mm/yyyy | SCD Type 2 - Début validité | 01/09/2015 |
| 14 | `Date_Fin_Validite` | Date | dd/mm/yyyy | SCD Type 2 - Fin validité | 31/12/9999 |
| 15 | `Est_Actuel` | Booléen | Oui/Non | Version actuelle (SCD Type 2) | Oui |

### Hiérarchies Créées

**Hiérarchie RH**
```
Corps Enseignant
  └─ Departement
      └─ Statut
          └─ Grade
              └─ Nom_Complet
```

### Colonnes Calculées DAX

```dax
// Catégorie Ancienneté
Categorie_Anciennete = 
SWITCH(
    TRUE(),
    'Dim_Professeurs'[Anciennete_Annees] < 3, "Junior",
    'Dim_Professeurs'[Anciennete_Annees] <= 10, "Confirmé",
    "Senior"
)
```

### Exemples de Données (5 lignes)

| ID | Nom_Complet | Specialite | Departement | Statut | Taux_Horaire | Anciennete |
|----|-------------|------------|-------------|--------|--------------|------------|
| 501 | Dr. BENNANI Ahmed | Réseaux Informatiques | Informatique | Permanent | 250 | 9 |
| 502 | Pr. ALAMI Fatima | Intelligence Artificielle | Informatique | Permanent | 300 | 12 |
| 503 | M. TAZI Hassan | Bases de Données | Informatique | Vacataire | 180 | 3 |
| 504 | Dr. IDRISSI Sara | Finance d'Entreprise | Gestion | Permanent | 280 | 7 |
| 505 | M. ALAOUI Mehdi | Marketing Digital | Gestion | Vacataire | 150 | 2 |

---

## 📝 7. Fait_Inscriptions (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Inscription` | Nombre Entier | ##### | PK | Identifiant unique | 10001 |
| 2 | `ID_Etudiant` | Nombre Entier | #### | FK | Lien Dim_Etudiants | 1001 |
| 3 | `ID_Filiere` | Nombre Entier | ## | FK | Lien Dim_Filieres | 1 |
| 4 | `ID_Niveau` | Nombre Entier | # | FK | Lien Dim_Niveaux | 3 |
| 5 | `Date_Inscription` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 15/09/2024 |
| 6 | `Type_Inscription` | Texte | - | Attribut | Nouvelle/Réinscription | Réinscription |
| 7 | `Montant_Annuel` | Devise | ## ### MAD | Measure | Frais scolarité | 45 000 |
| 8 | `Statut_Inscription` | Texte | - | Attribut | En cours/Validé/Abandon | Validé |
| 9 | `Annee_Universitaire` | Texte | ####-#### | Attribut | Année académique | 2024-2025 |

### Relations Power BI

```
Dim_Etudiants[ID_Etudiant] 1 ──> * Fait_Inscriptions[ID_Etudiant]
Dim_Filieres[ID_Filiere] 1 ──> * Fait_Inscriptions[ID_Filiere]
Dim_Niveaux[ID_Niveau] 1 ──> * Fait_Inscriptions[ID_Niveau]
Dim_Calendrier[Date] 1 ──> * Fait_Inscriptions[Date_Inscription]
```

### Mesures DAX Associées

```dax
// Nombre Total Inscriptions
Total_Inscriptions = COUNTROWS('Fait_Inscriptions')

// Nouvelles Inscriptions
Nouvelles_Inscriptions = 
CALCULATE(
    [Total_Inscriptions],
    'Fait_Inscriptions'[Type_Inscription] = "Nouvelle"
)

// Revenus Inscriptions
Revenus_Inscriptions = SUM('Fait_Inscriptions'[Montant_Annuel])

// Taux Réinscription
Taux_Reinscription = 
DIVIDE(
    CALCULATE([Total_Inscriptions], 'Fait_Inscriptions'[Type_Inscription] = "Réinscription"),
    [Total_Inscriptions],
    0
) * 100
```

### Exemples de Données (5 lignes)

| ID_Inscription | ID_Etudiant | ID_Filiere | Date_Inscription | Type | Montant_Annuel | Statut |
|----------------|-------------|------------|------------------|------|----------------|--------|
| 10001 | 1001 | 1 | 15/09/2024 | Réinscription | 45 000 | Validé |
| 10002 | 1002 | 2 | 16/09/2024 | Nouvelle | 42 000 | En cours |
| 10003 | 1003 | 1 | 15/09/2024 | Réinscription | 45 000 | Validé |
| 10004 | 1005 | 3 | 17/09/2024 | Nouvelle | 43 000 | Validé |
| 10005 | 1006 | 4 | 18/09/2024 | Nouvelle | 38 000 | En cours |

---

## 🚫 8. Fait_Absences_Etudiants (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Absence` | Nombre Entier | ##### | PK | Identifiant unique | 20001 |
| 2 | `ID_Etudiant` | Nombre Entier | #### | FK | Lien Dim_Etudiants | 1001 |
| 3 | `ID_Matiere` | Nombre Entier | ### | FK | Lien Dim_Matieres | 101 |
| 4 | `Date_Absence` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 20/01/2024 |
| 5 | `Type_Absence` | Texte | - | Attribut | Justifiée/Non justifiée | Non justifiée |
| 6 | `Duree_Heures` | Nombre Entier | # | Measure | Nombre d'heures | 2 |
| 7 | `Justification` | Texte | - | Attribut | Raison (optionnel) | Maladie |

### Relations Power BI

```
Dim_Etudiants[ID_Etudiant] 1 ──> * Fait_Absences_Etudiants[ID_Etudiant]
Dim_Matieres[ID_Matiere] 1 ──> * Fait_Absences_Etudiants[ID_Matiere]
Dim_Calendrier[Date] 1 ──> * Fait_Absences_Etudiants[Date_Absence]
```

### Mesures DAX Associées

```dax
// Total Absences
Total_Absences = COUNTROWS('Fait_Absences_Etudiants')

// Total Heures Absences
Total_Heures_Absences = SUM('Fait_Absences_Etudiants'[Duree_Heures])

// Taux Absentéisme
Taux_Absenteisme = 
DIVIDE(
    [Total_Heures_Absences],
    [Total_Heures_Prevues],
    0
) * 100

// Absences Non Justifiées
Absences_Non_Justifiees = 
CALCULATE(
    [Total_Absences],
    'Fait_Absences_Etudiants'[Type_Absence] = "Non justifiée"
)
```

### Exemples de Données (5 lignes)

| ID_Absence | ID_Etudiant | ID_Matiere | Date_Absence | Type | Duree_Heures | Justification |
|------------|-------------|------------|--------------|------|--------------|---------------|
| 20001 | 1001 | 101 | 20/01/2024 | Justifiée | 2 | Maladie |
| 20002 | 1002 | 102 | 22/01/2024 | Non justifiée | 2 | - |
| 20003 | 1001 | 103 | 25/01/2024 | Non justifiée | 3 | - |
| 20004 | 1003 | 101 | 27/01/2024 | Justifiée | 2 | Problème familial |
| 20005 | 1005 | 104 | 28/01/2024 | Non justifiée | 1 | - |

---

## 💰 9. Fait_Paiements (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Paiement` | Nombre Entier | ##### | PK | Identifiant unique | 30001 |
| 2 | `ID_Etudiant` | Nombre Entier | #### | FK | Lien Dim_Etudiants | 1001 |
| 3 | `Date_Paiement` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 15/10/2024 |
| 4 | `Montant_Paye` | Devise | ## ### MAD | Measure | Montant encaissé | 15 000 |
| 5 | `Mode_Paiement` | Texte | - | Attribut | Espèces/Chèque/Virement | Virement |
| 6 | `Reference_Paiement` | Texte | - | Attribut | N° reçu/chèque | R2024-12345 |
| 7 | `Annee_Universitaire` | Texte | ####-#### | Attribut | Année concernée | 2024-2025 |
| 8 | `Trimestre` | Texte | T# | Attribut | Trimestre | T1 |
| 9 | `Montant_Attendu` | Devise | ## ### MAD | Measure | Montant facturé | 45 000 |
| 10 | `Solde_Restant` | Devise | ## ### MAD | Measure | Reste à payer | 30 000 |

### Relations Power BI

```
Dim_Etudiants[ID_Etudiant] 1 ──> * Fait_Paiements[ID_Etudiant]
Dim_Calendrier[Date] 1 ──> * Fait_Paiements[Date_Paiement]
```

### Mesures DAX Associées

```dax
// Total Encaissé
Total_Encaisse = SUM('Fait_Paiements'[Montant_Paye])

// Total Attendu
Total_Attendu = SUM('Fait_Paiements'[Montant_Attendu])

// Total Impayé
Total_Impaye = SUM('Fait_Paiements'[Solde_Restant])

// Taux Recouvrement
Taux_Recouvrement = 
DIVIDE([Total_Encaisse], [Total_Attendu], 0) * 100

// Paiements par Mode
Paiements_Virement = 
CALCULATE(
    [Total_Encaisse],
    'Fait_Paiements'[Mode_Paiement] = "Virement"
)
```

### Exemples de Données (5 lignes)

| ID_Paiement | ID_Etudiant | Date_Paiement | Montant_Paye | Mode | Montant_Attendu | Solde_Restant |
|-------------|-------------|---------------|--------------|------|-----------------|---------------|
| 30001 | 1001 | 15/10/2024 | 15 000 | Virement | 45 000 | 30 000 |
| 30002 | 1002 | 16/10/2024 | 42 000 | Chèque | 42 000 | 0 |
| 30003 | 1003 | 18/10/2024 | 20 000 | Virement | 45 000 | 25 000 |
| 30004 | 1005 | 20/10/2024 | 10 000 | Espèces | 43 000 | 33 000 |
| 30005 | 1006 | 22/10/2024 | 38 000 | Virement | 38 000 | 0 |

---

## 👨‍🏫 10. Fait_Planning_Enseignement (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Planning` | Nombre Entier | ##### | PK | Identifiant unique | 40001 |
| 2 | `ID_Professeur` | Nombre Entier | ### | FK | Lien Dim_Professeurs | 501 |
| 3 | `ID_Matiere` | Nombre Entier | ### | FK | Lien Dim_Matieres | 101 |
| 4 | `ID_Filiere` | Nombre Entier | ## | FK | Lien Dim_Filieres | 1 |
| 5 | `Date_Debut` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 01/09/2024 |
| 6 | `Annee_Universitaire` | Texte | ####-#### | Attribut | Année académique | 2024-2025 |
| 7 | `Semestre` | Texte | S# | Attribut | S1/S2 | S1 |
| 8 | `Heures_Prevues` | Nombre Entier | ## | Measure | Volume horaire planifié | 42 |
| 9 | `Heures_Realisees` | Nombre Entier | ## | Measure | Heures effectuées | 40 |
| 10 | `Taux_Horaire` | Devise | ### MAD | Measure | Coût par heure | 250 |
| 11 | `Jour_Semaine` | Texte | - | Attribut | Lundi/Mardi... | Lundi |

### Relations Power BI

```
Dim_Professeurs[ID_Professeur] 1 ──> * Fait_Planning_Enseignement[ID_Professeur]
Dim_Matieres[ID_Matiere] 1 ──> * Fait_Planning_Enseignement[ID_Matiere]
Dim_Filieres[ID_Filiere] 1 ──> * Fait_Planning_Enseignement[ID_Filiere]
Dim_Calendrier[Date] 1 ──> * Fait_Planning_Enseignement[Date_Debut]
```

### Mesures DAX Associées

```dax
// Total Heures Prévues
Total_Heures_Prevues = SUM('Fait_Planning_Enseignement'[Heures_Prevues])

// Total Heures Réalisées
Total_Heures_Realisees = SUM('Fait_Planning_Enseignement'[Heures_Realisees])

// Taux Réalisation
Taux_Realisation = 
DIVIDE([Total_Heures_Realisees], [Total_Heures_Prevues], 0) * 100

// Coût Total Enseignement
Cout_Total_Enseignement = 
SUMX(
    'Fait_Planning_Enseignement',
    'Fait_Planning_Enseignement'[Heures_Realisees] * 'Fait_Planning_Enseignement'[Taux_Horaire]
)
```

### Exemples de Données (5 lignes)

| ID_Planning | ID_Prof | ID_Matiere | ID_Filiere | Heures_Prevues | Heures_Realisees | Taux_Horaire |
|-------------|---------|------------|------------|----------------|------------------|--------------|
| 40001 | 501 | 101 | 1 | 42 | 40 | 250 |
| 40002 | 502 | 102 | 1 | 48 | 48 | 300 |
| 40003 | 503 | 103 | 2 | 36 | 34 | 180 |
| 40004 | 504 | 105 | 4 | 30 | 30 | 280 |
| 40005 | 505 | 106 | 4 | 24 | 22 | 150 |

---

## 🚫 11. Fait_Absences_Professeurs (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Absence_Prof` | Nombre Entier | ##### | PK | Identifiant unique | 50001 |
| 2 | `ID_Professeur` | Nombre Entier | ### | FK | Lien Dim_Professeurs | 501 |
| 3 | `Date_Absence` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 25/01/2024 |
| 4 | `Type_Absence` | Texte | - | Attribut | Maladie/Congé/Autre | Maladie |
| 5 | `Justifiee` | Booléen | Oui/Non | Attribut | 0/1 | Oui |
| 6 | `Cours_Annules` | Nombre Entier | # | Measure | Heures perdues | 3 |
| 7 | `Remplacement` | Booléen | Oui/Non | Attribut | Cours remplacé | Non |
| 8 | `ID_Remplacant` | Nombre Entier | ### | FK | Professeur remplaçant | NULL |

### Relations Power BI

```
Dim_Professeurs[ID_Professeur] 1 ──> * Fait_Absences_Professeurs[ID_Professeur]
Dim_Calendrier[Date] 1 ──> * Fait_Absences_Professeurs[Date_Absence]
```

### Mesures DAX Associées

```dax
// Total Absences Professeurs
Total_Absences_Profs = COUNTROWS('Fait_Absences_Professeurs')

// Total Heures Cours Annulés
Total_Heures_Annulees = SUM('Fait_Absences_Professeurs'[Cours_Annules])

// Taux Absence Professeurs
Taux_Absence_Profs = 
DIVIDE([Total_Heures_Annulees], [Total_Heures_Prevues], 0) * 100

// Taux Remplacement
Taux_Remplacement = 
DIVIDE(
    CALCULATE([Total_Absences_Profs], 'Fait_Absences_Professeurs'[Remplacement] = TRUE()),
    [Total_Absences_Profs],
    0
) * 100
```

### Exemples de Données (5 lignes)

| ID_Absence | ID_Prof | Date_Absence | Type | Justifiee | Cours_Annules | Remplacement |
|------------|---------|--------------|------|-----------|---------------|--------------|
| 50001 | 501 | 25/01/2024 | Maladie | Oui | 3 | Non |
| 50002 | 503 | 28/01/2024 | Congé | Oui | 2 | Oui |
| 50003 | 502 | 05/02/2024 | Autre | Non | 4 | Non |
| 50004 | 505 | 10/02/2024 | Maladie | Oui | 2 | Oui |
| 50005 | 504 | 15/02/2024 | Congé | Oui | 3 | Non |

---

## 📊 12. Fait_Resultats (Table de Faits)

### Structure de la Table

| # | Nom Colonne | Type Données | Format | Rôle | Description | Exemple |
|---|-------------|--------------|--------|------|-------------|---------|
| 1 | `ID_Resultat` | Nombre Entier | ##### | PK | Identifiant unique | 60001 |
| 2 | `ID_Etudiant` | Nombre Entier | #### | FK | Lien Dim_Etudiants | 1001 |
| 3 | `ID_Matiere` | Nombre Entier | ### | FK | Lien Dim_Matieres | 101 |
| 4 | `Date_Examen` | Date | dd/mm/yyyy | FK | Lien Dim_Calendrier | 20/01/2024 |
| 5 | `Note` | Décimal | ##.## | Measure | Note obtenue (/20) | 14.50 |
| 6 | `Statut` | Texte | - | Attribut | Admis/Ajourné/Absent | Admis |
| 7 | `Session` | Texte | - | Attribut | Normale/Rattrapage | Normale |
| 8 | `Annee_Universitaire` | Texte | ####-#### | Attribut | Année concernée | 2023-2024 |

### Relations Power BI

```
Dim_Etudiants[ID_Etudiant] 1 ──> * Fait_Resultats[ID_Etudiant]
Dim_Matieres[ID_Matiere] 1 ──> * Fait_Resultats[ID_Matiere]
Dim_Calendrier[Date] 1 ──> * Fait_Resultats[Date_Examen]
```

### Mesures DAX Associées

```dax
// Moyenne Générale
Moyenne_Generale = AVERAGE('Fait_Resultats'[Note])

// Nombre Admis
Nb_Admis = 
CALCULATE(
    COUNTROWS('Fait_Resultats'),
    'Fait_Resultats'[Statut] = "Admis"
)

// Taux Réussite
Taux_Reussite = 
DIVIDE([Nb_Admis], COUNTROWS('Fait_Resultats'), 0) * 100

// Note Maximale
Note_Max = MAX('Fait_Resultats'[Note])

// Note Minimale
Note_Min = MIN('Fait_Resultats'[Note])
```

### Exemples de Données (5 lignes)

| ID_Resultat | ID_Etudiant | ID_Matiere | Date_Examen | Note | Statut | Session |
|-------------|-------------|------------|-------------|------|--------|---------|
| 60001 | 1001 | 101 | 20/01/2024 | 14.50 | Admis | Normale |
| 60002 | 1002 | 101 | 20/01/2024 | 16.00 | Admis | Normale |
| 60003 | 1003 | 102 | 22/01/2024 | 8.00 | Ajourné | Normale |
| 60004 | 1003 | 102 | 15/06/2024 | 11.00 | Admis | Rattrapage |
| 60005 | 1005 | 103 | 25/01/2024 | 12.50 | Admis | Normale |

---

## 🔗 Configuration du Modèle dans Power BI

### Diagramme de Relations

```
[Dim_Calendrier] ──────────┬─────────────────┬──────────────┬─────────────┐
                           │                 │              │             │
                           ▼                 ▼              ▼             ▼
                    [Fait_Inscriptions] [Fait_Absences] [Fait_Paiements] [Fait_Planning]
                           │                 │
                           │                 │
[Dim_Etudiants] ───────────┼─────────────────┼──────────────┘
                           │                 │
[Dim_Filieres] ────────────┼─────────────────┘
                           │                 
[Dim_Niveaux] ─────────────┘                 
                                             │
[Dim_Matieres] ──────────────────────────────┴───────────────┐
                                                              │
[Dim_Professeurs] ────────────────────────────────────────────┴──> [Fait_Planning]
```

### Paramètres de Relations

**Toutes les relations :**
- **Cardinalité** : Many-to-One (N:1)
- **Direction du filtre** : Simple (Dimension → Fait)
- **Assume Referential Integrity** : Activé

### Propriétés du Modèle

```
// Marquer Dim_Calendrier comme Table Date
Modèle > Dim_Calendrier > Marquer comme table de dates

// Désactiver auto-date/time
Fichier > Options > Données actives > Time Intelligence > Désactiver

// Trier colonnes par d'autres colonnes
Mois_Nom trié par Mois
Jour_Semaine_Nom trié par Jour_Semaine
Trimestre trié par Mois
```

---

## 📈 Mesures Globales DAX (Table Mesures)

### Créer Table Dédiée Mesures

```dax
// Créer table vide pour organiser mesures
Mesures = DATATABLE("Dummy", STRING, {{"x"}})
```

### Mesures Essentielles

```dax
// ===== ÉTUDIANTS =====
Total_Etudiants = DISTINCTCOUNT(Dim_Etudiants[ID_Etudiant])

// ===== INSCRIPTIONS =====
Total_Inscriptions = COUNTROWS(Fait_Inscriptions)
Nouvelles_Inscriptions = CALCULATE([Total_Inscriptions], Fait_Inscriptions[Type_Inscription] = "Nouvelle")
Taux_Reinscription = DIVIDE(CALCULATE([Total_Inscriptions], Fait_Inscriptions[Type_Inscription] = "Réinscription"), [Total_Inscriptions], 0) * 100

// ===== ACADÉMIQUE =====
Taux_Reussite = DIVIDE(CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Statut] = "Admis"), COUNTROWS(Fait_Resultats), 0) * 100
Moyenne_Generale = AVERAGE(Fait_Resultats[Note])

// ===== ABSENCES ÉTUDIANTS =====
Total_Absences_Etudiants = COUNTROWS(Fait_Absences_Etudiants)
Taux_Absenteisme = DIVIDE(SUM(Fait_Absences_Etudiants[Duree_Heures]), [Total_Heures_Prevues], 0) * 100

// ===== FINANCIER =====
Total_Encaisse = SUM(Fait_Paiements[Montant_Paye])
Total_Attendu = SUM(Fait_Paiements[Montant_Attendu])
Total_Impaye = SUM(Fait_Paiements[Solde_Restant])
Taux_Recouvrement = DIVIDE([Total_Encaisse], [Total_Attendu], 0) * 100

// ===== RH =====
Total_Heures_Prevues = SUM(Fait_Planning_Enseignement[Heures_Prevues])
Total_Heures_Realisees = SUM(Fait_Planning_Enseignement[Heures_Realisees])
Taux_Realisation = DIVIDE([Total_Heures_Realisees], [Total_Heures_Prevues], 0) * 100
Cout_Total_Enseignement = SUMX(Fait_Planning_Enseignement, [Heures_Realisees] * [Taux_Horaire])

// ===== TIME INTELLIGENCE =====
YoY_Etudiants = 
VAR CurrentYear = [Total_Etudiants]
VAR PreviousYear = CALCULATE([Total_Etudiants], SAMEPERIODLASTYEAR(Dim_Calendrier[Date]))
RETURN DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0) * 100

YTD_Encaissements = TOTALYTD([Total_Encaisse], Dim_Calendrier[Date])
```

---

## 📅 Informations Document

**Version** : 1.0  
**Date de création** : 10 Décembre 2025  
**Dernière mise à jour** : 10 Décembre 2025  
**Auteur** : Équipe BI EMSI  
**Statut** : ✅ Validé  
**Tables** : 6 Dimensions + 6 Faits = 12 tables  
**Colonnes Totales** : 120+  
**Mesures DAX** : 50+

---

*École Marocaine des Sciences de l'Ingénieur - Projet Business Intelligence*
*Toutes les tables et mesures sont directement visibles et utilisables dans Power BI Desktop*
