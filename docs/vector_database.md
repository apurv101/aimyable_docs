Using Vector Databases in Aimyable

### Using Vector Databases in Aimyable

To effectively utilize a vector database (Milvus, Chroma, or Pinecone) within the Aimyable architecture, we will integrate it primarily for storing and querying conversational history, user interactions, task metadata, and vector embeddings of documents related to task execution. Vector databases are excellent for handling high-dimensional data, such as text embeddings generated from user queries or tasks, and enable efficient similarity searches and contextual retrieval of past interactions or instructions.

#### 1. **Where Vector Databases Will Be Used**

The vector database will be used in several key areas within Aimyable:

- **Conversational History Retrieval**: Vector databases will store embeddings of past user queries, conversations, and interaction data. These embeddings will allow the system to retrieve similar past interactions for contextual continuity and to answer user queries efficiently.
  
- **Task and Project Metadata**: Task descriptions and project goals will be embedded and stored in the vector database, allowing users to query task-related data efficiently (e.g., "What project did I schedule last week?" or "What was my last QuickBooks task?").

- **Instruction Retrieval**: Embedding task instructions or user-specified modifications (e.g., "Use the top menu instead of the side menu") will allow for easy retrieval of similar instructions from previous executions, facilitating more refined execution for recurring or similar tasks.

- **Action History**: Embeddings of user actions or task completions will help identify and retrieve related historical data for user queries, optimizing task executions that involve repetitive actions.

#### 2. **How Vector Databases Will Be Used**

1. **Embedding Generation**:
   - **LLMs (Large Language Models)** will be used to generate embeddings for user queries, task descriptions, instructions, and historical interactions. These embeddings will be used as vectors and stored in the vector database.

2. **Storage of Embeddings**:
   - Every user query, task, instruction, or interaction will be embedded and stored as a vector in the vector database. These embeddings will be indexed to enable fast retrieval based on similarity.

3. **Similarity Search**:
   - When a user provides a new query (e.g., "Show me the tasks involving email from vendors"), the system will generate an embedding for this query and perform a similarity search against the vector database. This will retrieve relevant tasks, instructions, or historical data based on the proximity of the vectors in high-dimensional space.

4. **Contextual Continuity**:
   - For maintaining conversation continuity, embeddings from previous conversations will be retrieved and re-embedded during a session, ensuring that the system maintains context over long conversations or repeated interactions with the user.

#### 3. **Collections/Shards/Indexes Needed for the Vector Database**

Depending on the vector database you choose (Milvus, Chroma, Pinecone), the structure will vary slightly. Here’s how to set up the collections and shards:

- **Milvus**:
  - **Collections**: You will need to create collections for different types of data that will be embedded. These can include:
    - `conversation_history`: Stores embeddings of user interactions and queries.
    - `task_metadata`: Stores embeddings of task descriptions and project goals.
    - `instruction_history`: Stores embeddings of detailed task instructions.
    - `action_history`: Stores embeddings of past actions (e.g., screenshots, GUI interactions).
  - **Shards**: If scaling is required, Milvus allows sharding to spread the data across multiple nodes. Sharding can be used to distribute the embeddings of conversation history across shards based on users or time frames (e.g., `conversation_history_2024_Q1`).
  - **Indexes**: Milvus supports building indexes like IVF_FLAT or HNSW to improve search performance, especially when dealing with large datasets. Choose HNSW for better performance with high-dimensional vector searches.

- **Pinecone**:
  - **Namespaces**: Pinecone uses namespaces instead of collections. You can create similar namespaces for each type of data:
    - `conversation_history_namespace`
    - `task_metadata_namespace`
    - `instruction_history_namespace`
    - `action_history_namespace`
  - **Shards**: Pinecone automatically handles partitioning of data, so no need to manually define shards.
  - **Indexes**: Pinecone uses an optimized similarity search by default, so no manual indexing is required.

- **Chroma**:
  - **Collections**: In Chroma, you can set up collections similar to Milvus:
    - `conversation_history`
    - `task_metadata`
    - `instruction_history`
    - `action_history`
  - **Shards**: Chroma also scales automatically, so you don’t need to manage sharding manually.
  - **Indexes**: Chroma does not require manual index building, as it handles vector searches natively.

#### 4. **Where and How to Query Vector Databases**

Querying the vector database will be a critical part of the system’s user interface, particularly for retrieving historical data, tasks, and instructions. Here’s how queries can be structured:

1. **Embedding-Based Queries**:
   - For user queries such as “What tasks did I schedule last week?” or “Show me my previous email-related tasks,” the system will:
     1. Generate an embedding for the user’s query using an LLM.
     2. Use the generated embedding to perform a similarity search in the appropriate collection (e.g., `task_metadata` or `conversation_history`).
     3. Retrieve the top-k closest vectors, which represent similar tasks or historical interactions, and return them to the user.

2. **Historical Task Queries**:
   - Users can retrieve specific tasks by embedding a new query and searching the `task_metadata` collection to fetch tasks with similar descriptions or instructions.

3. **Instruction and Action-Based Queries**:
   - In cases where the user specifies detailed task modifications (e.g., “Use the top menu in QuickBooks”), these custom instructions can be embedded and stored in the `instruction_history` collection. Future queries about similar actions can retrieve relevant data efficiently.

4. **Contextual Querying**:
   - Contextual queries can also be enabled. For example, when a user resumes a session, the system can query the `conversation_history` collection for recent embeddings that match the ongoing conversation’s context, allowing it to provide relevant responses.

5. **Hybrid Querying**:
   - In some scenarios, you might combine vector searches with traditional filtering based on metadata (such as timestamps or task completion status). For instance, querying for "tasks with vendor emails from last month" will involve both vector-based similarity search and traditional MongoDB queries to filter by date.

#### 5. **Query Workflow Example**

Here’s how the query process would work in practice:

1. User asks: “What tasks related to QuickBooks did I perform in the past week?”
   
2. **Embedding Generation**:
   - The system generates an embedding for the query using an LLM.

3. **Similarity Search**:
   - The query embedding is sent to the vector database (e.g., Milvus, Pinecone, or Chroma), searching in the `task_metadata` collection.

4. **Retrieving Similar Tasks**:
   - The top-k similar vectors (representing task descriptions) are retrieved from the database.

5. **Filtering by Time**:
   - A secondary query is made to MongoDB to filter these results by tasks completed within the past week.

6. **Response**:
   - The system returns the filtered, most relevant tasks to the user.

### Summary

Vector databases like Milvus, Chroma, or Pinecone will be critical for handling high-dimensional data in Aimyable. They will be used primarily for storing and querying conversational history, task metadata, instructions, and actions. Collections/namespaces will be set up for each type of data, with efficient querying achieved through vector-based similarity searches. This integration will enable efficient, real-time retrieval of data for user queries and task management.

