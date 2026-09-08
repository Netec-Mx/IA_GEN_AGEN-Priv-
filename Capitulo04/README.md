# 1. Práctica 1. Desarrolle una solución multi-agent con Microsoft Agent Framework

## Metadatos

- El agent **Summarizer** condensará el feedback crudo en una oración corta y neutral.
- El agent **Classifier** categorizará el feedback como Positive, Negative o un Feature request.
- Por último, el agent **Recommended Action** recomendará un follow-up step apropiado.

Aprenderá a usar el Microsoft Agent Framework SDK para descomponer un problema, enrutarlo a través de los agents correctos y producir resultados accionables. ¡Comencemos!

Este ejercicio debería tomar aproximadamente *40*** minutos.

> **Nota:** Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Antes de comenzar este ejercicio, asegúrese de tener:

- [Visual Studio Code](https://code.visualstudio.com/) instalado en su equipo local
- Una [Azure subscription](https://azure.microsoft.com/free/) activa
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Git](https://git-scm.com/downloads) instalado en su equipo local

> \* Python 3.14 aún no está soportado: algunas dependencies no tienen build para 3.14. Este lab se validó con Python 3.13.12.

## Crear un proyecto de Foundry con la extensión Foundry Toolkit for VS Code

Como desarrollador, es posible que pase algún tiempo trabajando en el Foundry portal; pero también es probable que pase mucho tiempo en Visual Studio Code. La extensión Foundry Toolkit for VS Code ofrece una forma conveniente de trabajar con Foundry project resources sin salir del entorno de desarrollo.

1. Abra Visual Studio Code.
2. Seleccione **Extensions** en el panel izquierdo (o presione **Ctrl+Shift+X**).
3. Busque en el marketplace de extensiones la extensión `Foundry Toolkit` de Microsoft y seleccione **Install**.

   > **Nota:** La extensión aparece actualmente como **Foundry Toolkit**, pero algunas etiquetas, commands o screenshots antiguos de VS Code aún pueden referirse a **AI Toolkit**. En este lab, trate esos nombres como la misma experiencia de extensión.
4. Después de instalar la extensión, seleccione su icono en la sidebar para abrir la vista Foundry Toolkit.

   Se le debería pedir iniciar sesión en su Azure account si aún no lo ha hecho.
5. Seleccione **Create Project** bajo **Microsoft Foundry Resources**.

   Si ya hay un default project activo, el project name aparecerá bajo **My Resources**. Puede crear un project nuevo haciendo clic derecho en el project activo y seleccionando **Switch Default Project in Azure Extension**.
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
   - **Tokens per minute**: Leave as default
5. Seleccione **Deploy to Microsoft Foundry** en la esquina inferior izquierda.
6. Espere a que se complete el deployment. El modelo desplegado aparecerá bajo la sección **Models** en la vista Resources.
7. Haga clic derecho en el nombre del project deployment y seleccione **Copy Project Endpoint**. Necesitará esta URL para conectar su agent al Foundry project en los siguientes pasos.

## Clonar el starter code repository

Para este ejercicio usará starter code que le ayudará a conectarse a su Foundry project y crear una multi-agent solution que pueda procesar customer feedback. Clonará este código desde un GitHub repository.

1. En VS Code, abra la Command Palette (**Ctrl+Shift+P** o **View > Command Palette**).
2. Escriba **Git: Clone** y selecciónelo de la lista.
3. Introduzca la repository URL:

   ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
   ```

4. Elija una ubicación en su equipo local para clonar el repository.
5. Cuando se le solicite, seleccione **Open** para abrir el repository clonado en VS Code.
6. Una vez que se abra el repository, seleccione **File > Open Folder** y navegue a `mslearn-ai-agents/Labfiles/08-agent-orchestration`; después elija **Select Folder**.
7. En el panel Explorer, expanda la carpeta **Python** para ver los code files de este ejercicio.
8. Haga clic derecho en el archivo **requirements.txt** y seleccione **Open in Integrated Terminal**.
9. En la terminal, introduzca el siguiente comando para instalar los required Python packages en un virtual environment:

   ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

10. Abra el archivo **.env**, reemplace el placeholder **your_project_endpoint** con el endpoint de su project (copiado del project deployment resource en la extensión Foundry Toolkit) y asegúrese de que la variable MODEL_DEPLOYMENT_NAME esté establecida con su model deployment name. Use **Ctrl+S** para guardar el archivo después de hacer estos cambios.

## Crear AI agents

¡Ahora está listo para crear los agents de su multi-agent solution! ¡Comencemos!

1. Abra el archivo **agents.py** en el code editor.
2. En la parte superior del archivo, bajo el comentario **Add references**, agregue el siguiente código para referenciar los namespaces de las libraries que necesitará para implementar su agent:

   ```python
   # Add references
   from agent_framework import Message
   from agent_framework.foundry import FoundryChatClient
   from agent_framework.orchestrations import SequentialBuilder
   from azure.identity import AzureCliCredential
   ```

3. En la función **main**, tómese un momento para revisar las agent instructions. Estas instructions definen el comportamiento de cada agent en la orchestration.
4. Agregue el siguiente código bajo el comentario **Create the chat client**:

   ```python
   # Create the chat client
   credential = AzureCliCredential()
   chat_client = FoundryChatClient(
       credential=credential,
       project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
       model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),
   )
   ```

   Observe que el objeto **AzureCliCredential** permitirá que su código se autentique en su Azure account. El objeto **FoundryChatClient** se conecta a su Foundry project usando el endpoint y el model deployment name de la configuración .env.
5. Agregue el siguiente código bajo el comentario **Create agents**:

   (Asegúrese de mantener el nivel de indentación)

   ```python
   # Create agents
   summarizer_agent = chat_client.as_agent(
       name="summarizer",
       instructions=summarizer_instructions,
   )

   classifier_agent = chat_client.as_agent(
       name="classifier",
       instructions=classifier_instructions,
   )

   action_agent = chat_client.as_agent(
       name="action",
       instructions=action_instructions,
   )
   ```

## Crear una sequential orchestration

1. En la función **main**, busque el comentario **Initialize the current feedback** y agregue el siguiente código:

   (Asegúrese de mantener el nivel de indentación)

   ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
   ```

2. Bajo el comentario **Build a sequential orchestration**, agregue el siguiente código para definir una sequential orchestration con los agents que definió:

   ```python
   # Build sequential orchestration
   workflow = SequentialBuilder(
       participants=[summarizer_agent, classifier_agent, action_agent],
       output_from="all",
   ).build()
   ```

   Los agents procesarán el feedback en el orden en que se agregan a la orchestration. El parámetro `output_from="all"` asegura que se recopilen los outputs de todos los agents.
3. Agregue el siguiente código bajo el comentario **Run and collect outputs**:

   ```python
   # Run and collect outputs
   result = await workflow.run(f"Customer feedback: {feedback}")
   outputs = result.get_outputs()
   ```

   Este código ejecuta la orchestration y recopila el output de cada uno de los participating agents.
4. Agregue el siguiente código bajo el comentario **Display outputs**:

   ```python
   # Display outputs
   i = 1
   for response in outputs:
       for msg in cast(list[Message], response.messages):
           name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
           i += 1
   ```

   Este código formatea y muestra los messages de los workflow outputs que recopiló de la orchestration.
5. Use el comando **CTRL+S** para guardar los cambios en el code file.

## Probar la application

Ahora está listo para ejecutar su código y ver colaborar a sus AI agents.

1. En la integrated terminal, introduzca los siguientes comandos para ejecutar la application:

   ```
   az login
   ```

   ```
   python agents.py
   ```

2. Debería ver un output similar al siguiente:

   ```output
   User requests a dark mode option for more comfortable nighttime use.
   Feature request
   Log as enhancement request to add dark mode for improved user comfort during nighttime use.
   ------------------------------------------------------------
   01 [summarizer]
   User requests a dark mode option for more comfortable nighttime use.
   ------------------------------------------------------------
   02 [classifier]
   Feature request
   ------------------------------------------------------------
   03 [action]
   Log as enhancement request to add dark mode for improved user comfort during nighttime use.
   ```

3. De forma opcional, puede intentar ejecutar el código usando distintos feedback inputs, por ejemplo:

   ```output
   I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
   ```

4. Cuando haya terminado, introduzca `deactivate` en la terminal para salir del Python virtual environment.

## Limpieza

Si ha terminado de explorar Azure AI Agent Service, debe eliminar los resources que ha creado en este ejercicio para evitar incurrir en costos innecesarios de Azure.

### Eliminar su model

1. En VS Code, actualice la vista **Azure Resources**.
2. Expanda la subsección **Models**.
3. Haga clic derecho en su deployed model y seleccione **Delete**.

### Eliminar el resource group

1. Abra el [Azure portal](https://portal.azure.com).
2. Navegue al resource group que contiene sus Microsoft Foundry resources.
3. Seleccione **Delete resource group** y confirme la eliminación.
