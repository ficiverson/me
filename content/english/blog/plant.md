---
title: "Introducing AI-Powered Plant Analyzer! 🌱"
meta_title: "Introducing AI-Powered Plant Analyzer! 🌱"
description: "Introducing AI-Powered Plant Analyzer! 🌱"
date: 2024-11-09T05:00:00Z
image: "/images/blog/plant/plant.webp"
categories: ["AI"]
author: "Fernando Souto"
tags: ["plant analyzer","OpenAI", "Gardentech","Plant care"]
draft: false 
---

I’ve developed an AI-powered plant analyzer that brings together cutting-edge tools from OpenAI Vision API and Tavily, with search capabilities to explore information on [gardenia.net](http://gardenia.net/).

🌿 With this tool, identifying plants and accessing detailed care information has never been easier! In this video, I walk through the creation process, explaining the technical steps and decisions behind building an AI that helps both plant enthusiasts and experts better understand and care for their plants.

🎥 Check out the video to learn more about how this AI analyzer works and the potential it holds for the world of horticulture. Let’s make plant care smarter, together!

{{< youtube ifhVcUsnqx0 >}}

## Let’s check the details

This code enables us to load an image directly from a specified folder, allowing for easy access and manipulation of image files stored locally.
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

We need to define a model that will serve as the output of the Language Learning Model (LLM). In this case, the model will generate information on the plant’s name and provide an assessment of its health status.
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

Develop the tool that the Language Learning Model (LLM) will use to analyze the image, enabling it to extract relevant insights and information from the visual data.
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

This code will invoke the tool using the specified model and provided prompt to analyze the image. The tool will process the visual information and return the results in a structured JSON response format
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

I utilized the Tavily search function to find information on the specified plant name, along with helpful tips for maintaining the plant’s health, and that completed the process!
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

From identification to insights — this tool has been a rewarding project combining my passion for tech and nature!

## 💡Key highlights of the PoC
- ElevenLabs: Voice autogenerated of the video based on a script created with chatgpt
- Tavily: Web search for information of the plants
- Toolkit: Langchain QA chain to create a JSON output from a text given
- Model: OpenAI 4o
