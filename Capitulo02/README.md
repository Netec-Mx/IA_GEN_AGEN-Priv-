# 1. Práctica 1. Implementar un cliente Python que consuma modelos de OpenAI y Claude mediante sus SDK oficiales, configurando parámetros de inferencia y comparando sus respuestas frente a distintos escenarios de negocio. 


En este ejercicio usará el OpenAI SDK y la Responses API para crear una chat app que se conecta a un modelo desplegado en un proyecto de Microsoft Foundry.

Este ejercicio toma aproximadamente **45** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- Una [Azure subscription](https://azure.microsoft.com/pricing/purchase-options/azure-account) activa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python version **3.13.xx](https://www.python.org/downloads/release/python-31312/)** instalado
- [Git](https://git-scm.com/install/) instalado y configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

>  Python 3.14 está disponible, pero algunas dependencies aún no están compiladas para esa versión. El lab se ha validado correctamente con Python 3.13.12.



## Crear un proyecto de Microsoft Foundry

Microsoft Foundry usa projects para organizar models, resources, data y otros assets usados para desarrollar una solución de IA.

1. En un web browser, abra el [Microsoft Foundry portal](https://ai.azure.com) en `https://ai.azure.com` para empezar a construir; inicie sesión con sus credenciales de Azure. Cierre cualquier panel de tips o quick start que se abra la primera vez que inicie sesión.
2. Si aún no está habilitada, en la tool bar de la parte superior de la página, active la opción **New Foundry**. Después, si se le solicita, cree un project nuevo con un nombre único; expandiendo el área **Advanced options** para especificar la siguiente configuración de su project:
  - **Foundry resource**: *Use the default name for your resource (usually {project_name}-resource)*
  - **Subscription**: *Your Azure subscription*
  - **Resource group**: *Create or select a resource group*
  - **Region**: Seleccione cualquiera de las regiones **AI Foundry recommended** en [esta lista](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
3. Espere a que se cree su project. Después, vea su home page.



## Desplegar un modelo

A continuación, vamos a hacer deploy de un modelo que usará en su chat application.

1. Ahora está listo para explorar models. En la página **Discover**, seleccione la pestaña **Models** para ver el Microsoft Foundry model catalog.
2. En el model catalog, busque `gpt-5.2`.
3. Revise la model card y, a continuación, haga deploy usando los default settings.
4. Cuando el modelo se haya desplegado, se abrirá en el model playground; puede probarlo ahí si lo desea.



## Obtener el endpoint

Necesitará un endpoint para conectarse al modelo desde una client application. En este ejercicio vamos a usar el OpenAI SDK para chatear con el modelo; y usaremos el Azure OpenAI endpoint con Entra ID authentication para conectarnos a él.

> **Nota:** Como alternativa a Entra ID authentication, podría usar el API Key del project. Usar Entra ID authentication es preferible siempre que sea posible.

1. En la menu bar, seleccione la página **Home**.
2. Anote el **Azure OpenAI Endpoint** que se muestra ahí.
  > **Sugerencia:** En este ejercicio usará el **Azure OpenAI Endpoint**, no el project endpoint.



## Crear una aplicación cliente para chatear con el modelo

Ahora que ha hecho deploy de un modelo, puede usar el OpenAI SDK y la Responses API para desarrollar una application que converse con él.

### Obtener los archivos de la aplicación desde GitHub

Los application files iniciales que necesitará para desarrollar su chat application se proporcionan en un GitHub repo.

1. Abra Visual Studio Code.
2. Abra la command palette (*Ctrl+Shift+P*) y use el comando `Git:clone` para clonar el repo `https://github.com/microsoftlearning/mslearn-ai-studio` en una carpeta local (no importa cuál). Después, ábralo.
  Es posible que se le pida confirmar que confía en los authors.



### Preparar la configuración de la aplicación

1. En Visual Studio Code, vea el panel **Extensions**; y si aún no está instalada, instale la extensión **Python**.
2. En la **Command Palette**, use el comando `python:select interpreter`. Después, cree un entorno **Venv** nuevo basado en su instalación de Python 3.13.
  > **Sugerencia:** Si se le pide instalar dependencies, puede instalar las del archivo *requirements.txt* en la carpeta */labfiles/foundry-chat/python/chat-app*; pero no hay problema si no lo hace: las instalaremos más adelante.
3. En el panel Explorer, navegue a la carpeta que contiene los application code files en **/labfiles/foundry-chat/python/chat-app**. Los application files incluyen:
  - **.env** (the application configuration file)
  - **requirements.txt** (the Python package dependencies that need to be installed)
  - **chat-app.py** (the code file for the chat application)
  - **chat-async.py** (the code file for an asynchronous version of the application)
4. En el panel **Explorer**, haga clic derecho en la carpeta **chat-app** que contiene los application files y seleccione **Open in integrated terminal** (o abra una terminal en el menú **Terminal** y navegue a la carpeta */labfiles/foundry-chat/python/chat-app*).
  > **Nota:** Abrir la terminal en Visual Studio Code activará automáticamente el Python environment. Es posible que deba habilitar la ejecución de scripts en su sistema.
5. Asegúrese de que la terminal esté abierta en la carpeta **labfiles/foundry-chat/python/chat-app** con el prefijo **(.venv)** para indicar que el Python environment que creó está activo.
6. Instale el OpenAI SDK, Azure Identity y los demás required packages ejecutando el siguiente comando:
  ```
   pip install -r requirements.txt
  ```
7. En el panel **Explorer**, en la carpeta **labfiles/foundry-chat/python/chat-app**, seleccione el archivo **.env** para abrirlo. Después, actualice los configuration values para incluir el **Azure OpenAI Endpoint** y el name assigned to the deployment del modelo **gpt-5.2**.
  > **Sugerencia:** Copie el **Azure OpenAI Endpoint** (no el project endpoint) desde la project home page en el Foundry portal, e introduzca el exact deployment name asignado a su deployment en el setting `MODEL_DEPLOYMENT`.
   Guarde el configuration file modificado.



### Usar la API *ChatCompletions* para chatear con el modelo

La *ChatCompletions* API es una forma consolidada de crear client applications para large language models, y ha sido ampliamente adoptada.

1. En el panel **Explorer**, en la carpeta **labfiles/foundry-chat/python/chat-app**, seleccione el archivo **chat-app.py** (no *chat-async.py*) para abrirlo.
2. Revise el código existente. Agregará código para usar el OpenAI SDK y acceder a su modelo.
  > **Sugerencia:** Al agregar código al code file, asegúrese de mantener la indentación correcta.
3. En la parte superior del code file, bajo las namespace references existentes, busque el comentario **Import namespaces** y agregue el siguiente código para importar el namespace que necesitará para usar el OpenAI SDK:
  ```python
   # import namespaces
   from openai import OpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
  ```
4. En la función **main**, observe que ya se ha proporcionado el código para cargar el endpoint y la key desde el configuration file. Después, busque el comentario **Initialize the OpenAI client** y agregue el siguiente código para crear un client para la OpenAI API:
  ```python
   # Initialize the OpenAI client
   token_provider = get_bearer_token_provider(
        DefaultAzureCredential(), "https://ai.azure.com/.default"
   )

   openai_client = OpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
  ```
5. En la función **main**, observe que ya se ha proporcionado el código para solicitar un user prompt hasta que el usuario cierre la app. Dentro de este loop, busque el comentario **Get a response** y agregue el siguiente código:
  ```python
   # Get a response
   completion = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {
                "role": "system",
                "content": "You are a helpful AI assistant that answers questions and provides information."
            },
            {
                "role": "user",
                "content": input_text
            }
        ]
   )
   print(completion.choices[0].message.content)
  ```
   Observe que la *ChatCompletions* API usa una colección JSON de *messages* para encapsular la conversation. Con frecuencia, estas consisten en un *system prompt* que proporciona instructions al modelo, y un *user prompt* que incluye el input del usuario.
6. Guarde los cambios en el code file. Después, en el panel de la terminal, use el siguiente comando para iniciar sesión en Azure.
  ```powershell
   az login
  ```
  > **Nota:** En la mayoría de los escenarios, usar solo *az login* será suficiente. Sin embargo, si tiene subscriptions en múltiples tenants, es posible que deba especificar el tenant usando el parámetro *--tenant*. Consulte [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más detalles.
7. Cuando se le solicite, siga las instrucciones para iniciar sesión en Azure. Después, complete el proceso de inicio de sesión en la command line, viendo (y confirmando si es necesario) los detalles de la subscription que contiene su Foundry resource.
8. Después de haber iniciado sesión, introduzca el siguiente comando para ejecutar la application:
  ```powershell
   python chat-app.py
  ```
   El program debería ejecutarse en la terminal (si no, resuelva cualquier error e inténtelo de nuevo).
9. Cuando se le solicite, introduzca el siguiente prompt:
  ```input
   Tell me about the ELIZA chatbot.
  ```
   Después de unos momentos, la app debería responder con información sobre el ELIZA chatbot creado en la década de 1960.
10. Introduzca el prompt `quit` para finalizar la application.



### Usar la API *Responses* para chatear con el modelo

Aunque la *ChatCompletions* API se usa ampliamente, cada vez está siendo sustituida por la *Responses* API más reciente. Actualicemos el código para usarla.

1. En el código de **chat-app.py**, en la función **main**, reemplace el código bajo el comentario **Get a response** con el siguiente código que usa la *Responses* API.
  ```python
   # Get a response
   response = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text
   )
   print(response.output_text)
  ```
   Observe la sintaxis más simple, en la que el system message se asigna al parámetro *instructions* y el user prompt se asigna al parámetro *input*.
2. Guarde los cambios en el código y, en el panel de la terminal, vuelva a ejecutar la application (`python chat-app.py`).
3. Cuando se le solicite, introduzca el mismo prompt que antes:
  ```input
   Tell me about the ELIZA chatbot.
  ```
   Después de unos momentos, la app debería responder de nuevo con información sobre el ELIZA chatbot.
4. Introduzca el siguiente prompt para intentar continuar la conversation:
  ```input
   How does it compare to modern LLMs?
  ```
   La app debería responder de un modo que indique que no entiende a qué se refiere “it”. El conversation context se ha perdido. Eso lo corregiremos.
5. Introduzca el prompt `quit` para finalizar la application.



### Agregar seguimiento de la conversación

Para mantener el conversational context, necesitamos incluir references a previous responses en cada new request.

1. En el código de **chat-app.py**, en la función **main**, busque el comentario **Loop until the user wants to quit** y agregue el siguiente código encima (*before* the loop):
  ```python
   # Track responses
   last_response_id = None
  ```
2. Modifique el código bajo el comentario **Get a response** con el siguiente código para pasar el previous response ID en el request y, a continuación, obtener el new response ID para poder agregarlo la próxima vez.
  ```python
   # Get a response
   response = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id,
   )
   print(response.output_text)
   last_response_id = response.id
  ```
   Con esta técnica puede pasar el ID de la previous response para mantener context. También podría implementar una lógica más compleja para pasar un ID de cualquier previous response y redirigir una conversation o reanudar un previous conversational thread.
3. Guarde los cambios en el código y, en el panel de la terminal, vuelva a ejecutar la application (`python chat-app.py`).
4. Cuando se le solicite, introduzca el mismo prompt que antes:
  ```input
   Tell me about the ELIZA chatbot.
  ```
   Después de unos momentos, la app debería responder de nuevo con información sobre el ELIZA chatbot.
5. Introduzca el siguiente prompt para intentar continuar la conversation:
  ```input
   How does it compare to modern LLMs?
  ```
   Esta vez, la app debería responder con una comparación del ELIZA chatbot y los modern LLMs. La response puede ser bastante extensa, y la app espera hasta haberla recibido completa del modelo antes de mostrarla, lo que puede hacer que la app parezca unresponsive. Eso lo corregiremos a continuación.
6. Introduzca el prompt `quit` para finalizar la application.



### Implementar respuestas en *streaming*

Para manejar responses largas, puede usar *streaming* para empezar a procesar partial responses antes de que se haya devuelto el texto completo.

1. En el código de **chat-app.py**, en la función **main**, reemplace el código bajo el comentario **Get a response** con el siguiente código que usa *streaming*.
  ```python
   # Get a response
   stream = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id,
                stream=True
   )
   for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="")
        elif event.type == "response.completed":
            last_response_id = event.response.id
   print()
  ```
   Observe que el parámetro *stream=True* crea una streamed response en la que ocurren *events* a medida que cada new chunk (o *delta*) está listo para procesarse.
2. Guarde los cambios en el código y, en el panel de la terminal, vuelva a ejecutar la application (`python chat-app.py`).
3. Cuando se le solicite, introduzca el mismo prompt que antes:
  ```input
   Tell me about the ELIZA chatbot.
  ```
   Después de unos momentos, la app debería empezar a responder con información sobre el ELIZA chatbot. La response debería aparecer de forma incremental a medida que se devuelve cada chunk.
4. Introduzca el siguiente prompt para intentar continuar la conversation:
  ```input
   How does it compare to modern LLMs?
  ```
   De nuevo, la response debería mostrarse de forma incremental.
5. Introduzca el prompt `quit` para finalizar la application.



### Usar la API asincrónica

El OpenAI SDK ofrece una opción asynchronous que puede aumentar la responsiveness de las applications cuando se usan operaciones de model o agent de larga duración.

1. En el panel **Explorer**, en la carpeta **labfiles/foundry-chat/python/chat-app**, seleccione el archivo **chat-async.py** (no *chat-app.py*) para abrirlo.
2. Revise el código existente. Agregará código para usar la OpenAI SDK async API y acceder a su modelo.
  > **Sugerencia:** Al agregar código al code file, asegúrese de mantener la indentación correcta.
3. En la parte superior del code file, bajo las namespace references existentes, busque el comentario **Import namespaces** y agregue el siguiente código para importar el namespace que necesitará para usar el OpenAI SDK:
  ```python
   # import namespaces for async
   import asyncio
   from openai import AsyncOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
  ```
4. En la función **main**, observe que ya se ha proporcionado el código para cargar el endpoint y la key desde el configuration file. Después, busque el comentario **Initialize an async OpenAI client** y agregue el siguiente código para crear un client para la OpenAI API:
  ```python
   # Initialize an async OpenAI client
   credential = DefaultAzureCredential()
   token_provider = get_bearer_token_provider(
    credential, "https://ai.azure.com/.default"
   )

   async_client = AsyncOpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
  ```
5. En la función **main**, observe que ya se ha proporcionado el código para solicitar un user prompt hasta que el usuario cierre la app. Dentro de este loop, busque el comentario **Await an asynchronous response** y agregue el siguiente código:
  ```python
   # Await an asynchronous response
   response = await async_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id
   )
   assistant_text = response.output_text
   print("Assistant:", assistant_text)
   last_response_id = response.id
  ```
   Este código awaits an asynchronous response del modelo.
6. Al final de la función **main**, en el bloque **finally**, busque el comentario **Close the async client session** y agregue el siguiente código para cerrar el asynchronous client:
  ```python
   # Close the async client session
    await credential.close()
  ```
7. Guarde los cambios en el code file. Después, en el panel de la terminal, use el siguiente comando para ejecutar el program:
  ```powershell
   python chat-async.py
  ```
   El program debería ejecutarse en la terminal (si no, resuelva cualquier error e inténtelo de nuevo).
8. Cuando se le solicite, introduzca el siguiente prompt:
  ```input
   Tell me about the Turing test.
  ```
   Después de unos momentos, la app debería responder con información sobre el Turing test.
9. Introduzca el prompt `quit` para finalizar la application.



## Resumen

En este ejercicio usó el OpenAI SDK y las APIs *ChatCompletions* y *Responses* para crear una client application para un generative AI model que desplegó en un Microsoft Foundry project. Personalizó el comportamiento del modelo haciendo tracking del conversational context e implementó streaming para ofrecer una experiencia de chat responsive.

## Limpieza

Si ha terminado de explorar Microsoft Foundry, debe eliminar los resources que ha creado en este ejercicio para evitar incurrir en costos innecesarios de Azure.

1. Abra el [Azure portal](https://portal.azure.com) y vea el contenido del resource group donde implementó los resources usados en este ejercicio.
2. En la toolbar, seleccione **Delete resource group**.
3. Introduzca el resource group name y confirme que desea eliminarlo.

---

# 2. Práctica 2. Cree una aplicación de IA generativa que usa herramientas

En este ejercicio usará el Microsoft Foundry portal y la Responses API para crear una AI chat application. Después integrará knowledge en su application usando las tools *web_search* y *file_search*.

Este ejercicio toma aproximadamente **30** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- Una [Azure subscription](https://azure.microsoft.com/pricing/purchase-options/azure-account) activa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python version **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado y configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

> \* Python 3.14 está disponible, pero algunas dependencies aún no están compiladas para esa versión. El lab se ha validado correctamente con Python 3.13.12.

## Crear un proyecto de Microsoft Foundry

Microsoft Foundry usa projects para organizar models, resources, data y otros assets usados para desarrollar una solución de IA.

1. En un web browser, abra el [Microsoft Foundry portal](https://ai.azure.com) en `https://ai.azure.com` para empezar a construir; inicie sesión con sus credenciales de Azure. Cierre cualquier panel de tips o quick start que se abra la primera vez que inicie sesión.
2. Si aún no está habilitada, en la tool bar de la parte superior de la página, active la opción **New Foundry**. Después, si se le solicita, cree un project nuevo con un nombre único; expandiendo el área **Advanced options** para especificar la siguiente configuración de su project:
   - **Foundry resource**: *Use the default name for your resource (usually {project_name}-resource)*
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Create or select a resource group*
   - **Region**: Seleccione cualquiera de las regiones **AI Foundry recommended** en [esta lista](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#region-availability)
3. Espere a que se cree su project. Después, vea su home page.

## Desplegar un modelo

A continuación, vamos a hacer deploy de un modelo que usará en su chat application.

1. Ahora está listo para explorar models. En la página **Discover**, seleccione la pestaña **Models** para ver el Microsoft Foundry model catalog.
2. En el model catalog, busque `gpt-5.2`.
3. Revise la model card y, a continuación, haga deploy usando los default settings.
4. Cuando el modelo se haya desplegado, se abrirá en el model playground.

## Experimentar con tools en el playground

Antes de desarrollar una chat application, exploremos cómo responde el modelo en el playground. Esto le ayudará a entender por qué importan los grounding data.

1. Después de hacer deploy de su modelo, debería estar en el playground con ese modelo seleccionado. Si no, seleccione **Build** en la top menu bar, después seleccione **Deployments** a la izquierda y, a continuación, seleccione el modelo que desplegó.
2. En el model playground, en el panel de la izquierda, asegúrese de que su modelo **gpt-5.2** esté seleccionado.
3. En el campo **Instructions**, introduzca el siguiente prompt:

   ```
   You are a travel assistant that provides information on travel services available from Margie's Travel.
   ```

4. En el chat pane, introduzca la query `What are some recommended tourist activities in New York next month?` y revise la response.

   La response debería ser bastante genérica: el modelo proporciona conocimiento general basado en sus training data, pero no tiene acceso a información actual sobre lo que ocurre en New York el próximo mes.
5. En el panel de la izquierda, bajo las instructions, en la sección **Tools**, seleccione **Add** y agregue la tool **web_search**.
6. En el chat pane, introduzca la misma query `What are some recommended tourist activities in New York next month?` y revise la response.

   Esta vez, el modelo usa la tool *web_search* para encontrar información actual sobre activities en New York.

## Crear una app que usa tools

Ahora que ha visto cómo las tools pueden extender las capabilities de un modelo en el playground, vamos a crear una client application que usa tools para ofrecer travel advice a los customers de Margie's Travel.

## Obtener el endpoint

Necesitará un endpoint para conectarse al modelo desde una client application. En este ejercicio vamos a usar el OpenAI SDK para chatear con el modelo; y usaremos el Azure OpenAI endpoint con Entra ID authentication para conectarnos a él.

> **Nota:** Como alternativa a Entra ID authentication, podría usar el API Key del project. Usar Entra ID authentication es preferible siempre que sea posible.

1. En la menu bar, seleccione la página **Home**.
2. Anote el **Azure OpenAI Endpoint** que se muestra ahí.

   > **Sugerencia:** En este ejercicio usará el **Azure OpenAI Endpoint**, no el project endpoint.

### Obtener los archivos de la aplicación desde GitHub

Los application files iniciales que necesitará para desarrollar su chat application se proporcionan en un GitHub repo.

1. Abra Visual Studio Code.
2. Abra la command palette (*Ctrl+Shift+P*) y use el comando `Git:clone` para clonar el repo `https://github.com/microsoftlearning/mslearn-ai-studio` en una carpeta local (no importa cuál). Después, ábralo.

   Es posible que se le pida confirmar que confía en los authors.

### Preparar la configuración de la aplicación

1. En Visual Studio Code, vea el panel **Extensions**; y si aún no está instalada, instale la extensión **Python**.
2. En la **Command Palette**, use el comando `python:select interpreter`. Después, seleccione un environment existente si tiene uno, o cree un entorno **Venv** nuevo basado en su instalación de Python 3.1x.

   > **Sugerencia:** Si se le pide instalar dependencies, puede instalar las del archivo *requirements.txt* en la carpeta */labfiles/tools/python/tools-app*; pero no hay problema si no lo hace: las instalaremos más adelante.
3. En el panel Explorer, navegue a la carpeta que contiene los application code files en **/labfiles/tools/python/tools-app**. Los application files incluyen:
   - **brochures** (a folder containing Margie's Travel brochures)
   - **.env** (the application configuration file)
   - **requirements.txt** (the Python package dependencies that need to be installed)
   - **tools-app.py** (the code file for the application)
4. En el panel **Explorer**, haga clic derecho en la carpeta **tools-app** que contiene los application files y seleccione **Open in integrated terminal** (o abra una terminal en el menú **Terminal** y navegue a la carpeta */labfiles/tools/python/tools-app*).

   > **Nota:** Abrir la terminal en Visual Studio Code activará automáticamente el Python environment. Es posible que deba habilitar la ejecución de scripts en su sistema.
5. Asegúrese de que la terminal esté abierta en la carpeta **/labfiles/tools/python/tools-app** con el prefijo **(.venv)** para indicar que el Python environment que creó está activo.
6. Instale el OpenAI SDK, Azure identity y los demás required packages ejecutando el siguiente comando:

   ```
   pip install -r requirements.txt
   ```

7. En el panel **Explorer**, en la carpeta **/labfiles/tools/python/tools-app**, seleccione el archivo **.env** para abrirlo. Después, actualice los configuration values para incluir el **Azure OpenAI Endpoint** y el name assigned to the deployment del modelo **gpt-5.2**.

   > **Sugerencia:** Copie el **Azure OpenAI Endpoint** (no el project endpoint) desde la project home page en el Foundry portal, e introduzca el exact deployment name asignado a su deployment en el setting `MODEL_DEPLOYMENT`.

   Guarde el configuration file modificado.

### Escribir código para implementar chat con tools

1. En el panel **Explorer**, en la carpeta **/labfiles/tools/python/tools-app**, seleccione el archivo **tools-app.py** para abrirlo.
2. Revise el código existente. Agregará código para usar el OpenAI SDK y acceder a su modelo.

   > **Sugerencia:** Al agregar código al code file, asegúrese de mantener la indentación correcta.
3. En la parte superior del code file, bajo las namespace references existentes, busque el comentario **Import namespaces** y agregue el siguiente código para importar el namespace que necesitará para usar el OpenAI SDK:

   ```python
   # import namespaces
   from openai import OpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   ```

4. En la función **main**, observe que ya se ha proporcionado el código para cargar el endpoint y la key desde el configuration file. Después, busque el comentario **Initialize the OpenAI client** y agregue el siguiente código para crear un client para la OpenAI API:

   ```python
   # Initialize the OpenAI client
   token_provider = get_bearer_token_provider(
        DefaultAzureCredential(), "https://ai.azure.com/.default"
   )

   openai_client = OpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
   ```

5. En la función **main**, busque el comentario **Create vector store and upload files** y agregue el siguiente código:

   ```python
   # Create vector store and upload files
   print("Creating vector store and uploading files...")
   vector_store = openai_client.vector_stores.create(
        name="travel-brochures"
   )
   file_streams = [open(f, "rb") for f in glob.glob("brochures/*.pdf")]
   if not file_streams:
        print("No PDF files found in the brochures folder!")
        return
   file_batch = openai_client.vector_stores.file_batches.upload_and_poll(
        vector_store_id=vector_store.id,
        files=file_streams
   )
   for f in file_streams:
        f.close()
   print(f"Vector store created with {file_batch.file_counts.completed} files.")
   ```

   Este código crea un vector store para su modelo y carga los brochures en él. Usaremos este vector store con la tool *file_search*.
6. En la función **main**, observe que ya se ha proporcionado el código para solicitar un user prompt hasta que el usuario cierre la app. Dentro de este loop, busque el comentario **Get a response using tools** y agregue el siguiente código:

   ```python
   # Get a response using tools
   response = openai_client.responses.create(
        model=model_deployment,
        instructions="""
        You are a travel assistant that provides information on travel services available from Margie's Travel.
        Answer questions about services offered by Margie's Travel using the provided travel brochures.
        Search the web for general information about destinations or current travel advice.
        """,
        input=input_text,
        previous_response_id=last_response_id,
        tools=[
            {
                "type": "file_search",
                "vector_store_ids": [vector_store.id]
            },
            {
                "type": "web_search"
            }
        ]
   )
   print(response.output_text)
   last_response_id = response.id
   ```

   Este código envía un prompt y especifica que la tool *file_search* se puede usar para buscar en el vector store y que la tool *web_search* se puede usar para búsquedas web generales.
7. Guarde los cambios en el code file. Después, en el panel de la terminal, use el siguiente comando para iniciar sesión en Azure.

   ```powershell
   az login
   ```

   > **Nota:** En la mayoría de los escenarios, usar solo *az login* será suficiente. Sin embargo, si tiene subscriptions en múltiples tenants, es posible que deba especificar el tenant usando el parámetro *--tenant*. Consulte [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más detalles.
8. Cuando se le solicite, siga las instrucciones para iniciar sesión en Azure. Después, complete el proceso de inicio de sesión en la command line, viendo (y confirmando si es necesario) los detalles de la subscription que contiene su Foundry resource.
9. Después de haber iniciado sesión, introduzca el siguiente comando para ejecutar la application:

   ```powershell
   python tools-app.py
   ```

   El program debería ejecutarse en la terminal (si no, resuelva cualquier error e inténtelo de nuevo).
10. Cuando se le solicite, introduzca `What's happening in San Francisco next month?` y revise la response de su generative AI model.

    La response debería incluir información recuperada usando la tool *web_search*.
11. Pruebe esta follow-up question: `What hotels does Margie's Travel offer there?`

    La response debería incluir información recuperada usando la tool *file_search*.
12. Cuando haya terminado, introduzca `quit` para salir del program.

## Limpieza

Si ha terminado de explorar Microsoft Foundry, debe eliminar los resources que ha creado en este ejercicio para evitar incurrir en costos innecesarios de Azure.

1. Abra el [Azure portal](https://portal.azure.com) y vea el contenido del resource group donde implementó los resources usados en este ejercicio.
2. En la toolbar, seleccione **Delete resource group**.
3. Introduzca el resource group name y confirme que desea eliminarlo.

