# Olist : Pipeline Snowflake et dashboard Power BI

## Contexte et problématique
Comment la qualité de livraison impacte-t-elle la satisfaction client et la performance
commerciale d'une marketplace e-commerce ?

## Données
Dataset public Olist (Kaggle), ~100 000 commandes, 2016 à 2018.
Ce qui peut être trompeur : 2016 et 2018 sont des années incomplètes, à l'inverse de 2017.

## Architecture
<img width="1890" height="723" alt="image" src="https://github.com/user-attachments/assets/fc1f8d23-6544-4fc1-be20-20c1630350b7" />

Kaggle (CSV Olist) → Snowflake [Stage/Upload] → RAW (8 tables) → MART (1 fact + 3 dims) → Power BI (modèle + 2 pages)



## Étapes réalisées
1. Chargement des CSV dans Snowflake (couche RAW, 8 tables, tout en VARCHAR)
2. Transformation SQL (couche MART : FACT_ORDERS + 3 dimensions)
3. Contrôles qualité (comptages, unicité, intégrité référentielle)
4. Rôle dédié lecture seule pour Power BI
5. Modèle en étoile et mesures DAX dans Power BI
6. Dashboard de 2 pages

## Résultats clés
- Taux de retard global : 7,91 %
- Note moyenne : 2,55 en retard vs 4,21 à l'heure
- État avec le plus fort taux de retard : AL (modifier pour avoir les noms complets

## Captures
<img width="1371" height="767" alt="image" src="https://github.com/user-attachments/assets/248ed850-e7dc-40cf-be11-683ab1891a59" />
<img width="1395" height="830" alt="image" src="https://github.com/user-attachments/assets/b823e8fc-833c-4cff-941b-7699746acd65" />



## Choix techniques et hypothèses
- Grain de la fact : un article commandé
- Avis agrégés par commande (moyenne) pour éviter les doublons
- Commandes annulées : [ta décision]

## Limites et évolutions possibles
- Chargement incrémental (Streams et Tasks), orchestration
- Transformations avec dbt (tests automatisés)
- Ajout de la géolocalisation (distance vendeur-client)
- Utilisateur de service dédié pour Power BI
- Passage en DirectQuery ou actualisation planifiée
