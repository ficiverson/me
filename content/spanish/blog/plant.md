---
title: "Analizador de plantas con IA! 🌱"
meta_title: "Analizador de plantas con IA! 🌱"
description: "Analizador de plantas con IA! 🌱"
date: 2024-11-09T05:00:00Z
image: "/images/blog/plant/plant.webp"
categories: ["AI"]
author: "Fernando Souto"
language: "es"
tags: ["plant analyzer","OpenAI", "Gardentech","Plant care"]
draft: false 
---

He desarrollado un analizador de plantas impulsado por IA que combina herramientas avanzadas del OpenAI Vision API y Tavily, con capacidades de búsqueda para explorar información en [gardenia.net](http://gardenia.net/).

🌿 ¡Con esta herramienta, identificar plantas y acceder a información detallada sobre su cuidado nunca ha sido tan fácil! En este video, explico el proceso de creación, detallando los pasos técnicos y las decisiones detrás de la construcción de una IA que ayuda tanto a entusiastas de las plantas como a expertos a comprender y cuidar mejor sus plantas.

🎥 Mira el video para conocer más sobre cómo funciona este analizador de IA y el potencial que tiene para el mundo de la horticultura. ¡Hagamos que el cuidado de las plantas sea más inteligente, juntos!

{{< youtube ifhVcUsnqx0 >}}

## Revisemos los detalles

Este código nos permite cargar una imagen directamente desde una carpeta específica, lo que facilita el acceso y la manipulación de archivos de imagen almacenados localmente.
```python
    def load_image(inputs: dict) -> dict:
        image_path = inputs["image_path"]
        
        def encode_image(image_path):
            with open(image_path, "rb") as image_file:
                return base64.b64encode(image_file.read()).decode("utf-8")
        
        image_base64 = encode_image(image_path)
        return {"image": image_base64}
    
    
    load_image_chain = TransformChain(
        input_variables=["image_path"], output_variables=["image"], transform=load_image
    )
```

Necesitamos definir un modelo que servirá como salida del Modelo de Aprendizaje de Lenguaje (LLM). En este caso, el modelo generará información sobre el nombre de la planta y proporcionará una evaluación de su estado de salud.
```python
    class PlantInformation(BaseModel):
        name: str = Field(
            example="Sansevieria",
            description="The name of the plant or Not a plant",
        )
        healthy: bool = Field(
            example="True",
            description="If the plant looks good or not",
        )
```

Desarrollamos la herramienta que utilizará el Modelo de Aprendizaje de Lenguaje (LLM) para analizar la imagen, permitiéndole extraer información relevante de los datos visuales.
```python
    def image_model(inputs: dict) -> str | list[str | dict[Any, Any]]:
        model = AzureChatOpenAI(
            azure_deployment=os.environ['AZURE_OPENAI_DEPLOYMENT'],
            model_version="2024-05-13",
            api_version="2024-02-01",
            temperature=0
        )
        msg = model.invoke(
                [
                    HumanMessage(
                        content=[
                            {"type": "text", "text": inputs["prompt"]},
                            {"type": "text", "text": parser.get_format_instructions()},
                            {
                                "type": "image_url",
                                "image_url": {
                                    "url": f"data:image/jpeg;base64,{inputs['image']}"
                                }
                            }
                        ]
                    )
                ]
            )
        return msg
```

Este código invocará la herramienta utilizando el modelo especificado y el mensaje proporcionado para analizar la imagen. La herramienta procesará la información visual y devolverá los resultados en un formato estructurado de respuesta JSON.
```python
    def get_image_information(image_path: str) -> dict:
        vision_prompt = """Analyze the image to get the name of the plant. Also analyze the health status of the plant. Answer Not a plant when is not a plant
        # Example 1: 
        name: Sansevieria
        healthy: True
        # Example 2: 
        name: Spathiphyllum
        healthy: False
        """
        vision_chain = load_image_chain | image_model | parser
        try:
            return vision_chain.invoke(
                {"image_path": f"{image_path}", "prompt": vision_prompt}
            )
        except OutputParserException:
            return {
                "name": "Not a plant",
                "healthy" : False
                
            }
        except Exception:
            return {
                "name": "Not a plant",
                "healthy" : False
            }
```

Utilicé la función de búsqueda de Tavily para encontrar información sobre el nombre de la planta identificada, junto con consejos útiles para su cuidado, completando así el proceso.
```python
    tool = TavilySearchResults(
        max_results=1,
        search_depth="advanced",
        include_answer=True,
        include_raw_content=True,
        include_images=True,
        include_domains=["https://www.gardenia.net/"]
    )
    if(result['healthy']):
        response = tool.invoke({"query": result["name"]})
    else:
        response = tool.invoke({"query": result["name"] + " plant care tips"})
```

Desde la identificación hasta la generación de información, ¡este proyecto ha sido una experiencia gratificante que combina mi pasión por la tecnología y la naturaleza!

## 💡Puntos clave del PoC
- **ElevenLabs:** Voz generada automáticamente para el video basada en un guion creado con ChatGPT.
- **Tavily:** Búsqueda en la web para obtener información sobre las plantas.
- **Toolkit:** Cadena de preguntas y respuestas de LangChain para generar una salida en formato JSON a partir de un texto dado.
- **Modelo:** OpenAI 4o
