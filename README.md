# Asesor Virtual SDIH — Bot de Recomendación de Talleres de IA
Sistema de recomendación inteligente que sugiere talleres del catálogo de Salazar Duke Impact Hub (SDIH) según el perfil del usuario. Integra Machine Learning supervisado, IA generativa, y un chatbot conversacional en Telegram.
**Proyecto Final — Bootcamp de Inteligencia Artificial**  
Autora: Jennifer Salazar Duque  
Año: 2026
---
## ¿Qué hace?
El bot vive en Telegram. Cuando un usuario le escribe contándole sobre sí mismo (rol, objetivo, nivel), el sistema:
1. Interpreta el mensaje y extrae perfil + objetivo + nivel + área de interés
2. Predice con ML (Regresión Logística multiclase) la ruta de aprendizaje más adecuada entre 5 rutas posibles
3. Genera con IA una recomendación personalizada de 1-2 talleres específicos
4. Registra demanda: si recomienda un taller marcado como "próximamente", lo guarda en un CSV — esto convierte al bot en un sensor de demanda que le indica a SDIH qué construir primero
---
## Arquitectura
Telegram → python-telegram-bot → asesor_sdih() → [LogReg ML + Qwen2.5-7B IA] → Respuesta
↓
log_demanda_talleres.csv

**Stack:**
- Lenguaje: Python 3.11
- ML: scikit-learn (LogisticRegression multinomial, PCA, LabelEncoder)
- IA Generativa: Qwen2.5-7B-Instruct vía Hugging Face Inference API
- Bot: python-telegram-bot v20+ (asyncio)
- Notebook: Jupyter (Anaconda)
---
## Datos
- Dataset: 500 registros sintéticos de perfiles de usuarios
- Distribuciones: basadas en estadísticas reales del DANE 2024 y MinTIC Colombia
- Variables: perfil, objetivo, nivel, area_interes
- Variable objetivo: ruta recomendada (multiclase, 5 valores)
---
## Cómo correrlo
### Requisitos previos
- Anaconda o Miniconda instalado
- Cuenta gratuita en Hugging Face (https://huggingface.co)
- Bot creado con @BotFather en Telegram
### Instalación
git clone https://github.com/jennifersalazarduke/asesor-sdih-bootcamp.git
cd asesor-sdih-bootcamp
conda create -n sdih_asesor python=3.11 -y
conda activate sdih_asesor
pip install -r requirements.txt
cp .env.example .env

Editá el archivo `.env` y pegá tus tokens reales.
### Ejecución
jupyter notebook

Abrir `Asesor_SDIH_PyMEs.ipynb` y ejecutar las celdas en orden (1 a 16).
La última celda inicia el bot de Telegram en modo polling. Cuando aparezca el mensaje de bot conectado, abrí tu bot en Telegram y mandá /start.
---
## Estructura del proyecto
asesor-sdih-bootcamp/
├── README.md
├── Asesor_SDIH_PyMEs.ipynb
├── dataset_sdih.csv
├── log_demanda_talleres.csv
├── requirements.txt
├── .env.example
├── .gitignore
└── docs/
└── Documentacion_Tecnica.pdf

---
## Seguridad
Los tokens (Hugging Face, Ngrok, Telegram) se gestionan vía variables de entorno en el archivo `.env` local, excluido del repositorio mediante `.gitignore`. Nunca subir el `.env` real al repo.
---
## Limitaciones conocidas
1. **Dataset sintético** — Las predicciones funcionan bien pero deben validarse con datos reales de inscripción.
2. **Cobertura del modelo** — Combinaciones de variables raras (ej. emprendedor + aprender_técnico + principiante, solo ~3 registros en 500) pueden generar predicciones inestables.
3. **Alucinación del LLM** — Mitigada vía prompt engineering estricto y temperatura baja (0.3).
4. **Tono regional** — El LLM tiende a usar expresiones rioplatenses por sesgo en datos de pre-entrenamiento. Se mitigó con restricciones explícitas en el prompt.
---
## Demo
Video de presentación: (pendiente — agregar link de YouTube cuando esté subido)  
Bot en vivo: @SDIH_asesor_bot en Telegram
---
## Sobre SDIH
Salazar Duke Impact Hub es una empresa colombiana de formación en IA para emprendedores, desarrolladores y profesionales en LATAM.  
Sitio: https://salazardukeimpacthubteams.com
---
## Licencia
MIT — Proyecto académico desarrollado como entrega final del Bootcamp de IA.