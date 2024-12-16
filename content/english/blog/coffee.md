---
title: "How I fixed my coffee machine using a RAG System"
meta_title: "How I fixed my coffee machine using a RAG System"
description: "How I fixed my coffee machine using a RAG System"
date: 2024-11-04T05:00:00Z
image: "/images/blog/coffee/coffee.webp"
categories: ["AI"]
author: "Fernando Souto"
tags: ["RAG", "Graph Database", "Neo4j","LLM", "Langchain"]
draft: false
---

When my coffee machine decided to quit on me, going through the manual was just painful and honestly, a waste of time. So, instead of giving up, I tried something different: I used a Retrieval-Augmented Generation (RAG) system with a Large Language Model (LLM) to figure it out.

I started by setting up a retrieval system from the manual — this helped me organize the info so the LLM could understand what was what. From there, I asked the LLM some specific questions. The answers were spot on for troubleshooting and saved me a ton of time and frustration!

Long story short, I got my coffee machine working without the guesswork. This just goes to show how RAG systems can make your everyday life easier. Excited to see where else this tech can help!

## Key highlights of the PoC

**— Knowledge Base:** Powered by Neo4j graph database

**— Toolkit:** Langchain QA chain

**— Model:** Azure OpenAI 4o

{{< youtube agxqDfRMQrM >}}

## How it works

### Step 1: Install dependencies

First, load up all the necessary libraries in your environment:
```python
    pip install neo4j langchain-openai langchain langchain-community langchain-huggingface pandas tabulate
```

### Step 2: Initialize your Neo4j database connection

Connect to Neo4j by initializing the database with your credentials. Make sure your credentials are stored securely as environment variables.
```python
    from neo4j_graph import Neo4jGraph
    
    enhanced_graph = Neo4jGraph(
        url=os.environ["NEO4J_INSTANCE"],
        username=os.environ["NEO4J_USER"],
        password=os.environ["NEO4J_PASS"],
        enhanced_schema=True
```

### Step 3: Initialize the language model

Now, set up the Azure OpenAI model using your own credentials:
```python
    from azure_openai import AzureChatOpenAI
    
    model = AzureChatOpenAI(
        azure_deployment=os.environ['AZURE_OPENAI_DEPLOYMENT'],
        model_version="2024-05-13",
        api_version="2024-02-01",
        temperature=0
    )
```

### Step 4: Create the Q&A chain with Neo4j

This is where the magic happens! You’ll build a question-answering chain by setting up embeddings, creating a vector store in Neo4j, and setting up a retriever to pull relevant answers.

```python
    from langchain_huggingface import HuggingFaceEmbeddings
    from langchain_qa_chain import Neo4jVector, RetrievalQAWithSourcesChain
    
    embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
    store = Neo4jVector.from_existing_index(
        embeddings,
        url=os.environ['NEO4J_INSTANCE'],
        username=os.environ['NEO4J_USER'],
        password=os.environ['NEO4J_PASS'],
        index_name="vector",
        keyword_index_name="text_index",
        search_type="hybrid"
    )
    retriever = store.as_retriever()
    sim_chain = RetrievalQAWithSourcesChain.from_chain_type(
        model, 
        chain_type="stuff", 
        retriever=retriever,
        verbose=False,
        return_source_documents=True
    )
```

### Step 5: Ask your question

Let’s put it to the test with a sample question! Query the model and get results back in seconds:
```python
    result = sim_chain("The milk has big bubbles")
```

### Step 6: Pretty-print the results in a table

Make your output visually appealing and organized. This formatting trick displays the answer and sources in a neatly structured table with page context:
```python
    import pandas as pd
    import json
    from tabulate import tabulate
    from IPython.display import display, Markdown
    
    data = json.dumps([{"page_content": doc.page_content, "metadata": doc.metadata} for doc in result["source_documents"]])
    df = pd.json_normalize(json.loads(data))
    df = df.drop(['metadata.position', 'metadata.content_offset', 'metadata.source', 'metadata.fileName', 'metadata.length'], axis=1)
    df.rename(columns={'page_content': 'Content', 'metadata.page_number': 'Page'}, inplace=True)
    display(Markdown('# Response:\n' + result["answer"]))
    display(Markdown('# Sources:\n'))
    display(Markdown(tabulate(df, headers='keys', tablefmt='github', showindex='never')))
```

![Results of the experiment](/images/blog/coffee/results.png)

### Finished!

With this setup, you’re now ready to ask questions, retrieve intelligent answers from Neo4j, and display them beautifully. This integration not only makes your knowledge base powerful but also easy to navigate and interact with.
