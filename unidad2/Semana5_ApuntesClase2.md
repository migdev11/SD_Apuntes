### Clase 9: Manejo de Pantallas LCD con I2C en ESP32
**Fecha:** Miércoles, 25 de noviembre de 2026

#### 1. ¿Qué es una pantalla LCD?
* **LCD (Liquid Crystal Display):** pantalla alfanumérica (texto y números, no gráficos) de bajo consumo. El cristal líquido bloquea o deja pasar la luz de fondo (retroiluminación LED).
* **Formatos comunes:** `16x2` (32 caracteres) y `20x4` (80 caracteres).
* **Controlador HD44780 (matriz 5×8 píxeles):**
  * **DDRAM:** guarda el texto que se muestra en pantalla (80 bytes).
  * **CGROM:** caracteres de fábrica (ASCII).
  * **CGRAM:** hasta 8 caracteres personalizados (útil para símbolos de grados `°`, flechas o iconos).

---

#### 2. Conexión Paralela vs Módulo I2C (PCF8574)
* **Modo paralelo (4 / 8 bits):** usa de 6 a 10 GPIO del ESP32, requiere potenciómetro externo y cableado complejo.
* **Adaptador I2C (PCF8574):** convierte los 2 hilos I2C en 8 salidas paralelas hacia el LCD.
  * **Dirección I2C:** `0x27` (PCF8574) o `0x3F` (PCF8574A). Se verifica con un escáner I2C.
  * **Trimmer de contraste:** se gira con destornillador hasta ver los caracteres.
  * **Jumper de luz:** conectado = retroiluminación activa.

**Esquema de conexión (4 cables):**
| ESP32 DevKit | Módulo I2C + LCD | Función |
| :--- | :--- | :--- |
| `GND` | `GND` | Tierra común |
| `VIN (5V)` | `VCC` | Alimentación 5 V *(no conectar a 3.3V)* |
| `GPIO 21` | `SDA` | Datos I2C |
| `GPIO 22` | `SCL` | Reloj I2C |

---

#### 3. Pasos y Funciones Principales (`LiquidCrystal_I2C`)
**Pasos para ponerlo en marcha:**
1. Instalar la librería **LiquidCrystal I2C** (Frank de Brabander) en Arduino IDE.
2. Conectar los 4 cables (`GND`, `VIN`, `GPIO 21`, `GPIO 22`).
3. Escanear la dirección I2C (`0x27` o `0x3F`).
4. Subir el código y ajustar el trimmer de contraste.

**Funciones clave:**
* `LiquidCrystal_I2C lcd(0x27, 16, 2);` -> crea el objeto (dirección, columnas, filas).
* `lcd.init();` -> inicializa la pantalla.
* `lcd.backlight();` -> enciende la luz de fondo.
* `lcd.setCursor(col, fila);` -> mueve el cursor (empieza en `0, 0`).
* `lcd.print(valor);` -> escribe texto o números.
* `lcd.clear();` -> borra toda la pantalla *(evitar usarlo dentro del `loop()` para que no parpadee; mejor sobrescribir con espacios `"     "`)*.

---

#### 4. Solución de Problemas Comunes
* ` Se ven cuadros negros: ajustar el trimmer de contraste.

* `Pantalla en blanco: probar dirección 0x3F o correr un escáner I2C.

* `Caracteres raros o nada: revisar cables SDA/SCL, GND común y alimentar con 5 V (VIN).

* `Lecturas que saltan: usar promedio móvil, capacitor de 100 nF y usar ADC1 si hay Wi-Fi activo.
