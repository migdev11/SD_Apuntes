### Clase 8: Introducción al Procesamiento Digital de Señales (PDS)
**Fecha:** Martes, 24 de noviembre de 2026

#### 1. Señales y PDS
* **Analógica:** continua (rampa) -> cualquier valor en cualquier instante.
* **Digital:** discreta (escalera) -> valores fijos en instantes separados. Con $N$ bits hay $2^N$ valores.
* **PDS (DSP):** convertir la señal en números para escalar, filtrar ruido, decidir y mostrar.
* **Cadena:** Sensor (°C -> mV) -> ADC (muestrea y cuantiza) -> Código (filtra y decide) -> Salida.

---

#### 2. Muestreo (Tiempo)
* **Muestra ($x[n]$):** valor en un instante (una foto). **Muestreo:** fotos a intervalos fijos.
* **Periodo ($T_s$) y Frecuencia ($f_s$):** $f_s = \frac{1}{T_s}$ (muestras por segundo en Hz).
* **Teorema de Nyquist:** $f_s > 2 \cdot f_{max}$.
* **Aliasing:** muestrear muy lento engaña (aparece una frecuencia falsa: $f_{falsa} = |f - k \cdot f_s|$). Se evita con filtro RC antes del ADC.
* **Temperatura:** cambia lento ($< 1\text{ Hz}$). Usaremos $f_s = 10\text{ Hz}$ ($T_s = 0.1\text{ s}$) para promediar.

---

#### 3. Cuantización (Amplitud)
* **Cuantización:** redondear cada muestra al escalón más cercano.
* **Error de cuantización:** lo que se pierde al redondear (máx. $\pm \frac{1}{2}$ escalón).
* **LSB (altura de un escalón):** $\text{LSB} = \frac{\text{fondo de escala}}{2^N}$
  * *Arduino UNO (10 bits, 5V):* $1024$ escalones $\rightarrow \approx 4.9\text{ mV}$.
  * *ESP32 (12 bits, 1.1V a 0 dB):* $4096$ escalones $\rightarrow \approx 0.27\text{ mV}$.
* **Resolución vs Exactitud:** resolución es el tamaño del escalón; exactitud es qué tan cerca está del valor real.

---

#### 4. ADC del ESP32
* **ADC1 (GPIO 32 a 39):** 8 canales, **funciona con Wi-Fi** (usaremos GPIO34, solo entrada).
* **ADC2:** no funciona si el Wi-Fi está encendido.
* **Atenuación (el zoom del ADC):** $V_{ref}$ interno es $\approx 1.1\text{ V}$. Se elige la más baja que cubra al sensor:
  * **0 dB:** zoom cerca ($100\text{ a } 950\text{ mV}$), escalones finos. *(Ideal para el LM35)*.
  * **11 dB (por defecto):** zoom lejos ($150\text{ a } 3100\text{ mV}$). *(No sirve para el LM35 en Loja porque bajo $15^\circ\text{C}$ lee mal)*.
* **Decibelios y Ruido:** $dB = 10 \cdot \log_{10}(P / P_0)$ | $\text{SQNR} \approx 6.02 \cdot N + 1.76\text{ dB}$.
  * *Quitar ruido:* condensador de $100\text{ nF}$ ("104") a GND en hardware + promedio en software.
* **Calibración:** usar `analogReadMilliVolts(pin)` (usa calibración de fábrica eFuse) en vez de `analogRead()`.

---

#### 5. Sensor LM35 y Procesamiento
* **LM35:** entrega $10\text{ mV}/^\circ\text{C}$ ($T = \frac{mV}{10}$). Alimentar con **VIN (5V)** (con 3.3V falla), `Vout` a **GPIO34** y `GND` a **GND**.
* **Paso 1 (Escalar):** dividir mV entre `10.0` (`float` para no perder decimales).
* **Paso 2 (Filtrar - Media móvil):** promediar últimas $N$ muestras con búfer circular ($\text{suma} = \text{suma} - \text{vieja} + \text{nueva}$).
  * *Retardo del filtro:* $N \times T_s$ (con $N=10$ y $T_s=0.1\text{ s}$, tarda $1\text{ s}$).
* **Paso 3 (Decidir - Histéresis):** usar dos umbrales para evitar parpadeos por ruido (ej. enciende a $28^\circ\text{C}$, apaga a $26^\circ\text{C}$).
```[cite: 14, 54, 55, 56]