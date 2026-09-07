# 1. Práctica 1. Integre un agente de IA con RAG

En este ejercicio usará Microsoft Foundry portal para crear un agent que se integra con Foundry IQ para buscar y recuperar información de knowledge bases. Creará un search resource, configurará una knowledge base con sample data, construirá un agent en el portal y después se conectará a él desde Visual Studio Code para interactuar de forma programática.

> **Sugerencia:** El código usado en este ejercicio se basa en el Microsoft Foundry SDK for Python. Puede desarrollar soluciones similares usando los SDKs para Microsoft .NET, JavaScript y Java. Consulte [Microsoft Foundry SDK client libraries](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para más detalles.

Este ejercicio debería tomar aproximadamente **45** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- Una [Azure subscription](https://azure.microsoft.com/free/) con permisos para crear AI resources
- [Visual Studio Code](https://code.visualstudio.com/) instalado en su equipo local
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Git](https://git-scm.com/downloads) instalado en su equipo local
- Familiaridad básica con Microsoft Foundry portal y programación en Python

> \* Python 3.14 aún no está soportado: algunas dependencies no tienen build para 3.14. Este lab se validó con Python 3.13.12.

## Crear un proyecto de Foundry

Comencemos creando un Foundry project con la nueva experiencia de Foundry.

1. En un web browser, abra el [Foundry portal](https://ai.azure.com) en `https://ai.azure.com` e inicie sesión con sus credenciales de Azure. Cierre cualquier panel de tips o quick start que se abra la primera vez que inicie sesión.

   > **Importante:** Asegúrese de que el toggle **New Foundry** esté *On* para que este lab use la interfaz actualizada.
2. Una vez que active **New Foundry**, se le pedirá seleccionar un project. En el dropdown, seleccione **Create a new project**.
3. En el diálogo **Create a project**, introduzca un nombre válido para su project (por ejemplo, *agent-iq-lab*).
4. Confirme o configure la siguiente settings para su project:
   - **Foundry resource**: *Create a new Foundry resource or select an existing one*
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Create or select a resource group*
   - **Location**: *Select any available region*\*

   > \* Algunos Azure AI resources están limitados por regional model quotas. Si más adelante en el ejercicio se excede un quota limit, es posible que deba crear otro resource en una región diferente.
5. Seleccione **Create** y espere a que se cree su project. Esto puede tardar unos minutos.
6. Cuando se cree su project, verá la project home page.

## Crear un agent

1. En la home page, seleccione la pestaña **Build** y, a continuación, en la pestaña **Agents** seleccione **Create agent**.
2. Cree su agent con un nombre descriptivo, por ejemplo `product-expert-agent`.

Al crear un agent, se hará deploy del modelo predeterminado (como `gpt-5`). Una vez creado el agent, verá el agent playground con ese modelo predeterminado seleccionado automáticamente.

## Configurar sus data y Foundry IQ

Ahora configurará su agent para que use Foundry IQ y busque en la knowledge base.

1. Primero, asigne a su agent las siguientes instructions:

   ```
   You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products. 
   You must ALWAYS search the knowledge base to answer questions about our products or product 
   catalog. Provide detailed, accurate information and always cite your sources.
   If you don't find relevant information in the knowledge base, say so clearly.
   ```

2. Seleccione **Save** para guardar la configuración actual del agent.
3. Después, en la sección **Knowledge**, expanda el dropdown **Add** y seleccione **Connect to Foundry IQ**.
4. En la ventana de Foundry IQ setup, seleccione **Connect to an AI Search resource** y después **Create new resource**, lo que debería abrir un diálogo para crear el resource.
5. Cree un search resource con los default settings:
   - **Resource name**: *A globally unique name*
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Use the same resource group as your project*
   - **Region**: *The same location as your project*
   - **Pricing tier**: Free *if available, otherwise choose Basic*
   - **Foundry IQ Knowledge base capabilities**: Pause til next month

   > **Nota:** Si tiene problemas al crear el resource aquí, seleccione el link en la parte inferior del formulario para crearlo desde el Azure portal.

Ahora cargará sample product information documents para conectarlos con Foundry IQ.

1. Descargue los sample product information files abriendo una pestaña nueva del browser y navegando a `https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/main/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip`
2. Extraiga los files del zip; deberían ser 3 PDFs que detallan los products de Contoso.
3. Abra una pestaña nueva y navegue al Azure portal en `https://portal.azure.com`. En la top search bar, busque **Storage accounts** y seleccione **Storage accounts** en la sección de services.
4. Cree un storage account con la siguiente settings:
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Use the same resource group as your project*
   - **Storage account name**: *A unique storage account name*
   - **Region**: *The same location as your project*
   - **Primary service**: *Azure Blob Storage or Azure Data Lake Storage*
   - **Performance**: *Standard*
   - **Redundancy**: *Locally-redundant storage (LRS)*
5. Una vez creado, vaya al storage account que creó y seleccione **Upload** en la top bar.
6. En el blade **Upload blob**, cree un container nuevo llamado `contosoproducts`.
7. Busque los files extraídos del zip, seleccione los 3 PDF files y seleccione **Upload**.
8. Cuando se hayan cargado los files, navegue al search service que creó.
9. En el panel izquierdo, bajo **Security + networking** > **Keys**, seleccione **Both** para API Access control y confirme la selección. Cuando termine, deje abierta la pestaña del Azure Portal, vuelva a la pestaña del Foundry portal y actualice la página.
10. Verifique que está en la página **Knowledge**, seleccione **Create a knowledge base**, eligiendo **Azure Blob Storage** como knowledge source, y después seleccione **Connect**.
11. Configure su knowledge source con la siguiente settings:
    - **Name**: `ks-contosoproducts`
    - **Description**: `Contoso product catalog items`
    - **Storage account name**: *Select your storage account*
    - **Container name**: `contosoproducts`
    - **Authentication type**: *API Key*
    - **Content extraction mode**: *minimal*
    - **Embedding model**: *Select the available deployed model, likely text-embedding-3-small*
    - **Chat completions model**: *Select the available deployed model, likely gpt-5*
12. Seleccione **Create**.
13. En la knowledge base creation page, seleccione el modelo `gpt-5` en el dropdown **Chat completions model**, dejando el resto de los field defaults como están.
14. Seleccione **Save knowledge base** y después actualice el browser para verificar que el knowledge source status es *active*. Si aún no lo es, espere un minuto y actualice la página hasta que lo sea.
15. Seleccione el botón atrás para volver a la página **Knowledge** y después seleccione el link **Manage** junto al drop-down *Connection*.
16. Desplácese hacia abajo hasta **Connected resources**, donde debería ver su search service. Seleccione esa fila y busque la sección **Authentication**.
17. Seleccione **Key authentication** y después **Edit authentication**.
18. Dejando el diálogo abierto, vuelva a la pestaña del Azure portal, que debería seguir en la página **Keys** de su search service. Copie una de esas keys en el diálogo de Foundry y seleccione **Save**.

La settings de Foundry IQ debería estar completa.

## Probar el Agent en el playground

Antes de conectarse desde código, pruebe su agent en el portal playground.

1. Navegue de nuevo a su agent en la página **Build** > **Agents** y seleccione el agent que creó.
2. En la agent page, debería ver una pestaña playground seleccionada. Busque la sección knowledge y agregue Foundry IQ, seleccionando la connection y la knowledge base que creó.
3. Pruebe las siguientes test queries para verificar que el agent puede recuperar información de la knowledge base:
   - `What types of tents does Contoso offer?`
   - `Tell me about which backpacks are available in XL.`
   - `What camping accessories are available?`
4. Revise las responses y observe:
   - El agent proporciona información específica de la knowledge base
   - Es posible que se incluyan citations o references a los source documents
   - El agent se mantiene enfocado en product information
5. También puede interactuar con su agent en **Preview agent** para una experiencia webapp más refinada.
6. En la agent details page, localice y copie la siguiente información a un notepad (la necesitará más adelante):
   - **Agent name**: Este es el nombre que creó (`product-expert-agent`)
   - **Project endpoint**: Se encuentra en project settings o en la home page

### Configurar el agent para requerir approval en tool calls

Cuando crea un agent en el portal, su Foundry IQ (knowledge) tool se ejecuta **sin** pedir approval de forma predeterminada. Para que su app pueda revisar y controlar cada knowledge base lookup, cambiará el agent para que requiera approval antes de usar tools, con la extensión Foundry Toolkit for VS Code.

> **Nota:** El Foundry portal no expone actualmente un setting para cambiar este approval behavior, así que lo configurará desde la extensión Foundry Toolkit.

1. En Visual Studio Code, seleccione **Extensions** en el panel izquierdo (o presione **Ctrl+Shift+X**), busque en el marketplace la extensión `Foundry Toolkit for VS Code` de Microsoft y seleccione **Install** (si aún no está instalada).

   > **Nota:** La extensión aparece actualmente como **Foundry Toolkit**, pero algunas etiquetas, commands o screenshots antiguos de VS Code aún pueden referirse a **AI Toolkit**. En este lab, trate esos nombres como la misma experiencia de extensión.
2. Seleccione el icono **Foundry Toolkit** en la sidebar e inicie sesión en su Azure account si se le solicita.

   > **Nota:** Si no puede iniciar sesión con la extensión Foundry Toolkit, es posible que deba seleccionar la Azure extension. Inicie sesión ahí y después vuelva a Foundry Toolkit para acceder a sus resources.
3. Bajo **Microsoft Foundry Resources**, elija **Set Default Project** y seleccione el project que creó antes.
4. Expanda la sección del project. Bajo **Prompt Agents**, seleccione su agent `product-expert-agent` para abrir la ventana **Agent Builder**.
5. En la sección **Tools**, ya debería ver una tool cuyo nombre tiene el prefijo `kb-knowledgebase` seguido de un ID único (por ejemplo, `kb-knowledgebase677-7w5fj`). Esta es la Foundry IQ knowledge base tool, y se agregó automáticamente cuando conectó Foundry IQ en el portal.

   > **Nota:** El agent lista más de una tool. El Foundry portal agrega una tool **Web search** a los agents nuevos de forma predeterminada, y también puede ver una tool independiente **Azure AI Search**. El agent realmente llama a la tool `kb-knowledgebase...` cuando busca en su knowledge base, así que configurar approval en cualquier otra tool no tiene efecto.
6. Seleccione el icono de elipsis (**...**) en la tool `kb-knowledgebase...`, después seleccione **Ask for approval for all tools** y, si se le solicita, guarde los cambios.

Su agent ahora pedirá approval cada vez que use Foundry IQ para buscar en la knowledge base, lo cual manejará la client app que completará a continuación.

## Conectarse a su agent desde una app

Ahora creará una Python application para interactuar con su agent de forma programática. Se han proporcionado starter files en el GitHub repository para ayudarle a comenzar rápidamente.

### Prepararse para desarrollar una app en Visual Studio Code

Ahora usemos Visual Studio Code para desarrollar una app. Los code files de su app se han proporcionado en un GitHub repo.

1. Inicie Visual Studio Code y abra la command palette (Shift+Ctrl+P). Después busque y ejecute el comando **Git: Clone** para clonar el repo `https://github.com/MicrosoftLearning/mslearn-ai-agents` en una carpeta local (no importa cuál).
2. Cuando se haya clonado el repository, abra la carpeta en Visual Studio Code.

   > **Nota:** Si Visual Studio Code muestra un mensaje emergente pidiéndole que confíe en el código que está abriendo, haga clic en la opción **Yes, I trust the authors** para continuar.
3. Espere mientras se instalan files adicionales para soportar los Python code projects del repo (si se le solicita).

   > **Nota:** Si se le pide instalar required assets to build and debug, seleccione **Not Now**.
4. En el panel **Explorer**, expanda la carpeta **Labfiles/04-integrate-agent-with-foundry-iq/Python**.

   Los files proporcionados incluyen application code, configuration settings y el agent client starter code.

### Configurar la application settings

1. En Visual Studio Code, en la carpeta **Labfiles/04-integrate-agent-with-foundry-iq/Python**, abra el configuration file **.env**.
2. En el code file, reemplace el placeholder **your_project_endpoint** con el endpoint de su project (copiado de la página **Home** del project en el Foundry portal) y asegúrese de que la variable AGENT_NAME esté establecida con el nombre de su agent (que debería ser *product-expert-agent*).
3. Después de reemplazar el placeholder, guarde el file.

### Completar el agent client code

> **Sugerencia:** Al agregar código, asegúrese de mantener la indentación correcta. Use los comment indentation levels como guía.

1. En Visual Studio Code, en la carpeta **Labfiles/04-integrate-agent-with-foundry-iq/Python**, abra el code file **agent_client.py**.
2. Revise el starter code que se ha proporcionado, incluyendo:
   - Import statements y configuration loading
   - La estructura de la función `send_message_to_agent()`
   - La función `display_conversation_history()`
   - El main program loop
3. Busque el primer comentario **TODO** y agregue el siguiente código para conectarse al project, obtener el OpenAI client, recuperar el agent y crear una conversation nueva:

   > **Sugerencia:** Tenga cuidado de mantener el nivel de indentación correcto.

   ```python
   # Connect to the project and agent
   credential = DefaultAzureCredential(
       exclude_environment_credential=True,
       exclude_managed_identity_credential=True
   )
   project_client = AIProjectClient(
       credential=credential,
       endpoint=project_endpoint
   )

   # Get the OpenAI client
   openai_client = project_client.get_openai_client()

   # Get the agent
   agent = project_client.agents.get(agent_name=agent_name)
   print(f"Connected to agent: {agent.name} (id: {agent.id})\n")

   # Create a new conversation
   conversation = openai_client.conversations.create(items=[])
   print(f"Created conversation (id: {conversation.id})\n")
   ```

4. Busque el segundo comentario **TODO** dentro de la función `send_message_to_agent()` y agregue el siguiente código para enviar messages y manejar responses, incluyendo MCP approval requests:

   ```python
   # Add user message to the conversation
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_message}],
   )

   # Store in conversation history (client-side)
   conversation_history.append({
       "role": "user",
       "content": user_message
   })

   # Create a response using the agent
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       input=""
   )

   # Loop until a response has no pending approval requests (zero, one, or many)
   while True:
       approval_requests = [
           item for item in (getattr(response, "output", None) or [])
           if getattr(item, "type", None) == "mcp_approval_request"
       ]

       if not approval_requests:
           break

       approval_items = []
       for approval_request in approval_requests:
           print(f"[Approval required for: {approval_request.name}]\n")
           print(f"Server: {approval_request.server_label}")

           # Show the tool call arguments for transparency
           import json
           try:
               args = json.loads(approval_request.arguments)
               print(f"Arguments: {json.dumps(args, indent=2)}\n")
           except Exception:
               print(f"Arguments: {approval_request.arguments}\n")

           approval_input = input("Approve this action? (yes/no): ").strip().lower()
           approved = approval_input in ['yes', 'y']
           print("Approving action...\n" if approved else "Action denied.\n")

           approval_items.append({
               "type": "mcp_approval_response",
               "approval_request_id": approval_request.id,
               "approve": approved
           })

       # Send the approval decisions and fetch the next response
       openai_client.conversations.items.create(
           conversation_id=conversation.id,
           items=approval_items
       )

       response = openai_client.responses.create(
           conversation=conversation.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
           input=""
       )
   ```

   > **Nota:** El agent no siempre solicita approval, y ocasionalmente puede solicitar approval para más de un tool call en el mismo turn. Iterar hasta que `approval_requests` esté vacío maneja ambos casos correctamente.
5. Después de agregar el código, guarde el file.
6. Revise que el código ahora usa la conversations API para administrar las interacciones con su agent, donde:
   - Se crea una conversation y se rastrea por su ID
   - Los user messages se agregan a la conversation usando `conversations.items.create()`
   - Las responses se generan usando `responses.create()` con un agent reference
   - **MCP approval handling**: Cuando el agent necesita acceder a Foundry IQ, solicita approval devolviendo uno o más items `mcp_approval_request` en el response output
   - El código itera, pidiéndole que apruebe o niegue cada pending request, hasta que el agent devuelve una response sin outstanding approval requests (incluido el caso en que nunca se necesitó approval)
   - Después de cada approval/denial, se agrega un `mcp_approval_response` a la conversation y se genera una response nueva
   - El agent recupera información de Foundry IQ según su approval decision

## Probar la Integration

Ahora ejecutará su application y probará la capacidad del agent para recuperar información de la knowledge base.

1. En Visual Studio Code, abra una integrated terminal para la carpeta **Labfiles/04-integrate-agent-with-foundry-iq/Python** haciendo clic derecho en la carpeta y seleccionando **Open in Integrated Terminal**.
2. Primero, cree un virtual environment e instale dependencies.

   ```
   python -m venv labenv
   ./labenv/Scripts/activate
   pip install -r requirements.txt
   ```

3. En el panel de la terminal, introduzca el siguiente comando para iniciar sesión en Azure.

   ```
   az login
   ```

   > **Nota:** En la mayoría de los escenarios, usar solo *az login* será suficiente. Sin embargo, si tiene subscriptions en múltiples tenants, es posible que deba especificar el tenant usando el parámetro *--tenant*. Consulte [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más detalles.
4. Cuando se le solicite, siga las instrucciones para abrir la sign-in page en una pestaña nueva e introducir el authentication code proporcionado y sus credenciales de Azure. Después complete el proceso de inicio de sesión en la command line, seleccionando la subscription que contiene su Foundry resource si se le solicita.
5. En el panel de la terminal, ejecute su application:

   ```
   python agent_client.py
   ```

6. Cuando inicie la application, pruebe el agent con las siguientes queries:

   **Query 1 - Product Categories:**

   ```
   What types of outdoor products does Contoso offer?
   ```

   Cuando se le pida approval, escriba **yes** para permitir que el agent busque en la knowledge base. Observe cómo el agent recupera información de múltiples documents de la knowledge base.

   **Query 2 - Specific Product Details:**

   ```
   Tell me about the weatherproof features of your tents.
   ```

   Apruebe el request y observe cómo el agent proporciona detalles específicos del tents catalog.

   **Query 3 - Product Comparisons:**

   ```
   What's the difference between your daypacks and expedition backpacks?
   ```

   Apruebe el request y vea cómo el agent puede sintetizar información de la backpacks guide.

   **Query 4 - Accessories and Add-ons:**

   ```
   What camping accessories would you recommend for a weekend hiking trip?
   ```

   Apruebe el request y observe la capacidad del agent para ofrecer recommendations basadas en la knowledge base.

   **Query 5 - Follow-up Question:**

   ```
   How much do those items typically cost?
   ```

   Observe cómo el agent mantiene el conversation context de su query anterior.
7. Escriba `history` para ver el conversation history completo.
8. Escriba `quit` cuando haya terminado de probar.

### Revisar los results

Considere los siguientes aspectos de las responses del agent:

- **MCP Approval Flow**: Cada vez que el agent necesita acceder a la knowledge base, solicita approval, lo que le da control sobre el external tool usage
- **Accuracy**: El agent proporciona información directamente de los knowledge base documents
- **Citations**: El agent puede incluir source references o document IDs
- **Context awareness**: El agent recuerda previous messages de la conversation
- **Grounding**: El agent indica cuando no puede encontrar información relevante en la knowledge base
- **Error handling**: La application maneja errors y connection issues de forma controlada

## Resumen

En este ejercicio usted:

- Creó un Foundry project y un agent con la nueva Foundry UI
- Construyó una knowledge base con product information documents
- Configuró un agent en el portal con Foundry IQ habilitado
- Se conectó a su agent desde Visual Studio Code usando el Python SDK
- Implementó una client application con MCP approval handling, conversation history y error handling
- Probó la capacidad del agent para recuperar y sintetizar información de la knowledge base con approval controlado por el usuario para el external tool access

Esto demuestra cómo integrar AI agents con Foundry IQ para crear applications inteligentes que pueden buscar y recuperar información de enterprise knowledge bases manteniendo el conversational context.

## Limpieza

Si ha terminado de explorar Azure AI Agent Service y Foundry IQ, debe eliminar los resources que ha creado en este ejercicio para evitar incurrir en costos innecesarios de Azure.

1. En un web browser, abra el [Azure portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Navegue al resource group que contiene su Foundry resource y sus AI Search resources.
3. En la toolbar, seleccione **Delete resource group**.
4. Introduzca el resource group name y confirme que desea eliminarlo.
