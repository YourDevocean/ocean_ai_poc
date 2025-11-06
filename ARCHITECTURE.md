# Ocean AI POC - Architecture Overview

## RAG Pipeline Architecture

The Ocean AI POC implements a Retrieval-Augmented Generation (RAG) system designed for ocean sustainability research. The system follows a two-phase architecture: **document ingestion** and **query processing**.

### How RAG Works

When a document is added to the system, it undergoes a sophisticated processing pipeline. The document is first parsed to extract its textual content, then **chunked** into smaller, semantically meaningful segments (typically 1000 characters with 200-character overlap). Each chunk is then sent to OpenAI's text-embedding-3-small model to generate high-dimensional vector embeddings that capture the semantic meaning of the text. These embeddings are stored in a PostgreSQL database enhanced with the pgvector extension, which enables efficient similarity search operations.

During query processing, the user's question follows a similar path. The question is sent to the same embedding model to generate a query vector. This query embedding is then compared against all stored document embeddings using cosine similarity. The system retrieves the most semantically similar chunks that exceed a minimum similarity threshold, providing contextually relevant information that is then fed to GPT-4o-mini along with the original question to generate a comprehensive, context-aware answer.

## Core Components

### Document Ingestion (`ingest.py`)

The ingestion pipeline is responsible for processing and storing documents in the knowledge base. Key functionalities include:

- **Document Processing**: Supports PDF and text files (.pdf, .txt, .md)
- **Text Extraction**: Uses PyPDF2 for PDF text extraction with fallback error handling
- **Intelligent Chunking**: Implements RecursiveCharacterTextSplitter from LangChain for semantic text segmentation
- **Metadata Extraction**: Automatically extracts document type, geographic focus, and topics from filenames
- **Embedding Generation**: Batch processes text chunks through OpenAI's embedding API for efficiency
- **Duplicate Detection**: Prevents re-ingestion of existing documents using filename and file size matching
- **Database Storage**: Stores document metadata, chunks, and embeddings in PostgreSQL with proper relational structure

The ingestion process handles documents through the `DocumentIngestor` class, which orchestrates the entire pipeline from file input to database storage. It supports both single-file and directory-based batch processing, making it suitable for both individual document uploads and bulk data ingestion.

### Query Processing (`rag_retriever.py`)

The retrieval system handles user queries and generates contextual responses through several key operations:

- **Query Embedding**: Converts natural language questions into vector representations using the same embedding model used for documents
- **Similarity Search**: Performs efficient vector similarity search using PostgreSQL's pgvector extension with configurable similarity thresholds
- **Context Assembly**: Aggregates retrieved chunks into coherent context while maintaining source attribution
- **Response Generation**: Sends assembled context and original question to GPT-4o-mini for final answer synthesis
- **Result Filtering**: Supports filtering by document type, geographic region, and topic categories
- **Source Attribution**: Tracks and returns source documents with similarity scores for transparency

The `OceanRAGRetriever` class implements a sophisticated retrieval strategy that balances relevance and diversity in the returned results. It includes configurable parameters for maximum results, similarity thresholds, and various content filters to ensure users receive the most relevant information for their specific queries.

## Data Flow

1. **Ingestion Phase**: Document → Text Extraction → Chunking → Embedding Generation → Database Storage
2. **Query Phase**: User Question → Query Embedding → Similarity Search → Context Retrieval → LLM Response Generation
3. **Response Delivery**: Generated Answer + Source Attribution + Metadata

## Technical Stack

- **Database**: PostgreSQL 14+ with pgvector extension for vector similarity search
- **Embeddings**: OpenAI text-embedding-3-small (1536 dimensions)
- **Language Model**: OpenAI GPT-4o-mini for response generation
- **Text Processing**: LangChain's RecursiveCharacterTextSplitter for intelligent chunking
- **Web Interface**: Streamlit for interactive user interface
- **Document Processing**: PyPDF2 for PDF text extraction

## Configuration and Deployment

The system uses YAML-based configuration (`config.yaml`) for managing API keys, database connections, and model parameters. The modular architecture allows for easy deployment in various environments, with support for both command-line and web-based interfaces through the Streamlit application (`app_streamlit.py`).