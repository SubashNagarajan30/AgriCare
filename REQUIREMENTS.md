# AgriCare Disease Sentinel - Requirements Document

## 1. Executive Summary

AgriCare is a WhatsApp-based AI system designed to help small-scale farmers identify crop diseases early and access treatment solutions through their local supply chain.

## 2. Problem Statement

Small-scale farmers face critical challenges:
- Lack of expertise to identify crop diseases early
- Limited access to agricultural extension services
- Difficulty finding appropriate treatments and suppliers
- Significant crop losses due to delayed intervention
- Language and literacy barriers to accessing digital resources

## 3. Solution Overview

A WhatsApp-based intelligent agent that:
- Accepts crop leaf images from farmers
- Identifies diseases using AI/ML
- Provides contextual treatment recommendations
- Connects farmers with local suppliers
- Considers local weather conditions

## 4. Functional Requirements

### 4.1 Image Processing
- **FR-1.1**: System shall accept JPEG/PNG images via WhatsApp
- **FR-1.2**: System shall support images between 100KB - 10MB
- **FR-1.3**: System shall store uploaded images in S3 for audit trail
- **FR-1.4**: System shall handle poor quality images gracefully

### 4.2 Disease Identification
- **FR-2.1**: System shall classify crop diseases with minimum 80% accuracy
- **FR-2.2**: System shall support at least 20 common crop diseases
- **FR-2.3**: System shall provide confidence score with each prediction
- **FR-2.4**: System shall identify crop type from leaf image
- **FR-2.5**: System shall handle multiple diseases on single leaf

### 4.3 Weather Integration
- **FR-3.1**: System shall retrieve real-time weather data from AWS IoT Core
- **FR-3.2**: System shall fallback to weather API if IoT data unavailable
- **FR-3.3**: System shall analyze temperature, humidity, and rainfall
- **FR-3.4**: System shall provide weather-based treatment timing advice
- **FR-3.5**: System shall alert farmers about disease-favorable conditions

### 4.4 Treatment Recommendations
- **FR-4.1**: System shall provide disease-specific treatment plans
- **FR-4.2**: System shall recommend both organic and chemical options
- **FR-4.3**: System shall include application instructions
- **FR-4.4**: System shall provide dosage information
- **FR-4.5**: System shall include safety precautions

### 4.5 Supplier Matching
- **FR-5.1**: System shall find suppliers within 50km radius
- **FR-5.2**: System shall rank suppliers by distance
- **FR-5.3**: System shall provide supplier contact information
- **FR-5.4**: System shall include product availability status
- **FR-5.5**: System shall provide map links to supplier locations
- **FR-5.6**: System shall support up to 5 supplier recommendations

### 4.6 WhatsApp Interface
- **FR-6.1**: System shall respond within 10 seconds
- **FR-6.2**: System shall support multiple languages (English, Hindi, regional)
- **FR-6.3**: System shall provide clear, actionable messages
- **FR-6.4**: System shall handle conversation context
- **FR-6.5**: System shall support help commands

## 5. Non-Functional Requirements

### 5.1 Performance
- **NFR-1.1**: Response time < 10 seconds for 95% of requests
- **NFR-1.2**: System shall handle 1000 concurrent users
- **NFR-1.3**: ML inference time < 3 seconds
- **NFR-1.4**: Image upload time < 5 seconds

### 5.2 Scalability
- **NFR-2.1**: System shall auto-scale based on demand
- **NFR-2.2**: System shall support 10,000 daily active users
- **NFR-2.3**: Database shall handle 100,000 supplier records

### 5.3 Availability
- **NFR-3.1**: System uptime shall be 99.5%
- **NFR-3.2**: Planned maintenance windows < 4 hours/month
- **NFR-3.3**: System shall have disaster recovery plan

### 5.4 Security
- **NFR-4.1**: All data shall be encrypted at rest
- **NFR-4.2**: All data shall be encrypted in transit (TLS 1.2+)
- **NFR-4.3**: System shall implement rate limiting
- **NFR-4.4**: System shall validate webhook signatures
- **NFR-4.5**: PII shall be anonymized in logs

### 5.5 Reliability
- **NFR-5.1**: System shall gracefully handle service failures
- **NFR-5.2**: System shall provide fallback responses
- **NFR-5.3**: System shall log all errors for debugging
- **NFR-5.4**: System shall retry failed operations (max 3 attempts)

### 5.6 Usability
- **NFR-6.1**: Messages shall be in simple, non-technical language
- **NFR-6.2**: System shall support voice messages (future)
- **NFR-6.3**: System shall provide visual guides (future)

### 5.7 Compliance
- **NFR-7.1**: System shall comply with data protection regulations
- **NFR-7.2**: System shall maintain audit logs for 90 days
- **NFR-7.3**: System shall allow user data deletion requests

## 6. User Stories

### 6.1 Farmer - Disease Detection
**As a** farmer  
**I want to** send a photo of my diseased crop  
**So that** I can quickly identify the problem

**Acceptance Criteria:**
- Photo uploads successfully via WhatsApp
- Receives disease name within 10 seconds
- Gets confidence score with result

### 6.2 Farmer - Treatment Access
**As a** farmer  
**I want to** receive treatment recommendations  
**So that** I can save my crop

**Acceptance Criteria:**
- Receives specific treatment plan
- Gets product names and dosages
- Receives application instructions

### 6.3 Farmer - Supplier Connection
**As a** farmer  
**I want to** find nearby suppliers  
**So that** I can purchase treatments quickly

**Acceptance Criteria:**
- Receives list of 3-5 nearby suppliers
- Gets contact information and directions
- Sees distance to each supplier

### 6.4 Farmer - Weather Awareness
**As a** farmer  
**I want to** know if weather affects treatment  
**So that** I can apply it at the right time

**Acceptance Criteria:**
- Receives weather-based alerts
- Gets timing recommendations
- Understands weather impact on disease

## 7. System Constraints

### 7.1 Technical Constraints
- Must use AWS services for infrastructure
- Must integrate with Twilio WhatsApp API
- Must support Node.js runtime
- Must work with limited internet connectivity

### 7.2 Business Constraints
- Must be cost-effective for small-scale deployment
- Must support pay-per-use pricing model
- Must minimize operational overhead

### 7.3 Regulatory Constraints
- Must comply with agricultural product regulations
- Must not provide medical advice
- Must include appropriate disclaimers

## 8. Assumptions

- Farmers have access to smartphones with WhatsApp
- Farmers can take reasonably clear photos
- Local suppliers maintain updated inventory
- Weather stations provide reliable data
- Internet connectivity is intermittent but available

## 9. Dependencies

### 9.1 External Services
- AWS SageMaker for ML inference
- Twilio for WhatsApp messaging
- OpenWeather API for weather data
- AWS IoT Core for sensor data

### 9.2 Data Sources
- Pre-trained plant disease models
- Supplier database (manually curated)
- Disease treatment database
- Weather sensor network

## 10. Success Metrics

### 10.1 Technical Metrics
- Disease classification accuracy > 85%
- System response time < 10 seconds
- System uptime > 99.5%
- Error rate < 1%

### 10.2 Business Metrics
- Daily active users > 1000
- User retention rate > 60%
- Average response rating > 4/5
- Crop loss reduction > 30%

### 10.3 User Satisfaction
- Farmer satisfaction score > 4/5
- Treatment success rate > 70%
- Supplier connection rate > 80%

## 11. Future Enhancements

### 11.1 Phase 2 Features
- Multi-language support (10+ languages)
- Voice message support
- Video consultation with experts
- Crop health monitoring over time
- Pest identification

### 11.2 Phase 3 Features
- Predictive disease alerts
- Community knowledge sharing
- Marketplace integration
- Crop yield optimization
- Financial assistance integration

## 12. Risks and Mitigation

### 12.1 Technical Risks
| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| ML model inaccuracy | High | Medium | Regular model retraining, human verification |
| Service downtime | High | Low | Multi-region deployment, fallback systems |
| Poor image quality | Medium | High | Image quality validation, user guidance |

### 12.2 Business Risks
| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low adoption | High | Medium | User training, local partnerships |
| Supplier data outdated | Medium | High | Regular supplier verification |
| Cost overruns | Medium | Low | Usage monitoring, cost optimization |

## 13. Glossary

- **Disease Sentinel**: Automated monitoring system for crop diseases
- **SageMaker JumpStart**: AWS pre-trained ML model library
- **IoT Core**: AWS service for IoT device management
- **Confidence Score**: ML model's certainty in prediction (0-100%)
- **Treatment Plan**: Recommended actions to address disease

