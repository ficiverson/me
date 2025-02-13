---
title: "Crea un podcast sin intervención humana"
meta_title: "Crea un podcast sin intervención humana"
description: "Crea un podcast sin intervención humana"
date: 2024-11-05T05:00:00Z
image: "/images/blog/podcast/podcast.jpg"
categories: ["AI"]
author: "Fernando Souto"
lang: "es"
tags: ["Podcast","NotebookLM", "Langchain"]
draft: false
---

Google Notebook LM está revolucionando la investigación impulsada por IA, la creación de contenido y la organización de datos. Esta herramienta proporciona información estructurada, lo que la hace ideal para investigadores, creadores de contenido y entusiastas de la IA.

## **Paso a paso para replicar su funcionamiento**

Un experimento reciente demostró las capacidades de un LLM para generar contenido dinámico y conversacional mediante la simulación de un diálogo entre dos personajes virtuales, Adam y Marina, sobre la historia de los videojuegos.

Contexto proporcionado:
```python
    HISTORY = """History of Video Games: A Detailed Review
    This briefing document reviews the key themes and significant facts from the provided excerpt of the Wikipedia article "History of video games." It explores the evolution of video games from their nascent stages to the present day, highlighting pivotal moments, technological advancements, and the emergence of distinct gaming cultures and markets.
    
    Early Beginnings (1948-1970): From Academic Experiments to Arcades
    Early Incarnations: Though debated, Bertie the Brain (1950) and Tennis for Two (1958) stand as early examples of electronic gaming. These pre-commercial endeavors laid the groundwork for future interactive entertainment.
    The Genesis of "Spacewar!": Created in 1962 by MIT students, Spacewar! is recognized as one of the first widely played computer games, utilizing a PDP-1 minicomputer. This development sparked further innovation and commercial interest.
    1970s: The Dawn of a New Industry
    Arcade Origins: The arcade game industry emerged from the existing arcade game industry dominated by electro-mechanical games. Sega's Periscope (1966) marked a significant advancement, setting the stage for commercial video games in arcades.
    The First Home Console: Ralph Baer and his team at Sanders Associates developed the "Brown Box" prototype, licensed to Magnavox, which became the Magnavox Odyssey (1972), the first commercial home console.
    "Pong" Ignites the Market: Nolan Bushnell and Ted Dabney, founders of Atari, developed Computer Space (1971), the first commercial arcade video game. Their subsequent release of Pong in 1972, is considered the first universally successful arcade game.
    "Pong was the first arcade video game to ever receive universal acclaim."
    The Rise and Fall of "Pong" Clones: The success of Pong led to a wave of clones, saturating the market by 1974. This forced developers to innovate, resulting in a decline of many clone manufacturers.
    The Rise of the Microprocessor: Affordable microprocessors replaced expensive discrete circuitry, leading to the development of ROM cartridge-based home consoles. The Fairchild Channel F (1976) pioneered this technology.
    The Golden Age of Arcades (1978-1982): A Period of Unprecedented Growth
    "Space Invaders" Revolutionizes Arcades: Tomohiro Nishikado's Space Invaders (1978) introduced new gameplay elements like lives, high scores, and interactive targets. This ignited the golden age of arcade games, a period of rapid innovation and immense popularity.
    Iconic Games Emerge: A slew of influential and bestselling arcade games, including Asteroids, Galaxian, Pac-Man, Donkey Kong, and Qbert*, captivated players and propelled the industry to new heights.
    Arcades Ascend to Cultural Dominance: By July 1982, coin-operated video games generated $7.7 billion in revenue, surpassing both pop music and Hollywood films, solidifying arcades as the leading entertainment medium in the United States.
    "These figures made arcade games the most popular entertainment medium in the country, far surpassing both pop music (at $4 billion in sales per year) and Hollywood films ($3 billion)."
    The Home Console Market Takes Shape
    Cartridge-Based Consoles Gain Traction: The Atari VCS (1977) and Magnavox Odyssey 2 (1978) solidified the use of programmable ROM cartridges, offering players a wider variety of games.
    "Space Invaders" Becomes the First Killer App: Atari's licensing deal with Taito to create a VCS version of Space Invaders propelled the console to unprecedented success, solidifying its position as the market leader.
    The Video Game Crash of 1983: A Crisis of Oversaturation and Quality
    Third-Party Developers Flood the Market: The success of Activision, a third-party developer for the Atari VCS, led to a surge of other companies creating games, often with poor quality and repetitive gameplay.
    Retailers Struggle with Inventory: The influx of low-quality games led to deep discounts and financial losses for retailers, further impacting the already-fragile market.
    Japan Emerges as the Leader: While the crash devastated the US market, Japanese companies remained relatively unscathed. This paved the way for Japan's dominance in the industry, led by Nintendo.
    The Home Console Recovery: Nintendo's Triumph and the "Bit Wars"
    The NES Revitalizes the Market: Nintendo’s meticulous approach with the NES (1985), including strict publishing control and marketing strategies, revitalized the US market.
    The Genesis and the "Bit Wars": Sega’s release of the 16-bit Genesis (1989) and Nintendo's subsequent release of the SNES initiated the "bit wars", where the bit size of the processor became a primary marketing tactic.
    Handheld Gaming Comes of Age: Nintendo's Game Boy (1989), bundled with Tetris, captured a new segment of the gaming market and achieved unprecedented success in the handheld console market.
    1990s: Technological Advancements and Genre Diversification
    Transition to Optical Media: The shift to CD-ROMs for both PCs and consoles provided greater storage capacity and reduced production costs. Sony's PlayStation (1994), designed solely for CD-ROMs, disrupted the market.
    The Rise of 3D Graphics: Advancements in microprocessor technology led to real-time 3D graphics, transforming the visual landscape of video games.
    New Genres Take Center Stage: The 1990s witnessed the emergence of iconic genres: First-person shooters (Wolfenstein 3D, Doom, Quake), Real-time strategy games (Dune II), and MMOs (Ultima Online).
    2000s: Online Gaming and the Rise of Indie Developers
    The Changing Console Landscape: Microsoft entered the console market with the Xbox (2001). Sony's PS2 became the best-selling console of all time. Nintendo's Wii (2006), featuring motion controls, drew in non-traditional players.
    MMOs, Esports, and Online Services: The growth of the internet brought online multiplayer gaming to the forefront, with titles like World of Warcraft dominating the MMO scene. Esports gained popularity. Online services for consoles, such as Xbox Live and PlayStation Network, emerged, enabling online multiplayer, digital downloads, and social features.
    The Rise of Mobile Gaming: Mobile gaming evolved beyond simple time-killers. The introduction of smartphones with advanced capabilities and app stores like Apple's App Store and Google Play revolutionized mobile gaming, making it a dominant force in the industry.
    AAA vs. Indie: The rising costs of game development led to the distinction between AAA (big-budget, blockbuster) games and indie (independent, often innovative and experimental) games.
    2010s: High-Definition, New Revenue Models, and VR/AR
    High-Definition Graphics and Online Advancements: Consoles and PCs pushed graphical fidelity with high-definition visuals. Cross-platform play and cloud gaming became more prominent.
    New Revenue Models: Microtransactions, season passes, and battle passes became increasingly common ways to monetize games. Loot boxes generated controversy due to their gambling-like mechanics.
    Mixed, Virtual and Augmented Reality Games: VR technology matured, with devices like the Oculus Rift, HTC Vive, and PlayStation VR entering the market. While VR gaming sought its killer app, augmented reality games like Pokémon Go achieved mainstream success.
    2020s: The Metaverse, Ray Tracing, and Industry Consolidation
    Ray Tracing and Photorealistic Graphics: The introduction of real-time ray tracing in GPUs and consoles like the Xbox Series X/S and PlayStation 5 brought unprecedented realism to video games.
    The Metaverse: The concept of a persistent, interconnected virtual world gained traction, leading to significant industry investment and speculation.
    "Moving into the 2020s, the concept of the metaverse grew in popularity. Similar in nature to the social spaces of Second Life, the concept of a metaverse is based on using more advanced technology like virtual and augmented reality to create immersive worlds that not only can be used for social and entertainment functions but as well as for personal and business purposes, giving the user the ability to earn from participation in the metaverse."
    Blockchain and NFT Games: The integration of blockchain technology and NFTs into games introduced new models of ownership, monetization, and player interaction. While met with excitement, the long-term viability and ethical implications remain debated.
    Video Game Acquisitions: The drive to dominate the metaverse and diversify offerings led to numerous high-profile acquisitions, with companies like Microsoft, Sony, and Tencent acquiring major studios and publishers.
    Conclusion: A Legacy of Innovation and Cultural Impact
    The history of video games is a testament to constant innovation, driven by technological advancements, changing consumer tastes, and the creative vision of developers. From humble beginnings in academic labs to a global entertainment behemoth, video games have shaped popular culture, inspired technological breakthroughs, and provided countless hours of entertainment for generations."""
```

Diseño del prompt
```python
    model = AzureChatOpenAI(
        azure_deployment=os.environ['AZURE_OPENAI_DEPLOYMENT'],
        model_version="2024-05-13",
        api_version="2024-02-01",
        temperature=0
    )
    prompt = PromptTemplate.from_template( """You are an experienced podcast host creating a lively conversation between two speakers, Adam and Marina, for a podcast called The videogames history. 
    Your output should be only JSON list with each item containing speaker and text attributes.
    Generate a conversation that feels dynamic and engaging, with a target length of at least 50,000 characters.
    Use a natural, conversational tone, filled with short sentences suitable for speech synthesis.
    Adam, video games expert, is the primary speaker providing insights, while Marina, the co-host, asks thoughtful questions and adds reactions.
    Do formal introductions to the podcast episode. Use casual phrasing to reflect their established rapport.
    Sprinkle in filler words like "uh," "you know," or slight repetition to make it sound organic.
    Capture a sense of excitement and curiosity to make the conversation compelling and authentic. Use this text to create the podcast episode {{HISTORY}}
     """)
```

¿La salida? El modelo respondió con contenido JSON estructurado, formando un guion atractivo para el pódcast. Ejemplo de salida:
```python
    [
        {
            "speaker": "Adam",
            "text": "Hey everyone, welcome back to another episode of The Videogames History! I'm Adam, your resident video game expert."
        },
        {
            "speaker": "Marina",
            "text": "And I'm Marina, your co-host. Super excited to dive into today's topic. How are you doing, Adam?"
        },
        {
            "speaker": "Adam",
            "text": "I'm doing great, Marina! Ready to talk about some awesome video game history. How about you?"
        },
        ...
        {
            "speaker": "Adam",
            "text": "Until then, keep on gaming!"
        }
    ]
```

Conversión de texto a voz con Eleven labs API:
```python
    def tts(text, speaker,index):
        speech_file_path = "audio-files/"+str(index)+"_"+speaker+".mp3"
        llm_speaker = "Xb7hH8MSUJpSbSDYk0k2"
        if speaker == "Adam":
            llm_speaker = "nPczCjzI2devNBz1zQrb"
        else:
            llm_speaker = "Xb7hH8MSUJpSbSDYk0k2"
    
    
        url = f"https://api.elevenlabs.io/v1/text-to-speech/{llm_speaker}"
        headers = {
            "Accept": "audio/mpeg",
            "Content-Type": "application/json",
            "xi-api-key": os.environ['ELEVEN_LABS_KEY'],
        }
        data = {
            "text": text,
            "model_id": "eleven_monolingual_v1",
            "voice_settings": {
                "stability": 1,
                "similarity_boost": 0,
            },
        }
        response = requests.post(url, json=data, headers=headers)
        response.raise_for_status()
        audio_data = response.content
    
        audio_segment = AudioSegment.from_file(io.BytesIO(audio_data), format="mp3")
        audio_segment = audio_segment._spawn(audio_segment.raw_data, overrides={"frame_rate": int(audio_segment.frame_rate * 1.0)})
    
        if speech_file_path:
            with open(speech_file_path, "wb") as f:
                audio_segment.export(f, format="mp3")
```

Después de generar 110 archivos de audio individuales, se combinaron en un solo archivo MP3 para una experiencia de pódcast sin interrupciones: 🎧
```python
    def merge_audios(audio_folder, output_file, audio_files):
        merged = AudioSegment.empty()
        for filename in audio_files:
            audio_path = os.path.join(audio_folder, filename)
            audio = AudioSegment.from_file(audio_path)
            merged += audio
        merged.export(output_file, format="mp3")
        print(f"Podcast created at: {output_file}")
```
 
Aquí el resultado en un podcast:

{{< youtube UvKENzuSEbo >}}

Este experimento demuestra cómo la IA puede utilizarse para la narración de historias, la generación de contenido y experiencias multimedia inmersivas.

## **Aspectos destacados del PoC**

- **Eleven labs:** Usado para voces TTS de alta calidad.
- **Toolkit:** Langchain QA generó salida JSON estructurada a partir del texto proporcionado.
- **Model:** OpenAI 4o
