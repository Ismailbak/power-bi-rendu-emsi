# Dashboard Filière - Vue Académique Détaillée

## 📊 Objectif
Fournir une analyse approfondie des performances académiques par filière pour le suivi pédagogique et l'optimisation des programmes.

---

## 🎯 Utilisateurs Cibles
- **Directeurs de filière**
- **Coordinateurs pédagogiques**
- **Responsables académiques**

---

## 📐 Layout du Dashboard

### Page 1 : Performance Académique

#### En-tête
- **Logo EMSI** + Titre : "Dashboard Filière - Performance Académique"
- **Filtres** : 
  - Filière (dropdown multi-sélection)
  - Année universitaire
  - Niveau (L1, L2, L3, M1, M2)
  - Semestre

---

### Section 1 : KPIs par Filière (Cards)

```
┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│   Effectif   │ Taux Réussite│    Moyenne   │  Absentéisme │   Abandons   │
│     850      │    82.3%     │    13.2/20   │     8.5%     │      3.2%    │
│  ↗ +5.2%    │  ↗ +1.8%    │  ↗ +0.3     │  ↘ -0.8%    │  ↘ -0.5%   │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘
```

**Mesures DAX :**
```dax
Effectif_Filiere = 
CALCULATE(
    DISTINCTCOUNT(Fait_Inscriptions[ID_Etudiant]),
    Fait_Inscriptions[Statut] <> "Abandon"
)

Taux_Reussite_Filiere = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Statut] = "Admis"),
    COUNTROWS(Fait_Resultats),
    0
)

Moyenne_Generale = 
AVERAGE(Fait_Resultats[Note_Finale])

Taux_Absenteisme = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Absences), Fait_Absences[Justifiee] = "Non"),
    CALCULATE(COUNTROWS(Fait_Planning) * [Effectif_Filiere]),
    0
)

Taux_Abandon = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Inscriptions), Fait_Inscriptions[Statut] = "Abandon"),
    COUNTROWS(Fait_Inscriptions),
    0
)
```

---

### Section 2 : Répartition des Étudiants (Visuals Multi-angles)

#### A. Par Niveau (Funnel Chart)
```
L1 (Licence 1)    ███████████████████  320 étudiants
L2 (Licence 2)    ████████████████     280 étudiants  (-12.5%)
L3 (Licence 3)    █████████████        250 étudiants  (-10.7%)
M1 (Master 1)     ██████               120 étudiants
M2 (Master 2)     ████                  80 étudiants  (-33.3%)
```

**Analyse de la déperdition** : Identifier où les étudiants abandonnent

#### B. Par Genre (Donut Chart)
- Hommes : 520 (61%)
- Femmes : 330 (39%)

#### C. Par Type d'Inscription (Clustered Column)
- Nouvelle inscription : 320
- Réinscription : 480
- Transfert : 50

---

### Section 3 : Résultats Académiques (Combo Chart)

**Graphique combiné** : Évolution des résultats par semestre
- **Axe X** : Semestre (S1 2023 → S2 2025)
- **Colonnes** : Nombre d'admis vs redoublants
- **Ligne** : Moyenne générale (axe secondaire)

**Mesures DAX :**
```dax
Nb_Admis = 
CALCULATE(
    COUNTROWS(Fait_Resultats),
    Fait_Resultats[Statut] = "Admis"
)

Nb_Redoublants = 
CALCULATE(
    COUNTROWS(Fait_Resultats),
    Fait_Resultats[Statut] = "Redoublement"
)

Moyenne_Par_Semestre = 
CALCULATE(
    [Moyenne_Generale],
    ALLEXCEPT('Calendrier', 'Calendrier'[Semestre])
)
```

---

### Section 4 : Matrice Matières par Niveau (Matrix)

**Matrice interactive** : Performance par matière et niveau

| Matière | L1 Moy. | L1 Taux | L2 Moy. | L2 Taux | L3 Moy. | L3 Taux | M1 Moy. | M1 Taux |
|---------|---------|---------|---------|---------|---------|---------|---------|---------|
| Programmation | 12.8 | 75% | 13.5 | 80% | 14.2 | 85% | 15.1 | 90% |
| Base de données | 11.5 | 68% | 12.8 | 75% | 13.9 | 82% | 14.5 | 88% |
| Réseaux | 13.2 | 78% | 13.8 | 82% | 14.5 | 87% | 15.3 | 92% |
| Mathématiques | 10.9 | 65% | 11.7 | 70% | 12.5 | 76% | 13.8 | 85% |

**Mise en forme conditionnelle** :
- 🟢 Vert : Moyenne ≥ 14 ou Taux ≥ 85%
- 🟠 Orange : Moyenne 12-14 ou Taux 70-85%
- 🔴 Rouge : Moyenne < 12 ou Taux < 70%

---

### Section 5 : Top & Bottom Performers (Tables)

#### Top 10 Étudiants
| Rang | Nom Étudiant | Niveau | Moyenne | Absences |
|------|--------------|--------|---------|----------|
| 1 | ALAMI Sara | L3 | 17.85 | 0 |
| 2 | BENJELLOUN Ali | M1 | 17.42 | 1 |
| 3 | CHAHDI Fatima | L2 | 17.18 | 0 |
| ... | ... | ... | ... | ... |

#### Étudiants à Risque (Moyenne < 10 ou Absences > 20%)
| Nom Étudiant | Niveau | Moyenne | Absences | Actions |
|--------------|--------|---------|----------|---------|
| IDRISSI Omar | L1 | 8.5 | 35% | ⚠️ Tutorat |
| KARIMI Leila | L2 | 9.2 | 25% | ⚠️ Suivi |
| MANSOURI Amine | L1 | 7.8 | 42% | 🚨 Urgent |

**Mesure DAX pour alertes :**
```dax
Alerte_Etudiant = 
VAR MoyenneEtudiant = [Moyenne_Generale]
VAR TauxAbsence = [Taux_Absenteisme]
RETURN
    SWITCH(
        TRUE(),
        MoyenneEtudiant < 8 || TauxAbsence > 0.4, "🚨 Urgent",
        MoyenneEtudiant < 10 || TauxAbsence > 0.2, "⚠️ Suivi",
        "✅ OK"
    )
```

---

### Section 6 : Analyse des Absences (Line & Clustered Column)

**Graphique combiné** : Absentéisme par mois et par type
- **Colonnes** : Absences justifiées vs non justifiées
- **Ligne** : Taux d'absentéisme global
- **Axe X** : Mois

**Insights** :
- Pic d'absences en janvier (période d'examens)
- Baisse en avril-mai (projets de fin d'année)

---

### Page 2 : Analyse Financière de la Filière

### Section 1 : KPIs Financiers (Cards)

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Revenus Total│ Taux Recouvr.│   Impayés    │  Bourses     │
│  38.25M MAD  │    94.2%     │  2.22M MAD   │  340 (40%)   │
│  ↗ +7.5%    │  ↗ +2.1%    │  ↘ -12%     │  ↗ +5%      │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

---

### Section 2 : Évolution des Paiements (Area Chart)

**Graphique en aires empilées** : Flux de paiements par trimestre
- **Aires** : Paiements par mode (Virement, Chèque, Espèces, Carte)
- **Axe X** : Trimestre
- **Axe Y** : Montant (MAD)

---

### Section 3 : Répartition des Tarifs (Treemap)

**Treemap** : Visualisation des revenus par type de tarif
- Tarif normal : 28.5M (75%)
- Tarif réduit (boursier) : 7.1M (18%)
- Tarif étranger : 2.65M (7%)

---

### Section 4 : État des Paiements (Waterfall Chart)

**Graphique en cascade** : Flux financiers
```
Montant Attendu : 40.5M
  - Paiements reçus : -38.25M
  - Annulations : -0.03M
  = Impayés restants : 2.22M
```

---

### Section 5 : Tableau Détaillé des Étudiants (Table avec filtres)

| ID | Nom | Niveau | Tarif Annuel | Payé | Solde | Statut | Mode | Dernier Paiement |
|----|-----|--------|--------------|------|-------|--------|------|------------------|
| 1001 | ALAMI Sara | L3 | 45,000 | 45,000 | 0 | ✅ À jour | Virement | 15/10/2024 |
| 1002 | IDRISSI Omar | L1 | 45,000 | 15,000 | 30,000 | ⚠️ Retard | Chèque | 20/09/2024 |
| 1003 | BENJELLOUN Ali | M1 | 50,000 | 50,000 | 0 | ✅ À jour | Virement | 10/09/2024 |

**Slicers** :
- Statut paiement (À jour, En retard, Impayé)
- Niveau
- Mode de paiement

---

## 🔄 Interactions et Drill-Through

### Navigation
- **Drill-through** : Cliquer sur un étudiant → Page détaillée "Fiche Étudiant"
- **Drill-down** : Cliquer sur une matière → Détails par séance
- **Filtres croisés** : Sélection d'un niveau filtre tous les visuels

### Boutons d'Action
- **"Voir Détails Financiers"** : Navigue vers page 2
- **"Exporter Liste"** : Export Excel des étudiants affichés
- **"Retour Dashboard Direction"** : Retour à la vue exécutive

---

## 📊 Mesures DAX Supplémentaires

```dax
// Performance académique
Moyenne_Par_Matiere = 
CALCULATE(
    AVERAGE(Fait_Resultats[Note_Finale]),
    ALLEXCEPT(Dim_Matieres, Dim_Matieres[Nom_Matiere])
)

Taux_Reussite_Par_Matiere = 
DIVIDE(
    CALCULATE(COUNTROWS(Fait_Resultats), Fait_Resultats[Note_Finale] >= 10),
    COUNTROWS(Fait_Resultats),
    0
)

// Analyse comparative
Moyenne_N_1 = 
CALCULATE(
    [Moyenne_Generale],
    SAMEPERIODLASTYEAR('Calendrier'[Date])
)

Ecart_Moyenne = [Moyenne_Generale] - [Moyenne_N_1]

// Étudiants à risque
Nb_Etudiants_Risque = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    FILTER(
        Fait_Inscriptions,
        [Moyenne_Generale] < 10 || [Taux_Absenteisme] > 0.2
    )
)

// Financier
Revenus_Filiere = 
CALCULATE(
    SUM(Fait_Paiements[Montant_Paye]),
    USERELATIONSHIP(Dim_Filieres[ID_Filiere], Fait_Inscriptions[ID_Filiere])
)

Taux_Recouvrement_Filiere = 
DIVIDE(
    [Revenus_Filiere],
    SUM(Fait_Paiements[Montant_Attendu]),
    0
)

Nb_Boursiers = 
CALCULATE(
    COUNTROWS(Fait_Inscriptions),
    Fait_Inscriptions[Type_Tarif] = "Boursier"
)

Pct_Boursiers = 
DIVIDE(
    [Nb_Boursiers],
    [Effectif_Filiere],
    0
)
```

---

## 🎨 Mise en Forme Spécifique

### Couleurs par Niveau
- **L1** : #4472C4 (Bleu clair)
- **L2** : #ED7D31 (Orange)
- **L3** : #A5A5A5 (Gris)
- **M1** : #FFC000 (Jaune)
- **M2** : #5B9BD5 (Bleu foncé)

### Icônes de Statut
- ✅ À jour (vert)
- ⚠️ Suivi/Alerte (orange)
- 🚨 Urgent (rouge)
- 📊 Info (bleu)

---

## ✅ Checklist de Création

- [ ] Créer page 1 "Performance Académique"
- [ ] Ajouter 5 cards KPIs académiques
- [ ] Créer funnel chart (déperdition par niveau)
- [ ] Ajouter donut chart (répartition genre)
- [ ] Créer combo chart (évolution résultats)
- [ ] Insérer matrice matières/niveaux avec mise en forme conditionnelle
- [ ] Créer tableaux Top/Bottom performers
- [ ] Ajouter graphique absences
- [ ] Créer page 2 "Analyse Financière"
- [ ] Ajouter 4 cards KPIs financiers
- [ ] Créer area chart (flux paiements)
- [ ] Ajouter treemap (tarifs)
- [ ] Créer waterfall chart (état paiements)
- [ ] Insérer table détaillée étudiants
- [ ] Configurer drill-through vers fiche étudiant
- [ ] Ajouter boutons de navigation
- [ ] Tester filtres croisés
- [ ] Valider avec un directeur de filière

---

## 📝 Notes d'Utilisation

### Scénarios d'Usage
1. **Réunion pédagogique** : Identifier les matières problématiques
2. **Suivi étudiant** : Repérer les étudiants à risque d'échec
3. **Planification** : Analyser la déperdition par niveau
4. **Reporting financier** : Suivre le recouvrement des frais

### Actualisation
- **Temps réel** : Absences (sync quotidienne à 8h)
- **Hebdomadaire** : Notes et résultats (dimanche 22h)
- **Mensuelle** : Paiements et inscriptions (1er du mois à 6h)

### Alertes Automatiques
- Email si taux de réussite < 70%
- Notification si impayés > 10% des revenus
- Alerte si taux d'abandon > 5%
