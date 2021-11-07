# TP4 Fotogrametría y Teledetección
### Elaborado por: Daniela Amador y Marget Martínez

## Introducción

Para el desarrollo de este Trabajo Práctico N°4 es necesario conocer algunos conceptos que son claves, como el Machine Learning y las clasificaciones supervisadas. 

El Machine Learning o aprendizaje automático, es un subcampo de las ciencias de la computación y una rama de la IA (Inteligencia Artificial) cuyo objetivo es que las computadoras adquieran conocimientos y mejoren su aprendizaje al proporcionarles datos y observaciones para que actúen y ejecuten acciones como humanos. Para lograr esto se necesita de datos estructurados tradicionales y de una persona que etiquete estos datos, para que el algoritmo utilizado en cada caso pueda reconocer características específicas de cada tipo o categoría y las identifique, por ejemplo, en otras imágenes. 

En cuanto a las clasificaciones supervisadas, estas parten de un conjunto de clases conocido previamente. Estas clases establecidas se caracterizan en función del conjunto de variables mediante la medición de las mismas en individuos cuya pertenencia a una de las clases no genere incertidumbre. Estas son mejor conocidas como las áreas de entrenamiento. 


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

## Clasificador smileCart

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
## Clasificador randomForest

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

## Descripción de los clasificadores

### 1. SmileCart
El clasificador smileCart forma parte del grupo llamado Classification and Regression Trees o CART, conocidos en español como Árboles de Clasificación y Decisión. Estos son algoritmos de árbol de decisión que se usan para problemas de modelado predictivo de clasificación o regresión. Este proporciona una base para algoritmos importantes como árboles de decisión empaquetados, bosque aleatorio y árboles de decisión potenciados. Para representar este modelo se usa un árbol binario. Para crear un modelo CART se deben seleccionar variables de entrada y puntos de división en esas variables hasta que se construya un árbol adecuado (Brownlee, 2006). 

Como lo vimos en clase, en el caso del clasificador SmileCart que es una clasificación supervisada, necesitará de áreas o puntos de entrenamiento de los tipos de cobertura para que pueda ajustar de una manera correcta los parámetros que utiliza. Para que posteriormente identifique y etiquete de manera automática cada área con características similares a las áreas de entrenamiento preestablecidas. 

### 2. RandomForest

## smileCart vs randomForest
sdfwefw

## Matrices 

### Matriz de smileCart
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/smileCart.jpeg)

Validación de precisión general: 1

### Matriz de randomForest
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/randomforest.jpeg)

Validación de precisión general: 0.996328350380278

###  Síntesis de ambas
Las matrices de confusión, según lo visto en clase, nos permiten conocer que tan preciso es un modelo de clasificación. Bajo esta lógica, al analizar ambas matrices el smileCart es el que muestra una mejor clasificación porque no ha confundido ninguna clase. Esto se ve reflejado, también, en el valor de validación con un 100% de precisión general. Sin embargo, el valor de validación para la matriz del clasificador randomForest es también alto, con un 99% de precisión general. 

## Mapas finales

![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/MapaFinal_smileCart.png)
![image](https://github.com/margetmartinez/TP4-fotogrametr-a/blob/main/randomForestmapa.png)

## Conclusiones

Este trabajo práctico ejemplifica uno de los infinitos usos del Machine Learning, tanto para la teledetección como para la geografía. Incluso el uso de esta ciencia de la computación en otras distintas áreas y disciplinas. Ya que es una herramienta que facilita el análisis y procesamiento de datos.

## Referencias bibliográficas 

Brownlee, J. (8 abril, 2016). Classification And Regression Trees for Machine Learning. Machine Learnign Matery. https://machinelearningmastery.com/classification-and-regression-trees-for-machine-learning/ 

