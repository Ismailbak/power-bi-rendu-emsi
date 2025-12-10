# Dashboard Scolarité - Gestion Administrative

## 📊 Objectif
Fournir aux services de scolarité un outil de suivi opérationnel des inscriptions, absences, et gestion administrative quotidienne des étudiants.

---

## 🎯 Utilisateurs Cibles
- **Service Scolarité**
- **Assistants administratifs**
- **Secrétariats pédagogiques**

---

## 📐 Layout du Dashboard

### Page 1 : Suivi des Inscriptions

#### En-tête
- **Logo EMSI** + Titre : "Dashboard Scolarité - Gestion des Inscriptions"
- **Filtres** : 
  - Année universitaire
  - Filière
  - Type d'inscription (Nouvelle/Réinscription/Transfert)
  - Statut (En cours/Validée/En attente)

---

### Section 1 : KPIs Inscriptions (Cards)

```
┌───────────────┬───────────────┬───────────────┬───────────────┬───────────────┐
│Total Inscrits │   Nouvelles   │Réinscriptions │   Transferts  │  En Attente   │
│    2,547      │      852      │    1,620      │       75      │       45      │
│  ↗ +12.3%    │  ↗ +15.2%    │  ↗ +10.5%    │  ↗ +5.0%     │  ↘ -8.0%     │
└───────────────┴───────────────┴───────────────┴───────────────┴───────────────┘
```

**Mesures DAX :**
```dax
Total_Inscriptions = COUNTROWS(Fait_Inscriptions)

Nouvelles_Inscriptions = 
CALCULATE(
    [Total_Inscriptions],
    Fait_Inscriptions[Type_Inscription] = "Nouvelle"
)

Reinscriptions = 
CALCULATE(
    [Total_Inscriptions],
    Fait_Inscriptions[Type_Inscription] = "Réinscription"
)

Transferts = 
CALCULATE(
    [Total_Inscriptions],
    Fait_Inscriptions[Type_Inscription] = "Transfert"
)

Inscriptions_En_Attente = 
CALCULATE(
    [Total_Inscriptions],
    Fait_Inscriptions[Statut_Inscription] = "En attente"
)
```

---

### Section 2 : Évolution des Inscriptions (Line Chart)

**Graphique en ligne** : Flux d'inscriptions par semaine
- **Axe X** : Semaine (Août → Octobre)
- **Axe Y** : Nombre d'inscriptions
- **Séries** : 
  - Inscriptions validées (vert)
  - En attente (orange)
  - Annulations (rouge)

**Insights** :
- Pic d'inscriptions : 3ème semaine de septembre
- Cible : 100% des inscriptions validées avant 15 octobre

---

### Section 3 : Répartition par Filière (Stacked Bar Chart)

**Graphique en barres empilées** : Inscriptions par filière et type

| Filière | Nouvelles | Réinscriptions | Transferts | Total |
|---------|-----------|----------------|------------|-------|
| Génie Informatique | 285 | 540 | 25 | 850 |
| Génie Civil | 210 | 395 | 15 | 620 |
| Génie Industriel | 185 | 345 | 10 | 540 |
| Finance | 102 | 268 | 10 | 380 |
| Marketing | 70 | 72 | 15 | 157 |

**Couleurs** :
- Nouvelles : Bleu (#4472C4)
- Réinscriptions : Vert (#70AD47)
- Transferts : Orange (#ED7D31)

---

### Section 4 : Tableau des Inscriptions en Attente (Table)

| ID | Nom Étudiant | Filière | Type | Date Demande | Motif Attente | Jours | Action |
|----|--------------|---------|------|--------------|---------------|-------|--------|
| 5012 | TAHIRI Imane | GI | Nouvelle | 25/09/2024 | Documents manquants | 15 | 📄 Relance |
| 5023 | MOUSSA Karim | GC | Transfert | 28/09/2024 | Validation équivalence | 12 | ⏳ En cours |
| 5045 | BENANI Sofia | FIN | Nouvelle | 02/10/2024 | Paiement en attente | 8 | 💰 Rappel |

**Mise en forme conditionnelle** :
- 🔴 > 15 jours : Urgent
- 🟠 10-15 jours : Priorité
- 🟢 < 10 jours : Normal

**Mesure DAX :**
```dax
Jours_Attente = 
DATEDIFF(
    Fait_Inscriptions[Date_Demande],
    TODAY(),
    DAY
)

Alerte_Delai = 
SWITCH(
    TRUE(),
    [Jours_Attente] > 15, "🔴 Urgent",
    [Jours_Attente] > 10, "🟠 Priorité",
    "🟢 Normal"
)
```

---

### Section 5 : Statistiques par Statut (Donut Chart)

**Graphique en anneau** : État des inscriptions
- Validées : 2,502 (98.2%)
- En attente documents : 28 (1.1%)
- En attente paiement : 17 (0.7%)

---

### Page 2 : Suivi des Absences

### Section 1 : KPIs Absences (Cards)

```
┌────────────────┬────────────────┬────────────────┬────────────────┐
│Total Absences  │  Non Justifiées│   Justifiées   │Taux Absentéisme│
│    4,285       │     1,542      │     2,743      │      8.5%      │
│  ↗ +5.2%      │  ↗ +12.5%     │  ↗ +1.8%      │  ↗ +0.8%      │
└────────────────┴────────────────┴────────────────┴────────────────┘
```

**Mesures DAX :**
```dax
Total_Absences = COUNTROWS(Fait_Absences)

Absences_Non_Justifiees = 
CALCULATE(
    [Total_Absences],
    Fait_Absences[Justifiee] = "Non"
)

Absences_Justifiees = 
CALCULATE(
    [Total_Absences],
    Fait_Absences[Justifiee] = "Oui"
)

Taux_Absenteisme_Global = 
DIVIDE(
    [Total_Absences],
    CALCULATE(COUNTROWS(Fait_Planning) * [Total_Etudiants]),
    0
)
```

---

### Section 2 : Absences par Jour de la Semaine (Column Chart)

**Graphique en colonnes** : Distribution des absences

```
Lundi     ████████████████  685 absences
Mardi     ██████████████    598 absences
Mercredi  █████████████     556 absences
Jeudi     ███████████████   642 absences
Vendredi  ████████████████████ 804 absences (⚠️ Pic)
```

**Insight** : Pic d'absences le vendredi → Mesures correctives

---

### Section 3 : Absences par Filière et Niveau (Matrix)

**Matrice** : Taux d'absentéisme détaillé

| Filière | L1 | L2 | L3 | M1 | M2 | Moyenne |
|---------|----|----|----|----|----|---------| 
| Génie Informatique | 12% | 8% | 6% | 5% | 4% | 7.0% ✅ |
| Génie Civil | 15% | 11% | 9% | 7% | 6% | 9.6% ⚠️ |
| Génie Industriel | 14% | 10% | 8% | 6% | 5% | 8.6% |
| Finance | 10% | 7% | 5% | 4% | 3% | 5.8% ✅ |
| Marketing | 13% | 9% | 7% | 5% | 4% | 7.6% |
| **GLOBAL** | **12.8%** | **9.0%** | **7.0%** | **5.4%** | **4.4%** | **7.7%** |

**Mise en forme** :
- 🟢 < 8% : Bon
- 🟠 8-12% : Moyen
- 🔴 > 12% : Alarmant

---

### Section 4 : Top 20 Étudiants Absentéistes (Table)

| Rang | ID | Nom | Filière | Niveau | Total Abs. | Non Just. | Taux | Sanctions |
|------|----|-----|---------|--------|------------|-----------|------|-----------|
| 1 | 1245 | KARIMI Youssef | GI | L1 | 45 | 38 | 42% | ⚠️ Avertissement 2 |
| 2 | 2156 | BENNANI Leila | GC | L2 | 38 | 30 | 35% | ⚠️ Avertissement 1 |
| 3 | 3421 | IDRISSI Hassan | GIND | L1 | 35 | 28 | 33% | ⚠️ Avertissement 1 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

**Actions automatiques** :
- Taux > 40% : Convocation + Avertissement 2
- Taux 30-40% : Avertissement 1
- Taux 20-30% : Rappel par email

**Mesure DAX :**
```dax
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

Sanction_Recommandee = 
SWITCH(
    TRUE(),
    [Taux_Absence_Etudiant] > 0.4, "⚠️ Avertissement 2",
    [Taux_Absence_Etudiant] > 0.3, "⚠️ Avertissement 1",
    [Taux_Absence_Etudiant] > 0.2, "📧 Rappel email",
    "✅ OK"
)
```

---

### Section 5 : Calendrier des Absences (Heatmap/Calendar Visual)

**Carte thermique** : Absences par jour sur l'année universitaire
- **Plus foncé** : Plus d'absences
- **Patterns visibles** : Jours avant/après vacances, examens

---

### Page 3 : Gestion des Documents

### Section 1 : KPIs Documents (Cards)

```
┌────────────────┬────────────────┬────────────────┬────────────────┐
│Dossiers Complets│Documents Manq. │  Attestations  │Relevés de Notes│
│    2,380       │      167       │    1,245       │      856       │
│  93.4%         │  6.6%          │  délivrées     │  demandés      │
└────────────────┴────────────────┴────────────────┴────────────────┘
```

---

### Section 2 : Documents Manquants par Type (Bar Chart)

```
Photocopie CIN              ██████████████  72 étudiants
Photos d'identité          ████████████    58 étudiants
Relevé Bac                 ████████        42 étudiants
Certificat Scolarité       ██████          28 étudiants
Certificat Médical         ████            18 étudiants
```

---

### Section 3 : Tableau Suivi des Demandes (Table)

| ID Demande | Étudiant | Type Document | Date Demande | Délai Traitement | Statut | Actions |
|------------|----------|---------------|--------------|------------------|--------|---------|
| DOC-1024 | ALAMI Sara | Attestation Scolarité | 05/12/2024 | 2 jours | ✅ Traitée | Télécharger |
| DOC-1025 | IDRISSI Omar | Relevé Notes S1 | 06/12/2024 | 1 jour | ⏳ En cours | - |
| DOC-1026 | TAHIRI Imane | Certificat Scolarité | 07/12/2024 | 3 jours | ⚠️ Retard | 📧 Rappel |

**Slicers** :
- Type de document
- Statut (Traitée, En cours, En retard)
- Filière

---

## 🔄 Interactions et Fonctionnalités

### Drill-Through
- **Clic sur étudiant** → Page "Fiche Étudiant Complète" avec :
  - Informations personnelles
  - Historique des inscriptions
  - Liste complète des absences
  - Documents fournis/manquants
  - Historique des paiements

### Boutons d'Action
- **"Générer Attestation"** : Export PDF automatique
- **"Envoyer Relance"** : Email automatique pour documents manquants
- **"Exporter Liste Absents"** : Export Excel pour convocation
- **"Imprimer Liste"** : Export PDF formaté

### Alertes Automatiques
- 🔴 **Urgent** : Inscription en attente > 15 jours
- 🟠 **Priorité** : Taux d'absence étudiant > 30%
- 🟢 **Info** : Nouvelle demande de document

---

## 📊 Mesures DAX Supplémentaires

```dax
// Inscriptions
Taux_Validation_Inscriptions = 
DIVIDE(
    CALCULATE([Total_Inscriptions], Fait_Inscriptions[Statut_Inscription] = "Validée"),
    [Total_Inscriptions],
    0
)

Delai_Moyen_Validation = 
AVERAGE(
    DATEDIFF(
        Fait_Inscriptions[Date_Demande],
        Fait_Inscriptions[Date_Validation],
        DAY
    )
)

// Absences
Absences_Par_Etudiant = 
DIVIDE(
    [Total_Absences],
    [Total_Etudiants],
    0
)

Taux_Justification = 
DIVIDE(
    [Absences_Justifiees],
    [Total_Absences],
    0
)

Etudiants_Risque_Absence = 
CALCULATE(
    COUNTROWS(Dim_Etudiants),
    [Taux_Absence_Etudiant] > 0.2
)

// Documents
Taux_Dossiers_Complets = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fait_Inscriptions),
        Fait_Inscriptions[Dossier_Complet] = "Oui"
    ),
    [Total_Inscriptions],
    0
)

Nb_Documents_Manquants = 
CALCULATE(
    COUNTROWS(Fait_Documents),
    Fait_Documents[Statut] = "Manquant"
)

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

## 🎨 Mise en Forme

### Codes Couleur pour Statuts
- ✅ **Validé/Complet** : Vert (#28A745)
- ⏳ **En cours** : Bleu (#007BFF)
- ⚠️ **Attention** : Orange (#FFC107)
- 🔴 **Urgent/Manquant** : Rouge (#DC3545)

### Icônes Visuelles
- 📄 Documents
- 📧 Email/Communication
- 💰 Paiement
- ⏰ Délai
- 👤 Étudiant
- 📊 Statistique

---

## ✅ Checklist de Création

### Page 1 - Inscriptions
- [ ] Créer 5 cards KPIs inscriptions
- [ ] Ajouter line chart évolution inscriptions
- [ ] Créer stacked bar chart par filière
- [ ] Insérer table inscriptions en attente
- [ ] Ajouter donut chart statuts
- [ ] Configurer mise en forme conditionnelle (délais)

### Page 2 - Absences
- [ ] Créer 4 cards KPIs absences
- [ ] Ajouter column chart par jour de semaine
- [ ] Créer matrice absences par filière/niveau
- [ ] Insérer table Top 20 absentéistes
- [ ] Ajouter heatmap calendrier absences
- [ ] Configurer alertes automatiques

### Page 3 - Documents
- [ ] Créer 4 cards KPIs documents
- [ ] Ajouter bar chart documents manquants
- [ ] Insérer table suivi demandes
- [ ] Créer page drill-through fiche étudiant
- [ ] Ajouter boutons d'action (export, relance)
- [ ] Tester envoi d'emails automatiques

### Configuration Générale
- [ ] Ajouter filtres globaux (année, filière)
- [ ] Configurer navigation entre pages
- [ ] Tester drill-through et interactions
- [ ] Valider avec service scolarité
- [ ] Former les utilisateurs

---

## 📝 Notes d'Utilisation

### Cas d'Usage Quotidiens
1. **Matin (9h)** : Vérifier inscriptions en attente > 10 jours → Relances
2. **Après-midi (14h)** : Valider nouvelles inscriptions du jour
3. **Fin de semaine (Vendredi 17h)** : Export liste absences > 20% → Convocations
4. **Mensuel (1er du mois)** : Rapport complet des statistiques

### Actualisation des Données
- **Inscriptions** : Temps réel (synchronisation automatique toutes les heures)
- **Absences** : Quotidienne à 8h (saisie par enseignants veille)
- **Documents** : Temps réel (mise à jour manuelle par scolarité)

### Sécurité RLS
```dax
// Scolarité : Accès à toutes les données
[Role] = "Scolarite"

// Secrétariat : Accès limité à sa filière
[Filiere] = LOOKUPVALUE(Users[Filiere], Users[Email], USERPRINCIPALNAME())
```
