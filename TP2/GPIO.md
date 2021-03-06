## Manejo de GPIOs para módulo ESP32

Para el proyecto del Banderillero Satelital se utilizará la placa [Wipy 3.0](https://docs.pycom.io/datasheets/development/wipy3/) Esta placa tiene un micro ESP32, y se puede usar MicroPython como lenguage para desarrollar.


Lo primero que hice fue investigar las librerías que podía usar en esa placa con ese lenguaje, y llegué a documentación de MicroPython para la ESP32: [docs.micropython.org](https://docs.micropython.org/en/latest/esp32/quickref.html#pins-and-gpio)

Se explica como configurar pines GPIO como entrada/salida en Python:

```python
from machine import Pin

p0 = Pin(0, Pin.OUT)    # create output pin on GPIO0
p0.on()                 # set pin to "on" (high) level
p0.off()                # set pin to "off" (low) level
p0.value(1)             # set pin to on/high

p2 = Pin(2, Pin.IN)     # create input pin on GPIO2
print(p2.value())       # get value, 0 or 1

p4 = Pin(4, Pin.IN, Pin.PULL_UP) # enable internal pull-up resistor
p5 = Pin(5, Pin.OUT, value=1) # set pin high on creation
```

![GPIO en ESP32](https://i.imgur.com/qw8sex3.png)



 A partir de esa documentación, llegué a la documentación del framework Espressif específico para el ESP32: [docs.espressif.com](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/gpio.html)
 
 ![doc-espressif](https://i.imgur.com/k2JOssT.png)
 
  En la documentación se muestran funciones básicas para menejo de los puertos GPIO, y también indica el link del repositorio de la librería y se llega a:
  
  [driver/include/driver/gpio.h](https://github.com/espressif/esp-idf/blob/7d75213/components/driver/include/driver/gpio.h)
  
  Entrando en el respoitorio entonces, en el directorio: ```esp-idf/components/driver/include/driver/gpio.h```
  
 Se encuentra el detalle de como configurar los puertos, por ejemplo:
 
 ```C
   // Resetear un GPIO
   esp_err_t gpio_reset_pin(gpio_num_t gpio_num);
   // Setear direccion en un GPIO
   esp_err_t gpio_set_direction(gpio_num_t gpio_num, gpio_mode_t mode);
   // Obtener valor de un GPIO
   int gpio_get_level(gpio_num_t gpio_num);
```
 
 
 Investigo más sobre la función para resetear un GPIO y llego al directorio ```esp-idf/components/driver/gpio.c```donde se detalla  ```esp_err_t gpio_reset_pin(gpio_num_t gpio_num)```:

```C
 esp_err_t gpio_reset_pin(gpio_num_t gpio_num)
 {
     assert(gpio_num >= 0 && GPIO_IS_VALID_GPIO(gpio_num));
     gpio_config_t cfg = {
         .pin_bit_mask = BIT64(gpio_num),
         .mode = GPIO_MODE_DISABLE,
         //for powersave reasons, the GPIO should not be floating, select pullup
         .pull_up_en = true,
         .pull_down_en = false,
         .intr_type = GPIO_INTR_DISABLE,
     };
     gpio_config(&cfg);
     return ESP_OK;
 }
```
 
 La variable ```gpio_config_t``` y  ```gpio_num_t```se encuentran definidas en ```esp-idf/components/soc/include/hal/gpio_types.h```
 
 ```C
  typedef struct {
     uint64_t pin_bit_mask;          /*!< GPIO pin: set with bit mask, each bit maps to a GPIO */
     gpio_mode_t mode;               /*!< GPIO mode: set input/output mode                     */
     gpio_pullup_t pull_up_en;       /*!< GPIO pull-up                                         */
     gpio_pulldown_t pull_down_en;   /*!< GPIO pull-down                                       */
     gpio_int_type_t intr_type;      /*!< GPIO interrupt type                                  */
 } gpio_config_t;
```
 
 ```C
 typedef enum {
     GPIO_NUM_NC = -1,    /*!< Use to signal not connected to S/W */
     GPIO_NUM_0 = 0,     /*!< GPIO0, input and output */
     GPIO_NUM_1 = 1,     /*!< GPIO1, input and output */
     GPIO_NUM_2 = 2,     /*!< GPIO2, input and output */
     GPIO_NUM_3 = 3,     /*!< GPIO3, input and output */
     GPIO_NUM_4 = 4,     /*!< GPIO4, input and output */
     GPIO_NUM_5 = 5,     /*!< GPIO5, input and output */
     GPIO_NUM_6 = 6,     /*!< GPIO6, input and output */
     GPIO_NUM_7 = 7,     /*!< GPIO7, input and output */
     GPIO_NUM_8 = 8,     /*!< GPIO8, input and output */
     GPIO_NUM_9 = 9,     /*!< GPIO9, input and output */
     GPIO_NUM_10 = 10,   /*!< GPIO10, input and output */
     GPIO_NUM_11 = 11,   /*!< GPIO11, input and output */
     GPIO_NUM_12 = 12,   /*!< GPIO12, input and output */
     GPIO_NUM_13 = 13,   /*!< GPIO13, input and output */
     GPIO_NUM_14 = 14,   /*!< GPIO14, input and output */
     GPIO_NUM_15 = 15,   /*!< GPIO15, input and output */
     GPIO_NUM_16 = 16,   /*!< GPIO16, input and output */
     GPIO_NUM_17 = 17,   /*!< GPIO17, input and output */
     GPIO_NUM_18 = 18,   /*!< GPIO18, input and output */
     GPIO_NUM_19 = 19,   /*!< GPIO19, input and output */
     GPIO_NUM_20 = 20,   /*!< GPIO20, input and output */
     GPIO_NUM_21 = 21,   /*!< GPIO21, input and output */
 #if CONFIG_IDF_TARGET_ESP32
     GPIO_NUM_22 = 22,   /*!< GPIO22, input and output */
     GPIO_NUM_23 = 23,   /*!< GPIO23, input and output */

     GPIO_NUM_25 = 25,   /*!< GPIO25, input and output */
 #endif
     /* Note: The missing IO is because it is used inside the chip. */
     GPIO_NUM_26 = 26,   /*!< GPIO26, input and output */
     GPIO_NUM_27 = 27,   /*!< GPIO27, input and output */
     GPIO_NUM_28 = 28,   /*!< GPIO28, input and output */
     GPIO_NUM_29 = 29,   /*!< GPIO29, input and output */
     GPIO_NUM_30 = 30,   /*!< GPIO30, input and output */
     GPIO_NUM_31 = 31,   /*!< GPIO31, input and output */
     GPIO_NUM_32 = 32,   /*!< GPIO32, input and output */
     GPIO_NUM_33 = 33,   /*!< GPIO33, input and output */
     GPIO_NUM_34 = 34,   /*!< GPIO34, input mode only(ESP32) / input and output(ESP32-S2) */
     GPIO_NUM_35 = 35,   /*!< GPIO35, input mode only(ESP32) / input and output(ESP32-S2) */
     GPIO_NUM_36 = 36,   /*!< GPIO36, input mode only(ESP32) / input and output(ESP32-S2) */
     GPIO_NUM_37 = 37,   /*!< GPIO37, input mode only(ESP32) / input and output(ESP32-S2) */
     GPIO_NUM_38 = 38,   /*!< GPIO38, input mode only(ESP32) / input and output(ESP32-S2) */
     GPIO_NUM_39 = 39,   /*!< GPIO39, input mode only(ESP32) / input and output(ESP32-S2) */
 #if GPIO_PIN_COUNT > 40
     GPIO_NUM_40 = 40,   /*!< GPIO40, input and output */
     GPIO_NUM_41 = 41,   /*!< GPIO41, input and output */
     GPIO_NUM_42 = 42,   /*!< GPIO42, input and output */
     GPIO_NUM_43 = 43,   /*!< GPIO43, input and output */
     GPIO_NUM_44 = 44,   /*!< GPIO44, input and output */
     GPIO_NUM_45 = 45,   /*!< GPIO45, input and output */
     GPIO_NUM_46 = 46,   /*!< GPIO46, input mode only */
 #endif
     GPIO_NUM_MAX,
 /** @endcond */
 } gpio_num_t;
```

En la estructura donde se define ```gpio_config_t``` se usan otras variables, como por ejemplo: ```gpio_mode_t```, que se encuentra definida en el mismo directorio ```esp-idf/components/soc/include/hal/gpio_types.h```:

```C
 typedef enum {
     GPIO_MODE_DISABLE = GPIO_MODE_DEF_DISABLE,                                                         /*!< GPIO mode : disable input and output             */
     GPIO_MODE_INPUT = GPIO_MODE_DEF_INPUT,                                                             /*!< GPIO mode : input only                           */
     GPIO_MODE_OUTPUT = GPIO_MODE_DEF_OUTPUT,                                                           /*!< GPIO mode : output only mode                     */
     GPIO_MODE_OUTPUT_OD = ((GPIO_MODE_DEF_OUTPUT) | (GPIO_MODE_DEF_OD)),                               /*!< GPIO mode : output only with open-drain mode     */
     GPIO_MODE_INPUT_OUTPUT_OD = ((GPIO_MODE_DEF_INPUT) | (GPIO_MODE_DEF_OUTPUT) | (GPIO_MODE_DEF_OD)), /*!< GPIO mode : output and input with open-drain mode*/
     GPIO_MODE_INPUT_OUTPUT = ((GPIO_MODE_DEF_INPUT) | (GPIO_MODE_DEF_OUTPUT)),                         /*!< GPIO mode : output and input mode                */
 } gpio_mode_t;
```

Y las variables que se utilizan en ```gpio_mode_t``` estan definidas en ```esp-idf/components/soc/soc/esp32/include/soc/gpio_caps.h```:

```C
 // ESP32 has 1 GPIO peripheral
 #define SOC_GPIO_PORT           (1)
 #define GPIO_PIN_COUNT          (40)

 // On ESP32 those PADs which have RTC functions must set pullup/down/capability via RTC register. 
 // On ESP32-S2, Digital IOs have their own registers to control pullup/down/capability, independent with RTC registers.
 #define GPIO_SUPPORTS_RTC_INDEPENDENT (0)
 // Force hold is a new function of ESP32-S2
 #define GPIO_SUPPORTS_FORCE_HOLD      (0)

 #define GPIO_APP_CPU_INTR_ENA      (BIT(0))
 #define GPIO_APP_CPU_NMI_INTR_ENA  (BIT(1))
 #define GPIO_PRO_CPU_INTR_ENA      (BIT(2))
 #define GPIO_PRO_CPU_NMI_INTR_ENA  (BIT(3))
 #define GPIO_SDIO_EXT_INTR_ENA     (BIT(4))

 #define GPIO_MODE_DEF_DISABLE         (0)
 #define GPIO_MODE_DEF_INPUT           (BIT0)
 #define GPIO_MODE_DEF_OUTPUT          (BIT1)
 #define GPIO_MODE_DEF_OD              (BIT2)

 #define GPIO_IS_VALID_GPIO(gpio_num)              ((gpio_num < GPIO_PIN_COUNT && GPIO_PIN_MUX_REG[gpio_num] != 0))                                     /*!< Check whether it is a valid GPIO number */
 #define GPIO_IS_VALID_OUTPUT_GPIO(gpio_num)       ((GPIO_IS_VALID_GPIO(gpio_num)) && (gpio_num < 34))                                                  /*!< Check whether it can be a valid GPIO number of output mode */
 #define GPIO_MASK_CONTAIN_INPUT_GPIO(gpio_mask)   ((gpio_mask & (GPIO_SEL_34 | GPIO_SEL_35 | GPIO_SEL_36 | GPIO_SEL_37 | GPIO_SEL_38 | GPIO_SEL_39)))  /*!< Check whether it contains input io */
```

