# Legal Contract Analyzer - RAG System with Local LLM

We built a system for reading and analyzing legal contracts using a Retrieval-Augmented Generation (RAG) setup powered by a local Large Language Model.

Because of computational limits, all code runs inside a Colab notebook. Here's the breakdown cell by cell:

---

## Code Pipeline

**Cell 1:**  
Install core LangChain packages:
```bash
!pip install -q langchain langchain-community
```

**Cell 2:**  
Install remaining packages:
```bash
!pip install -q streamlit pdfplumber transformers sentence-transformers langchain faiss-cpu
!npm install -g localtunnel
```

**Cell 3:**  
Write and save `app.py` - main Streamlit app.
Main features:

* PDF contract text extraction
* Named Entity Recognition with HuggingFace
* Semantic clause detection using sentence embeddings
* Basic risk assessment based on keywords
* Retrieval-Augmented QA generation for simple explanations
* Clean UI with Streamlit

**Cell 4:** 
Run Streamlit server in the background:
```bash
!nohup streamlit run app.py &
```

**Cell 5:** 
Spin up a LocalTunnel to expose the app publicly:
```bash
!nohup npx localtunnel --port 8501 > lt.log 2>&1 &
```

**Cell 6:** 
Print the public LocalTunnel URL:
```bash
!tail -n 10 lt.log
```

**Cell 7:** 
Manual log inspection (sometimes the url doesn't display - execute this cell in those cases):
```bash
!cat lt.log
```

**Cell 8:** 
Get the webapp password:
```bash
!curl https://loca.lt/mytunnelpassword
```

## Techincal Details:

**PDF Extraction:**
We use pdfplumber to extract the text from the uploaded PDF file, page by page.

**Named Entity Recognition:**
We run a HuggingFace BERT NER model on the extracted text to tag people, organizations, locations, etc. We chunk long text into smaller pieces to avoid overloading the model.

**Semantic Clause Detection:**
We define templates for common legal clauses (Termination, Confidentiality, etc.), embed both the contract paragraphs and the templates using all-MiniLM-L6-v2, then find the most similar matches using cosine similarity. If the score is high enough (>0.5), we treat it as a detected clause.

**Risk Assessment:**
We do a simple keyword scan on each detected clause. High-risk keywords (like "indemnify" or "liable") bump the score up faster. Based on the count, we classify each clause as Low, Medium, or High risk.

**RAG:**
We turn the detected clauses into mini-documents, embed them, and build a FAISS vectorstore. When a user asks for an explanation, we pull the relevant clause and feed it into a tiny HuggingFace LLM running locally to generate a simple, layman's explanation.

**Frontend:**
Streamlit handles everything on the UI side: upload the contract, show named entities, show detected clauses + risk scores, and display the generated explanations.

**Deployment:**
Because we're in Colab (due to local computational limitations), we launch Streamlit in the background and then expose it with LocalTunnel so anyone can access the app via a public URL.
