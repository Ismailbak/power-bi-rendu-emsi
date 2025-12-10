# Dashboard RH - Gestion des Ressources Humaines (Professeurs)

## 📊 Objectif
Fournir au service RH un outil de pilotage de la charge d'enseignement, suivi des absences professeurs, et gestion des heures complémentaires.

---

## 🎯 Utilisateurs Cibles
- **Directeur des Ressources Humaines**
- **Service Paie**
- **Responsables pédagogiques**
- **Administration RH**

---

## 📐 Layout du Dashboard

### Page 1 : Vue d'Ensemble RH

#### En-tête
- **Logo EMSI** + Titre : "Dashboard RH - Gestion du Corps Enseignant"
- **Filtres** : 
  - Département
  - Statut professeur (Permanent/Vacataire/Contractuel)
  - Année universitaire
  - Semestre

---

### Section 1 : KPIs Globaux (Cards)

```
┌───────────────┬───────────────┬───────────────┬───────────────┬───────────────┐
│Total Profs    │  Permanents   │  Vacataires   │ Heures Totales│ Coût Salarial │
│     185       │    65 (35%)   │   120 (65%)   │   18,540 h    │  14.2M MAD    │
│  ↗ +8 (4.5%) │  ↗ +2        │  ↗ +6        │  ↗ +1,250 h  │  ↗ +8.5%     │
└───────────────┴───────────────┴───────────────┴───────────────┴───────────────┘

┌───────────────┬───────────────┬───────────────┬───────────────┐
│Heures/Prof Moy│ Taux Absence  │Heures Complém.│  Vacances     │
│   18.2 h/sem  │     3.5%      │   2,840 h     │   42 profs    │
│  ↗ +0.5 h    │  ↘ -0.2%     │  ↗ +12%      │  En cours     │
└───────────────┴───────────────┴───────────────┴───────────────┘
```

**Mesures DAX :**
```dax
Total_Professeurs = DISTINCTCOUNT(Dim_Professeurs[ID_Professeur])

Nb_Permanents = 
CALCULATE(
    [Total_Professeurs],
    Dim_Professeurs[Statut] = "Permanent"
)

Nb_Vacataires = 
CALCULATE(
    [Total_Professeurs],
    Dim_Professeurs[Statut] = "Vacataire"
)

Total_Heures_Enseignement = SUM(Fait_Planning[Nb_Heures])

Cout_Salarial_Total = 
SUMX(
    Fait_Planning,
    Fait_Planning[Nb_Heures] * 
    RELATED(Dim_Professeurs[Taux_Horaire])
)

Heures_Moy_Par_Prof = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs],
    0
)

Taux_Absence_Profs = 
DIVIDE(
    COUNTROWS(Fait_Absences_Profs),
    [Total_Heures_Enseignement],
    0
)

Heures_Complementaires = 
SUMX(
    Fait_Planning,
    VAR HeuresNormales = 18 * 16 // 18h/semaine * 16 semaines
    VAR HeuresRealisees = 
        CALCULATE(
            SUM(Fait_Planning[Nb_Heures]),
            ALLEXCEPT(Dim_Professeurs, Dim_Professeurs[ID_Professeur])
        )
    RETURN
        MAX(HeuresRealisees - HeuresNormales, 0)
)

Nb_Profs_Vacances = 
CALCULATE(
    [Total_Professeurs],
    Fait_Absences_Profs[Type_Absence] = "Congé",
    MONTH(Fait_Absences_Profs[Date_Absence]) IN {7, 8}
)
```

---

### Section 2 : Répartition du Corps Enseignant (Visuals)

#### A. Par Statut (Donut Chart)
```
┌────────────────────────────┐
│ Permanents : 65 (35%)      │
│ Vacataires : 120 (65%)     │
│ Contractuels : 0 (0%)      │
└────────────────────────────┘
```

#### B. Par Département (Clustered Bar Chart)
```
Informatique    ████████████████████ 58 profs
Génie           ██████████████       42 profs
Management      ████████████         35 profs
Sciences        ██████████           28 profs
Langues         ████████             22 profs
```

#### C. Par Ancienneté (Stacked Column Chart)
```
< 2 ans     ████████  45 profs (24%)
2-5 ans     ██████████████  68 profs (37%)
5-10 ans    ████████  42 profs (23%)
> 10 ans    ██████  30 profs (16%)
```

---

### Section 3 : Charge d'Enseignement (Combo Chart)

**Graphique combiné** : Heures par département
- **Colonnes empilées** :
  - Heures CM (Cours Magistraux) - bleu foncé
  - Heures TD (Travaux Dirigés) - bleu clair
  - Heures TP (Travaux Pratiques) - vert
- **Ligne** : Charge moyenne par prof (axe secondaire)

**Mesures DAX :**
```dax
Heures_CM = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "CM"
)

Heures_TD = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "TD"
)

Heures_TP = 
CALCULATE(
    SUM(Fait_Planning[Nb_Heures]),
    Fait_Planning[Type_Cours] = "TP"
)

Charge_Moy_Dept = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs],
    0
)
```

---

### Section 4 : Matrice Charge par Professeur (Matrix)

**Matrice interactive** : Heures par prof et par matière

| Professeur | Dept | Statut | Matière 1 | Matière 2 | Matière 3 | Total | Charge % | Complém. |
|------------|------|--------|-----------|-----------|-----------|-------|----------|----------|
| ALAOUI Mohamed | INFO | Perm. | 120h | 80h | 60h | 260h | 90% | 0h |
| BENJELLOUN S. | INFO | Vac. | 180h | - | - | 180h | 62% | 0h |
| CHAKIR Fatima | GC | Perm. | 140h | 100h | 80h | 320h | 111% | 32h ⚠️ |
| DARIF Ali | MNG | Perm. | 160h | 90h | 50h | 300h | 104% | 12h |
| EL AMRANI L. | INFO | Vac. | 144h | - | - | 144h | 50% | 0h |

**Mise en forme conditionnelle** :
- 🟢 < 90% : Sous-charge
- 🟡 90-105% : Normal
- 🟠 105-120% : Surcharge modérée
- 🔴 > 120% : Surcharge critique

**Mesure DAX :**
```dax
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
        240  // Charge réduite pour vacataires
    )
RETURN
    DIVIDE(HeuresRealisees, HeuresNormales, 0)

Alerte_Charge = 
SWITCH(
    TRUE(),
    [Taux_Charge_Prof] > 1.20, "🔴 Surcharge critique",
    [Taux_Charge_Prof] > 1.05, "🟠 Surcharge modérée",
    [Taux_Charge_Prof] > 0.90, "🟢 Normal",
    "🟡 Sous-charge"
)

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

---

### Page 2 : Suivi des Absences Professeurs

### Section 1 : KPIs Absences (Cards)

```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│Total Absences   │  Non Justifiées │   Justifiées    │Heures Perdues   │
│      645        │      42 (6.5%)  │   603 (93.5%)   │    2,580 h      │
│  ↗ +12 (1.9%)  │  ↗ +5          │  ↗ +7          │  ↗ +48 h       │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

**Mesures DAX :**
```dax
Total_Absences_Profs = COUNTROWS(Fait_Absences_Profs)

Absences_Non_Just_Profs = 
CALCULATE(
    [Total_Absences_Profs],
    Fait_Absences_Profs[Justifiee] = "Non"
)

Absences_Just_Profs = 
CALCULATE(
    [Total_Absences_Profs],
    Fait_Absences_Profs[Justifiee] = "Oui"
)

Heures_Perdues = 
SUMX(
    Fait_Absences_Profs,
    Fait_Absences_Profs[Nb_Heures_Manquees]
)
```

---

### Section 2 : Absences par Type (Pie Chart)

**Graphique circulaire** : Motifs d'absence

```
┌────────────────────────────────────┐
│ Maladie : 385 (59.7%) 🏥          │
│ Congé : 180 (27.9%) ✈️            │
│ Formation : 38 (5.9%) 📚          │
│ Personnel : 32 (5.0%) 👤          │
│ Non justifié : 10 (1.5%) ❌       │
└────────────────────────────────────┘
```

---

### Section 3 : Top 10 Profs Absents (Table)

| Rang | Nom | Département | Statut | Total Abs. | Non Just. | Heures Perdues | Impact |
|------|-----|-------------|--------|------------|-----------|----------------|--------|
| 1 | CHAKIR Fatima | GC | Perm. | 28 | 3 | 112h | 🔴 Élevé |
| 2 | BENNANI Said | INFO | Vac. | 22 | 5 | 88h | 🔴 Élevé |
| 3 | IDRISSI Hassan | MNG | Perm. | 18 | 2 | 72h | 🟠 Moyen |
| 4 | KARIMI Youssef | GC | Vac. | 15 | 0 | 60h | 🟠 Moyen |
| 5 | TAHIRI Imane | INFO | Perm. | 14 | 1 | 56h | 🟡 Faible |

**Mesure DAX :**
```dax
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
```

---

### Section 4 : Évolution des Absences (Area Chart)

**Graphique en aires** : Tendance mensuelle

- **Axe X** : Mois (Sept 2024 → Juin 2025)
- **Aires** :
  - Justifiées (vert transparent)
  - Non justifiées (rouge transparent)

**Insights** :
- Pic d'absences : Janvier (grippe) et Mai (fin d'année)
- Creux : Octobre-Novembre (période stable)

---

### Section 5 : Taux d'Absence par Département (Bar Chart)

**Graphique en barres** : Comparaison départements

```
Génie Civil         ████████  4.8% 🔴
Management          ██████    3.9% 🟠
Informatique        █████     3.2% 🟢
Sciences            ████      2.8% 🟢
Langues             ████      2.6% 🟢
```

**Seuil objectif** : < 3.5% (ligne de référence)

---

### Page 3 : Analyse des Coûts et Paie

### Section 1 : KPIs Coûts (Cards)

```
┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Masse Salariale  │  Coût Permanent  │  Coût Vacataires │Heures Complém.   │
│   14.2M MAD      │   9.8M (69%)     │   4.4M (31%)     │   425K MAD       │
│   ↗ +8.5%       │   ↗ +6.2%       │   ↗ +14.5%      │   ↗ +18%        │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

**Mesures DAX :**
```dax
Masse_Salariale_Total = 
SUMX(
    Fait_Planning,
    Fait_Planning[Nb_Heures] * RELATED(Dim_Professeurs[Taux_Horaire])
)

Cout_Permanents = 
CALCULATE(
    [Masse_Salariale_Total],
    Dim_Professeurs[Statut] = "Permanent"
)

Cout_Vacataires = 
CALCULATE(
    [Masse_Salariale_Total],
    Dim_Professeurs[Statut] = "Vacataire"
)

Cout_Heures_Complementaires = 
SUMX(
    Fait_Planning,
    VAR HeuresCompl = [Heures_Complementaires_Prof]
    VAR TauxMajore = RELATED(Dim_Professeurs[Taux_Horaire]) * 1.25
    RETURN HeuresCompl * TauxMajore
)
```

---

### Section 2 : Coût par Département (Treemap)

**Treemap** : Visualisation hiérarchique des coûts

```
┌─────────────────────────────────────────────────────────┐
│ INFORMATIQUE (5.8M - 41%)                              │
│ ┌─────────────────────────────────────────────────────┐│
│ │ Permanents: 3.8M    │ Vacataires: 2.0M             ││
│ └─────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────┤
│ GÉNIE (4.2M - 30%)          │ MANAGEMENT (2.8M - 20%)  │
│ ┌──────────┬──────────────┐ │ ┌────────┬──────────┐  │
│ │ Perm 3.0M│ Vac 1.2M     │ │ │Perm 1.8M│Vac 1.0M │  │
│ └──────────┴──────────────┘ │ └────────┴──────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

### Section 3 : Coût Horaire Moyen (Bar Chart)

**Graphique en barres** : Comparaison taux horaires

```
Professeur Émérite      ██████████████████ 650 MAD/h
Professeur (> 10 ans)   ████████████████   550 MAD/h
Professeur (5-10 ans)   ██████████████     480 MAD/h
Professeur (< 5 ans)    ████████████       420 MAD/h
Vacataire Expert        ██████████         380 MAD/h
Vacataire               ████████           300 MAD/h
```

---

### Section 4 : Évolution Masse Salariale (Line Chart)

**Graphique en ligne** : Tendance mensuelle des coûts

- **Ligne principale** : Coût total par mois
- **Ligne pointillée** : Budget alloué
- **Zone grisée** : Écart budget

**Mesure DAX :**
```dax
Ecart_Budget_RH = 
VAR BudgetMensuel = 1200000 // 1.2M MAD/mois
VAR CoutReel = [Masse_Salariale_Total]
RETURN
    CoutReel - BudgetMensuel

Alerte_Budget = 
IF(
    [Ecart_Budget_RH] > 0,
    "🔴 Dépassement",
    "🟢 Dans budget"
)
```

---

### Section 5 : Tableau Récapitulatif Paie (Table)

| Professeur | Statut | Heures Normales | Heures Complém. | Taux | Coût Normal | Coût Complém. | Total | Notes |
|------------|--------|-----------------|-----------------|------|-------------|---------------|-------|-------|
| ALAOUI M. | Perm. | 288 | 0 | 550 | 158,400 | 0 | 158,400 | ✅ |
| CHAKIR F. | Perm. | 288 | 32 | 480 | 138,240 | 19,200 | 157,440 | ⚠️ Complém. |
| DARIF A. | Perm. | 288 | 12 | 550 | 158,400 | 8,250 | 166,650 | ⚠️ Complém. |
| BENJELLOUN S. | Vac. | 180 | 0 | 300 | 54,000 | 0 | 54,000 | ✅ |

**Slicers** :
- Département
- Statut
- Avec/sans heures complémentaires

---

## 🔄 Interactions et Fonctionnalités

### Drill-Through
- **Clic sur professeur** → Page "Fiche Prof Détaillée" avec :
  - Informations personnelles
  - Historique emploi
  - Planning détaillé
  - Absences complètes
  - Évolution salariale

### Boutons d'Action
- **"Exporter Paie"** : CSV pour service comptabilité
- **"Générer Planning"** : PDF planning professeurs
- **"Détecter Surcharges"** : Alerte automatique profs > 120%
- **"Simuler Recrutement"** : Impact coût nouveau prof

### Alertes RH
- 🔴 **Urgent** : Taux absence prof > 10%
- 🟠 **Attention** : Surcharge > 120%
- 🟢 **Info** : Heures complémentaires en hausse

---

## 📊 Mesures DAX Complémentaires

```dax
// Analyse de la charge
Taux_Occupation = 
DIVIDE(
    [Total_Heures_Enseignement],
    [Total_Professeurs] * 288, // Capacité théorique
    0
)

Nb_Profs_Surcharge = 
CALCULATE(
    [Total_Professeurs],
    [Taux_Charge_Prof] > 1.05
)

Nb_Profs_Sous_Charge = 
CALCULATE(
    [Total_Professeurs],
    [Taux_Charge_Prof] < 0.90
)

// Analyse des coûts
Cout_Moyen_Par_Prof = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Professeurs],
    0
)

Cout_Heure_Moyen = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Heures_Enseignement],
    0
)

Part_Heures_Complementaires = 
DIVIDE(
    [Heures_Complementaires],
    [Total_Heures_Enseignement],
    0
)

// Rentabilité
Cout_Par_Etudiant = 
DIVIDE(
    [Masse_Salariale_Total],
    [Total_Etudiants],
    0
)

Ratio_Etudiants_Prof = 
DIVIDE(
    [Total_Etudiants],
    [Total_Professeurs],
    0
)

// Ancienneté
Anciennete_Moyenne = 
AVERAGE(
    DATEDIFF(
        Dim_Professeurs[Date_Recrutement],
        TODAY(),
        YEAR
    )
)

Taux_Turnover = 
VAR Departs = COUNTROWS(FILTER(Dim_Professeurs, Dim_Professeurs[Statut] = "Parti"))
VAR EffectifMoyen = [Total_Professeurs]
RETURN
    DIVIDE(Departs, EffectifMoyen, 0)
```

---

## 🎨 Mise en Forme

### Codes Couleur Statut
- **Permanent** : Bleu foncé (#003366)
- **Vacataire** : Vert (#28A745)
- **Contractuel** : Orange (#FFC107)

### Codes Couleur Charge
- 🟢 < 90% : Sous-charge
- 🟡 90-105% : Normal
- 🟠 105-120% : Surcharge modérée
- 🔴 > 120% : Critique

---

## ✅ Checklist de Création

### Page 1 - Vue d'Ensemble
- [ ] Créer 9 cards KPIs globaux
- [ ] Ajouter 3 graphiques répartition (statut, département, ancienneté)
- [ ] Créer combo chart charge enseignement
- [ ] Insérer matrice charge par professeur
- [ ] Configurer mise en forme conditionnelle (surcharge)

### Page 2 - Absences
- [ ] Créer 4 cards KPIs absences
- [ ] Ajouter pie chart types d'absence
- [ ] Créer table Top 10 absents
- [ ] Insérer area chart évolution
- [ ] Ajouter bar chart taux par département

### Page 3 - Coûts
- [ ] Créer 4 cards KPIs coûts
- [ ] Ajouter treemap coûts départements
- [ ] Créer bar chart taux horaires
- [ ] Insérer line chart évolution masse salariale
- [ ] Créer table récapitulatif paie

### Configuration
- [ ] Créer page drill-through fiche prof
- [ ] Ajouter boutons export et simulation
- [ ] Configurer alertes automatiques
- [ ] Tester calculs paie
- [ ] Valider avec service RH

---

## 📝 Notes d'Utilisation

### Routine Mensuelle
- **Début de mois** : Valider planning professeurs
- **Mi-mois** : Vérifier absences et remplacements
- **Fin de mois** : Calculer heures complémentaires → Export paie

### Reporting
- **Hebdomadaire** : Absences + remplacements nécessaires
- **Mensuel** : Coûts réels vs budget + heures complémentaires
- **Trimestriel** : Analyse charge + besoins recrutement
- **Annuel** : Bilan RH complet + évolution masse salariale

### Sécurité RLS
```dax
// DRH : Accès total
[Role] = "DRH"

// Responsable pédagogique : Son département uniquement
[Departement] = LOOKUPVALUE(Users[Departement], Users[Email], USERPRINCIPALNAME())
```
