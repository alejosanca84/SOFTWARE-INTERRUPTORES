# SOFTWARE-INTERRUPTORES
Desarrollo del software SOFTDEGELK-SP3 para el cálculo del desgaste de los contactos en interruptores de GIS ELK-SP3

# Análisis de Degradación de Contactos y Modelado del Desgaste Acumulado en Interruptores de Potencia en GIS

Este repositorio detalla la fundamentación teórica, la formulación matemática y la implementación computacional para la evaluación del desgaste eléctrico y la estimación de la vida útil restante (*Remaining Useful Life - RUL*) de los contactos de arco en interruptores de potencia en subestaciones encapsuladas (GIS), con base en la curva de vida eléctrica proporcionada por el fabricante (p. ej., ABB ELK-SP3).

---

## 1. Introducción y Física del Problema

Los interruptores de alta tensión en instalaciones GIS sufren una erosión estructural, térmica y eléctrica en sus contactos de arco (típicamente fabricados de aleaciones de Cobre-Tungsteno, Cu-W) cada vez que interrumpen o establecen una corriente eléctrica bajo condiciones de carga o de falla.

Cuando el interruptor abre sus contactos bajo carga o cortocircuito:
1. **Formación del Arco Eléctrico:** Al separarse los contactos de arco, la rigidez dieléctrica del medio disminuye y se establece un arco de plasma.
2. **Disipación Térmica y Ablación:** La energía disipada en el arco provoca la fusión, vaporización y erosión por ablación del material conductor de los contactos.
3. **Dependencia Fuertemente No Lineal:** La pérdida de masa ($\Delta m$) por maniobra no es lineal; depende exponencialmente de la magnitud de la corriente interrumpida ($I$).

Los fabricantes proporcionan **Diagramas de Vida Eléctrica** (o *Curvas de Endurancia Eléctrica*) en coordenadas logarítmicas/semilogarítmicas, que relacionan el número máximo admisible de operaciones Cierre-Apertura ($N(I)$) en función de la corriente de interrupción $I$.

---

## 2. Base Teórica de la Erosión de Contactos

### 2.1 Física del Desgaste Eléctrico
La pérdida de masa por maniobra $\Delta m_i$ es proporcional a la energía eléctrica total integrada durante la duración del arco $t_{arc}$:

$$E_{arc} = \int_{0}^{t_{arc}} u_{arc}(t) \cdot i(t) \, dt$$

Dado que la tensión del arco $u_{arc}$ se mantiene relativamente constante durante la fase estable de arco, la pérdida de masa se relaciona directamente con la integral de la corriente $\int |i(t)|^{ lpha} dt$, traduciéndose empíricamente a una ley de potencias en función de la corriente eficaz (RMS) interrumpida $I$:

$$\Delta m \propto I^{ lpha}$$

Donde $ lpha$ (exponente de corriente) varía típicamente entre **1.5 y 2.0**, según la geometría de los contactos, la presión del gas SF6 y el diseño de la boquilla (*nozzle*):
* **$ lpha  pprox 1.0 - 1.5$:** Rango de corriente nominal o de carga (predomina la ablación térmica).
* **$ lpha  pprox 2.0$:** Rango de corrientes de cortocircuito (fuerte erosión por arco y efectos de estrangulamiento magnético o *pinch effect*).

### 2.2 Definición Matemática de la Curva de Vida Eléctrica

La curva del fabricante define la envolvente de seguridad $N_{max}(I)$, modelada a tramos mediante expresiones potenciales:

$$N(I) = C \cdot I^{-m}$$

O en forma por tramos (como se observa en la gráfica del manual de operación):

$$N(I) =  egin{cases} 
N_{méc} & 	ext{para } I \le I_{umbral} \
C_1 \cdot I^{- lpha_1} & 	ext{para } I_{umbral} < I \le I_{media} \
C_2 \cdot I^{- lpha_2} & 	ext{para } I > I_{media}
\end{cases}$$

#### Desglose de Parámetros de la Gráfica Proporcionada:
* **Límite Mecánico Máximo:** $N  pprox 10.000$ operaciones para corrientes bajas ($I \le 5 	ext{ kA}$).
* **Rango de Interrupción Nominal:** Caída suave de $10.000$ maniobras a $5	ext{ kA}$ hasta $1.000$ maniobras a $ pprox 12	ext{ kA}$.
* **Rango de Falla / Cortocircuito:** Decaimiento no lineal rápido hasta $ pprox 10 - 20$ operaciones a la capacidad máxima de ruptura en cortocircuito ($I_{sc} = 63	ext{ kA}$).

---

## 3. Desglose Paso a Paso de la Gráfica de Endurancia

```
Número de operaciones admisibles N(I)
 10000 +----+----------------------------------+
       |   |\                                  |
       |   | \  Zona 1: Límite Mecánico/Carga  |
  1000 +---+--\--------------------------------+
       |       \                               |
       |        \  Zona 2: Sobrecarga / Media  |
   100 +---------\-----------------------------+
       |          \                            |
       |           \  Zona 3: Cortocircuito    |
    10 +------------\--------------------------+
       0     10     20    30    40    50    60  70
                     Corriente de interrupción (kA)
```

1. **Zona 1 ($0 	ext{ kA} - 5 	ext{ kA}$):**
   * Límite constante en $N = 10.000$ operaciones.
   * Causa dominante: Desgaste mecánico del mecanismo de accionamiento (p. ej., HMB8) y erosión térmica menor.
2. **Zona 2 ($5 	ext{ kA} - 25 	ext{ kA}$):**
   * Pendiente moderada ($ lpha  pprox 1.5$).
   * Maniobras de carga nominal, corrientes inductivas/capacitivas o fallas de línea de baja magnitud.
3. **Zona 3 ($25 	ext{ kA} - 63 	ext{ kA}$):**
   * Pendiente pronunciada ($ lpha  pprox 2.0$).
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

## 6. Implementación Computacional en Python

### 6.1 Interpolación Log-Log de la Curva
Para digitalizar la curva del fabricante, se realiza una interpolación lineal en espacio logarítmico sobre los puntos digitalizados de la gráfica $(I_j, N_j)$:

$$\log_{10}(N(I)) = 	ext{Interp}\left(I, I_{tabla}, \log_{10}(N_{tabla})
ight)$$

### 6.2 Código de Ejemplo en Python

```python
import numpy as np

class ModeloDesgasteInterruptor:
    def __init__(self):
        # Puntos digitalizados de la curva ABB ELK-SP3
        self.puntos_corriente = np.array([0.0, 5.0, 10.0, 20.0, 30.0, 40.0, 50.0, 63.0])  # kA
        self.puntos_operaciones = np.array([10000, 10000, 2000, 350, 100, 40, 20, 12])   # Operaciones
        
    def obtener_operaciones_admisibles(self, corriente_kA):
        if corriente_kA <= self.puntos_corriente[0]:
            return self.puntos_operaciones[0]
        
        # Escala logaritmica para el numero de operaciones
        log_ops = np.log10(self.puntos_operaciones)
        log_ops_interp = np.interp(corriente_kA, self.puntos_corriente, log_ops)
        return 10**log_ops_interp

    def calcular_dano_evento(self, corriente_kA):
        n_max = self.obtener_operaciones_admisibles(corriente_kA)
        return 1.0 / n_max

    def calcular_desgaste_acumulado(self, historial_corrientes_kA):
        fraccion_desgaste_total = sum(self.calcular_dano_evento(I) for I in historial_corrientes_kA)
        porcentaje_usado = fraccion_desgaste_total * 100.0
        porcentaje_restante = max(0.0, 100.0 - porcentaje_usado)
        
        return {
            "desgaste_acumulado_porcentaje": porcentaje_usado,
            "vida_util_restante_porcentaje": porcentaje_restante,
            "total_operaciones": len(historial_corrientes_kA)
        }

# Ejemplo de uso:
modelo = ModeloDesgasteInterruptor()

# Historial de eventos simulados de un interruptor GIS:
# 500 maniobras de carga normal a 2 kA, 10 disparos por falla moderada a 25 kA, 2 cortocircuitos severos a 50 kA
historial_disparos = [2.0]*500 + [25.0]*10 + [50.0]*2

estado = modelo.calcular_desgaste_acumulado(historial_disparos)
print(f"Total de Operaciones Registradas: {estado['total_operaciones']}")
print(f"Desgaste Acumulado: {estado['desgaste_acumulado_porcentaje']:.2f}%")
print(f"Vida Útil Restante (RUL): {estado['vida_util_restante_porcentaje']:.2f}%")
```

---

## 7. Integración en Mantenimiento Predictivo y Gestión de Activos

Al integrar este algoritmo en IEDs (Relés de Protección), sistemas SCADA o plataformas de Gemelo Digital (*Digital Twin*), los ingenieros de subestaciones pueden:

1. **Monitoreo del Índice de Salud en Tiempo Real:** Cálculo automático e inmediato de $D_{cum}$ tras cada apertura bajo falla registrada por los transformadores de corriente (TCs).
2. **Estimación Dinámica de Vida Útil Restante (RUL):** Proyección de fechas de intervención basadas en la tasa de falla histórica de la bahía.
3. **Mantenimiento Basado en Condición (CBM):** Transición de mantenimientos con periodicidad fija (p. ej., cada 10 años) a sobredimensionamientos u *overhauls* basados en el estado real del contacto, optimizando OPEX y previniendo fallas catastróficas.

---

## 8. Resumen de Fórmulas Principales

| Parámetro | Expresión Matemática | Descripción |
|---|---|---|
| Energía del Arco | $E_{arc} = \int u_{arc} \cdot i \, dt$ | Causa física principal de la erosión de masa |
| Daño por Operación | $d_i = rac{1}{N(I_i)}$ | Fracción de vida consumida por una maniobra $i$ |
| Daño Acumulado | $D_{cum} = \sum_{i=1}^{k} rac{1}{N(I_i)}$ | Porcentaje total de desgaste de los contactos |
| Vida Restante | $RUL(\%) = (1 - D_{cum}) 	imes 100\%$ | Margen de operabilidad restante de los contactos |
