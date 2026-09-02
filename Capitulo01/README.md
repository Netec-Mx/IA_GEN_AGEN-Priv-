# 1. Práctica 1. Prepárese para un proyecto de desarrollo de IA

# Metadatos


| Campo          | Valor      |
| -------------- | ---------- |
| Duración       | 30 minutos |
| Complejidad    | Media      |
| Nivel de Bloom | Aplicar    |


## Descripción general

En esta práctica utilizará el [portal de Microsoft Foundry](https://ai.azure.com) para crear un proyecto de desarrollo de IA, listo para construir una solución generativa. Creará el recurso y el proyecto en Azure, desplegará el modelo `gpt-5.2` desde el catálogo, lo probará en el playground con instrucciones de sistema y localizará la clave, el endpoint del proyecto y el endpoint de Azure OpenAI que usarán las aplicaciones cliente.

El resultado será un entorno de Foundry operativo y conectado desde Visual Studio Code mediante la extensión Foundry Toolkit. Así podrá administrar el proyecto, revisar el despliegue del modelo y repetir pruebas en el playground local, sin abandonar el entorno de desarrollo. Anote la región seleccionada: la necesitará en prácticas posteriores.

> **Nota:** Algunas de las tecnologías de este ejercicio están en versión preliminar o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.



## Objetivos de aprendizaje

Al finalizar la práctica, podrá:

- [ ] Crear un proyecto en Microsoft Foundry con su recurso, grupo de recursos y región recomendada.
- [ ] Desplegar el modelo `gpt-5.2` desde el catálogo y revisar su model card para evaluar capacidades y limitaciones.
- [ ] Probar el modelo en el playground del portal con instrucciones de sistema y una consulta de desarrollo de IA.
- [ ] Identificar la clave, el endpoint del proyecto y el endpoint de Azure OpenAI, y distinguir el nivel de recurso del nivel de proyecto.
- [ ] Instalar la extensión Foundry Toolkit en Visual Studio Code, conectar el proyecto y repetir una prueba en el playground local.



## Prerrequisitos



### Conocimientos requeridos

- Antes de comenzar este ejercicio, asegúrese de tener:
  - Una [suscripción activa a Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account)
  - [Visual Studio Code](https://code.visualstudio.com/) instalado
  - [Python versión **3.13.xx](https://www.python.org/downloads/release/python-31312/)** instalado*
  - [Git](https://git-scm.com/install/) instalado y configurado
  - [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado



### Accesos requeridos

Debe disponer de:

- Credenciales de Azure para iniciar sesión en el [portal de Microsoft Foundry](https://ai.azure.com).
- Permisos para crear o seleccionar un grupo de recursos, un recurso de Foundry y un proyecto.
- Permisos para desplegar un modelo desde el catálogo de Foundry.
- Acceso de red a Azure, al portal de Foundry y al marketplace de extensiones de Visual Studio Code.
- Permisos para instalar la extensión Foundry Toolkit y conectar Visual Studio Code a su suscripción de Azure.

> **Importante:** No comparta ni versiona la clave del proyecto. En la mayoría de los escenarios de producción debe preferir autenticación con Microsoft Entra ID, basada en identidades de usuario y de aplicación.



## Entorno del laboratorio



### Recursos de hardware orientativos


| Recurso                | Mínimo            | Recomendado                     |
| ---------------------- | ----------------- | ------------------------------- |
| Procesador             | 2 núcleos         | 4 núcleos o superior            |
| Memoria RAM            | 8 GB              | 16 GB                           |
| Espacio libre en disco | 5 GB              | 10 GB SSD                       |
| Conectividad           | Acceso a Internet | Conexión estable de banda ancha |
| Pantalla               | 1280 × 720        | 1920 × 1080 o superior          |


> El cómputo del modelo se ejecuta en Azure. El equipo local solo necesita capacidad para el navegador, Visual Studio Code, Azure CLI y la extensión Foundry Toolkit.



### Software utilizado


| Componente                  | Versión de referencia                                |
| --------------------------- | ---------------------------------------------------- |
| Sistema operativo           | Windows 10/11, macOS o Linux                         |
| Navegador web               | Microsoft Edge o Google Chrome (más reciente)        |
| Visual Studio Code          | Más reciente                                         |
| Python                      | 3.13.12                                              |
| Git                         | 2.43.0 o superior                                    |
| Azure CLI                   | Más reciente                                         |
| Extensión Foundry Toolkit   | 1.6.9 o superior                                     |
| Portal de Microsoft Foundry | [https://ai.azure.com](https://ai.azure.com)         |
| Portal de Azure             | [https://portal.azure.com](https://portal.azure.com) |
| Modelo desplegado           | `gpt-5.2`                                            |


> Python 3.14 ya está disponible, pero algunas dependencias aún no están compiladas para esa versión. El laboratorio se ha validado con Python 3.13.12.

---

## Procedimiento paso a paso



### Paso 1. Crear un proyecto de Microsoft Foundry

Microsoft Foundry usa proyectos para organizar modelos, recursos, datos y otros activos empleados en el desarrollo de una solución de IA.

**Objetivo:** Crear un proyecto de Microsoft Foundry listo para construir una solución de IA.

**Instrucciones:**

1. En un explorador web, abra el [portal de Microsoft Foundry](https://ai.azure.com) en `https://ai.azure.com` para empezar a construir. Inicie sesión con sus credenciales de Azure. Cierre los paneles de sugerencias o de inicio rápido que se abran la primera vez que inicie sesión.
2. Si aún no está habilitada, en la barra de herramientas de la parte superior de la página, active la opción **New Foundry**. A continuación, cree un proyecto nuevo con un nombre único; expanda el área **Advanced options** para especificar la siguiente configuración:
  - **Foundry resource**: *Use el nombre predeterminado del recurso (normalmente {project_name}-resource)*
  - **Subscription**: *Su suscripción de Azure*
  - **Resource group**: *Cree o seleccione un grupo de recursos*
  - **Region**: Seleccione cualquiera de las regiones **recomendadas de AI Foundry** en [esta lista](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
    > **Sugerencia:** Anote la región que seleccionó. La necesitará más adelante.
3. Seleccione **Create**. Espere a que se cree el proyecto.
  Cuando esté listo, se abrirá la página de inicio del proyecto.

**Salida esperada:**

La página de inicio del proyecto de Foundry se abre en el portal y muestra el proyecto recién creado.

**Verificación:**

Confirme que ve la página de inicio del proyecto y que anotó el nombre del proyecto, el grupo de recursos y la región.

---



### Paso 2. Desplegar y probar un modelo

En el núcleo de cualquier proyecto de IA generativa hay, como mínimo, un modelo de IA generativa.

**Objetivo:** Explorar el catálogo de modelos, desplegar `gpt-5.2` y probarlo en el playground.

**Instrucciones:**

1. Ahora está listo para explorar modelos. En la página **Discover**, seleccione la pestaña **Models** para ver el catálogo de modelos de Microsoft Foundry.
2. Busque el modelo `gpt-5.2` y, a continuación, selecciónelo en los resultados de búsqueda para ver su model card.
  Las model cards proporcionan información sobre los modelos para ayudarle a comprender sus capacidades y limitaciones, y a determinar si son adecuados para sus requisitos.
3. Seleccione **Deploy** con la configuración predeterminada para crear un despliegue del modelo.
  Los despliegues de modelos le permiten trabajar con un modelo en su proyecto.
   Cuando el modelo se haya desplegado, el playground del modelo se abrirá automáticamente para que pueda probarlo.
4. En el cuadro **Instructions**, escriba las siguientes instrucciones:
  ```text
   You are an AI assistant that can provide information and advice about AI software development.
  ```
5. En la ventana de chat, escriba una consulta como `Describe three key considerations for working with Large Language Models for AI application development.` y revise la respuesta.
  El modelo debería proporcionar algunas consideraciones clave para reflexionar.

**Salida esperada:**

El playground muestra una respuesta del modelo desplegado `gpt-5.2` con consideraciones para el desarrollo de aplicaciones con modelos de lenguaje grandes.

**Verificación:**

Confirme que el modelo está desplegado en el proyecto y que el playground devolvió una respuesta coherente con las instrucciones de sistema.

---



### Paso 3. Ver el recurso de Azure de Foundry y los endpoints del proyecto

**Objetivo:** Distinguir el nivel de recurso del nivel de proyecto y anotar la clave y los endpoints que usarán las aplicaciones cliente.

**Instrucciones:**

1. En el portal de Foundry, en la barra de menú superior, seleccione **Manage**.
  El centro de administración es el lugar donde puede ver y administrar sus proyectos y sus recursos primarios.
  - El nivel de *recurso* se refiere al recurso **Foundry** que se creó en Azure para admitir su proyecto. Este recurso incluye conexiones a Foundry Services y a modelos, y proporciona un lugar central para administrar el acceso de los usuarios a los proyectos de desarrollo de IA.
  - El nivel de *proyecto* se refiere a su proyecto individual, donde puede agregar y administrar recursos específicos del proyecto. Un recurso puede admitir varios proyectos (el primero que se crea es el proyecto *predeterminado* del recurso).
2. Seleccione el vínculo al **Parent resource** asociado al proyecto.
  Deben mostrarse los detalles de configuración del recurso.
   Observe que el recurso de Foundry tiene un *endpoint*, a través del cual las aplicaciones cliente pueden acceder a la funcionalidad de nivel de recurso (como Foundry Tools, que se comparten entre todos los proyectos del recurso).
3. En la barra de menú superior, seleccione **Home** para volver a la página de inicio del proyecto.
4. Anote la clave, el endpoint del proyecto y el endpoint de Azure OpenAI.
  Esta información se usa para conectar las aplicaciones cliente con los recursos de nivel de proyecto.
  - La *clave* se usa para la autenticación basada en clave hacia modelos y herramientas (aunque en la mayoría de los escenarios de producción debe considerar autenticación con Microsoft Entra ID, basada en identidades autenticadas de usuario y de aplicación).
  - El *endpoint del proyecto* se usa para acceder a modelos proporcionados directamente en Foundry (incluidos los modelos de OpenAI) mediante la API **Responses** de OpenAI, y para acceder a APIs específicas de Foundry (como el servicio Foundry Agent).
  - El *endpoint de OpenAI* se usa para acceder a modelos mediante las APIs de OpenAI, incluidas la API **Chat Completions** y la API **Responses**.

**Salida esperada:**

Dispone de la clave, el endpoint del proyecto y el endpoint de Azure OpenAI, y puede distinguir el endpoint del recurso primario de los endpoints de nivel de proyecto.

**Verificación:**

Confirme que anotó los tres valores y que comprende la diferencia entre el nivel de recurso y el nivel de proyecto. No comparta ni versiona la clave.

---



### Paso 4. Instalar la extensión Foundry Toolkit para Visual Studio Code

Como desarrollador, es posible que pase algún tiempo trabajando en el portal de Foundry; pero también es probable que pase mucho tiempo en Visual Studio Code. La extensión Foundry Toolkit ofrece una forma conveniente de trabajar con los recursos del proyecto de Foundry sin salir del entorno de desarrollo.

**Objetivo:** Conectar Visual Studio Code al proyecto de Foundry y probar el modelo en el playground local.

**Instrucciones:**

1. Inicie Visual Studio Code.
2. En la barra de navegación de la izquierda, abra la página **Extensions**.
3. Busque en el marketplace de extensiones `Foundry Toolkit` e instale la extensión **Foundry Toolkit for VS Code**.
  La extensión puede tardar un minuto o más en instalarse.
4. Después de instalar la extensión, seleccione la página **Foundry Toolkit** en la barra de navegación izquierda y espere a que se cargue.
5. En el panel Foundry Toolkit, expanda **Microsoft Foundry Resources** y establezca el proyecto predeterminado: conéctese a Azure (inicie sesión con sus credenciales) y seleccione el proyecto de Foundry que creó anteriormente.
6. Después de establecer el proyecto predeterminado, expanda el proyecto, expanda **Models** y seleccione el modelo **gpt-5.2** que desplegó anteriormente.
  Aquí puede ver los detalles del despliegue del modelo.
7. En el panel Foundry Toolkit, en la sección **Developer Tools**, expanda **Build** y seleccione **Model playground**. A continuación, seleccione el modelo **gpt-5.2** (si no está ya seleccionado).
  Se abrirá en Visual Studio Code un playground interactivo en el que puede probar el modelo.

**Salida esperada:**

Visual Studio Code muestra el proyecto de Foundry como proyecto predeterminado, el despliegue de `gpt-5.2` y un playground interactivo del modelo.

**Verificación:**

Confirme que puede ver los detalles del modelo `gpt-5.2` y que el playground de Visual Studio Code está listo para enviar una consulta.

---



## Resumen

En este ejercicio ha creado un recurso de Microsoft Foundry y lo ha explorado en el portal de Foundry. También ha explorado la extensión Foundry Toolkit en Visual Studio Code, que ofrece a los desarrolladores una forma conveniente de trabajar con proyectos de Foundry y sus activos.

### Recursos opcionales

- [Portal de Microsoft Foundry](https://ai.azure.com)
- [Disponibilidad regional de la API Responses](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
- [Información general de Microsoft Foundry Toolkit para Visual Studio Code](https://learn.microsoft.com/azure/foundry/how-to/develop/get-started-projects-vs-code)
- [Foundry Toolkit for VS Code en Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-windows-ai-studio.windows-ai-studio)
- [Guía de selección de modelos en Azure AI Foundry](https://learn.microsoft.com/es-es/azure/ai-foundry/how-to/model-catalog-overview)

---

# 2. Práctica 2. Explore y Compare Modelos

# Metadatos


| Campo          | Valor      |
| -------------- | ---------- |
| Duración       | 45 minutos |
| Complejidad    | Media      |
| Nivel de Bloom | Analizar   |


## Descripción general

El catálogo de modelos de Microsoft Foundry es el repositorio central donde puede explorar y usar una variedad de modelos para crear su escenario de IA generativa. En esta práctica explorará el model catalog, comparará modelos mediante benchmarks, los probará en el model playground y ejecutará una evaluation con un dataset sintético.

El resultado será una comparación documentada entre `gpt-5.2` y `gpt-5-mini`, más una evaluation llamada `travel-assistant-eval` que le permitirá revisar métricas, fallos y sugerencias de mejora.

> **Nota:** Algunas de las tecnologías de este ejercicio están en versión preliminar o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Objetivos de aprendizaje

Al finalizar la práctica, podrá:

- [ ] Explorar el model catalog de Microsoft Foundry y revisar la model card, **Details** y **Benchmarks** de `gpt-5.2`.
- [ ] Comparar modelos en el **Model leaderboard** y en el **Trade-off chart** usando las métricas **Benchmark Cost**, **Throughput** y **Safety**.
- [ ] Desplegar `gpt-5.2` y `gpt-5-mini` y anotar el deployment name de cada uno.
- [ ] Comparar ambos deployments lado a lado en el model playground con un prompt de razonamiento.
- [ ] Ejecutar una evaluation de `gpt-5.2` con **Synthetic generation** y revisar los resultados en **Analyze results**.

## Prerrequisitos

### Conocimientos requeridos

- Antes de comenzar este ejercicio, asegúrese de tener:
  - Una [suscripción de Azure](https://azure.microsoft.com/free/) con permisos para crear recursos de IA.
  - Haber completado la Práctica 1 o disponer de un proyecto de Microsoft Foundry operativo.

### Accesos requeridos

Debe disponer de:

- Credenciales de Azure para iniciar sesión en el [portal de Microsoft Foundry](https://ai.azure.com).
- Permisos para crear o reutilizar un **Foundry resource**, un **Resource group** y un proyecto.
- Permisos para desplegar `gpt-5.2` y `gpt-5-mini` desde el model catalog.
- Permisos para crear una evaluation y generar un dataset con **Synthetic generation**.

> **Importante:** No comparta ni versiona la clave del proyecto. En la mayoría de los escenarios de producción debe preferir autenticación con Microsoft Entra ID.

## Entorno del laboratorio

### Recursos de hardware orientativos


| Recurso                | Mínimo            | Recomendado                     |
| ---------------------- | ----------------- | ------------------------------- |
| Procesador             | 2 núcleos         | 4 núcleos o superior            |
| Memoria RAM            | 8 GB              | 16 GB                           |
| Espacio libre en disco | 5 GB              | 10 GB SSD                       |
| Conectividad           | Acceso a Internet | Conexión estable de banda ancha |
| Pantalla               | 1280 × 720        | 1920 × 1080 o superior          |


> El cómputo de los modelos y de la evaluation se ejecuta en Azure. El equipo local solo necesita un navegador para el portal de Foundry.

### Software utilizado


| Componente                  | Versión de referencia                                |
| --------------------------- | ---------------------------------------------------- |
| Sistema operativo           | Windows 10/11, macOS o Linux                         |
| Navegador web               | Microsoft Edge o Google Chrome (más reciente)        |
| Portal de Microsoft Foundry | [https://ai.azure.com](https://ai.azure.com)         |
| Portal de Azure             | [https://portal.azure.com](https://portal.azure.com) |
| Modelos desplegados         | `gpt-5.2`, `gpt-5-mini`                              |
| Evaluation                  | `travel-assistant-eval`                              |


---

## Procedimiento paso a paso

### Paso 1. Crear un proyecto de Microsoft Foundry

Microsoft Foundry usa proyectos para organizar modelos, recursos, datos y otros activos empleados en el desarrollo de una solución de IA.

**Objetivo:** Crear o reutilizar un proyecto de Microsoft Foundry listo para explorar y comparar modelos.

**Instrucciones:**

1. En un explorador web, abra el [portal de Microsoft Foundry](https://ai.azure.com) en `https://ai.azure.com` para empezar a construir. Inicie sesión con sus credenciales de Azure. Cierre los paneles de sugerencias o de inicio rápido que se abran la primera vez que inicie sesión.
2. Si aún no está habilitada, en la barra de herramientas de la parte superior de la página, active la opción **New Foundry**. A continuación, si se le solicita, cree un proyecto nuevo con un nombre único; expanda el área **Advanced options** para especificar la siguiente configuración:
   - **Foundry resource**: *Use el nombre predeterminado del recurso (normalmente {project_name}-resource)*
   - **Subscription**: *Su suscripción de Azure*
   - **Resource group**: *Cree o seleccione un grupo de recursos*
   - **Region**: Seleccione cualquiera de las regiones **recomendadas de AI Foundry** en [esta lista](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
3. Espere a que se cree el proyecto. Después, vea su página de inicio.

**Salida esperada:**

La página de inicio del proyecto de Foundry está visible en el portal.

**Verificación:**

Confirme que ve la home page del proyecto. Si reutiliza el proyecto de la Práctica 1, confirme que sigue siendo el proyecto activo.

---

### Paso 2. Explorar modelos en el catalog

Microsoft Foundry Models proporciona un catálogo de modelos que puede usar en su proyecto. Puede recorrer el catalog y comparar modelos para encontrar el adecuado para sus necesidades.

**Objetivo:** Revisar la model card de `gpt-5.2`, incluida la información de **Details** y **Benchmarks**.

**Instrucciones:**

1. Ahora está listo para explorar modelos. En la página **Discover**, seleccione la pestaña **Models** para ver el model catalog de Microsoft Foundry.

   El model catalog lista todos los modelos disponibles en Foundry. Algunos se proporcionan directamente desde Azure (y se facturan a través de su suscripción de Azure), mientras que otros los proporcionan partners y la comunidad.

   Observe que puede buscar y filtrar el catalog, según nombres de modelo, capabilities y otros factores.
2. Busque `gpt-5.2`. Después, en los resultados de búsqueda, seleccione el modelo **gpt-5.2** para ver su *model card*. Las model cards proporcionan información sobre los modelos para ayudarle a determinar si son adecuados para sus necesidades.
3. Lea la descripción y revise la demás información disponible en la página **Details**.
4. Vea la página **Benchmarks** del modelo gpt-5.2 para observar cómo se compara en algunos performance benchmarks estándar con otros modelos usados en escenarios similares.
5. Use la flecha atrás (**←**) junto al título de la página **gpt-5.2** para volver al model catalog.

**Salida esperada:**

Ha revisado la model card de `gpt-5.2`, incluidas las páginas **Details** y **Benchmarks**, y ha regresado al model catalog.

**Verificación:**

Confirme que puede describir, a partir de la model card, al menos una capability, una limitación y un resultado de **Benchmarks** de `gpt-5.2`.

---

### Paso 3. Comparar modelos usando el model leaderboard

Ahora usará el model leaderboard y las funciones de comparación lado a lado para comparar modelos de forma visual.

**Objetivo:** Comparar `gpt-5.2` y `gpt-5-mini` en el **Model leaderboard** y en el **Trade-off chart**.

**Instrucciones:**

1. En la página del model catalog, seleccione **View leaderboard**.
2. En la página **Model leaderboard**, revise los modelos principales clasificados por quality, safety, cost y performance. Anote qué modelos obtienen las puntuaciones más altas en las métricas de AI quality.
3. Desplácese hacia abajo para usar la sección **Trade-off chart** y comparar modelos en varias dimensiones.
4. Seleccione **Benchmark Cost** en el dropdown para ver cómo se relaciona la quality del modelo con el cost y, a continuación, use la lista de modelos para comparar **gpt-5.2** y **gpt-5-mini**. Si desea explorar más, puede agregar otros modelos a la comparación.
5. Seleccione la métrica **Throughput** en el dropdown para ver cómo se relaciona la quality de estos modelos con las puntuaciones de throughput.
6. Seleccione la métrica **Safety** en el dropdown para ver cómo se relaciona la quality de estos modelos con las puntuaciones de safety.
7. En la tabla justo encima de los trade-off charts puede comparar benchmarks. Seleccione **gpt-5.2** y **gpt-5-mini**, y de forma opcional cualquier otro modelo que desee explorar, y después use el botón **Compare models** para ver sus benchmarks lado a lado.
8. Revise la comparación en los siguientes datos:
   - **Performance benchmarks**: puntuaciones de quality, safety y throughput.
   - **Input** y **output**: los formatos admitidos para prompts y responses.
   - **Context**: el número de tokens que se pueden mantener en una conversación y producir como output, y cuándo se entrenó el modelo.
   - **Endpoints**: los API endpoints a través de los cuales el modelo puede ser consumido por aplicaciones cliente, y si puede ser usado por un agent.
   - **Supported features**: capabilities específicas que puede requerir en el escenario de su aplicación.
9. Use la flecha atrás (**←**) junto al título de la página **gpt-5.2** para volver al model catalog.

**Salida esperada:**

Ha comparado `gpt-5.2` y `gpt-5-mini` en el **Model leaderboard**, en el **Trade-off chart** y en la vista **Compare models**.

**Verificación:**

Anote al menos una diferencia observada entre `gpt-5.2` y `gpt-5-mini` en quality, cost, throughput o safety.

---

### Paso 4. Desplegar modelos

Ahora desplegará los modelos que usará para testing y evaluation. Debe desplegar **gpt-5.2** y **gpt-5-mini**.

**Objetivo:** Crear los deployments de `gpt-5.2` y `gpt-5-mini` y anotar cada deployment name.

#### Deploy the gpt-5.2 model

**Instrucciones:**

1. En el model catalog, busque `gpt-5.2` y selecciónelo.
2. En la página del modelo, seleccione **Deploy** y despliegue el modelo usando los *default settings*.

   El modelo desplegado se abrirá en el model playground, donde quedará seleccionado en la lista desplegable **Model**.
3. Anote el deployment name asignado al modelo **gpt-5.2**. Necesitará identificar este deployment más adelante.

#### Deploy the gpt-5-mini model

**Instrucciones:**

1. En el model playground, en la lista **Model**, seleccione **Browse more models**.
2. Busque `gpt-5-mini` y, a continuación, selecciónelo y despliéguelo.

   El modelo queda desplegado y seleccionado en el model playground.
3. Anote el deployment name asignado al modelo **gpt-5-mini**.

**Salida esperada:**

Los deployments de `gpt-5.2` y `gpt-5-mini` están disponibles y uno de ellos está seleccionado en el model playground.

**Verificación:**

Confirme que anotó el deployment name de **gpt-5.2** y el de **gpt-5-mini**.

---

### Paso 5. Comparar modelos en el model playground

Ahora que tiene dos model deployments, compárelos en el playground.

**Objetivo:** Ejecutar el mismo prompt en ambos deployments y comparar accuracy, reasoning quality y response style.

**Instrucciones:**

1. En el playground, asegúrese de que el deployment del modelo **gpt-5-mini** esté seleccionado en la lista **Models** y, a continuación, en el lado derecho de la página, en la lista **Compare models**, seleccione el deployment del modelo **gpt-5.2**.
2. La vista de comparación lado a lado se abre directamente en paneles de chat separados para cada modelo. Seleccione la pestaña **Chat** para ambos modelos e introduzca el siguiente prompt:

   ```text
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
   ```

3. Envíe el prompt y vea las responses de ambos modelos. Después, introduzca el siguiente follow-up prompt:

   ```text
   Explain your reasoning.
   ```

4. Compare las responses de cada modelo. Anote cualquier diferencia en accuracy, reasoning quality y response style.

**Salida esperada:**

El playground muestra, lado a lado, las responses de `gpt-5-mini` y `gpt-5.2` al puzzle y a `Explain your reasoning.`

**Verificación:**

Confirme que ambas pestañas **Chat** devolvieron una response y que anotó al menos una diferencia de accuracy, reasoning quality o response style.

---

### Paso 6. Evaluar un modelo con un synthetic dataset

El model playground es útil para testing manual rápido, pero para evaluar de forma sistemática el performance de un modelo en muchas entradas puede ejecutar una evaluation. Evaluará el modelo **gpt-5.2** usando un dataset generado de forma sintética con preguntas relacionadas con travel.

**Objetivo:** Crear la evaluation `travel-assistant-eval` sobre `gpt-5.2` con **Synthetic generation** y revisar métricas y fallos.

#### Step 1: Target

**Instrucciones:**

1. En el playground, seleccione la pestaña **Evaluations**.
2. Seleccione **Create** para abrir el asistente **Create new evaluation**.
3. Para el evaluation target, seleccione **Model**.
4. En la tabla de modelos, quite la selección de cualquier deployment preseleccionado de modo que solo quede marcada la casilla de **gpt-5.2** y, a continuación, seleccione **Next**.

#### Step 2: Data

En lugar de cargar un test dataset, usará la funcionalidad de synthetic data generation de Foundry para crear uno automáticamente.

**Instrucciones:**

1. En el paso **Data**, en **Dataset source**, seleccione **Synthetic generation**.

   Con synthetic generation, se usa un deployment para generar automáticamente preguntas para cada target cuando envía la evaluation.
2. Seleccione **Generate** y, a continuación, establezca y confirme lo siguiente:
   - **Name of the new dataset**: *Leave as default*
   - **Model**: gpt-5.2
   - **Number of rows**: 45
   - **Prompt**: `Create various travel related questions, and include some content safety and security tests`
   - **Seed data**: *Leave blank*
3. Seleccione **Next** para continuar.

#### Step 3: Configure models

**Instrucciones:**

1. En el paso **Configure models**, establezca el prompt **Developer** para el modelo que se está evaluando:

   ```text
   You are a helpful travel assistant that provides accurate, detailed, and practical travel advice to help users plan their trips.
   ```

2. Deje el resto de los valores en su default y, a continuación, seleccione **Next**.

#### Step 4: Criteria

**Instrucciones:**

1. En el paso **Criteria**, vea todos los evaluators sugeridos. Estos usan un modelo de IA como judge para evaluar la quality de las responses.
2. Quite todos los criteria de *Agents* y *Safety*, y deje habilitados el resto de los evaluators.
3. Seleccione **Next**.

#### Step 5: Review and submit

**Instrucciones:**

1. En el paso **Review**, verifique la configuración de la evaluation, incluido el target model, el dataset y los criteria seleccionados.
2. Asigne un nombre a la evaluation, por ejemplo `travel-assistant-eval`.
3. Seleccione **Submit** para iniciar el evaluation run.
4. Espere a que la evaluation se complete. Esto puede tardar varios minutos, según la carga del data center.

#### Review the results

**Instrucciones:**

1. Cuando la evaluation se complete, seleccione el evaluation run para ver la página de results, que muestra un overview de las evaluation metrics.
2. Revise las scores y los results de cada evaluation en la tabla detallada de la run page. Desplácese hacia la derecha y vea páginas adicionales, donde verá en su mayoría valores passing. Según la response del modelo, es posible que vea algunos failures. Si los hay, examínelos con atención.
3. Seleccione el botón **Analyze results**, elija **gpt-5.2** en el dropdown y, a continuación, seleccione **Start analysis**.
4. En esta página verá los failures agrupados por la razón por la que fallaron, y podrá ver detalles de por qué falló. La mayoría de esos failures se deberán a que el modelo indica que no puede ayudar por la naturaleza de la pregunta; no obstante, debe explorar cada failure y considerar si la response es la que desea ver.
5. Revise cualquier failure y las AI suggestions sobre cómo mejorar. Esta guía le ayudará a ajustar su configuración para obtener mejor performance.

**Salida esperada:**

La evaluation `travel-assistant-eval` se completó. La run page muestra métricas y, en **Analyze results**, los failures agrupados con AI suggestions.

**Verificación:**

Confirme que el nombre de la evaluation es `travel-assistant-eval`, que el target es `gpt-5.2` y que revisó al menos un result passing o un failure.

---

## Limpieza

Si ha terminado de explorar Microsoft Foundry, debe eliminar los recursos que creó en este ejercicio para evitar incurrir en costos innecesarios de Azure.

1. Abra el [Azure portal](https://portal.azure.com) y vea el contenido del **Resource group** donde implementó los recursos usados en este ejercicio.
2. En la toolbar, seleccione **Delete resource group**.
3. Escriba el nombre del **Resource group** y confirme que desea eliminarlo.

> Si continuará con las prácticas siguientes de este capítulo, conserve el **Resource group**, el proyecto de Foundry y los deployments de `gpt-5.2` y `gpt-5-mini` hasta finalizar esas prácticas.

## Resumen

En este ejercicio exploró el model catalog de Microsoft Foundry, comparó `gpt-5.2` y `gpt-5-mini` con el **Model leaderboard** y el model playground, y ejecutó la evaluation `travel-assistant-eval` con un synthetic dataset de preguntas de travel.

### Recursos opcionales

- [Explore and compare models](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/02-model-catalog-evaluation.html)
- [Portal de Microsoft Foundry](https://ai.azure.com)
- [Disponibilidad regional de la API Responses](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
- [Guía de selección de modelos en Azure AI Foundry](https://learn.microsoft.com/es-es/azure/ai-foundry/how-to/model-catalog-overview)

