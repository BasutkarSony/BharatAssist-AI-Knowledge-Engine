# BharatAssist: AI-Powered Community Knowledge & Service Access Engine

BharatAssist is an AI system that helps citizens and communities access accurate information about public services. It processes queries, classifies intent using language models, recommends relevant knowledge resources through semantic search, and identifies gaps in information delivery.

The system addresses a common problem: citizens struggle to find timely, accurate information about public services due to fragmented portals, manual processes, and keyword-based systems that fail to understand intent. This leads to misinformation, delays, and reduced service efficiency.

BharatAssist is designed for public service helpdesks, community platforms, and government or NGO-supported information systems.



## Features

- **Data Ingestion:** Load queries from Google Sheets or CSV files with schema validation and error handling
- **Text Preprocessing:** Clean and normalize raw text while preserving semantic meaning
- **Query Classification:** Classify queries into categories using LLMs (LLaMA / Groq) with confidence scores
- **Semantic Embeddings:** Generate vector embeddings using SentenceTransformers for intelligent matching
- **Vector Similarity Search:** Retrieve top-k relevant knowledge resources using FAISS with sub-second performance
- **Recommendation API:** FastAPI service that accepts queries and returns ranked recommendations with metadata
- **Engagement Tracking:** Log impressions and clicks to calculate click-through rates (CTR)
- **Knowledge Gap Analysis:** Identify low-performing resources (CTR < 0.5) and query patterns without matches
- **Automated Alerts:** Send Slack notifications for low-performing resources and system issues
- **Monitoring Dashboard:** Streamlit interface for visualizing metrics, reports, and system status


## Architecture

The system is organized into seven layers:

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



## Components

| Module | Description |
|--------|--------------|
| `preprocessing2.py` | Cleans and normalizes raw text from queries while preserving semantic meaning |
| `classification_tagging.py` | Classifies queries using LLMs (LLaMA/Groq) with confidence scores |
| `build_index.py` | Generates embeddings and builds FAISS index for semantic search |
| `recommend_api.py` | FastAPI service that returns top-k ranked knowledge resources for queries |
| `gap_analysis.py` | Calculates impressions, clicks, and CTR to identify low-performing resources |
| `slack_alerts.py` | Sends Slack notifications for resources below performance thresholds |
| `gsheet_loader.py` | Loads query data from Google Sheets using service account authentication |
| `app.py` | Streamlit dashboard for monitoring metrics and triggering workflows |



## Technology Stack

- **Python 3.10+**
- **AI/ML:** SentenceTransformers, FAISS, LLaMA/Groq
- **Web Frameworks:** FastAPI, Streamlit
- **Data Processing:** Pandas, NumPy
- **Integrations:** Google Sheets API (gspread), Slack SDK
- **Deployment:** Local or cloud-ready (AWS, GCP, Render)



## Installation

### Clone the Repository
```bash
git clone https://github.com/BasutkarSony/AI-Ticket-Resolution.git
cd AI-Ticket-Resolution
```

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Configure Environment
Create a `.env` file in the project root:
```bash
SLACK_WEBHOOK_URL=<your_slack_webhook_url>
```

### Add Google Sheets Credentials
Place your Google service account JSON file at:
```
credentials/service_account.json
```



## Usage

### Start the Recommendation API
```bash
uvicorn recommend_api:app --reload
```
Access the API documentation at: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

### Launch the Dashboard
```bash
streamlit run app.py
```

The dashboard allows you to:
- Load queries from Google Sheets or CSV
- Run preprocessing and classification
- Generate recommendations
- Execute gap analysis
- Send Slack alerts



## Output Files

| File | Description |
|------|--------------|
| `data/processed/preprocessed_tickets*.csv` | Cleaned and normalized query data |
| `logs/coverage_report*.csv` | Engagement metrics and CTR analysis |
| `logs/recommendation_results*.csv` | Recommendation logs with scores and rankings |
| `logs/alerts*.log` | Alert generation and delivery logs |



## Performance

- Query classification accuracy: ~85%
- Recommendation API response time: <1 second per query
- Gap detection: Identifies resources with CTR < 0.5
- Alert delivery: Automated notifications for low-performing resources



## Future Enhancements

- Multilingual query understanding and response
- Voice-based query processing
- Personalized recommendations based on user history
- Integration with public portals and CRM systems
- Advanced analytics and trend analysis
- Fine-tuning models on domain-specific data




## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Acknowledgements

Built with:
- SentenceTransformers for semantic embeddings
- FAISS for efficient vector similarity search
- FastAPI for high-performance API services
- Streamlit for interactive dashboards
- Google Sheets API for data integration
- Slack API for real-time notifications  




