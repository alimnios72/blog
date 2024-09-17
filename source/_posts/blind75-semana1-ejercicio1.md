---
title: Blind 75 Semana 1, Ejercicio 1
date: 2023-08-09 22:39:27
tags:
  - algoritmos
  - typescript
  - entrevista tecnica
  - blind 75
  - secuencias
---

En la siguiente serie de posts voy a mostrar mis soluciones a un conjunto de problemas de preparación para entrevistas técnicas denominado [Blind 75](https://www.techinterviewhandbook.org/best-practice-questions/). Los problemas están divididos por semana y por tipo de problema.
Mis soluciones estarán en typescript. Sin más que añadir, empecemos a programar.
<!-- more -->

Semana 1 | Secuencias | Two Sum | [Link a leetcode](https://leetcode.com/problems/two-sum/)

### Problema
Dado un arreglo de números enteros `nums` y un número entero `target` (objetivo), regresa los índices de dos números que sumados tengan el valor del número `target`.

*Ejemplo 1:*
```
Entrada: nums = [2,7,11,15], target = 9
Salida: [0,1]
Explicación: Dado que nums[0] + nums[1] == 9, regresamos [0, 1].
```

*Ejemplo 2:*
```
Entrada: nums = [3,2,4], target = 6
Salida: [1,2]
```

*Ejemplo 3:*
```
Entrada: nums = [3,3], target = 6
Salida: [0,1]
```

*Restricciones:*

- 2 <= nums.length <= 104
- -109 <= nums[i] <= 109
- -109 <= target <= 109
- Sólo existe una respuesta válida

### Solución 1
La primera solución que me viene a la mente, sin considerar la eficiencia del código, es utilizar dos `for` loops. El primero recorre todos los elementos desde el primero hasta el penúltimo número y el segundo `for` va siempre un índice más adelante que el loop anterior. Explicado de otra forma, esto es lo mismo a tener 2 apuntadores: el primero avanza por todos los números del arreglo, a excepción del último elemento, y el segundo apuntador checa todos los numéros delante del primer apuntador. En cualquier caso, si la suma de los dos números es igual al objetivo, entonces tenemos una respuesta y podemos abandonar los ciclos.

La solución descrita es la siguiente:

```typescript
function twoSum(nums: number[], target: number): number[] {
    let result = [];
    for (let i = 0; i < nums.length - 1; i++) {
        for (let j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] === target) {
                result.push(i);
                result.push(j);
                break;
            }
        }
    }

    return result;
};
```

A pesar de que ésta solución resuelve el problema en cuestión no es la más óptima dado que tiene una complejidad O(n<sup>2</sup>), es decir, por cada elemento visitado en el primer loop se visitan de nuevo todos los elementos del arreglo.

### Solución 2
La solución anterior no es eficiente porque para cada elemento en el arreglo debemos de volver a recorrer todo el arreglo en cada iteración hasta encontrar el par de números que satisfacen la condición u objetivo. Una solución más eficiente sería recorrer el arreglo en tan sólo una ocasión pero, ¿cómo podemos encontrar el segundo número que satisface la condición?. Una 