---
layout: post
title: "Introduction aux données géographiques"
categories: jekyll update

mathjax: true
---

# Préparation de l'environnement de développement

Dans le cadre de ce cours, nous allons utiliser différents outils de développement pour mettre en pratique les principales notions de la fouille de données spatiales. Ces outils sont [Python3](https://www.python.org/), [Anaconda](https://www.anaconda.com/) et [Jupyter Notebook](https://jupyter.org/). Nous utiliserons aussi différentes bibliothèques telles que [Gdal](https://gdal.org/index.html), [GeoPandas](https://geopandas.org/en/stable/), [rasterio](https://rasterio.readthedocs.io/en/stable/), ou [Scikit-learn](https://scikit-learn.org/stable/). 
Afin d'éviter d'avoir à installer ces différents outils sur votre machine, nous allons utiliser GitHub, et plus particulièrement les [GitHub Codespaces](https://github.com/features/codespaces) pour coder en ligne. 

Vous devez tout d'abord cloner le dépôt (_repo_ dans la terminologie git) `flouvat/codespaces-spatial-dm`. Il contient les données géographiques utilisées dans le cours et divers fichiers de configuration. Pour cela, il faut suivre les étapes suivantes:
- Dans votre espace GitHub, recherchez le dépôt  `flouvat/codespaces-spatial-dm` et ouvrez-le.
- Pour le cloner dans votre compte, il suffit de cliquer sur le bouton `Fork` et de valider les paramètres en cliquant sur le bouton `Create fork`.

![Fork a Repo](td1-img/1-forkRepo.png)

L'environnement de développement en ligne de GitHub s'appelle [Codespaces](https://docs.github.com/fr/codespaces). Pour l'utiliser, il vous suffit de cliquer sur le bouton `<> Code`, puis de sélectionner l'onglet `Codespaces` et de créer un codespace sur le main. L’initialisation du codespace prend un peu de temps car elle installe certaines bibliothèques Python (p.ex. `matplotlib`, `numpy` et `pandas`).  
Un nouvel onglet s'ouvrira alors, et vous pourrez commencez à configurer votre environnement permettant de traiter des données géographiques.

![codespaces](td1-img/1-codespaces.png)

Dans le terminal de Codespace, vous devez ensuite installer toutes les bibliothèques nécessaires en utilisant la commande `conda env create -f environment.yml`. Elle permet de créer un environnement virtuel, appelé `geopy`, avec les librairies nécessaires installées. Si la commande n'est pas reconnue, ouvrez un autre terminal (menu déroulant ![](td1-img/1-bash.png) ) et relancez la. La procédure prend un peu de temps (5 minutes environ). Attention, il faudra donc ensuite utiliser cet environnement lorsque vous exécuterez votre code dans Jupyter Notebook. 

Pour tester votre installation, vous pouvez aller dans le répertoire `\jupyter`, ouvrir le fichier `test.ipynb`, et exécuter le code Python inclut à l'intérieur. Si vous l’exécutez sans utiliser l'environnement virtuel `geopy`, vous obtiendrez une erreur `ModuleNotFound` car les bibliothèques utilisées dans le code ne sont pas accessibles. Pour utiliser `geopy`, il vous faut utiliser le bon "noyau Python" pour lancer le code. Ce noyau peut être sélectionné en cliquant sur le bouton  ![Sélectionner le noyau](td1-img/1-noyau.png), puis en cliquant `Environnements Python` > `geopy`. Le code vous affichera alors un simple avertissement concernant `ResamplingOperation`. Une fois cela fait, vous pouvez exécuter le code de test.

# Les principaux types des données géographiques
Les données spatiales (ou données "géospatiales" ou encore "géographiques") sont des données contenant des informations de localisation.

Ces données  se présentent principalement sous deux formes :
- les données vectorielles 
- les données `raster`

## Les données vectorielles

Les données vectorielles s'attachent à représenter des formes (des objets géométriques) et à leur positionnement dans un système de coordonnées. Elles correspondent donc simplement à une collection de coordonnées à deux dimensions `(x,y)` ou a trois dimensions `(x,y,z)`, appelées "sommets". Ces sommets sont utilisés pour définir les objets suivants :
- `Point` : un seul point (p.ex. l'emplacement d'une personne)
- `Ligne` : deux ou plusieurs points connectés (p.ex. une route)
- `Polygone` : trois points ou plus connectés et fermés (p.ex. un bâtiment,  un lac ou la frontière d'un pays)


![basic vector data](td1-img/2-vector.png)


Comme illustré ci-dessous, ces objets peuvent être combinés pour aboutir à des données plus complexes telles que des `multi-points`, `multi-lignes`, `multi-polygones` ou des `collections géométriques`.

![complex vector data](td1-img/2-SpatialDataModel.png)

## Les données rasters

Contrairement aux données vectorielles, les données `raster` correspondent à une `matrice` de valeurs de `pixels` (également appelées "cells"). Chaque pixel représente une petite zone et contient une valeur numérique.

![raster](td1-img/2-raster.png)


Les données raster sont des images pour lesquelles chaque pixel représente une région spatiale. On parle de `résolution d’un raster` pour expliciter les dimensions de la zone (un carré) représenté par chaque pixel de l'image. Un raster de résolution de 1 mètre signifie que chaque pixel représente une zone de 1 m x 1 m au sol. On parle de "haute résolution" lorsque la valeur associée à la résolution est faible. Per exemple, une résolution de 1 mètre est supérieure à une résolution de 8 mètres, comme l'illustre l'image ci-dessous.

![resolution](td1-img/2-raster-resolution.png)

Les données raster sont utilisées dans différents contextes. Par exemple, les images satellitaires sont des raster, tout comme les Modèle Numérique de Terrain (MNT, i.e. Digital Elevation Model ou DEM en anglais). Les données raster peuvent être issues de télédétection ("remote sensing"), i.e. d’instruments permettant d’acquérir à distance des informations sur un objet géographique. Souvent, ces rasters sont capturés par des satellites, des avions ou des drones. 

Comme expliqué dans la documentation de [ArcGIS](https://pro.arcgis.com/fr/pro-app/latest/help/data/imagery/raster-bands-pro-.htm), certaines de ces images peuvent être composées d'une bande ou couche (i.e. mesurer une seule caractéristique), alors que d'autres peuvent être composées de plusieurs bandes (i.e. mesurer plusieurs caractéristiques). On parle alors d'`images multi-bandes`. Dans ce cas, chaque pixel est associé à plusieurs valeurs (autant que de bandes). Chaque bande correspond ainsi à une matrice de valeurs. Une bande représente une partie du spectre électromagnétique détecté par un capteur (p.ex. rouge, vert, bleu, proche infrarouge ou ultraviolet). Par exemple, les images Landsat-9 comportent 11 bandes différentes.

![raster multiband](td1-img/2-multiband-raster.gif)

Le nombre et le type de bandes capturées par les différents capteurs peuvent beaucoup varier. Ces informations sont importantes, et déterminent souvent le choix du capteur (p.ex. du satellite d'acquisition). En effet,  certaines bandes sont plus adaptées pour observer certains types d'objets (p.ex. le proche infra-rouge pour la végétation, l'ultraviolet pour les objets sous la surface de l'eau, ou le bleu pour les bâtiments). D'ailleurs, les géomaticiens ont défini un grand nombre d'indices basés sur une combinaison de bandes afin de mieux identifier et quantifier certains objets. Un exemple classique est le NDVI ("Normalized Difference Vegetative Index") permettant d'étudier la couverture végétale. La  [base de données IDB](https://www.indexdatabase.de/) ("Index DataBase") recense les différents capteurs existants, leurs caractéristiques et les indices calculables à partir de ceux-ci.

Une autre caractéristique importante d'un raster est  sa `profondeur de bit`, également appelée `profondeur de pixel`  ("bit depth" ou "pixel depth"). Elle définit la plage de valeurs distinctes que le raster peut stocker. Par exemple, un raster 1 bit ne peut stocker que 2 valeurs distinctes : 0 et 1, tandis qu'un raster 8 bits peut avoir 256 valeurs différentes comprises entre 0 et 255, comme le montre la figure suivante.

![bit detph](td1-img/3-raster_bit_depths.jpg)

## Les systèmes de coordonnées de référence (CRS)

Un `système de coordonnées de référence` ("Coordinate Reference System" ou CRS) est un élément clé de l'information géographique. Un CRS précise comment les coordonnées ou les géométries sont liées aux lieux sur Terre. Sans le CRS, les données géographiques sont simplement une collection de coordonnées dans un espace arbitraire. Cette information appartient aux `métadonnées`, tout comme l'horodatage de création des données.

Un CRS est composé de trois éléments:
- le `datum`  est un modèle de la taille et de la forme de la Terre (p.ex. ellipsoïde ou géoïde). Il indique aussi l'origine du système de coordonnées et son orientation par rapport à la surface de la terre. L'un des systèmes de référence les plus couramment utilisés est le système géodésique mondial (`WGS84`).
- la `projection cartographique` définit la transformation mathématique utilisée pour projeter la surface de la Terre sur un plan bidimensionnel. 
- des paramètres supplémentaires telles que le méridien central, le parallèle standard et le facteur d'échelle.

![map projection](td1-img/2-curved_surface_to_flat_plane.png)

La projection cartographique est une transformation particulièrement complexe qui engendre des [approximations](https://www.axismaps.com/guide/map-projections). Par conséquent, une projection doit être choisie en fonction du but d'utilisation, afin de préserver les aspects spécifiques de la carte qui sont les plus importants pour l'utilisateur, comme par exemple :
- la projection `Mercator` pour préserver les angles (donc les formes) ;
- les projections `Mollweide` ou `Albers equal area` pour préserver les surfaces ;
- la projection `Azimuthal equidistant` pour préserver les distances au centre de la projection.

![projections](td1-img/2-projections.jpg)

**A noter que ces distorsions restent minimes à une petite échelle (locale).**

