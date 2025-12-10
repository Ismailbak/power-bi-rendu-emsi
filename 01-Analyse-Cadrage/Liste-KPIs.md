# 📊 Liste des KPIs - Indicateurs de Performance

## Vue d'Ensemble

Ce document recense tous les indicateurs clés de performance (KPIs) utilisés dans les dashboards Power BI de l'EMSI, organisés par domaine métier.

---

## 1. KPIs Académiques

### 1.1 Effectifs et Inscriptions

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Nombre Total Étudiants** | `Total_Etudiants = DISTINCTCOUNT(Etudiants[ID_Etudiant])` | Suivi | Carte |
| **Nouvelles Inscriptions** | `Nouvelles_Inscriptions = CALCULATE(DISTINCTCOUNT(Inscriptions[ID_Etudiant]), Inscriptions[Type_Inscription] = "Nouvelle")` | +10% YoY | Jauge |
| **Réinscriptions** | `Reinscriptions = CALCULATE(DISTINCTCOUNT(Inscriptions[ID_Etudiant]), Inscriptions[Type_Inscription] = "Réinscription")` | >85% | Jauge |
| **Taux de Réinscription** | `Taux_Reinscription = DIVIDE([Reinscriptions], [Total_Etudiants_Annee_Precedente], 0) * 100` | >85% | KPI |
| **Effectif par Filière** | `Effectif_Filiere = CALCULATE([Total_Etudiants], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | Suivi | Graphique barres |
| **Effectif par Niveau** | `Effectif_Niveau = CALCULATE([Total_Etudiants], ALLEXCEPT(Niveaux, Niveaux[Niveau]))` | Suivi | Graphique colonnes |
| **Évolution Inscriptions** | `Evolution_Inscriptions = DIVIDE([Total_Etudiants] - [Total_Etudiants_N-1], [Total_Etudiants_N-1], 0) * 100` | +5 à +15% | Graphique ligne |

**Mesure Support - Année N-1**
```dax
Total_Etudiants_N-1 = 
CALCULATE(
    [Total_Etudiants],
    DATEADD(Calendrier[Date], -1, YEAR)
)
```

### 1.2 Résultats et Réussite

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Taux de Réussite Global** | `Taux_Reussite = DIVIDE(COUNTROWS(FILTER(Resultats, Resultats[Statut] = "Admis")), COUNTROWS(Resultats), 0) * 100` | >75% | Jauge |
| **Taux de Réussite par Filière** | `Taux_Reussite_Filiere = CALCULATE([Taux_Reussite], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | >70% | Graphique barres |
| **Taux de Réussite par Matière** | `Taux_Reussite_Matiere = CALCULATE([Taux_Reussite], ALLEXCEPT(Matieres, Matieres[Nom_Matiere]))` | >65% | Tableau |
| **Nombre d'Admis** | `Nb_Admis = COUNTROWS(FILTER(Resultats, Resultats[Statut] = "Admis"))` | Maximiser | Carte |
| **Nombre de Redoublants** | `Nb_Redoublants = COUNTROWS(FILTER(Resultats, Resultats[Statut] = "Redoublant"))` | Minimiser | Carte |
| **Moyenne Générale** | `Moyenne_Generale = AVERAGE(Resultats[Note])` | >12/20 | KPI |
| **Top 10% Étudiants** | `Top_10_Etudiants = TOPN(CEILING([Total_Etudiants] * 0.1, 1), Etudiants, [Moyenne_Generale], DESC)` | Excellence | Tableau |

**Mesure Complexe - Taux Réussite avec Filtre Année**
```dax
Taux_Reussite_Annee = 
VAR AnneeSelectionnee = SELECTEDVALUE(Calendrier[Annee_Universitaire])
VAR TotalInscrits = 
    CALCULATE(
        COUNTROWS(Inscriptions),
        Calendrier[Annee_Universitaire] = AnneeSelectionnee
    )
VAR TotalAdmis = 
    CALCULATE(
        COUNTROWS(Resultats),
        Resultats[Statut] = "Admis",
        Calendrier[Annee_Universitaire] = AnneeSelectionnee
    )
RETURN
    DIVIDE(TotalAdmis, TotalInscrits, 0) * 100
```

### 1.3 Assiduité et Absences

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Taux d'Absentéisme Étudiant** | `Taux_Absenteisme = DIVIDE(SUM(Absences_Etudiants[Duree_Heures]), [Total_Heures_Prevues], 0) * 100` | <10% | Jauge |
| **Nombre Total Absences** | `Total_Absences = COUNTROWS(Absences_Etudiants)` | Minimiser | Carte |
| **Absences Non Justifiées** | `Absences_Non_Justifiees = CALCULATE([Total_Absences], Absences_Etudiants[Type] = "Non justifiée")` | <5% | Carte |
| **Taux Absences par Filière** | `Taux_Absences_Filiere = CALCULATE([Taux_Absenteisme], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | <10% | Graphique barres |
| **Étudiants à Risque (>20% abs)** | `Etudiants_Risque = CALCULATE(DISTINCTCOUNT(Absences_Etudiants[ID_Etudiant]), [Taux_Absenteisme] > 20)` | 0 | Alerte |
| **Moyenne Heures Absence** | `Moy_Heures_Absence = AVERAGE(Absences_Etudiants[Duree_Heures])` | <5h/mois | KPI |

**Mesure Support - Total Heures Prévues**
```dax
Total_Heures_Prevues = 
SUMX(
    Planning_Enseignement,
    Planning_Enseignement[Heures_Prevues]
)
```

### 1.4 Abandons et Rétention

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Taux d'Abandon** | `Taux_Abandon = DIVIDE(COUNTROWS(FILTER(Etudiants, Etudiants[Statut] = "Abandon")), [Total_Etudiants], 0) * 100` | <5% | Jauge |
| **Nombre d'Abandons** | `Nb_Abandons = COUNTROWS(FILTER(Etudiants, Etudiants[Statut] = "Abandon"))` | Minimiser | Carte |
| **Abandons par Niveau** | `Abandons_Niveau = CALCULATE([Nb_Abandons], ALLEXCEPT(Niveaux, Niveaux[Niveau]))` | Identifier | Graphique barres |
| **Taux de Rétention** | `Taux_Retention = (1 - [Taux_Abandon]) * 100` | >95% | KPI |
| **Période Moyenne Abandon** | `Moy_Periode_Abandon = AVERAGE(DATEDIFF(Etudiants[Date_Premiere_Inscription], Etudiants[Date_Abandon], MONTH))` | Analyse | Carte |

---

## 2. KPIs Financiers

### 2.1 Revenus et Encaissements

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Montant Total Encaissé** | `Total_Encaisse = SUM(Paiements[Montant])` | Budget | Carte |
| **Montant Total Attendu** | `Total_Attendu = SUM(Factures[Montant_Total])` | Référence | Carte |
| **Montant Total Impayé** | `Total_Impaye = SUM(Factures[Solde])` | Minimiser | Carte |
| **Revenus par Filière** | `Revenus_Filiere = CALCULATE([Total_Encaisse], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | Analyse | Graphique secteurs |
| **Revenus Mensuels** | `Revenus_Mois = CALCULATE([Total_Encaisse], ALLEXCEPT(Calendrier, Calendrier[Annee], Calendrier[Mois]))` | Suivi | Graphique ligne |
| **CA Prévisionnel vs Réalisé** | `Ecart_Budget = [Total_Encaisse] - [Budget_Previsionnel]` | Objectif atteint | Graphique colonnes |

**Mesure Complexe - Revenus YTD**
```dax
Revenus_YTD = 
CALCULATE(
    [Total_Encaisse],
    DATESYTD(Calendrier[Date])
)
```

### 2.2 Recouvrement

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Taux de Recouvrement** | `Taux_Recouvrement = DIVIDE([Total_Encaisse], [Total_Attendu], 0) * 100` | >90% | Jauge |
| **Taux Recouvrement par Filière** | `Taux_Recouv_Filiere = CALCULATE([Taux_Recouvrement], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | >85% | Graphique barres |
| **Délai Moyen Paiement** | `Delai_Moy_Paiement = AVERAGE(DATEDIFF(Factures[Date_Echeance], Paiements[Date_Paiement], DAY))` | <30 jours | KPI |
| **Nombre Étudiants Débiteurs** | `Nb_Debiteurs = CALCULATE(DISTINCTCOUNT(Factures[Code_Etudiant]), Factures[Statut] = "Impayé")` | Minimiser | Carte |
| **Top 10 Débiteurs** | `Top_10_Debiteurs = TOPN(10, VALUES(Etudiants[ID_Etudiant]), [Total_Impaye], DESC)` | Action | Tableau |
| **Montant Impayé >90 jours** | `Impaye_90j = CALCULATE([Total_Impaye], DATEDIFF(Factures[Date_Echeance], TODAY(), DAY) > 90)` | Urgence | Alerte |

**Mesure Complexe - Recouvrement Trimestriel**
```dax
Taux_Recouv_Trimestre = 
VAR TrimestreActuel = SELECTEDVALUE(Calendrier[Trimestre])
VAR EncaisseTrimestre = 
    CALCULATE(
        [Total_Encaisse],
        Calendrier[Trimestre] = TrimestreActuel
    )
VAR AttenduTrimestre = 
    CALCULATE(
        [Total_Attendu],
        Calendrier[Trimestre] = TrimestreActuel
    )
RETURN
    DIVIDE(EncaisseTrimestre, AttenduTrimestre, 0) * 100
```

### 2.3 Modes de Paiement

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Paiements Espèces** | `Paiements_Especes = CALCULATE([Total_Encaisse], Paiements[Mode_Paiement] = "Espèces")` | Analyse | Graphique secteurs |
| **Paiements Chèque** | `Paiements_Cheque = CALCULATE([Total_Encaisse], Paiements[Mode_Paiement] = "Chèque")` | Analyse | Graphique secteurs |
| **Paiements Virement** | `Paiements_Virement = CALCULATE([Total_Encaisse], Paiements[Mode_Paiement] = "Virement")` | Favoriser | Graphique secteurs |
| **% Paiements Dématérialisés** | `Pct_Dematerialise = DIVIDE([Paiements_Virement], [Total_Encaisse], 0) * 100` | >60% | Jauge |

---

## 3. KPIs Ressources Humaines

### 3.1 Charge d'Enseignement

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Total Heures Prévues** | `Total_Heures_Prevues = SUM(Planning_Enseignement[Heures_Prevues])` | Planning | Carte |
| **Total Heures Réalisées** | `Total_Heures_Realisees = SUM(Planning_Enseignement[Heures_Realisees])` | Suivi | Carte |
| **Taux de Réalisation** | `Taux_Realisation = DIVIDE([Total_Heures_Realisees], [Total_Heures_Prevues], 0) * 100` | 100% | Jauge |
| **Heures par Professeur** | `Heures_Prof = CALCULATE([Total_Heures_Realisees], ALLEXCEPT(Professeurs, Professeurs[Nom_Complet]))` | 18h/sem | Graphique barres |
| **Charge Moyenne Professeur** | `Charge_Moy_Prof = DIVIDE([Total_Heures_Prevues], DISTINCTCOUNT(Professeurs[ID_Professeur]), 0)` | 18h/sem | KPI |
| **Professeurs Surchargés (>24h)** | `Profs_Surcharges = CALCULATE(DISTINCTCOUNT(Professeurs[ID_Professeur]), [Heures_Prof] > 24)` | 0 | Alerte |
| **Heures par Département** | `Heures_Departement = CALCULATE([Total_Heures_Realisees], ALLEXCEPT(Professeurs, Professeurs[Departement]))` | Répartition | Graphique colonnes |

**Mesure Complexe - Charge Hebdomadaire**
```dax
Charge_Hebdo = 
VAR SemaineActuelle = SELECTEDVALUE(Calendrier[Semaine])
VAR HeuresSemaine = 
    CALCULATE(
        [Total_Heures_Realisees],
        Calendrier[Semaine] = SemaineActuelle
    )
RETURN
    HeuresSemaine
```

### 3.2 Absences Professeurs

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Taux Absence Professeurs** | `Taux_Absence_Profs = DIVIDE(SUM(Absences_Professeurs[Cours_Annules]), [Total_Heures_Prevues], 0) * 100` | <5% | Jauge |
| **Nombre Absences Professeurs** | `Nb_Absences_Profs = COUNTROWS(Absences_Professeurs)` | Minimiser | Carte |
| **Heures Cours Annulés** | `Heures_Annulees = SUM(Absences_Professeurs[Cours_Annules])` | Minimiser | Carte |
| **Taux Remplacement** | `Taux_Remplacement = DIVIDE(COUNTROWS(FILTER(Absences_Professeurs, Absences_Professeurs[Remplacement] = TRUE())), [Nb_Absences_Profs], 0) * 100` | 100% | Jauge |
| **Absences par Département** | `Absences_Dept = CALCULATE([Nb_Absences_Profs], ALLEXCEPT(Professeurs, Professeurs[Departement]))` | Analyse | Graphique barres |

### 3.3 Coûts et Masse Salariale

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Coût Total Enseignement** | `Cout_Total = SUMX(Planning_Enseignement, Planning_Enseignement[Heures_Realisees] * RELATED(Professeurs[Taux_Horaire]))` | Budget RH | Carte |
| **Coût par Filière** | `Cout_Filiere = CALCULATE([Cout_Total], ALLEXCEPT(Filieres, Filieres[Nom_Filiere]))` | Analyse | Graphique secteurs |
| **Coût Moyen par Étudiant** | `Cout_Moy_Etudiant = DIVIDE([Cout_Total], [Total_Etudiants], 0)` | Optimiser | KPI |
| **Nombre Professeurs Permanents** | `Nb_Permanents = CALCULATE(DISTINCTCOUNT(Professeurs[ID_Professeur]), Professeurs[Statut] = "Permanent")` | Stabilité | Carte |
| **Nombre Vacataires** | `Nb_Vacataires = CALCULATE(DISTINCTCOUNT(Professeurs[ID_Professeur]), Professeurs[Statut] = "Vacataire")` | Flexibilité | Carte |
| **Ratio Permanent/Vacataire** | `Ratio_Perm_Vac = DIVIDE([Nb_Permanents], [Nb_Vacataires], 0)` | 1.5 | KPI |

---

## 4. KPIs Transverses

### 4.1 Indicateurs Stratégiques Direction

| KPI | Formule DAX | Objectif | Visualisation |
|-----|-------------|----------|---------------|
| **Score Performance Global** | `Score_Global = ([Taux_Reussite] * 0.3) + ([Taux_Recouvrement] * 0.3) + ([Taux_Retention] * 0.2) + ([Taux_Realisation] * 0.2)` | >85 | Jauge |
| **Rentabilité par Étudiant** | `Rentabilite_Etudiant = [Total_Encaisse] / [Total_Etudiants] - [Cout_Moy_Etudiant]` | Positif | KPI |
| **Taux Satisfaction Global** | `Satisfaction = (Basé sur enquêtes - à implémenter)` | >4/5 | Étoiles |
| **Nb Filières Performantes** | `Filieres_Performantes = CALCULATE(DISTINCTCOUNT(Filieres[ID_Filiere]), [Taux_Reussite] > 75)` | Toutes | Carte |

### 4.2 Comparaisons et Benchmarks

| KPI | Formule DAX | Description | Visualisation |
|-----|-------------|-------------|---------------|
| **Évolution YoY** | `YoY = DIVIDE([Mesure_Actuelle] - [Mesure_Annee_Precedente], [Mesure_Annee_Precedente], 0) * 100` | Croissance | Flèche |
| **Rang Filière (Réussite)** | `Rang_Filiere = RANKX(ALL(Filieres[Nom_Filiere]), [Taux_Reussite], , DESC, DENSE)` | Classement | Badge |
| **Écart vs Objectif** | `Ecart_Objectif = [Valeur_Reelle] - [Valeur_Objectif]` | Performance | Barre |
| **Variance Budget** | `Variance = DIVIDE([Total_Encaisse] - [Budget_Previsionnel], [Budget_Previsionnel], 0) * 100` | ±5% | Graphique |

---

## 5. Mesures Utilitaires

### 5.1 Filtres et Contexte

```dax
// Année Universitaire Sélectionnée
Annee_Selectionnee = SELECTEDVALUE(Calendrier[Annee_Universitaire])

// Filière Sélectionnée
Filiere_Selectionnee = SELECTEDVALUE(Filieres[Nom_Filiere])

// Vérifier si filtre appliqué
A_Filtre = NOT(ISFILTERED(Filieres[Nom_Filiere]))

// Nombre d'éléments sélectionnés
Nb_Filieres_Selectionnees = COUNTROWS(VALUES(Filieres[Nom_Filiere]))
```

### 5.2 Formatage Conditionnel

```dax
// Couleur selon seuil
Couleur_Taux_Reussite = 
SWITCH(
    TRUE(),
    [Taux_Reussite] >= 75, "Vert",
    [Taux_Reussite] >= 60, "Orange",
    "Rouge"
)

// Icône Tendance
Icone_Tendance = 
IF(
    [YoY] > 0,
    "▲ " & FORMAT([YoY], "0.0%"),
    "▼ " & FORMAT([YoY], "0.0%")
)
```

### 5.3 Texte Dynamique

```dax
// Titre dynamique
Titre_Dashboard = 
"Dashboard " & [Filiere_Selectionnee] & 
" - Année " & [Annee_Selectionnee]

// Message Alerte
Message_Alerte = 
IF(
    [Taux_Recouvrement] < 80,
    "⚠️ ALERTE : Taux de recouvrement critique !",
    "✅ Objectif atteint"
)
```

---

## 6. Seuils et Objectifs par Indicateur

### 6.1 Code Couleur Universel

| Plage | Couleur | Signification |
|-------|---------|---------------|
| ≥ Objectif | 🟢 Vert | Excellent |
| 90-99% Objectif | 🟡 Jaune | Acceptable |
| 80-89% Objectif | 🟠 Orange | Vigilance |
| < 80% Objectif | 🔴 Rouge | Critique |

### 6.2 Tableau Récapitulatif Seuils

| KPI | 🟢 Excellent | 🟡 Acceptable | 🟠 Vigilance | 🔴 Critique |
|-----|-------------|---------------|--------------|-------------|
| Taux Réussite | ≥75% | 70-74% | 65-69% | <65% |
| Taux Recouvrement | ≥90% | 85-89% | 80-84% | <80% |
| Taux Absentéisme | ≤5% | 6-10% | 11-15% | >15% |
| Taux Abandon | ≤5% | 6-8% | 9-12% | >12% |
| Taux Réalisation | ≥95% | 90-94% | 85-89% | <85% |

---

## 7. Priorisation des KPIs

### 7.1 KPIs Critiques (Surveillance Quotidienne)

1. ✅ Taux de Recouvrement
2. ✅ Montant Impayé >90 jours
3. ✅ Nombre Étudiants à Risque
4. ✅ Heures Cours Annulés

### 7.2 KPIs Importants (Surveillance Hebdomadaire)

1. Taux d'Absentéisme Étudiant
2. Nouvelles Inscriptions
3. Charge Professeurs
4. Délai Moyen Paiement

### 7.3 KPIs Stratégiques (Surveillance Mensuelle)

1. Taux de Réussite Global
2. Score Performance Global
3. Évolution Effectifs YoY
4. Rentabilité par Étudiant

---

## 8. Alertes Automatiques

### 8.1 Règles d'Alerte

| Condition | Seuil | Action | Destinataire |
|-----------|-------|--------|--------------|
| Taux Recouvrement < 80% | Critique | Email immédiat | Comptabilité + Direction |
| Étudiants à Risque > 50 | Élevé | Alerte dashboard | Scolarité |
| Taux Absence Prof > 10% | Moyen | Notification | RH + Filière |
| Impayés >90j > 100k MAD | Critique | SMS + Email | Direction Financière |
| Taux Abandon > 8% | Élevé | Rapport hebdo | Direction + Filières |

---

## 📅 Informations Document

**Version** : 1.0  
**Date de création** : 10 Décembre 2025  
**Dernière mise à jour** : 10 Décembre 2025  
**Auteur** : Équipe BI EMSI  
**Statut** : ✅ Validé  
**Total KPIs** : 65+

---

*École Marocaine des Sciences de l'Ingénieur - Projet Business Intelligence*
