
# Prévision de la Demande Électrique et Dimensionnement de Groupe Électrogène

Ce projet implémente un pipeline complet d'analyse et de prévision de séries temporelles de consommation électrique résidentielle (données réelles mesurées minute par minute sur près de 4 ans). L'objectif d'ingénierie final est de traduire ces prévisions de consommation en une recommandation optimale de dimensionnement pour un groupe électrogène de secours (Genset) en appliquant les normes internationales **ISO 8528-1** et **IEC 60364**.

##  Présentation du Projet

Dans le secteur de l'énergie, le dimensionnement d'un groupe électrogène est un défi d'optimisation critique :
* **Surdimensionner** entraîne un gaspillage financier important et risque d'endommager le moteur par encrassement (phénomène de *wet stacking* dû à un fonctionnement à vide).
* **Sousdimensionner** fait peser un risque majeur de coupure généralisée lors des pics de consommation.

Ce projet résout ce problème à l'aide d'une approche rigoureuse basée sur les données et le Machine Learning.

---

##  Stack Technique

* **Langage :** !!!!! PYTHON VERSION: 3.12.6
* **Analyse de données & Preprocessing :** Pandas, NumPy
* **Modélisation Statistique & Classique :** Statsmodels (ETS, SARIMA, SARIMAX)
* **Modélisation Avancée :** Prophet (Meta), XGBoost
* **Évaluation :** Scikit-Learn (RMSE, MAE, MAPE)
* **Visualisation :** Matplotlib, Seaborn

---

##  Étapes du Pipeline

### 1. Analyse Exploratoire & Diagnostics (EDA)
* Analyse de la stationnarité à l'aide des tests statistiques de Dickey-Fuller Augmenté (**ADF**) et **KPSS**.
* Décomposition saisonnière (**STL**) pour isoler la tendance, la saisonnalité journalière et hebdomadaire, ainsi que les résidus.
* Étude de l'autocorrélation via les tracés **ACF** et **PACF**.

### 2. Modélisation de Référence (Baselines)
* Entraînement d'un modèle d'auto-lissage exponentiel (**ETS**) avec saisonnalité additive.
* Entraînement d'un modèle **SARIMA(1,0,1)(1,0,1)24** pour capturer la dynamique autorégressive et la saisonnalité quotidienne.
* Analyse des résidus (QQ-Plot et ACF) pour mettre en évidence les limites des modèles statistiques classiques (incapacité à modéliser efficacement la saisonnalité hebdomadaire sans coût de calcul prohibitif).

### 3. Modélisation Avancée & Variables Exogènes
* Intégration de variables de calendrier (jours de la semaine, week-ends) et de température synthétique (modélisation de cycles annuels et quotidiens) dans **SARIMAX** et **Prophet**.
* Évaluation de l'impact des variables exogènes sur la précision des prévisions.

### 4. Modélisation non-linéaire avec XGBoost
* Feature engineering poussé (création de lags à t-1, t-24, t-168, statistiques glissantes sur 24 heures, encodages cycliques sinus/cosinus des heures et jours).
* Entraînement d'un régresseur **XGBoost** pour capturer les interactions non-linéaires complexes.

---

##  Résultats et Comparaison des Modèles

Les modèles ont été évalués de manière équitable sur une fenêtre de test stricte de 7 jours (168 heures).

### Impact de l'ajout de la Température (Météo)

L'ajout de données de température (simulées) a montré une légère dégradation des métriques classiques dues au bruit et à la non-linéarité thermique (courbe de consommation en "U" pour le chauffage/climatisation) :

| Modèle | RMSE | MAE | MAPE |
| :--- | :---: | :---: | :---: |
| **SARIMAX (calendrier seul)** | 0.7101 | 0.5433 | 78.44 % |
| **SARIMAX + Température** | 0.7125 | 0.5459 | 79.27 % |
| **Prophet (calendrier seul)** | 0.7088 | 0.5699 | 86.04 % |
| **Prophet + Température** | 0.7562 | 0.6259 | 104.30 % |

---

##  Rapport de Dimensionnement du Groupe Électrogène

En utilisant les prévisions de notre meilleur modèle de prédiction des pics (XGBoost), nous appliquons la méthodologie d'ingénierie :

1. **Conversion en Puissance Apparente (kVA) :** Application d'un facteur de puissance de `0.8` (norme ISO 8528-1).
2. **Marge de sécurité :** Application d'un coefficient multiplicateur de `1.25` (marge de 25 % selon IEC 60364) pour absorber les courants d'appel au démarrage des moteurs.
3. **Sélection de matériel :** Arrondi à la taille supérieure standard du marché.

### Rapport d'Ingénierie Généré :
```text
=== GENERATOR SET SIZING REPORT ===
Predicted peak:       3.08 kW
Predicted average:    1.23 kW
Required apparent:    3.85 kVA  (PF=0.8, ISO 8528-1)
With 25% margin:      4.81 kVA
Recommended genset:   5 kVA  (next standard size)
Load factor:          39.96%  [HEALTHY]

Justification: Peak demand of 3.08 kW converted to 3.85 kVA
at PF=0.8 per ISO 8528-1. 25% safety margin per IEC 60364 yields 4.81 kVA.
Nearest standard size: 5 kVA.
```

### Validation du Facteur de Charge (*Load Factor*)
Le facteur de charge obtenu est de **39.96%**. 
* Il se situe bien dans la zone saine (**[HEALTHY]** définie entre 30 % et 80 %).
* Évite le risque de **wet stacking** (encrassement du moteur par imbrûlés sous 30 % de charge).
* Conserve une marge suffisante pour tolérer d'éventuelles hausses de charges transitoires (limite supérieure de 80 %).

---

## ⚙️ Comment lancer le projet

1. Clonez ce dépôt.

!!!!! PYTHON VERSION: 3.12.6

2. Installez les dépendances :
   ```bash
   pip install -r requirements.txt
   ```
3. Téléchargez le jeu de données `Individual Household Electric Power Consumption` depuis l'UCI Machine Learning Repository et placez-le dans le dossier racine.
4. Lancez le notebook Jupyter :
   ```bash
   jupyter notebook
   ```
5. Exécutez l'ensemble des cellules pour reproduire l'analyse et générer le rapport final.