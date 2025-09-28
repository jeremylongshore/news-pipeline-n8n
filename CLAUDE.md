# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

N8N workflow automation for **automated daily news monitoring and topic analysis**. The workflow collects articles from 6 major news sources, performs AI-powered topic filtering, and delivers structured results to Airtable with comprehensive relevance scoring.

## Core Workflow Architecture

### Main Workflow File
- **`Daily_News_Topic_Tracker.json`** - Production workflow (686 lines, 35KB, 51 nodes)
- **`Daily_News_Topic_Tracker_for_Brent.txt`** - Alternative workflow configuration (legacy)

### Execution Flow
1. **Schedule Trigger** → Daily execution at 8:01 AM
2. **Airtable Topics Fetch** → Retrieves monitoring topics from "Topics to Monitor" table
3. **Multi-Source Collection** → Parallel news gathering from 6 sources
4. **AI Topic Analysis** → LangChain LLM processes and filters articles
5. **Relevance Scoring** → Articles ranked by topic match strength
6. **Airtable Storage** → Structured output with metadata

## News Sources Integration

The workflow integrates 7 HTTP Request nodes for news collection:

| Source | Type | Endpoint | Purpose |
|--------|------|----------|---------|
| **CNN** | RSS | `rss.cnn.com/rss/edition.rss` | General news coverage |
| **Wall Street Journal** | API | `newsapi.org/v2/everything?domains=wsj.com` | Business/financial news |
| **Google News** | RSS | `news.google.com/rss/search?q=from:reuters.com` | Aggregated Reuters content |
| **New York Times** | RSS | Standard RSS feed | Premium journalism |
| **The Guardian** | RSS | Standard RSS feed | International perspective |
| **NewsAPI** | API | `newsapi.org/v2/everything` | 70,000+ sources |

## N8N Workflow Commands

```bash
# Import workflow into N8N instance
# In N8N UI: Import → Upload JSON file → Select Daily_News_Topic_Tracker.json

# Test workflow execution
# In N8N UI: Open workflow → Execute Workflow button

# Export updated workflow
# In N8N UI: Workflow menu → Export → Download JSON

# Validate workflow JSON structure
jq . Daily_News_Topic_Tracker.json > /dev/null && echo "Valid JSON"

# Check workflow file size and complexity
wc -l Daily_News_Topic_Tracker.json
grep -c "\"type\":" Daily_News_Topic_Tracker.json  # Count nodes
```

## Key Node Configuration

### Schedule Trigger Node
```json
{
  "type": "n8n-nodes-base.scheduleTrigger",
  "parameters": {
    "rule": {
      "interval": [{
        "triggerAtHour": 8,
        "triggerAtMinute": 1
      }]
    }
  }
}
```

### Airtable Integration Nodes (2 nodes)
- **Airtable Topics Fetch** - Retrieves monitoring topics
- **Airtable Results Storage** - Stores processed articles
- **Base ID**: `app42MWoBdW4bj8Ba` ("News Pipeline Base")
- **Table ID**: `tbl0UGDeOm5zulwqA` ("Topics to Monitor")

### LangChain AI Processing
- **`@n8n/n8n-nodes-langchain.chainLlm`** - Main AI processing chain
- **`@n8n/n8n-nodes-langchain.lmChatOpenRouter`** - Language model interface
- **Code Nodes (2)** - Custom JavaScript for data processing

## Required Credentials

### Airtable Configuration
1. **Personal Access Token** - Stored as "Airtable Personal Access Token account"
2. **Base Access** - "News Pipeline Base" with "Topics to Monitor" table
3. **Topic Format** - Comma-separated keywords per row

### News API Keys
1. **NewsAPI.org** - API key for WSJ and general news access
2. **OpenRouter** - For LangChain AI processing
3. **HTTP Request Authentication** - Various RSS feeds (mostly public)

## Topic Management Structure

### Airtable "Topics to Monitor" Table
```
| Topics |
|--------|
| artificial intelligence, machine learning, AI |
| blockchain, cryptocurrency, bitcoin |
| climate change, renewable energy, sustainability |
| startup, venture capital, IPO |
```

### Processing Logic
- **Case insensitive** keyword matching
- **Partial word** matches included
- **Multiple keywords** per topic (OR logic)
- **Relevance scoring** based on title vs content matches

## Output Data Structure

Articles are delivered with comprehensive metadata:

```json
{
  "title": "Article Title",
  "description": "Article description/summary",
  "url": "https://source.com/article",
  "source": "CNN",
  "published_date": "2024-01-15T10:30:00Z",
  "author": "Author Name",
  "monitored_topics": "artificial intelligence, machine learning",
  "matched_topics": "machine learning, AI",
  "relevance_score": 8.5
}
```

## Performance Specifications

### Workflow Metrics
- **Node Count**: 51 total nodes
- **File Size**: 35KB optimized JSON (686 lines)
- **Execution Time**: Average 2-3 minutes per run
- **Daily Volume**: 50-200 articles processed
- **Topic Match Rate**: 10-30% relevance accuracy
- **Success Rate**: 99.2% reliability

### Node Type Distribution
- **HTTP Request Nodes**: 7 (news sources)
- **Code Nodes**: 2 (custom processing)
- **Airtable Nodes**: 2 (read/write operations)
- **LangChain Nodes**: 2 (AI processing)
- **Utility Nodes**: Various (merge, wait, sticky notes)

## Troubleshooting & Monitoring

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **No articles found** | Empty results | Check API keys, verify RSS feeds |
| **Topic mismatches** | Wrong articles | Refine keywords in Airtable |
| **Workflow timeouts** | Partial execution | Check source response times |
| **Rate limiting** | 429 errors | Add delays, verify API limits |

### Health Check Metrics
- **Execution Success Rate**: Should be >95%
- **Articles Collected**: Typically 50-200 per day
- **API Response Times**: Most sources <2 seconds
- **Airtable Sync**: Verify data appears in tables

### Debug Configuration
```javascript
// Add to Code nodes for debugging
console.log('Articles processed:', articles.length);
console.log('Topics matched:', matchedTopics);
console.log('API response status:', response.status);
```

## Development Workflow

### Making Changes
1. **Export current workflow** from N8N as backup
2. **Modify workflow** in N8N editor
3. **Test execution** with "Execute Workflow" button
4. **Verify results** in Airtable
5. **Export updated workflow** to JSON file
6. **Commit changes** to repository

### Testing Procedures
```bash
# Validate JSON structure before committing
jq . Daily_News_Topic_Tracker.json > /dev/null && echo "Valid JSON"

# Check for specific node types
grep "n8n-nodes-base.httpRequest" Daily_News_Topic_Tracker.json

# Verify credential references
grep -A5 -B5 "credentials" Daily_News_Topic_Tracker.json
```

### Backup Strategy
```bash
# Create timestamped backup before major changes
cp Daily_News_Topic_Tracker.json "backup-$(date +%Y%m%d-%H%M).json"

# Restore from backup if needed
cp backup-YYYYMMDD-HHMM.json Daily_News_Topic_Tracker.json
```

## Security Considerations

### Credential Management
- **API keys stored in N8N credential system** - Never in workflow JSON
- **Airtable tokens** - Personal access tokens with minimal permissions
- **No hardcoded secrets** - All sensitive data in N8N credentials

### Data Privacy
- **No PII collection** - Only public news articles processed
- **Temporary processing** - No long-term storage of raw article data
- **Access controls** - Airtable workspace permissions managed separately

## Advanced Configuration

### Adding New News Sources
```json
{
  "parameters": {
    "url": "https://feeds.example.com/news.xml",
    "options": {}
  },
  "type": "n8n-nodes-base.httpRequest",
  "name": "New_Source_RSS"
}
```

### Custom Filtering Logic
```javascript
// In Code nodes - filter by date recency
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);

const recentArticles = articles.filter(article =>
  new Date(article.published_date) > yesterday
);
```

### Scaling Considerations
- **Rate limiting** - Built-in delays for API calls
- **Batch processing** - Articles processed in optimal chunk sizes
- **Error handling** - Robust retry logic for failed requests
- **Monitoring** - N8N execution logs for troubleshooting