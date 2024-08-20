# Documentación de la Implementación de una ALU de Tres Entradas en nand2tetris

## Introducción

En el contexto del curso nand2tetris, se ha implementado una Unidad Aritmético-Lógica (ALU) capaz de operar sobre tres entradas de 16 bits: X, Y, y Z. Esta ALU extiende las funcionalidades de una ALU tradicional de dos entradas para soportar operaciones adicionales, lo que permite mayor flexibilidad y capacidad de procesamiento.

Las operaciones están controladas por diversos bits de control (`zx`, `nx`, `zy`, `ny`, `f`, `no`, `sel`) que permiten configurar las entradas, seleccionar la operación deseada, y manipular el resultado. Además, se incluyen indicadores de estado (`zr`, `ng`) que permiten identificar condiciones específicas del resultado, como si es cero o negativo.

## Descripción Técnica de las Variables

- **X, Y, Z**: Son las entradas principales de la ALU, cada una de 16 bits. Estas pueden contener cualquier valor dentro del rango permitido por los 16 bits.
- **zx, zy**: Estos bits de control permiten establecer las entradas X o Y a cero. Si `zx=1`, X se iguala a 0, y lo mismo aplica para `zy` respecto a Y.
- **nx, ny**: Permiten negar (invertir) las entradas X o Y. Si `nx=1`, X se invierte, y lo mismo aplica para `ny` respecto a Y.
- **f**: Controla la operación aritmético-lógica a realizar. Si `f=1`, la ALU suma las entradas seleccionadas. Si `f=0`, realiza una operación AND bit a bit.
- **no**: Este bit de control permite negar (invertir) la salida final de la ALU.
- **sel[2 bits]**: Este selector determina si la ALU debe sumar, restar o no hacer ninguna operación con la entrada Z a la operación entre X e Y. El bit más significativo de `sel` define la operación: `sel=10` para restar Z y `sel=01` para sumar Z.
- **zr**: Indicador que se activa (es igual a 1) si todos los bits de la salida de la ALU son cero.
- **ng**: Indicador que se activa si la salida es negativa.

## Implementación y Estructura ALU #1

### ALU General



## Implementación y Estructura ALU #2

### ALU General

Esta es la unidad principal que encapsula dos sub-ALUs: **ALUxy** y **ALUxyz**. Su responsabilidad es coordinar las operaciones sobre las tres entradas X, Y, y Z, utilizando los bits de control mencionados anteriormente.

### ALUxy

Esta sub-ALU se encarga de procesar las operaciones relacionadas con las entradas X e Y. Aplica las configuraciones de `zx`, `nx`, `zy`, y `ny`, y realiza la operación aritmética o lógica indicada por `f` y `no`. La salida de esta sub-ALU es un valor intermedio que será utilizado por la **ALUxyz**.

#### Proceso Interno de ALUxy:
1. **Negación de Entradas**: Se preparan las versiones negadas de X e Y utilizando puertas NOT, listas para ser seleccionadas si `nx` o `ny` son activos.
2. **Operaciones AND y ADD**: Se calculan ambas operaciones de manera predeterminada. Luego, un multiplexor (MUX) selecciona cuál será la salida en función del bit `f`.
3. **Negación de la Salida**: Si `no` es activo, la salida seleccionada es invertida.

### ALUxyz

Esta sub-ALU toma la salida de **ALUxy** y la combina con la entrada Z. Aquí se decide si se niega la salida de **ALUxy** y si se sumará o restará Z, dependiendo del valor de `sel`. Además, se activan los indicadores de estado `zr` y `ng`.

#### Proceso Interno de ALUxyz:
1. **Negación Condicional de XY**: Dependiendo del bit `xyn`, se selecciona entre la salida directa o la negada de **ALUxy**.
2. **Operación Aritmética (ADD)**: Se suma la salida de **ALUxy** con Z.
3. **Negación y Selección de la Salida**: Si `no` es activo, se niega la salida del ADD. Un MUX selecciona el resultado final que será la salida de la ALU.
4. **Indicadores de Estado**: Se verifican las condiciones de la salida para activar `zr` si es cero y `ng` si es negativa.

### Ventajas y Desventajas de la Implementación

#### Ventajas
1. **Reducción de Compuertas**: La segunda implementación, que utiliza sub-ALUs encapsuladas, reduce la cantidad de compuertas y elimina redundancias al segmentar las operaciones.
2. **Modularidad y Escalabilidad**: La división en sub-ALUs facilita la identificación de errores y mejora la capacidad de ampliación futura de la ALU, permitiendo una adaptación más fácil a operaciones más complejas o entradas adicionales.
3. **Claridad en el Diseño**: La segmentación por grupos de operaciones (XY y XYZ) permite una mejor organización del flujo de datos y procesamiento, lo que simplifica el seguimiento del estado y la depuración.

#### Desventajas
1. **Complejidad Inicial**: La implementación modular, aunque ventajosa a largo plazo, introduce complejidad adicional en las primeras etapas de desarrollo y puede requerir más tiempo para su diseño y prueba inicial.
2. **Consumo de Recursos**: A pesar de la reducción en el número de compuertas, la introducción de múltiples ALUs y MUX adicionales puede aumentar el consumo de recursos en términos de área en un chip o FPGA.
3. **Posible Latencia**: La segmentación de operaciones a través de múltiples sub-ALUs podría incrementar la latencia en el procesamiento de ciertas operaciones, lo que podría afectar el rendimiento en aplicaciones donde la velocidad es crítica.

### Conclusión

Esta implementación de la ALU de tres entradas en nand2tetris demuestra un enfoque eficiente y modular para manejar operaciones aritméticas y lógicas complejas. La organización en sub-ALUs no solo optimiza el uso de compuertas, sino que también prepara el camino para futuras expansiones y adaptaciones, a pesar de las posibles complicaciones iniciales y el aumento en la latencia.
