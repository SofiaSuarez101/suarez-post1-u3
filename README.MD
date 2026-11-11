## Postcontenido 1 Unidad 3

### Checkpoint 1: Estado inicial de los registros

![Estado de registros](capturas/parte%201/CP1_registros.png)

Al ejecutar el comando `R` sin argumentos, se observó que los registros de propósito general (AX, BX, CX y DX) valen 0000 y que SP vale FFFE, el tope inicial de la pila. Los cuatro registros de segmento (DS, ES, SS y CS) tienen el mismo valor, porque DOS asigna el segmento del PSP al programa. El IP vale 0100, la primera dirección ejecutable después del PSP. Esta captura corresponde a la sesión en `C:\LAB3PT1>`.

### Checkpoint 2: Volcado hexadecimal

![Volcado de memoria](capturas/parte%201/CP2_volcado_memoria.png)

Se rellenaron 64 bytes desde DS:0200 con el patrón AB CD EF usando `F 200 L40 AB CD EF` y se volcaron con `D 200 L40`. La salida del comando `D` tiene tres columnas. La primera muestra la dirección en formato segmento:desplazamiento. La segunda muestra 16 bytes por fila en hexadecimal, separados en dos grupos de 8 por un guion. La tercera muestra la representación ASCII de esos bytes. En esta captura la columna ASCII solo muestra puntos, porque AB, CD y EF no son caracteres imprimibles (el rango imprimible es 0x20 a 0x7E). El patrón se repite cíclicamente en las cuatro filas.

### Checkpoint 3: Ensamblado y desensamblado

![Ensamblado y desensamblado](capturas/parte%201/CP3_ensamblado_desensamblado.png)

Con el comando `A 100` se ingresó un programa de cuatro instrucciones y con `U 100 109` se verificó su codificación. Se observó que `MOV AX,0005` se codifica como `B8 05 00`, `MOV BX,0003` como `BB 03 00`, `ADD AX,BX` como `03 C3` e `INT 20` como `CD 20`. El programa ocupa 10 bytes. En los valores inmediatos de 16 bits se aprecia el formato little-endian: el byte menos significativo se almacena primero.

### Checkpoint 4: Memoria y direccionamiento directo

![Memoria y direccionamiento](capturas/parte%201/CP4_memoria_direccionamiento.png)

Se limpió una región de 16 bytes en DS:0300 con `F 300 L10 00`, y luego `E 300 78 56` modificó únicamente los dos primeros bytes, lo que se confirmó con dos volcados `D 300 L10`. Después se ensambló `MOV AX,[0300]` en CS:0320, que `U` mostró codificada como `A1 00 03`. Al ejecutarla con `T` (con IP en 0320), AX pasó a valer 5678, lo cual confirma que los bytes 78 56 se leyeron como la palabra little-endian 0x5678.

### Decisión técnica 1: Verificación no destructiva de una escritura (Paso 12)

Para confirmar que la escritura del Paso 11 tuvo el efecto esperado, se selecciona el comando D, porque es el único de los cuatro que solo lee memoria y no altera el estado del programa. El comando E con la lista completa de bytes (E 300 78 56) es determinista: escribe exactamente esos dos bytes y termina. En cambio, E 300 sin lista entra en modo interactivo, muestra el valor actual de cada byte y espera una tecla; si se escribe un valor por error, ese byte queda sobrescrito, de modo que la verificación podría modificar justamente lo que debía comprobar. El comando F tampoco es adecuado: aunque opera sobre un rango como D, escribe su patrón sobre todo el rango indicado y destruiría los bytes 78 56 que se querían verificar. El comando R muestra registros, no memoria, y con un nombre de registro (R AX) permite modificarlos. Por eso D 300 L10 es la opción segura: muestra los 16 bytes en hexadecimal y ASCII sin escribir nada y puede repetirse las veces que sea necesario.

### Decisión técnica 2: Direccionamiento inmediato frente a directo (Paso 14)

Aunque MOV AX,0005 (B8 05 00) y MOV AX,[0300] (A1 00 03) ocupan 3 bytes cada una, el direccionamiento directo a memoria requiere un acceso adicional al bus de memoria durante la ejecución. En el modo inmediato, el valor 0x0005 viaja dentro del propio flujo de instrucciones, así que queda disponible en el mismo fetch que trae la instrucción. En el modo directo, la instrucción solo contiene la dirección 0x0300: el procesador debe formar la dirección DS:0300 y realizar una lectura extra para obtener el dato. El estudiante preferiría el direccionamiento directo cuando el valor puede cambiar en tiempo de ejecución, como el 0x5678 escrito previamente con E en DS:0300, porque la instrucción lee el contenido actual de esa posición sin necesidad de reensamblar el programa; el modo inmediato solo sirve para constantes conocidas al ensamblar. Con el comando U, sin ejecutar ninguna instrucción, se distingue la codificación: B8 con el valor sin corchetes indica inmediato, y A1 con la dirección entre corchetes indica directo a memoria.

### Formato little-endian

En los valores de 16 bits, el byte menos significativo se almacena primero. Por ejemplo, el inmediato 0x0005 se guarda como 05 00, la dirección 0x0300 como 00 03 y los bytes 78 56 en DS:0300 se leen como la palabra 0x5678. Esto se comprobó al ejecutar MOV AX,[0300] y obtener AX = 5678.

## Parte 2

### Checkpoint 1: Traza del programa de suma

![Traza de suma](capturas//parte%202/CP1_traza_suma.png)

| Instrucción | AX   | BX   | CX   | IP siguiente | ZF  | CF  | SF  |
| ----------- | ---- | ---- | ---- | ------------ | --- | --- | --- |
| MOV AX,000A | 000A | 0000 | 0000 | 0103         | 0   | 0   | 0   |
| MOV BX,0005 | 000A | 0005 | 0000 | 0106         | 0   | 0   | 0   |
| MOV CX,0003 | 000A | 0005 | 0003 | 0109         | 0   | 0   | 0   |
| ADD AX,BX   | 000F | 0005 | 0003 | 010B         | 0   | 0   | 0   |
| ADD AX,CX   | 0012 | 0005 | 0003 | 010D         | 0   | 0   | 0   |

Se ejecutó el programa con `T` instrucción por instrucción. AX terminó valiendo 0012 (18 decimal) después de las dos instrucciones ADD, que es el resultado esperado de 10 + 5 + 3.

### Checkpoint 2: Traza del bucle con LOOP

![Traza LOOP](capturas/parte%202/CP2_traza_loop.png)

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿LOOP salta? |
| --------- | ----------- | ---------- | ---------- | ------------ | ------------ |
| –         | MOV AX,0    | 0000       | –          | 0103         | –            |
| –         | MOV CX,4    | 0000       | 0004       | 0106         | –            |
| 1         | ADD AX,2    | 0002       | 0004       | 0109         | –            |
| 1         | LOOP        | 0002       | 0003       | 0106         | Sí           |
| 2         | ADD / LOOP  | 0004       | 0002       | 0106         | Sí           |
| 3         | ADD / LOOP  | 0006       | 0001       | 0106         | Sí           |
| 4         | ADD         | 0008       | 0001       | 0109         | –            |
| 4         | LOOP        | 0008       | 0000       | 010B         | No           |
| –         | INT 20      | –          | –          | –            | –            |

CX disminuyó de 4 a 0 y AX acumuló 2 en cada iteración hasta valer 0008. Cuando CX llegó a 0, LOOP no saltó y el programa continuó con INT 20. Se ejecutaron 11 instrucciones con T: 2 de inicialización, 4 pares ADD y LOOP, y la terminación. En el ensamblado, `LOOP 0106` se codificó como `E2 FB`, donde FB (−5) es el desplazamiento relativo firmado.

#### Análisis del código máquina (Paso 8)

El comando `D CS:100 L0D` mostró los 13 bytes del programa con bucle:
`B8 00 00 B9 04 00 05 02 00 E2 FB CD 20`

| Instrucción | Bytes    | Tamaño  |
| ----------- | -------- | ------- |
| MOV AX,0000 | B8 00 00 | 3 bytes |
| MOV CX,0004 | B9 04 00 | 3 bytes |
| ADD AX,0002 | 05 02 00 | 3 bytes |
| LOOP 0106   | E2 FB    | 2 bytes |
| INT 20      | CD 20    | 2 bytes |

Las instrucciones con operando inmediato de 16 bits ocupan 3 bytes (opcode más dos bytes en little-endian), mientras que LOOP e INT 20 ocupan 2. El total es de 13 bytes. En LOOP, E2 es el opcode y FB es el desplazamiento relativo con signo: -5, obtenido de 0106 menos 010B, que es la dirección de la instrucción siguiente.

### Checkpoint 3: Bucle equivalente con DEC/JNZ

![Traza DEC/JNZ](capturas/parte%202/CP3_traza_dec_jnz.png)

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿JNZ salta? |
| --------- | ----------- | ---------- | ---------- | ------------ | ----------- |
| –         | MOV AX,0000 | 0000       | –          | 0203         | –           |
| –         | MOV CX,0004 | 0000       | 0004       | 0206         | –           |
| 1         | ADD AX,0002 | 0002       | 0004       | 0209         | –           |
| 1         | DEC CX      | 0002       | 0003       | 020A         | –           |
| 1         | JNZ 0206    | 0002       | 0003       | 0206         | Sí (ZF=0)   |
| 2         | ADD AX,0002 | 0004       | 0003       | 0209         | –           |
| 2         | DEC CX      | 0004       | 0002       | 020A         | –           |
| 2         | JNZ 0206    | 0004       | 0002       | 0206         | Sí (ZF=0)   |
| 3         | ADD AX,0002 | 0006       | 0002       | 0209         | –           |
| 3         | DEC CX      | 0006       | 0001       | 020A         | –           |
| 3         | JNZ 0206    | 0006       | 0001       | 0206         | Sí (ZF=0)   |
| 4         | ADD AX,0002 | 0008       | 0001       | 0209         | –           |
| 4         | DEC CX      | 0008       | 0000       | 020A         | –           |
| 4         | JNZ 0206    | 0008       | 0000       | 020C         | No (ZF=1)   |
| –         | INT 20      | 0008       | 0000       | –            | –           |

#### Comparación de instrucciones ejecutadas

| Versión | Bytes del control del bucle | Invocaciones de T |
| ------- | --------------------------- | ----------------- |
| LOOP    | 2 (`E2 FB`)                 | 11                |
| DEC/JNZ | 3 (`49` + `75 FA`)          | 15                |

Ambas versiones terminaron con AX = 0008. La versión con DEC/JNZ ejecuta cuatro instrucciones más, una por iteración. Con `G 20C` se obtuvo el mismo resultado final sin pasos intermedios.

### Decisión técnica 3: LOOP frente a DEC/JNZ (Paso 10)

Para un bucle contador simple en el que CX no se usa con otro propósito, se recomienda LOOP. En bytes de código máquina, LOOP 0106 ocupa 2 bytes (E2 FB), mientras que DEC CX y JNZ 0206 ocupan 3 (49 y 75 FA). En instrucciones por iteración, LOOP requiere extraer dos (ADD y LOOP) y DEC/JNZ tres (ADD, DEC y JNZ), lo que se observó en las trazas: 11 invocaciones de T frente a 15. Por tanto, LOOP reduce el tamaño del programa y el número de fetch por iteración. DEC/JNZ sería preferible si el cuerpo del bucle necesitara reutilizar CX en otra operación aritmética, o si la salida dependiera de una condición evaluada con CMP y no solo de que CX sea distinto de cero. Además, DEC actualiza banderas como ZF, mientras que LOOP no modifica ninguna. Con U, sin ejecutar nada, se compara el tamaño: U 100 10D muestra E2FB (2 bytes) y U 200 20D muestra 49 y 75FA (3 bytes) para el mismo control de bucle.

### Decisión técnica 4: T frente a G (Paso 12)

Con 100 iteraciones, G 20C es más práctico que T porque ejecuta el programa a velocidad completa hasta la dirección 020C con un solo comando, mientras que T exigiría más de 300 invocaciones manuales en el bucle DEC/JNZ (2 de inicialización, 100 × 3 instrucciones y 1 de terminación). Al usar G se pierde la información intermedia: no se observan los valores de AX, CX, IP ni de las banderas después de cada instrucción, solo el estado final en el punto de parada. T fue indispensable en los Pasos 4, 7 y 11, porque las tablas de traza exigen registrar el estado tras cada instrucción, y esa granularidad solo la ofrece T. Después de G 20C, el comando R muestra los registros y permite confirmar que AX vale 0008 sin haber visto ningún paso intermedio.

## Conclusiones

El laboratorio permitió observar cómo el procesador interpreta los bytes en memoria como instrucciones, y cómo cada mnemónico se traduce a un código máquina concreto: los inmediatos de 16 bits se almacenan en little-endian y los saltos cortos usan desplazamientos relativos de 8 bits con signo. Se comprobó que los comandos de DEBUG tienen propósitos distintos: D y U solo leen, mientras que E, F y A escriben, por lo que la elección del comando adecuado evita modificar el estado por accidente. La comparación entre LOOP y DEC/JNZ mostró que ambos son equivalentes en resultado (AX = 0008), pero LOOP es más compacto (2 bytes frente a 3) y ejecuta menos instrucciones (11 frente a 15), mientras que DEC/JNZ ofrece más flexibilidad cuando se necesitan banderas o condiciones con CMP. Por último, se concluyó que T y G son complementarios: T permite analizar el comportamiento paso a paso y G verifica rápidamente el resultado final de programas con muchas iteraciones.
