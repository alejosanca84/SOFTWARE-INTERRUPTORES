# SOFTWARE-INTERRUPTORES
Desarrollo del software SOFTDEGELK-SP3 para el cálculo del desgaste de los contactos de arco en interruptores de GIS ELK-SP3

# Teoria sorbre la degradación de Contactos y Modelado del Desgaste Acumulado en Interruptores de Potencia en GIS

Este repositorio detalla la fundamentación teórica para la evaluación del desgaste eléctrico y la estimación de la vida útil restante  de los contactos de arco en interruptores de potencia en subestaciones encapsuladas (GIS), con base en la curva de vida eléctrica proporcionada por el fabricante ( ABB ELK-SP3).

---

## 1. Introducción y Física del Problema

Los interruptores de alta tensión en instalaciones GIS sufren una erosión estructural, térmica y eléctrica en sus contactos de arco (típicamente fabricados de aleaciones de Cobre-Tungsteno) cada vez que interrumpen una corriente eléctrica bajo condiciones de carga o de falla.

Cuando el interruptor abre sus contactos bajo carga o cortocircuito:
1. **Formación del Arco Eléctrico:** Al separarse los contactos de arco, la rigidez dieléctrica del medio disminuye y se establece un arco de plasma.
2. **Disipación Térmica y Ablación (destrucción del material):** La energía disipada en el arco provoca la fusión, vaporización y erosión por ablación del material conductor de los contactos.
3. **Degradación No Lineal:** La pérdida de masa ($\Delta m$) por maniobra no es lineal; depende exponencialmente de la magnitud de la corriente interrumpida ($I$).

Los fabricantes proporcionan **Diagramas de Vida Eléctrica**  en coordenadas logarítmicas/semilogarítmicas, que relacionan el número máximo admisible de operaciones Cierre-Apertura ($ N(I)$) en función de la corriente de interrupción $I$.

---

## 2. Base Teórica de la Erosión de Contactos

### 2.1 Física del Desgaste Eléctrico
La pérdida de masa por maniobra $\Delta m_i$ es proporcional a la energía eléctrica total integrada durante la duración del arco $t_{arc}$:

$$E_{arc} = \int_{0}^{t_{arc}} u_{arc}(t) \cdot i(t) \, dt$$

Dado que la tensión del arco $u_{arc}$ se mantiene relativamente constante durante la fase estable de arco, la pérdida de masa se relaciona directamente con la integral de la corriente $\int |i(t)|^{alpha} dt$, traduciéndose empíricamente a una ley de potencias en función de la corriente eficaz (RMS) interrumpida $I$:

$$\Delta m \propto I^{alpha}$$

Donde $ \alpha$ (exponente de corriente) varía típicamente entre **1.5 y 2.0**, según la geometría de los contactos, la presión del gas SF6 y el diseño de la boquilla (*nozzle*):
* **$alpha  pprox 1.0 - 1.5$:** Rango de corriente nominal o de carga (predomina la ablación térmica).
* **$alpha  pprox 2.0$:** Rango de corrientes de cortocircuito (fuerte erosión por arco y efectos de estrangulamiento magnético o *pinch effect*).

### 2.2 Definición Matemática de la Curva de Vida Eléctrica

La curva del fabricante define la envolvente de seguridad $N_{max}(I)$, modelada a tramos mediante expresiones potenciales:

$$N(I) = C \cdot I^{-m}$$

O en forma por tramos (como se observa en la gráfica del manual de operación):

!<img width="699" height="815" alt="image" src="https://github.com/user-attachments/assets/9ef116cd-178a-4015-b9aa-88225fb39127" />

#### Desglose de Parámetros de la Gráfica Proporcionada:
* **Límite Mecánico Máximo:** $N  pprox 10.000$ operaciones para corrientes bajas ($I \le 5 	ext{ kA}$).
* **Rango de Interrupción Nominal:** Caída suave de $10.000$ maniobras a $5	ext{ kA}$ hasta $1.000$ maniobras a $ pprox 12	ext{kA}$.
* **Rango de Falla / Cortocircuito:** Decaimiento no lineal rápido hasta $ pprox 10 - 20$ operaciones a la capacidad máxima de ruptura en cortocircuito ($I_{sc} = 63	ext{ kA}$).

---

## 3. Desglose Paso a Paso de la Gráfica de Endurancia

1. **Zona 1 ($0 	ext{kA} - 5 	ext{kA}$):**
   * Límite constante en $N = 10.000$ operaciones.
   * Causa dominante: Desgaste mecánico del mecanismo de accionamiento (ejemplo el HMB8) y erosión térmica menor.
2. **Zona 2 ($5 	ext{kA} - 25 	ext{kA}$):**
   * Pendiente moderada ($alpha  pprox 1.5$).
   * Maniobras de carga nominal, corrientes inductivas/capacitivas o fallas de línea de baja magnitud.
3. **Zona 3 ($25 	ext{kA} - 63 	ext{kA}$):**
   * Pendiente pronunciada ($alpha  pprox 2.0$).
   * Cortocircuitos severos de alta energía que producen una pérdida significativa de material conductor en cada disparo.

---

## 4. Modelo Computacional de Desgaste Acumulativo (Adaptación de la Regla de Miner)

En la operación real del sistema eléctrico, un interruptor experimenta corrientes variables a lo largo de su vida útil (maniobras de carga, energización de transformadores, disparos por protecciones y cortocircuitos severos).

Para calcular la degradación acumulada, se aplica la **Hipótesis del Daño Acumulado de Miner** (utilizada habitualmente en fatiga de materiales y cálculo de vida eléctrica):

### 4.1 Daño Relativo por Operación
Para una maniobra individual $i$ a una corriente interrumpida $I_i$, la fracción de vida útil consumida $d_i$ es:

$$d_i = rac{1}{N(I_i)}$$

Donde $N(I_i)$ es el número máximo de operaciones permitidas a la corriente $I_i$, obtenido mediante interpolación sobre la curva del fabricante.

### 4.2 Desgaste Acumulado Total ($D_{cum}$)
Tras $k$ operaciones de interrupción, el porcentaje total de desgaste acumulado $D_{cum}$ se define como:

$$D_{cum}(\%) = \left( \sum_{i=1}^{k} rac{1}{N(I_i)} 
ight) 	imes 100\%$$

* **Criterio de Mantenimiento / *Overhaul*:** Cuando $D_{cum} \ge 100\%$, la reserva de material erosionable de los contactos de arco se ha agotado, requiriendo la inspección o sustitución de la unidad de extinción (*arcing unit*).

---

## 5. Método Alternativo de Corriente Acumulada Ponderada ($\sum I^x$)

Un método estándar alternativo (según IEC 62271-100 / IEEE C37.09) expresa la degradación en términos de corriente acumulada ponderada por un exponente $x$:

$$I_{acum} = \sum_{i=1}^{k} I_i^x$$

Donde $x$ se deriva de la pendiente de la curva $N(I)$ en escala doblemente logarítmica:

$$x = -rac{\log(N_2) - \log(N_1)}{\log(I_2) - \log(I_1)}$$

---


## 6. Integración en Mantenimiento Predictivo y Gestión de Activos

Al integrar este algoritmo en IEDs (Relés de Protección), sistemas SCADA o plataformas de Gemelo Digital (*Digital Twin*), los ingenieros de subestaciones pueden:

1. **Monitoreo del Índice de Salud en Tiempo Real:** Cálculo automático e inmediato de $D_{cum}$ tras cada apertura bajo falla registrada por los transformadores de corriente (TCs).
2. **Estimación Dinámica de Vida Útil Restante (RUL):** Proyección de fechas de intervención basadas en la tasa de falla histórica de la bahía.
3. **Mantenimiento Basado en Condición (CBM):** Transición de mantenimientos con periodicidad fija (p. ej., cada 10 años) a sobredimensionamientos u *overhauls* basados en el estado real del contacto, optimizando OPEX y previniendo fallas catastróficas.

---


