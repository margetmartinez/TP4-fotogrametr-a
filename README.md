# TP4-fotogrametr-a
```
//Compare la cobertura de validación contra los datos de los resultados de la clasificacion
var testAccuracy = validation.errorMatrix('landcover', 'classification');
//Imprimir la matriz de confusión en la consula
print('Validation error matrix: ', testAccuracy);
//Imprimir la presición general en la consola
print('Validation overall accuracy: ', testAccuracy.accuracy());
```
