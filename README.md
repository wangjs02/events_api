# Event API Service

A unified Python API service that aggregates event data from multiple sources including Eventbrite, Meetup, Ticketmaster, AllEvents, SerpApi, and PredictHQ.

## Features

- 🎯 **Multi-Source Aggregation**: Fetch events from 6+ different providers
- 🔐 **Built-in Authentication**: API Key-based authentication
- ⚡ **Rate Limiting**: Configurable request limits
- 🌍 **Geocoding Support**: Automatic location resolution
- 📦 **Easy to Deploy**: Simple Flask-based architecture
- 🔧 **Configurable**: Environment-based configuration

## Installation

```bash
pip install event_api_service
```

## Quick Start

### Option 1: Direct Usage (Simplest) ⭐

No server needed! Just install and use:

```python
from event_api.services.scraper import UnifiedEventService

# Create service instance
service = UnifiedEventService()

# Get events (works with Eventbrite without any API keys!)
events = service.get_events("New York", "music")

# Use the data
for event in events:
    print(f"{event['title']} - {event['source']}")
```

**That's it! No API keys required for basic usage with Eventbrite.**

### Option 2: With Additional Data Sources

For more results, set API keys for other providers:

```python
import os
from event_api.services.scraper import UnifiedEventService

# Optional: Add more data sources
os.environ['TICKETMASTER_KEY'] = 'your_ticketmaster_key'
os.environ['MEETUP_TOKEN'] = 'your_meetup_token'

service = UnifiedEventService()
events = service.get_events("San Francisco", "tech")
```

### Option 3: Run as API Server

If you want to run it as a web service:

**1. Create `.env` file:**

```bash
# Optional: Event provider API keys
TICKETMASTER_KEY=your_ticketmaster_key
MEETUP_TOKEN=your_meetup_token

# Required: Your API service key (set any secure string)
API_KEY=your_secret_api_key
```

**2. Create `server.py`:**

```python
from dotenv import load_dotenv
from event_api import create_app

load_dotenv()
app = create_app()

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

**3. Run and use:**

```bash
# Start server
python server.py

# Make requests
curl -H "x-api-key: your_secret_api_key" \
  "http://localhost:5000/api/v1/events?location=New York&category=music"
```

## API Documentation

### Authentication

All endpoints require an API Key in the request header:

```
x-api-key: your_secret_api_key
```

### Endpoints

#### GET /api/v1/events

Fetch events by location and category.

**Parameters:**
- `location` (string, required): City name (e.g., "New York", "London")
- `category` (string, required): Event category (e.g., "music", "tech", "sports")

**Response:**

```json
{
  "status": "success",
  "count": 15,
  "data": [
    {
      "id": "evt-123",
      "title": "Tech Conference 2025",
      "description": "Annual technology conference...",
      "start_time": "2025-12-01T09:00:00",
      "url": "https://example.com/event/123",
      "location": "Convention Center",
      "source": "eventbrite"
    }
  ]
}
```

## Supported Event Sources

| Provider | Configuration Required | Notes |
|----------|----------------------|-------|
| Eventbrite | No | Free, supports major US cities |
| Ticketmaster | `TICKETMASTER_KEY` | Music, sports, arts events |
| Meetup | `MEETUP_TOKEN` | Community events |
| AllEvents | `ALLEVENTS_KEY` | Global event aggregator |
| SerpApi | `SERPAPI_KEY` | Google Events search |
| PredictHQ | `PREDICTHQ_TOKEN` | Professional event data |

## Rate Limits

- 10 requests per minute
- 50 requests per hour
- 200 requests per day

## Python Client Example

```python
import requests

class EventAPIClient:
    def __init__(self, api_key, base_url="http://localhost:5000/api/v1"):
        self.api_key = api_key
        self.base_url = base_url
    
    def get_events(self, location, category):
        headers = {"x-api-key": self.api_key}
        params = {"location": location, "category": category}
        
        response = requests.get(
            f"{self.base_url}/events",
            headers=headers,
            params=params
        )
        return response.json()

# Usage
client = EventAPIClient(api_key="your_secret_api_key")
events = client.get_events("San Francisco", "tech")
print(f"Found {events['count']} events")
```

## Documentation

For complete API documentation, see [API_DOCUMENTATION.md](API_DOCUMENTATION.md)

## License

MIT License

## Support

For issues and questions, please open an issue on GitHub.

