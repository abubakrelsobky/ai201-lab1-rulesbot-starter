<<<<<<< HEAD
# Planning

Planning provenance: this plan was drafted before implementation as the design baseline, then updated after implementation to reflect the final retrieval behavior and evaluation findings.

## Corpus

I used all 10 text files in `data/raw` as the source set. The files are short forum threads, a news-style article, and professor-rating snippets, so they behave more like conversational evidence than long formal documents.

## Cleaning

The cleaning step removes obvious non-content noise:

- promotional blocks like `Promoted` and `Shop Now`
- ad and embedded-player artifacts
- avatar labels and repeated UI text
- blank lines and simple timestamp fragments

I kept the actual comment text and article text because those are the useful signals for retrieval.

## Chunking strategy

I chunked the cleaned text by word windows using:

- chunk size: 120 words
- overlap: 30 words

Why this works here:

- The documents are short, so large chunks would blur unrelated comments together.
- Review-style and forum-style text often needs the neighboring sentence for context, so overlap keeps the key claim from falling on a chunk boundary.
- 120 words is large enough to preserve local context while still keeping chunks narrow enough for precise retrieval.

## Retrieval design

I used a TF-IDF vector index with cosine similarity.

Why this is a reasonable first pass:

- It is simple and reproducible.
- It runs entirely offline.
- It is good enough for this small, domain-specific corpus where query terms usually appear in the relevant text.

If the project were expanded, I would replace this with a stronger embedding model and likely a managed vector database.

## Response generation

The response step is grounded in the retrieved chunks only. The notebook extracts the most relevant sentences from the top results and prints the source file and chunk label for each answer.

## Evaluation plan

I designed five questions that cover:

- beginner-friendliness of CS50
- the value of the CS50 certificate
- the overall Harvard CS department experience
- professor approachability
- a harder numeric question about faculty industry involvement

The last question is intentionally more difficult because it asks the system to turn a sentence into an approximate percentage.
=======
# RulesBot — Planning Doc

Use this file to record your design decisions as you work through the lab.
There are no wrong answers — write enough that you could explain your reasoning to another group.

---

## Chunking Strategy

**Chunk size:**


**Overlap:**


**Why this strategy fits rule book text:**


---

## Retrieval Observations

After implementing retrieval, try these test queries and record what comes back:

| Query | Top result game | Does it make sense? |
|-------|----------------|---------------------|
| "How do you win?" | | |
| "What happens when you roll a 7?" | | |
| "Can two players share a route?" | | |

**Anything surprising?**


---

## Response Quality

After implementing generation, try 2–3 questions and assess the answers:

| Query | Answer accurate? | Properly grounded? | Cited the right game? |
|-------|-----------------|-------------------|----------------------|
| | | | |
| | | | |

**What would you change about the prompt to improve grounding?**

>>>>>>> cab4525908eca0e98b4fef8902fe7110f3ce5f72
