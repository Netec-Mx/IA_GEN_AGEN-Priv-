# 1. Práctica 1. Fine-tune un language model

Cuando desea que un language model se comporte de cierta forma, puede usar prompt engineering para definir el desired behavior. Cuando desea mejorar la consistencia de ese desired behavior, puede optar por hacer fine-tune de un modelo y compararlo con su enfoque de prompt engineering para evaluar qué método se ajusta mejor a sus necesidades.

En este ejercicio hará fine-tune de un language model con Microsoft Foundry para un escenario de custom chat application. Comparará el fine-tuned model con un base model para evaluar si el fine-tuned model se ajusta mejor a sus necesidades.

Imagine que trabaja en una travel agency y está desarrollando una chat application para ayudar a las personas a planear sus vacations. El objetivo es crear un chat simple e inspiring que sugiera destinations y activities con un conversational tone consistente y friendly.

Este ejercicio tomará aproximadamente **90** minutos\*.

> \* **Nota:** Este tiempo es una estimación basada en la experiencia promedio. Fine-tuning depende de cloud infrastructure resources, que pueden tardar un tiempo variable en provisionarse según la capacidad del data center y la demanda concurrente. Algunas actividades de este ejercicio pueden tardar <u>mucho</u> tiempo y requieren paciencia. Si el proceso se demora, considere revisar la [documentación de fine-tuning de Microsoft Foundry](https://learn.microsoft.com/azure/ai-foundry/openai/how-to/fine-tuning?view=foundry) o tomar un descanso. Es posible que algunos procesos hagan time-out o parezcan ejecutarse indefinidamente. Algunas de las tecnologías usadas en este ejercicio están en preview o en desarrollo activo. Es posible que observe comportamientos inesperados, advertencias o errores.

## Prerrequisitos

Para completar este ejercicio, necesita:

- Una [Azure subscription](https://azure.microsoft.com/free/) con permisos para crear AI resources.

## Crear un proyecto de Microsoft Foundry

Microsoft Foundry usa projects para organizar models, resources, data y otros assets usados para desarrollar una solución de IA.

1. En un web browser, abra el [Microsoft Foundry portal](https://ai.azure.com) en `https://ai.azure.com` para empezar a construir; inicie sesión con sus credenciales de Azure. Cierre cualquier panel de tips o quick start que se abra la primera vez que inicie sesión.
2. Si aún no está habilitada, en la tool bar de la parte superior de la página, active la opción **New Foundry**. Después, si se le solicita, cree un project nuevo con un nombre único; expandiendo el área **Advanced options** para especificar la siguiente configuración de su project:
   - **Foundry resource**: *Use the default name for your resource (usually {project_name}-resource)*
   - **Subscription**: *Your Azure subscription*
   - **Resource group**: *Create or select a resource group*
   - **Region**: *Select one of the following regions*:\*
     - North Central US
     - Sweden Central

   > \* Al momento de escribir este lab, estas regions soportan fine-tuning para modelos gpt-5. Consulte la [models page](https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure?&pivots=azure-openai#fine-tuning-models) para la disponibilidad regional más reciente.
3. Espere a que se cree su project. Después, vea su home page.

## Desplegar un modelo

A continuación, vamos a hacer deploy de un modelo que usará para obtener un performance baseline.

1. Ahora está listo para explorar models. En la página **Discover**, seleccione la pestaña **Models** para ver el Microsoft Foundry model catalog.
2. En el model catalog, busque `gpt-5`.
3. Revise la model card y, a continuación, haga deploy usando los default settings.
4. Cuando el modelo se haya desplegado, se abrirá en el model playground.

## Fine-tune a model

Como fine-tune de un modelo tarda en completarse, iniciará el fine-tuning job ahora y volverá a él después de explorar el base gpt-5 model que ya desplegó.

1. Descargue el [training dataset](https://microsoftlearning.github.io/mslearn-ai-studio/data/travel-finetune-hotel.jsonl) en `https://microsoftlearning.github.io/mslearn-ai-studio/data/travel-finetune-hotel.jsonl` y guárdelo localmente como un archivo JSONL.

   > **Nota:** Su dispositivo podría guardar el archivo de forma predeterminada como .txt. Seleccione all files y quite el sufijo .txt para asegurarse de guardar el archivo como JSONL.
2. En el Foundry portal, mientras ve el model playground, en el left navigation pane, seleccione **Fine-tune**.
3. Seleccione el botón **Fine-tune** en la parte superior derecha y, a continuación, configure el fine-tuning job con la siguiente settings:
   - **Base model**: Seleccione **gpt-5**
   - **Customization method**: Supervised
   - **Training type**: Standard
   - **Training data**: Seleccione **Upload new dataset** y cargue el archivo .jsonl que descargó antes.
   - **Suffix**: `ft-travel`
   - **Automatically deploy model after job completion**: Selected
   - **Deployment type**: Developer
   - *Leave the remaining hyperparameters at their defaults*
4. Seleccione **Submit** para iniciar el fine-tuning job. Puede tardar en completarse. Puede continuar con la siguiente sección del ejercicio mientras espera.

> **Nota:** Fine-tuning y deployment pueden tardar un tiempo significativo (60 minutos o más), así que es posible que deba revisar periódicamente. Puede ver más detalles del progreso seleccionando el fine-tuning job y viendo su pestaña **Monitor**.

## Chatear con un base model

Mientras espera a que se complete el fine-tuning job, chateemos con un foundation model *gpt-5* para evaluar cómo se desempeña.

1. En el panel izquierdo, seleccione **Deployments** y después seleccione el base model **gpt-5** que desplegó antes.
2. En el chat pane, introduzca el prompt `What can you do?` y vea la response.

   Las answers pueden ser bastante genéricas. Recuerde que queremos crear una chat application que inspire a las personas a viajar.
3. Cambie las **Instructions** del modelo al siguiente prompt:

   ```
   You are an AI assistant that helps people plan their travel.
   ```

4. En la ventana de chat, introduzca de nuevo la query `What can you do?` y vea la response.

   Como response, el assistant puede decirle que puede ayudarle a book flights, hotels y rental cars para su trip. Usted desea evitar este behavior.
5. En el campo **Instructions**, introduzca un prompt nuevo:

   ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   You should not provide any hotel, flight, rental car or restaurant recommendations.
   Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
   ```

6. Continúe probando el modelo para revisar su behavior. Por ejemplo, haga las siguientes questions y anote las answers del modelo, prestando especial atención al tone y al writing style con el que responde:

   `Where in Rome should I stay?`

   `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

   `What are some local delicacies I should try?`

   `When is the best time of year to visit in terms of the weather?`

   `What's the best way to get around the city?`

## Revisar el training file

El base model parece funcionar suficientemente bien, pero es posible que busque un conversational style particular para su generative AI app. El training data usado para fine-tuning le ofrece la oportunidad de crear ejemplos explícitos de los tipos de response que desea.

1. Abra el archivo JSONL que descargó antes (puede abrirlo en cualquier text editor).
2. Examine la lista de JSON documents del training data file. El primero debería ser similar a este (formateado para legibilidad):

   ```json
   {"messages": [
       {"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."},
       {"role": "user", "content": "What's a must-see in Paris?"},
       {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}
       ]}
   ```

   Cada example interaction de la lista incluye el mismo system message que probó con el base model, un user prompt relacionado con una travel query y una response. El style de las responses del training data ayudará al fine-tuned model a aprender cómo debe responder.

## Probar el fine-tuned model

Cuando su fine-tuned model esté listo, puede probarlo como probó su deployed base model.

1. En el panel de la izquierda, seleccione **Fine-tune** y revise el status del fine-tuning job que inició antes.
2. Seleccione el job para ver sus details. Puede usar la pestaña **Logs** para revisar las fine-tuning tasks que se han realizado hasta ahora.
3. Cuando fine-tuning esté complete y el modelo se haya desplegado automáticamente, vea la página **Deployments** para verificar que aparece en la lista.

   > **Sugerencia:** Si automatic deployment falla, seleccione el fine-tuning job completed y haga deploy del modelo desde ahí.
4. Seleccione el fine-tuned model para abrirlo en el model playground.
5. Actualice las **Instructions** para que sean las mismas que probó con el base model:

   ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   You should not provide any hotel, flight, rental car or restaurant recommendations.
   Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
   ```

6. Pruebe su fine-tuned model para evaluar si su behavior es más consistente que el del base model. Por ejemplo, haga de nuevo las siguientes questions y explore las answers del modelo:

   `Where in Rome should I stay?`

   `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

   `What are some local delicacies I should try?`

   `When is the best time of year to visit in terms of the weather?`

   `What's the best way to get around the city?`

## Limpieza

Si ha terminado de explorar Microsoft Foundry, debe eliminar los resources que ha creado en este ejercicio para evitar incurrir en costos innecesarios de Azure.

1. Abra el [Azure portal](https://portal.azure.com) y vea el contenido del resource group donde implementó los resources usados en este ejercicio.
2. En la toolbar, seleccione **Delete resource group**.
3. Introduzca el resource group name y confirme que desea eliminarlo.
