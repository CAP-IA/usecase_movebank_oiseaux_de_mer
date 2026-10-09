<h1 align="center">
  Analyse des déplacements et de l'alimentation des oiseaux marins
</h1>

<p align="center">
  Un tableau de bord interactif pour explorer les déplacements et les comportements alimentaires des oiseaux marins.
</p>

<p align="center">
  <a href="https://usecase-oiseaux-de-mer.onrender.com">
    <img src="https://img.shields.io/badge/🚀_Accéder_à_l'application-0078D4?style=for-the-badge"
         alt="Accéder à l'application" height="32">
  </a>
</p>

<p align="center">
  <sub>Cliquez sur le bouton pour découvrir l'application interactive</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white" alt="Streamlit">
  <img src="https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white" alt="Pandas">
  <img src="https://img.shields.io/badge/Plotly-3F4F75?style=flat-square&logo=plotly&logoColor=white" alt="Plotly">
</p>

<hr>

Ce projet est un **dashboard (tableau de bord) interactif développé avec Streamlit** permettant d’explorer les déplacements de goélands bruns suivis par GPS. L’application est hébergée sur **Render** et accessible directement en cliquant sur le lien intégré au titre de ce README. Elle permet d’analyser les trajectoires individuelles, les distances parcourues, les rythmes d’activité horaires, les excursions effectuées hors du nid ainsi que les principales caractéristiques du mouvement.

Un second mode détecte des **zones potentielles de recherche alimentaire**. Les positions lentes situées hors de la zone du nid sont regroupées spatialement avec l’algorithme DBSCAN. Une carte de chaleur, des indicateurs de réutilisation des zones et une chronologie des visites complètent cette analyse.

> Les zones détectées correspondent à des concentrations de positions lentes. Elles constituent un indicateur exploratoire et ne prouvent pas directement qu’une capture de nourriture a eu lieu.

## Auteur

Dashboard développé par [CAP IA - université de Bordeaux](https://www.u-bordeaux.fr/universite/notre-strategie/nos-leviers/cma-competences-et-metiers-davenir/cap-ia).

## Fonctionnalités principales

- filtrage par individu et période
- choix du fond de carte, avec un fond satellite optionnel via Mapbox
- nettoyage des horodatages, coordonnées et positions signalées comme aberrantes
- calcul des distances entre positions successives avec la formule de Haversine
- carte interactive des trajectoires
- distance parcourue par jour et par individu
- cycle horaire descriptif à partir de fenêtres temporelles
- typologie des excursions hors de la zone du nid
- signature du mouvement à partir de la vitesse recalculée et du changement de direction
- détection de zones alimentaires potentielles avec DBSCAN
- carte de chaleur spatiale allégée
- mesure de la fidélité et du partage des zones
- chronologie des visites
- onglet méthodologique expliquant la construction et les limites des clusters
- tableau récapitulatif par individu

## Données utilisées

Le dashboard utilise le jeu de données suivant :

> Garthe et al. (2016), *Data from: Terrestrial and Marine Foraging Strategies of an Opportunistic Seabird Species Breeding in the Wadden Sea*, Movebank Data Repository. DOI : [10.5441/001/1.nk286sc0](https://doi.org/10.5441/001/1.nk286sc0).


Le fichier CSV n’est pas distribué avec le code source. Il peut être placé manuellement dans le dossier `data/` après avoir été téléchargé conformément aux conditions d’utilisation de la source, ou récupéré directement depuis le site via une API.  

En cas de téléchargement manuel, placez le fichier dans le dossier `data/` en conservant exactement le nom suivant :

```text
FTZ_Foraging_in_lesser_black-backed_gulls.csv
```

Les colonnes indispensables sont :

| Colonne | Rôle |
|---|---|
| `tag-local-identifier` | identifiant de l’individu |
| `timestamp` | date et heure de la position GPS |
| `location-lat` | latitude |
| `location-long` | longitude |

La vitesse Movebank exprimée en m/s est convertie en km/h.

## Méthode d’analyse

### Préparation des trajectoires

Les coordonnées incomplètes ou hors limites sont supprimées, ainsi que les observations marquées manuellement comme aberrantes. Les positions sont ensuite triées par individu et par horodatage.

La distance entre deux positions successives est calculée sur la sphère terrestre avec la formule de Haversine. La vitesse de segment correspond au rapport entre cette distance et le temps écoulé.

Dans cette version, le nid de chaque individu est estimé à partir de sa première position GPS valide. Il s’agit d’un proxy opérationnel qui doit être remplacé par des coordonnées de nid vérifiées lorsqu’elles sont disponibles.

### Cycle horaire

Les déplacements sont regroupés par fenêtres de 15 minutes. Une fenêtre est conservée lorsqu’elle contient au moins quatre positions et couvre au moins huit minutes. Pour chaque heure UTC, le dashboard calcule la distance médiane ainsi que les quartiles Q25 et Q75.

### Zones alimentaires potentielles

Avec les paramètres par défaut, la démarche est la suivante :

1. conserver les positions dont la vitesse GPS est inférieure à 5 km/h
2. exclure les positions situées à moins de 1 km du nid estimé
3. convertir les coordonnées en radians
4. appliquer DBSCAN avec une distance de Haversine, un rayon de 500 m et au moins 20 points
5. attribuer l’étiquette `-1` aux points considérés comme du bruit
6. classer les clusters par nombre de positions : Z1 est le plus fréquenté, puis Z2, Z3, etc.
7. afficher par défaut les 20 zones les plus denses

Ces seuils sont modifiables depuis la barre latérale du dashboard.

Une nouvelle visite commence lorsqu’un individu revient dans une zone après plus de 30 minutes sans position dans cette même zone. La durée affichée reste une estimation dépendante du pas d’échantillonnage GPS.

## Organisation du projet

```text
.
├── .streamlit/
│   └── config.toml
│
├── src/
│   └── movebank_seabirds/
│       ├── app.py
│       ├── config.py
│       │
│       ├── data/
│       │   └── processing.py
│       │
│       ├── analysis/
│       │   ├── movement.py
│       │   └── foraging.py
│       │
│       ├── visualization/
│       │   ├── charts.py
│       │   └── maps.py
│       │
│       └── ui/
│           └── components.py
│
├── .gitignore
├── .python-version
├── requirements.txt
├── render.yaml
├── LICENSE
└── README.md
```

Le projet est organisé de manière à séparer la logique de traitement des données, les analyses, les visualisations et l'interface Streamlit.

- `src/movebank_seabirds/data/` : chargement et préparation des données
- `src/movebank_seabirds/analysis/` : analyses des déplacements et du comportement de recherche alimentaire
- `src/movebank_seabirds/visualization/` : génération des graphiques et des cartes
- `src/movebank_seabirds/ui/` : composants de l'interface Streamlit
- `data/` : données utilisées par l'application

### `app.py` - orchestration

Ce fichier constitue le point d’entrée de Streamlit. Il ne contient que l’enchaînement général des traitements et des affichages.

| Fonction | Responsabilité |
|---|---|
| `main()` | charge les données, construit les filtres, applique la sélection et choisit le mode d’affichage |
| `render_standard_mode()` | calcule et affiche les trajectoires, analyses temporelles et analyses comportementales |
| `render_foraging_mode()` | calcule DBSCAN, la carte de chaleur, les visites et l’onglet méthodologique |

`st.set_page_config()` doit rester au début de ce fichier, avant toute autre commande Streamlit.

### `config.py` - configuration partagée

Ce fichier centralise les constantes et ne contient pas de traitement :

- noms du fichier et des colonnes
- rayon terrestre
- paramètres des fenêtres temporelles
- seuils alimentaires et paramètres DBSCAN par défaut
- taille de la grille de chaleur
- intervalle séparant deux visites
- hauteurs des graphiques
- palette des individus
- configuration Plotly commune

Les constantes sont importées dans les autres modules avec :

```python
import config as cfg
```

### `data/processing.py` - chargement et préparation

| Fonction | Responsabilité |
|---|---|
| `find_data_file()` | recherche le CSV dans les emplacements prévus |
| `load_and_prepare_data()` | charge, contrôle, nettoie et enrichit les positions GPS |
| `normalize_date_range()` | transforme la sélection Streamlit en dates de début et de fin fiables |
| `filter_trajectories()` | filtre les individus et les dates, puis prépare les segments valides |
| `build_individual_summary()` | calcule la synthèse statistique par individu |

### `analysis/movement.py` - analyses temporelles et comportementales

| Fonction | Responsabilité |
|---|---|
| `build_daily_distance()` | calcule la distance quotidienne de chaque individu |
| `build_hourly_summary()` | crée les fenêtres temporelles et les statistiques horaires médiane, Q25 et Q75 |
| `build_excursion_summary()` | identifie et résume les séquences continues observées hors du nid |
| `add_turning_angles()` | calcule le cap et le changement de direction entre segments successifs |
| `build_movement_signature_histogram()` | agrège la vitesse et l’angle de rotation dans une grille bidimensionnelle |
| `estimate_typical_interval_minutes()` | estime le pas GPS médian utilisé pour calculer la durée des visites |

### `analysis/foraging.py` - détection des zones alimentaires

| Fonction | Responsabilité |
|---|---|
| `empty_foraging_stats()` | initialise les compteurs de détection à zéro |
| `detect_foraging_zones()` | sélectionne les positions lentes hors nid et applique DBSCAN avec Haversine |
| `select_displayed_zones()` | conserve les clusters les plus denses pour l’affichage |
| `build_heat_grid()` | agrège les positions dans une grille spatiale pour alléger la carte de chaleur |
| `build_zone_visit_summary()` | construit les visites, la durée estimée et les indicateurs de réutilisation |
| `build_detection_histogram()` | résume l’espace vitesse–distance au nid utilisé pour sélectionner les candidats |
| `build_method_zone_table()` | prépare le tableau méthodologique des clusters affichés |

La fonction privée `_empty_zone_summary()` crée la structure d’un tableau de zones vide.

### `visualization/maps.py` - cartes Plotly

| Fonction | Responsabilité |
|---|---|
| `build_map_config()` | prépare les options interactives de la carte et ajoute le jeton Mapbox lorsqu’il existe |
| `build_trajectory_map()` | construit la carte normale des trajectoires et des positions GPS |
| `build_foraging_map()` | construit la carte de chaleur et les couches des zones alimentaires |

Les fonctions privées `_map_center()`, `_french_integer()` et `_add_foraging_zone_layers()` sont des fonctions internes utilisées uniquement pour construire les cartes.

### `visualization/charts.py` - figures Plotly

Chaque fonction retourne un objet `plotly.graph_objects.Figure` sans l’afficher directement.

| Fonction | Graphique construit |
|---|---|
| `build_empty_figure()` | message graphique lorsqu’une analyse ne contient aucune donnée |
| `build_daily_distance_figure()` | distance journalière par individu |
| `build_hourly_cycle_figure()` | cycle horaire médian avec intervalle Q25–Q75 |
| `build_excursion_figure()` | typologie des excursions hors du nid |
| `build_movement_signature_figure()` | densité vitesse–changement de direction |
| `build_zone_reuse_figure()` | fidélité et partage des zones alimentaires |
| `build_zone_timeline_figure()` | chronologie des visites par individu |
| `build_detection_space_figure()` | espace de sélection des positions candidates |

### `ui/components.py` - interface Streamlit

| Fonction | Responsabilité |
|---|---|
| `french_integer()` | formate les nombres selon la présentation française |
| `get_mapbox_token()` | lit le jeton Mapbox depuis l’environnement ou les secrets Streamlit |
| `build_color_map()` | attribue une couleur stable à chaque individu |
| `render_shared_individual_legend()` | affiche la légende commune des individus |
| `render_sidebar()` | affiche les filtres et retourne leurs valeurs |
| `render_metrics()` | affiche les quatre indicateurs principaux |
| `render_dashboard()` | organise la carte et les graphiques selon le mode actif |
| `render_foraging_method()` | affiche l’explication détaillée de DBSCAN et des visites |
| `render_summary_table()` | affiche le tableau de synthèse par individu |

## Bibliothèques utilisées

| Bibliothèque | Version | Utilisation |
|---|---:|---|
| [Streamlit](https://streamlit.io/) | 1.64.0 | interface, filtres, onglets, cache et affichage du dashboard |
| [pandas](https://pandas.pydata.org/) | 3.0.6 | chargement, nettoyage, regroupements et tableaux |
| [NumPy](https://numpy.org/) | 2.5.3 | calculs vectorisés, trigonométrie, histogrammes et Haversine |
| [Plotly](https://plotly.com/python/) | 7.1.0 | cartes et graphiques interactifs |
| [scikit-learn](https://scikit-learn.org/) | 1.9.1 | clustering DBSCAN des positions candidates |

## Installation avec Python

### 1. Récupérer le projet

```bash
git clone <URL_DU_DEPOT>
cd <NOM_DU_DEPOT>
```

### 2. Installer les dépendances

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 3. Ajouter les données

Placez le fichier suivant dans `data/` :

```text
data/FTZ_ Foraging_in_lesser_black-backed_gulls.csv
```

### 4. Lancer le dashboard

```bash
streamlit run src/movebank_seabirds/app.py --server.port 8501
```

Streamlit affiche ensuite dans le terminal l’adresse locale, généralement :

```text
http://localhost:8501
```

## Paramètres principaux

Les valeurs par défaut sont définies dans `config.py` :

| Paramètre | Valeur par défaut |
|---|---:|
| durée d’une fenêtre temporelle | 15 min |
| nombre minimal de positions par fenêtre | 4 |
| couverture minimale d’une fenêtre | 8 min |
| vitesse maximale d’une position candidate | 5 km/h |
| rayon d’exclusion autour du nid | 1 km |
| rayon DBSCAN | 500 m |
| nombre minimal de points DBSCAN | 20 |
| nombre maximal de zones affichées | 20 |
| séparation entre deux visites | 30 min |

## Limites d’interprétation

- le nid est estimé à partir de la première position valide et non d’une observation de terrain
- les distances supposent une trajectoire directe entre deux positions successives
- les résultats dépendent de la fréquence d’échantillonnage GPS
- DBSCAN utilise ici uniquement la proximité spatiale
- les seuils de vitesse, d’éloignement et de densité influencent directement le nombre de zones
- une position lente et concentrée peut correspondre à d’autres comportements que l’alimentation

## Licences

Les éléments du projet ont des licences distinctes :

| Élément | Licence |
|---|---|
| code source du dashboard | [MIT](https://opensource.org/license/mit) |
| données Movebank | [CC0 1.0. Universal](https://creativecommons.org/publicdomain/zero/1.0/) |

Copyright © 2026 CAP IA

La licence MIT concerne uniquement le code original du dashboard. Elle ne remplace pas la licence des données et ne permet pas de requalifier les données Movebank en données MIT ou CC0. Toute réutilisation des données ou de visualisations qui en sont dérivées doit conserver l’attribution de la source et respecter la restriction d’usage non commercial.

Pour un usage commercial des données, il est nécessaire d’obtenir une autorisation distincte auprès des titulaires des droits.
