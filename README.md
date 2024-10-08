# Documentación de la Implementación de ALUs de Tres Entradas en nand2tetris

## Introducción

En el contexto del curso nand2tetris, se han implementado dos Unidades Aritmético-Lógicas (ALUs) capaz de operar sobre tres entradas de 16 bits: X, Y, y Z. Estas ALUs extienden las funcionalidades de una ALU tradicional de dos entradas para soportar operaciones adicionales, lo que permite mayor flexibilidad y capacidad de procesamiento.

Las operaciones están controladas por diversos bits de control (`zx`, `nx`, `zy`, `ny`, `f`, `no`, `sel`) que permiten configurar las entradas, seleccionar la operación deseada, y manipular el resultado. Además, se incluyen indicadores de estado (`zr`, `ng`) que permiten identificar condiciones específicas del resultado, como si es cero o negativo.

## Descripción Técnica de las Variables

- **X, Y, Z**: Son las entradas principales de la ALU, cada una de 16 bits. Estas pueden contener cualquier valor dentro del rango permitido por los 16 bits.
- **zx, zy**: Estos bits de control permiten establecer las entradas X o Y a cero. Si `zx=1`, X se iguala a 0, y lo mismo aplica para `zy` respecto a Y.
- **nx, ny**: Permiten negar (invertir) el resultado que proporciona aplicar el bit de control `zx` o `zy`. Si `nx=1`, X se invierte, y lo mismo aplica para `ny` respecto a Y (siendo X e Y el resultado que se proporciona al hacer la operación con `zx` ó `zy`).
- **f**: Controla la operación aritmético-lógica a realizar. Si `f=1`, la ALU suma las entradas seleccionadas. Si `f=0`, realiza una operación AND bit a bit.
- **no**: Este bit de control permite negar (invertir) la salida final de la ALU proporcionada según el resultado de aplicar el bit de control `f` anteriormente.
- **sel[2 bits]**: Este selector determina si la ALU debe sumar, restar o no hacer ninguna operación con la entrada Z a la operación entre X e Y. El bit más significativo de `sel` define la operación: `sel=10` para restar Z y `sel=01` para sumar Z. Mientras que si `sel=00` no se realiza ninguna operación.
- **zr**: Indicador que se activa (es igual a 1) si todos los bits de la salida de la ALU son cero.
- **ng**: Indicador que se activa si la salida es negativa.

## Tabla de operaciones

![image](https://i.imgur.com/zHi1Vh8.png)

## Implementación y Estructura ALU #1

### ALU General
Para esta implementación, lo primero llevado a cabo fué el diseño inicial de la ALU con sus nuevas entradas, como se puede ver a continuacion:

![image](https://i.imgur.com/Mz0itqv.jpeg)

Como se puede observar, se definió la ALU con su selector de dos bits y su entrada `z` explicados anteriormente. Esta versión extendida del modelo clásico de la ALU busca permitir operaciones adicionales gracias a estos dos nuevos inputs (suma con `z`, resta con `z`, o dejar la salida sin operar con `z`).

### Proceso interno de la ALU

![image](https://i.imgur.com/xd0Gp65.png)

1. **Cambio de Entradas a 0's:** Se preparan buses de 16 bits con ceros para las entradas `X` e `Y`. Estos buses están listos para ser seleccionados si los bits de control `zx` o `zy` están activos, forzando a las entradas `X` o `Y` a cero respectivamente.

2. **Negación de Entradas:** Se preparan las versiones negadas de las entradas `X` e `Y` utilizando compuertas `NOT`. Estas versiones negadas estarán listas para ser seleccionadas si los bits de control nx o ny están activos, invirtiendo los valores de `X` e `Y` antes de la operación.

3. **Operaciones AND y ADD:** Se calculan ambas operaciones (`AND` y `ADD`) de manera predeterminada con las entradas `X` e `Y` ya modificadas según los bits de control aplicados en los pasos anteriores. Luego, un multiplexor (`MUX`) selecciona cuál será la salida en función del bit `f`. Si `f` está activo, se selecciona la `ADD`; si no, se selecciona la operación `AND`.

4. **Negación de la Salida:** Si el bit de control no está activo, la salida seleccionada (proporcionada según el resultado de aplicar o no el bit `f`) es invertida utilizando una puerta `NOT`.

5. **Operaciones con Z (Suma, Resta, o Sin Cambios):** El resultado de la operación entre `X` e `Y` se pasa al CHIP `OPZ`, que también recibe la entrada `Z`. Se calculan tanto la suma como la resta entre el resultado de `X` e `Y` y la entrada `Z`. Estos dos resultados, junto con la opción de no modificar el resultado con `Z`, se envían al CHIP `OPZ`, que funciona como un multiplexor de 3 entradas. El selector de 2 bits (Sel[2]) determina cuál de las tres posibles salidas se seleccionará.

6. **Indicadores de Estado:** Finalmente, se verifican las condiciones de la salida proporcionada por el CHIP `OPZ`. Si la salida es cero, se activa el indicador `zr`. Si la salida es negativa, se activa el indicador `ng`. Estos indicadores de estado son útiles para operaciones posteriores o para determinar la lógica de control en el sistema.

Este flujo asegura que la ALU pueda realizar operaciones complejas con tres entradas (`X`, `Y`, y `Z`) y aplicar las transformaciones necesarias según los bits de control proporcionados.

### Detalle de implementacion para operaciones con el tercer registro (CHIP OPZ) 

![image](https://i.imgur.com/xdg5Az3.png)

1. **Selección de Operación:** Se utiliza una compuerta OR para determinar si se realizará una operación con `Z` (ya sea suma o resta) o si no se realizará ninguna operación con `Z`. Esta compuerta OR combina las señales de control y define si `Z` se incluirá en el cálculo.

2. **Selección entre Suma, Resta o Sin Operación:**
- Para seleccionar la resta, se utiliza una compuerta `AND` que combina el bit `Sel[1]` con el resultado del `OR` anterior, lo que activa la resta si ambas condiciones son verdaderas.
- Para seleccionar la suma, se utiliza otra compuerta `AND` que combina el bit `Sel[0]` y el resultado del `OR`.
- Si no se realiza ninguna operación con `Z`, se utiliza una compuerta `NOR` que toma el resultado del `OR` y el bit `Sel[0]`. Este `NOR` asegura que la salida no sea modificada cuando no se selecciona `Z` para la operación.
- Entre estas tres compuertas (`AND`, `AND`, y `NOR`), solo una de ellas producirá un 1, determinando cuál operación debe salir del "multiplexor" de 3 entradas (como se comporta el CHIP `OPZ`).

3. **Selección del Output:**
- Cada bit del resultado (suma, resta o sin cambios) (16 bits en total) pasa por una compuerta `AND161` junto al resultado de la compuerta anterior que determinaba la operación a realizar.
- Solo la compuerta `AND` o `NOR` seleccionada en el paso anterior (que permitó el paso de un 1) permitirá pasar el bus de 16 bits de la operación correspondiente en la compuerta `AND161`, mientras que las otras compuertas, al realizar `ANDs` con 0s, darán un bus de 16 bits en 0s.

4. **Unificación mediante ORs:**
- Finalmente, se realiza un `OR` entre las salidas de los buses de 16 bits.
- Como los buses que no fueron seleccionados tienen valores de 0s, el resultado final será el bus de 16 bits con la operación seleccionada, que se enviará como salida del CHIP `OPZ`.

El CHIP `OPZ` actúa como un multiplexor de 3 entradas. Dado un selector de dos bits y las tres entradas (suma, resta, sin operación), decide cuál debe sacar por su salida (output). Esta lógica permite una operación eficiente y modular, integrando la tercera entrada `Z` en las operaciones de la ALU.


### Ventajas y Desventajas de la Implementación

#### Ventajas

- **Modularidad del Diseño:** El uso del CHIP `OPZ` como un multiplexor de 3 entradas permite una implementación modular, facilitando el mantenimiento y la expansión del diseño en el futuro. La lógica está separada en componentes claros y manejables, lo que hace que sea más sencillo modificar o añadir nuevas funciones si es necesario. Esto facilita la vista del circuito en general.

- **Flexibilidad Aumentada:** La inclusión de una tercera entrada `Z`, junto con un selector de dos bits, amplía significativamente las capacidades de la ALU. Esto permite realizar operaciones aritméticas más complejas como sumas y restas con tres operandos, lo que es útil en aplicaciones más avanzadas.

#### Desventajas

- **Poca Homogeneidad del Diseño de las Operaciones con Z con Respecto a la Implementación Clásica:** Para esta implementación, dejamos de lado, para operaciones con `z`, la estructura clasica de la ALU de realizar operaciones a través de sus bits de control, lo que puede dificultar el entendimiento de la implementación ya que el proceso de aprendizaje debe hacerse desde 0, entendiendo el funcionamiento del circuito y dejando de lado la teoría que se tiene sobre como opera la ALU para su tercer registro.
- **Posible Penalización en el Rendimiento:** La necesidad de seleccionar entre tres operaciones diferentes a través de múltiples compuertas (9 compuertas adicionales `AND`, `OR`, `NOR` + las dos necesarias para calcular la suma y la resta con `z` pasadas al CHIP `OPZ`) puede introducir retardos adicionales en el proceso de cálculo. Esto podría impactar el rendimiento en aplicaciones que requieren alta velocidad de procesamiento. Esto sería una desventaja con respecto a otras implementaciones que utilicen menos compuertas para añadir la nueva funcionalidad de la ALU con su tercer registro.
- **Consumo de Energía Incrementado:** Muy relacionado a lo mencionado anteriormente, al tener más componentes activos (las 9 adicionales para el manejo del tercer circuito, y las 2 que hacen el calculo de la resta y la suma para pasar al CHIP `OPZ`), es probable que el consumo de energía del circuito sea mayor en comparación con una ALU tradicional y las diferentes implementaciones de la ALU con sus nuevas funcionaldiades. Esto puede ser una desventaja en sistemas donde la eficiencia energética es crítica. 




## Implementación y Estructura ALU #2

### ALU General

![image](https://github.com/user-attachments/assets/97d06f81-f423-4ae7-958b-21ba6b4ff988)

Esta es la unidad principal que encapsula dos sub-ALUs: **ALUxy** y **ALUxyz**. Su responsabilidad es coordinar las operaciones sobre las tres entradas X, Y, y Z, utilizando los bits de control mencionados anteriormente.

### ALUxy

Esta sub-ALU se encarga de procesar las operaciones relacionadas con las entradas X e Y. Aplica las configuraciones de `zx`, `nx`, `zy`, y `ny`, y realiza la operación aritmética o lógica indicada por `f` y `no`. La salida de esta sub-ALU es un valor intermedio que será utilizado por la **ALUxyz**.

![image](https://github.com/user-attachments/assets/1386d3b7-1821-4634-8339-97c51a181839)

#### Proceso Interno de ALUxy:

1. **Función de false en el Mux**: Cuando se introduce false como una de las entradas en el Mux16, esto representa un valor constante de 16 bits de 0. Esencialmente, esto permite que, si el selector (zx o zy) es 1, la salida del Mux sea un valor de 0 para esa entrada (X o Y).
2. **Uso de Not16**: Posteriormente, se pasa este valor a un Not16 que invierte todos los bits de la salida. Por lo tanto, si el selector es 1, se pasa un valor de 0 que luego se niega para obtener 1111...1111, lo que permite manejar de manera efectiva los bits en la operación posterior.
3. **Negación de Entradas**: Se preparan las versiones negadas de X e Y utilizando puertas NOT, listas para ser seleccionadas si `nx` o `ny` son activos.
4. **Operaciones AND y ADD**: Se calculan ambas operaciones de manera predeterminada. Luego, un multiplexor (MUX) selecciona cuál será la salida en función del bit `f`.
5. **Negación de la Salida**: Si `no` es activo, la salida seleccionada es invertida.

### Detalle de implementacion para operaciones con el tercer registro (ALUxyz)

Esta sub-ALU toma la salida de **ALUxy** y la combina con la entrada Z. Aquí se decide si se niega la salida de **ALUxy** y si se sumará o restará Z, dependiendo del valor de `sel`. Además, se activan los indicadores de estado `zr` y `ng`.

![image](https://i.imgur.com/JS48CsD.png)

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
2. **Consumo de Recursos**: A pesar de la reducción en el número de compuertas, la introducción de múltiples ALUs y MUX adicionales puede aumentar el consumo de recursos en términos de área en un CHIP o FPGA.
3. **Posible Tiempo de procesamiento Mayor**: La segmentación de operaciones a través de múltiples sub-ALUs podría incrementar el tiempo de procesamiento en el procesamiento de ciertas operaciones, lo que podría afectar el rendimiento en aplicaciones donde la velocidad es crítica.


## Análisis de Ventajas y Desventajas de Incluir el Tercer Registro
### Ventajas
1. **Mayor Capacidad y Flexibilidad Operacional**: La inclusión del tercer registro (Z) permite realizar operaciones aritméticas más complejas, como sumas y restas con tres operandos, que no serían posibles en una ALU tradicional de dos entradas. Esto incrementa la capacidad de procesamiento y la versatilidad del circuito, siendo especialmente útil en aplicaciones avanzadas donde es necesario combinar múltiples valores de manera simultánea.
2. **Menor Acceso a Memoria**: Al permitir operaciones directas con tres operandos, se reduce la necesidad de acceder a la memoria para cargar o almacenar resultados intermedios, lo que puede mejorar la eficiencia y el rendimiento.
3. **Optimización de Código**: Una ALU de mayor capacidad puede facilitar la creación de circuitos más eficientes al reducir la cantidad de instrucciones necesarias para realizar operaciones complejas. Esto permite que el código sea más simple y directo.

### Desventajas
1. **Complejidad Aumentada en el Diseño**: La inclusión de un tercer registro introduce una complejidad adicional en el diseño de la ALU. Esto se traduce en un aumento en el número de compuertas lógicas necesarias, como AND, OR, y NOR, que gestionan las operaciones con el tercer registro. Esta complejidad puede dificultar el aprendizaje y la comprensión del funcionamiento de la ALU.
2. **Incompatibilidad con Diseños Clásicos**: La introducción del tercer registro puede desalinearse con los principios de diseño tradicionales de las ALUs, haciendo que el circuito sea menos intuitivo para quienes están familiarizados con implementaciones más simples. 
3. **Rendimiento Potencialmente Afectado**: Debido a la necesidad de realizar una mayor cantidad de operaciones, se podría incrementar el tiempo de procesamiento en el proceso de cálculo.
   
### Conclusión
La inclusión de un tercer registro en la ALU ofrece ventajas significativas al permitir operaciones aritméticas más complejas, reducir la necesidad de acceso a memoria y optimizar el código, lo que se traduce en un procesamiento más eficiente y versátil. Sin embargo, este diseño también conlleva desventajas, como una mayor complejidad en el diseño del circuito, potencial incompatibilidad con enfoques tradicionales y un posible incremento en el tiempo de procesamiento del proceso de cálculo.


## ¿Qué Implementación es Mejor?
### Comparativa entre ambas implementaciones
|  Aspecto  | Implementación 1 | Implementación 2 |
| --------- | ---------------- | ---------------- |
| Cantidad de Compuertas | 25  | 21 |
| Modularidad | Menor modularidad, mayor complejidad en la gestión del tercer registro (Z) | Mayor modularidad, con sub-ALUs separadas (ALUxy y ALUxyz) que facilitan el diseño y mantenimiento |
| Flexibilidad | Permite operaciones con Z pero con mayor complejidad en el diseño del CHIP OPZ | Mayor flexibilidad con la segmentación en sub-ALUs que facilita la integración de operaciones más complejas |
| Rendimiento | Puede experimentar una disminución en el rendimiento debido al mayor número de compuertas adicionales necesarias para seleccionar las operaciones. | La reducción en el número de compuertas y la segmentación en sub-ALUs pueden mejorar el rendimiento general, aunque la segmentación podría tener un impacto menor en la velocidad de procesamiento |
| Consumo de Energía | Incrementado debido al mayor número de compuertas y operaciones adicionales | Puede ser más eficiente en comparación, pero aún puede consumir recursos adicionales por las múltiples sub-ALUs y MUX |
| Claridad del Diseño | Menor claridad debido a la complejidad de la implementación con el CHIP OPZ y múltiples compuertas | Mayor claridad con un diseño modular y segmentado en sub-ALUs que simplifica el seguimiento y la depuración |


### Conclusión
La Implementación #2 es preferible debido a su mayor modularidad, claridad en el diseño, y mejor manejo de la complejidad mediante la segmentación en sub-ALUs. La segmentación podría afectar ligeramente el tiempo de procesamiento, pero su capacidad para reducir el número de compuertas y facilitar el mantenimiento y expansión hace que sea más adecuada para aplicaciones que requieran una ALU con un tercer registro. Además, su consumo de energía es menor al ser más eficiente en el uso de recursos.

## Conclusión General

- Agregar una nueva funcionalidad a la ALU implica complejidad en el diseño del circuito, un probable tiempo de procesamiento mayor y un consumo de energia mayor, por lo tanto, se debe buscar dar implementaciones lo más eficiente posible desde los tres aspectos mencionados anteriormente.

- El "Encapsulamiento de ALUs" es una implemetación modular para manejar operaciones aritméticas y lógicas complejas. La organización en sub-ALUs no solo optimiza el uso de compuertas, sino que también prepara el camino para futuras expansiones y adaptaciones, a pesar de las posibles complicaciones iniciales y el aumento en el tiempo de procesamiento.
