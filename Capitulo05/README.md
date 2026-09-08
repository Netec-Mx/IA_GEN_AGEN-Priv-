# 1. Práctica 1. Extienda agents con tools de Model Context Protocol (MCP)

En este ejercicio usará la extensión Foundry Toolkit for VS Code para crear un agent que pueda usar tools de un Model Context Protocol (MCP) server y acceder a external data sources y APIs. El agent podrá recuperar información actualizada e interactuar con custom services a través de MCP tools.

Este ejercicio debería tomar aproximadamente **60** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- [Visual Studio Code](https://code.visualstudio.com/) instalado en su equipo local
- Una [Azure subscription](https://azure.microsoft.com/free/) activa
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Git](https://git-scm.com/downloads) instalado en su equipo local

> \* Python 3.14 aún no está soportado: algunas dependencies no tienen build para 3.14. Este lab se validó con Python 3.12.

## Crear un proyecto de Foundry con la extensión Foundry Toolkit for VS Code

Como desarrollador, es posible que pase algún tiempo trabajando en el Foundry portal; pero también es probable que pase mucho tiempo en Visual Studio Code. La extensión Foundry Toolkit for VS Code ofrece una forma conveniente de trabajar con Foundry project resources sin salir del entorno de desarrollo.

1. Abra Visual Studio Code.
2. Seleccione **Extensions** en el panel izquierdo (o presione **Ctrl+Shift+X**).
3. Busque en el marketplace de extensiones la extensión `Foundry Toolkit` de Microsoft y seleccione **Install**.

   > **Nota:** La extensión aparece actualmente como **Foundry Toolkit**, pero algunas etiquetas, commands o screenshots antiguos de VS Code aún pueden referirse a **AI Toolkit**. En este lab, trate esos nombres como la misma experiencia de extensión.
4. Después de instalar la extensión, seleccione su icono en la sidebar para abrir la vista Foundry Toolkit.

   Se le debería pedir iniciar sesión en su Azure account si aún no lo ha hecho.
5. Seleccione **Create Project** bajo **Microsoft Foundry Resources**.

   Si ya hay un default project activo, el project name aparecerá bajo **My Resources**. Puede crear un project nuevo haciendo clic derecho en el project activo y seleccionando **Switch Default Project in Azure Resources**.
6. Seleccione su Azure subscription y resource group, y después introduzca un nombre para su Foundry project a fin de crear un project nuevo para este ejercicio.

   Cuando el deployment se complete, debería ver el project en el panel Foundry Toolkit como default project.

## Desplegar un modelo

En el núcleo de cualquier generative AI project hay, como mínimo, un generative AI model. En esta tarea, hará deploy de un modelo desde el Model Catalog para usarlo con su agent.

1. Cuando aparezca el popup “Project deployed successfully”, seleccione el botón **Deploy a new model**. Esto abre el Model Catalog.

   > **Sugerencia:** También puede acceder al Model Catalog seleccionando el icono **+** junto a **Models** en la sección Resources, o presionando **F1** y ejecutando el comando **Foundry Toolkit: Show model catalog**.
2. En el Model Catalog, localice el modelo **gpt-5** (puede usar la search bar para encontrarlo rápidamente).
3. Seleccione **Deploy** junto al modelo gpt-5.
4. Configure la deployment settings:
   - **Deployment name**: Introduzca un nombre como “gpt-5”
   - **Deployment type**: Seleccione **Global Standard** (o **Standard** si Global Standard no está disponible)
   - **Model version**: Leave as default
   - **Tokens per minute**: Suba el límite de Tokens per Minute a 150000 o superior.
5. Seleccione **Deploy to Microsoft Foundry** en la esquina inferior izquierda.
6. Espere a que se complete el deployment. El modelo desplegado aparecerá bajo la sección **Models** en la vista Resources.
7. Haga clic derecho en el nombre del project deployment y seleccione **Copy Project Endpoint**. Necesitará esta URL para conectar su agent al Foundry project en los siguientes pasos.

## Clonar el starter code repository

Para este ejercicio usará starter code que le ayudará a conectarse a su Foundry project y crear un agent que usa MCP server tools.

1. En VS Code, abra la Command Palette (**Ctrl+Shift+P** o **View > Command Palette**).
2. Escriba **Git: Clone** y selecciónelo de la lista.
3. Introduzca la repository URL:

   ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
   ```

4. Elija una ubicación en su equipo local para clonar el repository.
5. Cuando se le solicite, seleccione **Open** para abrir el repository clonado en VS Code.
6. Una vez que se abra el repository, seleccione **File > Open Folder** y navegue a `mslearn-ai-agents/Labfiles/03-mcp-integration`; después elija **Select Folder**.
7. En el panel Explorer, expanda la carpeta **Python** para ver los code files de este ejercicio.
8. Haga clic derecho en el archivo **requirements.txt** y seleccione **Open in Integrated Terminal**.
9. En la terminal, introduzca el siguiente comando para instalar los required Python packages en un virtual environment:

   ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

10. Abra el archivo **.env**, reemplace el placeholder **your_project_endpoint** con el endpoint de su project (copiado del project deployment resource en la extensión Foundry Toolkit) y asegúrese de que la variable MODEL_DEPLOYMENT_NAME esté establecida con su model deployment name. Use **Ctrl+S** para guardar el archivo después de hacer estos cambios.

Ahora está listo para crear un AI agent que usa MCP server tools para acceder a external data sources y APIs.

## Conectar un Azure AI Agent a un remote MCP server

En esta tarea se conectará a un remote MCP server, preparará el AI agent y ejecutará un user prompt.

1. Abra el archivo **agent.py** en el code editor.

   > **Sugerencia:** Al agregar código, asegúrese de mantener la indentación correcta. Use los comment indentation levels como guía.
2. Busque el comentario **Add references** y agregue el siguiente código para importar las classes:

   ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import PromptAgentDefinition, MCPTool
   from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
   ```

3. Busque el comentario **Connect to the agents client** y agregue el siguiente código para conectarse al Azure AI project usando las current Azure credentials.

   ```python
   # Connect to the agents client
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
   ```

4. Bajo el comentario **Initialize agent MCP tool**, agregue el siguiente código:

   ```python
   # Initialize agent MCP tool
   mcp_tool = MCPTool(
       server_label="api-specs",
       server_url="https://learn.microsoft.com/api/mcp",
       require_approval="always",
   )
   ```

   Este código se conectará al Microsoft Learn Docs remote MCP server. Este es un cloud-hosted service que permite a los clients acceder a información de confianza y actualizada directamente desde la documentación oficial de Microsoft.
5. Bajo el comentario **Create a new agent with the MCP tool**, agregue el siguiente código:

   ```python
   # Create a new agent with the MCP tool
   agent = project_client.agents.create_version(
       agent_name="MyAgent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="You are a helpful agent that can use MCP tools to assist users. Use the available MCP tools to answer questions and perform tasks.",
           tools=[mcp_tool],
       ),
   )
   print(f"Agent created (id: {agent.id}, name: {agent.name}, version: {agent.version})")
   ```

   En este código, proporciona instructions para el agent y le entrega las MCP tool definitions.
6. Busque el comentario **Create a conversation thread** y agregue el siguiente código:

   ```python
   # Create a conversation thread
   conversation = openai_client.conversations.create()
   print(f"Created conversation (id: {conversation.id})")
   ```

7. Busque el comentario **Send initial request that will trigger the MCP tool** y agregue el siguiente código:

   ```python
   # Send initial request that will trigger the MCP tool
   response = openai_client.responses.create(
       conversation=conversation.id,
       input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
      extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
   )
   ```

8. Busque el comentario **Process any MCP approval requests that were generated** y agregue el siguiente código:

   ```python
   # Process any MCP approval requests that were generated
   # The agent may issue several tool calls, each needing its own approval,
   # so we loop until there are none left.
   while True:
       # Collect any MCP approval requests from the latest response
       input_list: ResponseInputParam = []
       for item in response.output:
           if item.type == "mcp_approval_request":
               if item.server_label == "api-specs" and item.id:
                   # Automatically approve the MCP request to allow the agent to proceed
                   input_list.append(
                       McpApprovalResponse(
                           type="mcp_approval_response",
                           approve=True,
                           approval_request_id=item.id,
                       )
                   )

       # No more approvals needed -> the agent has produced its final response
       if not input_list:
           break

       # Send the approval response back and retrieve the next response
       response = openai_client.responses.create(
           input=input_list,
           previous_response_id=response.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       )

   print(f"\nAgent response: {response.output_text}")
   ```

   Este código escucha cualquier MCP approval request en la response del agent y las aprueba automáticamente.
9. Busque el comentario **Clean up resources by deleting the agent version** y agregue el siguiente código:

   ```python
   # Clean up resources by deleting the agent version
   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Agent deleted")
   ```

10. Guarde el code file (*CTRL+S*) cuando haya terminado.

## Probar la conexión al remote MCP server

Ahora está listo para ejecutar la application y ver cómo el agent usa la MCP tool para recuperar información del Microsoft Learn Docs remote MCP server.

1. En la integrated terminal, introduzca el siguiente comando para ejecutar la application:

   ```
   az login
   ```

   ```
   python agent.py
   ```

2. Espere a que el agent procese el prompt, usando el MCP server para encontrar una tool adecuada y recuperar la información solicitada. Debería ver un output similar al siguiente:

   ```
   Agent created (id: MyAgent:2, name: MyAgent, version: 2)
   Created conversation (id: conv_086911ecabcbc05700BBHIeNRoPSO5tKPHiXRkgHuStYzy27BS)

   Agent response: Here are Azure CLI commands to create an Azure Container App with a managed identity:

   **1. For a System-assigned Managed Identity**
    ```sh
    az containerapp create \
    --name <CONTAINERAPP_NAME> \
    --resource-group <RESOURCE_GROUP> \
    --environment <CONTAINERAPPS_ENVIRONMENT> \
    --image <CONTAINER_IMAGE> \
    --identity 'system'
    ```

   [continued...]

   Agent deleted
   ```

   Observe que el agent pudo invocar la MCP tool para cumplir automáticamente el request.
3. Puede actualizar el input del request para pedir información distinta. En cada caso, el agent intentará encontrar technical documentation usando la MCP tool.

## Crear un MCP server con custom tools

Además de conectarse a remote MCP servers, también puede crear sus propias custom MCP server tools y conectarlas a su agent. Un Model Context Protocol (MCP) Server es un componente que hospeda callable tools. Estas tools son Python functions que se pueden exponer a AI agents. Cuando las tools se anotan con `@mcp.tool()`, se vuelven discoverable para el client, lo que permite que un AI agent las llame de forma autónoma durante una conversation o task. En esta tarea, agregará tools que permitirán a un agent realizar inventory inquiries y recommendations.

1. Abra el archivo **server.py** en el code editor.

   En este code file definirá las tools que el agent puede usar para simular un backend service de la retail store. Observe el server setup code en la parte superior del archivo. Usa `FastMCP` para levantar rápidamente una MCP server instance llamada “Inventory”. Este server hospedará las tools que defina y las hará accesibles al agent durante el lab.
2. Bajo el comentario **Add references**, agregue el siguiente código:

   ```python
   # Add references
   from fastmcp import FastMCP
   ```

3. Bajo el comentario **Create an MCP server**, agregue el siguiente código para crear una MCP server instance nueva:

   ```python
   # Create an MCP server
   mcp = FastMCP(name="Inventory")
   ```

   Este código inicializa un MCP server nuevo con el label “Inventory”.
4. Busque el comentario **Add an inventory check mcp tool** y agregue el siguiente decorator encima de la function definition, que ahora debería verse así:

   ```python
   # Add an inventory check mcp tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
      # continued...
   ```

   Este dictionary representa un sample inventory. El decorator `@mcp.tool()` registra la function como una tool en el MCP server, lo que permite que el LLM descubra su function.
5. Busque el comentario **Add a weekly sales mcp tool** y agregue el siguiente decorator encima de la function definition, que ahora debería verse así:

   ```python
   # Add a weekly sales mcp tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
      # continued...
   ```

6. Busque el comentario **Run the MCP server** y agregue el siguiente código para iniciar el server:

   ```python
   # Run the MCP server
   mcp.run(show_banner=False)
   ```

   Este código inicia el MCP server y deja sus tools disponibles para discovery y uso por el agent. Establecer `show_banner=False` evita que el startup banner se imprima en stdout, lo que corrompería el MCP stdio protocol.
7. Guarde el archivo (*CTRL+S*).

## Implementar un MCP client para conectarse al custom MCP server

Un MCP client es el componente que se conecta al MCP server para descubrir y llamar tools. Puede pensarlo como el puente entre el agent y las server-hosted functions, lo que habilita el dynamic tool use en respuesta a user prompts.

1. Navegue al archivo **client.py**.
2. Busque el comentario **Add references** y agregue el siguiente código para importar las classes:

   ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   ```

3. En el método **connect_to_server**, busque el comentario **Start the MCP server** y agregue el siguiente código:

   ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
   ```

   En un standard production setup, el server se ejecutaría por separado del client. Pero para este lab, el client es responsable de iniciar el server usando standard input/output transport. Esto crea un lightweight communication channel entre los dos componentes y simplifica el local development setup.
4. Busque el comentario **Create an MCP client session** y agregue el siguiente código:

   ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
   ```

   Esto crea una client session nueva usando los input y output streams del paso anterior. Llamar a `session.initialize` prepara la session para descubrir y llamar tools registradas en el MCP server.
5. Bajo el comentario **List available tools**, agregue el siguiente código para verificar que el client se ha conectado al server:

   ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
   ```

   Ahora su client session está lista para usarse con su Azure AI Agent.

## Conectar las MCP tools a su agent

En esta tarea conectará las MCP server tools a su agent para que pueda llamarlas en respuesta a user prompts.

> **Sugerencia:** Al agregar código, asegúrese de mantener la indentación correcta. Use los comment indentation levels como guía.

1. En el método **chat_loop**, busque el comentario **Build a function for each tool** y agregue el siguiente código:

   ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
       async def tool_func(**kwargs):
           result = await session.call_tool(tool_name, kwargs)
           return result

       tool_func.__name__ = tool_name
       return tool_func

   # Store the functions in a dictionary for easy access when processing function calls
   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   ```

   Este código envuelve de forma dinámica las tools disponibles en el MCP server para que el AI agent pueda llamarlas. Cada tool se convierte en una async function que el agent puede invocar.
2. Busque el comentario **Create FunctionTool definitions for the agent** y agregue el siguiente código:

   ```python
   # Create FunctionTool definitions for the agent
   mcp_function_tools: FunctionTool = []
   for tool in tools:
       function_tool = FunctionTool(
           name=tool.name,
           description=tool.description,
           parameters={
               "type": "object",
               "properties": {},
               "additionalProperties": False,
           },
           strict=True
       )
       mcp_function_tools.append(function_tool)
   ```

3. Busque el comentario **Create the agent** y agregue el siguiente código:

   ```python
   # Create the agent
   agent = project_client.agents.create_version(
       agent_name="inventory-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="""
           You are an inventory assistant. Here are some general guidelines:
           - Recommend restock if item inventory < 10  and weekly sales > 15
           - Recommend clearance if item inventory > 20 and weekly sales < 5
           """,
           tools=mcp_function_tools
       ),
   )
   ```

   Con estas instructions y tools, el agent puede invocar las tools para recuperar inventory y sales data, y después usar esa información para ofrecer responses útiles al usuario.
4. Localice el comentario **Process function calls** y agregue el siguiente código:

   ```python
   # Process function calls
   for item in response.output:
       if item.type == "function_call":
           # Retrieve the matching function tool
           function_name = item.name
           kwargs = json.loads(item.arguments)
           required_function = functions_dict.get(function_name)

           # Invoke the function
           output = await required_function(**kwargs)

           # Append the output text
           input_list.append(
              FunctionCallOutput(
                 type="function_call_output",
                 call_id=item.call_id,
                 output=output.content[0].text,
              )
           )
   ```

   Este código escucha cualquier function call en la response del agent, invoca la corresponding tool function y prepara el output para enviarlo de vuelta al agent.
5. Busque el comentario **Send function call outputs back to the model and retrieve a response** y agregue el siguiente código:

   ```python
   # Send function call outputs back to the model and retrieve a response
   if input_list:
      response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
      )
   print(f"Agent response: {response.output_text}")
   ```

6. Guarde el code file (*CTRL+S*) cuando haya terminado.

## Probar las custom MCP tools con su agent

1. En la integrated terminal, introduzca el siguiente comando para ejecutar la application:

   ```
   python client.py
   ```

2. Cuando se le solicite, introduzca un prompt como:

   ```
   Show me the current inventory levels for all products.
   ```

   > **Sugerencia:** Si la app falla porque se excedió el rate limit, espere unos segundos e inténtelo de nuevo. Si no hay quota suficiente en su subscription, es posible que el modelo no pueda responder.

   Debería ver un output similar al siguiente:

   ```
    MessageRole.AGENT:
    Agent response: Here are the current inventory levels for all items:

   - Moisturizer: 6
   - Shampoo: 8
   - Body Spray: 28
   [continued ...]

   Would you like recommendations for restocking or clearance? If so, I can check the weekly sales to advise accordingly.
   ```

   Observe que el agent pudo llamar las MCP tools para recuperar inventory y sales data, y después usar esa información para ofrecer una response útil al usuario.
3. Puede continuar la conversation si lo desea. El thread es *stateful*, así que retiene el conversation history; es decir, el agent tiene el full context para cada response.

   Pruebe introducir prompts como:

   ```
   Are there any products that should be restocked?
   ```

   ```
   Which products would you recommend for clearance?
   ```

   ```
   What are the best sellers this week?
   ```

4. Introduzca `quit` para salir de la application.

   También puede usar `deactivate` para salir del Python virtual environment en la terminal.

## Limpieza

Cuando haya terminado de explorar la extensión Foundry Toolkit for VS Code, debe limpiar los resources para evitar incurrir en costos innecesarios de Azure.

### Eliminar su model

1. En VS Code, actualice la vista **Azure Resources**.
2. Expanda la subsección **Models**.
3. Haga clic derecho en su deployed model y seleccione **Delete**.

### Eliminar el resource group

1. Abra el [Azure portal](https://portal.azure.com).
2. Navegue al resource group que contiene sus Microsoft Foundry resources.
3. Seleccione **Delete resource group** y confirme la eliminación.
