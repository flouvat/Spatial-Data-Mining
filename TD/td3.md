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
band5 = big_data.isel(band=0).values
print(band5)

# Get the value of pixel (1000,500) fro the first band
value_pixel = big_data.isel(band=0, x=1000, y=500).values
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
# Traiter des données rasters

Nous allons maintenant voir quelques opérations classiques faites sur les rasters.

## Reprojeter un raster

 Lorsque l'on doit effectuer des opérations et des analyses spatiales, il est important de s'assurer que les données manipulées se trouvent dans le même système de coordonnées  de référence. Il est donc courant de devoir reprojeter un raster dans un autre référentiel afin de garantir la précision des calculs.

 Dans l'exemple ci-dessous, nous reprenons notre image `RapidEye` et la projetons dans le référentiel `WGS84`.

```python
import rasterio
from rasterio.enums import Resampling
from rasterio.warp import calculate_default_transform, reproject

# Read the source raster
with rasterio.open('../data/raster/Poro/2011_.tif') as src:

    # Define the new CRS (WGS84)
    new_crs = 'EPSG:4326'  

    # Calculate the transformation and the dimensions of the new raster
    transform, width, height = calculate_default_transform(
        src.crs, new_crs, src.width, src.height, *src.bounds)

    # Copy the metadata of the source raster
    kwargs = src.meta.copy()
    kwargs.update({
        'crs': new_crs,
        'transform': transform,
        'width': width,
        'height': height
    })

    # Create a new raster with all the bands in the new CRS
    with rasterio.open('../data/raster/Poro/2011_WGS84.tif', 'w', **kwargs) as dst:
        for i in range(1, src.count + 1):
            reproject(
                source=rasterio.band(src, i),
                destination=rasterio.band(dst, i),
                src_transform=src.transform,
                src_crs=src.crs,
                dst_transform=transform,
                dst_crs=new_crs,
                resampling=Resampling.nearest
            )
```

Nous pouvons vérifier le raster a bien été projeté dans le nouveau système de coordonnées.

```python
# Read the initial Rapideye image
poro_2011 = rasterio.open('../data/raster/Poro/2011_.tif')
print(poro_2011.crs)

# Read the Rapideye image in WGS84 projection
poro_2011_WGS84 = rasterio.open('../data/raster/Poro/2011_WGS84.tif')
print("\n",poro_2011_WGS84.crs)
```

**Attention**, lorsque les traitements à effectuer s'appuient sur des distances ou des surfaces, il est important d'utiliser un système de coordonnées métrique. Dans l'exemple précédent, le référentiel de départ était RGNC91-93 basé sur la projection Lambert NC. Ce référentiel est déjà métrique, alors que WGS84 ne l'est pas (les mesures sont exprimées en degrés de latitude et longitude). 

## Créer une mosaïque de rasters

Il est souvent nécessaire de fusionner plusieurs rasters. Par exemple, un satellite peut ne capturer qu'une partie d'une zone ciblée lors d'un passage et une autre partie lors d'un second passage. Dans ce cas, il est nécessaire de fusionner les deux images pour avoir des données sur l'ensemble de la zone d'étude. On dit encore que l'on créé une mosaïque de rasters. `Rasterio` offre aussi des fonctionnalités permettant de faire cette opération.

Pour illustrer cela, nous allons utiliser deux images satellites [Landsat-7](https://www.indexdatabase.de/db/s-single.php?id=8) mise à disposition par l'[USGS](https://www.usgs.gov/landsat-missions/landsat-data-access) ("US Geological Service"). Ces deux images capturent des parties différentes de la région autour de Marseille. L'objectif est de les fusionner en un seul raster. 

Nous commençons par lire et afficher ces deux rasters.

```python
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt

# Read and display the two rasters
marseille_nord = rasterio.open("../data/raster/Marseille/marseille_nord.tif")
marseille_sud = rasterio.open("../data/raster/Marseille/marseille_sud.tif")

plt.axis("off")
show(marseille_nord)
plt.axis("off")
show(marseille_sud)
```

Nous les fusionnons ensuite en mémoire et enregistrons le raster résultant.

```python
from rasterio.merge import merge

# Create a list of the rasters to merge
src_files_to_mosaic = [marseille_nord, marseille_sud]

# Merge the rasters
mosaic, out_trans = merge(src_files_to_mosaic)

# Get the spatial informations for the raster mosaic 
out_meta = marseille_nord.meta.copy()
out_meta.update({
    "driver": "GTiff",  # Le format de sortie, dans cet exemple, GeoTIFF
    "height": mosaic.shape[1],
    "width": mosaic.shape[2],
    "transform": out_trans
})

# Save the mosaic in a new file
output_path = "../data/raster/Marseille/mosaique.tif"
with rasterio.open(output_path, "w", **out_meta) as dest:
    dest.write(mosaic)

# Display the newly created mosaic
plt.axis("off")
show(rasterio.open(output_path))
```

## Découper un raster en fonction d'une emprise

Une autre opération classique consiste à découper un raster en fonction d'un polygone ("clipping"). Le polygone représente les limites d'une zone d'intérêt (p.ex. une ville). On peut par exemple utiliser les données vectorielles décrivant les quartiers de la ville de Marseille pour découper l'image précédente. En effet, ce raster est de taille importante. Son emprise spatiale est bien au delà des limites de Marseille. Si l'objectif est d'étudier Marseille, il contient donc beaucoup de  données inutiles qui pourraient ralentir les autres traitements. Il faut donc les supprimer et ne conserver que les données de la ville.

Afin de constater cela, nous allons à nouveau ouvrir la mosaïque précédente en utilisant ` .

```python
import xarray as xr
import matplotlib.pyplot as plt

# Load a Landsat-7 mosaic
marseille_mosaique= rasterio.open("../data/raster/Marseille/mosaique.tif")
           
print(marseille_mosaique)  

# Check the CRS
print("CRS:",marseille_mosaique.crs)

# Display the raster
plt.title("Landsat-7 RGB Image")
plt.axis("off")
show(marseille_mosaique)
```

Nous allons ensuite charger les données vectorielles représentant les quartiers de Marseille.

```python
import geopandas as gpd
import matplotlib.pyplot as plt

# Load Marseille districts
quartiers_marseille = gpd.read_file('../data/vector/quartiersmarseille.zip')

# Check the CRS
print("CRS:",quartiers_marseille.crs)

quartiers_marseille.plot()
plt.axis("off")
plt.show()
```

On constate alors que les données n'ont pas le même système de coordonnées. Nous allons donc projeter les données vectorielles dans le référentiel du raster (EPSG:32631, i.e. WGS 84 / UTM zone 31N).

```python
quartiers_marseille = quartiers_marseille.to_crs(epsg=32631)
```

Nous allons ensuite procéder au découpage du raster gra^ce à la méthode [mask()](https://rasterio.readthedocs.io/en/stable/api/rasterio.mask.html) de `Rasterio`.

```python
import geopandas as gpd
import xarray as xr
import rasterio
from rasterio.mask import mask
from shapely.geometry import mapping

# Select all polygons 
shapes = [mapping(geom) for geom in quartiers_marseille.geometry]

# Clip the raster regarding input polygons
out_image, out_transform = mask(dataset=marseille_mosaique, shapes=shapes, crop=True)

# Define the metadata of the resulting raster
out_meta = marseille_mosaique.meta.copy()
out_meta.update({
        "driver": "GTiff",
        "height": out_image.shape[1],
        "width": out_image.shape[2],
        "transform": out_transform
    })

# Save the clipped raster
output_path = "../data/raster/Marseille/marseille.tif"
with rasterio.open(output_path, "w", **out_meta) as dest:
    dest.write(out_image)
```

Pour vérifier le résultat, il suffit d'ouvrir le nouveau raster et de l'afficher.

```python
# Read the clipped raster
marseille_clip= rasterio.open("../data/raster/Marseille/marseille.tif")
           
# Display the raster
plt.title("Landsat-7 RGB Image")
plt.axis("off")
show(marseille_clip)
```