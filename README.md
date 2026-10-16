# snowflake-powerbi-olist-
Projet snowflake powerbi avec la database olist
Kaggle (CSV Olist) → Snowflake [Stage/Upload] → RAW (8 tables) → MART (1 fact + 3 dims) → Power BI (modèle + 2 pages)

# Olist : Pipeline Snowflake et dashboard Power BI

## Contexte et problématique
Comment la qualité de livraison impacte-t-elle la satisfaction client et la performance
commerciale d'une marketplace e-commerce ?

## Données
Dataset public Olist (Kaggle), ~100 000 commandes, 2016 à 2018.

## Architecture
<img width="1890" height="723" alt="image" src="https://github.com/user-attachments/assets/e80970fb-29bc-42e9-b201-142febb497cc" />


## Étapes réalisées
1. Chargement des CSV dans Snowflake (couche RAW, 8 tables, tout en VARCHAR)
2. Transformation SQL (couche MART : FACT_ORDERS + 3 dimensions)
3. Contrôles qualité (comptages, unicité, intégrité référentielle)
4. Rôle dédié lecture seule pour Power BI
5. Modèle en étoile et mesures DAX dans Power BI
6. Dashboard de 2 pages

## Résultats clés
- (Ex.) Taux de retard global : X %
- (Ex.) Note moyenne : X en retard vs Y à l'heure
- (Ex.) État avec le plus fort taux de retard : ...

## Captures
![Page 1](images/page1.png)
![Page 2](images/page2.png)

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



Ordres de grandeur attendus : ORDERS ≈ 99 441, ORDER_ITEMS ≈ 112 650, CUSTOMERS ≈ 99 441, PRODUCTS ≈ 32 951, SELLERS ≈ 3 095, ORDER_REVIEWS ≈ 99 224 (peut varier selon les retours à la ligne), ORDER_PAYMENTS ≈ 103 886, CATEGORY_TRANSLATION ≈ 71.

Pense à noter ces chiffres pour ton README.

On l’ignore pour une raison de rapport effort/bénéfice dans un projet de 2 jours, pas parce qu’il serait inutile.

Pourquoi il est gênant :

Il n’a pas de clé unique. Il contient une ligne par point de coordonnées GPS, avec environ 1 million de lignes pour seulement ~19 000 préfixes de code postal distincts. Un même zip_code_prefix apparaît des dizaines de fois avec des latitudes/longitudes légèrement différentes.
Il ne se joint pas proprement. Pour le relier aux clients ou aux vendeurs (via customer_zip_code_prefix / seller_zip_code_prefix), il faudrait d’abord le dédoublonner, par exemple en faisant la moyenne des coordonnées par préfixe. Sinon, la jointure multiplie tes lignes et fausse tous tes chiffres (CA gonflé, comptages doublés). C’est l’erreur classique.
Il ajoute du travail pour un gain limité. Il demande une étape de transformation en plus, plus des contrôles qualité, pour un bénéfice qui se limite à des cartes plus fines.

Ce qui te permet de t’en passer : Power BI sait déjà placer les États brésiliens (customer_state, ex. SP, RJ, MG) sur une carte, sans coordonnées. Pour ton dashboard (CA par État, taux de retard par État), c’est suffisant.




La couche MART (de data mart) est la couche finale de ta base : celle qui contient les données prêtes à être consommées par les utilisateurs métier et les outils de BI comme Power BI.

Le principe des couches

Couche	Contenu	Qui l’utilise
RAW	Copie brute des CSV, sans transformation (tout en texte)	Toi, le data engineer
MART	Données nettoyées, typées, organisées en tables de faits et dimensions	Power BI, les analystes

Chaque couche a un rôle : RAW garde une trace fidèle de la source (si tu te trompes dans une transformation, tu peux tout recalculer sans retélécharger les fichiers), et MART est la version “propre” qu’on expose.

Ce qu’il y a dans ton MART

FACT_ORDERS : la table de faits. Elle contient les événements mesurables (une ligne par article commandé) avec les chiffres : prix, frais de port, délai de livraison, retard, note client.
DIM_CUSTOMER, DIM_PRODUCT, DIM_SELLER : les tables de dimensions. Elles décrivent le contexte : qui a acheté, quel produit, quel vendeur.

C’est ce qu’on appelle un modèle en étoile : la table de faits au centre, reliée aux dimensions autour.

        DIM_CUSTOMER
              |
DIM_PRODUCT — FACT_ORDERS — DIM_SELLER

Pourquoi ne pas brancher Power BI directement sur RAW ?

Les données RAW sont en texte : Power BI ne saurait pas calculer une somme ou une évolution dans le temps sans conversion.
Les tables RAW ne sont pas pensées pour l’analyse : il faudrait refaire les jointures et les calculs (délai, retard) dans Power BI, avec des risques d’erreurs et de lenteur.
C’est une question de sécurité et de propreté : Power BI n’accède qu’à MART (c’est le rôle de POWERBI_ROLE à l’étape 8), jamais aux données brutes.

En une phrase : RAW = ce qu’on reçoit, MART = ce qu’on livre.

Dans le monde professionnel, tu entendras aussi les termes Bronze / Silver / Gold (architecture médaillon). RAW correspond à Bronze et MART à Gold. On a sauté la couche Silver (nettoyage intermédiaire) pour tenir en 2 jours, en faisant le nettoyage directement dans les requêtes qui créent MART.

<img width="996" height="708" alt="image" src="https://github.com/user-attachments/assets/9157e218-2aea-44e2-89aa-fb15ce666048" />

