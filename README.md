[![Waffle.io - Columns and their card count](https://badge.waffle.io/Nealarec/cube-sum.png?columns=all)](https://waffle.io/Nealarec/cube-sum?utm_source=badge)
[![Build Status](https://travis-ci.org/nealarec/cube-sum.svg?branch=master)](https://travis-ci.org/nealarec/cube-sum)

## React Create App

Este proyecto fue creado con [Create React App](https://github.com/facebookincubator/create-react-app)
en este [archivo](CREATE-APP-README.md) encontrarás toda la información sobre la aplicación

### Instrucciones para el desarrollo

```bash
git clone https://github.com/nealarec/cube-sum

cd cube-sum && npm install

npm test

// En otra consola
npm start
```

# Caso de uso
Este repositorio busca dar una solución al [Ejercicio](https://www.hackerrank.com/challenges/cube-summation/problem) propuesto en https://www.hackerrank.com/challenges

En el ejercicio nos proponen un sistema capaz de crear matrices cúbicas `n*n*n`,
modificar cada uno de sus elementos y sumar los elementos que se encuentran
contenidos entre dos de sus puntos, mediante texto plano. El formato de entrada
cuenta con 4 tipos de registro que son:

* `T`: Test Case - es un numero único que se utiliza para el seguimiento del caso de prueba.
* `N M`: Son dos números que representan la dimensión de la matriz y la cantidad de operaciones que se deben realizar sobre ella.
* `UPDATE x y z W`: Modifica el elemento `(x,y,z)` con el valor `W`.
* `QUERY x1 y1 z1 x2 y2 z2`: Obtiene la suma de los elementos presentes entre las coordenadas `(x1,y1,z1)` y `(x2,y2,z2)` incluyéndolas.

El ejercicio también cuenta con las siguientes restricciones:

* 1 <= T <= 50
* 1 <= N <= 100
* 1 <= M <= 1000
* 1 <= x1 <= x2 <= N
* 1 <= y1 <= y2 <= N
* 1 <= z1 <= z2 <= N
* 1 <= z, y, z <= N
* -10<sup>9</sup> <= W <= 10<sup>9</sup>

### Entrada válida
```
2
4 5
UPDATE 2 2 2 4
QUERY 1 1 1 3 3 3
UPDATE 1 1 1 23
QUERY 2 2 2 4 4 4
QUERY 1 1 1 3 3 3
2 4
UPDATE 2 2 2 1
QUERY 1 1 1 1 1 1
QUERY 1 1 1 2 2 2
QUERY 2 2 2 2 2 2
```
### Salida de dicha entrada
```
4
4
27
0
1
1
```

# Implementación

Para dar solución a este sistema he decidido trabajar en javascript mediante interfaz web,
más exactamente una aplicación de React usando ES6, la implementación cuenta con 3
clases principales:

- `Matrix3D`: Se encarga de contener, modificar y operar sobre las matrices.
- `InputValidator`: Se encarga de validar la entrada del sistema así como de interpretar las operaciones que se defienden en ella.
- `MatrixController`: Su función es comunicar los pasos entregados por `InputValidator` a una instancia de `Matrix3D` y almacenar la salida.

Un componente de React comunica las acciones de usuario con instancias
de `InputValidator` y `MatrixController` para completar el sistema.

## Matrix3D
Implementa funciones para realizar las operaciones descritas en el problema,
sus responsabilidades son: crear matrices `n*n*n`, almacenarlas, obtener o notificar la existencia
de una posición dentro de la matriz almacenada, obtener la matriz existente entre dos puntos de la
matriz almacenada y sumar sus elementos.

| Función                                       | Descripción                                       |
|-                                              |-                                                  |
| Matrix3D.make(N)                              | Crea una matriz `N*N*N`                           |
| Matrix3D.get(x, y, z)                         | Obtiene el valor de una posición en la matriz     |
| Matrix3D.set(x, y, z, W)                      | Cambia el valor del posición en la matriz         |
| Matrix3D.subMatrix(x1, y1, z1, x2, y2, z2)    | Obtiene la matriz existente entre 2 putos         |
| Matrix3D.sum(x1, y1, z1, x2, y2, z2)          | Obtiene la suma de los valores entre 2 puntos     |

## InputValidator
Analiza la entrada del sistema y determina si puede o no ser procesada, sus responsabilidades son:
analizar de manera individual las líneas de la entrada y determinar su coherencia, interpretar las
operaciones que ellas contienen, traducirlas y almacenarlas para su procesamiento, comunicar
la validez de dicha entrada y presentar sugerencias para continuar, en caso de que
esta sea inválida debe ubicar el error y motivo del fallo e indicar como corregirlo.

Es importante destacar que, aun cuando es una clase con funcionalidad extensa, no debemos
dividirla, porque toda esta funcionalidad está ligada directamente al proceso de validación
y si la separamos nos veríamos en la obligación repetir código con todas las desventajas que esto implica.

| Función                                           | Descripción                                       |
|-                                                  |-                                                  |
| InputValidator.setInput(txt)                      | Almacena la entrada para ser analizada            |
| InputValidator.reset()                            | Devuelve las propiedades al valor original        |
| InputValidator.isValid()                          | Valida cada línea de la entrada                   |
| InputValidator.processDefinition(line)            | Registrar una definición N M                      |
| InputValidator.processOperation(line)             | Registra una operación ya sea UPDATE o QUERY      |
| InputValidator.testTestCase(line, line_number)    | Valida un registro T                              |
| InputValidator.testDefinition(line, line_number)  | Valida un registro N M                            |
| InputValidator.testOperation(line, line_number)   | Valida un registro de operación                   |
| InputValidator.testUpdate(line, line_number)      | Valida un registro UPDATE x y z W                 |
| InputValidator.testQuery(line, line_number)       | Valida un registro QUERY x1 y1 z1 x2 y2 z2        |

Los métodos `test` retornan falso o verdadero dependiendo de los criterios de validación para cada caso,
si el registro es inválido, almacena el motivo en la variable error junto al registro y el número de línea.
Los métodos `process` realizan modificaciones sobre las variables de estado
de la clase y registran los procesos en un objeto, para que posteriormente sean entregados como operaciones
al controlador, quien se encargará de ejecutarlas. El método `isValid` itera sobre las líneas de archivo
realizando una búsqueda selectiva de cada registro de acuerdo a las variables de estado, una vez se
identifica el registro, se procesa con el método definido para ello.

## MatrixController

Esta clase solo implementa el método `process`, el cual recibe un arreglo de operaciones con el formato:

```
MAKE N
UPDATE x y z W
QUERY x1 y1 z1 x2 y2 z2
```

y ejecuta las funciones de una instancia de `Matrix3D`, almacenando los resultados de cada `QUERY` en
la variable plain_output, la cual es la salida standard de sistema.
