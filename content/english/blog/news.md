---
title: "Revolutionizing news with AI: How I built an automated news Podcast generator"
meta_title: "Revolutionizing news with AI: How I built an automated news Podcast generator"
description: "Revolutionizing news with AI: How I built an automated news Podcast generator"
date: 2025-04-02T05:00:00Z
image: "/images/blog/news/intro.jpeg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["vibe code", "Agents", "Eleven labs","LLM", "Langchain","Cursor"]
draft: false
---

I’ve always admired **Ángel Martín’s** approach to delivering news—straight to the point, no fluff, just the essentials. Inspired by that philosophy, I built an AI-powered news podcast generator focused on delivering concise, relevant news for my city, A Coruña. My goal was to create a system that keeps people informed without the need to sift through lengthy articles or multiple sources.

This fully automated process follows a structured workflow:

- **Gather information** – analyzing the latest headlines.

- **Select the top 3 news stories** – filtering the 3 most relevant topics.

- **Search for more context** – enhancing the selected news with additional details.

- **Create a script** – structuring a natural, engaging news narrative.

- **Generate and normalize the audio** – AI voices narrate, with mixing and music.

- **Generate podcast metadata** – ensuring proper tagging and description.

![AI characters Emilia and Marina](/images/blog/news/marina-emilia-news-ai-podcast.jpg)

Using a fusion of cutting-edge AI tools, this system automatically finds, structures, and narrates news, making staying informed easier than ever. **Average time for generation is about 1.5 mintues.**

# The tech stack: AI superpowers in action 

To make this possible, I combined some of the best AI tools available:

- **OpenAI** – Generates structured news and summaries based on the latest headlines.
- **LangChain** – Manages workflow that selects the most relevant news, creates a script, and ensures coherence.
- **Tavily** – Aggregates data from trusted sources to keep content fresh and reliable.
- **ElevenLabs** – Brings the news to life with realistic AI-generated voices that sound natural and engaging.
- **Cursor & Vibe Coding** – Supercharged development, allowing me to build the system efficiently in FastAPI.

![API generating the episode](/images/blog/news/podcast-generation.png) 

# How It Works 

- **Gather information & script generation** 

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
- **Example of a script** 
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

- **Generate audios**
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

- **Generate metadata**

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

# The future of AI-generated radio news

This project is just the beginning. I’m taking it further by integrating real-time personalization, multi-language support, and interactive features. But the most exciting part? I’m making this a real milestone by bringing it to **CUAC FM, a community radio station in A Coruña**. AI-powered news will soon hit FM waves, **merging the digital and analog worlds** in a way that hasn’t been done before—most likely making this the **first AI-generated news broadcast on FM radio in Spain**.

One key aspect of this workflow is **human-in-the-loop** intervention. While the AI automates most of the process, **the podcast isn’t uploaded automatically**. I personally review and approve each script before generating the final episode. This go or no-go decision ensures that every episode is accurate, contextually relevant, and aligned with my vision for **responsible AI-driven journalism.**

The future of news is **fast, intelligent, and effortless** and I’m here to make it happen.

![Podcast en Spotify](/images/blog/news/podcast.png) 

🎧 Want to check it out? Listen here: [Spotify Link](https://open.spotify.com/show/1aoJwQZPKoDEZ1SvvwrWrJ)

🗞️ Follow the lastest news on bluesky: [Bluesky Link](https://bsky.app/profile/noticiasacoruna.bsky.social)

👂 Would love to hear your thoughts! Drop a comment or reach out if you’re interested in AI-driven media.

