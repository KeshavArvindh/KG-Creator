# Cypher Query Executor with Error Handling and Gemini API Debugging

## Overview

This project provides a robust Python-based solution for executing Cypher queries in a Neo4j graph database. It includes advanced error handling to ensure smooth query execution, even in the presence of errors. If a Cypher query fails, the error is sent to the **Gemini API**, which analyzes the error and provides a corrected query. The solution retries the corrected query until successful execution.

## Features

- **Cypher Query Execution**: Executes a list of Cypher queries in a Neo4j database.
- **Error Handling**: Catches execution errors and logs them for debugging.
- **Gemini API Integration**: Uses the Gemini API to debug and fix Cypher queries automatically.
- **Retry Mechanism**: Retries the corrected query until it succeeds.
- **Neo4j Integration**: Easily connects to your Neo4j database.

## Technologies Used

- **Python**: Primary programming language.
- **Neo4j**: Graph database for storing and querying data.
- **Py2Neo**: Python client library for connecting to Neo4j.
- **OpenAI API**: For leveraging the Gemini API to debug and fix queries.

## Installation

### Prerequisites

1. **Neo4j Database**:
   - Install and set up Neo4j on your system.
   - Ensure the Neo4j server is running and accessible.
   
2. **Python 3.x**:
   - Install Python 3.x from [python.org](https://www.python.org/).

3. **Python Libraries**:
   Install the required libraries using the following command:
   ```bash
   pip install py2neo openai
### 4. OpenAI API Key

- Obtain an API key for Gemini from [OpenAI](https://openai.com/).
- Replace `your_gemini_api_key` in the code with your actual API key.

## Usage

### Code Overview

The script processes a list of Cypher queries, executes them in Neo4j, and handles errors as follows:

1. **Query Execution**:
   The script iterates through a list of Cypher queries and executes them using `graph.run()`.

2. **Error Handling**:
   If a query fails, the error is caught, logged, and sent to the Gemini API for debugging.

3. **Gemini API Debugging**:
   The Gemini API analyzes the error and provides an updated query.

4. **Retry Mechanism**:
   The script retries the updated query until successful execution or an unrecoverable error occurs.

### Example Script

Here’s the core implementation:

```python
from py2neo import Graph
import openai

# Initialize Neo4j connection
graph = Graph("bolt://localhost:7687", auth=("neo4j", "password"))

# Set OpenAI API key
openai.api_key = "your_gemini_api_key"

def debug_cypher_with_gemini(error_message, query):
    """
    Debugs Cypher queries using the Gemini API.
    """
    prompt = f"""
    The following Cypher query resulted in an error:
    Query: {query}
    Error: {error_message}

    Please debug the query and ONLY provide the corrected Cypher query as output, ensuring no additional explanations.
    """
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are a Cypher query debugging assistant."},
            {"role": "user", "content": prompt}
        ]
    )
    return response['choices'][0]['message']['content'].strip()

# List of Cypher queries
cypher_queries = [
    "CREATE (u:User {userId: 1})",
    "MATCH (u:User {userId: 1}) CREATE (m:Movie {movieId: 101})-[r:TAGGED]->(u)",
    "MATCH (m:Movie {movieId: 101}) CREATE (t:Tag {name: 'Action'})-[:HAS_TAG]->(m)"
]

# Execute queries with error handling
for query in cypher_queries:
    if query.strip():
        while True:
            try:
                graph.run(query)
                print(f"Executed successfully: {query}")
                break
            except Exception as e:
                error_message = str(e)
                print(f"Error encountered: {error_message}")
                try:
                    query = debug_cypher_with_gemini(error_message, query)
                    print(f"Updated query: {query}")
                except Exception as gemini_error:
                    print(f"Error with Gemini API: {gemini_error}")
                    raise RuntimeError("Failed to debug query using Gemini API.") from gemini_error
```

## Example Output

### Successful Query Execution
```bash



Executed successfully: CREATE (u:User {userId: 1})
Executed successfully: MATCH (u:User {userId: 1}) CREATE (m:Movie {movieId: 101})-[:TAGGED]->(u)
Executed successfully: MATCH (m:Movie {movieId: 101}) CREATE (t:Tag {name: 'Action'})-[:HAS_TAG]->(m)
```

### Gemini API Debugging in Action
If a query fails, you’ll see output like this:
```bash



Error encountered: SyntaxError: ...
Sending error to Gemini API for debugging...
Updated query received: MATCH (u:User {userId: 1}) CREATE (m:Movie {movieId: 101})-[:TAGGED]->(u)
Executed successfully: MATCH (u:User {userId: 1}) CREATE (m:Movie {movieId: 101})-[:TAGGED]->(u)
```

## Contributing

Contributions to this project are welcome! Feel free to fork the repository, create feature branches, and submit pull requests.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

For questions or support, please contact:

**Name:** Keshav Arvindh  
**Email:** akeshav0601@gmail.com  
**LinkedIn:** https://www.linkedin.com/in/akeshav0601/
