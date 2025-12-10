# 📋 Cahier des Charges - Projet Power BI EMSI

## 1. Contexte du Projet

### 1.1 Présentation
L'École Marocaine des Sciences de l'Ingénieur (EMSI) souhaite mettre en place une solution de Business Intelligence permettant le pilotage et l'analyse de ses activités académiques, financières et administratives.

### 1.2 Problématique
- Dispersion des données dans plusieurs systèmes
- Absence de vision consolidée des performances
- Difficulté à générer des rapports en temps réel
- Manque d'outils d'aide à la décision

### 1.3 Objectif Global
Créer un ensemble de tableaux de bord Power BI permettant aux différents acteurs de l'école de suivre les indicateurs clés et de prendre des décisions éclairées.

---

## 2. Périmètre du Projet

### 2.1 Domaines Couverts
1. **Académique**
   - Suivi des étudiants par filière et niveau
   - Gestion des inscriptions et réinscriptions
   - Suivi des absences étudiants
   - Taux de réussite et résultats

2. **Financier**
   - Suivi des paiements et recouvrements
   - Analyse des revenus par filière
   - Gestion des impayés
   - Budget et prévisions

3. **Ressources Humaines**
   - Suivi de la charge d'enseignement
   - Absences professeurs
   - Heures réalisées vs. planifiées
   - Répartition par département

### 2.2 Exclusions
- Gestion détaillée des notes (hors périmètre phase 1)
- Système de paie RH
- Gestion des locaux et équipements

---

## 3. Utilisateurs Cibles

### 3.1 Profils Utilisateurs

| Profil | Besoins | Dashboard Principal |
|--------|---------|-------------------|
| **Direction Générale** | Vision stratégique, indicateurs globaux | Dashboard Direction |
| **Responsables de Filière** | Performance académique par filière | Dashboard Filière |
| **Service Scolarité** | Inscriptions, absences, effectifs | Dashboard Scolarité |
| **Service Comptabilité** | Paiements, recouvrements, budget | Dashboard Financier |
| **Professeurs/RH** | Charge enseignement, absences | Dashboard RH |

### 3.2 Droits d'Accès
- **Direction** : Accès complet à tous les dashboards
- **Responsables Filière** : Données de leur filière uniquement (RLS)
- **Scolarité** : Données académiques et étudiants
- **Comptabilité** : Données financières
- **Professeurs** : Leurs propres données uniquement

---

## 4. Objectifs Fonctionnels

### 4.1 Dashboard Direction (Stratégique)
- Vue consolidée des KPIs globaux
- Évolution des effectifs étudiants
- Performance financière
- Taux de réussite global
- Alertes sur indicateurs critiques

### 4.2 Dashboard Filière
- Effectifs par niveau et année
- Taux de réussite par matière
- Comparaison inter-filières
- Évolution temporelle des inscriptions
- Analyse des abandons

### 4.3 Dashboard Scolarité
- Nouvelles inscriptions et réinscriptions
- Suivi des absences étudiants
- Répartition par filière et niveau
- Listes d'étudiants à risque
- Statistiques d'assiduité

### 4.4 Dashboard Financier
- Montant encaissé vs. attendu
- Taux de recouvrement
- Analyse des impayés par filière
- Prévisions de trésorerie
- Top étudiants en retard de paiement

### 4.5 Dashboard RH
- Heures réalisées par professeur
- Charge d'enseignement par département
- Absences professeurs
- Répartition des cours
- Coût salarial par filière

---

## 5. Objectifs Techniques

### 5.1 Architecture Cible
- **Sources de données** : Excel, SQL Server, Access
- **ETL** : Power Query
- **Modélisation** : Schéma en étoile
- **Visualisation** : Power BI Desktop
- **Publication** : Power BI Service
- **Sécurité** : Row-Level Security (RLS)

### 5.2 Performances Attendues
- Temps de chargement < 3 secondes
- Actualisation automatique quotidienne
- Historique de données : 3 ans minimum
- Capacité : 10 000+ étudiants

### 5.3 Standards et Bonnes Pratiques
- Respect de la charte graphique EMSI
- Nommage cohérent des mesures DAX
- Documentation complète du modèle
- Versioning des fichiers PBIX

---

## 6. KPIs et Métriques

### 6.1 KPIs Académiques
| KPI | Formule | Objectif |
|-----|---------|----------|
| Taux de réussite | (Admis / Inscrits) × 100 | > 75% |
| Taux d'absentéisme étudiant | (Absences / Total séances) × 100 | < 10% |
| Taux d'abandon | (Abandons / Inscrits) × 100 | < 5% |
| Effectif par filière | COUNT(Étudiants) | Suivi |

### 6.2 KPIs Financiers
| KPI | Formule | Objectif |
|-----|---------|----------|
| Taux de recouvrement | (Encaissé / Attendu) × 100 | > 90% |
| Montant impayé | SUM(Attendu - Payé) | Minimiser |
| Revenus par filière | SUM(Paiements) | Suivi |
| Délai moyen de paiement | AVG(Date paiement - Date échéance) | < 30 jours |

### 6.3 KPIs RH
| KPI | Formule | Objectif |
|-----|---------|----------|
| Heures réalisées | SUM(Heures effectuées) | = Planifié |
| Taux d'absence prof | (Absences / Séances prévues) × 100 | < 5% |
| Charge moyenne | AVG(Heures / Professeur) | 18h/semaine |

---

## 7. Contraintes et Exigences

### 7.1 Contraintes Techniques
- Utilisation exclusive de Power BI (licence Pro)
- Pas de développement Custom Visual
- Compatibilité navigateurs : Chrome, Edge, Firefox
- Responsive design pour tablettes

### 7.2 Contraintes de Données
- Protection des données personnelles (RGPD)
- Anonymisation si nécessaire
- Backup quotidien du fichier PBIX
- Traçabilité des modifications

### 7.3 Contraintes Organisationnelles
- Formation utilisateurs : 2 sessions de 2h
- Support technique assuré
- Documentation en français
- Livraison : Janvier 2026

---

## 8. Livrables Attendus

### 8.1 Livrables Techniques
1. ✅ Fichier Power BI (.pbix) complet
2. ✅ Documentation du modèle de données
3. ✅ Scripts Power Query (M)
4. ✅ Bibliothèque de mesures DAX
5. ✅ Configuration RLS
6. ✅ Guide de publication

### 8.2 Livrables Documentaires
1. ✅ Cahier des charges (ce document)
2. ✅ Schéma des sources de données
3. ✅ Documentation technique complète
4. ✅ Guide utilisateur par profil
5. ✅ Procédures d'actualisation
6. ✅ Plan de formation

---

## 9. Planning Prévisionnel

| Phase | Durée | Période | Statut |
|-------|-------|---------|--------|
| **Phase 1 : Analyse** | 1 semaine | 10-16 Déc 2025 | ✅ En cours |
| Phase 2 : Modélisation | 1 semaine | 17-23 Déc 2025 | 🔜 À venir |
| Phase 3 : Dashboards | 2 semaines | 24 Déc - 6 Jan 2026 | ⏳ Planifié |
| Phase 4 : Sécurité | 3 jours | 7-9 Jan 2026 | ⏳ Planifié |
| Phase 5 : Tests | 1 semaine | 10-16 Jan 2026 | ⏳ Planifié |
| Formation & Go-Live | 3 jours | 20-22 Jan 2026 | ⏳ Planifié |

---

## 10. Critères de Succès

### 10.1 Critères Fonctionnels
- ✅ Tous les dashboards sont fonctionnels
- ✅ Les KPIs s'affichent correctement
- ✅ Les filtres et interactions fonctionnent
- ✅ RLS appliqué et testé

### 10.2 Critères d'Acceptation
- ✅ Validation par la Direction
- ✅ Tests utilisateurs concluants
- ✅ Performance satisfaisante (< 3s)
- ✅ Formation dispensée
- ✅ Documentation complète

### 10.3 Indicateurs de Réussite
- Taux d'adoption > 80% après 1 mois
- Satisfaction utilisateurs > 4/5
- Réduction du temps de reporting de 50%
- Utilisation quotidienne par la Direction

---

## 11. Risques et Mitigation

| Risque | Impact | Probabilité | Mitigation |
|--------|--------|-------------|------------|
| Qualité données insuffisante | Élevé | Moyenne | Audit préalable + nettoyage ETL |
| Performances dégradées | Moyen | Faible | Optimisation modèle + agrégations |
| Résistance au changement | Élevé | Moyenne | Formation + accompagnement |
| Disponibilité sources | Élevé | Faible | Plan de backup + données test |
| Dérive du scope | Moyen | Moyenne | Validation jalons + priorisation |

---

## 12. Validation et Approbation

### 12.1 Comité de Pilotage
- **Directeur Général** : Validation stratégique
- **DSI** : Validation technique
- **Responsables métiers** : Validation fonctionnelle

### 12.2 Points de Validation
1. ✅ Cahier des charges validé
2. ⏳ Modèle de données validé
3. ⏳ Dashboards validés
4. ⏳ Recette utilisateur validée
5. ⏳ Mise en production validée

---

## 📅 Informations Document

**Version** : 1.0  
**Date de création** : 10 Décembre 2025  
**Dernière mise à jour** : 10 Décembre 2025  
**Auteur** : Équipe BI EMSI  
**Statut** : ✅ Validé

---

*École Marocaine des Sciences de l'Ingénieur - Projet Business Intelligence*
