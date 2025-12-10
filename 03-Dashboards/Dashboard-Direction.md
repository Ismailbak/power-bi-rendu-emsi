# Dashboard Direction - Vue Exécutive

## 📊 Objectif
Fournir à la direction une vue d'ensemble stratégique de l'établissement avec les indicateurs clés de performance (KPIs) globaux pour la prise de décision.

---

## 🎯 Utilisateurs Cibles
- **Direction Générale**
- **Directeurs de département**
- **Conseil d'administration**

---

## 📐 Layout du Dashboard

### Page 1 : Vue d'Ensemble

#### En-tête (Header)
- **Logo EMSI** (gauche)
- **Titre** : "EMSI - Dashboard Direction" (centre)
- **Filtres globaux** : Année universitaire, Département (droite)
- **Dernière actualisation** : Date et heure

---

### Section 1 : KPIs Principaux (Cards - Ligne du haut)

```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Total Étudiants│  Taux Réussite  │ Taux Recouvrement│ Revenus Totaux │
│      2,547      │      78.5%      │      92.3%      │   115.2M MAD   │
│   ↗ +12% YoY    │   ↗ +2.3%      │   ↘ -1.2%      │   ↗ +8.5%     │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

**Mesures DAX utilisées :**
- `Total_Etudiants`
- `Taux_Reussite`
- `Taux_Recouvrement`
- `Total_Revenus`

---

### Section 2 : Évolution Trimestrielle (Line Chart)

**Graphique en ligne** : Évolution des effectifs et revenus sur 3 ans
- **Axe X** : Trimestre/Année
- **Axe Y1** (gauche) : Nombre d'étudiants
- **Axe Y2** (droite) : Revenus (MAD)
- **Légende** : 2 lignes (Effectifs en bleu, Revenus en vert)

**Mesures DAX :**
```dax
Effectif_Trimestre = 
CALCULATE(
    [Total_Etudiants],
    DATESQTD('Calendrier'[Date])
)

Revenus_Trimestre = 
CALCULATE(
    [Total_Revenus],
    DATESQTD('Calendrier'[Date])
)
```

---

### Section 3 : Répartition par Filière (Clustered Bar Chart)

**Graphique en barres groupées** : Comparaison des filières
- **Axe Y** : Nom de la filière
- **Axe X** : Valeurs
- **Séries** : 
  - Nombre d'étudiants (bleu)
  - Revenus générés (vert)
  - Taux de réussite (orange, axe secondaire)

**Table source** : `Dim_Filieres`, `Fait_Inscriptions`, `Fait_Paiements`

---

### Section 4 : Performance Financière (Donut Chart)

**Graphique en anneau** : Répartition des revenus par département
- **Valeurs** : Revenus par département
- **Légende** : Noms des départements
- **Centre** : Total des revenus
- **Couleurs** : Palette cohérente avec la charte EMSI

**Mesure DAX :**
```dax
Revenus_Par_Departement = 
CALCULATE(
    [Total_Revenus],
    ALLEXCEPT(Dim_Filieres, Dim_Filieres[Departement])
)
```

---

### Section 5 : Tableau de Bord Synthétique (Matrix)

**Matrice** : KPIs détaillés par département

| Département | Étudiants | Taux Réussite | Revenus | Taux Recouvrement |
|-------------|-----------|---------------|---------|-------------------|
| Informatique | 850 | 82% | 38.2M | 94% |
| Génie Civil | 620 | 75% | 28.0M | 90% |
| Management | 540 | 80% | 24.3M | 91% |
| Finance | 380 | 77% | 17.1M | 93% |
| Marketing | 157 | 76% | 7.6M | 89% |
| **TOTAL** | **2,547** | **78.5%** | **115.2M** | **92.3%** |

**Mise en forme conditionnelle** :
- Taux de réussite : Vert si ≥ 75%, Orange si 65-75%, Rouge si < 65%
- Taux de recouvrement : Vert si ≥ 90%, Orange si 80-90%, Rouge si < 80%

---

### Section 6 : Alertes et Indicateurs (Scorecards)

```
⚠️ ALERTES
┌───────────────────────────────────────────────┐
│ • 3 filières sous l'objectif de réussite     │
│ • Impayés > 8M MAD (Dépassement de 15%)     │
│ • Taux d'abandon en hausse (+2.1%)           │
└───────────────────────────────────────────────┘

✅ RÉUSSITES
┌───────────────────────────────────────────────┐
│ • Effectif global : +12% vs année dernière    │
│ • Revenus : +8.5% (objectif atteint)         │
│ • Informatique : 94% de recouvrement         │
└───────────────────────────────────────────────┘
```

**Mesures DAX pour alertes :**
```dax
Alert_Filieres_Sous_Objectif = 
CALCULATE(
    COUNTROWS(Dim_Filieres),
    FILTER(
        Dim_Filieres,
        [Taux_Reussite] < 0.75
    )
)

Alert_Impayes = 
IF(
    [Total_Impayes] > 8000000,
    "⚠️ Impayés élevés",
    "✅ Sous contrôle"
)
```

---

## 🎨 Mise en Forme

### Palette de Couleurs (Charte EMSI)
- **Primaire** : #003366 (Bleu marine)
- **Secondaire** : #00A651 (Vert)
- **Accent** : #F39200 (Orange)
- **Neutre** : #6C757D (Gris)
- **Alerte** : #DC3545 (Rouge)

### Polices
- **Titres** : Segoe UI Bold, 14pt
- **Texte** : Segoe UI Regular, 10pt
- **KPIs** : Segoe UI Semibold, 18pt

### Espacement
- Marges : 15px entre sections
- Padding : 10px dans les visuels

---

## 🔄 Interactions

### Filtres Croisés
- Cliquer sur une filière → Filtre tous les visuels
- Cliquer sur un département → Filtre par département
- Cliquer sur un trimestre → Filtre temporel

### Drill-Through
- Clic droit sur une filière → Accès au **Dashboard Filière** détaillé
- Clic droit sur un KPI financier → Accès au **Dashboard Financier**

### Tooltips Personnalisés
Créer des infobulles avec détails supplémentaires :
- Effectif : Répartition hommes/femmes
- Revenus : Détail par type de paiement
- Taux de réussite : Comparaison N vs N-1

---

## 📊 Mesures DAX Requises

```dax
// KPI Étudiant
Total_Etudiants = DISTINCTCOUNT(Fait_Inscriptions[ID_Etudiant])

Etudiants_N_1 = 
CALCULATE(
    [Total_Etudiants],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

Variation_Etudiants = 
DIVIDE(
    [Total_Etudiants] - [Etudiants_N_1],
    [Etudiants_N_1],
    0
)

// KPI Réussite
Taux_Reussite = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Statut] = "Admis"),
    COUNTROWS(Fait_Resultats),
    0
)

// KPI Financier
Total_Revenus = SUM(Fait_Paiements[Montant_Paye])

Total_Attendu = SUM(Fait_Paiements[Montant_Attendu])

Taux_Recouvrement = 
DIVIDE(
    [Total_Revenus],
    [Total_Attendu],
    0
)

Total_Impayes = [Total_Attendu] - [Total_Revenus]

// KPI Abandons
Taux_Abandon = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Inscriptions), Fait_Inscriptions[Statut] = "Abandon"),
    [Total_Etudiants],
    0
)
```

---

## ✅ Checklist de Création

- [ ] Créer une nouvelle page "Direction"
- [ ] Ajouter l'en-tête avec logo et titre
- [ ] Insérer 4 cards pour les KPIs principaux
- [ ] Créer le graphique en ligne (évolution)
- [ ] Ajouter le graphique en barres (filières)
- [ ] Insérer le donut chart (départements)
- [ ] Créer la matrice synthétique
- [ ] Ajouter les zones d'alerte
- [ ] Appliquer la mise en forme (couleurs, polices)
- [ ] Configurer les interactions croisées
- [ ] Tester les drill-through
- [ ] Créer les tooltips personnalisés
- [ ] Valider avec un utilisateur de la direction

---

## 📝 Notes Techniques

### Performance
- Utiliser des mesures implicites pour les calculs simples
- Éviter les calculs en colonnes calculées si possible
- Agréger au niveau le plus haut (département) puis drill-down

### Actualisation
- Actualisation automatique : Quotidienne à 6h00
- Données historiques : 3 ans glissants
- Cache : Activé pour améliorer les performances

### Sécurité (RLS)
```dax
// Filtre pour la direction (accès total)
[Role] = "Direction"

// Filtre pour les directeurs de département (accès limité)
[Departement] = USERPRINCIPALNAME()
```
