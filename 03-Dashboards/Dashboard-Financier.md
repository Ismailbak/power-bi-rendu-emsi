# Dashboard Financier - Suivi des Revenus et Recouvrement

## 📊 Objectif
Fournir au service financier un outil de pilotage des revenus, suivi des paiements, recouvrement des créances et analyse budgétaire.

---

## 🎯 Utilisateurs Cibles
- **Directeur Financier**
- **Service Comptabilité**
- **Trésorerie**
- **Contrôleurs de gestion**

---

## 📐 Layout du Dashboard

### Page 1 : Vue d'Ensemble Financière

#### En-tête
- **Logo EMSI** + Titre : "Dashboard Financier - Suivi des Revenus"
- **Filtres** : 
  - Année universitaire
  - Filière
  - Mode de paiement
  - Statut (À jour / En retard / Impayé)

---

### Section 1 : KPIs Financiers Globaux (Cards)

```
┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│  Revenus Totaux  │ Montant Attendu  │Taux Recouvrement │    Impayés       │
│   115.2M MAD     │   124.5M MAD     │      92.5%       │   9.3M MAD       │
│   ↗ +8.5%       │   ↗ +10.2%      │   ↗ +1.2%       │   ↘ -5.8%       │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘

┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│  Délai Moyen     │Étudiants À Jour  │  Retards > 90j   │  Provisions      │
│    22 jours      │    2,105 (83%)   │      128 (5%)    │   2.8M MAD       │
│   ↗ +3 jours    │   ↘ -2%         │   ↗ +15         │   ↗ +12%        │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

**Mesures DAX :**
```dax
Total_Revenus = SUM(Fait_Paiements[Montant_Paye])

Montant_Attendu = SUM(Fait_Paiements[Montant_Attendu])

Taux_Recouvrement = 
DIVIDE(
    [Total_Revenus],
    [Montant_Attendu],
    0
)

Total_Impayes = [Montant_Attendu] - [Total_Revenus]

Delai_Moyen_Paiement = 
AVERAGE(
    DATEDIFF(
        Fait_Paiements[Date_Echeance],
        Fait_Paiements[Date_Paiement],
        DAY
    )
)

Nb_Etudiants_A_Jour = 
CALCULATE(
    DISTINCTCOUNT(Fait_Paiements[ID_Etudiant]),
    Fait_Paiements[Solde_Restant] = 0
)

Nb_Retards_90j = 
CALCULATE(
    DISTINCTCOUNT(Fait_Paiements[ID_Etudiant]),
    DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY) > 90
)

Provisions_Creances_Douteuses = 
CALCULATE(
    [Total_Impayes] * 0.3,
    [Nb_Retards_90j] > 0
)
```

---

### Section 2 : Évolution des Revenus (Combo Chart)

**Graphique combiné** : Flux financiers mensuels
- **Colonnes empilées** : 
  - Revenus encaissés (vert)
  - Revenus attendus non perçus (rouge transparent)
- **Ligne** : Taux de recouvrement (axe secondaire)
- **Axe X** : Mois (Sept 2024 → Juin 2025)

**Mesures DAX :**
```dax
Revenus_Mensuels = 
CALCULATE(
    [Total_Revenus],
    DATESMTD('Calendrier'[Date])
)

Attendu_Non_Percu = 
CALCULATE(
    [Montant_Attendu],
    DATESMTD('Calendrier'[Date])
) - [Revenus_Mensuels]

Taux_Recouvrement_Mensuel = 
DIVIDE(
    [Revenus_Mensuels],
    [Montant_Attendu],
    0
)
```

---

### Section 3 : Répartition par Mode de Paiement (Donut Chart)

**Graphique en anneau** : Préférence des modes de paiement

```
┌────────────────────────────────────┐
│  Virement : 68.5M (59.5%) 💳      │
│  Chèque : 32.1M (27.9%) 📝        │
│  Espèces : 10.8M (9.4%) 💵        │
│  Carte Bancaire : 3.8M (3.2%) 💳  │
└────────────────────────────────────┘
```

**Insight** : Promotion du virement bancaire (frais réduits)

---

### Section 4 : Revenus par Filière et Département (Treemap)

**Treemap** : Visualisation hiérarchique des revenus

```
┌─────────────────────────────────────────────────────────┐
│ INFORMATIQUE (38.25M - 33.2%)                          │
│ ┌─────────────────┬───────────────┬──────────────────┐ │
│ │ Génie Info      │ Réseaux & Tel │ Cyber Sécurité   │ │
│ │ 28.5M           │ 6.2M          │ 3.55M            │ │
│ └─────────────────┴───────────────┴──────────────────┘ │
├─────────────────────────────────────────────────────────┤
│ GENIE (28.0M - 24.3%)          │ MANAGEMENT (24.3M)   │
│ ┌──────────┬──────────────────┐ │ ┌────────┬────────┐│
│ │ GC       │ GIND             │ │ │ Finance│ MKT    ││
│ │ 17.2M    │ 10.8M            │ │ │ 17.1M  │ 7.2M   ││
│ └──────────┴──────────────────┘ │ └────────┴────────┘│
└─────────────────────────────────────────────────────────┘
```

**Couleurs** : Dégradé selon contribution aux revenus

---

### Section 5 : Analyse du Recouvrement (Waterfall Chart)

**Graphique en cascade** : Flux de recouvrement

```
Revenus Attendus (Sept)          124.5M
  - Paiements Virement             -68.5M
  - Paiements Chèque               -32.1M
  - Paiements Espèces              -10.8M
  - Paiements Carte                 -3.8M
  + Pénalités de retard             +0.5M
  - Annulations                     -0.6M
= Impayés Restants                  9.3M
```

---

### Page 2 : Gestion des Impayés et Créances

### Section 1 : KPIs Impayés (Cards)

```
┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Total Impayés    │  0-30 jours      │  30-90 jours     │  > 90 jours      │
│   9.3M MAD       │   4.2M (45%)     │   3.3M (35%)     │   1.8M (20%)     │
│   🔴 Critique    │   🟢 Gérable     │   🟠 Attention   │   🔴 Urgent      │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

**Mesures DAX :**
```dax
Impayes_0_30j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard >= 0 && JoursRetard <= 30
    )

Impayes_30_90j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard > 30 && JoursRetard <= 90
    )

Impayes_Plus_90j = 
VAR JoursRetard = DATEDIFF(Fait_Paiements[Date_Echeance], TODAY(), DAY)
RETURN
    CALCULATE(
        [Total_Impayes],
        JoursRetard > 90
    )
```

---

### Section 2 : Liste des Impayés (Table Détaillée)

**Tableau prioritaire** : Créances à recouvrer

| Priorité | ID | Nom Étudiant | Filière | Montant Dû | Jours Retard | Dernière Relance | Actions | Risque |
|----------|----|--------------|---------|-----------| -------------|------------------|---------|--------|
| 🔴 1 | 2156 | BENNANI L. | GC | 38,500 | 125 | 02/12/2024 | 📞 Appel + 📧 Mise en demeure | Élevé |
| 🔴 2 | 3421 | IDRISSI H. | GIND | 35,000 | 118 | 28/11/2024 | 📧 Relance 3 | Élevé |
| 🟠 3 | 1542 | KARIMI Y. | GI | 30,000 | 65 | 05/12/2024 | 📧 Relance 2 | Moyen |
| 🟠 4 | 4235 | TAHIRI I. | FIN | 28,750 | 58 | 01/12/2024 | 📧 Relance 1 | Moyen |
| 🟢 5 | 5120 | MOUSSA K. | MKT | 22,500 | 22 | 08/12/2024 | 📧 Rappel | Faible |

**Mesure DAX pour priorisation :**
```dax
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

**Actions Automatiques** :
- **Jours 0-15** : Email automatique de rappel
- **Jours 16-30** : Relance 1 + SMS
- **Jours 31-60** : Relance 2 + Appel téléphonique
- **Jours 61-90** : Relance 3 + Convocation
- **Jours 90+** : Mise en demeure + Suspension accès

---

### Section 3 : Évolution des Impayés (Area Chart)

**Graphique en aires** : Suivi des créances par ancienneté

- **Axe X** : Mois
- **Aires empilées** :
  - 0-30 jours (vert clair)
  - 30-60 jours (orange clair)
  - 60-90 jours (orange foncé)
  - > 90 jours (rouge)

**Objectif** : Réduire l'aire rouge (créances anciennes)

---

### Section 4 : Taux de Recouvrement par Filière (Bar Chart)

**Graphique en barres** : Performance recouvrement

```
Génie Informatique    ██████████████████████████ 94.2% ✅
Finance               ████████████████████████   93.1% ✅
Marketing             ███████████████████████    92.5% ✅
Génie Industriel      ██████████████████████     91.8% 🟠
Génie Civil           ████████████████████       89.7% 🟠
```

**Seuil objectif** : 90% (ligne de référence)

---

### Section 5 : Analyse par Type de Tarif (Clustered Column)

**Graphique en colonnes groupées** : Impayés par catégorie

| Type Tarif | Attendu | Encaissé | Impayés | Taux |
|------------|---------|----------|---------|------|
| Normal | 93.6M | 87.2M | 6.4M | 93.2% |
| Boursier (50%) | 22.3M | 20.8M | 1.5M | 93.3% |
| Boursier (100%) | 4.5M | 4.3M | 0.2M | 95.6% |
| Étranger | 4.1M | 2.9M | 1.2M | 70.7% 🔴 |

**Insight** : Étudiants étrangers = risque accru → Caution obligatoire

---

### Page 3 : Analyse Budgétaire et Prévisionnelle

### Section 1 : Budget vs Réalisé (Gauge Charts)

**Jauges circulaires** : Suivi des objectifs

```
┌───────────────┬───────────────┬───────────────┬───────────────┐
│   Revenus     │  Recouvrement │   Impayés     │   Délais      │
│               │               │               │               │
│  Objectif:    │  Objectif:    │  Objectif:    │  Objectif:    │
│  120.0M       │  95%          │  < 6.0M       │  < 15 jours   │
│               │               │               │               │
│  Réalisé:     │  Réalisé:     │  Réalisé:     │  Réalisé:     │
│  115.2M       │  92.5%        │  9.3M         │  22 jours     │
│  96.0% ✅     │  97.4% ✅     │  155% 🔴      │  147% 🔴      │
└───────────────┴───────────────┴───────────────┴───────────────┘
```

**Couleurs** :
- 🟢 Vert : Objectif atteint (≥ 95%)
- 🟠 Orange : Proche objectif (90-95%)
- 🔴 Rouge : Sous objectif (< 90%)

---

### Section 2 : Prévisions Trimestrielles (Line Chart avec Prévisions)

**Graphique en ligne** : Projection des revenus

- **Ligne pleine** : Revenus réels (Sept 2024 → Déc 2024)
- **Ligne pointillée** : Prévisions (Jan 2025 → Juin 2025)
- **Zone grisée** : Intervalle de confiance

**Mesure DAX :**
```dax
Prevision_Revenus = 
VAR MoyenneHistorique = CALCULATE([Total_Revenus], DATESINPERIOD('Calendrier'[Date], MAX('Calendrier'[Date]), -3, MONTH))
VAR TauxCroissance = 0.085 // +8.5% YoY
RETURN
    IF(
        MAX('Calendrier'[Date]) > TODAY(),
        MoyenneHistorique * (1 + TauxCroissance),
        [Total_Revenus]
    )
```

---

### Section 3 : Analyse des Écarts (Variance Analysis)

**Matrice** : Budget vs Réalisé par filière

| Filière | Budget | Réalisé | Écart | Écart % | Tendance |
|---------|--------|---------|-------|---------|----------|
| Génie Informatique | 40.0M | 38.25M | -1.75M | -4.4% | 🟠 |
| Génie Civil | 30.0M | 28.0M | -2.0M | -6.7% | 🔴 |
| Génie Industriel | 26.0M | 24.3M | -1.7M | -6.5% | 🔴 |
| Finance | 18.0M | 17.1M | -0.9M | -5.0% | 🟠 |
| Marketing | 6.0M | 7.6M | +1.6M | +26.7% | 🟢 |
| **TOTAL** | **120.0M** | **115.2M** | **-4.8M** | **-4.0%** | 🟠 |

**Mesure DAX :**
```dax
Budget_Annuel = 
CALCULATE(
    SUM(Budget[Montant]),
    Budget[Type] = "Revenus"
)

Ecart_Budget = [Total_Revenus] - [Budget_Annuel]

Ecart_Pct = DIVIDE([Ecart_Budget], [Budget_Annuel], 0)

Tendance = 
SWITCH(
    TRUE(),
    [Ecart_Pct] > 0, "🟢 Au-dessus",
    [Ecart_Pct] > -0.05, "🟠 Légèrement sous",
    "🔴 Sous objectif"
)
```

---

### Section 4 : Cash Flow Mensuel (Clustered Bar Chart)

**Graphique en barres** : Entrées vs Sorties

```
Septembre   ████████████ 12.5M  ████ -4.2M    = +8.3M
Octobre     ██████████████ 15.2M  █████ -5.8M  = +9.4M
Novembre    ███████████ 13.8M  ████ -4.5M    = +9.3M
Décembre    █████████ 11.2M  ███████ -7.1M  = +4.1M (Fin d'année)
```

---

## 🔄 Interactions et Fonctionnalités

### Drill-Through
- **Clic sur filière** → Détail paiements par étudiant
- **Clic sur impayé** → Historique complet de recouvrement
- **Clic sur mois** → Liste des transactions du mois

### Boutons d'Action
- **"Exporter Impayés"** : CSV pour équipe recouvrement
- **"Envoyer Relance Massive"** : Email automatique aux retardataires
- **"Générer Rapport Mensuel"** : PDF exécutif pour direction
- **"Simuler Scénarios"** : Outil de prévision interactive

### Alertes Automatiques
- 🔴 **Critique** : Taux recouvrement < 90%
- 🟠 **Attention** : Impayés > 90 jours en hausse
- 🟢 **Info** : Objectif mensuel atteint

---

## 📊 Mesures DAX Complémentaires

```dax
// Analyse temporelle
Revenus_N_1 = 
CALCULATE(
    [Total_Revenus],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

Variation_Revenus_YoY = 
DIVIDE(
    [Total_Revenus] - [Revenus_N_1],
    [Revenus_N_1],
    0
)

Revenus_YTD = 
CALCULATE(
    [Total_Revenus],
    DATESYTD('Calendrier'[Date])
)

// Segmentation
Revenus_Par_Departement = 
CALCULATE(
    [Total_Revenus],
    ALLEXCEPT(Dim_Filieres, Dim_Filieres[Departement])
)

Part_Marche_Filiere = 
DIVIDE(
    [Total_Revenus],
    CALCULATE([Total_Revenus], ALL(Dim_Filieres)),
    0
)

// Recouvrement avancé
Taux_Recouvrement_7j = 
DIVIDE(
    CALCULATE([Total_Revenus], Fait_Paiements[Jours_Retard] <= 7),
    [Montant_Attendu],
    0
)

Efficacite_Recouvrement = 
VAR RelancesEnvoyees = COUNTROWS(Fait_Relances)
VAR PaiementsObtenus = COUNTROWS(FILTER(Fait_Paiements, Fait_Paiements[Suite_Relance] = "Oui"))
RETURN
    DIVIDE(PaiementsObtenus, RelancesEnvoyees, 0)

// Provisions et risques
Provision_Globale = 
CALCULATE(
    [Total_Impayes] * 0.05,
    [Impayes_0_30j]
) +
CALCULATE(
    [Total_Impayes] * 0.15,
    [Impayes_30_90j]
) +
CALCULATE(
    [Total_Impayes] * 0.50,
    [Impayes_Plus_90j]
)
```

---

## 🎨 Mise en Forme

### Palette Financière
- **Positif/Encaissé** : Vert (#28A745)
- **Neutre/Attendu** : Bleu (#007BFF)
- **Négatif/Impayé** : Rouge (#DC3545)
- **Alerte** : Orange (#FFC107)

### Format des Montants
- **Milliers** : 1,5 K MAD
- **Millions** : 1,5 M MAD
- **Pourcentages** : 92,5 %
- **Jours** : 22 j

---

## ✅ Checklist de Création

### Page 1 - Vue d'Ensemble
- [ ] Créer 8 cards KPIs globaux
- [ ] Ajouter combo chart évolution revenus
- [ ] Créer donut chart modes de paiement
- [ ] Insérer treemap revenus par filière
- [ ] Ajouter waterfall chart recouvrement
- [ ] Configurer slicers (année, filière, mode)

### Page 2 - Impayés
- [ ] Créer 4 cards impayés par ancienneté
- [ ] Insérer table détaillée impayés avec actions
- [ ] Ajouter area chart évolution impayés
- [ ] Créer bar chart taux recouvrement/filière
- [ ] Insérer clustered column impayés/tarif
- [ ] Configurer système d'alertes automatiques

### Page 3 - Budget
- [ ] Créer 4 gauges budget vs réalisé
- [ ] Ajouter line chart avec prévisions
- [ ] Insérer matrice écarts budget
- [ ] Créer clustered bar cash flow
- [ ] Configurer drill-through détails
- [ ] Ajouter boutons export et relance

### Configuration Générale
- [ ] Définir KPIs cibles dans modèle
- [ ] Configurer mise en forme conditionnelle
- [ ] Tester scénarios de prévision
- [ ] Valider calculs avec comptabilité
- [ ] Former équipe financière

---

## 📝 Notes d'Utilisation

### Routine Quotidienne
- **9h** : Vérifier impayés > 90 jours → Actions urgentes
- **11h** : Traiter paiements reçus la veille
- **15h** : Envoyer relances automatiques
- **17h** : Mettre à jour prévisions

### Reporting
- **Hebdomadaire** : Taux recouvrement + Top 10 impayés
- **Mensuel** : Rapport complet direction + analyse écarts budget
- **Trimestriel** : Révision prévisions + stratégie recouvrement
- **Annuel** : Bilan financier complet + provisions

### Sécurité RLS
```dax
// Direction financière : Accès total
[Role] IN {"Directeur_Financier", "Controleur_Gestion"}

// Comptabilité : Lecture seule
[Role] = "Comptable" && [Modification] = FALSE
```
