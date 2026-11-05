# Segmentation clients RFM – Olist (e-commerce)

> Regrouper **94 989 clients** d'une marketplace brésilienne en segments utiles pour le marketing, puis déterminer **tous les combien recalculer la segmentation**.

![Python](https://img.shields.io/badge/Python-3-blue) ![scikit--learn](https://img.shields.io/badge/scikit--learn-clustering-orange) ![pandas](https://img.shields.io/badge/pandas-data%20prep-150458)

## Le problème

Olist, une marketplace brésilienne, voulait mieux cibler ses campagnes marketing. Deux questions :
1. **Quels types de clients avons-nous ?** Des segments compréhensibles et exploitables par l'équipe marketing.
2. **Combien de temps une segmentation reste-t-elle valable ?** Pour proposer un contrat de maintenance.

## Résultats clés

| | |
|---|---|
| Clients segmentés | **94 989** (9 tables relationnelles fusionnées) |
| Constat principal | **97 %** des clients n'ont commandé qu'une seule fois |
| Modèle retenu | **K-Means à 5 clusters** sur la récence, la fréquence, le montant et la note moyenne |
| Stabilité à l'initialisation | **ARI de 0,995** en moyenne sur 10 initialisations |
| Fréquence de mise à jour recommandée | **toutes les 6 semaines** |

### Les 5 segments

| Segment | Part | Profil moyen | Action marketing proposée |
|---|---:|---|---|
| **Petits paniers récents** | 44 % | 1 commande il y a 4 mois, 133 réals | Offre sur la 2e commande |
| **Petits paniers anciens** | 34 % | 1 commande il y a plus d'un an, 134 réals | Réactivation à faible coût |
| **Clients mécontents** | 17 % | Note moyenne de 1,65 sur 5 | Contact du service client |
| **Clients fidèles** | 3 % | 2,1 commandes, 289 réals au total | Programme de fidélité |
| **Gros paniers** | 2 % | 1 commande de 1 250 réals | Relance personnalisée |

**Enseignement principal :** l'enjeu pour Olist n'est pas d'attirer de nouveaux clients, mais de **les faire revenir**, en commençant par les 17 % de clients mécontents.

## Démarche

### 1. Exploration et préparation – [notebook 1](ER-REGYBY_Khadija_1_notebook_exploration_092022.ipynb)

Fusion des 9 fichiers Olist (clients, commandes, produits, paiements, avis, vendeurs, géolocalisation), traitement des doublons, des valeurs manquantes et des dates. Les commandes annulées et indisponibles sont exclues. Le notebook produit `data/rfm.csv` (une ligne par client) et `data/data_commandes.csv` (une ligne par commande).

### 2. Modélisation et choix du modèle – [notebook 2](ER-REGYBY_Khadija_2_notebook_essais_092022.ipynb)

Une première approche par **scores RFM** classiques sert de point de comparaison. Puis trois algorithmes et quatre jeux de variables sont comparés :

| Modèle | Variables | Clusters | Silhouette | Plus gros cluster | Temps |
|---|---|---:|---:|---:|---:|
| K-Means 1 | R, F, M | 4 | 0,487 | 54,3 % | 0,4 s |
| **K-Means 2 (retenu)** | **R, F, M + note** | **5** | **0,414** | **44,2 %** | **0,4 s** |
| K-Means 3 | R, F, M + note + produits | 6 | 0,381 | 43,5 % | 3,5 s |
| K-Means 4 | log(R, F, M) | 4 | 0,394 | 44,3 % | 2,9 s |
| CAH (Ward) | R, F, M (20 000 clients) | 4 | 0,451 | 54,8 % | 63,4 s |

- **DBSCAN** est écarté : il réunit 94 % des clients dans un seul cluster et classe en bruit les meilleurs clients.
- **La CAH** est écartée : trop lente pour une segmentation à mettre à jour régulièrement.
- **L'essai 1** a la meilleure silhouette, mais un cluster réunit 54 % des clients.
- **L'essai 2** est retenu : aussi équilibré que l'essai 4, avec une meilleure silhouette, et il fait apparaître le segment des **clients mécontents**.

### 3. Fréquence de mise à jour – [notebook 3](ER-REGYBY_Khadija_3_notebook_similation_092022.ipynb)

*Présentation des résultats : [ER-REGYBY_Khadija_4_Présentation_092022.pptx](ER-REGYBY_Khadija_4_Présentation_092022.pptx)*

Un modèle est figé à une date donnée, puis comparé chaque semaine à un modèle réentraîné sur les données à jour, avec l'**Adjusted Rand Index** (ARI). Une mise à jour est nécessaire quand l'ARI passe sous 0,8.

| Simulation | ARI au-dessus de 0,8 pendant |
|---|---|
| Méthode 1 : en avançant depuis le 1er janvier 2018 | 7 semaines |
| Méthode 2 : en remontant depuis le 3 septembre 2018 | 13 semaines |
| Vérification : en avançant depuis le 4 juin 2018 | 6 semaines |

Les deux simulations qui avancent dans le temps, comme le ferait Olist en pratique, donnent **6 à 7 semaines**. La méthode 2 applique un modèle récent à des clients du passé, ce qui ne correspond pas à l'usage réel. **Recommandation : réentraîner la segmentation toutes les 6 semaines.**

## Stack

Python · pandas · NumPy · scikit-learn (K-Means, DBSCAN, CAH, silhouette, ARI) · SciPy · Matplotlib · Seaborn · GeoPandas · Yellowbrick

## Lancer le projet

```bash
git clone https://github.com/khadijaerregyby/segmentation-clients-olist.git
cd segmentation-clients-olist
pip install pandas numpy scikit-learn scipy matplotlib seaborn geopandas shapely yellowbrick jupyter
mkdir data images
```

Les données ne sont pas incluses dans le dépôt. Téléchargez le [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) sur Kaggle, placez les 9 fichiers CSV dans le dossier `data/`, puis exécutez les notebooks dans l'ordre 1 → 2 → 3. Le notebook 3 prend environ 35 minutes, à cause des simulations semaine par semaine.

## Limites et perspectives

- Les segments sont à **valider avec l'équipe marketing**, puis à tester par des campagnes A/B.
- Le seuil de 0,8 est une **convention** : un seuil plus strict imposerait des mises à jour plus fréquentes.
- La simulation couvre **8 mois** : plus d'historique permettrait de mesurer la saisonnalité (Black Friday).
- Pistes : automatiser le réentraînement, analyser les commentaires des clients mécontents, ajouter les catégories achetées et les délais de livraison.

---

Projet réalisé en 09/2022 dans le cadre du parcours Data Scientist d'OpenClassrooms, entièrement mis à jour en 11/2026 · [Khadija ER-REGYBY](https://github.com/khadijaerregyby)
