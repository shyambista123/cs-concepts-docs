# Building RAG Applications with Spring Boot and Spring AI

Retrieval-Augmented Generation (RAG) is a powerful technique that combines information retrieval with large language models (LLMs) to provide accurate, context-aware responses based on your own data. Spring AI makes it easy to build RAG applications in Java.

## What is RAG?

RAG enhances LLM responses by:
1. **Retrieving** relevant information from a knowledge base
2. **Augmenting** the LLM prompt with retrieved context
3. **Generating** accurate responses based on your data

This approach reduces hallucinations and provides answers grounded in your specific documents.

## Prerequisites

- Java 17 or higher
- Maven or Gradle
- Spring Boot 3.2+
- OpenAI API key (or other LLM provider)

## Project Setup

### 1. Create Spring Boot Project

Use [start.spring.io](https://start.spring.io/) with these dependencies:
- Spring Web
- Spring AI OpenAI
- Spring AI Vector Store (PGVector, Chroma, or Pinecone)

### 2. Add Dependencies (Maven)

```xml
<dependencies>
    <!-- Spring AI -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>
    
    <!-- Vector Store - Choose one -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
    </dependency>
    
    <!-- Document Readers -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pdf-document-reader</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-tika-document-reader</artifactId>
    </dependency>
</dependencies>
```

### 3. Configuration

```properties
# application.properties

# OpenAI Configuration
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4
spring.ai.openai.chat.options.temperature=0.7

# Embedding Model
spring.ai.openai.embedding.options.model=text-embedding-ada-002

# Vector Store (PGVector example)
spring.datasource.url=jdbc:postgresql://localhost:5432/vectordb
spring.datasource.username=postgres
spring.datasource.password=password
spring.ai.vectorstore.pgvector.initialize-schema=true
spring.ai.vectorstore.pgvector.dimensions=1536
```

## Building a RAG Application

### 1. Document Loader Service

```java
package com.example.rag.service;

import org.springframework.ai.document.Document;
import org.springframework.ai.reader.pdf.PagePdfDocumentReader;
import org.springframework.ai.reader.tika.TikaDocumentReader;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class DocumentLoaderService {

    @Autowired
    private VectorStore vectorStore;

    public void loadPdfDocument(Resource pdfResource) {
        // Read PDF document
        PagePdfDocumentReader pdfReader = new PagePdfDocumentReader(pdfResource);
        List<Document> documents = pdfReader.get();
        
        // Split documents into chunks
        TokenTextSplitter splitter = new TokenTextSplitter();
        List<Document> chunks = splitter.apply(documents);
        
        // Store in vector database
        vectorStore.add(chunks);
    }

    public void loadTextDocument(Resource resource) {
        TikaDocumentReader reader = new TikaDocumentReader(resource);
        List<Document> documents = reader.get();
        
        TokenTextSplitter splitter = new TokenTextSplitter(
            800,  // chunk size
            200   // overlap
        );
        List<Document> chunks = splitter.apply(documents);
        
        vectorStore.add(chunks);
    }

    public void loadMultipleDocuments(List<Resource> resources) {
        resources.forEach(this::loadTextDocument);
    }
}
```

### 2. RAG Service

```java
package com.example.rag.service;

import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.ChatResponse;
import org.springframework.ai.chat.messages.Message;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.prompt.SystemPromptTemplate;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Service
public class RagService {

    @Autowired
    private ChatClient chatClient;

    @Autowired
    private VectorStore vectorStore;

    private static final String SYSTEM_PROMPT = """
            You are a helpful assistant that answers questions based on the provided context.
            Use only the information from the context to answer the question.
            If the answer cannot be found in the context, say "I don't have enough information to answer that."
            
            Context:
            {context}
            """;

    public String query(String question) {
        // 1. Retrieve relevant documents
        List<Document> relevantDocs = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(5)
        );

        // 2. Build context from retrieved documents
        String context = relevantDocs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));

        // 3. Create prompt with context
        SystemPromptTemplate systemPromptTemplate = new SystemPromptTemplate(SYSTEM_PROMPT);
        Message systemMessage = systemPromptTemplate.createMessage(Map.of("context", context));
        UserMessage userMessage = new UserMessage(question);

        // 4. Generate response
        Prompt prompt = new Prompt(List.of(systemMessage, userMessage));
        ChatResponse response = chatClient.call(prompt);

        return response.getResult().getOutput().getContent();
    }

    public RagResponse queryWithSources(String question) {
        // Retrieve relevant documents
        List<Document> relevantDocs = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(5)
        );

        String context = relevantDocs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));

        // Generate response
        SystemPromptTemplate systemPromptTemplate = new SystemPromptTemplate(SYSTEM_PROMPT);
        Message systemMessage = systemPromptTemplate.createMessage(Map.of("context", context));
        UserMessage userMessage = new UserMessage(question);

        Prompt prompt = new Prompt(List.of(systemMessage, userMessage));
        ChatResponse response = chatClient.call(prompt);

        String answer = response.getResult().getOutput().getContent();

        // Extract sources
        List<String> sources = relevantDocs.stream()
            .map(doc -> doc.getMetadata().get("source"))
            .map(Object::toString)
            .distinct()
            .collect(Collectors.toList());

        return new RagResponse(answer, sources);
    }
}
```

### 3. Response Model

```java
package com.example.rag.service;

import java.util.List;

public record RagResponse(String answer, List<String> sources) {}
```

### 4. REST Controller

```java
package com.example.rag.controller;

import com.example.rag.service.DocumentLoaderService;
import com.example.rag.service.RagResponse;
import com.example.rag.service.RagService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RestController
@RequestMapping("/api/rag")
public class RagController {

    @Autowired
    private RagService ragService;

    @Autowired
    private DocumentLoaderService documentLoaderService;

    @PostMapping("/upload")
    public ResponseEntity<String> uploadDocument(@RequestParam("file") MultipartFile file) {
        try {
            Resource resource = file.getResource();
            documentLoaderService.loadTextDocument(resource);
            return ResponseEntity.ok("Document uploaded and processed successfully");
        } catch (IOException e) {
            return ResponseEntity.badRequest().body("Error processing document: " + e.getMessage());
        }
    }

    @PostMapping("/query")
    public ResponseEntity<String> query(@RequestBody QueryRequest request) {
        String answer = ragService.query(request.question());
        return ResponseEntity.ok(answer);
    }

    @PostMapping("/query-with-sources")
    public ResponseEntity<RagResponse> queryWithSources(@RequestBody QueryRequest request) {
        RagResponse response = ragService.queryWithSources(request.question());
        return ResponseEntity.ok(response);
    }
}

record QueryRequest(String question) {}
```

## Advanced Features

### 1. Custom Embedding Model

```java
@Configuration
public class EmbeddingConfig {

    @Bean
    public EmbeddingClient embeddingClient() {
        return new OpenAiEmbeddingClient(
            new OpenAiApi(apiKey),
            MetadataMode.EMBED,
            OpenAiEmbeddingOptions.builder()
                .withModel("text-embedding-3-large")
                .build()
        );
    }
}
```

### 2. Metadata Filtering

```java
public String queryWithFilter(String question, String category) {
    SearchRequest searchRequest = SearchRequest.query(question)
        .withTopK(5)
        .withSimilarityThreshold(0.7)
        .withFilterExpression("category == '" + category + "'");
    
    List<Document> relevantDocs = vectorStore.similaritySearch(searchRequest);
    // ... rest of the implementation
}
```

### 3. Hybrid Search (Keyword + Semantic)

```java
public List<Document> hybridSearch(String query) {
    // Semantic search
    List<Document> semanticResults = vectorStore.similaritySearch(
        SearchRequest.query(query).withTopK(10)
    );
    
    // Keyword search (implement based on your vector store)
    List<Document> keywordResults = performKeywordSearch(query);
    
    // Combine and re-rank results
    return combineAndRerank(semanticResults, keywordResults);
}
```

### 4. Conversation Memory

```java
@Service
public class ConversationalRagService {

    @Autowired
    private ChatClient chatClient;

    @Autowired
    private VectorStore vectorStore;

    private final Map<String, List<Message>> conversationHistory = new ConcurrentHashMap<>();

    public String queryWithHistory(String sessionId, String question) {
        List<Document> relevantDocs = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(5)
        );

        String context = relevantDocs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));

        List<Message> messages = conversationHistory.getOrDefault(sessionId, new ArrayList<>());
        
        if (messages.isEmpty()) {
            messages.add(new SystemMessage("Context: " + context));
        }
        
        messages.add(new UserMessage(question));

        Prompt prompt = new Prompt(messages);
        ChatResponse response = chatClient.call(prompt);
        
        String answer = response.getResult().getOutput().getContent();
        messages.add(new AssistantMessage(answer));
        
        conversationHistory.put(sessionId, messages);

        return answer;
    }
}
```

## Testing the RAG Application

### 1. Upload Documents

```bash
curl -X POST http://localhost:8080/api/rag/upload \
  -F "file=@document.pdf"
```

### 2. Query the System

```bash
curl -X POST http://localhost:8080/api/rag/query \
  -H "Content-Type: application/json" \
  -d '{"question": "What is the main topic of the document?"}'
```

### 3. Query with Sources

```bash
curl -X POST http://localhost:8080/api/rag/query-with-sources \
  -H "Content-Type: application/json" \
  -d '{"question": "Explain the key concepts"}'
```

## Best Practices

1. **Chunk Size**: Experiment with chunk sizes (500-1000 tokens typically work well)
2. **Overlap**: Use 10-20% overlap between chunks to maintain context
3. **Top-K**: Start with 3-5 relevant documents, adjust based on results
4. **Similarity Threshold**: Filter out low-relevance documents (0.7+ similarity)
5. **Prompt Engineering**: Craft clear system prompts that guide the model
6. **Metadata**: Add rich metadata to documents for better filtering
7. **Caching**: Cache embeddings to reduce API calls
8. **Error Handling**: Implement robust error handling for API failures

## Vector Store Options

### PGVector (PostgreSQL)
- Best for: Production applications with existing PostgreSQL
- Pros: ACID compliance, mature ecosystem
- Cons: Requires PostgreSQL setup

### Chroma
- Best for: Development and prototyping
- Pros: Easy setup, no external dependencies
- Cons: Less scalable for production

### Pinecone
- Best for: Large-scale production applications
- Pros: Fully managed, highly scalable
- Cons: Requires external service, cost

### Weaviate
- Best for: Complex semantic search requirements
- Pros: Advanced features, GraphQL API
- Cons: More complex setup

## Common Use Cases

- **Customer Support**: Answer questions from documentation
- **Knowledge Management**: Search internal company documents
- **Research Assistant**: Query academic papers and reports
- **Code Documentation**: Search and explain codebases
- **Legal Document Analysis**: Query contracts and legal texts

## Troubleshooting

### Low-Quality Responses
- Increase chunk overlap
- Adjust similarity threshold
- Improve document preprocessing
- Use better embedding models

### Slow Performance
- Implement caching
- Optimize chunk size
- Use batch processing for uploads
- Consider vector store indexing

### High Costs
- Cache embeddings
- Use smaller embedding models
- Implement rate limiting
- Optimize chunk sizes

---

- Go back to [Frameworks](./index.md)
- Return to [Home](../../index.md)
