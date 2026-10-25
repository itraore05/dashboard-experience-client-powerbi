# 📊 Dashboard Expérience Client — Satisfaction, NPS et voix du client (Power BI)

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-155%20mesures-informational?style=flat)
![Power Query](https://img.shields.io/badge/Power%20Query-19%20requêtes-blue?style=flat)
![Modèle](https://img.shields.io/badge/Modèle-schéma%20en%20étoile-6A1B9A?style=flat)
![NLP](https://img.shields.io/badge/IA-analyse%20des%20verbatims-orange?style=flat)
![Statut](https://img.shields.io/badge/statut-livré-success?style=flat)

> Tableau de bord Power BI de **5 pages** qui mesure l'expérience client d'un **organisme public de prévoyance sociale** à partir de son enquête de satisfaction en agence : CSAT, CES, NPS, index d'expérience client, analyse des verbatims par IA et suivi des actions correctives.
> **Projet réel pour un client** (2026). Les données et le fichier Power BI sont confidentiels : ce dépôt présente la démarche, le modèle et les règles métier, sans aucune donnée.

---

## 🎯 Contexte et objectifs

La direction Qualité du client collecte des questionnaires de satisfaction dans ses agences. Il fallait transformer ces réponses en un outil de pilotage :

- suivre les **indicateurs clés de l'expérience client** par rapport à des cibles fixées par la direction ;
- comparer les **agences et les services**, en tenant compte de la fiabilité statistique ;
- exploiter les **commentaires libres** (verbatims) enrichis par un modèle d'IA : sentiment, thème, mots-clés ;
- relier les constats à un **plan d'actions correctives** suivi dans le temps.

Point de départ : une maquette HTML avec des données fictives. Le dashboard reprend sa structure et la branche sur les vraies données.

## 🧭 Architecture

```
Collecte (questionnaire)  ──▶  Data mart Excel (15 onglets)  ──▶  Power Query (19 requêtes)  ──▶  Modèle en étoile  ──▶  Rapport Power BI (5 pages)
                                        ▲
                     3 onglets saisis par les métiers : cibles, thèmes, actions
```

- **Source unique** : un data mart Excel. Les données ne circulent que dans un sens : on modifie l'Excel, puis on actualise le rapport.
- **Onglets métiers** : cibles des KPI, référentiel des thèmes et registre des actions sont des tableaux Excel nommés, complétés par les équipes, sans toucher au rapport.
- **Corrections dans Power Query**, jamais dans la source : colonnes mal nommées, libellés à casse variable, calendrier traduit en français, conversion des réponses texte en notes de 1 à 5.

## 🗂️ Modèle de données

| Élément | Volume |
|---|---|
| Tables | 22 |
| Relations | 12 (schéma en étoile autour de la table des réponses) |
| Mesures DAX | 155, rangées en 10 dossiers |
| Requêtes Power Query | 19 |

- **Table de faits** : une ligne par répondant (réponses, indicateurs 0/1, NPS, verbatim enrichi par l'IA).
- **Dimensions** : agence (avec coordonnées pour la carte), service, sexe, tranche d'âge, calendrier, fréquence de visite, niveaux de réponse.
- **Relation plusieurs-à-plusieurs** gérée par une table de liaison : un répondant peut citer plusieurs services.
- **Table des mots-clés** dérivée en Power Query : la liste produite par l'IA est éclatée en une ligne par mot.

## 📐 Indicateurs et règles métier

| Indicateur | Définition |
|---|---|
| **CSAT** | (Très satisfaits + Satisfaits) ÷ répondants |
| **CES** | (Très facile + Facile) ÷ répondants |
| **NPS** | (Promoteurs − Détracteurs) ÷ avis exprimés × 100, sur une échelle à 5 niveaux |
| **IEC** (index d'expérience client) | 0,4 × CSAT + 0,3 × CES + 0,3 × NPS normalisé |
| **Accueil / Attente** | Part des réponses positives sur la qualité de l'accueil et le temps d'attente |

```DAX
Nb Répondants = COUNTROWS(F_REPONSES)

CSAT % = DIVIDE(SUM(F_REPONSES[is_csat_satisfait]), [Nb Répondants])

NPS = DIVIDE([Nb Promoteurs] - [Nb Détracteurs], [Nb Avis NPS]) * 100

NPS Normalisé = IF(ISBLANK([NPS]), BLANK(), ([NPS] + 100) / 2)

IEC = IF(ISBLANK([NPS Normalisé]), BLANK(),
    0.40 * [CSAT %] * 100 + 0.30 * [CES %] * 100 + 0.30 * [NPS Normalisé])

Cible CSAT = LOOKUPVALUE(PARAM_CIBLES[cible], PARAM_CIBLES[code_kpi], "CSAT")

Couleur CSAT =
VAR v = [CSAT %] * 100
VAR c = [Cible CSAT]
RETURN IF(v >= c, "#098D39", IF(v >= c * 0.85, "#DE7C0F", "#D13438"))
```

**Fiabilité statistique intégrée au rapport :**

| Effectif n | Niveau | Traitement |
|---|---|---|
| n < 3 | Insuffisant | Barre grise, exclu des classements et des alertes |
| 3 à 9 | Directionnel | Tendance seulement |
| 10 à 29 | Acceptable | Interprétable avec prudence |
| 30 et plus | Fiable | Interprétable |

Marge d'erreur à 95 % affichée en info-bulle : ± 1,96 × √(p × (1 − p) ÷ n).

## 🧩 Les 5 pages du rapport

| Page | Public | Contenu |
|---|---|---|
| 1. Vue synthétique | Direction | Cartes KPI colorées selon la cible, répartition NPS, CSAT par agence, **alertes automatiques** |
| 2. Parcours et opérations | Responsables d'agence | Matrice accueil × attente, heatmap agence × service, drivers, carte du réseau, évolution dans le temps |
| 3. Segmentation client | Statistiques | CSAT et NPS par âge, profil et fréquence de visite, Top 3 / Flop 3 des agences, écart hommes / femmes |
| 4. Voix du client | Qualité | Sentiment, mots-clés, thèmes dominants, **plan d'action à priorité calculée**, suivi des actions avec retard automatique |
| 5. Analytique avancée | Statistiques | **Corrélations de Pearson** (drivers de la satisfaction), segmentation comportementale, fiabilité par agence, **détection d'anomalies** |

Cadre commun : bandeau avec période et effectif dynamiques, navigation par boutons, panneau de filtres synchronisés sur les 5 pages, thème Power BI personnalisé (JSON).

## 💡 Compétences démontrées

- Conception d'un **modèle en étoile** propre à partir d'une source imparfaite
- **DAX avancé** : 155 mesures organisées, textes dynamiques, mise en forme conditionnelle, corrélations, détection d'anomalies
- **Power Query** : nettoyage, normalisation des libellés, conversion texte → score, éclatement de listes
- Exploitation de **verbatims enrichis par l'IA** (sentiment, thème, mots-clés) dans un outil de décision
- Prise en compte de la **fiabilité statistique** sur de petits échantillons
- **Documentation technique complète** (17 pages) pour que le client puisse reprendre et maintenir le rapport seul

## 🔒 Confidentialité

Ce projet a été réalisé pour un client. Le fichier `.pbix`, le data mart, les résultats chiffrés, le nom du client et sa charte graphique ne sont pas publiés. Seules la démarche et la méthode sont présentées ici.

## 👤 Auteur

**Ismaël Traoré** — Développeur Data & IA
📧 contact.traoreimael@gmail.com · 💼 [LinkedIn](https://www.linkedin.com/in/ismael-traor%C3%A9-m-4007a2225/)
