# Prix alimentaires au Congo-Brazzaville : prédire le prix sur les marchés

Projet de **Licence 3 Big Data** (Dakar Institute of Technology), cours d'Outils statistiques pour la Data Science.
Objectif : préparer 15 ans de relevés de prix alimentaires, puis comparer trois modèles pour **prédire le prix d'un produit selon le marché et la date**.

## Données

- **Source** : Programme alimentaire mondial (WFP), publié sur [HDX](https://data.humdata.org/dataset/wfp-food-prices-for-congo).
- **Fichier** : `wfp_food_prices_cog.csv`, soit 6 414 relevés mensuels de 2011 à 2026, 13 marchés, 7 départements et 19 produits (manioc, riz, huile, poisson fumé…).
- **Variable cible** : `price`, le prix de détail en francs CFA.

## Démarche

### 1. Analyse exploratoire
- Distribution du prix : asymétrique (coefficient d'asymétrie de 3,7), avec une valeur extrême suspecte à 23 542 FCFA.
- Le prix dépend surtout du **produit** (de 432 FCFA/kg pour le manioc frais à 5 000 FCFA/kg pour le poisson salé), puis du **lieu** et de la **date**.
- Collecte irrégulière : Brazzaville représente 36,5 % des relevés, et certaines années sont presque vides.

### 2. Pré-traitement
| Étape | Résultat |
|---|---|
| Colonnes inutiles | 7 supprimées, dont `usdprice` (fuite de données) |
| Incohérences de texte | « Point-Noire » corrigé en « Pointe-Noire » |
| Doublons (date, marché, produit) | 0 |
| Valeurs manquantes cachées | 1 328 mois révélés : 443 interpolés (3 mois maximum), 885 supprimés |
| Valeurs aberrantes | 4 erreurs de saisie supprimées (prix > 5 fois la médiane de leur série) |
| Encodage | one-hot sur le département, le produit et l'unité ; année et mois extraits |
| Séparation temporelle | train 2011–2021 (6 134 lignes), test 2022–2026 (719 lignes), 29 variables |
| Standardisation | latitude, longitude, année, mois (scaler ajusté sur le train uniquement) |

### 3. Modélisation
Trois modèles entraînés sur 2011–2021 et évalués sur 2022–2026 :
- Régression linéaire
- Random Forest
- XGBoost

## Résultats

Évaluation sur le jeu de test (2022–2026, 719 relevés) :

| Modèle | R² | MSE | MAE (FCFA) |
|---|---|---|---|
| **Régression linéaire** | **−0,053** | **375 401** | **415** |
| XGBoost | −0,311 | 467 435 | 441 |
| Random Forest | −0,331 | 474 530 | 441 |

Sur les 4 produits vus à l'entraînement (riz, haricots, farine de manioc, huile végétale ; 598 relevés) :

| Modèle | R² | MSE | MAE (FCFA) |
|---|---|---|---|
| **Régression linéaire** | **0,445** | **208 367** | 284 |
| XGBoost | 0,407 | 222 677 | **283** |
| Random Forest | 0,394 | 227 464 | 286 |

**Meilleur modèle : la régression linéaire.**

### Ce que montrent les résultats
- **Surapprentissage des arbres** : Random Forest (R² = 0,994) et XGBoost (0,975) sont presque parfaits sur le passé, mais s'effondrent sur le futur. La régression linéaire, plus simple, généralise mieux.
- **Le sel explique les R² négatifs** : relevé seulement à partir de 2022, il est absent du train, et aucun modèle ne peut prédire un produit jamais vu (erreur de 197 % de son prix). Sans lui, les R² redeviennent positifs.
- **Le produit fait le prix** : 9 des 10 variables les plus importantes sont des produits (poissons et viande en tête).
- **Le riz est le mieux prédit** (MAE de 147 FCFA). L'huile végétale est sous-estimée, à cause de sa forte hausse après 2022.

## Limites et pistes
- Le test ne couvre que 5 produits sur 19, car peu de marchés sont suivis après 2021.
- Un produit absent du train (le sel) ne peut pas être prédit.
- Pistes : une séparation train/test qui garde tous les produits des deux côtés, l'ajout de la pluviométrie par département (HDX), la modélisation du logarithme du prix, et le réglage des arbres pour limiter le surapprentissage.

## Lancer le projet
1. Ouvrir `projet_prix_congo_complet.ipynb` dans Google Colab ou Jupyter.
2. Importer `wfp_food_prices_cog.csv` dans le même dossier.
3. Exécuter toutes les cellules. Dépendances : `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `xgboost`.

## Auteur
**Olivier Jonathan DIPO WANDO**, L3 Big Data, Dakar Institute of Technology, 2026.
