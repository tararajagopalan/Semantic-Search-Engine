
# Universal Semantic Search

A Flask web app that searches across PDFs, text files, images, and videos using natural language. Type a query, optionally filter by file type, and the app returns the 5 most semantically similar files, each with an AI-generated summary and a similarity score.

## How it works

Images and videos can't be compared to a text query directly, so the pipeline first converts every file into text, then searches over those text representations.

1. **Summarize every file with Gemini.** Each file is sent to Gemini 2.5 Flash with a prompt tailored to its type:
   - PDFs and text files: a document summary
   - Images: a concise caption (under 30 words)
   - Videos (under 20 MB): a 3-sentence summary
2. **Embed the summaries.** Each summary is converted into a 768-dimension vector using pymilvus's default embedding function.
3. **Store in a vector database.** Summaries, embeddings, file paths, and file extensions are loaded into a Milvus collection (`project_collection`).
4. **Search.** The user's query is embedded the same way. Milvus returns the 5 nearest files by vector distance, optionally filtered by file extension.
5. **Serve results.** Flask renders each result as a link to the original file, along with its summary and distance score. Lower scores mean closer matches.

## Tech stack

Python, Flask, Milvus (pymilvus), Google Gemini API (gemini-2.5-flash), HTML/CSS. Developed in Google Colab.

## Project structure

```
Flask/
├── Server.ipynb                     # Flask app: routes, search, result rendering
├── Data Processing/
│   ├── GeneratingTextSummaries.ipynb   # Gemini summarization helpers
│   ├── projectdatatable_script.ipynb   # Milvus collection setup and data loading
│   └── project_querySearch.ipynb       # Query testing
├── Static/                          # CSS plus sample images, PDFs, text, and videos
└── Templates/                       # main, results, and per-file-type HTML pages
```

## Example

Query: "how to make brownies" (all file formats)

| Rank | File | Distance |
|---|---|---|
| 1 | ClayMoldMaking.pdf | 1.41 |
| 2 | EasyRecipes.pdf | 1.77 |
| 3 | dance.mp4 | 1.90 |

## Known limitations and next steps

In the example above, a clay mold-making lesson plan outranked a recipe collection for a baking query. Likely causes:

- **Embedding model strength.** The default pymilvus embedding function is lightweight. A stronger embedding model would likely capture meaning more accurately.
- **One vector per file.** Each file is represented by a single summary. Long documents covering several topics may be diluted into one vague vector. Chunking documents into sections and embedding each chunk would improve precision.
- **No keyword signal.** Pure vector search can miss exact word matches like "recipe." A hybrid search combining vector similarity with keyword matching would help.

## Walkthrough

`QueryingProjectPresentation.pdf` walks through each step of the build with code screenshots.
