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
<img width="1316" height="737" alt="image" src="https://github.com/user-attachments/assets/52b8047a-55cb-4238-8843-45f5359e8221" />
<img width="1312" height="739" alt="image" src="https://github.com/user-attachments/assets/38fbac31-6749-4759-9e83-1255de48b2ab" />



## Choix techniques et hypothèses
- Grain de la fact : un article commandé
- Avis agrégés par commande (moyenne) pour éviter les doublons
- Commandes annulées : Pas considéré
- Warehouse XSMALL avec AUTO_SUSPEND = 60 s
- DirectQuery pas utilisé car les données sont figées, donc ce sera plus fluide

## Limites et évolutions possibles
- Chargement incrémental (Streams et Tasks), orchestration
- Transformations avec dbt (tests automatisés)
- Ajout de la géolocalisation (distance vendeur-client)
- Utilisateur de service dédié pour Power BI
- Passage en DirectQuery ou actualisation planifiée
