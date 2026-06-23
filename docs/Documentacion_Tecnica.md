# Documentación Técnica y Manual del Usuario

**Asesor Virtual SDIH — Bot de Recomendación de Talleres de IA**

---

## 1. Información general del sistema

| Campo | Valor |
|---|---|
| **Nombre del modelo** | Asesor Virtual SDIH |
| **Tipo de modelo** | Clasificación multiclase supervisada + IA generativa conversacional |
| **Autora** | Jennifer Salazar Duque |
| **Fecha** | 22 de junio de 2026 |
| **Versión** | v1.0 |
| **Dataset utilizado** | 500 perfiles sintéticos con distribuciones reales DANE 2025 + MinTIC Colombia |
| **Variable objetivo** | `ruta_recomendada` (multiclase: dev, emprendedor, profesional, bienestar, estrategica) |
| **URL del repositorio** | https://github.com/SalazarDukeImpactHub/asesor-sdih-bootcamp |
| **Demo en vivo** | [@SDIH_asesor_bot](https://t.me/SDIH_asesor_bot) en Telegram |

---

## 2. Resumen ejecutivo

El **Asesor Virtual SDIH** es un chatbot inteligente que recomienda talleres del catálogo de Salazar Duke Impact Hub a usuarios según su perfil, objetivo y nivel de experiencia. El sistema combina **Machine Learning supervisado** (Regresión Logística multiclase) para clasificar al usuario en una de 5 rutas de aprendizaje, con un **modelo de IA generativa** (Qwen2.5-7B-Instruct) que produce respuestas naturales y personalizadas. El bot vive en Telegram y atiende automáticamente 24/7, sirviendo como puerta de entrada al catálogo de 22 talleres.

Además de recomendar, el sistema actúa como **sensor de demanda**: cuando recomienda un taller marcado como "próximamente" (aún no construido), registra la solicitud en un CSV. Esto le permite a SDIH **priorizar la construcción de nuevos talleres basándose en demanda real**, no en intuición.

**A quién impacta:** emprendedores, desarrolladores, profesionales y personas en procesos personales en LATAM que buscan formación estructurada en IA.

**Qué problema resuelve:** reduce la fricción de descubrimiento del catálogo (22 talleres dispersos), automatiza la asesoría inicial, y convierte cada consulta en un dato accionable para la evolución del producto.

---

## 3. Problema y contexto

### 3.1 Problema identificado

Salazar Duke Impact Hub (SDIH) ofrece un catálogo en expansión de talleres de IA dirigidos a 5 audiencias distintas: desarrolladores, emprendedores, profesionales, personas en procesos personales y líderes organizacionales. Actualmente, **9 talleres están disponibles y 13 están planificados** para construcción en los próximos 12 meses.

Esta diversidad genera tres problemas concretos:

1. **Fricción de descubrimiento**: un usuario que llega al sitio web tiene que leer descripciones largas de 22 talleres para identificar cuál le sirve, lo cual genera abandono.
2. **Falta de asesoría personalizada escalable**: SDIH no puede atender manualmente a cada potencial cliente con una recomendación curada.
3. **Decisiones de roadmap sin datos**: ¿cuál de los 13 talleres "próximamente" debería construirse primero? Hoy esa decisión se toma por intuición, no por demanda medida.

### 3.2 Justificación del uso de IA

El problema requiere combinar dos capacidades que solo la IA resuelve eficientemente:

- **Clasificación multivariable no lineal**: el "match" entre un usuario y una ruta de aprendizaje depende de la interacción de 4 variables (perfil, objetivo, nivel, área de interés). Un sistema basado en reglas se vuelve inmanejable rápidamente. **Machine Learning** aprende estas interacciones desde datos.
- **Conversación en lenguaje natural**: los usuarios no responden a formularios — quieren describir su situación libremente. Solo un **modelo generativo (LLM)** puede interpretar respuestas abiertas y generar recomendaciones personalizadas que se sientan humanas.

---

## 4. Arquitectura del modelo

### Flujo del sistema

```
👤 Usuario
    │ escribe en Telegram
    ▼
💬 Telegram Bot API
    │
    ▼
🐍 python-telegram-bot (polling async)
    │
    ▼
🧠 Función asesor_sdih(pregunta)
    │
    ├──► interpretar_input()  →  extrae: perfil, objetivo, nivel, area
    │
    ├──► predecir_ruta()      →  Regresión Logística → 1 de 5 rutas
    │
    ├──► seleccionar_talleres() →  filtra catálogo por ruta + nivel
    │
    ├──► registrar_demanda()    →  log CSV si taller "próximamente"
    │
    └──► chatbot()             →  Qwen2.5-7B IA → respuesta natural
            │
            ▼
        Respuesta al usuario en Telegram
```

### Componentes

1. **Capa de presentación**: Telegram (mobile-first, ubicuo)
2. **Capa de procesamiento**: Python en notebook Jupyter (Anaconda)
3. **Capa de ML**: scikit-learn (entrenamiento local, modelo en memoria)
4. **Capa de IA generativa**: Hugging Face Inference API (cloud, sin descarga local)
5. **Persistencia**: CSV files (dataset + log de demanda)

---

## 5. Datos y variables

### Fuente del dataset

**Dataset sintético** generado programáticamente con `random.choices()` usando distribuciones de peso basadas en estadísticas reales del **DANE 2025** (Departamento Administrativo Nacional de Estadística de Colombia) y del **MinTIC** (Medición de la Transformación Digital de las MiPymes).

Esta decisión es deliberada: para un MVP académico, generar datos sintéticos basados en distribuciones reales es **más válido que usar datos reales mal limpiados o no representativos**. Se documenta como roadmap para v2 el reemplazo por datos orgánicos de inscripciones reales.

### Número de registros

**500 filas** divididas 80/20 (400 entrenamiento, 100 prueba) con stratificación por variable objetivo.

### Variables utilizadas

| Variable | Tipo | Valores posibles |
|---|---|---|
| `perfil` | Categórica | dev · emprendedor · profesional · persona_proceso · lider_organizacional |
| `objetivo` | Categórica | aprender_tecnico · lanzar_proyecto · productividad · bienestar · gobernanza_ia |
| `nivel` | Categórica | principiante · intermedio · avanzado |
| `area_interes` | Categórica | ia_general · ia_negocio · salud_mental · blockchain · compliance · datos |

### Variable objetivo

`ruta_recomendada` — **multiclase** con 5 valores: `dev`, `emprendedor`, `profesional`, `bienestar`, `estrategica`.

### Variables descartadas (y por qué)

- **Edad/Género**: no aporta al match con la ruta de aprendizaje y agregaría sesgo ético innecesario.
- **Ubicación geográfica**: SDIH atiende toda LATAM, no se justifica filtrar por país.
- **Ingresos**: invade privacidad y los talleres tienen precios accesibles.

### Tipo de datos

100% **categóricos**, codificados con `LabelEncoder` de scikit-learn para entrada al modelo.

---

## 6. Modelo de IA utilizado

### Capa de Machine Learning

- **Algoritmo**: Logistic Regression (regresión logística multinomial)
- **Librería**: scikit-learn 1.5+
- **Hiperparámetros**:
  - `max_iter=1000`
  - `solver='lbfgs'`
  - Detección automática de multiclase (scikit-learn 1.5+ deprecó el parámetro explícito `multi_class`)
- **Tipo de aprendizaje**: Supervisado (clasificación)

### Capa de IA Generativa

- **Modelo**: `Qwen/Qwen2.5-7B-Instruct` (Alibaba, 2024)
- **Acceso**: vía Hugging Face Inference API (cloud)
- **Librería**: `huggingface_hub.InferenceClient`
- **Método**: `chat_completion()` con formato de mensajes role/content
- **Hiperparámetros del prompt**:
  - `max_tokens=250`
  - `temperature=0.3` (baja, para reducir creatividad/alucinación)
- **Tipo de aprendizaje**: Pre-entrenado (zero-shot con prompt engineering)

### Decisión arquitectónica: HF API en vez de modelo local

Se evaluó cargar Phi-3-mini localmente (descarga de 7.6 GB + 8-12 GB RAM en inferencia). Se descartó por:
- Tamaño de descarga incompatible con setup académico
- Lentitud en CPU (30-90s por respuesta) inviable para demo

Se eligió HF Inference API: latencia ~3 seg, sin descarga, free tier suficiente para MVP.

---

## 7. Entrenamiento y evaluación

### División de datos

- **80% entrenamiento** (400 registros)
- **20% prueba** (100 registros)
- **Stratificación** activada (`stratify=y`) para asegurar distribución equilibrada de clases en train/test

### Métricas obtenidas

| Métrica | Valor |
|---|---|
| **Accuracy global** | ~100% en test |
| **Precision (macro avg)** | 1.00 |
| **Recall (macro avg)** | 1.00 |
| **F1-score (macro avg)** | 1.00 |
| **Support (test set)** | 100 registros |

### Interpretación de resultados

El modelo alcanza **accuracy cercana al 100%** porque la función `asignar_ruta()` que generó el dataset es **determinística** (dadas las variables de entrada, la ruta de salida es predecible). El modelo aprende esa regla con muy pocos ejemplos.

**Esto NO es overfitting** — es un modelo correctamente ajustado a una regla determinística. La limitación real aparece en producción cuando los inputs son inputs naturales del usuario (no del dataset sintético), tema documentado en Sección 12.

---

## 8. Interpretabilidad

### Método utilizado

**Coeficientes nativos de Logistic Regression** + **visualización con PCA**.

A diferencia de modelos black-box (redes neuronales, random forests), la Regresión Logística expone directamente cuánto pesa cada variable para predecir cada clase. Esto se accede vía `modelo.coef_`.

### Variables más importantes

El análisis de coeficientes muestra que las dos variables con más peso para predecir la ruta son:

1. **`perfil`** — empuja fuertemente hacia rutas específicas (dev → ruta DEV, lider_organizacional → ruta ESTRATÉGICA)
2. **`objetivo`** — modula la ruta dentro del perfil (objetivo "gobernanza_ia" → ESTRATÉGICA aunque perfil sea profesional)

Las variables `nivel` y `area_interes` tienen menor peso individual pero influyen en la selección final del taller dentro de la ruta.

### Explicación del modelo

Cada ruta se identifica con combinaciones específicas de las variables. Por ejemplo:
- Si `perfil = dev` → siempre ruta DEV (regla más fuerte del modelo)
- Si `objetivo = gobernanza_ia` o `perfil = lider_organizacional` → ruta ESTRATÉGICA
- Si `perfil = persona_proceso` o `objetivo = bienestar` → ruta BIENESTAR

El modelo es **transparente y auditable**: cualquier predicción puede explicarse mostrando los coeficientes activados.

---

## 9. Uso del modelo

Flujo desde la perspectiva del usuario final:

1. **El usuario abre Telegram** en su celular o navegador
2. **Busca el bot**: `@SDIH_asesor_bot`
3. **Envía `/start`**: el bot saluda y le pide que se presente
4. **Responde en lenguaje natural**: ej. _"Soy emprendedora colombiana, quiero lanzar mi negocio usando IA, soy principiante"_
5. **El sistema procesa internamente** (3-5 segundos):
   - Interpreta el mensaje
   - Predice la ruta de aprendizaje
   - Selecciona 1-2 talleres específicos
   - Genera la respuesta en lenguaje natural
6. **El bot responde** con la recomendación personalizada, indicando qué talleres están disponibles ahora y cuáles están por construirse

---

## 10. Implementación técnica

| Capa | Tecnología | Detalle |
|---|---|---|
| **Backend** | Python 3.11 en Jupyter | Notebook ejecutándose en Anaconda env `sdih_asesor` |
| **Frontend** | Telegram nativo | Sin UI propia — usa la app de Telegram |
| **Modelo ML** | scikit-learn (en memoria) | Entrenado en cada arranque desde `dataset_sdih.csv` |
| **Modelo IA** | Qwen2.5-7B (cloud) | Hugging Face Inference API |
| **Bot Framework** | python-telegram-bot v20+ | Polling async con `nest_asyncio` para Jupyter |
| **Persistencia** | CSV files | `dataset_sdih.csv` + `log_demanda_talleres.csv` |
| **Despliegue** | Notebook + HF cloud | No requiere servidor propio |
| **Gestión de secretos** | python-dotenv | Variables de entorno en `.env` (gitignored) |

### Arquitectura inicial explorada (descartada)

Durante el desarrollo se exploraron dos arquitecturas alternativas que se descartaron por las siguientes razones:

1. **Flask + Ngrok + Botpress**: arquitectura clásica donde Telegram → Botpress (UI visual) → Flask API. Se descartó porque el sandbox de Botpress Cloud Free **bloquea peticiones a dominios `*.ngrok-free.dev`**, lo cual impedía la conexión. Esta limitación NO es resoluble desde el lado del cliente.

2. **Phi-3-mini local**: descargar el modelo de Microsoft a la PC. Se descartó por tamaño (7.6 GB) y latencia en CPU (30-90s por respuesta), inviable para demo en vivo.

La arquitectura final (Telegram ↔ python-telegram-bot ↔ HF API) es **más simple, más profesional y más confiable** que las alternativas exploradas.

---

## 11. Resultados y valor

### Resultados técnicos

- ✅ Pipeline ML completo: dataset → preprocesamiento → entrenamiento → evaluación → predicción
- ✅ Visualización PCA generada (clusters de usuarios por ruta)
- ✅ Interpretabilidad nativa documentada
- ✅ Bot funcionando 24/7 en Telegram con respuesta natural
- ✅ Sensor de demanda registrando solicitudes en `log_demanda_talleres.csv`

### Valor de negocio para SDIH

1. **Reduce fricción de descubrimiento**: usuario consulta y recibe recomendación específica en lugar de tener que leer 22 descripciones.
2. **Asesoría 24/7**: atiende sin intervención humana cuando una persona no podría.
3. **Lead-magnet funcional**: cada conversación es un potencial cliente identificado.
4. **Sensor de demanda accionable**: a los 3 meses de uso real, el CSV `log_demanda_talleres.csv` indica qué taller "próximamente" tiene más solicitudes → SDIH sabe cuál construir primero, basándose en **datos, no intuición**.
5. **MVP integrable**: el código está listo para conectarse a la plataforma productiva [SDIH Talleres v1](https://talleres.salazardukeimpacthubteam.com/) en una fase futura.

---

## 12. Limitaciones

Limitaciones identificadas durante el desarrollo y pruebas reales:

### 12.1 Dataset sintético

El dataset son 500 registros generados artificialmente. Aunque las distribuciones siguen estadísticas reales (DANE 2025 + MinTIC), no reemplazan datos orgánicos. **Mitigación v2**: reemplazar dataset sintético por respuestas reales del formulario de inscripción a los talleres SDIH.

### 12.2 Cobertura del modelo en combinaciones raras

Durante las pruebas reales se identificó que combinaciones de variables poco representadas en el dataset (ej: `emprendedor` + `aprender_tecnico` + `principiante` + `ia_general`, con apenas ~3 registros de 500) pueden generar predicciones inestables. El modelo tiende a "caer" en clases más frecuentes para estas combinaciones marginales.

**Mitigación**: implementar sampling estratificado por combinaciones poco representadas en futuros datasets, o aumentar el dataset usando técnicas como SMOTE para clases minoritarias.

### 12.3 Alucinación del LLM

Durante el desarrollo se detectó que Qwen2.5-7B, en su configuración inicial (temperatura 0.7), **inventaba nombres de talleres** que sonaban plausibles pero no existen en el catálogo (ej: "Introducción a Máquinas Encontradas", "Taller de Transformación de Datos con Python").

**Mitigaciones aplicadas**:
- Prompt engineering estricto con reglas absolutas ("PROHIBIDO inventar nombres")
- Nombres de talleres entre comillas para indicar entidades fijas
- Temperatura reducida a 0.3 (menos creatividad, más fidelidad al contexto)
- Limpieza posterior en Python (eliminación de Markdown si el modelo igual lo incluye)

### 12.4 Sesgo regional en el LLM

Qwen2.5 fue pre-entrenado mayormente con datos en español europeo y rioplatense. Sin restricciones, tiende a usar "che", "vos", "mate" en sus respuestas, lo cual NO es natural para usuarios colombianos.

**Mitigación**: lista negra explícita en el prompt + instrucción de usar variantes colombianas ("tú", "tienes", "puedes").

### 12.5 URL de Ngrok temporal (versión Flask explorada)

En la arquitectura inicial con Flask + Ngrok, las URLs gratuitas de Ngrok cambian cada vez que se reinicia el servicio. No afecta la versión final del proyecto (que no usa Flask), pero queda documentado como limitación de esa arquitectura.

---

## 13. Ética y seguridad

### Privacidad de datos del usuario

- **No se almacenan** datos personales identificables (nombres, emails, teléfonos).
- El bot solo procesa los mensajes en tiempo real para generar la respuesta, sin persistir conversaciones.
- El único log que se mantiene es `log_demanda_talleres.csv`, que registra **qué taller** se recomendó como "próximamente" y **cuándo** — sin atribución al usuario individual.

### Gestión de credenciales

- Los tokens de Hugging Face, Telegram y Ngrok se gestionan vía variables de entorno cargadas desde un archivo `.env` local.
- El `.env` está excluido del repositorio mediante `.gitignore`.
- Se incluye un `.env.example` como template público.
- Esta es una **práctica estándar de la industria** ("principle of least privilege") para prevenir filtración accidental de credenciales en commits públicos.

### Sesgo en el dataset

El dataset sintético se generó con distribuciones de peso que reflejan la realidad demográfica de empresas colombianas (DANE 2025). Sin embargo, está sesgado hacia microempresas (92% del peso), lo cual es representativo pero **podría limitar la efectividad del modelo para grandes empresas**. Documentado como roadmap.

### Transparencia algorítmica

El modelo de Regresión Logística usado es **interpretable de forma nativa** (coeficientes lineales accesibles vía `.coef_`). Cualquier predicción puede auditarse y explicarse al usuario si lo solicita. Esto cumple con principios de **IA explicable (XAI)** y se alinea con la propuesta regulatoria de IA en Colombia.

### Limitación responsable del LLM

Se reconoce explícitamente la limitación de los LLMs de alucinar contenido plausible pero incorrecto. Se aplicaron múltiples capas de mitigación (prompt engineering + temperatura baja + validación posterior) en lugar de ocultar el problema.

---

## 14. Manual de uso

### Para el usuario final (vía Telegram)

**Paso 1 — Abrir Telegram**
Desde el celular: abrir la app. Desde el computador: ir a [web.telegram.org](https://web.telegram.org).

**Paso 2 — Buscar el bot**
En la barra de búsqueda escribir: `@SDIH_asesor_bot` y seleccionarlo de los resultados (tendrá un check de verificado si está configurado).

**Paso 3 — Iniciar conversación**
Hacer click en el botón **Iniciar** o **Start** (o escribir `/start`).

**Paso 4 — Recibir bienvenida**
El bot saludará y pedirá que el usuario se presente: rol, objetivo, nivel.

**Paso 5 — Responder en lenguaje natural**
Escribir libremente. Ejemplos válidos:
- _"Soy emprendedora colombiana y quiero lanzar mi negocio usando IA, principiante"_
- _"Soy desarrollador con 8 años de experiencia, quiero aplicar IA a sistemas en producción"_
- _"Trabajo en una organización mediana y necesito implementar IA con compliance"_

**Paso 6 — Esperar la respuesta (3-7 segundos)**
El bot mostrará "Procesando tu consulta..." mientras el sistema:
- Interpreta el mensaje
- Predice la ruta de aprendizaje
- Selecciona talleres
- Genera la respuesta natural

**Paso 7 — Interpretar la recomendación**
El bot responderá con 1-2 talleres específicos, indicando cuáles están **disponibles** (puede inscribirse ya) y cuáles están **próximamente** (se le ofrecerá avisar cuando se abran).

### Para desarrolladores que clonan el repo

Ver sección **"Cómo correrlo"** del [README principal](../README.md).

---

## 15. Checklist de entrega

### Documentos

- [x] Documento de Documentación Técnica y Manual del Usuario diligenciado

### Repositorio

- [x] **Enlace del repositorio (OBLIGATORIO):** https://github.com/SalazarDukeImpactHub/asesor-sdih-bootcamp
- [ ] Docente agregado como colaborador (pendiente — solicitar username de GitHub al docente)

### Video de explicación

- [ ] Video de explicación del modelo (OBLIGATORIO, 2-15 minutos)
- [ ] Enlace al video (YouTube/Drive) incluido en este documento

El video debe cubrir:

1. **Problema y objetivo** — qué problema resuelve y por qué es relevante
2. **Datos** — fuente del dataset, variables principales
3. **Modelo** — algoritmo usado, por qué se eligió, cómo funciona a alto nivel
4. **Demostración** — mostrar el bot en vivo respondiendo en Telegram, mostrar el notebook con las métricas y el PCA
5. **Conclusiones** — qué aprendí, limitaciones honestas, posibles mejoras

---

## Anexo: Notas de la estudiante

Este proyecto se desarrolló como **adaptación del taller original de "Agente Fitness con IA"** del bootcamp, redirigido a un caso de uso real de mi startup, **Salazar Duke Impact Hub (SDIH)**. La adaptación se hizo manteniendo el 100% del stack técnico requerido (Jupyter, scikit-learn, IA generativa de Hugging Face, integración con Telegram) y agregando valor pedagógico extra:

- **De clasificación binaria a multiclase** (5 rutas en vez de sí/no)
- **Distribuciones del dataset basadas en DANE 2025** (no random puro)
- **Sensor de demanda** integrado (registra talleres próximamente solicitados)
- **Pivot arquitectónico documentado** (de Botpress a integración directa)
- **Limitación real detectada en producción** (combinaciones raras) usada como caso de estudio honesto

El resultado es un MVP funcional que será integrado a mi plataforma productiva [talleres.salazardukeimpacthubteam.com](https://talleres.salazardukeimpacthubteam.com/).
