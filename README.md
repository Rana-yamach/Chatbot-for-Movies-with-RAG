
# Movie RAG Chatbot with Gemini & LangChain
🎬 Movie Chatbot (Genre/Overview Recommendations)

This project is a Retrieval-Augmented Generation (RAG) chatbot developed for the **Akbank GenAI Bootcamp**. It provides answers to user questions about movies by retrieving information from a dedicated movie dataset.

The chatbot is built using LangChain and Google's Gemini Pro model, and it is deployed with a Gradio web interface.

## 1. Project Purpose

The primary goal of this project is to implement a complete RAG pipeline. This involves:
1.  Loading a document corpus (a movie dataset) into a vector store.
2.  Accepting a user query through a web interface.
3.  Retrieving the most relevant documents (movie info) from the vector store based on the query.
4.  Augmenting (pre-pending) the retrieved context to the user's original query.
5.  Passing the combined prompt to a Generative LLM (Gemini) to generate a final, context-aware answer.

## 2. Dataset Information

The chatbot uses the `Pablinho/movies-dataset` available on Hugging Face.

* **Content:** The dataset contains information on over 9,800 movies.
* **Preparation:** During the data loading phase, each movie is processed into a LangChain `Document` object. The `page_content` of each document (which is used for embedding) is a combination of the movie's
**Title, Release Date, Genres, Rating (Vote Average), and Overview**.
* **Metadata:** The movie's `Poster_Url` is stored separately in the document's metadata to be displayed in the final answer.

## 3. Solution Architecture

This project is built on a modern RAG stack using LangChain to orchestrate the different components.

* **Framework:** `LangChain`
* **Generation LLM:** Google `gemini-2.0-flash` (via `langchain-google-genai`) 
* **Embedding Model:** `sentence-transformers/all-mpnet-base-v2` (via `langchain-huggingface`) 
* **Vector Store:** `Chroma` (in-memory) 
* **Web Interface:** `Gradio` 

### Pipeline Flow:
1.  **Load:** The movie dataset is loaded from Hugging Face.
2.  **Chunk:** Documents are split into smaller, 500-token chunks using `RecursiveCharacterTextSplitter`.
3.  **Embed & Store:** All document chunks are embedded using `all-mpnet-base-v2` and indexed into a `Chroma` vector database.
4.  **Chat:** The RAG chain processes user input.
    * A `contextualize_q_chain` reformulates the user's question to be a standalone query based on chat history.
    * The `retriever` fetches the top 3 relevant movie chunks from `Chroma`.
    * A `qa_prompt` instructs the `gemini-2.0-flash` model to answer the question *only* using the provided movie context.
5.  **Serve:** The entire chain is served as an interactive chatbot using `Gradio`.

## 4. How to Run (Working Guide)

To run this project locally, follow these steps.

**1. Clone the Repository:**


```
git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
cd your-repo-name
```

**2. Create a Virtual Environment:**

```
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**3. Install Dependencies:**


A requirements.txt file is provided, containing all necessary packages


```
pip install -r requirements.txt
```

**4. Set Up Environment Variables:** 


Create a file named .env in the root of the project folder. Add your API keys to this file. The script uses python-dotenv to load them automatically.
```
GOOGLE_API_KEY="YOUR_GEMINI_API_KEY_HERE"
HF_TOKEN="YOUR_HUGGINGFACE_TOKEN_HERE"
```

**5. Run the Application:**


```
python app.py
This will start the Gradio server on your local machine.
```

## 5. Web Application & Guide


The chatbot is deployed as a public web application using Gradio and Hugging Face Spaces.


🚀 Live Demo Link: https://huggingface.co/spaces/sakurana/Chatbot-for-Movies-with-RAG

**Application Features (Obtained Results)**
The chatbot is designed to be a helpful movie expert whose knowledge is strictly limited to the provided context (Title, Overview, Genre, Rating, Date, Poster).

**You CAN:**

Ask for details about a specific movie.

* "What is 'Spider-Man: No Way Home' about?" 

* "Tell me about 'The Batman' (2022) including its rating and poster." 

Ask for recommendations based on Genre or Overview.

* "Recommend some action movies."

* "Any movies about space travel?"

**You CANNOT (By Design):**

Ask for recommendations based on Rating or Popularity.

* If you ask, "Recommend movies with a rating above 8," the bot will (correctly) respond that it cannot make comparisons based on ratings and will offer to help based on genre or overview instead.

Ask for information not in the dataset (e.g., actors, directors, box office).

* If you ask, "Who directed 'Inception'?", the bot will respond that it does not have that specific type of information.
