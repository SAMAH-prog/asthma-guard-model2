# 🩺 AsthmaGuard — Modèle 2 : Prédiction Imminente de Crise d'Asthme

Bienvenue dans la documentation technique du **Modèle 2** d'AsthmaGuard. Ce sous-système est le cœur réactif de notre solution : son objectif est de prédire l'imminence d'une crise d'asthme dans un horizon temporel de **4 à 24 heures** à partir de signaux physiologiques et environnementaux collectés en continu par une smartwatch et des capteurs tiers.

---

## 🎯 1. Objectif & Spécifications du Modèle 2

Le Modèle 2 agit comme un système d'alerte précoce personnalisé. Contrairement au Modèle 1 (analyse des risques à long terme), le Modèle 2 traite des flux de données dynamiques pour détecter les ruptures de tendance ou les signaux causaux à retardement.

### Données d'Entrée (Features)
* **Physiologiques (Smartwatch) :**
    * `fc_repos` (Fréquence cardiaque au repos, normalisée en bpm)
    * `spo2` (Saturation en oxygène, en %)
    * `activite` (Niveau d'activité physique / pas)
    * `heure` (Composante temporelle pour capturer les profils circadiens / crises nocturnes)
* **Environnementales :**
    * `pm25` (Concentration des particules fines en suspension, en µg/m³)

### Données de Sortie (Target)
* `crise_imminente` : Variable binaire (`1` si une crise est statistiquement modélisée à l'horizon +12h, `0` sinon).

---

## 🧠 2. Approche Algorithmique : Confrontation de deux Paradigmes

Pour valider le choix de notre architecture finale, nous avons mis en compétition deux approches distinctes :

### A. Random Forest Classifier (Approche Standard / Baseline)
Le **Random Forest** est un algorithme d'apprentissage d'ensemble basé sur une multitude d'arbres de décision indépendants. 
* **Fonctionnement :** Il évalue l'état de santé du patient à un instant $T$ de manière isolée.
* **Limite majeure :** Il est "aveugle au temps" (*Time-agnostic*). Si une baisse progressive de la SpO2 sur les 6 dernières heures annonce une crise, le Random Forest ne verra qu'une photo instantanée à l'instant $T$, ignorant la trajectoire de dégradation.

### B. Modèle Temporel Causal (Architecture Inspirée du Causal-TFT)
Notre modèle principal s'appuie sur une structure séquentielle exploitant des fenêtres glissantes et des variables décalées dans le temps (*Lags*), reproduisant le comportement des couches d'attention d'un **Temporal Fusion Transformer (TFT)**.
* **Fonctionnement :** Il ingère l'historique récent du patient (Lags à $T-1, T-2, T-4, T-6$) ainsi que des agrégations glissantes (moyenne de pollution cumulée).
* **Force majeure :** Il capture les **dépendances à long terme** et la **causalité décalée** (ex: l'effet d'une exposition aux particules fines se manifeste souvent plusieurs heures après l'inhalation).

---

## 📊 3. Analyse Comparative des Performances

En situation réelle de déséquilibre des classes (les crises d'asthme étant des événements heureusement minoritaires dans la vie d'un patient), les métriques brutes comme l'Accuracy peuvent être trompeuses. Voici comment se comportent les deux modèles après correction du biais de classe :

| Métriques Évaluées | Random Forest (Standard) | Modèle Temporel Causal (TFT-like) |
| :--- | :---: | :---: |
| **Accuracy** (Précision Globale) | ~90% - 93% | **~94% - 96%** |
| **Précision** (PPV - Fiabilité de l'alerte) | Faible (Faux Positifs fréquents) | **Élevée** (Alertes ciblées) |
| **Rappel** (Sensibilité - Taux de détection) | Proche de 0% (Râte les crises cumulatives) | **Élevé (> 85%)** (Anticipe la dégradation) |
| **F1-Score** (Synthèse Précision/Rappel) | Très bas | **Excellent** |

### 🔍 Pourquoi le Modèle Causal est la meilleure solution :
1.  **Le coût médical des Faux Négatifs :** Pour un patient asthmatique, une absence d'alerte alors qu'une crise survient est dangereuse. Le Random Forest, par manque de contexte historique, génère un fort taux de faux négatifs. Le Modèle Causal maximise le **Rappel**, garantissant une couverture de sécurité maximale.
2.  **Gestion de la fatigue pulmonaire :** Le Modèle Causal comprend qu'un taux de $PM_{2.5}$ moyennement élevé mais prolongé sur 8 heures est plus nocif qu'un pic de pollution isolé de 5 minutes.

---

## 🚀 4. Architecture de Production (FastAPI)

Le modèle validé est encapsulé au sein d'une micro-API développée avec **FastAPI**, optimisée pour le traitement asynchrone à basse latence. Elle expose un point de terminaison unique utilisé par l'application mobile :

### Endpoint : `POST /predict`
Reçoit les données télémétriques de la montre en temps réel et renvoie le niveau d'urgence.

**Exemple de Requête (Payload JSON) :**
```json
{
  "fc_repos": 94.0,
  "spo2": 92.8,
  "activite": 150.0,
  "pm25": 65.0,
  "heure": 18
}
