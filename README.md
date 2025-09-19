# 📰 News Pipeline N8N

![N8N](https://img.shields.io/badge/N8N-Compatible-blue?style=flat-square&logo=n8n)
![Version](https://img.shields.io/badge/version-1.0.0-green?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![Workflow Status](https://img.shields.io/badge/workflow-production%20ready-brightgreen?style=flat-square)

**Automated daily news monitoring and topic analysis pipeline built for N8N**

Transform scattered news sources into organized, topic-focused intelligence with AI-powered analysis and seamless Airtable integration.

## 🚀 Quick Start

1. **Import the workflow** into your N8N instance
2. **Configure credentials** for news APIs and Airtable
3. **Set up your topics** in Airtable
4. **Schedule the workflow** to run daily at 8:01 AM
5. **Monitor results** automatically delivered to your dashboard

## 📊 What It Does

- **🔄 Automated Collection**: Pulls from 6 major news sources daily
- **🎯 Smart Filtering**: AI-powered topic matching based on your interests
- **📈 Relevance Scoring**: Ranks articles by topic relevance and recency
- **📋 Organized Output**: Delivers structured data to Airtable/spreadsheets
- **⏰ Scheduled Execution**: Runs automatically every morning

## 🏗️ Architecture

```
Schedule Trigger (8:01 AM)
    ↓
Airtable Topics ← → 6 News Sources (CNN, WSJ, Google News, NY Times, Guardian, NewsAPI)
    ↓
AI Topic Analysis & Filtering
    ↓
Relevance Scoring & Ranking
    ↓
Structured Output → Airtable/Dashboard
```

## 📡 News Sources

| Source | Type | Coverage | Update Frequency |
|--------|------|----------|------------------|
| **CNN** | RSS | General News | Every 15 min |
| **Wall Street Journal** | API | Business/Finance | Hourly |
| **Google News** | RSS | Aggregated | Real-time |
| **New York Times** | RSS | Premium News | Every 30 min |
| **The Guardian** | RSS | International | Every 20 min |
| **NewsAPI** | API | 70,000+ sources | Real-time |

## ⚙️ Installation

### Prerequisites

- **N8N instance** (Cloud or Self-hosted)
- **Airtable account** with API access
- **News API keys** (WSJ, NewsAPI)

### Step 1: Import Workflow

```bash
# Download the workflow
curl -O https://raw.githubusercontent.com/jeremylongshore/news-pipeline-n8n/main/Daily_News_Topic_Tracker.json

# Import into N8N
# In N8N interface: Import → Upload JSON file
```

### Step 2: Configure Credentials

**Airtable Setup:**
1. Create new base with table "Topics to Monitor"
2. Add column: "Topics" (long text field)
3. Generate personal access token
4. Add credentials in N8N: `Airtable Personal Access Token`

**News API Setup:**
1. Register at [NewsAPI.org](https://newsapi.org) → Get API key
2. WSJ subscription (optional) → API access
3. Add HTTP Request credentials in N8N

### Step 3: Configure Topics

In your Airtable "Topics to Monitor" table:

| Topics |
|--------|
| artificial intelligence, machine learning, AI |
| blockchain, cryptocurrency, bitcoin |
| climate change, renewable energy, sustainability |
| startup, venture capital, IPO |

**Topic Format:** Comma-separated keywords for each monitoring category

### Step 4: Activate Workflow

1. Open workflow in N8N editor
2. Test with "Execute Workflow" button
3. Verify all connections work
4. Activate workflow (toggle switch)
5. Confirm scheduled trigger is set to 8:01 AM

## 🔧 Configuration

### Customizing News Sources

Edit the workflow to add/remove sources:

```javascript
// Add new RSS feed
{
  "parameters": {
    "url": "https://feeds.example.com/news.xml"
  },
  "type": "n8n-nodes-base.rssFeedRead"
}

// Add new API source
{
  "parameters": {
    "url": "https://api.example.com/articles?apiKey=YOUR_KEY"
  },
  "type": "n8n-nodes-base.httpRequest"
}
```

### Topic Matching Logic

The workflow uses flexible keyword matching:
- **Case insensitive** matching
- **Partial word** matches included
- **Multiple keywords** per topic (OR logic)
- **Relevance scoring** based on title vs content matches

### Scheduling Options

Change the trigger schedule:

```json
{
  "parameters": {
    "rule": {
      "interval": [
        {
          "triggerAtHour": 8,
          "triggerAtMinute": 1
        }
      ]
    }
  },
  "type": "n8n-nodes-base.scheduleTrigger"
}
```

## 📈 Output Format

Articles are delivered with comprehensive metadata:

```json
{
  "title": "AI Breakthrough in Medical Diagnosis",
  "description": "Researchers develop new machine learning model...",
  "url": "https://example.com/article",
  "source": "CNN",
  "published_date": "2024-01-15T10:30:00Z",
  "author": "Dr. Jane Smith",
  "monitored_topics": "artificial intelligence, machine learning",
  "matched_topics": "machine learning, AI",
  "relevance_score": 8.5
}
```

## 🔍 Monitoring & Troubleshooting

### Workflow Health Check

Monitor these key metrics:
- **Execution Success Rate**: Should be >95%
- **Articles Collected**: Typically 50-200 per day
- **Topic Match Rate**: Usually 10-30% of articles
- **API Response Times**: Most sources <2 seconds

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **No articles found** | Empty results | Check API keys, verify RSS feeds |
| **Topic mismatches** | Wrong articles | Refine keywords, check spelling |
| **Workflow timeouts** | Partial execution | Reduce batch sizes, check timeouts |
| **Rate limiting** | 429 errors | Add delays, check API limits |

### Debug Mode

Enable detailed logging:

```javascript
// Add to code nodes
console.log('Articles processed:', articles.length);
console.log('Topics matched:', matchedTopics);
console.log('API response:', response.status);
```

## 🛠️ Advanced Features

### Custom Filters

Add date filtering for recency:

```javascript
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);

const recentArticles = articles.filter(article =>
  new Date(article.published_date) > yesterday
);
```

### Sentiment Analysis

Integrate sentiment scoring:

```javascript
// Add sentiment analysis node
{
  "parameters": {
    "text": "{{ $json.description }}",
    "operation": "analyzeSentiment"
  },
  "type": "n8n-nodes-base.sentimentAnalysis"
}
```

### Multi-language Support

Handle international sources:

```javascript
// Language detection
const detectLanguage = (text) => {
  // Implementation for language detection
  return language;
};
```

## 📊 Analytics Dashboard

Track workflow performance:

- **Daily article volume trends**
- **Source reliability metrics**
- **Topic popularity analysis**
- **Response time monitoring**

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Test workflow thoroughly
4. Commit changes (`git commit -m 'Add amazing feature'`)
5. Push to branch (`git push origin feature/amazing-feature`)
6. Open Pull Request

## 📝 Release Notes

See [CHANGELOG.md](CHANGELOG.md) for detailed release history.

## 🆘 Support

- **Documentation**: [Wiki](https://github.com/jeremylongshore/news-pipeline-n8n/wiki)
- **Issues**: [GitHub Issues](https://github.com/jeremylongshore/news-pipeline-n8n/issues)
- **Discussions**: [GitHub Discussions](https://github.com/jeremylongshore/news-pipeline-n8n/discussions)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🎯 Use Cases

- **Business Intelligence**: Monitor industry trends and competitor news
- **Research**: Track academic developments and policy changes
- **Investment**: Follow market-moving news across sectors
- **Content Creation**: Discover trending topics for content planning
- **Crisis Management**: Early detection of relevant news events

---

**Built with ❤️ for the N8N community**

*Transform information chaos into actionable intelligence*