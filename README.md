<div align="center">

# 🎯 Monitor Inteligente de Precios y Competencia
### El motor analítico de un SaaS de Dynamic Pricing para sellers de e-commerce

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-Interactive%20Viz-3F4F75?logo=plotly&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-ML-F7931E?logo=scikitlearn&logoColor=white)
![Best Buy API](https://img.shields.io/badge/API-Best%20Buy-FFE000)
![Status](https://img.shields.io/badge/Status-Portfolio%20Project-success)

</div>

---

## 📖 La historia

Son las 11 de la noche y Martín, dueño de una tienda de electrónica online, recién está revisando sus ventas del día. Algo no cierra: vendió la mitad de lo habitual. Entra a buscar el producto de su competidor más cercano y ve que bajó el precio un 15% esa misma mañana — una liquidación para sacarse stock viejo de encima. Martín no se enteró hasta 14 horas después. Para cuando reaccionó, ya había perdido un día entero de ventas.

Dos semanas más tarde, ese mismo competidor se quedó completamente sin stock por varios días. Hubo una ventana enorme para capturar esa demanda subiendo apenas el precio propio sin perder competitividad. Martín tampoco se enteró a tiempo.

**Este es el problema que viven, todos los días, miles de vendedores medianos en e-commerce.** Las herramientas de *dynamic pricing* que resuelven esto existen — pero están pensadas (y tarifadas) para grandes retailers, no para el vendedor que factura unos pocos millones al año.

Este proyecto es la prueba de concepto del motor analítico que resolvería el problema de Martín: un sistema que vigila a la competencia 24/7, detecta movimientos de precio agresivos y quiebres de stock en tiempo real, entiende qué tan sensible es la demanda al precio, y le devuelve una sola cosa simple — **el precio óptimo a cobrar hoy**.

---

## ⚠️ Sobre la fuente de datos (y por qué cambió a mitad de camino)

La idea original era consumir la API pública de MercadoLibre, pensando en el vendedor argentino. Al ponerla a prueba, la API devolvió `403 forbidden` en cada consulta. La razón no es un bug del notebook: **desde abril de 2025, MercadoLibre cerró el acceso público a su endpoint de búsqueda** — cualquier consulta sin un token OAuth de un usuario autorizado queda bloqueada. Es un cambio de política documentado por la propia comunidad de desarrolladores, no algo específico de este proyecto.

Para que el notebook funcione con **datos reales y verificables hoy mismo**, se migró a la **[Best Buy Products API](https://developer.bestbuy.com/)**: pública, gratuita, con key instantánea (sin OAuth, sin aprobación de partner, sin esperar días), y con precio/stock/descuento en tiempo real sobre más de un millón de productos.

Esto implicó un cambio de enfoque, no del problema de negocio: en vez de "vendedores compitiendo por el mismo ítem dentro de un marketplace", el caso de estudio pasa a **"marcas competidoras dentro de una misma categoría de producto"** (Sony vs. JBL vs. Bose vs. Apple en auriculares Bluetooth). Es el mismo desafío de Dynamic Pricing — y de hecho es exactamente cómo operan los productos reales de *MAP monitoring* (cumplimiento de precio mínimo publicado) que usan las marcas para vigilar a sus retailers, y cómo cualquier retailer vigila a la competencia dentro de su categoría.

> 💡 La arquitectura del análisis (detección de quiebres de stock, bajadas agresivas de precio, elasticidad, clustering, motor de precio óptimo) es **agnóstica a la fuente de datos**. El día que tengas credenciales OAuth de MercadoLibre, Amazon o cualquier otro marketplace, el mismo pipeline se conecta ahí sin cambios estructurales — solo cambia la función de recolección de datos.

---

## 🧩 El problema, en números

| Síntoma | Costo para el vendedor |
|---|---|
| No detecta bajadas de precio de la competencia | Pierde ventas por horas o días sin saberlo |
| No detecta quiebres de stock de la competencia | Deja pasar ventanas de demanda capturable |
| Fija un precio "a ojo" y no lo vuelve a tocar | Se queda fuera de mercado o vende con márgenes innecesariamente bajos |
| Las herramientas de pricing dinámico son corporativas y caras | Solo las grandes cadenas pueden automatizar esto |

---

## 🛠️ Qué hace el notebook

El notebook (`monitor_precios_competencia.ipynb`, pensado para correr en Google Colab) construye, paso a paso, el motor analítico completo:

1. **Recolección de datos en tiempo real** desde la Best Buy Products API (con fallback automático a un dataset sintético equivalente si no se configura una API key o la API no responde).
2. **Simulación de historial de 14 días** de precio y stock por competidor — necesaria porque la API solo entrega el snapshot actual, no historial; cuando hay datos reales, se usan como ancla (marca y precio reales) y solo se simula la evolución temporal.
3. **Detección de quiebres de stock** y **bajadas agresivas de precio** (>8% en un día), con generación automática de alertas.
4. **Estimación de elasticidad precio-demanda** mediante regresión log-log, para saber si conviene competir por precio o por diferenciación.
5. **Segmentación de competidores con K-Means** en arquetipos: *Premium/Diferenciado*, *Guerra de Precios* y *Estable/Nicho*.
6. **Motor de Precio Óptimo**: combina costo propio, margen mínimo deseado, precio mínimo de la competencia activa y elasticidad estimada para sugerir un precio único y justificado.
7. **Sistema de alertas** simulando integración real con webhook de Telegram/Discord.
8. **Dashboard interactivo** en Plotly que consolida todo en un panel único — la vista que tendría un cliente real del producto.

---

## 📊 Hallazgos del análisis (sobre el caso de estudio)

- La categoría analizada (auriculares Bluetooth) muestra una **elasticidad precio-demanda fuertemente negativa**: es un mercado donde el precio es un driver fuerte de la decisión de compra.
- Se identificaron **arquetipos de competidor claramente diferenciados**: marcas premium que sostienen precio sin perder stock, marcas en plena guerra de precios, y marcas estables de nicho con rotación más errática.
- Las bajadas agresivas de precio detectadas se concentraron en marcas puntuales — no es ruido generalizado del mercado, es una **estrategia identificable y monitoreable**.
- El motor de precio óptimo, aplicado al caso, recomienda un precio que respeta el margen mínimo del vendedor mientras se mantiene competitivo frente al mínimo de mercado activo (es decir, con stock disponible).

---

## 💡 3 recomendaciones estratégicas para el negocio

> Estas son las conclusiones de negocio que se desprenden directamente del análisis — el tipo de insight que un producto de Dynamic Pricing debería entregarle a su cliente, no solo el dato crudo.

**1. No compitas a ciegas contra el "guerrero de precios".**
El análisis de clusters muestra que siempre hay un competidor dispuesto a sostener el precio más bajo del mercado, a costa de su propio margen. Perseguirlo en una guerra de precios es una carrera que un vendedor mediano no puede ganar. La estrategia ganadora es **monitorear su comportamiento, no igualarlo**: dejarlo capturar al cliente 100% sensible al precio, y posicionarse contra los segmentos *Estable* y *Premium*, donde el envío, los tiempos de entrega y la reputación del vendedor pesan más que los últimos pesos de diferencia.

**2. Los quiebres de stock de la competencia son la oportunidad más barata que existe — y la más desaprovechada.**
Cuando un competidor se queda sin stock, su demanda no se esfuma: migra. Detectarlo en horas (no en días) y reaccionar con un ajuste de precio temporal o una mayor inversión en visibilidad de esa publicación es, literalmente, capturar ventas sin pelear por precio. Esto debería ser una alerta automática, no algo que el vendedor descubre por casualidad navegando publicaciones de la competencia.

**3. El "precio correcto" no es un número fijo, es una decisión que cambia con el contexto competitivo.**
La elasticidad estimada en este análisis indica que la demanda es sensible al precio — pero ese coeficiente puede (y va a) cambiar según el producto, la temporada y quién esté liquidando stock esa semana. Un vendedor que revisa precios "una vez al mes" está, en la práctica, operando con información vieja la mayoría del tiempo. La automatización no es un lujo: es lo que separa a quien reacciona en horas de quien reacciona en días.

---

## 💼 El modelo de negocio detrás del análisis

Este notebook es la **prueba de concepto del motor analítico** de un producto SaaS de *Dynamic Pricing* dirigido a sellers medianos de e-commerce que hoy no pueden pagar herramientas de pricing dinámico corporativas.

- **Propuesta de valor:** automatizar lo que hoy el vendedor hace manualmente (o no hace): vigilar competencia, detectar oportunidades y sugerir precio óptimo.
- **Monetización:** suscripción mensual por catálogo de productos monitoreados (modelo SaaS clásico, con planes por volumen de SKUs).
- **Lo que falta para producción:**
  - Reemplazar la simulación de historial por persistencia real (un job programado guardando cada consulta a la API en una base de datos).
  - Conectar la fuente de datos al marketplace específico del cliente (MercadoLibre, Amazon, Best Buy, o el catálogo propio del retailer) según credenciales que el cliente provea.
  - Reemplazar la alerta simulada por integración real con bot de Telegram/Discord o notificaciones por email.
  - Un frontend simple (o directamente el bot de Telegram como interfaz) para que el vendedor reciba las alertas y el precio sugerido sin tener que abrir un notebook.

---

## 🗂️ Estructura del repositorio

```
motor-inteligente-de-precios/
├── monitor_precios_competencia.ipynb   # Notebook principal (Google Colab)
├── requirements.txt                     # Dependencias
├── data/
│   └── historico_precios_competencia.csv  # Dataset generado por el notebook
└── README.md
```

## 🚀 Cómo correrlo

1. Abrí `monitor_precios_competencia.ipynb` en [Google Colab](https://colab.research.google.com/).
2. (Opcional, para datos 100% reales) Entrá a [developer.bestbuy.com](https://developer.bestbuy.com/), generá tu API key gratis e instantánea, y pegala en la variable `BESTBUY_API_KEY` de la sección 1.
3. Ejecutá las celdas en orden (`Entorno > Ejecutar todas`).
4. Si no configuraste una key (o la API no responde), el notebook cae automáticamente al dataset sintético equivalente — el análisis funciona igual en ambos casos.
5. (Opcional) Para probar el envío real de alertas, crear un bot de Telegram y pasar `bot_token` y `chat_id` a la función `enviar_alerta_telegram()`.

```bash
pip install -r requirements.txt
```

---

## 🔭 Próximos pasos

- Conectar el job de recolección a un scheduler real (Airflow / cron) que corra cada N minutos y persista en una base de datos en vez de simular el historial.
- Sumar más categorías de producto para validar si la elasticidad y los arquetipos de competidores se mantienen entre nichos.
- Evaluar la integración con MercadoLibre vía OAuth2 (registrando una app y autorizándola una vez) para llevar el caso de estudio de vuelta al mercado argentino con datos propios del vendedor.
- Construir un MVP del frontend (o bot de Telegram) para validar el producto con 3-5 vendedores reales antes de escalar.

---

<div align="center">

**Proyecto de portfolio — Data Analyst**
Conectemos en [https://www.linkedin.com/in/davidardizzonedev/](#) · Código en [https://github.com/ardizzone88/motor-inteligente-de-precios](#)

</div>

