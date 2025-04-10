---
title: "Revolucionando las noticias con IA: Cómo creé un generador automatizado de noticias en formato podcast"
meta_title: "Revolucionando las noticias con IA: Cómo creé un generador automatizado de noticias en formato podcast"
description: "Revolucionando las noticias con IA: Cómo creé un generador automatizado de noticias en formato podcast"
date: 2025-04-02T05:00:00Z
image: "/images/blog/news/intro.jpeg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["vibe code", "Agents", "Eleven labs","LLM", "Langchain","Cursor"]
draft: false
---

Siempre he admirado el enfoque de **Ángel Martín** para dar las noticias: directo y al grano, sin rodeos, solo lo esencial. Inspirado por esa filosofía, creé un generador de pódcast de noticias impulsado por IA centrado en ofrecer noticias concisas y relevantes para mi ciudad, A Coruña. Mi objetivo era desarrollar un sistema que mantuviera a las personas informadas sin necesidad de recorrer artículos extensos o múltiples fuentes.

Este proceso completamente automatizado sigue un flujo de trabajo estructurado:

- **Recopilar información** – analizar los titulares más recientes.

- **Seleccionar las 3 noticias principales** – filtrar los temas más relevantes.

- **Buscar más contexto** – ampliar las noticias seleccionadas con detalles adicionales.

- **Crear un guion** – estructurar una narrativa informativa y atractiva.

- **Generar y normalizar el audio** – voces de IA narran con música de fondo.

- **Generar metadatos del pódcast** – asegurar el etiquetado y descripción adecuados.

![AI characters Emilia and Marina](/images/blog/news/marina-emilia-news-ai-podcast.jpg) 

Mediante una combinación de herramientas de IA de vanguardia, este sistema encuentra, estructura y narra noticias de forma automática, facilitando el acceso a la información como nunca antes. **El tiempo medio de generación es de 1 minuto y medio.**

## El stack tecnológico: superpoderes de AI en acción 

Para hacer esto posible, combiné algunas de las mejores herramientas de IA disponibles:

- **OpenAI** –  Genera noticias estructuradas y resúmenes basados en los últimos titulares.
- **LangChain** – Gestiona el flujo de trabajo que selecciona noticias relevantes, crea guiones y garantiza coherencia.
- **Tavily** – Agrega datos de fuentes confiables para mantener el contenido fresco y fiable.
- **ElevenLabs** –  Da vida a las noticias con voces de IA realistas, naturales y atractivas.
- **Cursor & Vibe Coding** –  Aceleran el desarrollo, permitiéndome construir el sistema de manera eficiente en FastAPI.

![API generating the episode](/images/blog/news/podcast-generation.png) 

## ¿Cómo funciona?

- **Recolectar información y generación del guión** 

```python
#generate the script of the episode
 async def generate_episode_script(self, day: str, rss_urls: List[str]) -> str:
        try:
            # Get news and context
            news = self._get_news(rss_urls)
            context = self._get_context(news, rss_urls)
            
            # Create the prompt
            prompt = self._get_context_prompt()
            
            # Run the chain with proper parameters
            result = run_chain(
                prompt=prompt,
                primary=news,
                secondary=context,
                additional_context=day
            )
            
            print("Result generation RSS news:", result)
            return result
        except Exception as e:
            print(f"Error in generate_episode: {str(e)}")
            raise
```
```python
#get news from sources feeds
 def _get_news(self, rss_urls: list[str]) -> str:
        try:
            rss_converted = self.process_multiple_rss(rss_urls)
            prompt = self._get_news_prompt()
            return run_chain(prompt=prompt, primary=rss_converted)
        except Exception as e:
            print(f"Error in _get_news: {str(e)}")
            raise
#pomrpt to get top 3 news from feeds
def _get_news_prompt(self) -> PromptTemplate:
        template = """Analyze the following RSS content and select the 3 most important news stories of the day, ensuring that you include news from different sources:

        {primary}

        Please provide a summary in the following format:

        **Top 3 news of the Day **
        - **Título: [Title]**
        - **Descripción:** [Description]
        - **Fuente:** [Source]

        IMPORTANT: Make sure to select news from different sources. Do not select more than 2 stories from the same source. Keep the original title, description and link of the news"""

```
```python
#prompt to generate the scrtipt using LLM
def _get_context_prompt(self) -> PromptTemplate:
        return PromptTemplate.from_template( """You are creating a professional radio news show called "Las noticias de A Coruña" with two presenters: Marina, the main news anchor, and Emilia, the news analyst and commentator.

                Your output should be only a valid JSON list with each item containing speaker and text attributes.
                Generate a professional news broadcast that feels like a high-quality radio show, with a target length less than 5,000 characters.

                Structure the show as follows:
                1. Opening theme and greetings (mention today is {additional_context})
                2. Marina presents only the headlines and name the sources
                3. For each news story:
                    - Marina presents the detailed news story
                    - Emilia provides analysis, context, and expert commentary using the following context: {secondary}
                    - Marina may ask Emilia for additional insights or clarification
                4. Closing remarks and sign-off

                Use a professional yet engaging tone:
                    - Marina should maintain a clear, authoritative news anchor voice
                    - Emilia should provide thoughtful analysis with a conversational tone
                    - Include natural transitions between stories
                    - Use professional radio phrases like "Continuamos con..." or "Volvemos con..."

                The conversation should be in Castillian Spanish.
                Only use for the context of the news following context: {primary}
                """)
```
- **Ejemplo de guión** 
```json
[
    {
        "speaker": "Marina",
        "text": "¡Bienvenidos a 'Las noticias de A Coruña'! Hoy es 3 de abril de 2025. En el programa de hoy, les traemos las siguientes noticias destacadas."
    },
    {
        "speaker": "Marina",
        "text": "1. El doctor Diego González Rivas opera desde China un tumor en Rumanía. Fuente: La Opinión A Coruña."
    },
    {
        "speaker": "Marina",
        "text": "2. La tromba de agua y granizo crea balsas de agua en A Coruña, pero sin cortes de circulación ni accidentes. Fuente: La Opinión A Coruña."
    },
    {
        "speaker": "Marina",
        "text": "3. Bernardo Fernández, proclamado Secretario Xeral del PSOE provincial de A Coruña. Fuente: La Opinión A Coruña."
    },
    {
        "speaker": "Marina",
        "text": "Comenzamos con la primera noticia. El doctor coruñés Diego González Rivas ha realizado una operación a distancia, manejando un robot quirúrgico desde Suzhou, China, para extirpar un tumor a una paciente en Bucarest, Rumanía. Esta intervención, calificada como la primera cirugía torácica transcontinental de la historia, marca un hito en el ámbito médico."
    },
    {
        "speaker": "Emilia",
        "text": "Así es, Marina. Esta operación no solo es impresionante por su complejidad técnica, sino que también subraya el avance en la telemedicina. La capacidad de operar a 8,000 kilómetros de distancia abre la puerta a nuevas posibilidades en la atención médica, especialmente para pacientes en áreas remotas. Es un claro ejemplo de cómo la tecnología puede superar barreras geográficas."
    },
    {
        "speaker": "Marina",
        "text": "Interesante análisis, Emilia. Además, el doctor González Rivas mencionó que la intervención fue posible gracias a una conexión 5G, lo que minimizó el retardo en las órdenes. Esto es crucial en procedimientos quirúrgicos donde cada milisegundo cuenta."
    },
    {
        "speaker": "Emilia",
        "text": "Exactamente, Marina. La tecnología 5G es un cambio de juego en este contexto. La rapidez de la conexión permite que el cirujano actúe en tiempo real, lo que es vital para la precisión en cirugías complejas. Esto podría transformar la manera en que abordamos la cirugía en el futuro."
    },
    {
        "speaker": "Marina",
        "text": "Continuamos con la siguiente noticia. Una tromba de agua y granizo ha afectado a A Coruña, creando balsas de agua en varias vías, aunque afortunadamente no se han reportado cortes de circulación ni accidentes. La borrasca Nuria ha llevado a los servicios de emergencia a actuar en diversas áreas de la ciudad."
    },
    {
        "speaker": "Emilia",
        "text": "La respuesta de las autoridades locales ha sido notable, Marina. A pesar de las condiciones climáticas extremas, la Policía Local y los bomberos se han movilizado rápidamente para garantizar la seguridad de los ciudadanos. Esto demuestra la importancia de tener un plan de emergencia efectivo en situaciones de crisis."
    },
    {
        "speaker": "Marina",
        "text": "Así es, Emilia. Aunque la situación ha sido complicada, la capacidad de respuesta ha sido clave para evitar mayores problemas. Pasemos a la tercera noticia. Bernardo Fernández ha sido proclamado nuevamente Secretario Xeral del PSOE provincial de A Coruña, tras no recibir ninguna otra propuesta alternativa durante el periodo de candidaturas."
    },
    {
        "speaker": "Emilia",
        "text": "La continuidad en el liderazgo de Fernández puede ser un factor estabilizador para el PSOE en A Coruña. Su experiencia como alcalde de Pontedeume y portavoz del grupo socialista en la Diputación seguramente jugará un papel importante en el futuro del partido en la provincia."
    },
    {
        "speaker": "Marina",
        "text": "Así es, Emilia. Su reelección podría influir en la estrategia política del PSOE, especialmente de cara al congreso provincial que se celebrará en mayo. Es un momento clave para la política local."
    },
    {
        "speaker": "Emilia",
        "text": "Sin duda, Marina. La política local está en constante evolución, y estos cambios pueden tener un impacto significativo en la vida de los ciudadanos de A Coruña."
    },
    {
        "speaker": "Marina",
        "text": "Y así concluimos nuestras principales noticias del día. Gracias por acompañarnos en 'Las noticias de A Coruña'."
    },
    {
        "speaker": "Emilia",
        "text": "Hasta la próxima, y recuerden mantenerse informados y seguros."
    }
]
```

- **Generación de audios**
```python
  # Merge audio generated
  async def merge_folder_audios(self, output_file: str) -> str:
        """Merge all audio files in the storage folder into one"""
        combined = AudioSegment.empty()
        audio_files = sorted(
            [f for f in os.listdir(self.files_base_path) if f.endswith(f".{self.settings.AUDIO_FORMAT}")],
            key=self._sort_key
        )
        
        target_frame_rate = 44100  # Standard frame rate
        
        for filename in audio_files:
            audio_path = self.files_base_path / filename
            print(f"Processing: {audio_path}")
            
            # Load and normalize frame rate
            audio = AudioSegment.from_file(audio_path)
            if audio.frame_rate != target_frame_rate:
                audio = audio.set_frame_rate(target_frame_rate)
            
            # Add small silence between segments to prevent frame misalignment
            if len(combined) > 0:
                silence = AudioSegment.silent(duration=50, frame_rate=target_frame_rate)
                combined = combined + silence
            
            combined = combined + audio
        
        output_path = self.base_path / output_file
        combined.export(
            output_path, 
            format=self.settings.AUDIO_FORMAT,
            parameters=["-ar", str(target_frame_rate)]  # Ensure consistent frame rate in output
        )
        print(f"Merged audio saved as {output_file}")
        return output_file

  # Generte audios using Eleven labs API
  def synthesize(
        self,
        text: str,
        voice_id: Optional[str] = None,
        model: Optional[str] = None,
        output_format: Optional[str] = None
    ) -> bytes:
        """
        Synthesize text to speech using ElevenLabs API.
        
        Args:
            text: The text to synthesize
            voice_id: Optional voice ID to use (defaults to None)
            model: Optional model to use (defaults to settings.ELEVENLABS_MODEL)
            output_format: Optional output format (defaults to settings format string)
            
        Returns:
            bytes: The audio data
        """
        # Use default settings if not provided
        model = model or self.settings.ELEVENLABS_MODEL
        output_format = output_format or self._get_format_string()

        # Generate audio
        client = ElevenLabs(
            api_key=get_settings().ELEVENLABS_API_KEY,
        )
        audio_data = client.text_to_speech.convert(
            text=text,
            voice_id=voice_id,
            model_id="eleven_multilingual_v2",
            output_format="mp3_44100_128",
        )

        return audio_data

```

- **Generación de metadata**

```python
 async def generate_metadata(self, script: str, day: str) -> PodcastMetadata:
        try:
            prompt = self._get_description_prompt()
            description = run_chain(
                prompt=prompt,
                primary=script
            )
            music_disclaimer = """Música by [⁠⁠⁠⁠⁠⁠⁠Creative Freed](https://pixabay.com/users/oqu-12487683/?utm_source=link-attribution&utm_medium=referral&utm_campaign=music&utm_content=254286) from [Pixabay] (https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=music&utm_content=254286)"""
            
            return PodcastMetadata(
                title=f"Noticias de A Coruña -  {day}",
                description=description + "\n\n" + music_disclaimer,
                script=script
            )
        except Exception as e:
            print(f"Error in generate_metadata: {str(e)}")
            raise

def _get_description_prompt(self) -> PromptTemplate:
        return PromptTemplate.from_template( """Create in spanish a description for a episode of a podcast called "Las noticias de A Coruña". The news discussed in the episode are following ones : {primary}
 """)

```

## El futuro de las noticias generadas por IA

Este proyecto es solo el comienzo. Voy más allá integrando personalización en tiempo real, soporte multilingüe y funciones interactivas. ¿Qué es lo más emocionante? Llevar esta tecnología a **CUAC FM, una emisora comunitaria de A Coruña**. Las noticias impulsadas por IA llegaron el 11/04/2025 a las ondas de FM marcando un hito histórico para la radiodifusión española, **fusionando los mundos digital y analógico** de una manera que no se ha hecho antes, convirtiéndolo probablemente en el **primer programa de noticias generado por IA en la radio FM en España**.

Un aspecto clave de este flujo de trabajo es la **intervención humana**. Aunque la IA automatiza la mayor parte del proceso, **el pódcast no se publica automáticamente**. Personalmente reviso y apruebo cada guion antes de generar el episodio final. Esta decisión de "publicar o no publicar" garantiza que cada episodio sea preciso, contextualmente relevante y alineado con mi visión de un periodismo impulsado por **IA responsable.**

El futuro de las noticias es **rápido, inteligente y sin esfuerzo** y estoy aquí para hacerlo realidad.

![Podcast en Spotify](/images/blog/news/podcast.png) 

🎧 ¿Quieres escucharlo? Encuéntralo aquí: [Spotify Link](https://open.spotify.com/show/1aoJwQZPKoDEZ1SvvwrWrJ)

🗞️ Sigue las últimas noticias en Bluesky: [Bluesky Link](https://bsky.app/profile/noticiasacoruna.bsky.social)

👂 ¡Me encantaría conocer tu opinión! Déjame un comentario o contáctame si te interesa el periodismo impulsado por IA.

