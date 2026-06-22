# AsthmaGuard 🫁⌚

[cite_start]**AsthmaGuard** est un système prédictif et personnalisé des crises d'asthme chez l'adulte (18-60 ans) s'appuyant sur l'intelligence artificielle, les données physiologiques issues de montres connectées et des API environnementales[cite: 1, 20, 21, 22]. [cite_start]Ce projet propose de passer d'une gestion réactive traditionnelle de l'asthme à une approche proactive et préventive[cite: 18].

## 📌 Table des matières
- [Architecture Générale du Système](#-architecture-générale-du-système)
- [Focus sur les Algorithmes Principaux](#-focus-sur-les-algorithmes-principaux)
  - [Modèle 1 : Risque à long terme (XGBoost)](#modèle-1--risque-à-long-terme-xgboost)
  - [Modèle 2 : Prédiction de crise imminente (Causal-TFT)](#modèle-2--prédiction-de-crise-imminente-causal-tft)
- [Comparaison et Justification Algorithmique](#-comparaison-et-justification-algorithmique)
- [Stratégie de Données : Dataset Synthétique](#-stratégie-de-données--dataset-synthétique)
- [Règles Métier & Validation Physiologique](#-règles-métier--validation-physologique)
- [Compatibilité Matérielle (Smartwatches)](#-compatibilité-matérielle-smartwatches)
- [Calendrier du Projet](#-calendrier-du-projet)

---

## 🏗️ Architecture Générale du Système

[cite_start]Le cœur d'AsthmaGuard repose sur une **architecture à trois modules complémentaires**, chacun répondant à un horizon temporel et un objectif précis[cite: 23, 25]:

| Module | Horizon Temporel | Objectif Principal | Algorithme Retenu | Nature du Modèle |
| :--- | :--- | :--- | :--- | :--- |
| **Modèle 1** | 5 à 15 jours | Évaluation du risque de fond (long terme) | **XGBoost** | [cite_start]Classification [cite: 25] |
| **Modèle 2** | 4 à 24 heures | Prédiction de crise imminente (court terme) | **Causal-TFT** | [cite_start]Prédiction de séries [cite: 25] |
| **Modèle 3** | Temps réel | Évaluation du risque environnemental extérieur | **Règles floues** | [cite_start]Classification [cite: 25] |

---

## 🧠 Focus sur les Algorithmes Principaux

### Modèle 1 : Risque à long terme (XGBoost)
* [cite_start]**Objectif** : Prédire la probabilité de survenue d'une crise sous 15 jours afin d'adapter le traitement de fond[cite: 25, 32].
* [cite_start]**Entrées clés** : Profil médical du patient (âge, sexe, IMC, sévérité GINA, score ACT), historique des crises sur 12 mois, usage des bronchodilatateurs (moyenne 7j) et prévisions environnementales moyennes à 7 jours[cite: 30].
* [cite_start]**Pourquoi XGBoost ?** Cet algorithme d'Extreme Gradient Boosting offre un excellent compromis entre performance (Score AUC de 0.85-0.90) et vitesse d'entraînement[cite: 47, 50]. [cite_start]De plus, l'intégration de la méthode **SHAP (SHapley Additive exPlanations)** garantit une explicabilité complète en isolant le top 3 des facteurs contribuant au risque[cite: 32, 47, 50].

### Modèle 2 : Prédiction de crise imminente (Causal-TFT)
* [cite_start]**Objectif** : Identifier les fenêtres critiques de 4 à 24 heures avant une crise à partir de signaux faibles[cite: 25, 38].
* [cite_start]**Entrées clés** : Séries temporelles physiologiques issues de la smartwatch (Fréquence Cardiaque au repos sur 1h, Variabilité de la FC via RMSSD, taux de SpO2, réveils nocturnes, activité physique, stress) combinées à la pollution immédiate (PM2.5 sur un historique glissant de 6h)[cite: 36].
* [cite_start]**Pourquoi le Causal-TFT ?** Le *Temporal Fusion Transformer* est nativement conçu pour traiter des séries temporelles multivariées complexes[cite: 50]. [cite_start]Sa composante **causale** permet de modéliser les relations de cause à effet à retardement (ex: une pollution accumulée couplée à une dégradation progressive de la SpO2 qui déclenche une crise 12h plus tard)[cite: 50]. [cite_start]Il surpasse les architectures LSTM standards en fournissant également des intervalles de confiance[cite: 50].

---

## 📊 Comparaison et Justification Algorithmique

[cite_start]Voici l'analyse comparative effectuée lors de la phase de cadrage d'AsthmaGuard pour valider la sélection de nos modèles[cite: 46]:

| Algorithme Candidat | Module Visé | Score AUC | Temps d'entraînement | Interprétabilité | Verdict |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **XGBoost** | **Modèle 1** | **0.85 - 0.90** | **Moyen (minutes)** | **Élevée (SHAP)** | [cite_start]**Retenu ✅** [cite: 47, 50] |
| Random Forest | Modèle 1 | 0.82 - 0.87 | Moyen | Élevée | [cite_start]Écarté (XGBoost gère mieux le déséquilibre de classe) [cite: 47, 50] |
| **Causal-TFT** | **Modèle 2** | **0.88 - 0.92** | **Élevé (heures)** | **Moyenne (Attention)** | [cite_start]**Retenu ✅** [cite: 47, 50] |
| LSTM Standard | Modèle 2 | 0.82 - 0.86 | Élevé | Faible (Boîte noire) | [cite_start]Écarté (Moins performant sur le long terme) [cite: 47, 50] |

### Synthèse des justifications :
1. [cite_start]**XGBoost (Modèle 1)** : Sélectionné car il s'agit d'une référence dans la littérature médicale pour sa robustesse face aux classes fortement déséquilibrées (les crises restent des événements heureusement rares dans la vie d'un patient)[cite: 50].
2. [cite_start]**Causal-TFT (Modèle 2)** : Choisi pour sa capacité à interpréter le contexte global et passé du patient à travers des mécanismes d'attention temporelle, là où les modèles classiques ne capturent que des instantanés[cite: 50].

---

## 📉 Stratégie de Données : Dataset Synthétique

[cite_start]En phase initiale du projet, face à l'absence de données réelles étiquetées, l'équipe a développé un pipeline de **génération de données synthétiques** réalistes afin de pré-entraîner et tester les modèles[cite: 95, 96].

[cite_start]Le processus respecte les étapes de validation clinique suivantes[cite: 100]:
1. [cite_start]**Distributions physiologiques réelles** : Utilisation de lois normales (ex: SpO2 centrée sur $96 \pm 2\%$) et de lois de Poisson ou Gamma pour simuler les variables[cite: 111, 127].
2. [cite_start]**Règles probabilistes médicales** : Injection de scores de risques pour corréler logiquement les crises à une baisse de SpO2, une hausse de la pollution ou une augmentation de la fréquence cardiaque[cite: 113, 115, 117, 124].
3. [cite_start]**Réisme terrain** : Ajout de bruit gaussien (imprécision des capteurs), de *label flipping* (erreurs d'annotation de l'ordre de 5%), et application de techniques de déséquilibre (rareté des crises)[cite: 98, 100].

---

## 🚨 Règles Métier & Validation Physiologique

[cite_start]Pour éviter les fausses alertes liées à d'autres facteurs (effort sportif, stress ponctuel), AsthmaGuard intègre un **Score de Tachycardie Asthmatique**[cite: 60, 150]. [cite_start]Un total de points est attribué selon les conditions captées par la montre connectée[cite: 59, 150]:

* [cite_start]**SpO2 < 94%** : $+3$ points (Alerte hypoxémie forte) [cite: 59, 150]
* [cite_start]**FC > seuil personnalisé (+20% baseline pendant 30min)** : $+2$ points (Tachycardie significative) [cite: 59, 150]
* [cite_start]**Activité physique faible (<500 pas / 30min)** : $+1$ point (Exclusion d'une cause sportive) [cite: 59, 150]
* [cite_start]**Réveil nocturne / Pollution élevée / Stress** : $+1$ point chacun [cite: 59, 150]

**Seuils de décision finaux** :
* [cite_start]**Score $\ge$ 5 ou 6** 🔴 : Alerte ROUGE - Risque imminent très élevé[cite: 54, 150].
* [cite_start]**Score 3 - 5** 🟠 / 🟡 : Alerte Orange/Jaune - Surveillance renforcée et prudence[cite: 55, 56, 151].
* [cite_start]**Score $\le$ 2** 🟢 : Alerte Verte - Pas d'alerte asthme (origine probablement bénigne)[cite: 57, 151].

---

## ⌚ Compatibilité Matérielle (Smartwatches)

[cite_start]L'architecture logicielle d'AsthmaGuard est agnostique et conçue pour s'interfacer avec n'importe quel constructeur fournissant un accès continu aux capteurs de base via API[cite: 144]:
* [cite_start]**Huawei Watch** (Séries 3, 4, GT, Ultimate) — API *Huawei Health Kit* (Retenu pour ses recherches actives sur l'asthme)[cite: 133, 134, 142].
* [cite_start]**Apple Watch** (Series 6 à 9, Ultra) — API *HealthKit* (Précision de référence sur le marché)[cite: 135, 136].
* [cite_start]**Samsung Galaxy Watch** & **Fitbit** — Entièrement compatibles (Samsung Health Stack / Web API)[cite: 138, 139, 140, 141].

---

## 📅 Calendrier du Projet (Jalons Clés)

[cite_start]Le développement s'est articulé sur un cycle intensif du **5 mai au 15 juin 2026**[cite: 4, 5, 155]:
- [cite_start]**Jalons 1 (7 jours)** : Implémentation du Module Environnemental 3 et pipeline synthétique[cite: 155].
- [cite_start]**Jalons 2 (10 jours)** : Modélisation et intégration du Module 1 (XGBoost + SHAP)[cite: 155].
- [cite_start]**Jalons 3 (9 jours)** : Implémentation de la structure temporelle avancée du Module 2 (Causal-TFT)[cite: 155].
- [cite_start]**Jalons 4 (12 jours)** : Intégration du feedback patient, tests unitaires, routage de l'API et rédaction finale[cite: 155].

---
[cite_start]**Auteur :** SAMAH ZAHRAOUI [cite: 3]  
[cite_start]**Statut du Projet :** Cadrage technique validé — Prêt pour l'intégration de production (FastAPI)[cite: 12].
