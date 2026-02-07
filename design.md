# Design: BharatAssist AI-Powered Community Knowledge & Service Access Engine

## 1. System Overview

BharatAssist is a modular AI system that processes citizen or service-related queries, classifies intent using LLMs, recommends relevant knowledge resources using semantic search, and performs analytics to detect gaps in information delivery.

## 2. High-Level Architecture

The system is composed of seven primary layers:

- Data Ingestion Layer
- Preprocessing Layer
- Classification Layer
- Embedding & Vector Index Layer
- Recommendation API Layer
- Analytics & Alerting Layer
- Visualization Layer

## 3. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Data Sources                              │
│                  (Google Sheets / CSV Files)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Data Ingestion Layer                           │
│              (gsheet_loader.py / CSV Reader)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Preprocessing Layer                            │
│                  (preprocessing2.py)                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Classification Layer                           │
│              (classification_tagging.py)                         │
│                    LLM-based Intent                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│              Embedding & Vector Index Layer                      │
│           (build_index.py / SentenceTransformers)                │
│                    FAISS Index                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Recommendation API Layer                         │
│              (recommend_api.py / FastAPI)                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│              Analytics & Alerting Layer                          │
│         (gap_analysis.py / slack_alerts.py)                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Visualization Layer                            │
│                  (app.py / Streamlit)                            │
└─────────────────────────────────────────────────────────────────┘
```

## 4. Component Design

### 4.1 Data Ingestion Layer

**Purpose:** Load queries and knowledge base articles from external sources

**Components:**
- `integrations/gsheet_loader.py`: Google Sheets API integration
- CSV file reader utilities

**Responsibilities:**
- Authenticate with Google Sheets API using service account credentials
- Load queries from specified sheets or CSV files
- Validate schema and data types
- Timestamp records for tracking
- Handle connection errors and retries

**Inputs:**
- Google Sheets URL or sheet ID
- CSV file paths
- Service account credentials

**Outputs:**
- Pandas DataFrame with validated query data
- Ingestion logs and error reports

**Key Design Decisions:**
- Use service account authentication for secure, unattended access
- Support both Google Sheets and CSV for flexibility
- Validate data schema on ingestion to catch errors early

### 4.2 Preprocessing Layer

**Purpose:** Clean and normalize raw text data for downstream processing

**Components:**
- `src/preprocessing2.py`: Text preprocessing pipeline

**Responsibilities:**
- Remove special characters and extra whitespace
- Normalize text encoding
- Handle empty or null values
- Preserve semantic meaning
- Support batch processing

**Inputs:**
- Raw query text
- Raw knowledge article text

**Outputs:**
- Cleaned and normalized text
- Preprocessing metadata

**Key Design Decisions:**
- Minimal preprocessing to preserve semantic information
- Language-agnostic approach for multilingual support
- Efficient batch processing for large datasets

### 4.3 Classification Layer

**Purpose:** Classify queries into predefined categories using LLMs

**Components:**
- `src/classification_tagging.py`: LLM-based classification

**Responsibilities:**
- Load and initialize LLM (LLaMA or Groq-based)
- Generate classification prompts
- Parse LLM responses
- Assign categories with confidence scores
- Handle ambiguous or multi-category queries

**Inputs:**
- Preprocessed query text
- Category definitions

**Outputs:**
- Assigned category
- Confidence score
- Classification metadata

**Key Design Decisions:**
- Use open-source LLMs for cost-effectiveness
- Prompt engineering for consistent classification
- Confidence thresholding for quality control

### 4.4 Embedding & Vector Index Layer

**Purpose:** Generate semantic embeddings and enable efficient similarity search

**Components:**
- `src/build_index.py`: Embedding generation and index building
- `models/embed_model.pkl`: Serialized embedding model
- `models/article_index.faiss`: FAISS vector index
- `models/articles_meta.pkl`: Article metadata

**Responsibilities:**
- Load SentenceTransformer model
- Generate embeddings for queries and articles
- Build and maintain FAISS index
- Perform k-nearest neighbor search
- Support index updates and versioning

**Inputs:**
- Preprocessed text (queries and articles)
- Embedding model configuration

**Outputs:**
- Vector embeddings
- FAISS index
- Similarity scores and rankings

**Key Design Decisions:**
- Use SentenceTransformers for high-quality semantic embeddings
- FAISS for efficient similarity search at scale
- Separate storage of embeddings and metadata for flexibility

### 4.5 Recommendation API Layer

**Purpose:** Expose real-time recommendation service via REST API

**Components:**
- `src/recommend_api.py`: FastAPI application
- `src/run_server.py`, `src/run_server2.py`: Server runners

**Responsibilities:**
- Accept query text via HTTP POST
- Preprocess and embed incoming queries
- Perform vector similarity search
- Return top-k ranked recommendations
- Log API requests and responses
- Provide OpenAPI documentation

**Inputs:**
- HTTP request with query text
- Configuration parameters (top-k, threshold)

**Outputs:**
- JSON response with ranked recommendations
- Article metadata (title, URL, score)
- Response time metrics

**API Endpoints:**
- `POST /recommend`: Get recommendations for a query
- `GET /health`: Health check endpoint
- `GET /docs`: OpenAPI documentation

**Key Design Decisions:**
- FastAPI for high performance and automatic documentation
- Stateless API design for horizontal scaling
- Structured JSON responses for easy integration

### 4.6 Analytics & Gap Detection Layer

**Purpose:** Track engagement metrics and identify knowledge gaps

**Components:**
- `src/gap_analysis.py`: Analytics and gap detection
- `logs/recommendation_results*.csv`: Recommendation logs
- `logs/coverage_report*.csv`: Coverage analysis

**Responsibilities:**
- Track impressions (recommendations shown)
- Track clicks (recommendations accessed)
- Calculate click-through rates (CTR)
- Identify low-performing resources (CTR < 0.5)
- Detect query patterns without matching resources
- Generate periodic reports

**Inputs:**
- Recommendation logs
- Engagement event data
- Performance thresholds

**Outputs:**
- Analytics reports
- Gap analysis summaries
- Low-performing resource lists

**Key Design Decisions:**
- Event-based tracking for accurate metrics
- Configurable thresholds for gap detection
- CSV-based logging for simplicity and portability

### 4.7 Alerting System

**Purpose:** Notify administrators of critical issues and insights

**Components:**
- `integrations/slack_alerts.py`: Slack notification integration
- `logs/alerts*.log`: Alert logs

**Responsibilities:**
- Generate alerts for low-performing resources
- Generate alerts for system errors
- Send notifications via Slack or email
- Rate-limit alerts to prevent spam
- Log all alert events

**Inputs:**
- Analytics results
- System error events
- Alert configuration

**Outputs:**
- Slack messages
- Email notifications
- Alert logs

**Key Design Decisions:**
- Slack integration for real-time notifications
- Configurable alert thresholds
- Rate limiting to avoid alert fatigue

### 4.8 Visualization Dashboard

**Purpose:** Provide interactive interface for monitoring and analysis

**Components:**
- `app.py`: Streamlit dashboard application

**Responsibilities:**
- Display key performance metrics
- Show classification accuracy trends
- Visualize engagement metrics by resource
- Present knowledge gap summaries
- Support manual workflow execution
- Provide filtering and time-range selection

**Inputs:**
- Analytics data
- System logs
- Configuration settings

**Outputs:**
- Interactive web dashboard
- Visualizations and charts
- Manual workflow triggers

**Key Design Decisions:**
- Streamlit for rapid dashboard development
- Real-time data refresh
- User-friendly interface for non-technical users

## 5. Data Flow

### 5.1 End-to-End Query Processing Flow

```
1. Query Ingestion
   ↓
2. Text Preprocessing
   ↓
3. Query Classification (LLM)
   ↓
4. Embedding Generation (SentenceTransformer)
   ↓
5. Vector Similarity Search (FAISS)
   ↓
6. Recommendation Ranking
   ↓
7. API Response
   ↓
8. Engagement Tracking
   ↓
9. Analytics & Gap Detection
   ↓
10. Alert Generation
    ↓
11. Dashboard Visualization
```

### 5.2 Knowledge Base Indexing Flow

```
1. Article Ingestion (Google Sheets / CSV)
   ↓
2. Text Preprocessing
   ↓
3. Embedding Generation
   ↓
4. FAISS Index Building
   ↓
5. Metadata Storage
   ↓
6. Index Persistence
```

### 5.3 Analytics Flow

```
1. Recommendation Logs Collection
   ↓
2. Engagement Event Tracking
   ↓
3. Metric Calculation (CTR, Coverage)
   ↓
4. Gap Detection
   ↓
5. Report Generation
   ↓
6. Alert Triggering
   ↓
7. Dashboard Update
```

## 6. Data Models

### 6.1 Query Data Model

```python
{
    "query_id": "string",
    "query_text": "string",
    "timestamp": "datetime",
    "source": "string",
    "preprocessed_text": "string",
    "category": "string",
    "confidence_score": "float",
    "embedding": "array[float]"
}
```

### 6.2 Knowledge Article Data Model

```python
{
    "article_id": "string",
    "title": "string",
    "content": "string",
    "url": "string",
    "category": "string",
    "tags": "array[string]",
    "preprocessed_content": "string",
    "embedding": "array[float]",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

### 6.3 Recommendation Data Model

```python
{
    "query_id": "string",
    "query_text": "string",
    "recommendations": [
        {
            "article_id": "string",
            "title": "string",
            "url": "string",
            "similarity_score": "float",
            "rank": "integer"
        }
    ],
    "timestamp": "datetime",
    "response_time_ms": "float"
}
```

### 6.4 Engagement Event Data Model

```python
{
    "event_id": "string",
    "event_type": "string",  # "impression" or "click"
    "query_id": "string",
    "article_id": "string",
    "timestamp": "datetime",
    "user_id": "string"  # optional
}
```

### 6.5 Analytics Report Data Model

```python
{
    "report_id": "string",
    "report_type": "string",  # "gap_analysis", "performance", etc.
    "period_start": "datetime",
    "period_end": "datetime",
    "metrics": {
        "total_queries": "integer",
        "total_recommendations": "integer",
        "average_ctr": "float",
        "classification_accuracy": "float"
    },
    "low_performing_articles": "array[object]",
    "knowledge_gaps": "array[object]",
    "generated_at": "datetime"
}
```

## 7. Technology Stack

### 7.1 Core Technologies

- **Python**: 3.10+
- **FastAPI**: REST API framework
- **Streamlit**: Dashboard framework
- **Pandas**: Data manipulation
- **NumPy**: Numerical operations

### 7.2 AI/ML Libraries

- **SentenceTransformers**: Semantic embeddings
- **FAISS**: Vector similarity search
- **LLaMA / Groq**: LLM for classification

### 7.3 Integration Libraries

- **gspread / google-auth**: Google Sheets API
- **slack-sdk**: Slack notifications
- **python-dotenv**: Environment configuration

### 7.4 Storage

- **CSV Files**: Data persistence
- **Pickle**: Model serialization
- **FAISS Index Files**: Vector storage

## 8. API Design

### 8.1 Recommendation API

**Endpoint:** `POST /recommend`

**Request:**
```json
{
    "query": "string",
    "top_k": "integer (optional, default: 5)",
    "threshold": "float (optional, default: 0.0)"
}
```

**Response:**
```json
{
    "query": "string",
    "recommendations": [
        {
            "article_id": "string",
            "title": "string",
            "url": "string",
            "score": "float",
            "rank": "integer"
        }
    ],
    "response_time_ms": "float",
    "timestamp": "string"
}
```

**Status Codes:**
- 200: Success
- 400: Bad request (invalid input)
- 500: Internal server error

### 8.2 Health Check API

**Endpoint:** `GET /health`

**Response:**
```json
{
    "status": "healthy",
    "timestamp": "string",
    "version": "string"
}
```

## 9. Configuration Management

### 9.1 Environment Variables

```
# Google Sheets Configuration
GOOGLE_SHEETS_CREDENTIALS_PATH=./credentials/service_account.json
GOOGLE_SHEETS_ID=<sheet_id>

# Model Configuration
EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
LLM_MODEL=llama-3-8b

# API Configuration
API_HOST=0.0.0.0
API_PORT=8000

# Slack Configuration
SLACK_WEBHOOK_URL=<webhook_url>

# Analytics Configuration
CTR_THRESHOLD=0.5
ALERT_FREQUENCY_HOURS=24
```

### 9.2 Configuration Files

- `.env`: Environment-specific configuration
- `requirements.txt`: Python dependencies
- `requirements-app.txt`: Application-specific dependencies

## 10. Deployment Design

### 10.1 Local Deployment

**Prerequisites:**
- Python 3.10+
- Virtual environment
- Service account credentials

**Steps:**
1. Clone repository
2. Create virtual environment
3. Install dependencies
4. Configure environment variables
5. Run data ingestion and indexing
6. Start API server
7. Launch dashboard

### 10.2 Cloud Deployment

**Supported Platforms:**
- AWS (EC2, Lambda, ECS)
- GCP (Compute Engine, Cloud Run)
- Render / Heroku

**Architecture:**
- Containerized services (Docker)
- Load balancer for API
- Persistent storage for models and indexes
- Scheduled jobs for analytics

**Deployment Steps:**
1. Build Docker images
2. Push to container registry
3. Deploy API service
4. Deploy dashboard service
5. Configure environment variables
6. Set up scheduled jobs
7. Configure monitoring and logging

### 10.3 Scaling Strategy

**Horizontal Scaling:**
- Multiple API server instances behind load balancer
- Stateless API design enables easy scaling

**Vertical Scaling:**
- Increase compute resources for embedding generation
- Optimize FAISS index for memory efficiency

**Data Scaling:**
- Partition large knowledge bases
- Implement incremental index updates

## 11. Security Design

### 11.1 Authentication & Authorization

- Service account authentication for Google Sheets
- API key authentication for recommendation API
- Role-based access for dashboard

### 11.2 Data Security

- Secure storage of credentials (environment variables, secrets manager)
- Encryption in transit (HTTPS)
- Minimal data persistence
- No storage of personally identifiable information (PII)

### 11.3 Access Control

- Restricted access to admin components
- Audit logging for sensitive operations
- Rate limiting on API endpoints

### 11.4 Security Best Practices

- Regular dependency updates
- Input validation and sanitization
- Error handling without information leakage
- Principle of least privilege

## 12. Monitoring & Observability

### 12.1 Logging

- Application logs (INFO, WARNING, ERROR)
- API request/response logs
- Analytics event logs
- Alert logs

### 12.2 Metrics

- API response time
- Classification accuracy
- Recommendation relevance
- System resource utilization
- Error rates

### 12.3 Health Checks

- API health endpoint
- Model availability check
- Index integrity check
- External service connectivity

## 13. Error Handling

### 13.1 Error Categories

- Data ingestion errors
- Preprocessing errors
- Classification errors
- Embedding generation errors
- API errors
- External service errors

### 13.2 Error Handling Strategy

- Graceful degradation
- Retry logic with exponential backoff
- Fallback mechanisms
- User-friendly error messages
- Detailed error logging

## 14. Performance Optimization

### 14.1 Embedding Generation

- Batch processing for efficiency
- Model caching
- GPU acceleration (if available)

### 14.2 Vector Search

- FAISS index optimization
- Approximate nearest neighbor search
- Index sharding for large datasets

### 14.3 API Performance

- Response caching
- Connection pooling
- Asynchronous processing

### 14.4 Dashboard Performance

- Data aggregation and caching
- Lazy loading of visualizations
- Pagination for large datasets

## 15. Testing Strategy

### 15.1 Unit Tests

- Component-level testing
- Mock external dependencies
- Test edge cases and error conditions

### 15.2 Integration Tests

- End-to-end workflow testing
- API endpoint testing
- Database and file I/O testing

### 15.3 Performance Tests

- Load testing for API
- Stress testing for vector search
- Latency benchmarking

### 15.4 User Acceptance Tests

- Dashboard usability testing
- Recommendation quality evaluation
- Alert delivery verification

## 16. Future Enhancements

### 16.1 Multilingual Support

- Language detection
- Multilingual embeddings
- Cross-lingual search

### 16.2 Voice Interface

- Speech-to-text integration
- Voice-first query processing
- Audio response generation

### 16.3 Personalization

- User profile management
- Personalized recommendations
- Learning from user feedback

### 16.4 Advanced Analytics

- Predictive analytics
- Trend analysis
- A/B testing framework

### 16.5 Integration Expansion

- CRM integration
- Public portal integration
- Mobile app support
- WhatsApp / SMS integration

### 16.6 Model Improvements

- Fine-tuning on domain-specific data
- Active learning from user feedback
- Ensemble models for improved accuracy

## 17. Correctness Properties

### 17.1 Data Integrity Properties

**Property 1.1: Ingestion Completeness**
- For all records in source data, ingestion must produce corresponding records in system
- No data loss during ingestion process

**Property 1.2: Preprocessing Idempotence**
- Preprocessing the same text multiple times produces identical output
- Preprocessing is deterministic

### 17.2 Classification Properties

**Property 2.1: Classification Consistency**
- Identical queries must receive identical classifications
- Classification is deterministic for same input

**Property 2.2: Confidence Score Validity**
- All confidence scores must be in range [0, 1]
- Higher confidence correlates with classification accuracy

### 17.3 Embedding Properties

**Property 3.1: Embedding Dimensionality**
- All embeddings have consistent dimensionality
- Embedding dimension matches model specification

**Property 3.2: Semantic Similarity**
- Semantically similar texts have higher cosine similarity
- Dissimilar texts have lower cosine similarity

### 17.4 Search Properties

**Property 4.1: Search Ranking**
- Results are ordered by descending similarity score
- Top-k results contain k highest scoring items

**Property 4.2: Search Completeness**
- If matching articles exist, search returns results
- Empty results only when no articles meet threshold

### 17.5 API Properties

**Property 5.1: Response Time**
- API response time < 1 second for 95th percentile
- Response time is measured end-to-end

**Property 5.2: API Correctness**
- Valid requests receive 200 status code
- Invalid requests receive appropriate error codes (400, 500)

### 17.6 Analytics Properties

**Property 6.1: Metric Accuracy**
- CTR = clicks / impressions
- All metrics are calculated correctly from event logs

**Property 6.2: Gap Detection Accuracy**
- Resources with CTR < threshold are flagged
- All flagged resources meet gap criteria

## 18. Testing Framework

### 18.1 Property-Based Testing

**Framework:** Hypothesis (Python)

**Test Categories:**
- Data integrity tests
- Classification consistency tests
- Embedding property tests
- Search correctness tests
- API contract tests
- Analytics calculation tests

### 18.2 Example Property Tests

**Test: Preprocessing Idempotence**
```python
@given(text=st.text())
def test_preprocessing_idempotence(text):
    result1 = preprocess(text)
    result2 = preprocess(text)
    assert result1 == result2
```

**Test: Embedding Dimensionality**
```python
@given(texts=st.lists(st.text(), min_size=1))
def test_embedding_dimensionality(texts):
    embeddings = generate_embeddings(texts)
    dimensions = [len(emb) for emb in embeddings]
    assert all(d == dimensions[0] for d in dimensions)
```

**Test: Search Ranking**
```python
@given(query=st.text(), k=st.integers(min_value=1, max_value=10))
def test_search_ranking(query, k):
    results = search(query, top_k=k)
    scores = [r['score'] for r in results]
    assert scores == sorted(scores, reverse=True)
```

## 19. Acceptance Criteria Mapping

### 19.1 Requirements to Design Mapping

| Requirement | Design Component | Correctness Property |
|-------------|------------------|---------------------|
| 2.1 Data Ingestion | Data Ingestion Layer | Property 1.1 |
| 2.2 Text Preprocessing | Preprocessing Layer | Property 1.2 |
| 2.3 Query Classification | Classification Layer | Property 2.1, 2.2 |
| 2.4 Semantic Embedding | Embedding Layer | Property 3.1, 3.2 |
| 2.5 Vector Search | Vector Index Layer | Property 4.1, 4.2 |
| 2.6 Recommendation API | API Layer | Property 5.1, 5.2 |
| 2.7 Engagement Tracking | Analytics Layer | Property 6.1 |
| 2.8 Gap Analysis | Analytics Layer | Property 6.2 |
| 2.9 Automated Alerts | Alerting System | N/A |
| 2.10 Dashboard | Visualization Layer | N/A |

## 20. Implementation Priorities

### 20.1 Phase 1: Core Functionality
1. Data ingestion
2. Preprocessing
3. Embedding generation
4. FAISS index building
5. Basic recommendation API

### 20.2 Phase 2: Intelligence Layer
1. LLM-based classification
2. Enhanced search ranking
3. API optimization

### 20.3 Phase 3: Analytics & Monitoring
1. Engagement tracking
2. Gap analysis
3. Alert system
4. Dashboard

### 20.4 Phase 4: Production Readiness
1. Security hardening
2. Performance optimization
3. Deployment automation
4. Comprehensive testing
