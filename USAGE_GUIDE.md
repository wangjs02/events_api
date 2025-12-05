# Event API Service - 使用指南

## 📦 安装

上传到 PyPI 后，用户可以通过 pip 直接安装：

```bash
pip install event_api_service
```

---

## 🚀 快速开始

### 方式 1: 作为独立服务运行

#### 步骤 1: 创建项目目录

```bash
mkdir my_event_service
cd my_event_service
```

#### 步骤 2: 创建 `.env` 文件

**重要说明**：这里的 `API_KEY` 是**您自己设置的密钥**，用于保护您的 API 服务。您可以设置为任何安全的字符串。

```bash
# .env
# API Keys for Event Providers (可选配置)
TICKETMASTER_KEY=your_ticketmaster_api_key
MEETUP_TOKEN=your_meetup_oauth_token
ALLEVENTS_KEY=your_allevents_api_key
SERPAPI_KEY=your_serpapi_key
PREDICTHQ_TOKEN=your_predicthq_token

# 本服务的 API Key（必需）
# ⚠️ 这是您自己定义的密钥，可以是任何安全字符串
# 建议使用随机生成的长字符串，例如：
API_KEY=your_secret_api_key_12345
# 或使用 Python 生成：python -c "import secrets; print(secrets.token_urlsafe(32))"
```

**如何生成安全的 API Key：**

```bash
# 方法 1: 使用 Python
python -c "import secrets; print('API_KEY=' + secrets.token_urlsafe(32))"

# 方法 2: 使用 OpenSSL (Linux/Mac)
echo "API_KEY=$(openssl rand -hex 32)"

# 方法 3: 手动设置（简单但不够安全）
# API_KEY=MyCompany_EventAPI_2024_SecretKey
```

#### 步骤 3: 创建运行脚本 `run_server.py`

```python
from dotenv import load_dotenv
from event_api import create_app

# 加载环境变量
load_dotenv()

# 创建 Flask 应用
app = create_app()

if __name__ == '__main__':
    # 开发环境
    app.run(debug=True, host='0.0.0.0', port=5000)
    
    # 生产环境建议使用 Gunicorn:
    # gunicorn -w 4 -b 0.0.0.0:5000 "run_server:app"
```

#### 步骤 4: 运行服务

```bash
python run_server.py
```

服务将在 `http://localhost:5000` 启动。

---

### 方式 2: 作为 Python 库使用

如果您只想使用事件抓取功能，而不需要运行 API 服务：

```python
import os
from event_api.services.scraper import UnifiedEventService

# 设置 API Keys（可选）
os.environ['TICKETMASTER_KEY'] = 'your_key'
os.environ['MEETUP_TOKEN'] = 'your_token'

# 创建服务实例
event_service = UnifiedEventService()

# 获取活动
events = event_service.get_events(
    location_name="New York",
    category="music"
)

# 处理结果
for event in events:
    print(f"📅 {event['title']}")
    print(f"   📍 {event['location']}")
    print(f"   🔗 {event['url']}")
    print(f"   📌 来源: {event['source']}")
    print()
```

---

## 🔧 API 使用示例

### 使用 cURL

```bash
# 基础请求
curl -H "x-api-key: your_secret_api_key_12345" \
  "http://localhost:5000/api/v1/events?location=San Francisco&category=tech"

# 格式化输出
curl -H "x-api-key: your_secret_api_key_12345" \
  "http://localhost:5000/api/v1/events?location=New York&category=music" \
  | jq '.'
```

### 使用 Python Requests

```python
import requests

# API 配置
API_URL = "http://localhost:5000/api/v1"
API_KEY = "your_secret_api_key_12345"

# 发送请求
response = requests.get(
    f"{API_URL}/events",
    headers={"x-api-key": API_KEY},
    params={
        "location": "Los Angeles",
        "category": "sports"
    }
)

# 处理响应
if response.status_code == 200:
    data = response.json()
    print(f"找到 {data['count']} 个活动")
    
    for event in data['data']:
        print(f"\n标题: {event['title']}")
        print(f"时间: {event['start_time']}")
        print(f"地点: {event['location']}")
        print(f"来源: {event['source']}")
        print(f"链接: {event['url']}")
else:
    print(f"错误: {response.status_code}")
    print(response.json())
```

### 创建客户端类

```python
import requests
from typing import List, Dict, Optional

class EventAPIClient:
    """Event API Service 客户端"""
    
    def __init__(self, api_key: str, base_url: str = "http://localhost:5000/api/v1"):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {"x-api-key": api_key}
    
    def get_events(
        self, 
        location: str, 
        category: str,
        timeout: int = 30
    ) -> Optional[Dict]:
        """
        获取活动列表
        
        Args:
            location: 城市名称（如 "New York", "London"）
            category: 活动类别（如 "music", "tech", "sports"）
            timeout: 请求超时时间（秒）
        
        Returns:
            包含活动数据的字典，失败返回 None
        """
        try:
            response = requests.get(
                f"{self.base_url}/events",
                headers=self.headers,
                params={"location": location, "category": category},
                timeout=timeout
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"请求失败: {e}")
            return None
    
    def search_multiple_cities(
        self, 
        cities: List[str], 
        category: str
    ) -> Dict[str, List]:
        """
        在多个城市搜索活动
        
        Args:
            cities: 城市列表
            category: 活动类别
        
        Returns:
            字典，键为城市名，值为活动列表
        """
        results = {}
        for city in cities:
            data = self.get_events(city, category)
            if data and data.get('status') == 'success':
                results[city] = data['data']
            else:
                results[city] = []
        return results


# 使用示例
if __name__ == "__main__":
    client = EventAPIClient(api_key="your_secret_api_key_12345")
    
    # 单个城市搜索
    events = client.get_events("Seattle", "tech")
    if events:
        print(f"Seattle 找到 {events['count']} 个科技活动")
    
    # 多城市搜索
    cities = ["New York", "San Francisco", "Chicago"]
    results = client.search_multiple_cities(cities, "music")
    
    for city, events in results.items():
        print(f"\n{city}: {len(events)} 个音乐活动")
```

---

## 🌐 部署到生产环境

### 使用 Gunicorn（推荐）

```bash
# 安装 Gunicorn
pip install gunicorn

# 运行（4个工作进程）
gunicorn -w 4 -b 0.0.0.0:5000 "run_server:app"

# 后台运行
gunicorn -w 4 -b 0.0.0.0:5000 "run_server:app" --daemon
```

### 使用 Docker

创建 `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY run_server.py .
COPY .env .

# 暴露端口
EXPOSE 5000

# 运行应用
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "run_server:app"]
```

创建 `requirements.txt`:

```
event_api_service
gunicorn
python-dotenv
```

构建和运行：

```bash
# 构建镜像
docker build -t event-api-service .

# 运行容器
docker run -d -p 5000:5000 --env-file .env event-api-service
```

### 使用 Docker Compose

创建 `docker-compose.yml`:

```yaml
version: '3.8'

services:
  event-api:
    build: .
    ports:
      - "5000:5000"
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - FLASK_ENV=production
```

运行：

```bash
docker-compose up -d
```

---

## 📊 监控和日志

### 添加日志记录

```python
import logging
from dotenv import load_dotenv
from event_api import create_app

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('event_api.log'),
        logging.StreamHandler()
    ]
)

load_dotenv()
app = create_app()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 🔐 安全建议

### 1. 保护 API Key

```python
# ❌ 不要硬编码
API_KEY = "secret-key-123"

# ✅ 使用环境变量
import os
API_KEY = os.getenv('API_KEY')
```

### 2. 使用 HTTPS

在生产环境中，使用 Nginx 作为反向代理并启用 HTTPS：

```nginx
server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 3. 限制访问

可以在 Nginx 中添加 IP 白名单：

```nginx
location / {
    allow 192.168.1.0/24;
    deny all;
    proxy_pass http://localhost:5000;
}
```

---

## 🧪 测试

### 单元测试示例

```python
import unittest
from event_api import create_app

class TestEventAPI(unittest.TestCase):
    def setUp(self):
        self.app = create_app()
        self.client = self.app.test_client()
        self.api_key = "test-api-key"
    
    def test_get_events_success(self):
        response = self.client.get(
            '/api/v1/events?location=NewYork&category=tech',
            headers={'x-api-key': self.api_key}
        )
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
        self.assertEqual(data['status'], 'success')
    
    def test_missing_api_key(self):
        response = self.client.get(
            '/api/v1/events?location=NewYork&category=tech'
        )
        self.assertEqual(response.status_code, 401)

if __name__ == '__main__':
    unittest.main()
```

---

## 📚 完整示例项目

创建一个完整的项目结构：

```
my_event_project/
├── .env                    # 环境变量
├── run_server.py          # 服务器启动脚本
├── requirements.txt       # 依赖列表
├── client.py              # 客户端示例
├── Dockerfile             # Docker 配置
└── docker-compose.yml     # Docker Compose 配置
```

这样用户就可以：
1. 安装包：`pip install event_api_service`
2. 配置环境变量
3. 运行服务或直接使用库
4. 通过 API 调用获取活动数据

---

## 🆘 故障排除

### 问题 1: 导入错误

```python
# 如果遇到 ImportError
# 确保已安装包
pip install event_api_service

# 或从源码安装
pip install -e .
```

### 问题 2: API Key 无效

检查 `.env` 文件中的 `API_KEY` 是否正确设置。

### 问题 3: 没有返回数据

确保至少配置了一个数据源的 API Key（如 `TICKETMASTER_KEY`）。

---

## 📞 支持

- GitHub Issues: [项目地址]
- 文档: 查看 `API_DOCUMENTATION.md`
- Email: contact@example.com
