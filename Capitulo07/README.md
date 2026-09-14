# 1. Práctica 1. Construya AI agents con el portal y VS Code

En este ejercicio construirá una solución completa de AI agent usando tanto el Microsoft Foundry portal como la extensión Foundry Toolkit VS Code. Comenzará creando un basic agent en el portal con grounding data y built-in tools, y después interactuará con él de forma programática usando VS Code para aprovechar capabilities avanzadas como code interpreter para data analysis.

Este ejercicio toma aproximadamente **45** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- Una [Azure subscription](https://azure.microsoft.com/free/) con permisos y quota suficientes para provisionar Azure AI resources
- [Visual Studio Code](https://code.visualstudio.com/) instalado en su equipo local
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Git](https://git-scm.com/downloads) instalado en su equipo local
- Familiaridad básica con Azure AI services y programación en Python

> \* Python 3.14 aún no está soportado: algunas dependencies no tienen build para 3.14. Este lab se validó con Python 3.13.12.

## Crear un Microsoft Foundry Project

Microsoft Foundry usa projects para organizar models, resources, data y otros assets usados para desarrollar una solución de IA.

1. En un web browser, abra el [Foundry portal](https://ai.azure.com) en `https://ai.azure.com` e inicie sesión con sus credenciales de Azure. Cierre cualquier panel de tips o quick start que se abra la primera vez que inicie sesión y, si es necesario, use el logo **Foundry** en la parte superior izquierda para navegar a la home page.

   > **Importante:** Para este lab, está usando la experiencia **New** Foundry.
2. En el top banner, seleccione **Start building** para probar la nueva Microsoft Foundry Experience.
3. Cuando se le solicite, cree un project **new** e introduzca un nombre válido para su project (por ejemplo, `it-support-agent-project`).
4. Expanda **Advanced options** y especifique la siguiente settings:
   - **Microsoft Foundry resource**: *A valid name for your Foundry resource*
   - **Region**: *Select one available near you*\*
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Select your resource group, or create a new one*

   > \* Algunos Azure AI resources están limitados por regional model quotas. Si más adelante en el ejercicio se excede un quota limit, es posible que deba crear otro resource en una región diferente.
5. Seleccione **Create** y espere a que se cree su project.
6. Cuando se cree su project, puede aparecer un welcome dialog. Seleccione **Next** para leer el welcome message y después seleccione **Create agent**.

   También puede seleccionar **Start building** en la home page y seleccionar **Create agents** en el drop-down menu.
7. Establezca el **Agent name** en `it-support-agent` y cree el agent.

El playground se abrirá para su agent recién creado. Verá que ya hay un deployed model disponible seleccionado para usted.

## Configurar su agent con instructions y grounding data

Ahora que tiene un agent creado, vamos a configurarlo con instructions y a agregar grounding data.

1. En el agent playground, establezca las **Instructions** en:

   ```prompt
   You are an IT Support Agent for Contoso Corporation.
   You help employees with technical issues and IT policy questions.

   Guidelines:
   - Always be professional and helpful
   - Use the IT policy documentation to answer questions accurately
   - If you don't know the answer, admit it and suggest contacting IT support directly
   - When creating tickets, collect all necessary information before proceeding
   ```

2. Descargue el IT policy document del lab repository. Abra una pestaña nueva del browser y navegue a:

   ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt
   ```

   Guarde el archivo en su equipo local.

   > **Nota:** Este document contiene sample IT policies para password resets, software installation requests y hardware troubleshooting.
3. Vuelva al agent playground. En la sección **Tools**, seleccione **Add** y después agregue tanto **File search** como **</> Code interpreter**.
4. A la derecha de **Add**, seleccione **Upload files**. Bajo **Attach files**, busque y cargue el archivo `IT_Policy.txt` que acaba de descargar, y después seleccione **Attach**.
5. Espere a que el archivo se indexe. Verá una confirmation cuando esté ready.
6. Ahora agreguemos performance data para que el code interpreter lo analice. Descargue el system performance data file desde:

   ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv
   ```

   Guarde este archivo en su equipo local.
7. A la derecha de **</> Code interpreter**, seleccione **+ Files** y después cargue el archivo `system_performance.csv` que acaba de descargar.

   > **Nota:** Este CSV file contiene simulated system metrics (CPU, memory, disk usage) a lo largo del tiempo que el agent puede analizar.
8. Guarde el agent.

## Probar su agent

Probemos el agent para ver cómo responde usando el grounding data.

1. En la chat interface del lado derecho del playground, introduzca el siguiente prompt:

   ```
   What's the policy for password resets?
   ```

2. Revise la response. El agent debería referenciar el IT policy document y proporcionar información precisa sobre password reset procedures.
3. Pruebe otro prompt:

   ```
   How do I request new software?
   ```

4. De nuevo, revise la response y observe cómo el agent usa el grounding data.
5. Ahora pruebe el code interpreter con un data analysis request:

   ```
   Can you analyze the system performance data and tell me if there are any concerning trends?
   ```

6. El agent debería usar el code interpreter para analizar el CSV file y ofrecer insights sobre system performance.
7. Pruebe pedir una visualization:

   ```
   Create a chart showing CPU usage over time from the performance data
   ```

8. El agent usará code interpreter para generar visualizations y analysis.

¡Muy bien! Ha creado un agent con grounding data, file search y code interpreter capabilities. En la siguiente sección, interactuará con este agent de forma programática usando VS Code.

## Interactuar con su agent usando VS Code

Como desarrollador, es posible que pase algún tiempo trabajando en el Foundry portal; pero también es probable que pase mucho tiempo en Visual Studio Code. La extensión Foundry Toolkit for VS Code ofrece una forma conveniente de trabajar con Foundry project resources sin salir del entorno de desarrollo.

### Instalar y configurar la VS Code extension

Si ya tiene instalada la extensión Foundry Toolkit, puede omitir esta sección.

1. Abra Visual Studio Code.
2. Seleccione **Extensions** en el panel izquierdo (o presione **Ctrl+Shift+X**).
3. Busque en el marketplace de extensiones la extensión `Foundry Toolkit for VS Code` de Microsoft y seleccione **Install**.

   Instalar Foundry Toolkit Extension agregará la extensión Foundry Toolkit a VS Code.

   > **Nota:** La extensión aparece actualmente como **Foundry Toolkit**, pero algunas etiquetas, commands o screenshots antiguos de VS Code aún pueden referirse a **AI Toolkit**. En este lab, trate esos nombres como la misma experiencia de extensión.
4. Después de instalar la extensión, seleccione el icono Foundry Toolkit en la sidebar.

   Se le debería pedir iniciar sesión en su Azure account si aún no lo ha hecho.

### Probar su agent en VS Code

Antes de escribir código, puede interactuar con su agent directamente en la interface de la extensión.

1. Bajo **Microsoft Foundry Resources**, elija **Set Default Project**.

   Si ya hay un default project activo, el project name aparecerá en la resources list. Puede seleccionar un project distinto seleccionando el mismo icono **Select project**.
2. Expanda la sección del project. Bajo **Prompt Agents**, debería ver el `it-support-agent` que creó en el portal. Seleccione el agent name para abrir la interface **Agent Builder**.

   El agent playground aparecerá en la interface Agent Builder, lo que le permitirá interactuar con el agent y configurar su settings sin salir de VS Code.
3. En el playground chat pane, escriba una question como:

   ```
   What is the policy for reporting a lost or stolen device?
   ```

4. Revise la response del agent. Debería usar el grounding data que cargó antes para ofrecer IT policy information relevante.

   > **Sugerencia:** Puede usar este built-in playground para probar rápidamente las instructions y el knowledge de su agent sin escribir código.

## Crear una client application para interactuar con su agent

Ahora vamos a crear una client application que interactúe con su agent de forma programática.

1. En VS Code, abra la Command Palette (**Ctrl+Shift+P** o **View > Command Palette**).
2. Escriba **Git: Clone** y selecciónelo de la lista.
3. Introduzca la repository URL:

   ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
   ```

4. Elija una ubicación en su equipo local para clonar el repository.
5. Cuando se le solicite, seleccione **Open** para abrir el repository clonado en VS Code.
6. Una vez que se abra el repository, seleccione **File > Open Folder** y navegue a `mslearn-ai-agents/Labfiles/01-build-agent-portal-and-vscode/Python`; después elija **Select Folder**.
7. En el panel Explorer, abra el archivo `agent_with_functions.py`. Si el archivo está vacío, reemplace su contenido con el siguiente código.
8. Use el siguiente código:

   ```python
   import base64
   import os
   from pathlib import Path

   from azure.ai.projects import AIProjectClient
   from azure.identity import DefaultAzureCredential
   from dotenv import load_dotenv

   OUTPUT_DIR = Path("agent_outputs")

   def get_output_path(filename):
       """Create a unique path for generated files."""
       OUTPUT_DIR.mkdir(exist_ok=True)
       file_name = Path(filename).name
       stem = Path(file_name).stem or "output"
       suffix = Path(file_name).suffix
       output_path = OUTPUT_DIR / file_name

       counter = 1
       while output_path.exists():
           output_path = OUTPUT_DIR / f"{stem}_{counter}{suffix}"
           counter += 1

       return output_path

   def save_bytes(file_bytes, filename):
       """Save binary content to a local file."""
       output_path = get_output_path(filename)
       with open(output_path, "wb") as file_handle:
           file_handle.write(file_bytes)
       return output_path

   def save_image(image_data, filename):
       """Save base64 image data to a file."""
       return save_bytes(base64.b64decode(image_data), filename)

   def download_container_file(openai_client, annotation, downloaded_files):
       """Download a cited container file once and return its local path."""
       cache_key = (annotation.container_id, annotation.file_id)
       if cache_key in downloaded_files:
           return downloaded_files[cache_key]

       file_content = openai_client.containers.files.content.retrieve(
           file_id=annotation.file_id,
           container_id=annotation.container_id,
       )
       output_path = save_bytes(
           file_content.read(),
           annotation.filename or f"{annotation.file_id}.bin",
       )
       downloaded_files[cache_key] = output_path
       return output_path

   def format_output_text(content_item, openai_client, downloaded_files):
       """Replace sandbox file citations with local file paths."""
       text = content_item.text or ""
       replacements = []
       referenced_files = set()

       for annotation in content_item.annotations or []:
           if getattr(annotation, "type", "") != "container_file_citation":
               continue

           output_path = download_container_file(openai_client, annotation, downloaded_files)
           replacement_text = f"{annotation.filename} (saved to {output_path})"
           referenced_files.add(output_path)

           start_index = getattr(annotation, "start_index", None)
           end_index = getattr(annotation, "end_index", None)
           if start_index is not None and end_index is not None:
               replacements.append((start_index, end_index, replacement_text))
               continue

           annotated_text = getattr(annotation, "text", "")
           if annotated_text:
               text = text.replace(annotated_text, replacement_text)

       for start_index, end_index, replacement_text in sorted(replacements, reverse=True):
           text = f"{text[:start_index]}{replacement_text}{text[end_index:]}"

       return text, referenced_files

   def main():
       # Initialize the project client
       load_dotenv()
       project_endpoint = os.environ.get("PROJECT_ENDPOINT")
       agent_name = os.environ.get("AGENT_NAME", "it-support-agent")

       if not project_endpoint:
           print("Error: PROJECT_ENDPOINT environment variable not set")
           print("Please set it in your .env file or environment")
           return

       print("Connecting to Microsoft Foundry project...")
       credential = DefaultAzureCredential()
       project_client = AIProjectClient(
           credential=credential,
           endpoint=project_endpoint
       )

       # Get the OpenAI client for Responses API
       openai_client = project_client.get_openai_client()

       # Get the agent created in the portal
       print(f"Loading agent: {agent_name}")
       agent = project_client.agents.get(agent_name=agent_name)
       print(f"Connected to agent: {agent.name} (id: {agent.id})")

       # Create a conversation
       conversation = openai_client.conversations.create(items=[])
       print(f"Conversation created (id: {conversation.id})")

       # Chat loop
       print("\n" + "="*60)
       print("IT Support Agent Ready!")
       print("Ask questions, request data analysis, or get help.")
       print("Type 'exit' to quit.")
       print("="*60 + "\n")

       while True:
           user_input = input("You: ").strip()

           if user_input.lower() in ['exit', 'quit', 'bye']:
               print("Goodbye!")
               break

           if not user_input:
               continue

           # Add user message to conversation
           openai_client.conversations.items.create(
               conversation_id=conversation.id,
               items=[{"type": "message", "role": "user", "content": user_input}]
           )

           # Get response from agent
           print("\n[Agent is thinking...]")
           response = openai_client.responses.create(
               conversation=conversation.id,
               extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
               input=""
           )

           # Display response and save any generated files locally
           handled_output = False
           downloaded_files = {}
           referenced_files = set()
           image_count = 0

           if hasattr(response, "output") and response.output:
               for item in response.output:
                   item_type = getattr(item, "type", "")

                   if item_type == "message" and getattr(item, "content", None):
                       for content_item in item.content:
                           if getattr(content_item, "type", "") != "output_text":
                               continue

                           formatted_text, message_files = format_output_text(
                               content_item,
                               openai_client,
                               downloaded_files,
                           )
                           referenced_files.update(message_files)

                           if formatted_text:
                               print(f"\nAgent: {formatted_text}\n")
                               handled_output = True

                   elif hasattr(item, "text") and item.text:
                       print(f"\nAgent: {item.text}\n")
                       handled_output = True

                   elif item_type == "image":
                       image_count += 1
                       filename = f"chart_{image_count}.png"

                       if hasattr(item, "image") and hasattr(item.image, "data"):
                           file_path = save_image(item.image.data, filename)
                           print(f"\n[Agent generated a chart - saved to: {file_path}]")
                       else:
                           print("\n[Agent generated an image]")
                       handled_output = True

               for file_path in downloaded_files.values():
                   if file_path not in referenced_files:
                       print(f"\n[Agent generated a file - saved to: {file_path}]")
                       handled_output = True

           if not handled_output and hasattr(response, "output_text") and response.output_text:
               print(f"\nAgent: {response.output_text}\n")

   if __name__ == "__main__":
       main()
   ```

9. Guarde el archivo `agent_with_functions.py` (**Ctrl+S** o **File > Save**).

### Configurar el environment y ejecutar la application

1. En el panel Explorer, verá los archivos `.env.example` y `requirements.txt` ya presentes en la carpeta.
2. Duplique el archivo `.env.example` y cámbiele el nombre a `.env`.
3. En el archivo `.env`, reemplace `your_project_endpoint_here` con su project endpoint real:

   ```
   PROJECT_ENDPOINT=<your_project_endpoint>
   AGENT_NAME=it-support-agent
   ```

   **Para obtener su project endpoint:** En VS Code, abra la extensión **Foundry Toolkit**, haga clic derecho en su active project y seleccione **Copy Endpoint**. Si **Copy Endpoint** no está disponible en la versión instalada de Foundry Toolkit, abra el Microsoft Foundry portal, vaya a su project y copie el project endpoint desde la project overview page.
4. Guarde el archivo `.env` (**Ctrl+S** o **File > Save**).
5. Abra una terminal en VS Code (**Terminal > New Terminal**) y navegue al working directory.
6. Instale los required packages e inicie sesión:

   ```bash
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

   ```bash
   az login
   ```

7. Ejecute la application:

   ```bash
   python agent_with_functions.py
   ```

## Probar la client application

Cuando el agent inicie, pruebe estos prompts para validar distintas capabilities:

1. Pruebe policy search con file search:

   ```
   What's the policy for password resets?
   ```

2. Solicite data analysis con code interpreter:

   ```
   Analyze the system performance data and identify any periods where CPU usage exceeded 80%
   ```

3. Solicite una visualization:

   ```
   Create a line chart showing memory usage trends over time
   ```

   La application guarda generated charts y cited files en la carpeta `agent_outputs` e imprime el local file path en la terminal.
4. Pida statistical analysis:

   ```
   What are the average, minimum, and maximum values for disk usage in the performance data?
   ```

5. Combined analysis:

   ```
   Find any correlation between high CPU usage and memory usage in the performance data
   ```

Observe cómo el agent usa tanto file search (para policy questions) como code interpreter (para data analysis) para cumplir sus requests. El code interpreter analizará el CSV data, realizará calculations y puede incluso generar visualizations. Escriba `exit` cuando haya terminado de probar.

## Limpieza

Para evitar Azure charges innecesarios, elimine los resources que creó:

1. En el Foundry portal, navegue a su project.
2. Seleccione **Settings** > **Delete project**.
3. Como alternativa, elimine el resource group completo desde el Azure portal.
