# 🤖 Chatbot IA Avanzado para Calificación de Leads (Make.com)

Este proyecto es un sistema automatizado de calificación de leads construido enteramente en la plataforma No-Code **Make.com**. Utiliza un Agente de IA conversacional a través de Telegram para interactuar con nuevos prospectos, realizar una encuesta de calificación, y actualizar el CRM (GoHighLevel) basado en las respuestas.

El sistema está diseñado de forma modular, separando el flujo principal de la conversación de las herramientas específicas que el agente puede invocar.

---

## Diagrama de Arquitectura


* **Consejo:** Te recomiendo crear un diagrama simple en una herramienta como [diagrams.net](https://diagrams.net) o Miro, exportarlo como imagen, subirlo al repositorio y enlazarlo aquí. ¡Marca una gran diferencia!*

---

## 🚀 Flujo de Funcionamiento del Sistema

El proceso completo, desde que un lead es creado hasta que es calificado, sigue los siguientes pasos a través de diferentes escenarios de Make.com:

1.  **Registro del Lead (Entrada):**
    * Un sistema externo (ej. un formulario en GoHighLevel) crea un nuevo contacto.
    * Se dispara un webhook que activa el escenario `01_registrar_lead_webhook.json`.
    * Este escenario guarda la información básica del lead (ID de contacto, email, teléfono) en una base de datos interna de Make.com (Data Store).

2.  **Inicio de Conversación en Telegram:**
    * El usuario inicia una conversación con el bot en Telegram.
    * Esto activa el escenario principal `02_chatbot_main_orchestrator.json`, que actúa como el cerebro del sistema.

3.  **Identificación del Usuario (Tool 1):**
    * El orquestador busca en el Data Store si el `telegramChatId` del usuario ya está registrado.
    * Si el usuario es **desconocido**, el Agente de IA le solicita su correo electrónico.
    * Una vez que el usuario provee el email, el Agente invoca la herramienta `tool_vincular_usuario.json` a través de un webhook. Esta herramienta busca el email en el Data Store y lo asocia con el `telegramChatId`, identificando así al usuario para futuras interacciones.

4.  **Encuesta de Calificación y FAQs:**
    * Una vez identificado, el Agente de IA saluda al lead por su nombre e inicia proactivamente la encuesta de 5 preguntas clave.
    * Durante este proceso, el Agente también está capacitado para responder preguntas frecuentes (FAQs) sobre el producto/servicio.

5.  **Análisis y Calificación de Respuestas (Tool 2):**
    * Cuando el Agente ha recolectado las 5 respuestas de la encuesta, invoca la herramienta `tool_calificar_respuestas.json`.
    * Esta herramienta envía las respuestas a la API de **OpenAI (GPT-4o)** con un prompt estructurado para que las analice y devuelva un objeto JSON con una calificación (`Califica` / `No Califica`), un resumen, y la categorización de cada respuesta.
    * Finalmente, actualiza la ficha del contacto en el CRM **GoHighLevel** con los resultados de la calificación y las etiquetas correspondientes.

6.  **Guardado de Preferencia de Contacto (Tool 3):**
    * Tras la calificación, el Agente pregunta al lead cuál es su método de contacto preferido (WhatsApp, llamada, etc.).
    * Su respuesta activa la herramienta `tool_guardar_preferencia.json`, que guarda esta preferencia en un campo personalizado en la ficha del contacto en GoHighLevel.

7.  **Cierre de Conversación:**
    * El Agente de IA envía un mensaje de despedida, informando al lead que un asesor se pondrá en contacto por el medio que eligió. El ciclo se completa.

---

## 🛠️ Tecnologías y Servicios Utilizados

* **Plataforma de Orquestación:** Make.com
* **Canal de Comunicación:** Telegram
* **Inteligencia Artificial:**
    * **Make.com AI Agent:** Para la gestión conversacional y el uso de herramientas (Tools).
    * **OpenAI API (GPT-4o):** Para el análisis y calificación inteligente de las respuestas de la encuesta.
* **CRM:** GoHighLevel (para la gestión de contactos).
* **Base de Datos:** Make.com Data Store (para la vinculación entre el ID de Telegram y el registro del lead).

---

## 📁 Estructura del Repositorio

Este repositorio contiene los blueprints (en formato JSON) de cada escenario de Make.com que compone el sistema.

* `blueprints/01_registrar_lead_webhook.json`: Escenario de tipo webhook que escucha por nuevos leads y los registra en el Data Store.
* `blueprints/02_chatbot_main_orchestrator.json`: El escenario principal. Se activa con mensajes de Telegram y contiene el Agente de IA que orquesta toda la conversación y decide cuándo llamar a las herramientas.
* `blueprints/tool_vincular_usuario.json`: Herramienta #1. Recibe un email y un `telegramChatId` para vincular a un usuario no reconocido.
* `blueprints/tool_calificar_respuestas.json`: Herramienta #2. Recibe las 5 respuestas de la encuesta, las procesa con OpenAI y actualiza el CRM.
* `blueprints/tool_guardar_preferencia.json`: Herramienta #3. Recibe la preferencia de contacto del usuario y la guarda en el CRM.