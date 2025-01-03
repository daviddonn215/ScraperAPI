
# Building an Advanced Web Scraper for Extracting Contact Information from Google Search

Extracting leads and contact information is a critical aspect of marketing and market research. While web scraping has been around for years, the introduction of GPT-based retrievers has significantly simplified analyzing websites for specific information, even when HTML tags are inconsistent across pages.

In this article, we will build a scraper using LangChain, a user-friendly framework for Retrieval-Augmented Generation (RAG). This scraper will extract contact information from websites by leveraging GPT models. The process involves:

1. Fetching web links from Google Search using Serper API.
2. Loading web pages with AsyncHTMLLoader and parsing HTML using BeautifulSoupTransformer.
3. Converting text to embeddings using OpenAI's text-embedding-ada-002 model and storing them in Supabase.
4. Retrieving relevant chunks via RAG with Supabase retriever.
5. Extracting contact details in JSON format using a GPT-powered LLM.

---

## Step 1: Fetching Google Search Results with Serper.dev

To retrieve links from Google Search, we use [Serper.dev](http://serper.dev). While there are alternatives like Google Custom Search Engine, they often have strict rate limits and are more complex to set up. Serper.dev offers 2,500 free credits initially, which is sufficient for this use case, and its paid plans are cost-effective compared to competitors.

To get started:
1. Create a Serper.dev account.
2. Navigate to the dashboard and select the "API Key" option in the top-right corner to reveal your API key.

Serper.dev is ideal for extracting organic search results—unpaid listings that are most relevant to a user’s query. For our scraper, we will use advanced search filters, such as the `-inurl` keyword, to exclude specific sites from the results.

---

**Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. Start your free trial today! 👉 [https://www.scraperapi.com/?fp_ref=coupons](https://www.scraperapi.com/?fp_ref=coupons)**

---

## Step 2: Parsing HTML with AsyncHTMLLoader and BeautifulSoupTransformer

Once we have the URLs, we use LangChain's `AsyncHTMLLoader` to fetch the web pages. This tool supports most modern web pages, and we configure it to use the `html5lib` parser for better compatibility.

Next, we parse the loaded HTML with `BeautifulSoupTransformer`. Contact details are typically located in the header or footer sections of a webpage, so we extract these sections using the relevant tags.

To ensure efficient processing, we split the extracted text into chunks of 1,000 tokens using `RecursiveCharacterTextSplitter`, with no overlap between chunks.

---

## Step 3: Storing Embeddings in Supabase

We store the extracted text as embeddings in Supabase, a database platform that uses PostgreSQL with the `pgvector` extension for handling vector data.

### Database Setup
Run the following SQL migration in Supabase's SQL editor to set up the database:

```sql
create extension if not exists vector;

-- Create a table to store all the documents
create table
  documents (
    id uuid primary key,
    content text, -- corresponds to Document.pageContent
    metadata jsonb, -- corresponds to Document.metadata
    embedding vector (1536) -- 1536 works for OpenAI embeddings, change if needed
  );

-- Create a function to search for documents using cosine distance
create function match_documents (
  query_embedding vector (1536),
  filter jsonb default '{}'
) returns table (
  id uuid,
  content text,
  metadata jsonb,
  similarity float
) language plpgsql as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding;
end;
$$;
```

This schema allows us to store embeddings and retrieve the most relevant documents using cosine similarity.

### Integration with LangChain
Using the `match_documents` SQL function, embeddings are stored in the `documents` table. Supabase then acts as both a vector store and a retriever for relevant content.

For detailed documentation on using Supabase with LangChain, refer to the [official guide](https://python.langchain.com/docs/integrations/vectorstores/supabase).

---

## Step 4: Designing the Prompt for Contact Information Extraction

To extract contact information (e.g., phone numbers, email addresses), we design a structured prompt for the GPT model. The prompt includes:

1. **Model Personality:** Define the model's purpose.
2. **Task Description:** Specify the desired output format, such as JSON schema.
3. **Guardrails:** Prevent hallucination by instructing the model to return `None` if no contact information is found.

We use OpenAI's `text-embedding-ada-002` for embeddings and GPT-3.5-turbo as the LLM. The embeddings guide the retrieval process, while the LLM converts relevant text chunks into structured JSON.

---

## Step 5: Summing It Up in a Scraper Function

The final scraper function integrates all the steps:
1. Fetches Google Search results using Serper.dev.
2. Parses HTML with AsyncHTMLLoader and BeautifulSoupTransformer.
3. Stores and retrieves embeddings via Supabase.
4. Extracts contact details using a GPT-powered RAG process.

To prevent overlap between URLs, we clear old records from the `documents` table after each query. This ensures clean embeddings and minimizes storage usage.

The output is delivered in the following JSON format:

```json
{
  "email": "example@example.com",
  "phone": "+1234567890"
}
```

---

## Future Scope

With GPT-based scrapers, handling the dynamic and varied structure of web content becomes more manageable. Future enhancements may include:
- Automating CAPTCHA resolution.
- Expanding the scraper to handle multi-language support.
- Improving the accuracy of contact information extraction with advanced prompts.

---

## Helpful Links

- [LangChain Documentation](https://python.langchain.com/)
- [Supabase VectorStore Integration](https://python.langchain.com/docs/integrations/vectorstores/supabase)
- [Serper.dev](http://serper.dev)

---
