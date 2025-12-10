# Bibliothèque Complète des Mesures DAX

## 📊 Table des Matières

1. [KPIs Académiques](#kpis-académiques)
2. [KPIs Financiers](#kpis-financiers)
3. [KPIs Scolarité](#kpis-scolarité)
4. [KPIs RH](#kpis-rh)
5. [Mesures Temporelles](#mesures-temporelles)
6. [Mesures d'Analyse](#mesures-danalyse)
7. [Mesures d'Alertes](#mesures-dalertes)
8. [Variables et Paramètres](#variables-et-paramètres)

---

## 📚 KPIs Académiques

### Effectifs et Inscriptions

```dax
// Nombre total d'étudiants actifs
Total_Etudiants = 
CALCULATE(
    DISTINCTCOUNT(Fait_Inscriptions[ID_Etudiant]),
    Fait_Inscriptions[Statut] <> "Abandon"
)

// Effectif par filière
Effectif_Filiere = 
CALCULATE(
    [Total_Etudiants],
    USERELATIONSHIP(Dim_Filieres[ID_Filiere], Fait_Inscriptions[ID_Filiere])
)

// Nouvelles inscriptions
Nouvelles_Inscriptions = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    Fait_Inscriptions[Type_Inscription] = "Nouvelle"
)

// Réinscriptions
Reinscriptions = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    Fait_Inscriptions[Type_Inscription] = "Réinscription"
)

// Transferts
Transferts = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    Fait_Inscriptions[Type_Inscription] = "Transfert"
)
```

### Résultats et Performance

```dax
// Taux de réussite global
Taux_Reussite = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Statut] = "Admis"),
    COUNTROWS(Fait_Resultats),
    0
)

// Taux de réussite par filière
Taux_Reussite_Filiere = 
VAR Admis = CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Statut] = "Admis")
VAR Total = COUNTROWS(Fait_Resultats)
RETURN
    DIVIDE(Admis, Total, 0)

// Moyenne générale
Moyenne_Generale = AVERAGE(Fait_Resultats[Note_Finale])

// Moyenne par matière
Moyenne_Par_Matiere = 
CALCULATE(
    AVERAGE(Fait_Resultats[Note_Finale]),
    ALLEXCEPT(Dim_Matieres, Dim_Matieres[Nom_Matiere])
)

// Taux de réussite par matière
Taux_Reussite_Par_Matiere = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Note_Finale] >= 10),
    COUNTROWS(Fait_Resultats),
    0
)

// Nombre d'admis
Nb_Admis = 
CALCULATE(
    COUNTROWS(Fait_Resultats),
    Fait_Resultats[Statut] = "Admis"
)

// Nombre de redoublants
Nb_Redoublants = 
CALCULATE(
    COUNTROWS(Fait_Resultats),
    Fait_Resultats[Statut] = "Redoublement"
)

// Taux d'abandon
Taux_Abandon = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Inscriptions), Fait_Inscriptions[Statut] = "Abandon"),
    COUNTROWS(Fait_Inscriptions),
    0
)
```

### Absences Étudiants

```dax
// Total absences étudiants
Total_Absences = COUNTROWS(Fait_Absences)

// Absences non justifiées
Absences_Non_Justifiees = 
CALCULATE(
    COUNTROWS(Fait_Absences),
    Fait_Absences[Justifiee] = "Non"
)

// Absences justifiées
Absences_Justifiees = 
CALCULATE(
    COUNTROWS(Fait_Absences),
    Fait_Absences[Justifiee] = "Oui"
)

// Taux d'absentéisme global
Taux_Absenteisme_Global = 
DIVIDE(
    [Total_Absences],
    CALCULATE(COUNTROWS(Fait_Planning) * [Total_Etudiants]),
    0
)

// Taux d'absentéisme par étudiant
Taux_Absence_Etudiant = 
VAR TotalSeances = 
    CALCULATE(
        COUNTROWS(Fait_Planning),
        ALLEXCEPT(Dim_Etudiants, Dim_Etudiants[ID_Etudiant])
    )
VAR AbsencesEtudiant = 
    CALCULATE(
        COUNTROWS(Fait_Absences),
        ALLEXCEPT(Dim_Etudiants, Dim_Etudiants[ID_Etudiant])
    )
RETURN
    DIVIDE(AbsencesEtudiant, TotalSeances, 0)

// Absences par étudiant (moyenne)
Absences_Par_Etudiant = 
DIVIDE(
    [Total_Absences],
    [Total_Etudiants],
    0
)

// Taux de justification
Taux_Justification = 
DIVIDE(
    [Absences_Justifiees],
    [Total_Absences],
    0
)
```

---

## 💰 KPIs Financiers

### Revenus et Recouvrement

```dax
// Total des revenus encaissés
Total_Revenus = SUM(Fait_Paiements[Montant_Paye])

// Montant total attendu
Montant_Attendu = SUM(Fait_Paiements[Montant_Attendu])

// Total des impayés
Total_Impayes = [Montant_Attendu] - [Total_Revenus]

// Taux de recouvrement
Taux_Recouvrement = 
DIVIDE(
    [Total_Revenus],
    [Montant_Attendu],
    0
)

// Taux de recouvrement par filière
Taux_Recouvrement_Filiere = 
VAR Revenus = [Total_Revenus]
VAR Attendu = [Montant_Attendu]
RETURN
    DIVIDE(Revenus, Attendu, 0)

// Revenus par filière
Revenus_Filiere = 
CALCULATE(
    [Total_Revenus],
    USERELATIONSHIP(Dim_Filieres[ID_Filiere], Fait_Inscriptions[ID_Filiere])
)

// Revenus par département
Revenus_Par_Departement = 
CALCULATE(
    [Total_Revenus],
    ALLEXCEPT(Dim_Filieres, Dim_Filieres[Departement])
)
```

### Analyse des Impayés

```dax
// Impayés 0-30 jours
Impayes_0_30j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard >= 0,
        JoursRetard <= 30
    )

// Impayés 30-90 jours
Impayes_30_90j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard > 30,
        JoursRetard <= 90
    )

// Impayés > 90 jours
Impayes_Plus_90j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard > 90
    )

// Délai moyen de paiement
Delai_Moyen_Paiement = 
AVERAGE(
    DATEDIFF(
        Fait_Paiements[Date_Echeance],
        Fait_Paiements[Date_Paiement],
        DAY
    )
)

// Nombre d'étudiants à jour
Nb_Etudiants_A_Jour = 
CALCULATE(
    DISTINCTCOUNT(Fait_Paiements[ID_Etudiant]),
    Fait_Paiements[Solde_Restant] = 0
)

// Nombre d'étudiants en retard > 90 jours
Nb_Retards_90j = 
CALCULATE(
    DISTINCTCOUNT(Fait_Paiements[ID_Etudiant]),
    DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY) > 90
)
```

### Provisions et Budget

```dax
// Provisions pour créances douteuses
Provisions_Creances_Douteuses = 
CALCULATE(
    [Impayes_0_30j] * 0.05
) +
CALCULATE(
    [Impayes_30_90j] * 0.15
) +
CALCULATE(
    [Impayes_Plus_90j] * 0.50
)

// Budget annuel
Budget_Annuel = 
CALCULATE(
    SUM(Budget[Montant]),
    Budget[Type] = "Revenus"
)

// Écart vs budget
Ecart_Budget = [Total_Revenus] - [Budget_Annuel]

// Écart en pourcentage
Ecart_Pct = DIVIDE([Ecart_Budget], [Budget_Annuel], 0)
```

---

## 📋 KPIs Scolarité

### Gestion des Inscriptions

```dax
// Total des inscriptions
Total_Inscriptions = COUNTROWS(Fait_Inscriptions)

// Inscriptions en attente
Inscriptions_En_Attente = 
CALCULATE(
    [Total_Inscriptions],
    Fait_Inscriptions[Statut_Inscription] = "En attente"
)

// Taux de validation des inscriptions
Taux_Validation_Inscriptions = 
DIVIDE(
    CALCULATE([Total_Inscriptions], Fait_Inscriptions[Statut_Inscription] = "Validée"),
    [Total_Inscriptions],
    0
)

// Délai moyen de validation
Delai_Moyen_Validation = 
AVERAGE(
    DATEDIFF(
        Fait_Inscriptions[Date_Demande],
        Fait_Inscriptions[Date_Validation],
        DAY
    )
)

// Jours d'attente
Jours_Attente = 
DATEDIFF(
    Fait_Inscriptions[Date_Demande],
    TODAY(),
    DAY
)
```

### Gestion des Documents

```dax
// Taux de dossiers complets
Taux_Dossiers_Complets = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fait_Inscriptions),
        Fait_Inscriptions[Dossier_Complet] = "Oui"
    ),
    [Total_Inscriptions],
    0
)

// Nombre de documents manquants
Nb_Documents_Manquants = 
CALCULATE(
    COUNTROWS(Fait_Documents),
    Fait_Documents[Statut] = "Manquant"
)

// Délai moyen de traitement des documents
Delai_Moyen_Traitement_Doc = 
AVERAGE(
    DATEDIFF(
        Fait_Documents[Date_Demande],
        Fait_Documents[Date_Delivrance],
        DAY
    )
)
```

---

## 👥 KPIs RH (Ressources Humaines)

### Effectifs Professeurs

```dax
// Total des professeurs
Total_Professeurs = DISTINCTCOUNT(Dim_Professeurs[ID_Professeur])

// Nombre de permanents
Nb_Permanents = 
CALCULATE(
    [Total_Professeurs],
    Dim_Professeurs[Statut] = "Permanent"
)

// Nombre de vacataires
Nb_Vacataires = 
CALCULATE(
    [Total_Professeurs],
    Dim_Professeurs[Statut] = "Vacataire"
)

// Ratio étudiants/professeur
Ratio_Etudiants_Prof = 
DIVIDE(
    [Total_Etudiants],
    [Total_Professeurs],
    0
)
```

### Charge d'Enseignement

```dax
// Total heures d'enseignement
Total_Heures_Enseignement = SUM(Fait_Planning[Nb_Heures])

// Heures CM (Cours Magistraux)
Heures_CM = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "CM"
)

// Heures TD (Travaux Dirigés)
Heures_TD = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "TD"
)

// Heures TP (Travaux Pratiques)
Heures_TP = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "TP"
)

// Heures moyennes par professeur
Heures_Moy_Par_Prof = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs],
    0
)

// Charge moyenne par département
Charge_Moy_Dept = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs],
    0
)

// Taux de charge professeur
Taux_Charge_Prof = 
VAR HeuresRealisees = 
    CALCULATE(
        SUM(Fait_Planning[Nb_Heures]),
        ALLEXCEPT(Dim_Professeurs, Dim_Professeurs[ID_Professeur])
    )
VAR HeuresNormales = 
    IF(
        SELECTEDVALUE(Dim_Professeurs[Statut]) = "Permanent",
        288, // 18h * 16 semaines
        240
    )
RETURN
    DIVIDE(HeuresRealisees, HeuresNormales, 0)

// Heures complémentaires
Heures_Complementaires = 
SUMX(
    Fait_Planning,
    VAR HeuresNormales = 288
    VAR HeuresRealisees = 
        CALCULATE(
            SUM(Fait_Planning[Nb_Heures]),
            ALLEXCEPT(Dim_Professeurs, Dim_Professeurs[ID_Professeur])
        )
    RETURN
        MAX(HeuresRealisees - HeuresNormales, 0)
)

// Heures complémentaires par professeur
Heures_Complementaires_Prof = 
VAR HeuresRealisees = 
    CALCULATE(
        SUM(Fait_Planning[Nb_Heures]),
        ALLEXCEPT(Dim_Professeurs, Dim_Professeurs[ID_Professeur])
    )
VAR HeuresNormales = 288
RETURN
    MAX(HeuresRealisees - HeuresNormales, 0)
```

### Absences Professeurs

```dax
// Total absences professeurs
Total_Absences_Profs = COUNTROWS(Fait_Absences_Profs)

// Absences non justifiées profs
Absences_Non_Just_Profs = 
CALCULATE(
    [Total_Absences_Profs],
    Fait_Absences_Profs[Justifiee] = "Non"
)

// Absences justifiées profs
Absences_Just_Profs = 
CALCULATE(
    [Total_Absences_Profs],
    Fait_Absences_Profs[Justifiee] = "Oui"
)

// Taux d'absence professeurs
Taux_Absence_Profs = 
DIVIDE(
    COUNTROWS(Fait_Absences_Profs),
    [Total_Heures_Enseignement],
    0
)

// Heures perdues (absences)
Heures_Perdues = 
SUMX(
    Fait_Absences_Profs,
    Fait_Absences_Profs[Nb_Heures_Manquees]
)
```

### Coûts et Masse Salariale

```dax
// Masse salariale totale
Masse_Salariale_Total = 
SUMX(
    Fait_Planning,
    Fait_Planning[Nb_Heures] * RELATED(Dim_Professeurs[Taux_Horaire])
)

// Coût permanents
Cout_Permanents = 
CALCULATE(
    [Masse_Salariale_Total],
    Dim_Professeurs[Statut] = "Permanent"
)

// Coût vacataires
Cout_Vacataires = 
CALCULATE(
    [Masse_Salariale_Total],
    Dim_Professeurs[Statut] = "Vacataire"
)

// Coût heures complémentaires
Cout_Heures_Complementaires = 
SUMX(
    Fait_Planning,
    VAR HeuresCompl = [Heures_Complementaires_Prof]
    VAR TauxMajore = RELATED(Dim_Professeurs[Taux_Horaire]) * 1.25
    RETURN HeuresCompl * TauxMajore
)

// Coût moyen par professeur
Cout_Moyen_Par_Prof = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Professeurs],
    0
)

// Coût par heure moyen
Cout_Heure_Moyen = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Heures_Enseignement],
    0
)

// Coût par étudiant
Cout_Par_Etudiant = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Etudiants],
    0
)
```

---

## 📅 Mesures Temporelles

### Comparaisons Annuelles

```dax
// Revenus année précédente
Revenus_N_1 = 
CALCULATE(
    [Total_Revenus],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

// Variation revenus YoY
Variation_Revenus_YoY = 
DIVIDE(
    [Total_Revenus] - [Revenus_N_1],
    [Revenus_N_1],
    0
)

// Étudiants année précédente
Etudiants_N_1 = 
CALCULATE(
    [Total_Etudiants],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

// Variation effectifs
Variation_Etudiants = 
DIVIDE(
    [Total_Etudiants] - [Etudiants_N_1],
    [Etudiants_N_1],
    0
)

// Moyenne N-1
Moyenne_N_1 = 
CALCULATE(
    [Moyenne_Generale],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

// Écart moyenne
Ecart_Moyenne = [Moyenne_Generale] - [Moyenne_N_1]
```

### Agrégations Temporelles

```dax
// Revenus Year-to-Date
Revenus_YTD = 
CALCULATE(
    [Total_Revenus],
    DATESYTD('Calendrier'[Date])
)

// Revenus mensuels
Revenus_Mensuels = 
CALCULATE(
    [Total_Revenus],
    DATESMTD('Calendrier'[Date])
)

// Revenus trimestriels
Revenus_Trimestriels = 
CALCULATE(
    [Total_Revenus],
    DATESQTD('Calendrier'[Date])
)

// Effectif par trimestre
Effectif_Trimestre = 
CALCULATE(
    [Total_Etudiants],
    DATESQTD('Calendrier'[Date])
)

// Moyenne par semestre
Moyenne_Par_Semestre = 
CALCULATE(
    [Moyenne_Generale],
    ALLEXCEPT('Calendrier', 'Calendrier'[Semestre])
)
```

---

## 📊 Mesures d'Analyse

### Segmentation et Parts

```dax
// Part de marché par filière
Part_Marche_Filiere = 
DIVIDE(
    [Total_Revenus],
    CALCULATE([Total_Revenus], ALL(Dim_Filieres)),
    0
)

// Taux d'occupation
Taux_Occupation = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs] * 288,
    0
)

// Part heures complémentaires
Part_Heures_Complementaires = 
DIVIDE(
    [Heures_Complementaires],
    [Total_Heures_Enseignement],
    0
)

// Pourcentage boursiers
Pct_Boursiers = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Inscriptions), Fait_Inscriptions[Type_Tarif] = "Boursier"),
    [Total_Etudiants],
    0
)
```

### Prévisions

```dax
// Prévision revenus
Prevision_Revenus = 
VAR MoyenneHistorique = 
    CALCULATE(
        [Total_Revenus], 
        DATESINPERIOD('Calendrier'[Date], MAX('Calendrier'[Date]), -3, MONTH)
    )
VAR TauxCroissance = 0.085
RETURN
    IF(
        MAX('Calendrier'[Date]) > TODAY(),
        MoyenneHistorique * (1 + TauxCroissance),
        [Total_Revenus]
    )

// Écart budget RH
Ecart_Budget_RH = 
VAR BudgetMensuel = 1200000
VAR CoutReel = [Masse_Salariale_Total]
RETURN
    CoutReel - BudgetMensuel
```

---

## ⚠️ Mesures d'Alertes

### Alertes Académiques

```dax
// Étudiants à risque
Nb_Etudiants_Risque = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    FILTER(
        Fait_Inscriptions,
        [Moyenne_Generale] < 10 || [Taux_Absence_Etudiant] > 0.2
    )
)

// Alerte étudiant
Alerte_Etudiant = 
VAR MoyenneEtudiant = [Moyenne_Generale]
VAR TauxAbsence = [Taux_Absence_Etudiant]
RETURN
    SWITCH(
        TRUE(),
        MoyenneEtudiant < 8 || TauxAbsence > 0.4, "🚨 Urgent",
        MoyenneEtudiant < 10 || TauxAbsence > 0.2, "⚠️ Suivi",
        "✅ OK"
    )

// Filières sous objectif
Alert_Filieres_Sous_Objectif = 
CALCULATE(
    COUNTROWS(Dim_Filieres),
    FILTER(
        Dim_Filieres,
        [Taux_Reussite] < 0.75
    )
)
```

### Alertes Financières

```dax
// Alerte impayés
Alert_Impayes = 
IF(
    [Total_Impayes] > 8000000,
    "⚠️ Impayés élevés",
    "✅ Sous contrôle"
)

// Priorité recouvrement
Priorite_Recouvrement = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
VAR MontantDu = Fait_Paiements[Solde_Restant]
VAR Score = (JoursRetard * 0.6) + (MontantDu / 1000 * 0.4)
RETURN
    SWITCH(
        TRUE(),
        Score > 100, "🔴 1 - Urgent",
        Score > 70, "🟠 2 - Prioritaire",
        Score > 40, "🟡 3 - Important",
        "🟢 4 - Suivi normal"
    )

// Risque créance
Risque_Creance = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    SWITCH(
        TRUE(),
        JoursRetard > 120, "Élevé",
        JoursRetard > 60, "Moyen",
        "Faible"
    )
```

### Alertes Scolarité

```dax
// Alerte délai inscription
Alerte_Delai = 
SWITCH(
    TRUE(),
    [Jours_Attente] > 15, "🔴 Urgent",
    [Jours_Attente] > 10, "🟠 Priorité",
    "🟢 Normal"
)

// Sanction recommandée (absence)
Sanction_Recommandee = 
SWITCH(
    TRUE(),
    [Taux_Absence_Etudiant] > 0.4, "⚠️ Avertissement 2",
    [Taux_Absence_Etudiant] > 0.3, "⚠️ Avertissement 1",
    [Taux_Absence_Etudiant] > 0.2, "📧 Rappel email",
    "✅ OK"
)
```

### Alertes RH

```dax
// Alerte charge professeur
Alerte_Charge = 
SWITCH(
    TRUE(),
    [Taux_Charge_Prof] > 1.20, "🔴 Surcharge critique",
    [Taux_Charge_Prof] > 1.05, "🟠 Surcharge modérée",
    [Taux_Charge_Prof] > 0.90, "🟢 Normal",
    "🟡 Sous-charge"
)

// Impact absence professeur
Impact_Absence = 
VAR HeuresPerdues = [Heures_Perdues]
VAR TauxAbsence = DIVIDE([Total_Absences_Profs], [Total_Heures_Enseignement], 0)
RETURN
    SWITCH(
        TRUE(),
        HeuresPerdues > 80 || TauxAbsence > 0.10, "🔴 Élevé",
        HeuresPerdues > 40 || TauxAbsence > 0.05, "🟠 Moyen",
        "🟡 Faible"
    )

// Alerte budget
Alerte_Budget = 
IF(
    [Ecart_Budget_RH] > 0,
    "🔴 Dépassement",
    "🟢 Dans budget"
)

// Nombre de professeurs en surcharge
Nb_Profs_Surcharge = 
CALCULATE(
    [Total_Professeurs],
    [Taux_Charge_Prof] > 1.05
)

// Nombre de professeurs en sous-charge
Nb_Profs_Sous_Charge = 
CALCULATE(
    [Total_Professeurs],
    [Taux_Charge_Prof] < 0.90
)
```

---

## 🔧 Variables et Paramètres

### Paramètres de Configuration

```dax
// Objectif taux de réussite
_Objectif_Taux_Reussite = 0.75

// Objectif taux de recouvrement
_Objectif_Taux_Recouvrement = 0.90

// Objectif taux d'absentéisme
_Objectif_Taux_Absenteisme = 0.10

// Heures normales annuelles (permanent)
_Heures_Normales_Permanent = 288

// Heures normales annuelles (vacataire)
_Heures_Normales_Vacataire = 240

// Budget mensuel RH
_Budget_Mensuel_RH = 1200000

// Taux de majoration heures complémentaires
_Taux_Majoration_HC = 1.25
```

### Tables de Paramètres

```dax
// Table paramètre : Analyse temporelle
Periode = {
    ("Année en cours", "YTD", 0),
    ("Mois en cours", "MTD", 1),
    ("Trimestre en cours", "QTD", 2),
    ("Semestre en cours", "STD", 3),
    ("Année précédente", "PY", 4)
}

// Mesure dynamique selon sélection
Analyse_Temporelle = 
SWITCH(
    SELECTEDVALUE(Periode[Code]),
    "YTD", [Revenus_YTD],
    "MTD", [Revenus_Mensuels],
    "QTD", [Revenus_Trimestriels],
    "PY", [Revenus_N_1],
    [Total_Revenus]
)
```

---

## 📝 Notes d'Utilisation

### Conventions de Nommage

- **Préfixe `Total_`** : Agrégation simple (SUM, COUNT)
- **Préfixe `Taux_`** : Ratio ou pourcentage
- **Préfixe `Nb_`** : Comptage (nombre de)
- **Préfixe `Cout_`** : Mesure financière
- **Préfixe `Alert_`** : Mesure d'alerte ou notification
- **Préfixe `_`** : Paramètre ou variable globale

### Performance

- Utiliser `VAR` pour stocker les calculs intermédiaires
- Éviter les colonnes calculées quand possible
- Privilégier les mesures pour les agrégations
- Utiliser `CALCULATE` avec parcimonie dans les boucles

### Maintenance

- Documenter chaque mesure complexe
- Grouper les mesures par thème dans des dossiers
- Tester les mesures sur des échantillons avant déploiement
- Valider les résultats avec les utilisateurs métier

---

## ✅ Checklist d'Implémentation

- [ ] Créer toutes les mesures dans Power BI Desktop
- [ ] Organiser les mesures en dossiers thématiques
- [ ] Tester chaque mesure avec des données réelles
- [ ] Valider les calculs avec les services métier
- [ ] Documenter les formules complexes
- [ ] Créer des infobulles explicatives
- [ ] Optimiser les mesures lentes
- [ ] Créer des tables de paramètres
- [ ] Définir les formats d'affichage
- [ ] Exporter la documentation des mesures
