# Event API Service - API 文档

## 概述

Event API Service 是一个统一的事件聚合 API，可以从多个事件平台（Eventbrite、Meetup、AllEvents、Ticketmaster、SerpApi、PredictHQ）获取活动信息。

**基础 URL**: `http://localhost:5000/api/v1`（本地开发）

---

## 认证

所有 API 请求都需要在 HTTP 请求头中包含 API Key。

### 请求头格式

```
x-api-key: your_api_key_here
```

### 获取 API Key

1. 在 `.env` 文件中设置 `API_KEY` 变量
2. 默认测试 Key: `test_key_123`（仅用于开发环境）

### 示例

```bash
curl -H "x-api-key: test_key_123" "http://localhost:5000/api/v1/events?location=NewYork&category=Tech"
```

---

## 限流

为了保护服务稳定性，API 实施了以下限流策略：

- **每分钟**: 10 次请求
- **每小时**: 50 次请求
- **每天**: 200 次请求

超过限制将返回 `429 Too Many Requests` 错误。

---

## API 端点

### 1. 获取活动列表

获取指定地区和类别的活动信息。

#### 端点

```
GET /api/v1/events
```

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `location` | string | 是 | 城市名称（如：New York, London, Beijing） |
| `category` | string | 是 | 活动类别（如：Tech, Music, Sports, Art） |

#### 请求示例

```bash
# 基础请求
curl -H "x-api-key: test_key_123" \
  "http://localhost:5000/api/v1/events?location=NewYork&category=Tech"

# Python 请求
import requests

headers = {"x-api-key": "test_key_123"}
params = {
    "location": "New York",
    "category": "Tech"
}

response = requests.get(
    "http://localhost:5000/api/v1/events",
    headers=headers,
    params=params
)

data = response.json()
print(data)
```

#### 响应格式

**成功响应** (200 OK)

```json
{
  "status": "success",
  "count": 15,
  "data": [
    {
      "id": "meetup-12345",
      "title": "Tech Meetup: AI & Machine Learning",
      "description": "Join us for an evening of AI discussions...",
      "start_time": "2025-11-25T18:00:00",
      "url": "https://www.meetup.com/event/12345",
      "location": "New York Tech Hub",
      "source": "meetup"
    },
    {
      "id": "tm-67890",
      "title": "Music Festival 2025",
      "description": "Annual music festival featuring...",
      "start_time": "2025-12-01",
      "url": "https://www.ticketmaster.com/event/67890",
      "location": "Madison Square Garden",
      "source": "ticketmaster"
    }
  ]
}
```

**错误响应**

```json
// 缺少参数 (400 Bad Request)
{
  "error": "Missing required parameters: location, category"
}

// 未授权 (401 Unauthorized)
{
  "error": "Unauthorized. Invalid or missing API Key."
}

// 限流 (429 Too Many Requests)
{
  "error": "Rate limit exceeded"
}

// 服务器错误 (500 Internal Server Error)
{
  "error": "Error message details"
}
```

---

## 数据源

API 会从以下平台聚合数据：

| 平台 | 需要配置 | 说明 |
|------|----------|------|
| **Eventbrite** | 无需配置 | ✅ 使用目标搜索 API（支持主要城市） |
| **Ticketmaster** | `TICKETMASTER_KEY` | 音乐、体育、艺术等活动 |
| **Meetup** | `MEETUP_TOKEN` | 基于地理位置的活动搜索（GraphQL API） |
| **AllEvents** | `ALLEVENTS_KEY` | 全球活动聚合平台 |
| **SerpApi** | `SERPAPI_KEY` | Google Events 搜索结果 |
| **PredictHQ** | `PREDICTHQ_TOKEN` | 专业活动数据和预测 |

### Eventbrite 支持的城市

Eventbrite 目前支持以下美国主要城市（使用完整城市名称）：

- San Francisco
- New York
- Los Angeles
- Miami
- Chicago
- Seattle
- Boston

**注意**: 城市名称不区分大小写，空格会自动处理。例如 "New York"、"new york"、"NEW YORK" 都可以正常工作。

---

## 配置指南

### 1. 环境变量设置

创建 `.env` 文件（参考 `.env.example`）：

```bash
# 复制示例文件
cp .env.example .env

# 编辑 .env 文件，填入你的 API Keys
nano .env
```

### 2. 获取各平台 API Keys

#### Meetup
1. 访问 [Meetup OAuth](https://www.meetup.com/api/oauth/)
2. 创建应用并获取 OAuth Token
3. 设置 `MEETUP_TOKEN=your_token`

#### Ticketmaster
1. 访问 [Ticketmaster Developer Portal](https://developer.ticketmaster.com/)
2. 注册并创建应用
3. 获取 API Key
4. 设置 `TICKETMASTER_KEY=your_key`

#### AllEvents
1. 访问 [AllEvents API](https://allevents.in/api)
2. 申请 API 访问权限
3. 设置 `ALLEVENTS_KEY=your_key`

#### SerpApi
1. 访问 [SerpApi](https://serpapi.com/)
2. 注册账号
3. 获取 API Key
4. 设置 `SERPAPI_KEY=your_key`

#### PredictHQ
1. 访问 [PredictHQ](https://www.predicthq.com/)
2. 申请开发者账号
3. 获取 OAuth Token
4. 设置 `PREDICTHQ_TOKEN=your_token`

---

## 快速开始

### 安装

```bash
# 克隆或下载项目
cd event_API

# 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 安装依赖
pip install -e .
```

### 配置

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，填入你的 API Keys
# 至少配置一个数据源的 API Key
```

### 运行

```bash
# 启动服务器
python run.py

# 服务器将在 http://localhost:5000 运行
```

### 测试

```bash
# 运行测试脚本
python test_api.py

# 或使用 curl
curl -H "x-api-key: test_key_123" \
  "http://localhost:5000/api/v1/events?location=NewYork&category=Music"
```

---

## 使用示例

### Python

```python
import requests

class EventAPIClient:
    def __init__(self, api_key, base_url="http://localhost:5000/api/v1"):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {"x-api-key": api_key}
    
    def get_events(self, location, category):
        """获取活动列表"""
        params = {
            "location": location,
            "category": category
        }
        
        response = requests.get(
            f"{self.base_url}/events",
            headers=self.headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"API Error: {response.status_code} - {response.text}")

# 使用示例
client = EventAPIClient(api_key="test_key_123")

# 获取纽约的科技活动
events = client.get_events("New York", "Tech")
print(f"找到 {events['count']} 个活动")

for event in events['data']:
    print(f"- {event['title']} ({event['source']})")
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

class EventAPIClient {
    constructor(apiKey, baseUrl = 'http://localhost:5000/api/v1') {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
    }
    
    async getEvents(location, category) {
        try {
            const response = await axios.get(`${this.baseUrl}/events`, {
                headers: {
                    'x-api-key': this.apiKey
                },
                params: {
                    location: location,
                    category: category
                }
            });
            
            return response.data;
        } catch (error) {
            throw new Error(`API Error: ${error.response.status} - ${error.response.data}`);
        }
    }
}

// 使用示例
const client = new EventAPIClient('test_key_123');

client.getEvents('London', 'Music')
    .then(data => {
        console.log(`找到 ${data.count} 个活动`);
        data.data.forEach(event => {
            console.log(`- ${event.title} (${event.source})`);
        });
    })
    .catch(error => console.error(error));
```

### cURL

```bash
# 基础请求
curl -H "x-api-key: test_key_123" \
  "http://localhost:5000/api/v1/events?location=Beijing&category=Tech"

# 格式化输出
curl -H "x-api-key: test_key_123" \
  "http://localhost:5000/api/v1/events?location=Tokyo&category=Art" \
  | jq '.'

# 保存到文件
curl -H "x-api-key: test_key_123" \
  "http://localhost:5000/api/v1/events?location=Paris&category=Music" \
  -o events.json
```

---

## 错误处理

### HTTP 状态码

| 状态码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权（API Key 无效或缺失） |
| 429 | 请求过于频繁（超过限流） |
| 500 | 服务器内部错误 |

### 错误处理最佳实践

```python
import requests
import time

def get_events_with_retry(location, category, max_retries=3):
    """带重试机制的请求"""
    headers = {"x-api-key": "test_key_123"}
    params = {"location": location, "category": category}
    
    for attempt in range(max_retries):
        try:
            response = requests.get(
                "http://localhost:5000/api/v1/events",
                headers=headers,
                params=params,
                timeout=10
            )
            
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:
                # 限流，等待后重试
                wait_time = 2 ** attempt  # 指数退避
                print(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
                continue
            elif response.status_code == 401:
                raise Exception("Invalid API Key")
            else:
                response.raise_for_status()
                
        except requests.exceptions.Timeout:
            print(f"Timeout on attempt {attempt + 1}")
            if attempt == max_retries - 1:
                raise
        except requests.exceptions.RequestException as e:
            print(f"Request failed: {e}")
            if attempt == max_retries - 1:
                raise
    
    return None
```

---

## 常见问题

### Q: 为什么有些数据源没有返回结果？

A: 可能的原因：
1. 未配置对应平台的 API Key
2. API Key 无效或已过期
3. 该地区/类别在该平台上没有活动
4. API 配额已用完

### Q: 如何提高请求限制？

A: 修改 `src/event_api/limiter.py` 中的限流配置：

```python
limiter = Limiter(
    key_func=get_remote_address,
    default_limits=["500 per day", "100 per hour"],  # 修改这里
    storage_uri="memory://"
)
```

### Q: 支持哪些活动类别？

A: 不同的数据源支持不同的类别系统：

#### Eventbrite
- **类型**: 关键词搜索（无限制）
- **支持**: 任何文本关键词（tech, music, sports, food, art, business, etc.）
- **示例**: `category=tech`, `category=music`, `category=food`

#### Ticketmaster
- **类型**: 分层分类系统
- **主要类别** (Segments):
  - `Music` - 音乐活动
  - `Sports` - 体育赛事
  - `Arts` 或 `Arts & Theatre` - 艺术和戏剧
  - `Film` - 电影
  - `Miscellaneous` - 其他
- **子类别** (Genres): Rock, Pop, Jazz, Basketball, Football, etc.
- **注意**: 必须使用 Ticketmaster 的官方分类名称

#### 其他提供商
- **Meetup**: 关键词搜索
- **AllEvents**: 关键词搜索
- **SerpApi**: 关键词搜索
- **PredictHQ**: 关键词搜索

#### 推荐的通用类别

为了获得最佳结果（所有提供商都能返回数据），建议使用以下类别：

| 类别 | Eventbrite | Ticketmaster | 其他 |
|------|-----------|--------------|------|
| `music` | ✅ | ✅ Music | ✅ |
| `sports` | ✅ | ✅ Sports | ✅ |
| `arts` | ✅ | ✅ Arts | ✅ |
| `film` | ✅ | ✅ Film | ✅ |
| `tech` | ✅ | ❌ (无匹配) | ✅ |
| `food` | ✅ | ❌ (无匹配) | ✅ |
| `business` | ✅ | ❌ (无匹配) | ✅ |

**提示**: 使用 `music`, `sports`, `arts`, `film` 可以从所有提供商获得结果。

### Q: 如何部署到生产环境？

A: 参考以下步骤：

1. 使用生产级 WSGI 服务器（如 Gunicorn）
2. 配置 Nginx 反向代理
3. 使用 Redis 作为限流存储
4. 设置环境变量（不要使用 .env 文件）
5. 启用 HTTPS

```bash
# 安装 Gunicorn
pip install gunicorn

# 运行
gunicorn -w 4 -b 0.0.0.0:5000 "src.event_api.app:create_app()"
```

---

## 支持

如有问题或建议，请联系开发团队或提交 Issue。

## 许可证

MIT License
