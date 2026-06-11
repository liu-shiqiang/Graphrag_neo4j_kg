# GraphRAG Neo4j Knowledge Graph

Methods for constructing knowledge graphs using GraphRAG with Neo4j backend and LLM-powered entity and relationship extraction.

## Overview

This project implements an automated knowledge graph construction pipeline that:

- **Extracts structured knowledge** from unstructured text documents using Large Language Models (LLMs)
- **Creates a knowledge graph** by identifying entities, relationships, and their properties
- **Stores the graph** in Neo4j for efficient querying and visualization
- **Specializes in biological domain** with domain-specific entity and relationship properties

The pipeline is particularly designed for processing biological literature and constructing knowledge graphs with biological entities (genes, proteins, cells, diseases, drugs, etc.) and their interactions.

## Features

- 🧬 **Domain-Specific Extraction**: Optimized for biological literature with gene, protein, cell type, disease, variant, marker, and drug entity types
- 🚀 **Parallel Processing**: Multi-threaded document processing for improved performance
- 🧠 **LLM-Powered**: Uses LLMs (via Ollama or OpenAI) for intelligent entity and relationship extraction
- 📊 **Neo4j Integration**: Direct integration with Neo4j graph database for persistent storage
- 🔄 **Graph Transformer**: Leverages LangChain's LLMGraphTransformer for knowledge extraction
- 📈 **Flexible Configuration**: Customizable entity and relationship properties for different domains

## Project Structure

```
Graphrag_neo4j_kg/
├── README.md                 # This file
├── main.py                   # Modular main entry point
├── main.ipynb                # Jupyter notebook for interactive exploration
├── extractkg.py              # Standalone knowledge extraction script
├── ms_graphrag.ipynb         # Microsoft GraphRAG reference notebook
├── ms_graphrag copy.ipynb    # Backup of reference notebook
├── src/                      # Module directory (planned)
│   ├── data_processing.py    # Text processing utilities
│   ├── llm_initializer.py    # LLM initialization
│   ├── graph_transformer.py  # Graph transformer setup
│   └── knowledge_extraction.py # Main extraction logic
└── .gitignore               # Git ignore file
```

## Installation

### Prerequisites

- Python 3.8+
- Neo4j database (cloud or local)
- Ollama (for local LLM) or OpenAI API key

### Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/liu-shiqiang/Graphrag_neo4j_kg.git
   cd Graphrag_neo4j_kg
   ```

2. **Create a virtual environment**:
   ```bash
   python -m venv graph_rag_env
   source graph_rag_env/bin/activate  # On Windows: graph_rag_env\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure Neo4j connection**:
   Set environment variables for Neo4j:
   ```bash
   export NEO4J_URI="neo4j+s://your-database-url"
   export NEO4J_USERNAME="neo4j"
   export NEO4J_PASSWORD="your-password"
   ```

5. **Configure LLM** (choose one):

   **Option A: Using Ollama (Local)**
   ```bash
   # Install Ollama from https://ollama.ai
   ollama pull llama3.1
   ollama serve  # Run in a separate terminal
   ```

   **Option B: Using OpenAI**
   ```bash
   export OPENAI_API_KEY="your-api-key"
   ```

## Usage

### Method 1: Using Jupyter Notebook (Recommended for exploration)

```bash
jupyter notebook main.ipynb
```

The notebook provides an interactive walkthrough of:
- Neo4j connection setup
- Token counting and document analysis
- LLM initialization
- Parallel document processing
- Graph construction and storage

### Method 2: Using Python Script

```bash
python main.py
```

The modular script uses functions from the `src/` directory:
- `process_and_store_biology_texts()` - Text preprocessing and CSV storage
- `initialize_llm()` - LLM setup
- `initialize_graph_transformer()` - Graph transformer configuration
- `extract_knowledge_from_csv()` - Knowledge extraction and storage

### Method 3: Standalone Extraction Script

```bash
python extractkg.py
```

This is a comprehensive standalone script that:
1. Reads biological texts from a directory
2. Cleans and preprocesses text
3. Initializes LLM and graph transformer
4. Extracts knowledge in parallel
5. Stores results in Neo4j

## Configuration

### Data Input

Update the paths in your script:
```python
# Biology texts directory (input)
biology_texts_directory = "/path/to/biology_texts"

# CSV output path (intermediate)
output_csv_path = "/path/to/output/biology_texts.csv"
```

### Entity Properties (Bioinformatics)

The graph transformer extracts the following entity properties:

- **Genes**: name, function, expression_level, location
- **Proteins**: name, structure, interaction
- **Cells**: type, function, lineage
- **Diseases**: name, classification, pathology
- **Variants**: id, gene_position, variant_type
- **Markers**: name, type
- **Drugs**: name, target, mechanism

### Relationship Properties

- **Gene-Gene**: interaction_type, evidence, confidence
- **Gene-Protein**: expression_regulation, functional_relationship
- **Protein-Protein**: binding_type, binding_location
- **Gene-Disease**: association_type, clinical_significance, variant_info
- **Variant-Disease**: clinical_relevance, association_strength
- **Cell-Marker**: marker_expression, marker_category
- **Drug Interactions**: target, treatment_effect, mechanism

### LLM Configuration

Modify LLM parameters:
```python
# Using Ollama
llm = ChatOllama(
    model="llama3.1",
    temperature=0.5,  # Adjust creativity (0-1)
    base_url="http://localhost:11434"
)

# Using OpenAI
llm = ChatOpenAI(
    temperature=0,
    model_name="gpt-4o"
)
```

### Parallel Processing

Adjust the number of worker threads:
```python
MAX_WORKERS = 2  # Increase for faster processing (depends on system resources)
```

## Output

The pipeline generates:

1. **CSV File**: Cleaned biological texts stored as intermediary format
2. **Neo4j Graph**: Structured knowledge graph with:
   - Nodes: Biological entities (genes, proteins, diseases, etc.)
   - Edges: Relationships between entities
   - Properties: Detailed attributes for both nodes and relationships

## Query Examples

Once the knowledge graph is built, you can query it using Cypher:

```cypher
// Find a specific gene and its relationships
MATCH (g:Gene {name: "TP53"}) - [r] - (entity)
RETURN g, r, entity

// Find disease associations
MATCH (gene:Gene) - [assoc:ASSOCIATED_WITH] - (disease:Disease)
WHERE assoc.clinical_significance = "high"
RETURN gene, assoc, disease

// Find drug targets
MATCH (drug:Drug) - [targets:TARGETS] - (gene:Gene)
RETURN drug, gene
```

## Performance Considerations

- **Token Limits**: Monitor token usage, especially with large documents
- **Parallel Workers**: Adjust `MAX_WORKERS` based on:
  - System RAM (each worker processes one document)
  - LLM rate limits (if using API-based models)
  - Available cores
- **Batch Processing**: For large datasets, consider processing in batches to manage memory

## Technologies Used

- **LangChain**: LLM orchestration and graph construction
- **LLMGraphTransformer**: Automated knowledge extraction from text
- **Neo4j**: Graph database backend
- **Ollama**: Local LLM inference
- **OpenAI API**: Alternative LLM provider
- **LangChain Community**: Integrations for various components
- **Pandas**: Data manipulation
- **Tiktoken**: Token counting for OpenAI models

## Future Enhancements

- [ ] Support for additional domains (finance, healthcare, scientific domains)
- [ ] Graph visualization tools
- [ ] Query optimization and indexing strategies
- [ ] Incremental knowledge graph updates
- [ ] Entity linking and deduplication
- [ ] Relationship confidence scoring
- [ ] Multi-document entity co-reference resolution
- [ ] Export to other graph formats (RDF, JSON-LD)

## Troubleshooting

### Neo4j Connection Issues
- Verify Neo4j URI, username, and password
- Check network connectivity to Neo4j database
- Ensure credentials have appropriate permissions

### LLM Not Responding
- For Ollama: Ensure Ollama is running (`ollama serve`)
- Check if the model is downloaded (`ollama list`)
- Verify base URL and port (default: `http://localhost:11434`)

### Out of Memory
- Reduce `MAX_WORKERS` to process fewer documents in parallel
- Decrease document batch size
- Process documents in smaller chunks

### Low Extraction Quality
- Adjust LLM temperature (lower = more deterministic, higher = more creative)
- Use a larger/more capable model
- Provide more domain-specific prompting in the transformer configuration

## References

- [LangChain LLMGraphTransformer](https://python.langchain.com/docs/use_cases/graph/)
- [Neo4j Python Driver](https://neo4j.com/docs/driver-manual/current/)
- [Ollama](https://ollama.ai/)
- [Microsoft GraphRAG](https://github.com/microsoft/graphrag)

## License

This project is provided as-is for research and educational purposes.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Contact

For questions or suggestions, please reach out to the repository maintainer.

---

**Note**: The repository includes Neo4j credentials in the code files. For production use, consider using environment variables or a configuration file that is not committed to version control.
