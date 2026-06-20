# 🌬️ AsthmaGuard - Modèle de Prédiction de Crises d'Asthme (Court Terme)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SAMAH-prog/asthma-guard-model2/blob/main/Untitled.ipynb)

## 📝 Description du Projet

**Modèle de Prédiction de Crises d'Asthme (Court Terme)** est un modèle d'intelligence artificielle conçu pour prédire le risque de crise d'asthme à court terme (dans les prochaines 24 heures). Basé sur un ensemble de données synthétiques, il utilise un algorithme de **Random Forest** pour analyser des indicateurs physiologiques, environnementaux et comportementaux.

Ce projet vise à démontrer la faisabilité d'un système d'alerte précoce pour les patients asthmatiques, permettant une intervention proactive et une meilleure gestion de la maladie.

### 🎯 Objectifs
- Développer un modèle de prédiction fiable (Accuracy > 92%) pour les crises d'asthme.
- Fournir une **API REST** pour intégrer facilement le modèle dans des applications de santé (IoT, applications mobiles, etc.).
- Définir des **niveaux d'urgence** (Vert, Jaune, Orange, Rouge) pour guider les décisions des patients et des professionnels de santé.

---

## 🧠 Le Modèle

### Caractéristiques (Features)
Le modèle utilise 10 variables clés pour effectuer sa prédiction :

| Feature | Description | Unité / Type |
| :--- | :--- | :--- |
| `fc_repos_mean_1h` | Fréquence cardiaque moyenne au repos (1 heure) | bpm |
| `vfc_rmssd` | Variabilité de la fréquence cardiaque (RMSSD) | ms |
| `spo2` | Niveau de saturation en oxygène dans le sang | % |
| `reveils_nocturnes` | Nombre de réveils nocturnes | Compteur |
| `activite_30min` | Niveau d'activité physique (30 minutes) | Pas / Métrique |
| `stress` | Niveau de stress perçu | Score / 100 |
| `pollution_pm25_6h` | Taux de pollution (PM2.5) sur les 6 dernières heures | µg/m³ |
| `broncho_24h` | Nombre d'utilisations de bronchodilatateurs (24h) | Compteur |
| `is_night` | Indicateur de période nocturne | Booléen |
| `temp_ext` | Température extérieure | °C |

### Algorithme
Le modèle est entraîné avec un **Random Forest Classifier**, un ensemble d'arbres de décision robuste et performant.
- **Nombre d'arbres (n_estimators)**: 400
- **Profondeur maximale (max_depth)**: 18

---

## 📊 Résultats et Performances

Le modèle a démontré d'excellentes performances, dépassant l'objectif initial.

| Métrique | Score |
| :--- | :--- |
| **Accuracy** | **93.61%** ✅ |
| **AUC (Area Under the Curve)** | **97.92%** |
| **Precision (Classe 1: Crise)** | 96% |
| **Recall (Classe 1: Crise)** | 94% |
| **F1-Score (Classe 1: Crise)** | 95% |

### Matrice de Confusion
La matrice de confusion montre une excellente capacité à distinguer les situations de crise de celles sans crise.
![Matrice de Confusion](https://github.com/SAMAH-prog/asthma-guard-model2/blob/main/images/confusion_matrix.png?raw=true)

### Importance des Caractéristiques
Les facteurs les plus importants pour la prédiction sont **l'activité physique**, **la saturation en oxygène (SpO2)** et **la fréquence cardiaque au repos**, ce qui est cohérent avec la physiologie de l'asthme.
![Importance des Features](https://github.com/SAMAH-prog/asthma-guard-model2/blob/main/images/feature_importance.png?raw=true)

---

## 🚀 Installation et Utilisation

### Prérequis
- Python 3.9+
- Un compte [ngrok](https://ngrok.com/) (pour l'exposition de l'API)

### Installation des Dépendances
```bash
pip install fastapi uvicorn nest-asyncio pyngrok pandas scikit-learn matplotlib seaborn joblib
