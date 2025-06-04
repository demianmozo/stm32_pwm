## Proyecto 2: Manejo de dimmer con PWM

### Introducción

Se programará un regulador de luz (dimmer) con una secuencia de regulación preestablecida, utilizando el kit `STM32F4DISCOVERY` y la placa de expansión `SDBoard`.
Los LED rojo y verde de la columna derecha (columna B) se usarán para simular luces reguladas.
El objetivo es introducir el uso y configuración de señales **PWM** y retardos temporales.

#### Comportamiento esperado:

1. Luces roja y verde al 100% (PWM) por 2 segundos.
2. Apagado completo.
3. Después de 1 segundo:
   - Luz roja al 100%
   - Luz verde apagada
4. Transición cruzada:
   - Rojo baja intensidad
   - Verde aumenta intensidad
5. Luego se invierte el proceso.
6. El ciclo se repite indefinidamente.

------

### Instrucciones

- Configurar los jumpers de la SDBoard.
- Identificar y configurar pines PD13 y PD12 (LEDs).
- Asociar al Timer 4 (canales 1 y 2).
- Realizar diagrama de flujo.
- Escribir algoritmo con funciones comentadas.
- Probar y corregir.

------

### Diagrama de Flujo

![diagrama_flujo](/doc/diagramaflujo.png)

------

### Definiciones en código (en `main.c`)

/* Private define ------------------------------------------------------------*/

```c
#define TENCENDIDO 2000 // Tiempo en ms luces encendidas al inicio
#define TAPAGADO   1000 // Tiempo en ms luces apagadas antes de transición
#define TCAMBIO      20 // Retardo entre cambios de PWM
#define VALCAMBIO   640 // Paso de cambio de ciclo de trabajo (1% de 64000)
```

------

### Funciones principales

#### Función `inicio()`

```c
void inicio(void){
    uint16_t ciclomaximo = TIM4->ARR;
    TIM4->CCR1 = ciclomaximo;
    TIM4->CCR2 = ciclomaximo;
    HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_1);
    HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_2);
    HAL_Delay(TENCENDIDO);
    TIM4->CCR1 = 0;
    TIM4->CCR2 = 0;
    HAL_Delay(TAPAGADO);
    TIM4->CCR2 = ciclomaximo;
}
```

#### Función `transicion1()`: rojo disminuye, verde aumenta

```c
void transicion1(void){
    uint16_t ciclomaximo = TIM4->ARR;
    uint16_t cicloverde = TIM4->CCR1;
    uint16_t ciclorojo = TIM4->CCR2;
    do{
        HAL_Delay(TCAMBIO);
        ciclorojo = (ciclorojo > VALCAMBIO) ? ciclorojo - VALCAMBIO : 0;
        TIM4->CCR2 = ciclorojo;
        cicloverde = (ciclomaximo - cicloverde > VALCAMBIO) ? cicloverde + VALCAMBIO : ciclomaximo;
        TIM4->CCR1 = cicloverde;
    } while(ciclorojo > 0 && cicloverde < ciclomaximo);
}
```

#### Función `transicion2()`: verde disminuye, rojo aumenta

```c
void transicion2(void){
    uint16_t ciclomaximo = TIM4->ARR;
    uint16_t cicloverde = TIM4->CCR1;
    uint16_t ciclorojo = TIM4->CCR2;
    do{
        HAL_Delay(TCAMBIO);
        cicloverde = (cicloverde > VALCAMBIO) ? cicloverde - VALCAMBIO : 0;
        TIM4->CCR1 = cicloverde;
        ciclorojo = (ciclomaximo - ciclorojo > VALCAMBIO) ? ciclorojo + VALCAMBIO : ciclomaximo;
        TIM4->CCR2 = ciclorojo;
    } while(cicloverde > 0 && ciclorojo < ciclomaximo);
}
```

------

### Loop principal

```c
// En USER CODE BEGIN 2
inicio();

// En USER CODE BEGIN 3
while (1) {
    transicion1();
    transicion2();
}
```

------

### Desafío extra

1. Inicia con LED **amarillo** al 100%
2. Aumenta **rojo** a 100%
3. Disminuye **amarillo** a 0%
4. Aumenta **verde** a 100%
5. Disminuye **rojo** a 0%
6. Aumenta **amarillo** a 100%
7. Disminuye **verde** a 0%
8. Repetir indefinidamente pasos 2–7.