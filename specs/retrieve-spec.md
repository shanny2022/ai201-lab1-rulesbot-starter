# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"game"` | `str` | The game name this chunk came from |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Query approach

*Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?*

```
[I will use _collection.query() to search the stored rulebook chunks by semantic similarity. I will pass the user’s question as a single query and request up to n_results chunks. The query will compare the meaning of the question against the embeddings stored in ChromaDB and return the closest matching chunks first. I will request the documents, metadata, and distances so the app can return the chunk text, the game name, and the similarity score.]
```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

```
[Each item returned by retrieve() will be a dictionary with exactly three keys: "text", "game", and "distance".

Example:

{
    "text": "When a 7 is rolled, no players receive resources. The robber is activated...",
    "game": "Catan",
    "distance": 0.142
}

The "text" value comes from results["documents"][0]. The "game" value comes from the chunk metadata in results["metadatas"][0]. The "distance" value comes from results["distances"][0].]
```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```
[_collection.query() returns nested lists because it can handle more than one query at the same time. Since this app sends only one user question at a time, the actual results are inside index [0]. That means I need to use results["documents"][0], results["metadatas"][0], and results["distances"][0] to get the real list of returned chunks for the single query.]
```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```
[I will return all n_results instead of filtering by a strict distance threshold for now. This keeps the behavior simple and makes sure the generation step has enough context to work with. The tradeoff is that some weak matches may be included if the query is vague or unrelated to the rulebooks. A threshold could reduce irrelevant context, but it could also accidentally remove useful chunks if the distance scores are higher than expected. I will use the printed distance scores during testing to judge whether a threshold is needed later.]
```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```
[If the collection is empty, the function should return an empty list. If the query matches no chunks well, the function should still return the closest results, but the distance scores will likely be high, which means the results may not be reliable. If the query matches chunks from multiple games, that is acceptable because some questions, such as “How do you win?”, can apply to many games. The generation step can use the retrieved game names and chunk text to answer carefully or ask for clarification if needed.]
```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```
Query: [your test query]
Top result game: [game name]
Distance score: [score]
Does it make sense? [yes / no / explain]
```

**One thing about the query results that surprised you:**

```
[your answer here]
```
