# Neural Roots AI - System Design Document

## 1. System Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           NEURAL ROOTS AI PLATFORM                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   FRONTEND      │    │    BACKEND      │    │   EXTERNAL      │         │
│  │   (Next.js)     │◄──►│   (FastAPI)     │◄──►│   SERVICES      │         │
│  │                 │    │                 │    │                 │         │
│  │ • Dashboard     │    │ • Multi-Agent   │    │ • MongoDB       │         │
│  │ • Real-time UI  │    │   AI System     │    │ • Twilio        │         │
│  │ • Maps          │    │ • REST APIs     │    │ • Weather APIs  │         │
│  │ • Analytics     │    │ • IoT Ingestion │    │ • ESP32 Devices │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Multi-Agent AI Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW ORCHESTRATOR                                │
│                     (Central Coordination Hub)                              │
└─────────┬─────────────────────────────────────────────────────────┬─────────┘
          │                                                         │
    ┌─────▼──────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────▼──┐
    │ FRESHNESS  │  │   MARKET    │  │  LOGISTICS  │  │  WEATHER   │
    │   AGENT    │  │   AGENT     │  │   AGENT     │  │   AGENT    │
    │            │  │             │  │             │  │            │
    │ • IoT Data │  │ • MongoDB   │  │ • Driver    │  │ • Forecast │
    │ • Score    │  │   Query     │  │   Scoring   │  │ • Impact   │
    │ • Level    │  │ • Pricing   │  │ • Route     │  │   Analysis │
    │ • Predict  │  │   Strategy  │  │   Optimize  │  │ • Risk     │
    └────────────┘  └─────────────┘  └─────────────┘  └────────────┘
          │                │                │                │
          └────────────────▼────────────────▼────────────────┘
                           │
                    ┌──────▼──────┐
                    │  SYNTHESIS  │
                    │ & FINAL     │
                    │ ASSESSMENT  │
                    └─────────────┘
```

## 2. Component Design

### 2.1 Backend Architecture

#### 2.1.1 FastAPI Application Structure

```
backend/app/
├── main.py                 # Application entry point
├── core/
│   ├── config.py          # Configuration management
│   ├── database.py        # MongoDB connection
│   └── graph.py           # Workflow graph execution
├── agents/
│   ├── workflow_orchestrator.py  # Central coordinator
│   ├── freshness_agent.py        # Freshness analysis
│   ├── market_agent.py           # Market intelligence
│   ├── logistics_agent.py        # Logistics optimization
│   └── weather_agent.py          # Weather impact
├── routers/
│   ├── workflow_assessment.py    # Main API endpoints
│   ├── iot_ingest.py            # IoT data ingestion
│   └── whatsapp_webhook.py      # WhatsApp integration
├── models/
│   └── schemas.py               # Pydantic data models
└── services/
    ├── twilio_client.py         # WhatsApp service
    ├── weather_api.py           # Weather service
    └── validation.py            # Data validation
```

#### 2.1.2 Agent Design Pattern

```python
class BaseAgent:
    """Abstract base class for all agents"""
    
    def __init__(self):
        self.agent_id = str(uuid4())
        self.execution_history = []
    
    async def execute(self, input_data: Dict) -> Dict:
        """Execute agent logic and return results"""
        pass
    
    def validate_input(self, data: Dict) -> bool:
        """Validate input data"""
        pass
    
    def log_execution(self, input_data: Dict, output: Dict):
        """Log execution for audit trail"""
        pass
```

#### 2.1.3 Workflow Orchestration Design

```python
class WorkflowOrchestrator:
    """
    5-Stage Pipeline:
    1. Freshness Analysis    (200-300ms)
    2. Market Analysis       (100-200ms) 
    3. Logistics Analysis    (150-250ms)
    4. Weather Analysis      (100-150ms)
    5. Synthesis            (50-100ms)
    Total: 600-1000ms
    """
    
    async def execute_workflow(self, crop_data: Dict) -> Dict:
        # Stage 1: Freshness Assessment
        freshness_result = await self.freshness_agent.execute(crop_data)
        
        # Stage 2: Market Intelligence
        market_result = await self.market_agent.execute({
            **crop_data, 
            "freshness_score": freshness_result["freshness_score"]
        })
        
        # Stage 3: Logistics Optimization
        logistics_result = await self.logistics_agent.execute({
            **crop_data,
            "freshness_level": freshness_result["freshness_level"]
        })
        
        # Stage 4: Weather Impact
        weather_result = await self.weather_agent.execute(crop_data)
        
        # Stage 5: Synthesis
        final_result = self.synthesize_results(
            freshness_result, market_result, 
            logistics_result, weather_result
        )
        
        return final_result
```

### 2.2 Frontend Architecture

#### 2.2.1 Next.js Application Structure

```
frontend/
├── app/
│   ├── layout.tsx         # Root layout with global styles
│   ├── page.tsx           # Main dashboard page
│   └── globals.css        # Tailwind CSS + custom styles
├── components/
│   ├── CommandCenter.tsx      # Executive dashboard
│   ├── FleetModule.tsx        # Driver tracking
│   ├── LeafletMap.tsx         # Interactive mapping
│   ├── FarmersModule.tsx      # Farmer management
│   ├── WholesalersModule.tsx  # Wholesaler management
│   ├── MarketTerminal.tsx     # Market price management
│   ├── TransportAnalytics.tsx # Logistics analytics
│   ├── Sidebar.tsx            # Navigation sidebar
│   ├── SimulationOverlay.tsx  # Live notifications
│   └── ui/                    # Reusable UI components
├── hooks/
│   ├── useBackendData.ts      # Backend API integration
│   ├── useLocalStorage.ts     # Local state persistence
│   └── use-toast.ts           # Toast notifications
├── data/
│   └── mockData.ts            # Mock data definitions
└── types/
    └── backend.ts             # TypeScript type definitions
```

#### 2.2.2 State Management Design

```typescript
// Central State Management in page.tsx
interface AppState {
  farmers: Farmer[];
  drivers: Driver[];
  marketItems: MarketItem[];
  wholesalers: Wholesaler[];
  activeTab: string;
  selectedDriver: Driver | null;
  isSimulationRunning: boolean;
}

// State Update Pattern
const [farmers, setFarmers] = useState<Farmer[]>(farmersData);
const [drivers, setDrivers] = useState<Driver[]>(driversData);

// Prop Drilling Pattern
<CommandCenter 
  farmers={farmers}
  drivers={drivers}
  marketItems={marketItems}
  wholesalers={wholesalers}
  onNavigate={handleTabChange}
/>
```

#### 2.2.3 Real-time Updates Design

```typescript
// Driver Position Simulation (2-second intervals)
useEffect(() => {
  const interval = setInterval(() => {
    setDrivers(prev => prev.map(driver => ({
      ...driver,
      lat: driver.lat + (Math.random() - 0.5) * 0.01,
      lng: driver.lng + (Math.random() - 0.5) * 0.01
    })));
  }, 2000);
  
  return () => clearInterval(interval);
}, []);

// Live Notifications (15-second intervals)
useEffect(() => {
  const interval = setInterval(() => {
    const activities = generateRandomActivities();
    setRecentActivities(activities);
  }, 15000);
  
  return () => clearInterval(interval);
}, []);
```

## 3. Database Design

### 3.1 MongoDB Collections Schema

#### 3.1.1 Wholesalers Collection
```javascript
{
  _id: ObjectId,
  crop_name: String,           // "tomato", "onion", etc.
  location: String,            // "Mumbai", "Pune", etc.
  price: Number,               // Price per kg in INR
  demand: String,              // "HIGH", "MEDIUM", "LOW"
  supply: String,              // "HIGH", "MEDIUM", "LOW"
  timestamp: Date,             // Last updated
  wholesaler_id: String,       // Unique wholesaler identifier
  warehouse_capacity: Number,   // Capacity in kg
  contact_info: {
    phone: String,
    email: String,
    address: String
  }
}
```

#### 3.1.2 Drivers Collection
```javascript
{
  _id: ObjectId,
  name: String,                // Driver name
  vehicle_type: String,        // "cold_chain", "refrigerated", "standard"
  capacity: Number,            // Vehicle capacity in kg
  rating: Number,              // 0-5 star rating
  status: String,              // "available", "busy", "offline"
  location: String,            // Current location
  available_hours: Number,     // Hours available per day
  capabilities: [String],      // ["long_distance", "fragile_goods"]
  phone: String,               // Contact number
  vehicle_id: String,          // Vehicle registration
  license_info: {
    license_number: String,
    expiry_date: Date,
    verified: Boolean
  }
}
```

#### 3.1.3 IoT Logs Collection
```javascript
{
  _id: ObjectId,
  farmer_id: String,           // Farmer identifier
  device_id: String,           // ESP32 device ID
  temperature: Number,         // Temperature in Celsius
  humidity: Number,            // Humidity percentage
  image_url: String,           // Path to uploaded image
  timestamp: Date,             // Sensor reading time
  created_at: Date,            // Record creation time
  location: {
    latitude: Number,
    longitude: Number
  },
  metadata: {
    battery_level: Number,
    signal_strength: Number,
    firmware_version: String
  }
}
```

#### 3.1.4 Weather Collection
```javascript
{
  _id: ObjectId,
  location: String,            // Location name
  timestamp: Date,             // Forecast time
  temperature: Number,         // Temperature in Celsius
  humidity: Number,            // Humidity percentage
  precipitation: Number,       // Rainfall in mm
  wind_speed: Number,          // Wind speed in km/h
  condition: String,           // "sunny", "rainy", "cloudy"
  forecast_hours: Number,      // Hours ahead forecast
  source: String,              // "openweather", "simulated"
  accuracy: Number             // Forecast accuracy percentage
}
```

### 3.2 Database Indexing Strategy

```javascript
// Performance Optimization Indexes
db.wholesalers.createIndex({ "crop_name": 1, "location": 1 });
db.drivers.createIndex({ "status": 1, "vehicle_type": 1 });
db.iot_logs.createIndex({ "farmer_id": 1, "timestamp": -1 });
db.weather.createIndex({ "location": 1, "timestamp": -1 });

// Compound Indexes for Complex Queries
db.wholesalers.createIndex({ 
  "crop_name": 1, 
  "demand": 1, 
  "timestamp": -1 
});

db.drivers.createIndex({ 
  "location": 1, 
  "capacity": 1, 
  "rating": -1 
});
```

## 4. API Design

### 4.1 RESTful API Endpoints

#### 4.1.1 Workflow Assessment APIs
```
POST /api/workflow/assess-freshness
Content-Type: application/json

Request:
{
  "crop_data": {
    "crop_name": "tomato",
    "temperature": 25.5,
    "humidity": 70.0,
    "age_hours": 12.0,
    "quantity": 100.0
  },
  "target_location": "Mumbai",
  "urgency": "MEDIUM"
}

Response:
{
  "workflow_id": "wf_123456789",
  "execution_time_ms": 750,
  "final_assessment": {
    "freshness_score": 75.5,
    "freshness_level": "GOOD",
    "adjusted_score": 73.2,
    "confidence": 0.85
  },
  "agent_results": {
    "freshness_agent": { ... },
    "market_agent": { ... },
    "logistics_agent": { ... },
    "weather_agent": { ... }
  },
  "recommendations": [
    {
      "priority": "HIGH",
      "action": "Use refrigerated transport",
      "reason": "Freshness level requires temperature control"
    }
  ]
}
```

#### 4.1.2 IoT Data Ingestion APIs
```
POST /api/iot/esp32/data
Content-Type: multipart/form-data

Request:
farmer_id: "F001"
temperature: 24.5
humidity: 68.0
image: [binary file data]
timestamp: "2026-01-24T10:30:00Z"

Response:
{
  "status": "success",
  "message": "Data saved & AI triggered",
  "data_id": "iot_123456789",
  "image_url": "/uploads/F001_abc123_capture.jpg",
  "workflow_triggered": true
}
```

### 4.2 Error Handling Design

```python
class APIErrorHandler:
    """Standardized error handling across all endpoints"""
    
    @staticmethod
    def validation_error(field: str, message: str):
        return {
            "error": "VALIDATION_ERROR",
            "field": field,
            "message": message,
            "status_code": 400
        }
    
    @staticmethod
    def agent_execution_error(agent: str, error: str):
        return {
            "error": "AGENT_EXECUTION_ERROR",
            "agent": agent,
            "message": error,
            "status_code": 500
        }
    
    @staticmethod
    def database_error(operation: str):
        return {
            "error": "DATABASE_ERROR",
            "operation": operation,
            "message": "Database operation failed",
            "status_code": 503
        }
```

## 5. Security Design

### 5.1 Authentication & Authorization

```python
# Future Implementation - JWT Token-based Auth
class SecurityManager:
    def __init__(self):
        self.secret_key = os.getenv("JWT_SECRET_KEY")
        self.algorithm = "HS256"
    
    def create_access_token(self, user_id: str, role: str):
        payload = {
            "user_id": user_id,
            "role": role,
            "exp": datetime.utcnow() + timedelta(hours=24)
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
    
    def verify_token(self, token: str):
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
            return payload
        except jwt.ExpiredSignatureError:
            raise HTTPException(401, "Token expired")
```

### 5.2 Data Validation & Sanitization

```python
class CropDataValidator(BaseModel):
    crop_name: str = Field(
        ..., 
        min_length=2, 
        max_length=50, 
        pattern=r"^[A-Za-z_\- ]+$"
    )
    temperature: float = Field(
        ..., 
        ge=-10, 
        le=60, 
        description="Temperature in Celsius"
    )
    humidity: float = Field(
        ..., 
        ge=0, 
        le=100, 
        description="Humidity percentage"
    )
    age_hours: Optional[float] = Field(0, ge=0)
    quantity: Optional[float] = Field(10, gt=0)
    
    class Config:
        extra = "forbid"  # Reject unknown fields
```

### 5.3 CORS & Security Headers

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "http://127.0.0.1:3000",
        "https://neuralroots.com"  # Production domain
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)

# Security Headers Middleware
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    return response
```

## 6. Performance Design

### 6.1 Caching Strategy

```python
from functools import lru_cache
import redis

class CacheManager:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
    
    @lru_cache(maxsize=1000)
    def get_market_data(self, crop_name: str, location: str):
        """Cache market data for 5 minutes"""
        cache_key = f"market:{crop_name}:{location}"
        cached_data = self.redis_client.get(cache_key)
        
        if cached_data:
            return json.loads(cached_data)
        
        # Fetch from database
        data = fetch_market_data_from_db(crop_name, location)
        
        # Cache for 5 minutes
        self.redis_client.setex(cache_key, 300, json.dumps(data))
        return data
```

### 6.2 Database Query Optimization

```python
class OptimizedQueries:
    @staticmethod
    async def get_available_drivers(location: str, capacity: float):
        """Optimized driver query with projection"""
        pipeline = [
            {
                "$match": {
                    "status": "available",
                    "capacity": {"$gte": capacity * 1.1}  # 10% buffer
                }
            },
            {
                "$project": {
                    "name": 1,
                    "vehicle_type": 1,
                    "capacity": 1,
                    "rating": 1,
                    "location": 1
                }
            },
            {
                "$sort": {"rating": -1, "capacity": 1}
            },
            {
                "$limit": 10
            }
        ]
        
        return await db.drivers.aggregate(pipeline).to_list(length=10)
```

### 6.3 Async Processing Design

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

class AsyncWorkflowExecutor:
    def __init__(self):
        self.executor = ThreadPoolExecutor(max_workers=4)
    
    async def execute_parallel_agents(self, crop_data: Dict):
        """Execute independent agents in parallel"""
        
        # Create tasks for parallel execution
        tasks = [
            self.freshness_agent.execute(crop_data),
            self.market_agent.execute(crop_data),
            self.weather_agent.execute(crop_data)
        ]
        
        # Execute in parallel
        freshness_result, market_result, weather_result = await asyncio.gather(*tasks)
        
        # Logistics agent depends on freshness result
        logistics_result = await self.logistics_agent.execute({
            **crop_data,
            "freshness_level": freshness_result["freshness_level"]
        })
        
        return {
            "freshness": freshness_result,
            "market": market_result,
            "logistics": logistics_result,
            "weather": weather_result
        }
```

## 7. Integration Design

### 7.1 ESP32 IoT Integration

```cpp
// ESP32 Firmware Design Pattern
class IoTDataCollector {
private:
    WiFiClient wifiClient;
    HTTPClient httpClient;
    String serverURL = "http://backend:8000/api/iot/esp32/data";
    
public:
    void sendSensorData(float temp, float humidity, String imageBase64) {
        HTTPClient http;
        http.begin(serverURL);
        http.addHeader("Content-Type", "multipart/form-data");
        
        String payload = createMultipartPayload(temp, humidity, imageBase64);
        
        int httpResponseCode = http.POST(payload);
        
        if (httpResponseCode == 200) {
            Serial.println("Data sent successfully");
        } else {
            Serial.printf("Error: %d\n", httpResponseCode);
            // Implement retry logic
        }
        
        http.end();
    }
};
```

### 7.2 WhatsApp Integration Design

```python
class WhatsAppService:
    def __init__(self):
        self.client = Client(
            account_sid=os.getenv("TWILIO_ACCOUNT_SID"),
            auth_token=os.getenv("TWILIO_AUTH_TOKEN")
        )
        self.whatsapp_number = os.getenv("TWILIO_WHATSAPP_NUMBER")
    
    async def process_farmer_message(self, from_number: str, message_body: str):
        """Process incoming farmer message and trigger workflow"""
        
        # Parse message for crop data
        crop_data = self.parse_crop_message(message_body)
        
        if crop_data:
            # Execute workflow
            result = await self.orchestrator.execute_workflow(crop_data)
            
            # Send response
            response_message = self.format_assessment_response(result)
            await self.send_message(from_number, response_message)
        else:
            # Send help message
            await self.send_message(from_number, self.get_help_message())
    
    def parse_crop_message(self, message: str) -> Dict:
        """Parse farmer message to extract crop data"""
        # Example: "Tomato 25C 70% 12hours 100kg"
        pattern = r"(\w+)\s+(\d+)C\s+(\d+)%\s+(\d+)hours\s+(\d+)kg"
        match = re.match(pattern, message, re.IGNORECASE)
        
        if match:
            return {
                "crop_name": match.group(1).lower(),
                "temperature": float(match.group(2)),
                "humidity": float(match.group(3)),
                "age_hours": float(match.group(4)),
                "quantity": float(match.group(5))
            }
        return None
```

### 7.3 Weather API Integration

```python
class WeatherAPIService:
    def __init__(self):
        self.api_key = os.getenv("OPENWEATHER_API_KEY")
        self.base_url = "http://api.openweathermap.org/data/2.5"
    
    async def get_weather_forecast(self, location: str, hours: int = 24):
        """Fetch weather forecast for specified location and duration"""
        
        url = f"{self.base_url}/forecast"
        params = {
            "q": location,
            "appid": self.api_key,
            "units": "metric",
            "cnt": hours // 3  # 3-hour intervals
        }
        
        async with httpx.AsyncClient() as client:
            response = await client.get(url, params=params)
            
            if response.status_code == 200:
                data = response.json()
                return self.parse_weather_data(data)
            else:
                # Fallback to simulated data
                return self.generate_simulated_weather(location, hours)
    
    def parse_weather_data(self, api_data: Dict) -> List[Dict]:
        """Parse OpenWeatherMap API response"""
        forecasts = []
        
        for item in api_data["list"]:
            forecast = {
                "timestamp": datetime.fromtimestamp(item["dt"]),
                "temperature": item["main"]["temp"],
                "humidity": item["main"]["humidity"],
                "precipitation": item.get("rain", {}).get("3h", 0),
                "wind_speed": item["wind"]["speed"] * 3.6,  # m/s to km/h
                "condition": item["weather"][0]["main"].lower()
            }
            forecasts.append(forecast)
        
        return forecasts
```

## 8. Deployment Design

### 8.1 Docker Configuration

```dockerfile
# Backend Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/v1/health || exit 1

# Start application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```dockerfile
# Frontend Dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/api/health || exit 1

# Start application
CMD ["npm", "start"]
```

### 8.2 Docker Compose Configuration

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - MONGODB_URL=mongodb://mongo:27017
      - DB_NAME=neural_roots
      - TWILIO_ACCOUNT_SID=${TWILIO_ACCOUNT_SID}
      - TWILIO_AUTH_TOKEN=${TWILIO_AUTH_TOKEN}
    depends_on:
      - mongo
      - redis
    volumes:
      - ./backend/app/uploads:/app/uploads
    networks:
      - neural-roots-network

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://backend:8000
    depends_on:
      - backend
    networks:
      - neural-roots-network

  mongo:
    image: mongo:6.0
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_DATABASE=neural_roots
    volumes:
      - mongo_data:/data/db
      - ./scripts/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    networks:
      - neural-roots-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - neural-roots-network

volumes:
  mongo_data:
  redis_data:

networks:
  neural-roots-network:
    driver: bridge
```

### 8.3 Kubernetes Deployment (Future)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: neural-roots-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: neural-roots-backend
  template:
    metadata:
      labels:
        app: neural-roots-backend
    spec:
      containers:
      - name: backend
        image: neural-roots/backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: MONGODB_URL
          valueFrom:
            secretKeyRef:
              name: neural-roots-secrets
              key: mongodb-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/v1/health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/v1/health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 9. Monitoring & Observability Design

### 9.1 Logging Strategy

```python
import logging
import structlog
from pythonjsonlogger import jsonlogger

class LoggingConfig:
    @staticmethod
    def setup_logging():
        # Structured logging configuration
        structlog.configure(
            processors=[
                structlog.stdlib.filter_by_level,
                structlog.stdlib.add_logger_name,
                structlog.stdlib.add_log_level,
                structlog.stdlib.PositionalArgumentsFormatter(),
                structlog.processors.TimeStamper(fmt="iso"),
                structlog.processors.StackInfoRenderer(),
                structlog.processors.format_exc_info,
                structlog.processors.UnicodeDecoder(),
                structlog.processors.JSONRenderer()
            ],
            context_class=dict,
            logger_factory=structlog.stdlib.LoggerFactory(),
            wrapper_class=structlog.stdlib.BoundLogger,
            cache_logger_on_first_use=True,
        )

# Usage in agents
logger = structlog.get_logger(__name__)

class FreshnessAgent:
    async def execute(self, crop_data: Dict) -> Dict:
        logger.info(
            "freshness_agent_execution_started",
            crop_name=crop_data.get("crop_name"),
            temperature=crop_data.get("temperature"),
            execution_id=str(uuid4())
        )
        
        try:
            result = await self._calculate_freshness(crop_data)
            
            logger.info(
                "freshness_agent_execution_completed",
                freshness_score=result["freshness_score"],
                execution_time_ms=result["execution_time_ms"]
            )
            
            return result
            
        except Exception as e:
            logger.error(
                "freshness_agent_execution_failed",
                error=str(e),
                crop_data=crop_data
            )
            raise
```

### 9.2 Metrics Collection

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server

class MetricsCollector:
    def __init__(self):
        # Counters
        self.workflow_executions = Counter(
            'workflow_executions_total',
            'Total number of workflow executions',
            ['status', 'crop_type']
        )
        
        self.api_requests = Counter(
            'api_requests_total',
            'Total number of API requests',
            ['endpoint', 'method', 'status_code']
        )
        
        # Histograms
        self.workflow_duration = Histogram(
            'workflow_duration_seconds',
            'Workflow execution duration',
            ['crop_type']
        )
        
        self.agent_duration = Histogram(
            'agent_execution_duration_seconds',
            'Individual agent execution duration',
            ['agent_name']
        )
        
        # Gauges
        self.active_iot_devices = Gauge(
            'active_iot_devices',
            'Number of active IoT devices'
        )
        
        self.freshness_score_avg = Gauge(
            'freshness_score_average',
            'Average freshness score',
            ['crop_type']
        )

# Usage in workflow orchestrator
metrics = MetricsCollector()

class WorkflowOrchestrator:
    async def execute_workflow(self, crop_data: Dict) -> Dict:
        crop_type = crop_data.get("crop_name", "unknown")
        
        with metrics.workflow_duration.labels(crop_type=crop_type).time():
            try:
                result = await self._execute_agents(crop_data)
                
                metrics.workflow_executions.labels(
                    status="success",
                    crop_type=crop_type
                ).inc()
                
                metrics.freshness_score_avg.labels(
                    crop_type=crop_type
                ).set(result["final_assessment"]["freshness_score"])
                
                return result
                
            except Exception as e:
                metrics.workflow_executions.labels(
                    status="error",
                    crop_type=crop_type
                ).inc()
                raise
```

### 9.3 Health Checks Design

```python
from fastapi import status
from typing import Dict, Any

class HealthCheckService:
    def __init__(self, db, redis_client):
        self.db = db
        self.redis_client = redis_client
    
    async def get_health_status(self) -> Dict[str, Any]:
        """Comprehensive health check"""
        
        health_status = {
            "status": "healthy",
            "timestamp": datetime.utcnow().isoformat(),
            "version": "1.0.0",
            "checks": {}
        }
        
        # Database connectivity
        try:
            await self.db.command("ping")
            health_status["checks"]["database"] = {
                "status": "healthy",
                "response_time_ms": 10
            }
        except Exception as e:
            health_status["checks"]["database"] = {
                "status": "unhealthy",
                "error": str(e)
            }
            health_status["status"] = "unhealthy"
        
        # Redis connectivity
        try:
            self.redis_client.ping()
            health_status["checks"]["cache"] = {
                "status": "healthy",
                "response_time_ms": 5
            }
        except Exception as e:
            health_status["checks"]["cache"] = {
                "status": "unhealthy",
                "error": str(e)
            }
        
        # Agent health
        agent_health = await self._check_agent_health()
        health_status["checks"]["agents"] = agent_health
        
        if agent_health["status"] == "unhealthy":
            health_status["status"] = "unhealthy"
        
        return health_status
    
    async def _check_agent_health(self) -> Dict[str, Any]:
        """Check individual agent health"""
        
        test_data = {
            "crop_name": "tomato",
            "temperature": 25.0,
            "humidity": 70.0,
            "age_hours": 12.0,
            "quantity": 100.0
        }
        
        agent_results = {}
        overall_status = "healthy"
        
        agents = [
            ("freshness", self.freshness_agent),
            ("market", self.market_agent),
            ("logistics", self.logistics_agent),
            ("weather", self.weather_agent)
        ]
        
        for agent_name, agent in agents:
            try:
                start_time = time.time()
                await agent.execute(test_data)
                execution_time = (time.time() - start_time) * 1000
                
                agent_results[agent_name] = {
                    "status": "healthy",
                    "response_time_ms": round(execution_time, 2)
                }
                
            except Exception as e:
                agent_results[agent_name] = {
                    "status": "unhealthy",
                    "error": str(e)
                }
                overall_status = "unhealthy"
        
        return {
            "status": overall_status,
            "agents": agent_results
        }

# Health check endpoint
@router.get("/health")
async def health_check():
    health_service = HealthCheckService(get_database(), redis_client)
    health_status = await health_service.get_health_status()
    
    status_code = status.HTTP_200_OK if health_status["status"] == "healthy" else status.HTTP_503_SERVICE_UNAVAILABLE
    
    return JSONResponse(
        content=health_status,
        status_code=status_code
    )
```

## 10. Testing Strategy Design

### 10.1 Unit Testing Design

```python
import pytest
from unittest.mock import AsyncMock, MagicMock
from app.agents.freshness_agent import FreshnessAgent

class TestFreshnessAgent:
    @pytest.fixture
    def freshness_agent(self):
        return FreshnessAgent()
    
    @pytest.fixture
    def sample_crop_data(self):
        return {
            "crop_name": "tomato",
            "temperature": 25.0,
            "humidity": 70.0,
            "age_hours": 12.0,
            "quantity": 100.0
        }
    
    @pytest.mark.asyncio
    async def test_freshness_calculation_excellent(self, freshness_agent, sample_crop_data):
        """Test freshness calculation for excellent conditions"""
        
        # Modify data for excellent conditions
        sample_crop_data.update({
            "temperature": 22.0,  # Optimal temperature
            "humidity": 65.0,     # Optimal humidity
            "age_hours": 2.0      # Very fresh
        })
        
        result = await freshness_agent.execute(sample_crop_data)
        
        assert result["freshness_score"] >= 80
        assert result["freshness_level"] == "EXCELLENT"
        assert "recommendations" in result
        assert len(result["recommendations"]) > 0
    
    @pytest.mark.asyncio
    async def test_freshness_calculation_critical(self, freshness_agent, sample_crop_data):
        """Test freshness calculation for critical conditions"""
        
        # Modify data for critical conditions
        sample_crop_data.update({
            "temperature": 35.0,  # High temperature
            "humidity": 90.0,     # High humidity
            "age_hours": 48.0     # Old crop
        })
        
        result = await freshness_agent.execute(sample_crop_data)
        
        assert result["freshness_score"] <= 20
        assert result["freshness_level"] == "CRITICAL"
        assert any("immediate" in rec["action"].lower() for rec in result["recommendations"])
    
    def test_temperature_score_calculation(self, freshness_agent):
        """Test temperature scoring logic"""
        
        # Test optimal temperature (20-25°C for tomato)
        score = freshness_agent._calculate_temperature_score("tomato", 22.0)
        assert score >= 90
        
        # Test high temperature
        score = freshness_agent._calculate_temperature_score("tomato", 35.0)
        assert score <= 50
        
        # Test low temperature
        score = freshness_agent._calculate_temperature_score("tomato", 5.0)
        assert score <= 70
```

### 10.2 Integration Testing Design

```python
import pytest
from httpx import AsyncClient
from app.main import app

class TestWorkflowIntegration:
    @pytest.fixture
    async def client(self):
        async with AsyncClient(app=app, base_url="http://test") as ac:
            yield ac
    
    @pytest.fixture
    def mock_database(self, monkeypatch):
        """Mock database for testing"""
        mock_db = AsyncMock()
        
        # Mock wholesalers data
        mock_db.wholesalers.find.return_value.to_list = AsyncMock(return_value=[
            {
                "crop_name": "tomato",
                "location": "Mumbai",
                "price": 50.0,
                "demand": "HIGH",
                "supply": "MEDIUM"
            }
        ])
        
        # Mock drivers data
        mock_db.drivers.find.return_value.to_list = AsyncMock(return_value=[
            {
                "name": "Driver 1",
                "vehicle_type": "refrigerated",
                "capacity": 500,
                "rating": 4.5,
                "status": "available"
            }
        ])
        
        monkeypatch.setattr("app.core.database.get_database", lambda: mock_db)
        return mock_db
    
    @pytest.mark.asyncio
    async def test_full_workflow_execution(self, client, mock_database):
        """Test complete workflow execution"""
        
        request_data = {
            "crop_data": {
                "crop_name": "tomato",
                "temperature": 25.0,
                "humidity": 70.0,
                "age_hours": 12.0,
                "quantity": 100.0
            },
            "target_location": "Mumbai",
            "urgency": "MEDIUM"
        }
        
        response = await client.post("/api/workflow/assess-freshness", json=request_data)
        
        assert response.status_code == 200
        
        result = response.json()
        assert "workflow_id" in result
        assert "final_assessment" in result
        assert "agent_results" in result
        assert "recommendations" in result
        
        # Verify agent results structure
        assert "freshness_agent" in result["agent_results"]
        assert "market_agent" in result["agent_results"]
        assert "logistics_agent" in result["agent_results"]
        assert "weather_agent" in result["agent_results"]
        
        # Verify final assessment
        final_assessment = result["final_assessment"]
        assert 0 <= final_assessment["freshness_score"] <= 100
        assert final_assessment["freshness_level"] in ["EXCELLENT", "GOOD", "FAIR", "POOR", "CRITICAL"]
```

### 10.3 Load Testing Design

```python
import asyncio
import aiohttp
import time
from concurrent.futures import ThreadPoolExecutor

class LoadTestRunner:
    def __init__(self, base_url: str, concurrent_users: int = 100):
        self.base_url = base_url
        self.concurrent_users = concurrent_users
        self.results = []
    
    async def single_request(self, session: aiohttp.ClientSession, request_id: int):
        """Execute a single workflow request"""
        
        request_data = {
            "crop_data": {
                "crop_name": "tomato",
                "temperature": 25.0 + (request_id % 10),  # Vary temperature
                "humidity": 70.0 + (request_id % 20),     # Vary humidity
                "age_hours": 12.0 + (request_id % 24),    # Vary age
                "quantity": 100.0
            },
            "target_location": "Mumbai",
            "urgency": "MEDIUM"
        }
        
        start_time = time.time()
        
        try:
            async with session.post(
                f"{self.base_url}/api/workflow/assess-freshness",
                json=request_data
            ) as response:
                await response.json()
                
                end_time = time.time()
                response_time = (end_time - start_time) * 1000  # Convert to ms
                
                self.results.append({
                    "request_id": request_id,
                    "status_code": response.status,
                    "response_time_ms": response_time,
                    "success": response.status == 200
                })
                
        except Exception as e:
            end_time = time.time()
            response_time = (end_time - start_time) * 1000
            
            self.results.append({
                "request_id": request_id,
                "status_code": 0,
                "response_time_ms": response_time,
                "success": False,
                "error": str(e)
            })
    
    async def run_load_test(self, duration_seconds: int = 60):
        """Run load test for specified duration"""
        
        print(f"Starting load test: {self.concurrent_users} concurrent users for {duration_seconds} seconds")
        
        async with aiohttp.ClientSession() as session:
            start_time = time.time()
            request_id = 0
            
            while time.time() - start_time < duration_seconds:
                # Create batch of concurrent requests
                tasks = []
                
                for _ in range(self.concurrent_users):
                    task = self.single_request(session, request_id)
                    tasks.append(task)
                    request_id += 1
                
                # Execute batch
                await asyncio.gather(*tasks, return_exceptions=True)
                
                # Small delay between batches
                await asyncio.sleep(0.1)
        
        self.analyze_results()
    
    def analyze_results(self):
        """Analyze load test results"""
        
        if not self.results:
            print("No results to analyze")
            return
        
        total_requests = len(self.results)
        successful_requests = sum(1 for r in self.results if r["success"])
        failed_requests = total_requests - successful_requests
        
        response_times = [r["response_time_ms"] for r in self.results if r["success"]]
        
        if response_times:
            avg_response_time = sum(response_times) / len(response_times)
            min_response_time = min(response_times)
            max_response_time = max(response_times)
            
            # Calculate percentiles
            sorted_times = sorted(response_times)
            p50 = sorted_times[int(len(sorted_times) * 0.5)]
            p95 = sorted_times[int(len(sorted_times) * 0.95)]
            p99 = sorted_times[int(len(sorted_times) * 0.99)]
        else:
            avg_response_time = min_response_time = max_response_time = 0
            p50 = p95 = p99 = 0
        
        success_rate = (successful_requests / total_requests) * 100
        
        print("\n" + "="*50)
        print("LOAD TEST RESULTS")
        print("="*50)
        print(f"Total Requests: {total_requests}")
        print(f"Successful Requests: {successful_requests}")
        print(f"Failed Requests: {failed_requests}")
        print(f"Success Rate: {success_rate:.2f}%")
        print(f"\nResponse Times (ms):")
        print(f"  Average: {avg_response_time:.2f}")
        print(f"  Minimum: {min_response_time:.2f}")
        print(f"  Maximum: {max_response_time:.2f}")
        print(f"  50th Percentile: {p50:.2f}")
        print(f"  95th Percentile: {p95:.2f}")
        print(f"  99th Percentile: {p99:.2f}")
        print("="*50)

# Usage
if __name__ == "__main__":
    load_tester = LoadTestRunner("http://localhost:8000", concurrent_users=50)
    asyncio.run(load_tester.run_load_test(duration_seconds=120))
```

---

**Document Version**: 1.0  
**Last Updated**: January 24, 2026  
**Prepared By**: System Architecture Team  
**Approved By**: Technical Leadership