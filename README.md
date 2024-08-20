# Documentación de la Implementación de ALUs de Tres Entradas en nand2tetris

## Introducción

En el contexto del curso nand2tetris, se han implementado dos Unidades Aritmético-Lógicas (ALUs) capaz de operar sobre tres entradas de 16 bits: X, Y, y Z. Estas ALUs extiende las funcionalidades de una ALU tradicional de dos entradas para soportar operaciones adicionales, lo que permite mayor flexibilidad y capacidad de procesamiento.

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

Como se puede observar, se definió la ALU con su selector de dos bits y su entrada z explicados anteriormente. Esta version extendida del modelo clasico de la ALU busca permitir operaciones adicionales gracias a estos dos nuevos inputs (suma con z, resta con z, o dejar la salida sin operar con z).

### Proceso interno de la ALU

![image](https://i.imgur.com/xd0Gp65.png)

1. **Cambio de Entradas a 0's:** Se preparan buses de 16 bits con ceros para las entradas X e Y. Estos buses están listos para ser seleccionados si los bits de control zx o zy están activos, forzando a las entradas X o Y a cero respectivamente.

2. **Negación de Entradas:** Se preparan las versiones negadas de las entradas X e Y utilizando puertas NOT. Estas versiones negadas estarán listas para ser seleccionadas si los bits de control nx o ny están activos, invirtiendo los valores de X e Y antes de la operación.

3. **Operaciones AND y ADD:** Se calculan ambas operaciones (AND y suma) de manera predeterminada con las entradas X e Y ya modificadas según los bits de control aplicados en los pasos anteriores. Luego, un multiplexor (MUX) selecciona cuál será la salida en función del bit f. Si f está activo, se selecciona la suma; si no, se selecciona la operación AND.

4. **Negación de la Salida:** Si el bit de control no está activo, la salida seleccionada (proporcionada según el resultado de aplicar o no el bit f) es invertida utilizando una puerta NOT.

5. **Operaciones con Z (Suma, Resta, o Sin Cambios):** El resultado de la operación entre X e Y se pasa al CHIP OPZ, que también recibe la entrada Z. Se calculan tanto la suma como la resta entre el resultado de X e Y y la entrada Z. Estos dos resultados, junto con la opción de no modificar el resultado con Z, se envían al CHIP OPZ, que funciona como un multiplexor de 3 entradas. El selector de 2 bits (Sel[2]) determina cuál de las tres posibles salidas se seleccionará.

6. **Indicadores de Estado:** Finalmente, se verifican las condiciones de la salida proporcionada por el CHIP OPZ. Si la salida es cero, se activa el indicador zr. Si la salida es negativa, se activa el indicador ng. Estos indicadores de estado son útiles para operaciones posteriores o para determinar la lógica de control en el sistema.

Este flujo asegura que la ALU pueda realizar operaciones complejas con tres entradas (X, Y, y Z) y aplicar las transformaciones necesarias según los bits de control proporcionados.

### Detalle de implementacion para operaciones con el tercer registro (CHIP OPZ) 

![image](https://i.imgur.com/xdg5Az3.png)

1. **Selección de Operación:** Se utiliza una compuerta OR para determinar si se realizará una operación con Z (ya sea suma o resta) o si no se realizará ninguna operación con Z. Esta compuerta OR combina las señales de control y define si Z se incluirá en el cálculo.

2. **Selección entre Suma, Resta o Sin Operación:**
- Para seleccionar la resta, se utiliza una compuerta AND que combina el bit Sel[1] con el resultado del OR anterior, lo que activa la resta si ambas condiciones son verdaderas.
- Para seleccionar la suma, se utiliza otra compuerta AND que también combina el bit Sel[1] y el resultado del OR.
- Si no se realiza ninguna operación con Z, se utiliza una compuerta NOR que toma el resultado del OR y el bit Sel[0]. Este NOR asegura que la salida no sea modificada cuando no se selecciona Z para la operación.
- Entre estas tres compuertas (AND, AND, y NOR), solo una de ellas producirá un 1, determinando cuál operación debe salir del "multiplexor" de 3 entradas (como se comporta el CHIP OPZ).

3. **Selección del Output:**
- Cada bit del resultado (16 bits en total) pasa por una compuerta AND.
- Solo la compuerta AND seleccionada en el paso anterior permitirá pasar los valores correctos, mientras que las otras compuertas, al realizar ANDs con 0, darán un bus de 16 bits en 0s.

4. **Unificación mediante ORs:**
- Finalmente, se realiza un OR entre las salidas de los buses de 16 bits.
- Como los buses que no fueron seleccionados tienen valores de 0s, el resultado final será el bus de 16 bits con la operación seleccionada, que se enviará como salida del CHIP OPZ.

El CHIP OPZ actúa como un multiplexor de 3 entradas. Dado un selector de dos bits y las tres entradas (suma, resta, sin operación), decide cuál debe sacar por su salida (output). Esta lógica permite una operación eficiente y modular, integrando la tercera entrada Z en las operaciones de la ALU.


### Ventajas y Desventajas de la Implementación

#### Ventajas

- **Modularidad del Diseño:** El uso del CHIP OPZ como un multiplexor de 3 entradas permite una implementación modular, facilitando el mantenimiento y la expansión del diseño en el futuro. La lógica está separada en componentes claros y manejables, lo que hace que sea más sencillo modificar o añadir nuevas funciones si es necesario. Esto facilita la vista del circuito en general.

- **Flexibilidad Aumentada:** La inclusión de una tercera entrada Z, junto con un selector de dos bits, amplía significativamente las capacidades de la ALU. Esto permite realizar operaciones aritméticas más complejas como sumas y restas con tres operandos, lo que es útil en aplicaciones más avanzadas.

#### Desventajas

- **Poca Homogeneidad del Diseño de las Operaciones con Z con Respecto a la Implementación Clásica:** Para esta implementación, dejamos de lado, para operaciones con z, la estructura clasica de la ALU de realizar operaciones a través de sus bits de control, lo que puede dificultar el entendimiento de la implementación ya que el proceso de aprendizaje debe hacerse desde 0, entendiendo el funcionamiento del circuito y dejando de lado la teoría que se tiene sobre como opera la ALU para su tercer registro.
- **Posible Penalización en el Rendimiento:** La necesidad de seleccionar entre tres operaciones diferentes a través de múltiples compuertas (9 compuertas adicionales AND, OR, NOR + las dos necesarias para calcular la suma y la resta con z pasadas al CHIP OPZ) puede introducir retardos adicionales en el proceso de cálculo. Esto podría impactar el rendimiento en aplicaciones que requieren alta velocidad de procesamiento. Esto sería una desventaja con respecto a otras implementaciones que utilicen menos compuertas para añadir la nueva funcionalidad de la ALU con su tercer registro.
- **Consumo de Energía Incrementado:** Muy relacionado a lo mencionado anteriormente, al tener más componentes activos (las 9 adicionales para el manejo del tercer circuito, y las 2 que hacen el calculo de la resta y la suma para pasar al CHIP OPZ), es probable que el consumo de energía del circuito sea mayor en comparación con una ALU tradicional y las diferentes implementaciones de la ALU con sus nuevas funcionaldiades. Esto puede ser una desventaja en sistemas donde la eficiencia energética es crítica. 




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
2. **Consumo de Recursos**: A pesar de la reducción en el número de compuertas, la introducción de múltiples ALUs y MUX adicionales puede aumentar el consumo de recursos en términos de área en un CHIP o FPGA.
3. **Posible Latencia**: La segmentación de operaciones a través de múltiples sub-ALUs podría incrementar la latencia en el procesamiento de ciertas operaciones, lo que podría afectar el rendimiento en aplicaciones donde la velocidad es crítica.


## Analisis de Ventajas y Desventajas de Incluir el Tercer Registro
## ¿Qué Implementación es Mejor?


## Conclusión General

- Agregar una nueva funcionalidad a la ALU implica complejidad en el diseño del circuito, una probable latencia y un consumo de energia mayor, por lo tanto, se debe buscar dar implementaciones lo más eficiente posible desde los tres aspectos mencionados anteriormente.

- El "Encapsulamiento de ALUs" es una implemetación modular para manejar operaciones aritméticas y lógicas complejas. La organización en sub-ALUs no solo optimiza el uso de compuertas, sino que también prepara el camino para futuras expansiones y adaptaciones, a pesar de las posibles complicaciones iniciales y el aumento en la latencia.