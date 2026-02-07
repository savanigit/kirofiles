# Neural Roots AI - System Requirements Document

## 1. Project Overview

### 1.1 System Purpose
Neural Roots AI is a comprehensive agricultural supply chain management platform that leverages multi-agent AI systems to optimize crop freshness assessment, pricing strategies, logistics coordination, and weather impact analysis for the Indian agricultural market.

### 1.2 Business Objectives
- **Reduce Food Waste**: Minimize crop spoilage through predictive freshness assessment
- **Optimize Pricing**: Maximize farmer revenue through dynamic market-aware pricing
- **Streamline Logistics**: Efficient driver allocation and delivery mode selection
- **Weather Resilience**: Mitigate weather-related crop losses during transport
- **Supply Chain Transparency**: Provide real-time visibility across the agricultural value chain

### 1.3 Target Users
- **Primary Users**: Farmers, Wholesalers/Traders, Logistics Providers
- **Secondary Users**: Market Administrators, Retailers, Agricultural Cooperatives
- **System Administrators**: IT Support, Data Analysts

## 2. Functional Requirements

### 2.1 Multi-Agent AI Workflow System

#### 2.1.1 Freshness Assessment Agent
**FR-001**: The system SHALL analyze crop freshness using environmental parameters
- **Input Parameters**: Crop name, temperature (°C), humidity (%), age (hours), quantity
- **Output**: Freshness score (0-100), freshness level (EXCELLENT/GOOD/FAIR/POOR/CRITICAL)
- **Scoring Algorithm**: Weighted formula (Temperature: 30%, Humidity: 40%, Age: 30%)
- **Crop Support**: Minimum 10 crop types (tomato, onion, potato, mango, etc.)

**FR-002**: The system SHALL provide crop-specific degradation predictions
- **Degradation Rates**: Variable by crop type (0.5-4% per hour)
- **Environmental Factors**: Temperature, humidity, storage conditions
- **Recommendations**: Actionable items based on freshness level

#### 2.1.2 Market Intelligence Agent
**FR-003**: The system SHALL fetch and analyze market data for pricing optimization
- **Data Sources**: MongoDB wholesalers collection, market APIs
- **Price Calculation**: Dynamic multipliers based on freshness (±20%), demand (±15%), urgency (±15%)
- **Pricing Strategies**: Premium, Market Rate, Discount, Clearance
- **Market Trends**: Real-time price tracking and trend analysis

**FR-004**: The system SHALL support demand-supply analysis
- **Demand Levels**: HIGH, MEDIUM, LOW classification
- **Supply Tracking**: Warehouse capacity, inventory levels
- **Price Recommendations**: Optimal pricing based on market conditions

#### 2.1.3 Logistics Optimization Agent
**FR-005**: The system SHALL recommend optimal delivery modes
- **Delivery Types**: Cold Chain (1.5x cost), Refrigerated (1.3x cost), Standard (1.0x cost)
- **Selection Criteria**: Freshness level, distance, urgency, cost optimization
- **Driver Scoring**: Capacity match (30%), rating (20%), vehicle type (20%), availability (10%)

**FR-006**: The system SHALL provide driver allocation and route optimization
- **Driver Database**: Name, vehicle type, capacity, rating, location, availability
- **Route Calculation**: Distance, time estimation, cost calculation
- **Top Recommendations**: Minimum 3 driver options with scoring rationale

#### 2.1.4 Weather Impact Agent
**FR-007**: The system SHALL assess weather impact on crop degradation
- **Weather Data**: Temperature, humidity, precipitation, wind speed
- **Degradation Calculation**: Crop-specific sensitivity multipliers
- **Risk Assessment**: LOW, MEDIUM, HIGH, CRITICAL weather risk levels
- **Transport Optimization**: Weather-aware delivery timing recommendations

### 2.2 IoT Integration System

#### 2.2.1 ESP32 Device Support
**FR-008**: The system SHALL ingest data from ESP32-CAM devices
- **Data Types**: Temperature, humidity, image capture
- **Communication Protocol**: HTTP POST with multipart form-data
- **Data Validation**: Range validation, timestamp verification
- **Image Storage**: Local file system with unique naming convention

**FR-009**: The system SHALL maintain device status tracking
- **Device Registry**: Device ID, last seen, status (online/offline)
- **Health Monitoring**: Connection status, data quality, battery level
- **Alert System**: Offline device notifications, data anomaly detection

#### 2.2.2 Real-time Data Processing
**FR-010**: The system SHALL process IoT data in real-time
- **Automatic Workflow Trigger**: Execute assessment on new sensor data
- **Data Persistence**: Store readings in MongoDB iot_logs collection
- **Historical Analysis**: Trend analysis, pattern recognition

### 2.3 Communication Systems

#### 2.3.1 WhatsApp Integration
**FR-011**: The system SHALL support WhatsApp communication via Twilio
- **Webhook Processing**: Receive farmer messages, trigger workflows
- **Response Generation**: Send assessment results, recommendations
- **Message Validation**: Input sanitization, error handling

#### 2.3.2 Web Dashboard
**FR-012**: The system SHALL provide a comprehensive web interface
- **Real-time Monitoring**: Live data updates, status indicators
- **Interactive Maps**: Driver tracking, location visualization
- **Data Management**: CRUD operations for farmers, drivers, wholesalers

### 2.4 Data Management

#### 2.4.1 Database Requirements
**FR-013**: The system SHALL use MongoDB for data persistence
- **Collections**: wholesalers, drivers, weather, iot_logs, iot_devices, farmers, market_items
- **Data Validation**: Schema validation using Pydantic models
- **Indexing**: Performance optimization for frequent queries

**FR-014**: The system SHALL maintain data integrity and consistency
- **ACID Properties**: Transaction support where required
- **Backup Strategy**: Regular automated backups
- **Data Retention**: Configurable retention policies

### 2.5 API Requirements

#### 2.5.1 RESTful API Design
**FR-015**: The system SHALL provide RESTful APIs for all operations
- **Workflow Endpoints**: 5 main endpoints for assessment operations
- **IoT Endpoints**: Device management, data ingestion
- **CRUD Endpoints**: Farmer, driver, wholesaler management
- **Response Format**: JSON with standardized error handling

**FR-016**: The system SHALL implement proper API documentation
- **OpenAPI Specification**: Auto-generated documentation
- **Request/Response Examples**: Comprehensive API examples
- **Error Codes**: Standardized HTTP status codes and error messages

## 3. Non-Functional Requirements

### 3.1 Performance Requirements

**NFR-001**: Response Time
- **Quick Assessment**: < 400ms
- **Full Workflow**: < 800ms
- **Database Queries**: < 100ms
- **IoT Data Ingestion**: < 200ms

**NFR-002**: Throughput
- **Concurrent Users**: Support 100+ simultaneous users
- **API Requests**: 1000+ requests per minute
- **IoT Data Points**: 10,000+ sensor readings per hour

**NFR-003**: Scalability
- **Horizontal Scaling**: Stateless agent design for load balancing
- **Database Scaling**: MongoDB sharding support
- **Resource Utilization**: < 100MB memory usage per instance

### 3.2 Reliability Requirements

**NFR-004**: Availability
- **System Uptime**: 99.5% availability (4.38 hours downtime per month)
- **Graceful Degradation**: Fallback to simulated data when external services fail
- **Error Recovery**: Automatic retry mechanisms for transient failures

**NFR-005**: Data Integrity
- **Data Validation**: Input validation at API and database levels
- **Backup Recovery**: RTO < 4 hours, RPO < 1 hour
- **Consistency**: Eventual consistency for distributed operations

### 3.3 Security Requirements

**NFR-006**: Authentication and Authorization
- **API Security**: Token-based authentication (future implementation)
- **Data Encryption**: HTTPS for all communications
- **Input Validation**: SQL injection, XSS prevention

**NFR-007**: Data Privacy
- **PII Protection**: Farmer personal information encryption
- **Access Control**: Role-based access control (RBAC)
- **Audit Logging**: Comprehensive activity logging

### 3.4 Usability Requirements

**NFR-008**: User Interface
- **Responsive Design**: Support for desktop, tablet, mobile devices
- **Accessibility**: WCAG 2.1 AA compliance
- **Internationalization**: Multi-language support (Hindi, English)

**NFR-009**: User Experience
- **Learning Curve**: < 30 minutes for basic operations
- **Error Messages**: Clear, actionable error messages
- **Help System**: Contextual help and documentation

### 3.5 Compatibility Requirements

**NFR-010**: Browser Support
- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile Browsers**: iOS Safari 14+, Chrome Mobile 90+

**NFR-011**: Platform Support
- **Operating Systems**: Windows 10+, macOS 10.15+, Ubuntu 20.04+
- **Container Support**: Docker 20.10+, Docker Compose 2.0+

## 4. Technical Constraints

### 4.1 Technology Stack Constraints
- **Backend Framework**: FastAPI (Python 3.8+)
- **Frontend Framework**: Next.js 15+ with React 18+
- **Database**: MongoDB 4.6+
- **Communication**: Twilio for WhatsApp integration
- **Containerization**: Docker for deployment

### 4.2 Integration Constraints
- **IoT Devices**: ESP32-CAM with HTTP communication
- **Weather APIs**: OpenWeatherMap or similar (future)
- **Payment Gateways**: Razorpay/PayU integration (future)

### 4.3 Deployment Constraints
- **Cloud Platform**: AWS/Azure/GCP compatible
- **Network Requirements**: Minimum 10 Mbps internet connection
- **Storage Requirements**: 100GB initial storage, 10GB growth per month

## 5. Business Rules

### 5.1 Freshness Assessment Rules
- **BR-001**: Freshness score below 20 triggers CRITICAL alert
- **BR-002**: Cold chain delivery mandatory for freshness < 40
- **BR-003**: Price discount increases as freshness decreases
- **BR-004**: Weather degradation cannot reduce freshness below 0

### 5.2 Pricing Rules
- **BR-005**: Maximum price premium: 20% above market rate
- **BR-006**: Maximum discount: 30% below market rate
- **BR-007**: Bulk orders (>100kg) receive 5% automatic discount
- **BR-008**: Emergency sales (freshness < 20) limited to 50% market rate

### 5.3 Logistics Rules
- **BR-009**: Driver capacity must exceed order quantity by 10%
- **BR-010**: Maximum delivery distance: 500km for perishables
- **BR-011**: Cold chain vehicles required for high-value crops (>₹100/kg)
- **BR-012**: Driver rating below 3.0 excluded from premium deliveries

## 6. Compliance Requirements

### 6.1 Regulatory Compliance
- **Food Safety Standards**: FSSAI compliance for food handling
- **Data Protection**: Personal Data Protection Bill (India) compliance
- **Agricultural Regulations**: APMC Act compliance for market operations

### 6.2 Industry Standards
- **API Standards**: REST API best practices, OpenAPI 3.0
- **Security Standards**: OWASP Top 10 compliance
- **Quality Standards**: ISO 9001 quality management principles

## 7. Success Criteria

### 7.1 Quantitative Metrics
- **Freshness Prediction Accuracy**: >85% accuracy in freshness assessment
- **Price Optimization**: 15% average increase in farmer revenue
- **Delivery Efficiency**: 20% reduction in delivery time
- **Food Waste Reduction**: 25% reduction in post-harvest losses

### 7.2 Qualitative Metrics
- **User Satisfaction**: >4.0/5.0 user rating
- **System Adoption**: 80% user retention after 3 months
- **Stakeholder Feedback**: Positive feedback from farmers, wholesalers, drivers

## 8. Assumptions and Dependencies

### 8.1 Assumptions
- **Internet Connectivity**: Reliable internet access for IoT devices
- **User Training**: Basic smartphone/computer literacy among users
- **Market Data**: Availability of wholesale market pricing data
- **Weather Data**: Access to reliable weather forecast APIs

### 8.2 Dependencies
- **Third-party Services**: Twilio for WhatsApp, weather API providers
- **Hardware**: ESP32-CAM devices for IoT data collection
- **Infrastructure**: Cloud hosting platform availability
- **Database**: MongoDB Atlas or self-hosted MongoDB instance

## 9. Risk Assessment

### 9.1 Technical Risks
- **High Risk**: IoT device connectivity issues in rural areas
- **Medium Risk**: Third-party API service disruptions
- **Low Risk**: Database performance degradation with scale

### 9.2 Business Risks
- **High Risk**: User adoption challenges in rural markets
- **Medium Risk**: Competition from established agricultural platforms
- **Low Risk**: Regulatory changes affecting operations

### 9.3 Mitigation Strategies
- **Connectivity**: Offline mode with data synchronization
- **API Failures**: Fallback to simulated/cached data
- **Adoption**: Comprehensive training and support programs
- **Competition**: Focus on unique AI-driven value proposition

## 10. Future Enhancements

### 10.1 Phase 2 Features
- **Machine Learning Models**: Advanced predictive analytics
- **Blockchain Integration**: Supply chain transparency and traceability
- **Mobile Applications**: Native iOS/Android apps
- **Advanced Analytics**: Business intelligence dashboard

### 10.2 Phase 3 Features
- **AI-Powered Recommendations**: Crop selection optimization
- **Carbon Footprint Tracking**: Sustainability metrics
- **Financial Services**: Credit scoring, insurance integration
- **Marketplace Integration**: Direct farmer-to-consumer sales

---

**Document Version**: 1.0  
**Last Updated**: January 24, 2026  
**Prepared By**: System Analysis Team  
**Approved By**: Project Stakeholders