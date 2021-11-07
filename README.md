# TP4 Fotogrametría y Teledetección
### Elaborado por: Daniela Amador y Marget Martínez

## Introducción

## Firmas espectrales

```
//Filtros y fechas
var image = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2020-01-01', '2020-02-15')
    .sort('CLOUD_COVER')
    .first());
    
//Composición color verdadero
Map.addLayer(image, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');
Map.addLayer(tempisque,{color:"2bbc69"},"Tempisque");

//Bandas de análisis, y feature collection.
var subset = image.select('B[1-7]')
var samples = ee.FeatureCollection([Bosque, Cultivo, Cuerpos_Agua, Suelo_Desnudo, Nubes, Urbano, Sombra_Nube]);

//Gráfico 1
var Chart1 = ui.Chart.image.regions(
 subset, samples, ee.Reducer.mean(), 10, 'etiqueta')
 .setChartType('ScatterChart');

//Desplegar gráfico
print(Chart1)

//Opciones de personalizado
var plotOptions = {
 title: 'Landsat-8 Surface reflectance spectra',
 hAxis: {title: 'Wavelength (nanometers)'},
 vAxis: {title: 'Reflectance'},
 lineWidth: 1,
 pointSize: 6,
 series: {
 0: {color: 'green'}, // Bosques
 1: {color: 'orange'}, // Cultivo
 2: {color: 'blue'}, //Cuerpos agua
 3: {color: 'brown'}, // Suelo_desnudo
 4: {color: 'purple'}, // Nubes
 5: {color: 'gray'}, // Urbano
 6: {color: 'black'}, // Sombra_nube
}};

//*Lista de longitudes de onda Landsat-8 para las etiquetas del eje X. Esto ser revisa en los metadatos de la colección*//
var wavelengths = [443, 482, 562, 655, 865, 1609, 2201];

// Gráfico 2 
var Chart2 = ui.Chart.image.regions(
 subset, samples, ee.Reducer.mean(), 10, 'etiqueta', wavelengths)
 .setChartType('ScatterChart')
 .setOptions(plotOptions);
 
// Desplegar gráfico
print(Chart2);

```
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/grafic2.jpeg)
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/grafico.jpeg)

De acuerdo al gráfico "Landsat-8 Surface reflectance espectra, las clases "Sombra_Nube", "Bosque", "Cultivo" y "Urbano" presentan mayor absorción de longitud de onda que las demás variables entre los 500 y 750 nanómetros. Lo que indica que la banda roja es donde presentan una mayor absorción. Luego se puede observar como todas las clases aumentan su reflectancia (presentando los valores más grandes), justo en la banda de infrarrojo cercano, entre los 750 y 1000 nanómetros. La reflectancia vuelve a bajar para todas las clases en la banda infrarroja de onda corta 1 (1500-1750 nm), a excepción del "Suelo_Desnudo" que aumenta ligeramente. Por último, todas las clases bajan en la banda de infrarrojo de onda corta 2. En este caso, las clases que manejan una menor distorción son "Sombra_Nubes" y "Cuerpos_Agua".

# Clasificador smileCart

```
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
# Clasificador randomForest

```
//Filtros y fechas
var image = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2020-01-01', '2020-02-15')
    .sort('CLOUD_COVER')
    .first());

Map.addLayer(image, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');
Map.addLayer(tempisque,{color:"2bbc69"},"Tempisque");

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

// Algoritmo de clasificación. 
var classifier = ee.Classifier.smileRandomForest(7).train({
  features: training,
  classProperty: 'landcover', 
  inputProperties: bands
});

//Correr clasificación
var classified = image.select(bands).classify(classifier);
var final= classified.clip(tempisque);

//Imprimir clasificación
Map.centerObject(classNames, 11);
Map.addLayer(final,
{min: 0, max: 6, palette: ['green', 'orange', 'blue','brown','white','gray', "black"]},
'classification');

Código Matriz Confusión

//Matriz de confusión

var valNames = vbosque.merge(vBodyWater).merge(vcultivo).merge(vSueloDesnudo).merge(vNubes).merge(vUrban).merge(vSombras_Nubes);

//
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

# Descripción de los clasificadores

segwsgrdgreh

# smileCart vs randomForest
sdfwefw

# Matrices 

## Matriz de smileCart
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/smileCart.jpeg)

Validación de precisión general: 1

## Matriz de randomForest
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/randomforest.jpeg)

Validación de precisión general: 0.996328350380278

##  Síntesis de ambas
Las matrices de confusión, según lo visto en clase, nos permiten conocer que tan preciso es un modelo de clasificación. Bajo esta lógica, al analizar ambas matrices el smileCart es el que muestra una mejor clasificación porque no ha confundido ninguna clase. Esto se ve reflejado, también, en el valor de validación con un 100% de precisión general. Sin embargo, el valor de validación para la matriz del clasificador randomForest es también alto, con un 99% de precisión general. 

# Mapas finales

![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/MapaFinal_smileCart.png)
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/randomForestmapa.png)

# Conclusiones

# Referencias Bibliográficas 
