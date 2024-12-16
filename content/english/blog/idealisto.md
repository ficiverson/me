---
title: "Introducing ‘Idealisto’: Your AI Chatbot for the Spanish Real Estate Market"
meta_title: "Introducing ‘Idealisto’: Your AI Chatbot for the Spanish Real Estate Market"
description: "Introducing ‘Idealisto’: Your AI Chatbot for the Spanish Real Estate Market"
date: 2024-11-19T05:00:00Z
image: "/images/blog/idealisto/idealisto.webp"
categories: ["AI"]
author: "Fernando Souto"
tags: ["langgraph", "Tavily","Idealista", "LLM"]
draft: false 
---

>  Navigating the Spanish real estate market just got easier with *Idealisto*! Whether you’re a savvy investor, a first-time buyer, or a real estate professional, this cutting-edge AI chatbot is here to help you uncover trends, get legal advice, and spot market opportunities with unprecedented precision.

## How Idealisto Works

Idealisto combines powerful technologies to provide you with actionable insights:

**LangGraph for query handling**

* Using advanced graph-based technology, LangGraph processes complex queries efficiently and also allows to debug every step of the chain like our *old backend systems*.

* It connects various data points to ensure your questions — no matter how specific — are understood and answered.

**Tavily for data retrieval**

* Tavily searches the *Idealista.com *website to pull up-to-date listings, market trends, and property insights.

* This ensures that you’re getting accurate, real-time data straight from one of Spain’s top real estate platforms.

**LLM for Smart Answers**

* Leveraging a large language model (LLM), Idealisto generates comprehensive and conversational responses.

* It synthesizes legal advice, market analysis, and investment opportunities into easy-to-understand answers tailored to your needs.

## How can you replicate Idealisto?

You can follow this step-by-step tutorial and easily copy and paste code snippets to recreate the functionality.

The next graph represent the logic that is executed with all the queries:

![LangGraph architecture](/images/blog/idealisto/lang_graph.webp)

 1. **graph creation**
>  This is usually the final step, but it’s just about showing how to create a graph with LangGraph.
```python
    from typing import Annotated
    from typing_extensions import TypedDict
    from langgraph.graph import StateGraph, START, END
    from langgraph.graph.message import add_messages
    
    
    class State(TypedDict):
        messages: Annotated[list, add_messages]
        memory: list = []
        context: list = []
        loop_needd:bool
        language:str
        idealista_search: bool
    
    
    graph_builder = StateGraph(State)
    graph_builder.add_node("idealisto", idealisto)
    graph_builder.add_node("context_check",context_check)
    graph_builder.add_node("reflect_general",reflect_general)
    graph_builder.add_node("reflect_idealista",reflect_idealista)
    graph_builder.add_node("web_search",web_search)
    
    
    graph_builder.add_edge(START, "context_check")
    graph_builder.add_conditional_edges("context_check", should_query_context)
    graph_builder.add_conditional_edges("reflect_general", reflect_conditional_general)
    graph_builder.add_conditional_edges("reflect_idealista", reflect_conditional_idealista)
    graph_builder.add_edge("web_search","idealisto")
    graph_builder.add_edge("idealisto",END)
    
    graph = graph_builder.compile()
```

**2. context_check**
>  Figure out if the query needs Tavily to search or if the LLM’s internal knowledge is enough to answer.
```python
    from pydantic import BaseModel, Field
    from typing import Literal
    
    # Pydantic
    class ContextEvaluator(BaseModel):
        """You are a specialist on identifying types of questions. Your task is to identify if the given question needs more context from Idealista website for properties or use LLM internal knowledge
        
        #Examples of questions that should use Idealista search
        Looking for an appartment on Madrid
        Cozy flat on Barcelona for renting?
        I want to buy a flat on Sevilla?
    
        #Examples of questions that should use internal LLM knowledge
        What are the environmental permit requirements in Valencia?
        What is idealista?
        Hello, how is going? or other smalltalks
        """
    
        provider: Literal["llm_knowledge", "idealista"] = Field(description="The type of the literal")
        user_query: str = Field(description="This is the query that we will send to the answer generation module.")
    
    which_context = llm_model.with_structured_output(ContextEvaluator)
    
    
    
    def context_check(state: State):
        result = which_context.invoke(state["messages"])
        return { "user_query":result.user_query,"idealista_search": result.provider == "idealista"}
```
**3. reflect_idealista**
>  A tool to revisit the user’s original question and rephrase it if needed.
```python
    from pydantic import BaseModel, Field
    
    
    # Pydantic
    class IsQuestionCorrectGeneral(BaseModel):
        """You are a highly skilled AI agent tasked with optimizing human-generated questions for effective searches using Tavily on the Idealista website. Your goal is to enhance the original query by making it more specific and search-friendly. Focus on extracting key details such as city, address, price range, property type, and any other relevant criteria from the query to improve search accuracy.
        """
    
        new_question: str = Field(description="Question rephrased or empty string")
        reason: str = Field(description="Explain the reason why you rephrased")
        language: str = Field(description="English or Spanish depending on the question")
        reason_language : str = Field(description="Explain the reason why you selected a that language")
    
    
    is_a_loop_need = llm_model.with_structured_output(IsQuestionCorrectGeneral)
    
    def reflect_idealista(state: State):
        result = is_a_loop_need.invoke(state["messages"])
        if len(result.new_question)==0:
            return {"loop_needd": False, "language":result.language}
        else:
           return {"loop_needd": True, "messages": [result.new_question], "language":result.language}
    
    def reflect_conditional_idealista(state: State) -> Literal["context_check", "web_search"]:
        if state["loop_needd"] and len(state["messages"]) < 2:
            return "context_check"
        return "web_search"
```

**4. reflect_general**
>  A tool to review the LLM’s answer and explore other similar questions if needed.
```python
    from pydantic import BaseModel, Field
    
    
    # Pydantic
    class IsResponseCorrectGeneral(BaseModel):
        """You are an expert AI agent responsible for evaluating answers generated by LLMs. Your task is to review each answer provided in response to a question and rephrase only if the answer is inaccurate or lacks clarity.
            Always maintain the language of the original question—if the question is in English, respond in English; if the question is in Spanish, respond in Spanish.
            Skip evaluations for responses that indicate the question is out of scope for the LLM, such as:
    
            'I'm sorry, I am a specialized chatbot for real estate questions in Spain.'
            'I don’t know the answer.'
           
           Do not rephrase simple greetings or small talk, including:
    
            'Hello'
            'How are you?'
            'What is your name?'
            Focus only on responses where clarity or accuracy could be improved, while leaving all other answers unchanged.
        """
    
        new_question: str = Field(description="Question rephrased or empty string")
        reason: str = Field(description="Explain the reason why you rephrased")
        language: str = Field(description="English or Spanish depending on the question")
        reason_language : str = Field(description="Explain the reason why you selected a that language")
    
    
    is_a_loop_need2 = llm_model.with_structured_output(IsResponseCorrectGeneral)
    
    def reflect_general(state: State):
        result = is_a_loop_need2.invoke(state["messages"])
        if len(result.new_question)==0:
            return {"loop_needd": False, "language":result.language}
        else:
           return {"loop_needd": True, "messages": [result.new_question], "language":result.language}
    
    def reflect_conditional_general(state: State) -> Literal["context_check", "idealisto"]:
        if state["loop_needd"] and len(state["messages"]) < 2:
            return "context_check"
        return "idealisto"
```

**5. web_search**
>  Tavily searches the idealista.com website.
```python
    from langchain_community.tools.tavily_search import TavilySearchResults
    
    def web_search(state: State):
        tool = TavilySearchResults(max_results=5)   
        question = state["messages"][0].content
        idealista_result = tool.invoke(f"site:www.idealista.com " + question)
        return {"context": idealista_result}
```

**6. idealisto**
>  The LLM generates the final answer using all the previous information.
```python
    from langchain_openai import AzureChatOpenAI
    
    
    history_list = []
    
    llm_model = AzureChatOpenAI(
                azure_deployment=os.environ.get("AZURE_OPENAI_DEPLOYMENT"),
                model_version=os.environ.get("AZURE_OPENAI_VERSION"),
                api_version=os.environ.get("AZURE_OPENAI_API_VERSION"),
            )
    
    template = os.getenv(
                "PROMPT",
                """You are Idealisto, an expert chatbot specialized in providing accurate and relevant answers about real estate in Spain, fully grounded in Spanish law.
    
                    Guidelines:
    
                    Audience: The users are people looking for a house in Spain, meaning that Spanish law is applicable in all responses.
                    Chat History: Use the provided chat history for any necessary background information: {memory}.
                    Context: Refer to the following context to ensure accuracy in answering the user's question: {context}. If the context does not explicitly cover the question, rely on your general knowledge.
                    User's Question: {messages}
                    Helpful Answer (in {language}):""",
            )
    
    prompt = PromptTemplate.from_template(template)
    chain = prompt | llm_model | StrOutputParser()
    
    def idealisto(state: State):
        messages = state["messages"]
        memory = state.get("memory","") 
        context =  state.get("context", "") or ""
        language = state.get("language", "") or ""
        inputs = {
            "context": context,
            "memory": memory,
            "language": language,
            "messages": messages
        }
        
        result = chain.invoke(inputs)
        history_list.append({"question": state["messages"][-1].content, "answer":result})
        return {"messages": [result], "memory" : history_list}
```

## Examples of questions and answers

```python
    Hi! which is your name?
    
    provider='llm_knowledge' user_query='Hi! which is your name?'
    The response does not need rephrasing, as it is a simple greeting.
    English
    The question is in English.
    loop need:
    False
    content='Hi! which is your name?' additional_kwargs={} response_metadata={} 
    
    Response:
    Hello! My name is Idealisto. I'm here to help you with any questions you may have about real estate in Spain. How can I assist you today?
    Sources:
    Content generated using the LLM knowledge
```
```python
    give me 4 porperties in the moncloa area of Madrid
    
    provider='idealista' user_query='give me 4 properties in the Moncloa area of Madrid'
    The question was rephrased to specify the request for properties in a certain area, making it clearer for search purposes.
    English
    The original question was in English, so I maintained the same language.
    loop need:
    True
    content='Find 4 properties in Moncloa, Madrid.' additional_kwargs={} response_metadata={} 
    give me 4 porperties in the moncloa area of Madrid
    
    
    Response:
    Here are four properties available in the Moncloa area of Madrid:
    
    Penthouses for Sale:
    
    Price: From 390,000 euros
    Description: 53 penthouses available, providing a unique living experience with landscaped terraces.
    More Information: View Penthouses
    Flats and Apartments for Sale:
    
    Price: From 84,000 euros
    Description: 509 flats and apartments, featuring a large kitchen of 20m² with a central island.
    More Information: View Flats and Apartments
    Houses and Flats for Sale:
    
    Price: From 210,000 euros
    Description: 75 properties available, including a spacious 244 m² property on the 2nd floor offering reform opportunities.
    More Information: View Houses and Flats
    Independent Houses for Sale:
    
    Price: From 101,000 euros
    Description: 663 properties, including a magnificent independent house located in the exclusive Aravaca area in Moncloa.
    More Information: View Independent Houses
    You can explore these listings further on the provided links to find the property that suits your needs!
    
    Sources:
    website content
    http://idealista.es [{'property': [{'text': '53 penthouses for sale in Madrid - Moncloa, Spain, from 390,000 euros. Property in Madrid - Moncloa for sale direct from owners and real estate agents. idealista, the leading real estate marketplace in Spain. ... It has been possible to give the property the feeling of living in an isolated home by landscaping the terraces and not having more', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/madrid/moncloa/con-aticos/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': '509 flats and apartments for sale in Madrid - Moncloa, Spain, from 84,000 euros. Property in Madrid - Moncloa for sale direct from owners and real estate agents. idealista, the leading real estate marketplace in Spain. ... - Large kitchen of 20m² with central island and large office area as a room was given to the kitchen to extend it', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/madrid/moncloa/con-pisos/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': 'Property for sale, Metro Moncloa, Madrid, Spain: 75 houses and flats from 210,000 euros in Spain. Property listings direct from owners and real estate agents on idealista, the leading real estate marketplace in Spain. ... Madrid. With a surface area of 244 m² built, this spacious property on the 2nd floor offers a unique opportunity to reform', 'metadata': {'path': 'https://www.idealista.com/en/geo/venta-viviendas/metro-moncloa/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': '663 houses and flats for sale in Madrid - Moncloa, Spain, from 101,000 euros. Property in Madrid - Moncloa for sale direct from owners and real estate agents. idealista, the leading real estate marketplace in Spain. ... This magnificent independent house is located in Aravaca, an exclusive residential area in the Moncloa - Aravaca district, in', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/madrid/moncloa/', 'relevance_score': 0.8}}]}]
    http://idealista.es
```
```python
    Which are the requisites to rent a hoouse in Spain?
    
    provider='llm_knowledge' user_query='Which are the requisites to rent a hoouse in Spain?'
    Rephrased for clarity and grammatical correctness.
    English
    The original question is in English.
    loop need:
    True
    content='What are the requirements to rent a house in Spain?' additional_kwargs={} response_metadata={}
    
    Response:
    To rent a house in Spain, you typically need to meet several requirements, which can vary depending on the landlord and the specific rental agreement. Here are the common requisites:
    
    Identification: You will need to provide a valid form of identification, such as a passport or national ID card.
    
    Proof of Income: Landlords usually require proof of income to ensure you can afford the rent. This can be in the form of pay slips, a work contract, or tax returns if you are self-employed.
    
    Employment Contract: A stable job is often a requirement, so landlords may ask for a copy of your employment contract.
    
    References: Some landlords may ask for references from previous landlords or personal references to vouch for your reliability as a tenant.
    
    Deposit: You will typically need to pay a security deposit, usually equivalent to one or two months' rent, which will be returned at the end of the rental period if there are no damages.
    
    Rental Agreement: A formal rental agreement (contrato de arrendamiento) will need to be signed, outlining the terms of the lease, including duration, rent amount, and responsibilities.
    
    Additional Documentation: Depending on the property or landlord, additional documents might be required, such as a bank statement or a Guarantor if you are a foreigner without a local income.
    
    Always review the specific terms of the rental agreement and ensure that you understand your rights and responsibilities as a tenant under Spanish law.
    
    Sources:
    Content generated using the LLM knowledge
```
```python
    The current owner of a flat knows an issues and he didnt told me during the selling. What I can do?
    
    
    provider='llm_knowledge' user_query='The current owner of a flat knows an issues and he didnt told me during the selling. What I can do?'
    Rephrased for clarity and grammatical accuracy.
    English
    The question is in English.
    loop need:
    True
    content='What can I do if the current owner of a flat knows about an issue and did not disclose it during the sale?' additional_kwargs={} response_metadata={}
    
    Response:
    In Spain, the seller has a legal obligation to disclose any known defects or issues related to the property that could affect its use or value. If you discover that the current owner of a flat knew about an issue and did not inform you before the sale, you have several options to consider:
    
    Negotiate with the Seller: Contact the seller and discuss the issue. Sometimes, a resolution can be reached amicably, such as a reduction in the sale price or an agreement to cover the repair costs.
    
    Document Everything: Keep detailed records of all communications and any evidence of the issue. This documentation will be crucial if you decide to pursue legal action.
    
    Legal Action: If the seller refuses to take responsibility and the issue is significant, you may consider filing a legal claim for breach of contract or misrepresentation. Under Spanish law, you may be entitled to compensation or even the annulment of the sale, depending on the severity of the issue and its impact on the property's habitability or value.
    
    Consult a Lawyer: It is advisable to consult with a lawyer who specializes in real estate law in Spain. They can provide you with tailored advice based on your specific situation and help you navigate the legal process.
    
    Contact Consumer Protection Authorities: If you believe your rights as a consumer have been violated, you can also reach out to consumer protection agencies in Spain for assistance.
    
    Acting quickly is important, as there are time limits for taking legal action (prescription periods), which vary depending on the nature of the claim.
    
    Sources:
    Content generated using the LLM knowledge
```
```python
    list 5 porperties close to schools in A Coruña
    
    provider='idealista' user_query='list 5 properties close to schools in A Coruña'
    The question specifies a location (A Coruña) and the type of property (for sale), which makes it easier to search for specific listings. It also clarifies that the properties should be near schools.
    English
    The question is in English, and the response is structured to match that language.
    loop need2:
    True
    content='List 5 properties for sale near schools in A Coruña.' additional_kwargs={} response_metadata={} 
    list 5 porperties close to schools in A Coruña
    
    
    Response:
    Here are five properties for sale in A Coruña that are likely to be close to schools, based on the general availability of amenities in the area:
    
    Property in A Coruña City Centre
    
    Price: €50,600
    Details: 1 bed, 114 m², 6th floor without lift.
    Description: This property is located in the city centre, providing easy access to various local services, including schools. It has good public transport connections with several bus stops nearby.
    Link: View Listing
    Renovated Apartment Near Nautical School
    
    Price: €550/month (for rent)
    Details: 2 bedrooms, 2 full bathrooms, living room, and laundry room.
    Description: This luxury apartment is in a newly built building and includes a garage space. It is located in an area close to educational institutions.
    Link: View Listing
    2-Bedroom Apartment in Corcubión
    
    Price: €170,000
    Details: 2 bedrooms, 110 m², 1st floor without lift.
    Description: This exterior apartment has the possibility of a third bedroom and is close to schools and other amenities.
    Link: View Listing
    Bright Exterior Property
    
    Price: €660/month (for rent)
    Details: 3 bedrooms, living room, furnished kitchen.
    Description: This renovated apartment is sunny and includes community fees in the price. Located in a well-connected area, it is close to schools and other services.
    Link: View Listing
    Modern Apartment with Garage Space
    
    Price: €550/month (for rent)
    Details: 2 bedrooms, large living room-kitchen.
    Description: This completely renovated apartment is bright and spacious, with all appliances included, making it ideal for families. It's also near schools and public transport.
    Link: View Listing
    For the most accurate listings and specific distances to schools, I recommend checking the properties on Idealista directly.
    
    Sources:
    website content
    http://idealista.es [{'property': [{'text': 'And for greater comfort, they include a garage space, with pre-installation for electric car. Property for sale, A Coruña, A Coruña, Spain: 861 houses and flats from 50,600 euros in Spain. Property listings direct from owners and real estate agents on idealista, the leading real estate marketplace in Spain.', 'metadata': {'path': 'https://www.idealista.com/en/geo/venta-viviendas/a-coruna-1/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': '1,145 listings of houses and flats for sale in A Coruña, Spain, from 50,600 euros. ... Property for sale in A Coruña, Spain: 1,145 houses and flats Buy; Rent; New homes; Filter List Map. New listings by email ... With all the nearby services, schools, bus lines, hospitals and supermarkets! . This UNIQ Renovated. Contact Call View phone', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/a-coruna-a-coruna/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': '50,600 €. 1 bed. 113 m² 6th floor without lift. 114 m² flat for sale in A Coruña. Located in the city centre, this property offers easy access by road to the AC-14, among others. It is also well connected by public transport, with several bus stops just a few metres away.', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/a-coruna-a-coruna/?ordenado-por=precios-asc', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': 'An exterior property, very bright and completely renovated to move into; It has 70 m2 built, distributed in two bedrooms, a bathroom and a large living room-kitchen with all the appliances equipped. FOR RENT LUXURY APARTMENT IN A NEWLY BUILT BUILDING WITH GARAGE SPACE AND STORAGE ROOM EXTERIOR - UNFURNISHED AREA: NAUTICAL SCHOOL - A CORUÑA CHARACTERISTICS: •2 bedrooms, 2 full bathrooms, plus living room, laundry room/drying rack, garage space and storage room that the elevator reaches. For rent apartment, €550, three bedrooms, living room, furnished kitchen, sunny, renovated, parquet, community fees included in the price, located in the Zalaeta area, ref. For rent apartment, €660, fully furnished, sunny, parquet, elevator, heating, storage room, garage, community fees included in the price, located in Plaza España area, ref.', 'metadata': {'path': 'https://www.idealista.com/en/geo/alquiler-viviendas/a-coruna-1/', 'relevance_score': 0.8}}]}]
    http://idealista.es [{'property': [{'text': 'Flat in calle Ramón Caamaño, 17, Corcubion. 170,000 € Parking included 176,000 € 3%. 2 bed. 110 m² 1st floor exterior without lift. 2 bedroom apartment with the possibility of a third bedroom, 2 bathrooms, one with a whirlpool tub and the other with a shower with a glass block screen. Floating flooring.', 'metadata': {'path': 'https://www.idealista.com/en/venta-viviendas/corcubion-a-coruna/', 'relevance_score': 0.8}}]}]
```

**🔗 Start using Idealisto today and take control of your real estate journey!**
