<img src="images/neteclogo (2).png" alt="logo" width="300"/>

# Inteligencia Artificial Generativa para creación de agentes

Curso diseñado para desarrolladores que desean diseñar, construir y desplegar soluciones de inteligencia artificial generativa y agentes con Microsoft Foundry. A lo largo del curso se abordará la preparación de proyectos en Azure, la comparación y el consumo de modelos, la integración con Foundry IQ, orquestación multi-agent, guardrails, Model Context Protocol (MCP), fine-tuning y la construcción de agents desde el portal y Visual Studio Code.

## Estructura

- `CapituloXX/README.md`: guía de laboratorio por capítulo.

## Lista de laboratorios

### Capítulo 1

- [1. Práctica 1. Prepárese para un proyecto de desarrollo de IA](Capitulo01/README.md#1-práctica-1-prepárese-para-un-proyecto-de-desarrollo-de-ia)
  - Descripción: Crear un proyecto en Microsoft Foundry, desplegar `gpt-5.2`, localizar endpoints y conectar el proyecto desde Visual Studio Code con Foundry Toolkit.
  - Duración estimada: 30 min
- [2. Práctica 2. Explore y Compare Modelos](Capitulo01/README.md#2-práctica-2-explore-y-compare-modelos)
  - Descripción: Explorar el model catalog, comparar `gpt-5.2` y `gpt-5-mini` en el leaderboard y el playground, y ejecutar la evaluation `travel-assistant-eval`.
  - Duración estimada: 45 min

### Capítulo 2

- [1. Práctica 1. Cree una aplicación de chat de IA generativa](Capitulo02/README.md#1-práctica-1-cree-una-aplicación-de-chat-de-ia-generativa)
  - Descripción: Crear una chat app con el OpenAI SDK y las APIs ChatCompletions y Responses, conectada a un modelo desplegado en Microsoft Foundry, con conversation tracking y streaming.
  - Duración estimada: 45 min
- [2. Práctica 2. Cree una aplicación de IA generativa que usa herramientas](Capitulo02/README.md#2-práctica-2-cree-una-aplicación-de-ia-generativa-que-usa-herramientas)
  - Descripción: Integrar knowledge en una chat application usando las tools `web_search` y `file_search` sobre el vector store `travel-brochures`.
  - Duración estimada: 30 min

### Capítulo 3

- [1. Práctica 1. Integre un agente de IA con Foundry IQ](Capitulo03/README.md#1-práctica-1-integre-un-agente-de-ia-con-foundry-iq)
  - Descripción: Crear el agent `product-expert-agent`, configurar una knowledge base con Foundry IQ y conectarse desde `agent_client.py` con MCP approval handling.
  - Duración estimada: 45 min

### Capítulo 4

- [1. Práctica 1. Desarrolle una solución multi-agent con Microsoft Agent Framework](Capitulo04/README.md#1-práctica-1-desarrolle-una-solución-multi-agent-con-microsoft-agent-framework)
  - Descripción: Orquestar los agents `summarizer`, `classifier` y `action` con SequentialBuilder para procesar customer feedback.
  - Duración estimada: 30 min
- [2. Práctica 2. Aplique guardrails para evitar la generación de contenido dañino](Capitulo04/README.md#2-práctica-2-aplique-guardrails-para-evitar-la-generación-de-contenido-dañino)
  - Descripción: Probar el default guardrail y crear un custom guardrail con Highest blocking para Hate, Violence, Sexual y Self-harm sobre `gpt-5.2`.
  - Duración estimada: 25 min

### Capítulo 5

- [1. Práctica 1. Extienda agents con tools de Model Context Protocol (MCP)](Capitulo05/README.md#1-práctica-1-extienda-agents-con-tools-de-model-context-protocol-mcp)
  - Descripción: Conectar `MyAgent` a un remote MCP server y crear el MCP server `Inventory` con el agent `inventory-agent` para consultas de inventario.
  - Duración estimada: 60 min

### Capítulo 6

- [1. Práctica 1. Fine-tune un language model](Capitulo06/README.md#1-práctica-1-fine-tune-un-language-model)
  - Descripción: Hacer fine-tune de `gpt-5` con el dataset `travel-finetune-hotel.jsonl` y el sufijo `ft-travel`, y compararlo con el base model.
  - Duración estimada: 90 min

### Capítulo 7

- [1. Práctica 1. Construya AI agents con el portal y VS Code](Capitulo07/README.md#1-práctica-1-construya-ai-agents-con-el-portal-y-vs-code)
  - Descripción: Crear el agent `it-support-agent` con File search y Code interpreter, y consumirlo desde `agent_with_functions.py`.
  - Duración estimada: 45 min

---

## 📬 **Contacto y más información**

Si tienes alguna pregunta o necesitas más detalles, no dudes en [contactarnos](mailto:soporte@netec.com). También puedes encontrar más recursos en nuestra [página](https://netec.com).

---

¡Gracias por visitar nuestra plataforma! No olvides revisar todos los laboratorios y comenzar tu viaje de aprendizaje hoy mismo.

## Flujo de colaboración

- Trabajar en `changes_course`.
- Crear Pull Request hacia `main`.
- Merge por `Squash and merge`.
