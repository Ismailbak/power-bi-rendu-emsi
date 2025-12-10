# 🔄 Documentation ETL - Power Query

## Vue d'Ensemble

Ce document détaille les transformations ETL (Extract, Transform, Load) appliquées aux données sources pour alimenter le modèle Power BI de l'EMSI. Toutes les transformations sont réalisées avec Power Query (langage M).

---

## 1. Architecture ETL

```
┌─────────────────────────────────────────────────────────────┐
│                    EXTRACT (Extraction)                      │
├─────────────────────────────────────────────────────────────┤
│  SQL Server    │    Excel Files    │    Access Database    │
│  (Académique)  │    (Finance)      │    (RH)               │
└────────┬───────┴────────┬───────────┴──────────┬────────────┘
         │                │                       │
         ▼                ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   TRANSFORM (Transformation)                 │
├─────────────────────────────────────────────────────────────┤
│  • Nettoyage données        • Normalisation formats         │
│  • Gestion valeurs manquantes • Déduplication              │
│  • Création colonnes calculées • Filtrage qualité          │
│  • Jointures et fusions     • Agrégations                  │
└────────┬────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                     LOAD (Chargement)                        │
├─────────────────────────────────────────────────────────────┤
│         Modèle Power BI (Schéma en Étoile)                 │
│    6 Tables de Faits  +  6 Tables de Dimensions            │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Extraction des Données (EXTRACT)

### 2.1 Connexion SQL Server (Académique)

**Source** : `EMSI-SQL-PROD\ACADEMIQUE`

**Code M - Connexion**
```m
let
    Source = Sql.Database("EMSI-SQL-PROD\ACADEMIQUE", "EMSI_Scolarite"),
    Navigation = Source{[Schema="dbo"]}[Data]
in
    Navigation
```

**Tables Extraites**
- `Etudiants`
- `Inscriptions`
- `Filieres`
- `Niveaux`
- `Absences_Etudiants`
- `Resultats`
- `Matieres`

**Paramètres Connexion**
```m
// Paramètre serveur (pour faciliter migration)
let
    ServeurSQL = "EMSI-SQL-PROD\ACADEMIQUE",
    BaseDonnees = "EMSI_Scolarite",
    RequeteSQL = null  // null = importer toutes les tables
in
    Sql.Database(ServeurSQL, BaseDonnees, [Query=RequeteSQL])
```

### 2.2 Connexion Excel (Finance)

**Source** : `\\EMSI-FILE-SRV\Comptabilite\`

**Code M - Combiner Fichiers Excel**
```m
let
    // Chemin du dossier
    CheminSource = "\\EMSI-FILE-SRV\Comptabilite\",
    Source = Folder.Files(CheminSource),
    
    // Filtrer uniquement fichiers Excel de paiements
    FichiersPaiements = Table.SelectRows(Source, 
        each Text.Contains([Name], "Paiements") and 
        [Extension] = ".xlsx"),
    
    // Fonction pour importer chaque fichier
    ImporterFichier = (contenu) => 
        Excel.Workbook(contenu, null, true),
    
    // Appliquer à tous les fichiers
    AjouterContenu = Table.AddColumn(FichiersPaiements, "Contenu", 
        each ImporterFichier([Content])),
    
    // Extraire les données
    DevelopperTables = Table.ExpandTableColumn(AjouterContenu, "Contenu", 
        {"Name", "Data"}, {"NomFeuille", "Donnees"}),
    
    // Ne garder que la feuille "Paiements"
    FiltrerFeuille = Table.SelectRows(DevelopperTables, 
        each [NomFeuille] = "Paiements"),
    
    // Développer les données
    DevelopperDonnees = Table.ExpandTableColumn(FiltrerFeuille, "Donnees")
in
    DevelopperDonnees
```

**Fichiers Traités**
- `Paiements_2023.xlsx`
- `Paiements_2024.xlsx`
- `Paiements_2025.xlsx`
- `Factures_*.xlsx`
- `Budget_Previsionnel.xlsx`

### 2.3 Connexion Access (RH)

**Source** : `\\EMSI-FILE-SRV\RH\GestionRH.accdb`

**Code M - Connexion Access**
```m
let
    Source = Access.Database(File.Contents("\\EMSI-FILE-SRV\RH\GestionRH.accdb")),
    
    // Importer table Professeurs
    Professeurs = Source{[Item="Professeurs",Kind="Table"]}[Data],
    
    // Importer table Planning
    Planning = Source{[Item="Planning_Enseignement",Kind="Table"]}[Data],
    
    // Importer table Absences
    Absences = Source{[Item="Absences_Professeurs",Kind="Table"]}[Data]
in
    Source
```

---

## 3. Transformations (TRANSFORM)

### 3.1 Dimension Calendrier (Générée)

**Code M Complet - Table Calendrier**
```m
let
    // Paramètres
    DateDebut = #date(2020, 9, 1),
    DateFin = #date(2028, 8, 31),
    
    // Générer liste de dates
    NbJours = Duration.Days(DateFin - DateDebut) + 1,
    ListeDates = List.Dates(DateDebut, NbJours, #duration(1,0,0,0)),
    TableDates = Table.FromList(ListeDates, Splitter.SplitByNothing(), {"Date"}),
    
    // Changer type
    ChangerType = Table.TransformColumnTypes(TableDates, {{"Date", type date}}),
    
    // Ajouter Année
    AjouterAnnee = Table.AddColumn(ChangerType, "Annee", 
        each Date.Year([Date]), Int64.Type),
    
    // Ajouter Mois
    AjouterMois = Table.AddColumn(AjouterAnnee, "Mois", 
        each Date.Month([Date]), Int64.Type),
    
    // Ajouter Nom Mois
    AjouterMoisNom = Table.AddColumn(AjouterMois, "Mois_Nom", 
        each Date.MonthName([Date], "fr-FR"), type text),
    
    // Ajouter Mois Court
    AjouterMoisCourt = Table.AddColumn(AjouterMoisNom, "Mois_Court", 
        each Text.Start([Mois_Nom], 3), type text),
    
    // Ajouter Trimestre
    AjouterTrimestre = Table.AddColumn(AjouterMoisCourt, "Trimestre", 
        each "T" & Number.ToText(Date.QuarterOfYear([Date])), type text),
    
    // Ajouter Semestre
    AjouterSemestre = Table.AddColumn(AjouterTrimestre, "Semestre", 
        each if [Mois] <= 6 then 1 else 2, Int64.Type),
    
    // Ajouter Semaine
    AjouterSemaine = Table.AddColumn(AjouterSemestre, "Semaine_Annee", 
        each Date.WeekOfYear([Date]), Int64.Type),
    
    // Ajouter Jour du Mois
    AjouterJourMois = Table.AddColumn(AjouterSemaine, "Jour_Mois", 
        each Date.Day([Date]), Int64.Type),
    
    // Ajouter Jour de l'Année
    AjouterJourAnnee = Table.AddColumn(AjouterJourMois, "Jour_Annee", 
        each Date.DayOfYear([Date]), Int64.Type),
    
    // Ajouter Jour de la Semaine (1=Lundi)
    AjouterJourSemaine = Table.AddColumn(AjouterJourAnnee, "Jour_Semaine", 
        each Date.DayOfWeek([Date], Day.Monday) + 1, Int64.Type),
    
    // Ajouter Nom Jour
    AjouterJourNom = Table.AddColumn(AjouterJourSemaine, "Jour_Semaine_Nom", 
        each Date.DayOfWeekName([Date], "fr-FR"), type text),
    
    // Ajouter Jour Court
    AjouterJourCourt = Table.AddColumn(AjouterJourNom, "Jour_Semaine_Court", 
        each Text.Start([Jour_Semaine_Nom], 3), type text),
    
    // Ajouter Est Weekend
    AjouterWeekend = Table.AddColumn(AjouterJourCourt, "Est_Weekend", 
        each [Jour_Semaine] >= 6, type logical),
    
    // Ajouter Jours Fériés Marocains
    AjouterFerie = Table.AddColumn(AjouterWeekend, "Est_Jour_Ferie", 
        each List.Contains({
            #date(Date.Year([Date]), 1, 1),   // Nouvel An
            #date(Date.Year([Date]), 1, 11),  // Manifeste Indépendance
            #date(Date.Year([Date]), 5, 1),   // Fête du Travail
            #date(Date.Year([Date]), 7, 30),  // Fête du Trône
            #date(Date.Year([Date]), 8, 14),  // Oued Eddahab
            #date(Date.Year([Date]), 8, 20),  // Révolution du Roi
            #date(Date.Year([Date]), 8, 21),  // Fête de la Jeunesse
            #date(Date.Year([Date]), 11, 6),  // Marche Verte
            #date(Date.Year([Date]), 11, 18)  // Fête Indépendance
        }, [Date]), type logical),
    
    // Ajouter Année Universitaire (Sept-Août)
    AjouterAnneeUniv = Table.AddColumn(AjouterFerie, "Annee_Universitaire", 
        each if [Mois] >= 9 
             then Text.From([Annee]) & "-" & Text.From([Annee]+1)
             else Text.From([Annee]-1) & "-" & Text.From([Annee]), type text),
    
    // Ajouter Semestre Universitaire
    AjouterSemestreUniv = Table.AddColumn(AjouterAnneeUniv, "Semestre_Univ", 
        each if [Mois] >= 9 or [Mois] <= 1 then "S1" else "S2", type text),
    
    // Ajouter Début/Fin de Mois
    AjouterDebutMois = Table.AddColumn(AjouterSemestreUniv, "Debut_Mois", 
        each Date.StartOfMonth([Date]), type date),
    
    AjouterFinMois = Table.AddColumn(AjouterDebutMois, "Fin_Mois", 
        each Date.EndOfMonth([Date]), type date)
in
    AjouterFinMois
```

### 3.2 Dimension Étudiants

**Transformations Appliquées**

**Code M - Nettoyage Étudiants**
```m
let
    // Source SQL
    Source = Sql.Database("EMSI-SQL-PROD\ACADEMIQUE", "EMSI_Scolarite"),
    Etudiants = Source{[Schema="dbo",Item="Etudiants"]}[Data],
    
    // 1. Supprimer colonnes inutiles
    SupprimerColonnes = Table.RemoveColumns(Etudiants, {
        "Photo", "Commentaires", "Date_Creation", "Utilisateur_Modif"
    }),
    
    // 2. Nettoyer Nom (Majuscules, Trim)
    NettoyerNom = Table.TransformColumns(SupprimerColonnes, {
        {"Nom", each Text.Upper(Text.Trim(_)), type text},
        {"Prenom", each Text.Proper(Text.Trim(_)), type text}
    }),
    
    // 3. Créer Nom_Complet
    AjouterNomComplet = Table.AddColumn(NettoyerNom, "Nom_Complet", 
        each [Nom] & " " & [Prenom], type text),
    
    // 4. Calculer Âge
    AjouterAge = Table.AddColumn(AjouterNomComplet, "Age", 
        each Duration.Days(DateTime.Date(DateTime.LocalNow()) - [Date_Naissance]) / 365.25, 
        Int64.Type),
    
    // 5. Arrondir Âge
    ArrondirAge = Table.TransformColumns(AjouterAge, {
        {"Age", each Number.RoundDown(_, 0), Int64.Type}
    }),
    
    // 6. Remplacer valeurs NULL
    RemplacerNull = Table.ReplaceValue(ArrondirAge, null, "Non renseigné", 
        Replacer.ReplaceValue, {"Ville", "Email", "Telephone"}),
    
    // 7. Nettoyer Email (lowercase)
    NettoyerEmail = Table.TransformColumns(RemplacerNull, {
        {"Email", each Text.Lower(Text.Trim(_)), type text}
    }),
    
    // 8. Formater Téléphone (supprimer espaces/tirets)
    FormaterTelephone = Table.TransformColumns(NettoyerEmail, {
        {"Telephone", each Text.Replace(Text.Replace(_, " ", ""), "-", ""), type text}
    }),
    
    // 9. Calculer Ancienneté
    AjouterAnciennete = Table.AddColumn(FormaterTelephone, "Anciennete_Annees", 
        each Duration.Days(DateTime.Date(DateTime.LocalNow()) - [Date_Premiere_Inscription]) / 365.25,
        Int64.Type),
    
    // 10. Arrondir Ancienneté
    ArrondirAnciennete = Table.TransformColumns(AjouterAnciennete, {
        {"Anciennete_Annees", each Number.RoundDown(_, 0), Int64.Type}
    }),
    
    // 11. Créer colonne Région (mapping Ville)
    AjouterRegion = Table.AddColumn(ArrondirAnciennete, "Region", 
        each if [Ville] = "Casablanca" then "Grand Casablanca"
             else if [Ville] = "Rabat" then "Rabat-Salé-Kénitra"
             else if [Ville] = "Marrakech" then "Marrakech-Safi"
             else if [Ville] = "Fès" then "Fès-Meknès"
             else if [Ville] = "Tanger" then "Tanger-Tétouan-Al Hoceïma"
             else if [Ville] = "Agadir" then "Souss-Massa"
             else "Autre", type text),
    
    // 12. Types de données finaux
    ChangerTypes = Table.TransformColumnTypes(AjouterRegion, {
        {"ID_Etudiant", Int64.Type},
        {"Code_Massar", type text},
        {"Date_Naissance", type date},
        {"Date_Premiere_Inscription", type date},
        {"Statut_Actuel", type text}
    })
in
    ChangerTypes
```

### 3.3 Dimension Filières

**Code M - Filières**
```m
let
    Source = Sql.Database("EMSI-SQL-PROD\ACADEMIQUE", "EMSI_Scolarite"),
    Filieres = Source{[Schema="dbo",Item="Filieres"]}[Data],
    
    // Nettoyer noms
    NettoyerNom = Table.TransformColumns(Filieres, {
        {"Nom_Filiere", each Text.Proper(Text.Trim(_)), type text}
    }),
    
    // Ajouter Code Court (si manquant)
    AjouterCode = Table.AddColumn(NettoyerNom, "Code_Filiere_Calc", 
        each if [Code_Filiere] = null or [Code_Filiere] = "" 
             then Text.Upper(Text.Start([Nom_Filiere], 2))
             else [Code_Filiere], type text),
    
    // Remplacer colonne Code
    SupprimerAncienCode = Table.RemoveColumns(AjouterCode, {"Code_Filiere"}),
    RenommerCode = Table.RenameColumns(SupprimerAncienCode, {
        {"Code_Filiere_Calc", "Code_Filiere"}
    }),
    
    // Types finaux
    ChangerTypes = Table.TransformColumnTypes(RenommerCode, {
        {"ID_Filiere", Int64.Type},
        {"Tarif_Annuel", type number},
        {"Capacite_Max", Int64.Type}
    })
in
    ChangerTypes
```

### 3.4 Fait Inscriptions

**Code M - Table de Faits Inscriptions**
```m
let
    // Source
    Source = Sql.Database("EMSI-SQL-PROD\ACADEMIQUE", "EMSI_Scolarite"),
    Inscriptions = Source{[Schema="dbo",Item="Inscriptions"]}[Data],
    
    // 1. Filtrer statut valides uniquement
    FiltrerStatut = Table.SelectRows(Inscriptions, 
        each [Statut] <> "Supprimé" and [Statut] <> "Erreur"),
    
    // 2. Supprimer doublons (même étudiant + filière + année)
    SupprimerDoublons = Table.Distinct(FiltrerStatut, {
        "ID_Etudiant", "ID_Filiere", "ID_Niveau", "Annee_Universitaire"
    }),
    
    // 3. Vérifier intégrité référentielle (FK valides)
    // Charger dimension Étudiants
    Etudiants = Dim_Etudiants,
    JoinEtudiants = Table.NestedJoin(SupprimerDoublons, {"ID_Etudiant"}, 
        Etudiants, {"ID_Etudiant"}, "VerifEtudiant", JoinKind.LeftOuter),
    
    // Filtrer uniquement inscriptions avec étudiant valide
    FiltrerOrphelins = Table.SelectRows(JoinEtudiants, 
        each [VerifEtudiant] <> null),
    
    // Supprimer colonne de vérification
    SupprimerVerif = Table.RemoveColumns(FiltrerOrphelins, {"VerifEtudiant"}),
    
    // 4. Remplacer valeurs NULL
    RemplacerNullMontant = Table.ReplaceValue(SupprimerVerif, null, 0, 
        Replacer.ReplaceValue, {"Montant_Annuel"}),
    
    // 5. Calculer Type_Inscription (si manquant)
    AjouterTypeInscription = Table.AddColumn(RemplacerNullMontant, "Type_Inscription_Calc",
        each if [Type_Inscription] = null or [Type_Inscription] = ""
             then 
                // Vérifier si étudiant a inscription année précédente
                let 
                    AnneeActuelle = Text.From(Date.Year([Date_Inscription])),
                    AnneePrecedente = Text.From(Date.Year([Date_Inscription]) - 1) & "-" & AnneeActuelle
                in
                    if Table.RowCount(
                        Table.SelectRows(Inscriptions, 
                            each [ID_Etudiant] = [ID_Etudiant] and 
                                 [Annee_Universitaire] = AnneePrecedente)
                    ) > 0 then "Réinscription" else "Nouvelle"
             else [Type_Inscription],
        type text),
    
    // Remplacer colonne
    SupprimerAncienType = Table.RemoveColumns(AjouterTypeInscription, {"Type_Inscription"}),
    RenommerType = Table.RenameColumns(SupprimerAncienType, {
        {"Type_Inscription_Calc", "Type_Inscription"}
    }),
    
    // 6. Types finaux
    ChangerTypes = Table.TransformColumnTypes(RenommerType, {
        {"ID_Inscription", Int64.Type},
        {"ID_Etudiant", Int64.Type},
        {"ID_Filiere", Int64.Type},
        {"ID_Niveau", Int64.Type},
        {"Date_Inscription", type date},
        {"Montant_Annuel", type number}
    })
in
    ChangerTypes
```

### 3.5 Fait Paiements (Excel)

**Code M - Combiner et Nettoyer Paiements**
```m
let
    // Source dossier
    Source = Folder.Files("\\EMSI-FILE-SRV\Comptabilite\"),
    
    // Filtrer fichiers Paiements
    FiltrerPaiements = Table.SelectRows(Source, 
        each Text.Contains([Name], "Paiements") and [Extension] = ".xlsx"),
    
    // Fonction import Excel
    ImporterExcel = (contenu) => 
        let
            Workbook = Excel.Workbook(contenu),
            Feuille = Workbook{[Item="Paiements",Kind="Sheet"]}[Data],
            PromoterHeaders = Table.PromoteHeaders(Feuille, [PromoteAllScalars=true])
        in
            PromoterHeaders,
    
    // Appliquer à tous fichiers
    AjouterDonnees = Table.AddColumn(FiltrerPaiements, "Donnees", 
        each ImporterExcel([Content])),
    
    // Combiner toutes les tables
    DevelopperDonnees = Table.ExpandTableColumn(AjouterDonnees, "Donnees",
        {"ID_Paiement", "Code_Etudiant", "Date_Paiement", "Montant", 
         "Mode_Paiement", "Reference", "Annee_Universitaire", "Trimestre"}),
    
    // Supprimer colonnes inutiles (métadonnées fichier)
    SupprimerColonnes = Table.SelectColumns(DevelopperDonnees, {
        "ID_Paiement", "Code_Etudiant", "Date_Paiement", "Montant",
        "Mode_Paiement", "Reference", "Annee_Universitaire", "Trimestre"
    }),
    
    // Nettoyer Code_Etudiant (supprimer espaces)
    NettoyerCode = Table.TransformColumns(SupprimerColonnes, {
        {"Code_Etudiant", each Text.Trim(_), type text}
    }),
    
    // Joindre avec Dim_Etudiants pour obtenir ID_Etudiant
    Etudiants = Dim_Etudiants,
    JoinEtudiants = Table.NestedJoin(NettoyerCode, {"Code_Etudiant"},
        Etudiants, {"Code_Massar"}, "Etudiant", JoinKind.LeftOuter),
    
    // Extraire ID_Etudiant
    DevelopperID = Table.ExpandTableColumn(JoinEtudiants, "Etudiant", 
        {"ID_Etudiant"}, {"ID_Etudiant"}),
    
    // Filtrer uniquement paiements avec étudiant trouvé
    FiltrerValides = Table.SelectRows(DevelopperID, 
        each [ID_Etudiant] <> null),
    
    // Supprimer Code_Etudiant (garder ID)
    SupprimerCode = Table.RemoveColumns(FiltrerValides, {"Code_Etudiant"}),
    
    // Supprimer doublons (même ID_Paiement)
    SupprimerDoublons = Table.Distinct(SupprimerCode, {"ID_Paiement"}),
    
    // Nettoyer Mode_Paiement (standardiser)
    StandardiserMode = Table.ReplaceValue(SupprimerDoublons, 
        each [Mode_Paiement],
        each if Text.Contains(Text.Upper([Mode_Paiement]), "ESP") then "Espèces"
             else if Text.Contains(Text.Upper([Mode_Paiement]), "CHE") then "Chèque"
             else if Text.Contains(Text.Upper([Mode_Paiement]), "VIR") then "Virement"
             else "Autre",
        Replacer.ReplaceValue, {"Mode_Paiement"}),
    
    // Ajouter colonnes calculées (Montant_Attendu depuis Factures)
    // Charger table Factures
    Factures = Fait_Factures,
    JoinFactures = Table.NestedJoin(StandardiserMode, 
        {"ID_Etudiant", "Annee_Universitaire"},
        Factures, {"ID_Etudiant", "Annee_Universitaire"},
        "Facture", JoinKind.LeftOuter),
    
    // Extraire Montant_Attendu
    DevelopperFacture = Table.ExpandTableColumn(JoinFactures, "Facture",
        {"Montant_Total", "Solde"}, {"Montant_Attendu", "Solde_Restant"}),
    
    // Remplacer NULL par 0
    RemplacerNull = Table.ReplaceValue(DevelopperFacture, null, 0,
        Replacer.ReplaceValue, {"Montant_Attendu", "Solde_Restant"}),
    
    // Types finaux
    ChangerTypes = Table.TransformColumnTypes(RemplacerNull, {
        {"ID_Paiement", Int64.Type},
        {"ID_Etudiant", Int64.Type},
        {"Date_Paiement", type date},
        {"Montant", type number},
        {"Montant_Attendu", type number},
        {"Solde_Restant", type number}
    })
in
    ChangerTypes
```

### 3.6 Fait Planning Enseignement (Access)

**Code M - Planning**
```m
let
    Source = Access.Database(File.Contents("\\EMSI-FILE-SRV\RH\GestionRH.accdb")),
    Planning = Source{[Item="Planning_Enseignement",Kind="Table"]}[Data],
    
    // Supprimer colonnes inutiles
    SupprimerColonnes = Table.RemoveColumns(Planning, {
        "Date_Creation", "Utilisateur_Modif"
    }),
    
    // Remplacer NULL dans Heures_Realisees par 0
    RemplacerNull = Table.ReplaceValue(SupprimerColonnes, null, 0,
        Replacer.ReplaceValue, {"Heures_Realisees"}),
    
    // Joindre avec Professeurs pour obtenir Taux_Horaire
    Professeurs = Dim_Professeurs,
    JoinProfs = Table.NestedJoin(RemplacerNull, {"ID_Professeur"},
        Professeurs, {"ID_Professeur"}, "Prof", JoinKind.LeftOuter),
    
    // Extraire Taux_Horaire
    DevelopperProf = Table.ExpandTableColumn(JoinProfs, "Prof",
        {"Taux_Horaire"}, {"Taux_Horaire"}),
    
    // Filtrer planning valide
    FiltrerValide = Table.SelectRows(DevelopperProf,
        each [Heures_Prevues] > 0 and [Taux_Horaire] <> null),
    
    // Convertir texte jours en date (calculer Date_Debut)
    // Si Jour_Semaine fourni, calculer première occurrence dans semestre
    AjouterDateDebut = Table.AddColumn(FiltrerValide, "Date_Debut_Calc",
        each 
            let
                AnneeDemarrage = if Text.Start([Annee_Universitaire], 4) <> null
                                 then Number.FromText(Text.Start([Annee_Universitaire], 4))
                                 else Date.Year(DateTime.Date(DateTime.LocalNow())),
                MoisDemarrage = if [Semestre] = "S1" then 9 else 2,
                PremierJourSemestre = #date(AnneeDemarrage, MoisDemarrage, 1),
                // Trouver premier jour correspondant
                JourCible = if [Jour_Semaine] = "Lundi" then 1
                           else if [Jour_Semaine] = "Mardi" then 2
                           else if [Jour_Semaine] = "Mercredi" then 3
                           else if [Jour_Semaine] = "Jeudi" then 4
                           else if [Jour_Semaine] = "Vendredi" then 5
                           else 1,
                JourActuel = Date.DayOfWeek(PremierJourSemestre, Day.Monday) + 1,
                DecalageJours = if JourCible >= JourActuel 
                               then JourCible - JourActuel
                               else 7 - JourActuel + JourCible
            in
                Date.AddDays(PremierJourSemestre, DecalageJours),
        type date),
    
    // Types finaux
    ChangerTypes = Table.TransformColumnTypes(AjouterDateDebut, {
        {"ID_Planning", Int64.Type},
        {"ID_Professeur", Int64.Type},
        {"ID_Matiere", Int64.Type},
        {"ID_Filiere", Int64.Type},
        {"Heures_Prevues", Int64.Type},
        {"Heures_Realisees", Int64.Type},
        {"Taux_Horaire", type number}
    })
in
    ChangerTypes
```

---

## 4. Règles de Qualité des Données

### 4.1 Valeurs Manquantes (NULL)

| Colonne | Action | Valeur Remplacement |
|---------|--------|-------------------|
| Montant_Annuel | Remplacer | 0 |
| Heures_Realisees | Remplacer | 0 |
| Ville | Remplacer | "Non renseigné" |
| Email | Remplacer | "Non renseigné" |
| Telephone | Remplacer | "Non renseigné" |
| Type_Absence | Conserver NULL | - |
| Justification | Conserver NULL | - |

**Code M Générique - Remplacer NULL**
```m
Table.ReplaceValue(
    Source,
    null,
    "Valeur_Par_Defaut",
    Replacer.ReplaceValue,
    {"Colonne1", "Colonne2"}
)
```

### 4.2 Déduplication

**Règles Appliquées**
- Dim_Etudiants : Dédupliquer sur `Code_Massar`
- Fait_Inscriptions : Dédupliquer sur `{ID_Etudiant, ID_Filiere, Annee_Universitaire}`
- Fait_Paiements : Dédupliquer sur `ID_Paiement`

**Code M - Supprimer Doublons**
```m
// Sur une seule colonne
Table.Distinct(Source, {"Code_Massar"})

// Sur plusieurs colonnes
Table.Distinct(Source, {"ID_Etudiant", "ID_Filiere", "Annee_Universitaire"})
```

### 4.3 Validation Formats

**Emails**
```m
// Valider format email
Table.SelectRows(Source, 
    each Text.Contains([Email], "@") and Text.Contains([Email], "."))
```

**Téléphones (Format Marocain)**
```m
// Valider format 06XXXXXXXX ou 07XXXXXXXX
Table.SelectRows(Source,
    each Text.Length([Telephone]) = 10 and 
         (Text.Start([Telephone], 2) = "06" or Text.Start([Telephone], 2) = "07"))
```

**Dates**
```m
// Filtrer dates valides (pas dans le futur)
Table.SelectRows(Source,
    each [Date_Paiement] <= DateTime.Date(DateTime.LocalNow()))
```

### 4.4 Intégrité Référentielle

**Vérifier FK Valides**
```m
// Exemple : Vérifier que chaque inscription a un étudiant valide
let
    Inscriptions = Fait_Inscriptions,
    Etudiants = Dim_Etudiants,
    
    // Left Join
    Verification = Table.NestedJoin(Inscriptions, {"ID_Etudiant"},
        Etudiants, {"ID_Etudiant"}, "Verif", JoinKind.LeftOuter),
    
    // Filtrer uniquement lignes avec étudiant trouvé
    FiltrerValides = Table.SelectRows(Verification, 
        each [Verif] <> null),
    
    // Supprimer colonne temporaire
    Resultat = Table.RemoveColumns(FiltrerValides, {"Verif"})
in
    Resultat
```

---

## 5. Optimisations Performance

### 5.1 Requêtes Pliables (Query Folding)

**✅ Bonnes Pratiques**
```m
// 1. Filtrer à la source (SQL)
let
    Source = Sql.Database("Server", "DB", [Query="
        SELECT * 
        FROM Etudiants 
        WHERE Statut_Actuel = 'Actif' 
          AND Date_Inscription >= '2020-01-01'
    "])
in
    Source
```

**❌ À Éviter**
```m
// Filtrer après import (non pliable)
let
    Source = Sql.Database("Server", "DB"),
    Etudiants = Source{[Schema="dbo",Item="Etudiants"]}[Data],
    Filtrer = Table.SelectRows(Etudiants, each [Statut_Actuel] = "Actif")
in
    Filtrer
```

### 5.2 Mise en Cache (Buffering)

**Code M - Buffer pour Requêtes Multiples**
```m
let
    // Charger et mettre en cache
    Source = Table.Buffer(
        Sql.Database("Server", "DB")
    ),
    
    // Utiliser plusieurs fois sans recharger
    Etudiants = Source{[Item="Etudiants"]}[Data],
    Inscriptions = Source{[Item="Inscriptions"]}[Data]
in
    Etudiants
```

### 5.3 Désactiver Détection Types Automatique

**Paramètre Connexion Excel**
```m
Excel.Workbook(File.Contents("chemin.xlsx"), null, true)
// true = pas de détection auto des types (plus rapide)
```

---

## 6. Gestion des Erreurs

### 6.1 Try-Catch

**Code M - Gestion Erreurs Import**
```m
let
    Source = try Sql.Database("Server", "DB") otherwise null,
    
    Resultat = if Source = null 
               then #table({"Erreur"}, {{"Connexion échouée"}})
               else Source
in
    Resultat
```

### 6.2 Logging des Erreurs

**Code M - Table Log Erreurs**
```m
let
    Source = Folder.Files("\\Chemin\"),
    
    // Fonction import avec gestion erreur
    ImporterAvecLog = (fichier) => 
        try 
            Excel.Workbook(fichier[Content])
        otherwise 
            #table(
                {"Fichier", "Erreur"},
                {{fichier[Name], "Échec import"}}
            ),
    
    Resultat = Table.AddColumn(Source, "Donnees", each ImporterAvecLog(_))
in
    Resultat
```

---

## 7. Paramètres et Variables

### 7.1 Paramètres Power BI

**Créer Paramètres**
```m
// Paramètre Chemin Serveur
let
    ServeurSQL = "EMSI-SQL-PROD\ACADEMIQUE" meta [IsParameterQuery=true, Type="Text"]
in
    ServeurSQL

// Paramètre Année Universitaire
let
    AnneeUniv = "2024-2025" meta [IsParameterQuery=true, Type="Text"]
in
    AnneeUniv
```

**Utiliser Paramètres**
```m
let
    Source = Sql.Database(ServeurSQL, "EMSI_Scolarite"),
    Filtrer = Table.SelectRows(Source, 
        each [Annee_Universitaire] = AnneeUniv)
in
    Filtrer
```

### 7.2 Variables Locales

**Code M - Variables**
```m
let
    // Variables
    DateDebut = #date(2020, 9, 1),
    DateFin = Date.From(DateTime.LocalNow()),
    
    // Utilisation
    Source = Sql.Database("Server", "DB", [Query="
        SELECT * FROM Inscriptions
        WHERE Date_Inscription BETWEEN '" & Date.ToText(DateDebut) & 
        "' AND '" & Date.ToText(DateFin) & "'
    "])
in
    Source
```

---

## 8. Documentation des Étapes

### 8.1 Commentaires dans Power Query

**Bonnes Pratiques**
```m
let
    // ===== EXTRACTION =====
    // Connexion à la base SQL Server académique
    Source = Sql.Database("EMSI-SQL-PROD\ACADEMIQUE", "EMSI_Scolarite"),
    
    // ===== TRANSFORMATION =====
    // Étape 1 : Supprimer colonnes métadonnées inutiles
    SupprimerColonnes = Table.RemoveColumns(Source, {"Date_Creation", "User_ID"}),
    
    // Étape 2 : Nettoyer format noms (UPPER + TRIM)
    NettoyerNoms = Table.TransformColumns(SupprimerColonnes, {
        {"Nom", each Text.Upper(Text.Trim(_)), type text}
    }),
    
    // ===== VALIDATION =====
    // Étape 3 : Filtrer uniquement étudiants actifs
    FiltrerActifs = Table.SelectRows(NettoyerNoms, 
        each [Statut] = "Actif")
in
    FiltrerActifs
```

### 8.2 Renommer Étapes Explicitement

**❌ Noms Générés Auto**
- Changed Type
- Removed Columns
- Filtered Rows

**✅ Noms Explicites**
- Conversion_Types_Initiaux
- Suppression_Colonnes_Metadata
- Filtrage_Etudiants_Actifs

---

## 9. Checklist ETL

### 9.1 Avant Actualisation

✅ **Vérifications**
- [ ] Connexions sources accessibles (réseau, VPN)
- [ ] Credentials à jour
- [ ] Espace disque suffisant
- [ ] Aucun fichier Excel source ouvert
- [ ] Paramètres Power BI validés

### 9.2 Après Actualisation

✅ **Contrôles Qualité**
- [ ] Nombre de lignes cohérent
- [ ] Pas d'erreurs dans tables
- [ ] Dates min/max correctes
- [ ] Relations modèle intactes
- [ ] Mesures DAX fonctionnelles
- [ ] Visuels s'affichent correctement

---

## 📅 Informations Document

**Version** : 1.0  
**Date de création** : 10 Décembre 2025  
**Dernière mise à jour** : 10 Décembre 2025  
**Auteur** : Équipe BI EMSI  
**Statut** : ✅ Validé  
**Requêtes Power Query** : 15+

---

*École Marocaine des Sciences de l'Ingénieur - Projet Business Intelligence*
