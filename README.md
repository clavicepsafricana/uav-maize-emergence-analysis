# Maize Plant Counting with RGB and Multispectral Imagery

## Project overview

This project presents a geospatial workflow for assessing early maize emergence (V3) and estimating plant density from high-resolution RGB and multispectral imagery.

The agronomic target was **80,000 plants/ha**, with approximately **0.52 m row spacing** and a practical within-row spacing of **0.25 m**. The theoretical density at those spacings is approximately **76,923 plants/ha**.

## Objectives

- Detect individual maize plants at early growth stages.
- Compare RGB-based object detection with multispectral vegetation indices.
- Reduce false positives caused by bare soil and crop residue.
- Produce GIS-ready point and raster outputs for validation.

## Data and coordinate system

- High-resolution RGB orthomosaic.
- Five-band multispectral orthomosaic:
  - Red
  - Green
  - NIR
  - RedEdge
  - Quality/alpha band
- Validated reference points from an RGB-based count.
- Working CRS: **WGS 84 / UTM zone 20S (EPSG:32720)**.
- Multispectral pixel size: approximately **5 cm**.

## Workflow

### 1. Spatial alignment

The RGB reference and multispectral imagery were originally stored with different coordinate systems and footprints. The RGB layer was reprojected to EPSG:32720 and the multispectral imagery was cropped to the same working extent before comparison.

### 2. RGB vegetation indices

RGB indices were calculated to help separate green vegetation from soil and residue:

```text
ExG = 2G - R - B
```

```text
ExR = 1.4R - G
ExGR = ExG - ExR
```

ExG and ExGR were used as vegetation masks and candidate-object generators. Because residue and overlapping leaves can form connected regions, these outputs require object separation and visual validation before being interpreted as individual plants.

### 3. MSAVI2

MSAVI2 was calculated from the multispectral Red and NIR bands:

```text
MSAVI2 = (2NIR + 1 - sqrt((2NIR + 1)^2 - 8(NIR - Red))) / 2
```

The implementation used a protected discriminant to avoid invalid values caused by sensor noise:

```text
(2NIR + 1 - sqrt(abs((2NIR + 1)^2 - 8(NIR - Red)))) / 2
```

MSAVI2 was used as a supporting layer for vegetation confirmation and local-maximum detection. RGB remains the preferred source for counting individual plants because of its finer spatial detail.

### 4. Local maxima detection

SAGA Next Gen was used through QGIS Processing to detect local MSAVI2 maxima. A 5-pixel radius configuration was used as a practical approximation for the 0.25 m within-row spacing constraint.

## Preliminary results

- Target density: **80,000 plants/ha**.
- Practical spacing reference: **0.25 m between plants**.
- Preliminary MSAVI2 local maxima: **28,482 candidates** in the working subset.
- RGB and multispectral outputs were exported as GIS-compatible layers.

These results are preliminary and should be validated against manually reviewed sample plots before being used as a final agronomic count.

## Repository structure

```text
outputs/
  conteo_maiz_preliminar.gpkg
  Plantas_MSAVI2_maximos_validos.*

indices/
  MSAVI2_recortada_RGB_gdalcalc.tif
  ExG_RGB_valido.tif
  ExGR_RGB_valido.tif

reference/
  conteov3_reproyectado_32720.gpkg
```

## Main conclusion

High-resolution RGB imagery is the primary source for individual plant counting. ExG, ExGR and MSAVI2 are complementary tools for vegetation masking, soil/residue separation and quality control. The workflow demonstrates how RGB and multispectral data can be combined in QGIS to support crop establishment assessment.

## Tools

- QGIS 3.44
- QGIS Processing
- SAGA Next Gen
- GDAL
- GeoTIFF
- GeoPackage
- Shapefile

