The first stage trains the [**foundation model**](https://arxiv.org/pdf/2108.07258) — a model that processes large amounts of text data and extracts various patterns from it (like grammar constructs, associations, contextual meaning, etc.). More formally, the model learns statistical representations and relationships between data at this point. This training happens in a self-supervised way, meaning there are no explicit labels, and the task focuses on extracting structures from the data.

![[LLM.png]]


![[GenAI 1.png]]



These models, controlled by user prompts, are the engines behind conversational AI and chatbots. Because they generate text by predicting the most statistically likely next word, they are prone to "hallucinating"—creating plausible-sounding but factually incorrect information. Furthermore, they have no inherent memory of their training data, making it impossible for them to cite the sources of their information. They are useful for most general tasks, but they are unreliable for tasks that demand high accuracy and trustworthiness.

To solve this, various techniques, such as **retrieval-augmented generation (RAG)**, have been popularized in recent years. RAG combines the indexing of document embeddings in a specific domain—a technique from representational models—with the generative aspect. This grounds the model with specific, verifiable data. The system first retrieves relevant facts from a knowledge base and then instructs the model to generate its response based only on that provided information. This makes the output more accurate, with the ability to reference the exact source of the output.

Beyond generating text based on provided context, the next frontier is building LLM-powered **agents** capable of autonomous action. This pattern uses an autoregressive model as a reasoning engine to interpret a goal, create a plan, and execute it by using external tools. The model's task is to decide what to do next, whether it's calling an API, running a script, or querying a database. This enables the creation of more capable personal assistants or automated systems that can perform complex tasks.