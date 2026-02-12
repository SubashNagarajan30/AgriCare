# Agri-Aegis Disease Sentinel - Design Document

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────┐
│   Farmer    │
│  (WhatsApp) │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────────────────────┐
│                    Twilio WhatsApp API                  │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                     AWS API Gateway                     │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      AWS Lambda                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         WhatsApp Controller                      │  │
│  └────┬─────────────┬──────────────┬────────────┬───┘  │
│       │             │              │            │       │
│  ┌────▼────┐  ┌────▼─────┐  ┌────▼────┐  ┌───▼────┐  │
│  │ Disease │  │ Weather  │  │Supplier │  │  S3    │  │
│  │ Service │  │ Service  │  │ Service │  │Service │  │
│  └────┬────┘  └────┬─────┘  └────┬────┘  └───┬────┘  │
└───────┼────────────┼─────────────┼───────────┼────────┘
        │            │             │           │
        ▼            ▼             ▼           ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────┐
│SageMaker │  │IoT Core  │  │ DynamoDB │  │  S3  │
│ Endpoint │  │Weather   │  │Suppliers │  │Images│
└──────────┘  └──────────┘  └──────────┘  └──────┘
```

### 1.2 Component Architecture

#### 1.2.1 Presentation Layer
- **WhatsApp Interface**: User interaction point
- **Twilio API**: Message routing and media handling

#### 1.2.2 Application Layer
- **API Gateway**: Request routing and throttling
- **Lambda Functions**: Serverless compute
- **Controllers**: Business logic orchestration
- **Services**: Domain-specific operations

#### 1.2.3 Data Layer
- **S3**: Image storage
- **DynamoDB**: Supplier and disease data
- **SageMaker**: ML model hosting

#### 1.2.4 Integration Layer
- **IoT Core**: Weather sensor data
- **OpenWeather API**: Fallback weather data

## 2. Detailed Component Design

### 2.1 WhatsApp Controller

**Responsibilities:**
- Webhook verification
- Message parsing
- Response formatting
- Error handling

**Key Methods:**
```javascript
verifyWebhook(req, res)
handleIncomingMessage(req, res)
sendMessage(to, body)
formatResponse(disease, weather, suppliers)
```

**Flow:**
1. Receive webhook from Twilio
2. Extract message content and media
3. Validate message format
4. Orchestrate service calls
5. Format and send response

### 2.2 Disease Service

**Responsibilities:**
- Image download and validation
- S3 storage management
- SageMaker inference
- Disease information retrieval

**Key Methods:**
```javascript
identifyDisease(imageUrl)
downloadImage(url)
uploadToS3(buffer)
invokeSageMaker(imageBuffer)
```

**ML Model Integration:**
- Endpoint: SageMaker real-time inference
- Input: JPEG/PNG image buffer
- Output: Disease class + confidence score
- Fallback: Generic treatment advice

**Disease Database Schema:**
```javascript
{
  disease_id: string,
  name: string,
  treatment: string,
  products: [string],
  severity: enum('low', 'medium', 'high'),
  crops_affected: [string]
}
```

### 2.3 Weather Service

**Responsibilities:**
- IoT data retrieval
- Weather API fallback
- Alert generation
- Risk assessment

**Key Methods:**
```javascript
getLocalWeather(latitude, longitude)
getIoTWeatherData()
getWeatherAPI(lat, lon)
generateWeatherAlert(weather)
```

**Weather Data Schema:**
```javascript
{
  temperature: float,
  humidity: float,
  rainfall: float,
  wind_speed: float,
  timestamp: datetime,
  alert: string
}
```

**Alert Logic:**
- High humidity (>80%) → Fungal risk
- High temperature (>30°C) → Irrigation needed
- Heavy rainfall (>10mm) → Delay application

### 2.4 Supplier Service

**Responsibilities:**
- Location-based search
- Distance calculation
- Supplier ranking
- Contact information formatting

**Key Methods:**
```javascript
findNearbySuppliers(latitude, longitude, treatment)
calculateDistance(lat1, lon1, lat2, lon2)
```

**Supplier Database Schema:**
```javascript
{
  id: uuid,
  name: string,
  phone: string,
  latitude: float,
  longitude: float,
  products: [string],
  website: string,
  rating: float,
  verified: boolean
}
```

**Search Algorithm:**
1. Query all suppliers from DynamoDB
2. Calculate distance using Haversine formula
3. Filter suppliers within 50km
4. Sort by distance
5. Return top 5 results

## 3. Data Models

### 3.1 Request Flow Data

**Incoming Message:**
```javascript
{
  From: "whatsapp:+1234567890",
  To: "whatsapp:+14155238886",
  Body: "Help with my crop",
  MediaUrl0: "https://api.twilio.com/...",
  MediaContentType0: "image/jpeg",
  Latitude: "12.9716",
  Longitude: "77.5946"
}
```

**Disease Detection Response:**
```javascript
{
  disease: {
    name: "Late Blight",
    confidence: 92,
    treatment: "Apply copper-based fungicide...",
    products: ["Copper Oxychloride", "Mancozeb"],
    imageKey: "images/uuid.jpg"
  },
  weather: {
    temperature: 28.5,
    humidity: 85,
    rainfall: 5.2,
    alert: "High humidity increases fungal disease risk"
  },
  suppliers: [
    {
      name: "Green Valley Agro Store",
      phone: "+1234567890",
      distance: "3.2",
      link: "https://maps.google.com/?q=12.9716,77.5946"
    }
  ]
}
```

## 4. API Design

### 4.1 Webhook Endpoints

**POST /webhook/whatsapp**
- Purpose: Receive incoming WhatsApp messages
- Authentication: Twilio signature validation
- Rate Limit: 100 requests/minute
- Timeout: 10 seconds

**GET /webhook/whatsapp**
- Purpose: Webhook verification
- Parameters: hub.mode, hub.verify_token, hub.challenge

### 4.2 Internal Service APIs

**Disease Service:**
```javascript
identifyDisease(imageUrl: string): Promise<DiseaseResult>
```

**Weather Service:**
```javascript
getLocalWeather(lat: float, lon: float): Promise<WeatherData>
```

**Supplier Service:**
```javascript
findNearbySuppliers(lat: float, lon: float, treatment: string): Promise<Supplier[]>
```

## 5. Database Design

### 5.1 DynamoDB Tables

**Suppliers Table:**
```
Primary Key: id (String)
Attributes:
  - name (String)
  - phone (String)
  - latitude (Number)
  - longitude (Number)
  - products (List)
  - website (String)
  - rating (Number)
  - verified (Boolean)
  - created_at (String)
  - updated_at (String)

Indexes:
  - GSI: location-index (latitude, longitude)
```

**Diseases Table (Future):**
```
Primary Key: disease_id (String)
Attributes:
  - name (String)
  - treatment (String)
  - products (List)
  - severity (String)
  - crops_affected (List)
  - symptoms (List)
```

### 5.2 S3 Bucket Structure

```
agri-aegis-images/
├── images/
│   ├── 2024/
│   │   ├── 01/
│   │   │   ├── uuid1.jpg
│   │   │   └── uuid2.jpg
│   │   └── 02/
│   └── 2025/
└── models/
    └── plant-disease-v1/
```

## 6. Machine Learning Design

### 6.1 Model Architecture

**Model Type:** Convolutional Neural Network (CNN)
**Framework:** TensorFlow/PyTorch
**Base Model:** ResNet50 or EfficientNet
**Training Data:** PlantVillage dataset + custom data

**Input Specifications:**
- Image size: 224x224 pixels
- Format: RGB
- Normalization: ImageNet standards

**Output:**
- Disease class (categorical)
- Confidence score (0-1)
- Top 3 predictions

### 6.2 Model Deployment

**SageMaker Endpoint:**
- Instance type: ml.m5.large
- Auto-scaling: 1-5 instances
- Trigger: CPU > 70% or requests > 100/min

**Model Versioning:**
- Version format: v{major}.{minor}.{patch}
- A/B testing capability
- Rollback mechanism

### 6.3 Model Monitoring

**Metrics:**
- Inference latency
- Prediction confidence distribution
- Error rate
- Model drift detection

## 7. Security Design

### 7.1 Authentication & Authorization

**Twilio Webhook Validation:**
```javascript
const crypto = require('crypto');

function validateSignature(signature, url, params) {
  const data = Object.keys(params)
    .sort()
    .reduce((acc, key) => acc + key + params[key], url);
  
  const hmac = crypto
    .createHmac('sha1', process.env.TWILIO_AUTH_TOKEN)
    .update(Buffer.from(data, 'utf-8'))
    .digest('base64');
  
  return hmac === signature;
}
```

**AWS IAM Roles:**
- Lambda execution role with minimal permissions
- S3 read/write access scoped to specific bucket
- DynamoDB query access only
- SageMaker invoke endpoint permission

### 7.2 Data Protection

**Encryption:**
- At rest: S3 SSE-KMS, DynamoDB encryption
- In transit: TLS 1.2+
- PII handling: Anonymize phone numbers in logs

**Data Retention:**
- Images: 90 days
- Logs: 30 days
- User data: On-demand deletion

### 7.3 Rate Limiting

**API Gateway:**
- 100 requests/minute per user
- 1000 requests/minute global
- Burst: 200 requests

**Lambda Concurrency:**
- Reserved: 10 instances
- Max: 100 instances

## 8. Performance Design

### 8.1 Optimization Strategies

**Image Processing:**
- Compress images before S3 upload
- Resize to model input size
- Lazy loading for thumbnails

**Database Queries:**
- DynamoDB query optimization
- Caching frequent queries (ElastiCache future)
- Batch operations where possible

**ML Inference:**
- Model quantization for faster inference
- Batch prediction support
- Async processing for non-critical paths

### 8.2 Caching Strategy

**Response Caching:**
- Weather data: 15 minutes
- Supplier data: 1 hour
- Disease info: 24 hours

**CDN (Future):**
- CloudFront for static assets
- Edge caching for common responses

## 9. Error Handling Design

### 9.1 Error Categories

**User Errors:**
- Invalid image format
- Missing location data
- Poor image quality

**System Errors:**
- Service unavailable
- Timeout
- Rate limit exceeded

**External Errors:**
- Twilio API failure
- Weather API failure
- SageMaker endpoint down

### 9.2 Error Response Format

```javascript
{
  error: true,
  code: "IMAGE_INVALID",
  message: "Please send a clear photo of the leaf",
  retry: true,
  help: "Take photo in good lighting"
}
```

### 9.3 Fallback Mechanisms

1. **ML Model Failure:** Return generic advice
2. **Weather API Failure:** Skip weather alerts
3. **Supplier DB Failure:** Provide general supplier info
4. **Complete Failure:** Queue for manual review

## 10. Monitoring & Observability

### 10.1 Metrics

**Application Metrics:**
- Request count
- Response time (p50, p95, p99)
- Error rate
- Success rate

**Business Metrics:**
- Daily active users
- Disease detection count
- Supplier connection rate
- User satisfaction score

### 10.2 Logging

**CloudWatch Logs:**
```javascript
{
  timestamp: "2024-01-15T10:30:00Z",
  level: "INFO",
  service: "disease-service",
  action: "identify_disease",
  user_id: "hashed_phone",
  disease: "late_blight",
  confidence: 0.92,
  latency_ms: 2340
}
```

### 10.3 Alerting

**Critical Alerts:**
- Error rate > 5%
- Response time > 15 seconds
- SageMaker endpoint down
- DynamoDB throttling

**Warning Alerts:**
- Error rate > 2%
- Response time > 10 seconds
- Low confidence predictions > 30%

## 11. Deployment Design

### 11.1 CI/CD Pipeline

```
Code Push → GitHub
    ↓
GitHub Actions
    ↓
Run Tests
    ↓
Build Lambda Package
    ↓
Deploy to Dev
    ↓
Integration Tests
    ↓
Deploy to Prod
    ↓
Smoke Tests
```

### 11.2 Environment Strategy

**Development:**
- Separate AWS account
- Mock external services
- Reduced instance sizes

**Production:**
- Multi-AZ deployment
- Auto-scaling enabled
- Full monitoring

### 11.3 Rollback Strategy

- Blue-green deployment
- Lambda version aliases
- Automated rollback on error spike
- Manual rollback capability

## 12. Scalability Design

### 12.1 Horizontal Scaling

**Lambda:**
- Auto-scales based on requests
- Concurrent execution limit: 1000

**SageMaker:**
- Auto-scaling policy
- Target tracking: 70% CPU

**DynamoDB:**
- On-demand billing mode
- Auto-scales read/write capacity

### 12.2 Vertical Scaling

**Lambda Memory:**
- Start: 512 MB
- Max: 3008 MB
- Adjust based on profiling

**SageMaker Instance:**
- Start: ml.m5.large
- Scale up: ml.m5.xlarge

## 13. Testing Strategy

### 13.1 Unit Tests

- Service layer: 80% coverage
- Controller layer: 70% coverage
- Utility functions: 90% coverage

### 13.2 Integration Tests

- WhatsApp webhook flow
- SageMaker inference
- DynamoDB operations
- S3 upload/download

### 13.3 End-to-End Tests

- Complete user journey
- Error scenarios
- Performance benchmarks

## 14. Future Enhancements

### 14.1 Technical Improvements

- GraphQL API for flexible queries
- WebSocket for real-time updates
- Edge computing for offline support
- Blockchain for supply chain tracking

### 14.2 Feature Additions

- Multi-crop support
- Pest identification
- Soil health analysis
- Yield prediction
- Market price integration

### 14.3 Platform Expansion

- Mobile app (iOS/Android)
- Web dashboard for farmers
- Admin portal for suppliers
- Analytics dashboard
