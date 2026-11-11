## Postcontenido 1 Unidad 3

### Checkpoint 1: Estado inicial de los registros

![Estado de registros](capturas/CP1_registros.png)

Al ejecutar el comando `R` sin argumentos, se observó que los registros de propósito general (AX, BX, CX y DX) valen 0000 y que SP vale FFFE, el tope inicial de la pila. Los cuatro registros de segmento (DS, ES, SS y CS) tienen el mismo valor, porque DOS asigna el segmento del PSP al programa. El IP vale 0100, la primera dirección ejecutable después del PSP. Esta captura corresponde a la sesión en `C:\LAB3PT1>`.

### Checkpoint 2: Volcado hexadecimal

![Volcado de memoria](capturas/CP2_volcado_memoria.png)

Se rellenaron 64 bytes desde DS:0200 con el patrón AB CD EF usando `F 200 L40 AB CD EF` y se volcaron con `D 200 L40`. La salida del comando `D` tiene tres columnas. La primera muestra la dirección en formato segmento:desplazamiento. La segunda muestra 16 bytes por fila en hexadecimal, separados en dos grupos de 8 por un guion. La tercera muestra la representación ASCII de esos bytes. En esta captura la columna ASCII solo muestra puntos, porque AB, CD y EF no son caracteres imprimibles (el rango imprimible es 0x20 a 0x7E). El patrón se repite cíclicamente en las cuatro filas.
