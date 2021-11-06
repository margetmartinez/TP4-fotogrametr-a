# TP4 Fotogrametría y Teledetección
## Elaborado por: Daniela Amador y Marget Martínez

# Parte teórica

# Parte práctica
Código proveniente de Google Earth Engine

```
//Código con el clasificador smileCart

//Filtros y fechas
var image = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2020-01-01', '2020-02-15')
    .sort('CLOUD_COVER')
    .first());

Map.addLayer(image, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');

//Unión de clases
var classNames = Bosque.merge(Cuerposagua).merge(Cultivo).merge(Suelodesnudo).merge(Nubes).merge(Urbano).merge(Sombranube);
print(classNames);

//Valores de reflectancia
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

//Selección de bandas y propiedades
var training = image.select(bands).sampleRegions({
  collection: classNames,
  properties: ['landcover'],
  scale: 30
});
print(training);

//Algoritmo de clasificación 
var classifier = ee.Classifier.smileCart().train({ 
  features: training,
  classProperty: 'landcover', 
  inputProperties: bands
});

//Correr clasificación
var classified = image.select(bands).classify(classifier);
var final = classified.clip(tempisque);

//Imprimir clasificación 
Map.centerObject(classNames, 11);
Map.addLayer(final,
{min: 0, max: 6, palette: ['green', 'orange', 'blue','brown','white','gray', "black"]},
'classification');


Código Matriz de Confusión

//Matriz Confusión

var valNames = vBosque.merge(vCuerposagua).merge(vCultivo).merge(vSuelodesnudo).merge(vNubes).merge(vUrbano).merge(vSombranube);

// Validación
var validation = classified.sampleRegions({
  collection: valNames,
  properties: ['landcover'],
  scale: 30,
});
print(validation);

//
var testAccuracy = validation.errorMatrix('landcover', 'classification');

//
print('Validación de error matrix: ', testAccuracy);

//
print('Validación de precisión general: ', testAccuracy.accuracy());

```
