# Informe técnico: viabilidad de GasWatch

## Introducción y objetivo técnico

GasWatch propone un sistema urbano de monitoreo preventivo para la detección temprana de posibles fugas de gas natural en redes de distribución. La solución consiste en desplegar una red rotativa de globos cautivos tipo [Skystar](https://www.rtaerostat.com/skystar-100), ubicados en puntos estratégicos de la ciudad, equipados con sensores remotos de metano basados en espectroscopía infrarroja, GPS, sensor meteorológico, módulo de comunicación y procesamiento en borde o centralizado.

El sistema no reemplaza a las cuadrillas técnicas ni a los procedimientos regulatorios de inspección. Su función es actuar como una capa de alerta temprana: detectar anomalías compatibles con metano, estimar la zona probable de origen, cruzar la información con viento y mapas GIS de la red de distribución, y generar alertas priorizadas para inspección en campo. La hipótesis técnica central es que una plataforma aérea persistente, combinada con sensado remoto y modelos de machine learning, puede reducir la incertidumbre espacial de una posible fuga y mejorar la eficiencia operativa de las cuadrillas.

## Fundamento físico: detección remota de metano

La viabilidad científica del sistema se apoya, primero, en una propiedad física bien establecida: el metano posee firmas espectrales detectables en distintas regiones del infrarrojo. Esto permite identificar concentraciones anómalas de CH₄ cuando una pluma altera la radiancia observada por un sensor. En el estudio “Detection of Methane Plumes Using Airborne Midwave Infrared (3–5 µm) Hyperspectral Data”, los autores evaluaron experimentalmente la detección de plumas de metano con datos hiperespectrales aerotransportados en MWIR y LWIR, demostrando que es posible identificar plumas de CH₄ mediante sensores infrarrojos, aunque la sensibilidad depende de la banda espectral, el fondo observado y las condiciones ambientales [[1]](#ref-1).

Este antecedente es directamente relevante para GasWatch porque valida el principio de transformar información espectral en mapas de anomalía o realce de metano. En la solución propuesta, los datos capturados por el sensor del globo se procesan mediante técnicas como matched filter o wide-window matched filter, cuyo objetivo es comparar la señal observada contra la firma espectral esperada del metano. El resultado no es una afirmación absoluta de “hay fuga”, sino una capa probabilística que indica dónde aparece una señal compatible con CH₄ por encima del fondo ambiental esperado.

La selección del tipo de sensor también debe justificarse experimentalmente. El paper “Comparison of Methane Detection Using Shortwave and Longwave Infrared Hyperspectral Sensors Under Varying Environmental Conditions” muestra que la capacidad de detección varía según la región espectral utilizada, la superficie observada, la atmósfera y las condiciones ambientales [[7]](#ref-7). Por eso, GasWatch debe ser presentado como una arquitectura flexible: el sistema puede utilizar espectroscopía infrarroja o láser, pero la banda final del sensor debe validarse en el piloto según distancia, altura, humedad, temperatura, viento y tipo de superficie urbana.

## Viabilidad de plataformas aéreas para inspección de gas

El uso de plataformas aéreas para detectar fugas de gas también cuenta con antecedentes experimentales. En “Detection of Natural Gas Leakages Using a Laser-Based Methane Sensor and UAV”, los autores probaron un UAV equipado con un detector remoto de metano para inspeccionar una red de gas natural. El trabajo demuestra que una plataforma aérea puede transportar sensores de detección de metano y localizar zonas con concentración elevada [[2]](#ref-2). Esta evidencia no implica que GasWatch deba usar drones; al contrario, refuerza la lógica de utilizar una plataforma aérea persistente como un globo cautivo, con mayor permanencia temporal sobre zonas críticas.

También existen antecedentes sobre monitoreo aéreo de infraestructura gasífera. El paper “Monitoring of gas pipelines – a civil UAV application” plantea el uso de vehículos aéreos no tripulados y procesamiento de imágenes para vigilancia de ductos, principalmente frente a daños o interferencias externas [[6]](#ref-6). Aunque ese contexto está más orientado a gasoductos lineales que a redes urbanas densas, sirve para fundamentar que la inspección aérea de infraestructura gasífera es técnicamente viable y que puede complementar patrullajes terrestres.

Por lo tanto, la contribución de GasWatch no es inventar desde cero la detección aérea de gas, sino adaptar una lógica ya explorada en la  plataforma urbana persistente. A diferencia de un dron, el globo cautivo puede permanecer durante períodos más largos sobre una zona crítica, capturando series temporales de datos y permitiendo observar persistencia, repetición o desplazamiento de anomalías compatibles con metano.

## Modelo de machine learning y variables utilizadas

La parte algorítmica de GasWatch se basa en transformar el dato espectroscópico en un mapa de realce de metano y utilizarlo como entrada para un modelo de segmentación semántica. El objetivo del modelo es clasificar cada zona observada como fondo normal y una posible pluma de metano con diferentes score en cada area.

El uso de inteligencia artificial para este tipo de problema se justifica con el paper “Semantic segmentation of methane plumes with hyperspectral machine learning models”, donde la detección de metano se formula como una tarea de segmentación espacial de plumas a partir de datos hiperespectrales [[9]](#ref-9). Este enfoque es consistente con GasWatch porque el sistema no busca únicamente responder si existe o no una fuga, sino localizar espacialmente una anomalía compatible con metano dentro de una escena urbana.

En la solución propuesta, el modelo no depende únicamente de la señal espectral. Para mejorar la confiabilidad operativa, se agregan variables físicas, espaciales y contextuales. Las entradas principales del modelo son: mapa de realce de metano, imagen RGB o infrarroja auxiliar, dirección y velocidad del viento, ubicación geográfica de la observación, distancia a cañerías, válvulas, cámaras y estaciones reguladoras, capas GIS de la red de gas, material, diámetro, presión, antigüedad de cañerías, historial operativo de fugas, reclamos, reparaciones, obras recientes, densidad poblacional y criticidad urbana.

Estas variables permiten que el modelo no interprete una anomalía espectral de manera aislada. Una señal compatible con metano ubicada cerca de una cañería antigua, a favor del viento y en una zona con historial de reclamos tiene mayor valor operativo que una anomalía aislada sin relación espacial con la red. De esta manera, GasWatch convierte la detección remota en una alerta priorizada y georreferenciada.

El resultado del modelo sería una máscara de probabilidad, donde cada píxel o celda indica la probabilidad de pertenecer a una posible pluma de metano. Luego, el sistema agrupa las zonas detectadas, elimina ruido aislado, estima la posible dirección de origen usando viento y cruza la anomalía con la infraestructura gasífera cercana. El producto final no es una decisión automática definitiva, sino un score de riesgo para orientar la inspección de cuadrillas.

## Priorización territorial, GIS y ubicación de globos cautivos

Una fuga de gas no necesariamente se observa justo encima del caño dañado. La pluma puede desplazarse por acción del viento, dispersarse por turbulencia urbana o mezclarse con otras fuentes. 

Hay estudios que muestran que la detectabilidad del metano cambia según condiciones atmosféricas, ambientales, superficie observada y tipo de sensor [[7]](#ref-7). En una ciudad, los falsos positivos pueden provenir de cloacas, vehículos, industrias, superficies reflectivas, humedad, sombras, obras, combustión o fuentes biogénicas. Por eso, la integración de GIS y meteorología local no es un agregado accesorio, sino una condición necesaria para mejorar la precisión operativa.

Además, la ubicación de los globos cautivos puede definirse mediante un criterio formal de cobertura espacial. Para esto se propone adaptar la lógica del Art Gallery Theorem, presentado por Do en “Art Gallery Theorems” [[10]](#ref-10). El problema original estudia cómo ubicar una cantidad mínima de observadores para cubrir visualmente un espacio poligonal. En GasWatch, esta idea puede trasladarse a la ubicación de globos: los “observadores” son los globos cautivos y las zonas a cubrir son los tramos críticos de la red de gas.

La adaptación no busca cubrir toda la ciudad de forma uniforme, sino maximizar la cobertura sobre activos relevantes. Cada tramo de cañería puede recibir un peso según antigüedad, material, presión, historial de fugas, densidad poblacional, cercanía a infraestructura sensible y criticidad operativa. Luego, a partir del GIS urbano, se calculan ubicaciones candidatas para globos cautivos y se seleccionan aquellas que cubren la mayor cantidad de tramos críticos bajo restricciones reales de visibilidad, altura, radio efectivo del sensor, permisos, viento, comunicación y mantenimiento.

De esta manera, la selección de posiciones no queda basada solo en intuición operativa, sino en un problema de cobertura ponderada: ubicar los globos donde mayor valor aporten para monitorear cañerías críticas. El cruce entre GIS, meteorología, criticidad de red y cobertura geométrica permite justificar cuántos globos se necesitan y dónde conviene instalarlos para maximizar la utilidad del sistema.

## Validación experimental propuesta

Una pregunta concreta: si el sistema puede transformar mediciones aéreas de metano en alertas útiles para que una cuadrilla inspeccione mejor y más rápido una zona crítica de la red.

Para eso, primero se debe probar el sensor en condiciones controladas. En esta etapa se mide desde qué distancia detecta metano, cómo cambia la señal con la altura del globo, qué efecto tienen el viento, la humedad, la temperatura y el tipo de superficie, y cuál es el nivel mínimo de concentración que puede identificar de forma confiable.

Luego, se realizan pruebas controladas en campo. La idea es liberar metano o un gas trazador en condiciones conocidas y comparar lo que detecta el sistema aéreo con mediciones terrestres de referencia. Esto permite saber si la pluma aparece en el mapa de realce de metano, si el modelo logra ubicarla correctamente y cuánto se desplaza respecto del punto real de fuga por efecto del viento.

Finalmente, el sistema debe probarse en una zona real de red urbana. En esta etapa, cada alerta generada por GasWatch se envía a verificación de cuadrilla. El resultado de campo se registra como fuga confirmada, falso positivo, fuente no gasífera o evento no concluyente. Esa información sirve tanto para medir la utilidad real del sistema como para mejorar el modelo con nuevos datos etiquetados.

Las métricas principales no deben enfocarse solamente en “acierta o no acierta”, sino en su valor operativo. Debe medirse cuántas fugas detecta, cuántas falsas alarmas genera, cuánto reduce el área de búsqueda, cuánto tarda en generar una alerta, qué tan cerca estima el origen probable y qué porcentaje de alertas termina siendo confirmado por cuadrilla. Si el piloto demuestra que el sistema reduce incertidumbre, prioriza mejor las inspecciones y evita recorridos innecesarios, entonces la solución queda validada como capa de alerta temprana.

## Modelo de negocio y mejora continua del sistema

El modelo de negocio de GasWatch no se basaría en obtener margen principal por la instalación inicial de globos, sensores, comunicaciones y dashboard. Esa etapa funcionaría como implementación técnica de entrada, con margen reducido, para facilitar la adopción por parte de distribuidoras, municipios o entes reguladores.

El valor económico principal estaría en el servicio recurrente de operación, mantenimiento y mejora continua del sistema. GasWatch funciona como un sistema vivo: cada alerta inspeccionada por una cuadrilla genera nueva información etiquetada, ya sea fuga confirmada, falso positivo, fuente no gasífera o evento no concluyente. Esa retroalimentación permite actualizar el modelo, ajustar umbrales, recalibrar sensores y mejorar progresivamente la precisión del sistema.

Este enfoque se vincula con lo planteado en el paper de segmentación de plumas de metano, donde se destaca que los modelos y datasets forman parte de un sistema operativo que puede actualizarse con eventos nuevos y etiquetados [[9]](#ref-9). Por eso, la propuesta comercial se estructura como un servicio: mantenimiento de sensores, calibración, monitoreo de calidad de datos, actualización GIS, soporte del dashboard y reentrenamiento periódico del modelo.

De esta manera, el cliente no paga únicamente por instalar tecnología, sino por sostener una capacidad operativa que mejora con el uso: detectar mejor, reducir falsos positivos, priorizar cuadrillas y construir evidencia histórica sobre zonas críticas de la red.

## Socios institucionales y habilitación regulatoria

Para viabilizar la operación de globos cautivos en entorno urbano, GasWatch requiere apoyo institucional de actores como municipios, distribuidoras de gas y ENARGAS. Estos socios no solo aportarían acceso a información crítica de red, historial de fugas y zonas prioritarias, sino que también facilitarían la gestión regulatoria ante la ANAC.

Dado que la operación de globos cautivos en Argentina requiere autorización bajo la RAAC Parte 32, la estrategia regulatoria se plantea en dos etapas. Primero, realizar un piloto en una ciudad con menor complejidad aérea, como Rosario o Córdoba, para validar la operación técnica y generar evidencia experimental. Segundo, avanzar hacia una habilitación para AMBA mediante un convenio respaldado por una distribuidora o por ENARGAS como actor institucional.

La participación de municipios y ENARGAS permite presentar el sistema no como una instalación privada aislada, sino como infraestructura de utilidad pública orientada a seguridad urbana, reducción de pérdidas de gas y mitigación de emisiones de metano. Esto fortalece la fundamentación técnica ante la autoridad aeronáutica y reduce el riesgo regulatorio del despliegue.

## Conclusión

GasWatch es científicamente defendible si se presenta como un sistema probabilístico de monitoreo preventivo y priorización de inspecciones, no como un reemplazo absoluto de la inspección técnica. Los papers revisados demuestran que existen tecnologías similares o componentes equivalentes: detección infrarroja de plumas de metano [[1]](#ref-1), sensores láser montados en plataformas aéreas [[2]](#ref-2), procesamiento hiperespectral y análisis multitemporal [[4]](#ref-4), monitoreo aéreo de infraestructura gasífera [[6]](#ref-6), análisis de sensibilidad bajo condiciones ambientales variables [[7]](#ref-7), segmentación de plumas de metano mediante modelos de machine learning [[9]](#ref-9) y optimización geométrica de cobertura para ubicar observadores sobre zonas críticas [[10]](#ref-10).

La contribución de GasWatch no está en inventar desde cero la detección remota de
metano, sino en integrar esas capacidades en una arquitectura urbana persistente. Por lo
tanto, la solución es técnicamente viable como capa de alerta temprana, siempre que el
piloto demuestre sensibilidad suficiente, baja tasa de falsos positivos, utilidad real para
priorizar inspecciones en campo y mejora medible en la cobertura de activos críticos.


## Referencias

- <a id="ref-1"></a>[1] [Detection of Methane Plumes Using Airborne Midwave Infrared (3–5 µm) Hyperspectral Data](https://www.mdpi.com/2072-4292/10/8/1237).
- <a id="ref-2"></a>[2] [Detection of Natural Gas Leakages Using a Laser-Based Methane Sensor and UAV](https://www.mdpi.com/2072-4292/13/3/510).
- <a id="ref-4"></a>[4] [A multi-temporal method for detection of underground natural gas leakage using hyperspectral imaging](https://www.sciencedirect.com/science/article/abs/pii/S1750583622000780).
- <a id="ref-6"></a>[6] [Monitoring of gas pipelines – a civil UAV application](https://www.researchgate.net/publication/241543602_Monitoring_of_gas_pipelines_-_A_civil_UAV_application).
- <a id="ref-7"></a>[7] [Comparison of Methane Detection Using Shortwave and Longwave Infrared Hyperspectral Sensors Under Varying Environmental Conditions](https://ieeexplore.ieee.org/abstract/document/10049588).
- <a id="ref-9"></a>[9] [Semantic segmentation of methane plumes with hyperspectral machine learning models](https://arxiv.org/html/2511.07719v2#S6).
- <a id="ref-10"></a>[10] [N. Do, “Art Gallery Theorems,” Gazette of the Australian Mathematical Society, vol. 31, no. 5, pp. 288–294, Nov. 2004](https://www.austms.org.au/wp-content/uploads/Gazette/2004/Nov04/mathellaneous.pdf).
