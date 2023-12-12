---
layout: post
title: "Manipuler des rasters en Python"
categories: jekyll update

mathjax: true
---


Comme nous l'avons vu précédemment, les objets géographiques peuvent être principalement représentées sous deux formes : `vectorielles` ou `raster`. Dans le modèle de données raster, les données sont représentées sous forme de `matrices de valeurs`, de `grilles de "pixels"`.  La grille est associée à un emplacement géographique spécifique et chaque cellule de la grille contient une valeur représentant certaines informations, telles que une information spectrale, une altitude, une température ou la présence/absence d'un objet. 


![raster model](td3-img/3-raster_data_model.jpg)

Cette représentation est couramment utilisée pour représenter des images satellite, des modèles numériques d'élévation, et d'autres types de données ayant une `grande étendue spatiale` (couvrant par exemple des pays entiers, des continents ou le monde) et des `mesures continues dans tout l'espace`.  Il est également possible de stocker des données discrètes ou catégorielles dans un raster, telles que des données de classification de l'occupation des sols. Les données raster sont couramment utilisées, par exemple, pour la surveillance environnementale, la météorologie et la cartographie. Les deux raster ci-dessous représentent une image prise par le satellite [RapidEye](https://earth.esa.int/eogateway/missions/rapideye) en 2008 et le résultat d'une classification des pixels de cette même image par rapport à la végétation.

![exemple image](td3-img/f2_20080619_ClipTot2.jpg)

![exemple classification](td3-img/f2_Clip_SVM2_SolsVeg_.jpg)

 La `résolution spatiale` d'un raster, i.e. la taille de la zone représentées par un pixel, est un aspect important de ce type de données. En effet, la résolution spatiale détermine avec quelle précision les entités du monde réel, telles que les bâtiments ou les éléments topographiques, peuvent être représentées ou identifiées à partir des données. Les autres caractéristiques importantes d'un raster sont sa `profondeur de bit` (la plage des valeurs de pixels) et son `nombre de bandes` (le nombre de couches, i.e. le nombre de valeurs associées à chaque pixel).

![image pleiade](td3-img/pleiadeVale.png)



Il existe de nombreux formats de fichiers pour stocker les données raster. Le plus courant est `GeoTIFF` ( .tif), qui est essentiellement un fichier image contenant des métadonnées de géoréférencement. Le package de base pour travailler avec des données raster en Python est [Rasterio](https://rasterio.readthedocs.io/en/latest/) .



# Lire des données raster avec Rasterio

`Rasterio` est une bibliothèque Python permettant de lire et écrire plusieurs formats raster différents. `Rasterio` est basé sur [GDAL](https://gdal.org/) et supporte donc tous les pilotes `GDAL` lorsque la bibliothèque est importée. `GDAL` est une des principales bibliothèques permettant de traiter de l'information géographique. Elle est principalement écrite en C/C++ et est utilisée dans un grand nombre de logiciels (p.ex. ArcGIS, ENVI, QGIS, Google Earth ou Orfeo Toolbox). 


## Lire et afficher une image satellitaire

Nous allons commencer par ouvrir avec `Rasterio` l'image `../data/raster/poro/2011_.tif` prise en 2011 par le satellite  [RapidEye](https://earth.esa.int/eogateway/missions/rapideye).

```Python
import rasterio

# Read the Rapideye image
poro_2011 = rasterio.open('../data/raster/Poro/2011_.tif')

print( type(poro_2011) )
print(poro_2011)
```

La méthode [open](https://rasterio.readthedocs.io/en/stable/api/rasterio.io.html#rasterio.io.ZipMemoryFile.open) de `Rasterio` permet de créer un objet [DatasetReader](https://rasterio.readthedocs.io/en/stable/contributing.html#dataset-objects) vers le fichier. Cet objet sert d'interface pour lire l'intégralité des données du fichier sur le disque et les charger en mémoire. 

Une fois chargé, nous pouvons étudier les principales caractéristiques de l'image tels son `nom`, le `nombre de bandes`, sa `largeur`, sa `hauteur`, son `système de coordonnées`, son `emprise`, et la `géo-transformation` effectuée. Ce dernier représente la transformation linéaire qui permet de convertir les coordonnées d'un pixel dans l'image en coordonnées spatiales dans le monde réel.  Cette information est **cruciale** pour associer les coordonnées des pixels aux positions géographiques. Elle permet de géoréférencer l'image, ce qui signifie qu'on peut déterminer les coordonnées géographiques d'un pixel donné dans l'image.

```python
print(poro_2011.name)
print(poro_2011.count)
print(poro_2011.width)
print(poro_2011.height)
print(poro_2011.crs)
print(poro_2011.bounds)
print(poro_2011.transform)
```

La méthode [profile](https://rasterio.readthedocs.io/en/stable/api/rasterio.io.html#rasterio.io.DatasetReader.profile) permet aussi d'avoir une grande partie des métadonnées classiques. Elle contient notamment la `profondeur des pixels` de l'image ("uint16", i.e. 16 bits).

```python
print(poro_2011.profile)
```

On peut retrouver une grande partie de ces informations sur le site [IDB sur la fiche associée au capter RapidEye](https://www.indexdatabase.de/db/s-single.php?id=10).

Il est possible d'importer n'importe quelle bande sous forme de tableau `numpy` grâce à la méthode [read(index)](https://rasterio.readthedocs.io/en/stable/api/rasterio._io.html#rasterio._io.DatasetReaderBase.read). Il est aussi possible d'afficher la bande en utilisant `matplotlib` en utilisant par exemple la méthode [imshow()](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.imshow.html).

```python
import matplotlib.pyplot as plt

# Read band 1 of the RapidEye image, i.e the blue band
blue_band = poro_2011.read(1)

# Print the numpy array with all values
print(blue_band)

# Display the band using matplotlib
plt.imshow(blue_band)
```

Pour avoir un rendu en RGB, il faut sélectionner les bandes correspondantes ( bande 1 -> bleu, bande 2 ->vert, bande 3 -> rouge, cf. [IDB(https://www.indexdatabase.de/db/s-single.php?id=10)]) et normaliser les valeurs (entre 0 et 1, ou 0 et 255).


```python
# Lire les autres bandes sélectionnées
red_band = poro_2011.read(3)
green_band = poro_2011.read(2)

# normalisation
blue_band_norm = blue_band.astype(float)/65535.0
red_band_norm = red_band.astype(float)/65535.0
green_band_norm = green_band.astype(float)/65535.0

# Afficher l'image RGB avec Matplotlib
# Créer une figure et un axe

rgb_image = [red_band_norm, green_band_norm, blue_band_norm]

plt.axis("off")
show(rgb_image, title='RGB Image', ax=ax)
```

L'image résultante paraît très sombre. Pour comprendre l'origine de ce problème, nous allons afficher l’histogramme de l'image.

```python
from rasterio.plot import show_hist

show_hist(poro_2011, bins=50, histtype='stepfilled', title="Histogram")
```

L'histogramme montre que les valeurs de l'image sont plutôt réparties entre 0 et 2500, et non 65535 comme le laisser présager l'encodage en 16 bits. Il faut donc normaliser différemment les bandes. Pour cela, nous pouvons utiliser la méthode [adjust_band()](https://rasterio.readthedocs.io/en/stable/api/rasterio.plot.html#rasterio.plot.adjust_band).


```python
from rasterio.plot import adjust_band

# Normalization between 0 and 1
red_band_norm = adjust_band(red_band)
green_band_norm = adjust_band(green_band)
blue_band_norm = adjust_band(blue_band)

rgb_image = [red_band_norm, green_band_norm, blue_band_norm]

plt.axis("off")
show(rgb_image, title='RGB Image', ax=ax)

```

## Lire et afficher un raster catégoriel

L'approche que nous venons d'utiliser pour afficher une image satellitaire peut être utilisée pour afficher tout type de rasters. Par exemple, nous pouvons l'utiliser pour afficher le raster `../data/raster/poro/Classif2011_5classeq_.tif` représentant une classification des pixels de l'image en 5 classes d'occupation du sol. Cette image a été obtenu par classification supervisée des pixels. Elle contient une seule bande avec des valeurs comprises entre 0 et 4.

```python
from rasterio.plot import show

# Read the raster
poro_2011_landuse = rasterio.open('../data/raster/Poro/Classif2011_5classes_.tif')

# Print the number of bands
print("Band count:",poro_2011_landuse.count)

# Display the raster
plt.axis("off")
show(poro_2011_landuse)
```

Nous pouvons d’ailleurs analyser plus en détails les informations de ce raster en affichant par exemple les différentes valeurs prises par les pixels et l'histogramme de ces valeurs.

```python
import numpy as np

# Get the first band
band1 = poro_2011_landuse.read(1)

# Get and display the classes
uniq_vals = np.unique(band1)

print(sorted(uniq_vals))

# Process the histogram using numpy
counts, bins = np.histogram(band1, bins=5)

# Print histogram outputs
for i in range(len(bins)-1):
    print("bin lower bound:", bins[i])
    print("counts:", counts[i])
```

## Manipuler des  rasters volumineux

Lorsque le raster est volumineux, la classe `DatasetReader` utilisée par défaut par `rasterio` peut ne pas être suffisamment performante. On utilise alors la bibliothèque [xarray](https://docs.xarray.dev/en/stable/index.html) en combinaison avec `rasterio`. Les deux principaux avantages de `xarray` sont :
- Une gestion plus simple des dimensions et des coordonnées : xarray permet d'utiliser, en plus des indices, des étiquettes pour les axes, ce qui peut être plus intuitif et plus facile à manipuler.
- Une prise en charge des opérations vectorisées sur les tableaux  : xarray permet donc de paralléliser les calculs en les appliquant  sur plusieurs pixels à la fois (tout comme le calcul matriciel de `numpy`).

Afin d'illustrer cela, nous allons reprendre l'image RapidEye précédente et refaire les mêmes opérations (lecture, affichage et accès) à partir de `xarray`. Nous allons commencer par la lecture et l'affichage des données. Lors de l'affichage, l'argument `robust=True` est utilisé pour ignorer les valeurs extrêmes  et ainsi améliorer la visibilité de l'image.


```python
import xarray as xr
import matplotlib.pyplot as plt

# More efficient way with big data
big_data = xr.open_rasterio("../data/raster/Poro/2011_.tif")
           
print(big_data)  

# Get the RGB bands
rgb_data = big_data.sel(band=[3,2,1])

# Afficher l'image RGB
rgb_data.plot.imshow(robust=True)
plt.title("RapidEye RGB Image")
plt.axis("off")
plt.show()
```
Le code suivant accède à une bande, ainsi qu'à la valeur d'un pixel, en utilisant les indices (tout comme avec `rasterio`).

```python
# Get all the values of the first band
band5 = big_data.isel(band=4).values
print(band5)

# Get the value of pixel (1000,500) fro the first band
value_pixel = big_data.isel(band=4, x=1000, y=500).values
print(value_pixel)
```
Nous modifions ensuite notre structure de données pour associer des étiquettes aux différentes bandes, et ainsi pouvoir accéder aux valeurs plus simplement.

```python
# Print band names
print("Band names: ",big_data.band.values)

# Rename the bands regarding RapidEye band names
big_data["band"] = ["Blue", "Green", "Red", "Red Edge", "Near Infrared"]

# Print new band names
print("\n New band names:")
print(big_data.band.values)

# Get all the values of the first band
band5 = big_data.sel(band="Near Infrared").values
print("\n Near Infrared band: ",band5)

# Get the value of pixel (1000,500) fro the first band
value_pixel = big_data.isel(x=1000, y=500).sel(band="Near Infrared").values
print("NIR value for a specific pixel:",value_pixel)
```
