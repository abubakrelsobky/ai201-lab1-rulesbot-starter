<<<<<<< HEAD
# The Unofficial Guide

This project builds a small RAG-style search assistant over 10 raw text files in `data/raw`.

## Domain and document sources

Domain: unofficial student-shared knowledge about Harvard CS experience, classes, and faculty.

Document sources used (file names in `data/raw`):

- `AI Startup.txt` (forum-style discussion text captured as raw text)
- `Harvard CS Professors.txt` (professor ratings snapshot)
- `Harvard CS Rank.txt` (discussion-thread ranking content)
- `Harvard_CS_Experience.txt` (student experience thread)
- `Harvard_CS_Faculty.txt` (article-like faculty/industry text)
- `Harvard_CS.txt` (empty file in source folder)
- `Harvard_CS50.txt` (CS50 value discussion)
- `Harvard_Experience.txt` (undergraduate experience text)
- `Harvard_Stanford_Yale.txt` (cross-school choice thread)
- `Harvrad_CS_Courses.txt` (course discussion text)

Most raw files were exported from web/forum pages into plain text. The raw text includes platform noise (ads, avatars, player UI text), which is cleaned during preprocessing.

## What it does

- Loads the raw Harvard CS documents.
- Cleans obvious page noise such as ads, avatar labels, and video-player artifacts.
- Chunks the cleaned text into overlapping windows.
- Builds a lightweight TF-IDF vector index.
- Answers questions with retrieved evidence and source attribution.
- Evaluates the system on five test questions.

## How to run

Open `unofficial_guide.ipynb` and run the cells from top to bottom.

The notebook is self-contained and uses only the Python standard library.

## Data notes

The folder contains 10 source files. One of them is empty, so it is recorded as a source file but contributes no chunks to the index.

## Embedding approach

This submission uses TF-IDF vectors as the embedding layer. That choice keeps the project portable and easy to run without external APIs or model downloads.

If I were choosing an embedding stack for a production system, I would compare:

- Hosted embeddings such as OpenAI `text-embedding-3-small` for strong retrieval quality and simple operations.
- Local sentence embeddings such as `all-MiniLM-L6-v2` for lower latency and better privacy.
- Multilingual models if the corpus might grow beyond English.

The tradeoff is usually cost and simplicity versus retrieval quality, context coverage, and model maintenance.

## Chunking strategy and reasoning

- Chunk size: 120 words
- Overlap: 30 words

Why this fits the documents:

- The corpus is mostly short forum/review style text, so smaller windows preserve local meaning better than very large chunks.
- Overlap helps when a key claim is split across boundaries between two nearby comments/sentences.
- 120/30 gave a practical balance between retrieval precision and context completeness for this dataset.

## Sample chunks (5 labeled examples)

1. `AI Startup.txt [chunk 1]`:
   `How is the CS department? I was recently admitted to the class of 2022, and I’m still not sure what I will concentrate in. I might consider going into CS, so I was wondering how the CS folks feel about their experience...`
2. `AI Startup.txt [chunk 2]`:
   `...in the midst of huge growth (hiring new faculty and building in Allston). It's really easy to get to know professors and do really cool research. Since there aren't that many graduate students in CS...`
3. `AI Startup.txt [chunk 6]`:
   `...Jelani is one of the worlds leading experts on streaming algorithms and Madhu Sudan is a big name in coding theory and probabilistic proofs. Profs, for the most part, are ridiculously approachable...`
4. `AI Startup.txt [chunk 11]`:
   `...borders between cultures, development. I really need a mentor help in order to master this process and get a perfect result...`
5. `Harvard CS Rank.txt [chunk 6]`:
   `...if a student graduates number one with a double major in math and CS at a big state program they can and will get into grad school... Its not all about the Ivy League in CS...`

## Retrieval test results

### Query 1

Query: `Is CS50 beginner friendly?`

Top returned chunks:

1. `Harvard_CS50.txt [chunk 3]` (score: 0.254)
2. `Harvrad_CS_Courses.txt [chunk 3]` (score: 0.240)
3. `Harvard_CS50.txt [chunk 2]` (score: 0.230)

Why relevant: these chunks directly include statements about CS50 being beginner-intended but still difficult for some beginners.

### Query 2

Query: `Are Harvard CS professors approachable?`

Top returned chunks:

1. `Harvard_CS_Experience.txt [chunk 2]` (score: 0.275)
2. `AI Startup.txt [chunk 2]` (score: 0.262)
3. `Harvard_CS_Experience.txt [chunk 1]` (score: 0.176)

Why relevant: these chunks mention getting to know professors, undergrad access, and direct research interactions with faculty.

### Query 3

Query: `How strong is Harvard CS for undergraduate research?`

Top returned chunks:

1. `Harvard_CS_Experience.txt [chunk 2]` (score: 0.158)
2. `Harvard_Stanford_Yale.txt [chunk 2]` (score: 0.153)
3. `AI Startup.txt [chunk 2]` (score: 0.146)

## How grounded generation is enforced

Grounding is enforced in the pipeline structure:

- Retrieval first: answers are generated only after top-k chunk retrieval.
- Context-bounded synthesis: the answer function extracts candidate sentences from retrieved chunks only.
- Citation requirement: output always prints source labels like `filename [chunk N]`.
- No external knowledge hook: the notebook does not call a general-purpose model API.

## Response generation

The notebook uses a grounded, citation-first answer synthesizer over the retrieved chunks. The answer is constrained to the text that was retrieved, and every answer includes source labels.

That keeps the demo reliable in an offline environment and makes the grounding easy to inspect.

## Example responses with attribution

Example 1:

- Query: `Is CS50 beginner friendly?`
- Response excerpt: `Based on the retrieved documents, ... is it beginner friendly? ... it can get really hard for new programmers.`
- Sources shown in output: `Harvard_CS50.txt [chunk 3]`, `Harvard_CS50.txt [chunk 2]`

Example 2:

- Query: `Approximately what percentage of Harvard CS faculty hold or have held industry positions?`
- Response excerpt: `... 12 of 43 SEAS tenure-track professors ... hold or have held industry positions ...`
- Sources shown in output: `Harvard_CS_Faculty.txt [chunk 1]`, `Harvard_CS_Faculty.txt [chunk 2]`

Out-of-scope example:

- Query: `Which Harvard dorm has the best laundry rooms in 2026?`
- Refusal behavior expected by system design: the system should refuse with `I could not find enough evidence in the retrieved documents to answer that clearly.` when retrieval confidence is insufficient.
- Current limitation: lexical overlap can still return weakly related chunks; this is tracked as a known improvement area for stronger refusal gating.

## Query interface

Interface type: notebook function interface (`ask(query, top_k=3)`).

Input field(s):

- `query` (string)
- optional `top_k` (integer, defaults to 3)

Output field(s):

- grounded answer text
- `Sources` list with chunk labels
- top retrieved chunk list with similarity scores

Sample interaction transcript:

- Input: `ask('Are Harvard CS professors approachable?')`
- Output summary:
  - Answer states professors are approachable and accessible for research conversations.
  - Sources: `Harvard_CS_Experience.txt [chunk 2]`, `Harvard_CS_Experience.txt [chunk 1]`
  - Top retrieved chunks include `Harvard_CS_Experience.txt` and `AI Startup.txt` with scores.

## Evaluation report

All five evaluation questions, expected answers, system outputs, retrieved chunks, and judgments are in `evaluation.md`.

Failure case included:

- Question about converting `12 of 43` into a percentage.
- Retrieval succeeds, but generation is partially accurate because it repeats the ratio instead of calculating the approximate percentage.

## Spec reflection

- How the spec helped: it forced explicit design choices (chunking, retrieval, evaluation) and prevented a vague demo-only implementation.
- Where implementation diverged: instead of a hosted embedding API + LLM generation, I used offline TF-IDF retrieval and extractive grounded synthesis to keep the project reproducible in a no-dependency environment.

## AI usage

1. I used AI assistance to scaffold the initial notebook pipeline (preprocessing, chunking, retrieval, and answer function) and then revised logic to include source-title text in each chunk for better ranking.
2. I used AI assistance to draft evaluation/report structure and then overrode generic claims with concrete chunk IDs, scores, and one honest failure case from actual runs.
=======
# 🎲 RulesBot

> A board game rules assistant — because "just read the rulebook" isn't always helpful at 11pm on game night.

RulesBot answers natural language questions about board game rules using a RAG (Retrieval-Augmented Generation) pipeline. Ask it anything: it retrieves relevant rule passages and generates an answer grounded in the actual text.

**This is a starter repo.** The UI and infrastructure are built. The retrieval and generation pipeline is yours to implement.

---

## Getting Started

### 1. Fork and clone

Fork this repo, then clone your fork locally.

### 2. Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate      # Mac/Linux
# or: .venv\Scripts\activate   # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

> **Note:** `sentence-transformers` will download the embedding model (~80MB) on first run. This only happens once — it's cached locally afterward.

### 4. Add your Groq API key

```bash
cp .env.example .env
```

Open `.env` and replace `your_key_here` with your key from [console.groq.com](https://console.groq.com). No credit card required.

### 5. Run the app

```bash
python app.py
```

RulesBot will start and open in your browser. Before you implement the retrieval pipeline, it will load and display the UI but won't be able to answer questions.

---

## Project Structure

```
ai201-lab1-rulesbot-starter/
├── app.py              # Gradio UI and startup logic — fully built
├── config.py           # Settings (models, paths, retrieval params) — fully built
├── ingest.py           # Document loading + chunking — TODO: chunk_document()
├── retriever.py        # Vector store + semantic search — TODO: embed_and_store(), retrieve()
├── generator.py        # LLM response generation — TODO: generate_response()
├── docs/               # Board game rule documents (pre-loaded)
│   ├── catan.txt
│   ├── clue.txt
│   ├── codenames.txt
│   ├── monopoly.txt
│   ├── pandemic.txt
│   ├── risk.txt
│   ├── ticket_to_ride.txt
│   └── uno.txt
├── specs/              # Design documents — start here before writing any code
│   ├── system-design.md         # Complete — read this first
│   ├── chunk-document-spec.md   # Partial — you complete before Milestone 1
│   ├── retrieve-spec.md         # Partial — you complete before Milestone 2
│   └── generate-response-spec.md # Partial — you complete before Milestone 3
└── planning.md         # Your observations and reflections — fill in as you go
```

## Where to Start

Before opening any `.py` file, read `specs/system-design.md`. It explains what's built, what's left for you, and why the technical decisions were made. Each milestone then begins by completing the corresponding spec file before writing code — that spec becomes the brief you hand to your AI tool when you're ready to implement.

---

## Re-ingesting After Changes

ChromaDB persists to disk in `./chroma_db`. If you change your chunking strategy and want to re-ingest, delete that folder and restart the app:

```bash
rm -rf chroma_db/   # Mac/Linux
# or: rmdir /s chroma_db   # Windows
python app.py
```

---

## Rule Books Included

| Game | File |
|------|------|
| Catan | `docs/catan.txt` |
| Clue | `docs/clue.txt` |
| Codenames | `docs/codenames.txt` |
| Monopoly | `docs/monopoly.txt` |
| Pandemic | `docs/pandemic.txt` |
| Risk | `docs/risk.txt` |
| Ticket to Ride | `docs/ticket_to_ride.txt` |
| Uno | `docs/uno.txt` |
>>>>>>> cab4525908eca0e98b4fef8902fe7110f3ce5f72
