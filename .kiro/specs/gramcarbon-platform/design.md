# Design Document: GramCarbon Platform

## Overview

The GramCarbon Platform is a cloud-native, AI-powered digital ecosystem that connects rural farmers with global carbon markets through verified agroforestry carbon credits. The system integrates satellite imagery analysis, machine learning-based biomass estimation, blockchain-based carbon tokenization, and mobile-first user interfaces optimized for low-bandwidth rural environments.

### Design Principles

1. **Mobile-First**: Optimized for basic smartphones with offline-first capabilities
2. **Scalability**: Cloud-native architecture supporting 1M+ farmers
3. **Transparency**: Blockchain-based immutable audit trails
4. **Scientific Rigor**: AI models validated against ground-truth measurements
5. **Rural-Friendly**: Low bandwidth, multi-language, simple UX
6. **Security**: End-to-end encryption and role-based access control

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
├─────────────────────────────────────────────────────────────────┤
│  Mobile App (Android)  │  Web Dashboard  │  Admin Portal        │
│  - Offline-first       │  - Analytics    │  - User Management   │
│  - GPS capture         │  - Reporting    │  - Audit Tools       │
│  - QR scanning         │  - Marketplace  │  - System Config     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                           │
├─────────────────────────────────────────────────────────────────┤
│  - Authentication/Authorization (JWT)                            │
│  - Rate Limiting                                                 │
│  - Request Routing                                               │
│  - API Versioning                                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Application Services Layer                    │
├─────────────────────────────────────────────────────────────────┤
│  Farmer Service  │  Land Service   │  Tree Service              │
│  Carbon Service  │  Payment Service│  Analytics Service         │
│  Alert Service   │  Marketplace    │  Audit Service             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AI/ML Processing Layer                        │
├─────────────────────────────────────────────────────────────────┤
│  Satellite Imagery Processor  │  Biomass Estimation Engine      │
│  NDVI Calculator              │  Anomaly Detection Engine       │
│  Demand Prediction Model      │  Price Recommendation Engine   │
│  Climate Risk Analyzer        │  Species Recommendation Engine │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                  │
├─────────────────────────────────────────────────────────────────┤
│  PostgreSQL (Relational)  │  MongoDB (Documents)                │
│  Redis (Cache)            │  S3 (Object Storage)                │
│  TimescaleDB (Time-series)│  Blockchain Ledger                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    External Integrations                         │
├─────────────────────────────────────────────────────────────────┤
│  Sentinel-2 API       │  Weather APIs    │  Payment Gateways   │
│  Landsat API          │  SMS Gateway     │  Blockchain Network │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Backend Services:**
- Language: Python 3.11+ (FastAPI for services)
- API Framework: FastAPI with async support
- Task Queue: Celery with Redis broker
- Container Orchestration: Kubernetes
- Service Mesh: Istio for inter-service communication

**AI/ML Stack:**
- ML Framework: PyTorch for deep learning models
- Geospatial: GDAL, Rasterio for satellite imagery processing
- Computer Vision: OpenCV, scikit-image
- Model Serving: TorchServe or TensorFlow Serving
- Feature Store: Feast for ML feature management

**Data Storage:**
- Primary Database: PostgreSQL 15+ with PostGIS extension
- Document Store: MongoDB for flexible schemas
- Time-Series: TimescaleDB for sensor and satellite data
- Cache: Redis for session and frequently accessed data
- Object Storage: AWS S3 or MinIO for imagery and documents
- Blockchain: Hyperledger Fabric or Ethereum for carbon tokens

**Mobile Application:**
- Framework: React Native for cross-platform development
- Offline Storage: SQLite with sync mechanism
- State Management: Redux with Redux Persist
- Maps: Mapbox GL for GPS and mapping features

**Infrastructure:**
- Cloud Provider: AWS (or Azure/GCP)
- CDN: CloudFront for static assets
- Monitoring: Prometheus + Grafana
- Logging: ELK Stack (Elasticsearch, Logstash, Kibana)
- Tracing: Jaeger for distributed tracing

## Components and Interfaces

### 1. Farmer Service

**Responsibility:** Manages farmer profiles, authentication, and onboarding workflows.

**Key Operations:**
- `createFarmerProfile(name, contact, language, documents) → FarmerProfile`
- `validateOwnership(farmerId, documents) → ValidationStatus`
- `updateFarmerPreferences(farmerId, preferences) → Success`
- `getFarmerById(farmerId) → FarmerProfile`
- `searchFarmers(filters) → List<FarmerProfile>`

**Interfaces:**
```python
class FarmerProfile:
    farmer_id: UUID
    name: str
    contact_info: ContactInfo
    preferred_language: LanguageCode
    registration_date: datetime
    ownership_validation_status: ValidationStatus
    digital_wallet_address: str
    total_land_area_hectares: float
    
class ContactInfo:
    phone_number: str
    alternate_phone: Optional[str]
    village: str
    district: str
    state: str
    pin_code: str
```

### 2. Land Service

**Responsibility:** Manages land parcel registration, GPS boundary capture, and overlap detection.

**Key Operations:**
- `registerLandParcel(farmerId, gpsCoordinates, area) → LandParcel`
- `validateBoundaries(coordinates) → ValidationResult`
- `detectOverlaps(parcelId) → List<OverlapInfo>`
- `calculateArea(coordinates) → float`
- `getLandParcelsByFarmer(farmerId) → List<LandParcel>`

**Interfaces:**
```python
class LandParcel:
    parcel_id: UUID
    farmer_id: UUID
    boundaries: List[GPSCoordinate]
    area_hectares: float
    registration_date: datetime
    ownership_documents: List[DocumentReference]
    validation_status: ValidationStatus
    overlap_flags: List[OverlapFlag]
    
class GPSCoordinate:
    latitude: float  # -90 to 90
    longitude: float  # -180 to 180
    altitude: Optional[float]
    accuracy_meters: float
    timestamp: datetime
    
class OverlapFlag:
    overlapping_parcel_id: UUID
    overlap_percentage: float
    status: str  # 'pending_review', 'resolved', 'confirmed_duplicate'
```

### 3. Tree Service

**Responsibility:** Tracks individual tree lifecycle from planting to maturity.

**Key Operations:**
- `plantTree(parcelId, species, location, qrTag) → TreeRecord`
- `updateTreeStatus(treeId, status, growthStage) → Success`
- `recordMortality(treeId, mortalityDate, cause) → Success`
- `getTreeByQRTag(qrTag) → TreeRecord`
- `getTreesByParcel(parcelId) → List<TreeRecord>`
- `calculateSurvivalRate(parcelId, dateRange) → float`

**Interfaces:**
```python
class TreeRecord:
    tree_id: UUID
    parcel_id: UUID
    farmer_id: UUID
    qr_tag: str
    species: TreeSpecies
    planting_date: datetime
    gps_location: GPSCoordinate
    status: TreeStatus  # 'alive', 'deceased', 'removed'
    growth_stage: GrowthStage  # 'sapling', 'juvenile', 'mature'
    mortality_date: Optional[datetime]
    mortality_cause: Optional[str]
    growth_history: List[GrowthUpdate]
    
class TreeSpecies:
    species_id: UUID
    common_name: str
    scientific_name: str
    carbon_sequestration_rate: float  # kg CO2/year
    water_requirement: str  # 'low', 'medium', 'high'
    soil_preference: List[str]
    maturity_years: int
    
class GrowthUpdate:
    update_date: datetime
    growth_stage: GrowthStage
    height_meters: Optional[float]
    diameter_cm: Optional[float]
    health_score: Optional[int]  # 0-100
    notes: Optional[str]
```

### 4. Satellite Imagery Service

**Responsibility:** Retrieves, processes, and stores satellite imagery for land parcels.

**Key Operations:**
- `subscribeToParcels(parcelIds) → Success`
- `retrieveImagery(parcelId, dateRange) → List<SatelliteImage>`
- `processImagery(imageId) → ProcessingResult`
- `calculateNDVI(imageId, parcelBoundaries) → NDVIResult`
- `getTimeSeriesData(parcelId, metric, dateRange) → TimeSeries`

**Interfaces:**
```python
class SatelliteImage:
    image_id: UUID
    parcel_id: UUID
    capture_date: datetime
    data_source: str  # 'Sentinel-2', 'Landsat-8', 'Planet'
    resolution_meters: float
    cloud_cover_percentage: float
    quality_flag: str  # 'good', 'acceptable', 'poor'
    storage_url: str
    bands: Dict[str, str]  # band_name -> storage_url
    metadata: Dict[str, Any]
    
class NDVIResult:
    image_id: UUID
    parcel_id: UUID
    calculation_date: datetime
    mean_ndvi: float  # -1 to 1
    std_ndvi: float
    vegetation_health_score: int  # 0-100
    ndvi_raster_url: str
    confidence: float
```

### 5. Biomass Estimation Service

**Responsibility:** Calculates biomass and carbon sequestration using AI models.

**Key Operations:**
- `estimateBiomass(parcelId, imageId) → BiomassEstimate`
- `calculateCarbonSequestration(biomass, treeData) → CarbonCalculation`
- `validateEstimate(estimateId) → ValidationResult`
- `getHistoricalEstimates(parcelId, dateRange) → List<BiomassEstimate>`

**AI Model Architecture:**
```
Input: Multi-spectral satellite imagery (10m resolution)
       + Tree species distribution
       + Tree age/density data
       + Soil type
       + Climate zone

Model: Ensemble of:
       1. CNN-based biomass regression (ResNet-50 backbone)
       2. Random Forest with handcrafted features (NDVI, EVI, SAVI)
       3. Allometric equations (species-specific)

Output: Biomass (tons/hectare) + Confidence interval
```

**Interfaces:**
```python
class BiomassEstimate:
    estimate_id: UUID
    parcel_id: UUID
    image_id: UUID
    estimation_date: datetime
    biomass_tons_per_hectare: float
    confidence_interval: Tuple[float, float]
    confidence_score: float  # 0-1
    methodology: str
    model_version: str
    validation_status: str  # 'pending', 'verified', 'rejected'
    
class CarbonCalculation:
    calculation_id: UUID
    parcel_id: UUID
    biomass_estimate_id: UUID
    calculation_date: datetime
    carbon_sequestered_tons_co2: float
    calculation_period_start: datetime
    calculation_period_end: datetime
    tree_count: int
    species_breakdown: Dict[str, float]  # species -> tons CO2
    methodology: str
    confidence_score: float
```

### 6. Anomaly Detection Service

**Responsibility:** Detects tree loss, vegetation decline, and suspicious patterns.

**Key Operations:**
- `detectAnomalies(parcelId, currentImage, previousImage) → List<Anomaly>`
- `analyzeVegetationTrend(parcelId, dateRange) → TrendAnalysis`
- `generateAlert(anomalyId) → Alert`
- `investigateAnomaly(anomalyId, findings) → Success`

**Detection Algorithm:**
```
1. Calculate NDVI difference between consecutive images
2. Apply change detection threshold (20% decline)
3. Segment affected areas using connected components
4. Estimate tree count impact using tree density map
5. Generate alert with severity based on:
   - Magnitude of change
   - Area affected
   - Historical patterns
   - Time since last inspection
```

**Interfaces:**
```python
class Anomaly:
    anomaly_id: UUID
    parcel_id: UUID
    detection_date: datetime
    anomaly_type: str  # 'vegetation_loss', 'health_decline', 'land_use_change'
    severity: str  # 'low', 'medium', 'high', 'critical'
    affected_area_hectares: float
    estimated_trees_impacted: int
    ndvi_change_percentage: float
    before_image_id: UUID
    after_image_id: UUID
    comparison_image_url: str
    investigation_status: str
    investigation_findings: Optional[str]
    
class Alert:
    alert_id: UUID
    anomaly_id: UUID
    farmer_id: UUID
    alert_type: str
    severity: str
    message: str
    sent_date: datetime
    delivery_channels: List[str]  # 'sms', 'app', 'email'
    acknowledgment_status: str
    acknowledgment_date: Optional[datetime]
```

### 7. Carbon Token Service

**Responsibility:** Manages blockchain-based carbon credit tokenization and prevents double counting.

**Key Operations:**
- `tokenizeCarbonCredits(calculationId, auditorSignature) → List<CarbonToken>`
- `verifyCarbonData(calculationId) → VerificationResult`
- `checkDoubleCount(calculationId) → bool`
- `transferToken(tokenId, newOwner) → TransactionReceipt`
- `getTokenProvenance(tokenId) → ProvenanceChain`

**Blockchain Integration:**
```
Smart Contract: CarbonCreditToken (ERC-721 or custom)

Functions:
- mintToken(parcelId, co2Amount, verificationHash, metadata)
- transferToken(tokenId, from, to)
- burnToken(tokenId)  // for retirement
- getTokenMetadata(tokenId)
- verifyProvenance(tokenId)

Events:
- TokenMinted(tokenId, parcelId, co2Amount, timestamp)
- TokenTransferred(tokenId, from, to, timestamp)
- TokenRetired(tokenId, retiredBy, timestamp)
```

**Interfaces:**
```python
class CarbonToken:
    token_id: UUID
    blockchain_token_id: str
    parcel_id: UUID
    farmer_id: UUID
    carbon_calculation_id: UUID
    co2_tons: float  # Always 1.0 (one token = one ton)
    minting_date: datetime
    verification_date: datetime
    auditor_id: UUID
    auditor_signature: str
    current_owner: str  # blockchain address
    status: str  # 'active', 'sold', 'retired'
    transaction_hash: str
    metadata_uri: str
    
class ProvenanceChain:
    token_id: UUID
    creation_record: CreationRecord
    ownership_history: List[OwnershipTransfer]
    verification_records: List[VerificationRecord]
    retirement_record: Optional[RetirementRecord]
    
class CreationRecord:
    parcel_id: UUID
    farmer_name: str
    tree_species: List[str]
    planting_date_range: Tuple[datetime, datetime]
    verification_methodology: str
    satellite_images_used: List[UUID]
    biomass_estimates: List[UUID]
```

### 8. Smart Contract Payment Service

**Responsibility:** Automates payment distribution when carbon credits are sold.

**Key Operations:**
- `executeSale(tokenId, buyerId, salePrice) → PaymentTransaction`
- `calculateDistribution(salePrice) → PaymentDistribution`
- `processPayment(distribution) → List[PaymentReceipt>`
- `retryFailedPayment(transactionId) → PaymentReceipt`
- `getPaymentHistory(farmerId) → List<PaymentTransaction>`

**Payment Distribution Logic:**
```
Sale Price = X
├─ Farmer Share: 70% of X
├─ Platform Fee: 20% of X
├─ Field Agent Commission: 8% of X
└─ Carbon Fund: 2% of X (community replanting)
```

**Interfaces:**
```python
class PaymentTransaction:
    transaction_id: UUID
    token_id: UUID
    farmer_id: UUID
    buyer_id: UUID
    sale_price: Decimal
    sale_date: datetime
    distribution: PaymentDistribution
    status: str  # 'pending', 'completed', 'failed', 'refunded'
    blockchain_tx_hash: str
    
class PaymentDistribution:
    farmer_amount: Decimal
    farmer_wallet: str
    platform_fee: Decimal
    agent_commission: Decimal
    agent_id: Optional[UUID]
    carbon_fund_amount: Decimal
    currency: str
    
class PaymentReceipt:
    receipt_id: UUID
    transaction_id: UUID
    recipient_id: UUID
    amount: Decimal
    payment_method: str  # 'blockchain', 'upi', 'bank_transfer'
    payment_date: datetime
    status: str
    confirmation_code: str
    retry_count: int
```

### 9. Marketplace Service

**Responsibility:** Manages agroforestry produce listings and marketplace operations.

**Key Operations:**
- `createListing(farmerId, produceType, quantity, price) → ProduceListing`
- `getPriceRecommendation(produceType, location, season) → PriceRecommendation`
- `predictDemand(produceType, location, futureDate) → DemandPrediction`
- `searchListings(filters) → List<ProduceListing>`
- `recordSale(listingId, salePrice) → SaleRecord`

**Interfaces:**
```python
class ProduceListing:
    listing_id: UUID
    farmer_id: UUID
    parcel_id: UUID
    produce_type: str  # 'fruit', 'timber', 'fodder', 'nuts', 'honey'
    produce_name: str
    quantity: float
    unit: str  # 'kg', 'tons', 'cubic_meters'
    quality_grade: str  # 'A', 'B', 'C'
    asking_price: Decimal
    currency: str
    listing_date: datetime
    harvest_date: datetime
    expiry_date: datetime
    status: str  # 'active', 'sold', 'expired', 'withdrawn'
    images: List[str]
    location: GPSCoordinate
    
class PriceRecommendation:
    produce_type: str
    location: str
    recommended_price: Decimal
    price_range: Tuple[Decimal, Decimal]
    market_average: Decimal
    confidence: float
    factors: Dict[str, Any]  # seasonality, demand, quality
    data_sources: List[str]
    
class DemandPrediction:
    produce_type: str
    location: str
    prediction_date: datetime
    predicted_demand_tons: float
    confidence_interval: Tuple[float, float]
    seasonal_factor: float
    trend: str  # 'increasing', 'stable', 'decreasing'
```

### 10. Climate Risk Service

**Responsibility:** Generates climate risk alerts and recommendations.

**Key Operations:**
- `analyzeClimateRisk(parcelId, forecastPeriod) → List<ClimateRisk>`
- `generateAlert(riskId) → Alert`
- `getRecommendations(riskType, parcelId) → List<Recommendation>`
- `calculateWaterStressIndex(parcelId) → WaterStressIndex`
- `predictSoilHealth(parcelId, futureDate) → SoilHealthPrediction`

**Interfaces:**
```python
class ClimateRisk:
    risk_id: UUID
    parcel_id: UUID
    risk_type: str  # 'drought', 'heat_stress', 'pest_outbreak', 'flood'
    severity: str  # 'low', 'medium', 'high', 'critical'
    probability: float  # 0-1
    impact_start_date: datetime
    impact_end_date: datetime
    affected_species: List[str]
    mitigation_actions: List[str]
    data_sources: List[str]
    
class WaterStressIndex:
    parcel_id: UUID
    calculation_date: datetime
    index_value: float  # 0-100 (0=no stress, 100=severe stress)
    rainfall_deficit_mm: float
    soil_moisture_percentage: float
    evapotranspiration_rate: float
    groundwater_level_meters: Optional[float]
    irrigation_recommendation: str
    
class SoilHealthIndex:
    parcel_id: UUID
    sample_date: datetime
    overall_score: int  # 0-100
    ph_level: float
    organic_matter_percentage: float
    nitrogen_ppm: float
    phosphorus_ppm: float
    potassium_ppm: float
    recommendations: List[str]
```

### 11. Analytics Service

**Responsibility:** Provides dashboards, reports, and aggregated metrics.

**Key Operations:**
- `getDashboardMetrics(userId, role) → DashboardData`
- `generateReport(reportType, filters, format) → Report`
- `aggregateMetrics(level, dateRange) → AggregatedMetrics`
- `exportData(query, format) → ExportFile`

**Interfaces:**
```python
class DashboardData:
    user_id: UUID
    role: str
    metrics: Dict[str, Any]
    charts: List[ChartData]
    alerts: List[Alert]
    last_updated: datetime
    
class AggregatedMetrics:
    aggregation_level: str  # 'village', 'block', 'district', 'state'
    location_id: str
    date_range: Tuple[datetime, datetime]
    total_farmers: int
    total_land_area_hectares: float
    total_trees_planted: int
    tree_survival_rate: float
    total_carbon_credits_issued: float
    total_farmer_income: Decimal
    average_income_per_farmer: Decimal
    top_species: List[Tuple[str, int]]
```

## Data Models

### Core Entity Relationships

```
Farmer (1) ──── (N) LandParcel
LandParcel (1) ──── (N) TreeRecord
LandParcel (1) ──── (N) SatelliteImage
SatelliteImage (1) ──── (N) BiomassEstimate
BiomassEstimate (1) ──── (N) CarbonCalculation
CarbonCalculation (1) ──── (N) CarbonToken
CarbonToken (1) ──── (N) PaymentTransaction
Farmer (1) ──── (N) ProduceListing
LandParcel (1) ──── (N) ClimateRisk
LandParcel (1) ──── (N) SoilHealthIndex
```

### Database Schema Design

**PostgreSQL Tables:**

```sql
-- Farmers table
CREATE TABLE farmers (
    farmer_id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20) NOT NULL UNIQUE,
    alternate_phone VARCHAR(20),
    village VARCHAR(100),
    district VARCHAR(100),
    state VARCHAR(100),
    pin_code VARCHAR(10),
    preferred_language VARCHAR(10),
    digital_wallet_address VARCHAR(255),
    registration_date TIMESTAMP NOT NULL,
    ownership_validation_status VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_farmers_phone ON farmers(phone_number);
CREATE INDEX idx_farmers_location ON farmers(district, state);

-- Land parcels table
CREATE TABLE land_parcels (
    parcel_id UUID PRIMARY KEY,
    farmer_id UUID NOT NULL REFERENCES farmers(farmer_id),
    boundaries GEOGRAPHY(POLYGON, 4326) NOT NULL,
    area_hectares DECIMAL(10, 4) NOT NULL,
    registration_date TIMESTAMP NOT NULL,
    validation_status VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_parcels_farmer ON land_parcels(farmer_id);
CREATE INDEX idx_parcels_boundaries ON land_parcels USING GIST(boundaries);

-- Tree records table
CREATE TABLE tree_records (
    tree_id UUID PRIMARY KEY,
    parcel_id UUID NOT NULL REFERENCES land_parcels(parcel_id),
    farmer_id UUID NOT NULL REFERENCES farmers(farmer_id),
    qr_tag VARCHAR(50) NOT NULL UNIQUE,
    species_id UUID NOT NULL,
    planting_date DATE NOT NULL,
    gps_location GEOGRAPHY(POINT, 4326),
    status VARCHAR(20) NOT NULL,
    growth_stage VARCHAR(20),
    mortality_date DATE,
    mortality_cause TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_trees_parcel ON tree_records(parcel_id);
CREATE INDEX idx_trees_qr ON tree_records(qr_tag);
CREATE INDEX idx_trees_status ON tree_records(status);

-- Carbon tokens table
CREATE TABLE carbon_tokens (
    token_id UUID PRIMARY KEY,
    blockchain_token_id VARCHAR(255) UNIQUE,
    parcel_id UUID NOT NULL REFERENCES land_parcels(parcel_id),
    farmer_id UUID NOT NULL REFERENCES farmers(farmer_id),
    carbon_calculation_id UUID NOT NULL,
    co2_tons DECIMAL(10, 4) NOT NULL,
    minting_date TIMESTAMP NOT NULL,
    verification_date TIMESTAMP NOT NULL,
    auditor_id UUID NOT NULL,
    current_owner VARCHAR(255),
    status VARCHAR(20) NOT NULL,
    transaction_hash VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tokens_farmer ON carbon_tokens(farmer_id);
CREATE INDEX idx_tokens_parcel ON carbon_tokens(parcel_id);
CREATE INDEX idx_tokens_status ON carbon_tokens(status);
```

**TimescaleDB Tables (Time-Series Data):**

```sql
-- Satellite imagery metadata
CREATE TABLE satellite_images (
    image_id UUID NOT NULL,
    parcel_id UUID NOT NULL,
    capture_date TIMESTAMP NOT NULL,
    data_source VARCHAR(50),
    resolution_meters DECIMAL(6, 2),
    cloud_cover_percentage DECIMAL(5, 2),
    quality_flag VARCHAR(20),
    storage_url TEXT,
    PRIMARY KEY (image_id, capture_date)
);

SELECT create_hypertable('satellite_images', 'capture_date');

-- NDVI time series
CREATE TABLE ndvi_measurements (
    measurement_id UUID NOT NULL,
    parcel_id UUID NOT NULL,
    image_id UUID NOT NULL,
    measurement_date TIMESTAMP NOT NULL,
    mean_ndvi DECIMAL(5, 4),
    std_ndvi DECIMAL(5, 4),
    vegetation_health_score INT,
    PRIMARY KEY (measurement_id, measurement_date)
);

SELECT create_hypertable('ndvi_measurements', 'measurement_date');
```

**MongoDB Collections (Flexible Schemas):**

```javascript
// Biomass estimates collection
{
  _id: ObjectId,
  estimate_id: UUID,
  parcel_id: UUID,
  image_id: UUID,
  estimation_date: ISODate,
  biomass_tons_per_hectare: Number,
  confidence_interval: [Number, Number],
  confidence_score: Number,
  methodology: String,
  model_version: String,
  model_inputs: {
    ndvi: Number,
    evi: Number,
    tree_count: Number,
    species_distribution: {},
    soil_type: String,
    climate_zone: String
  },
  validation_status: String,
  created_at: ISODate
}

// Anomaly detection results
{
  _id: ObjectId,
  anomaly_id: UUID,
  parcel_id: UUID,
  detection_date: ISODate,
  anomaly_type: String,
  severity: String,
  affected_area_hectares: Number,
  estimated_trees_impacted: Number,
  ndvi_change_percentage: Number,
  before_image_id: UUID,
  after_image_id: UUID,
  comparison_analysis: {
    change_map_url: String,
    affected_regions: [],
    statistical_summary: {}
  },
  investigation_status: String,
  investigation_findings: String,
  created_at: ISODate
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Unique Identifier Generation

*For any* entity creation operation (farmer profile, land parcel, tree record, QR tag, carbon token), the generated identifier SHALL be unique across all existing entities of that type in the system.

**Validates: Requirements 1.1, 1.7, 2.2**

### Property 2: GPS Boundary Validation

*For any* set of GPS coordinates submitted for land parcel registration, the system SHALL accept the coordinates if and only if they form a closed polygon with at least 3 points, where the first and last coordinates are identical or within 1 meter of each other.

**Validates: Requirements 1.2**

### Property 3: Land Area Calculation Consistency

*For any* valid closed polygon of GPS coordinates, the calculated land area SHALL be positive and SHALL remain consistent when the coordinate order is reversed (clockwise vs counter-clockwise).

**Validates: Requirements 1.3**

### Property 4: Entity Data Completeness

*For any* created entity (farmer profile, tree record, satellite image, carbon token, payment transaction, produce listing), all required fields as defined in the data model SHALL be present and non-null.

**Validates: Requirements 1.4, 2.1, 3.5, 4.7, 6.2, 8.4, 9.1**

### Property 5: Parcel Overlap Detection

*For any* newly registered land parcel, if the parcel's boundaries overlap with an existing parcel by more than 10% of either parcel's area, the system SHALL flag the registration for manual review.

**Validates: Requirements 1.6, 7.2**

### Property 6: Species Validation

*For any* tree species identifier submitted during tree planting, the system SHALL accept it if and only if the species exists in the predefined species database.

**Validates: Requirements 2.3**

### Property 7: Tree Lifecycle State Transitions

*For any* tree record, when a mortality event is logged, the tree status SHALL transition to 'deceased', the mortality date SHALL be recorded, and the tree SHALL no longer be included in active tree counts for carbon calculations.

**Validates: Requirements 2.4**

### Property 8: QR Tag Round-Trip Retrieval

*For any* tree record created with a QR tag, querying the system by that QR tag SHALL return a tree record with identical core attributes (tree_id, species, planting_date, parcel_id, farmer_id).

**Validates: Requirements 2.7**

### Property 9: Referential Integrity

*For any* created entity with foreign key references (tree record → parcel, produce listing → farmer, soil sample → parcel), all referenced entities SHALL exist in the system at the time of creation.

**Validates: Requirements 2.6, 9.2, 11.6**

### Property 10: Satellite Imagery Subscription

*For any* land parcel registration, the system SHALL create a satellite imagery subscription for the parcel's geographic area, and the subscription SHALL remain active while the parcel status is 'active'.

**Validates: Requirements 3.1**

### Property 11: NDVI Extraction from Imagery

*For any* satellite image processed for a land parcel, the system SHALL extract NDVI values, and the mean NDVI SHALL be within the valid range of -1.0 to 1.0.

**Validates: Requirements 3.3**

### Property 12: Poor Quality Imagery Flagging

*For any* satellite image with cloud cover percentage exceeding 30%, the system SHALL mark the image quality_flag as 'poor' or 'unreliable' and SHALL NOT use it as the sole basis for carbon credit verification.

**Validates: Requirements 3.4**

### Property 13: Time-Series Query Boundaries

*For any* query for historical satellite imagery or NDVI data with a specified date range [start, end], all returned records SHALL have capture_date or measurement_date within the range start ≤ date ≤ end.

**Validates: Requirements 3.6**

### Property 14: Biomass to Carbon Conversion

*For any* biomass estimate in tons per hectare, when carbon sequestration is calculated, the resulting CO₂ equivalent SHALL be positive and SHALL be within the scientifically valid range based on the tree species and age (typically 0.1 to 50 tons CO₂ per hectare per year).

**Validates: Requirements 4.2**

### Property 15: Vegetation Health Score Range

*For any* calculated NDVI value, the derived vegetation health score SHALL be an integer in the range [0, 100], where higher NDVI values produce higher health scores.

**Validates: Requirements 4.3**

### Property 16: Low Confidence Flagging

*For any* biomass estimate with a confidence score below 0.80, the system SHALL mark the estimate's validation_status as 'pending' or 'requires_verification' and SHALL NOT automatically tokenize the associated carbon sequestration.

**Validates: Requirements 4.4**

### Property 17: Species-Specific Biomass Coefficients

*For any* two biomass calculations with identical NDVI and tree density but different tree species, the biomass estimates SHALL differ according to species-specific carbon sequestration rates defined in the species database.

**Validates: Requirements 4.5**

### Property 18: Tree Age Impact on Carbon Calculation

*For any* carbon sequestration calculation, if all other factors are held constant, trees with greater age SHALL produce higher carbon sequestration values up to the species-specific maturity age, after which the rate SHALL stabilize.

**Validates: Requirements 4.6**

### Property 19: Vegetation Decline Anomaly Detection

*For any* pair of consecutive NDVI measurements for a land parcel, if the vegetation health score decreases by more than 20%, the system SHALL generate an anomaly alert with severity proportional to the magnitude of decline.

**Validates: Requirements 5.1**

### Property 20: Anomaly Alert Completeness

*For any* generated anomaly alert, the alert SHALL include the affected parcel_id, estimated tree impact count, comparison imagery references, and SHALL be delivered to the associated farmer and field agent.

**Validates: Requirements 5.2, 5.5**

### Property 21: Sustained Decline Escalation

*For any* land parcel with anomaly alerts in N consecutive measurement periods (N ≥ 3), the most recent alert's severity SHALL be escalated to a higher priority level than the initial alert.

**Validates: Requirements 5.4**

### Property 22: Carbon Token Standardization

*For any* carbon token created in the system, the co2_tons field SHALL equal exactly 1.0, ensuring each token represents one metric ton of CO₂ removal.

**Validates: Requirements 6.3**

### Property 23: Tokenization Prevents Double-Counting

*For any* carbon calculation record, once a carbon token is created referencing that calculation, the calculation's tokenization_status SHALL be marked as 'tokenized', and any subsequent tokenization attempt for the same calculation SHALL be rejected.

**Validates: Requirements 6.4, 6.5, 7.1**

### Property 24: Token Provenance Completeness

*For any* carbon token query, the returned provenance data SHALL include the complete chain: farmer_id → parcel_id → tree records → satellite images → biomass estimates → carbon calculation → verification details → current ownership.

**Validates: Requirements 6.6**

### Property 25: Token Ownership Consistency

*For any* carbon token transfer operation, the blockchain ownership record and the system's current_owner field SHALL be updated atomically, maintaining consistency between on-chain and off-chain state.

**Validates: Requirements 7.3**

### Property 26: Single-Sale Enforcement

*For any* carbon token, once its status is set to 'sold', any subsequent sale attempt for the same token SHALL be rejected, and the token SHALL remain associated with its buyer until explicitly transferred or retired.

**Validates: Requirements 7.4**

### Property 27: Temporal Non-Overlap in Tokenization

*For any* land parcel, when calculating carbon sequestration for a time period [T1, T2], the system SHALL exclude any sub-periods that have already been covered by issued carbon tokens, ensuring no temporal overlap in tokenized carbon credits.

**Validates: Requirements 7.5**

### Property 28: Duplicate Tokenization Audit Logging

*For any* rejected tokenization attempt due to duplicate detection, the system SHALL create an audit log entry containing the timestamp, user_id, calculation_id, and rejection reason.

**Validates: Requirements 7.6**

### Property 29: Payment Distribution Formula

*For any* carbon token sale with sale price P, the payment distribution SHALL allocate exactly 70% to the farmer, 20% as platform fee, 8% as agent commission, and 2% to the carbon fund, such that the sum equals P within rounding tolerance of 0.01.

**Validates: Requirements 8.3**

### Property 30: Payment Transaction Completeness

*For any* initiated payment transaction, the transaction record SHALL include amount, recipient wallet address, timestamp, carbon_token_id reference, and transaction status.

**Validates: Requirements 8.4**

### Property 31: Payment Retry Logic

*For any* payment transaction that fails, the system SHALL retry the transaction up to 3 times, and if all retries fail, SHALL update the status to 'failed' and generate a notification to the farmer.

**Validates: Requirements 8.5**

### Property 32: Payment Confirmation Notification

*For any* payment transaction that reaches 'completed' status, the system SHALL generate and send a confirmation notification to the farmer containing the transaction amount, date, and confirmation code.

**Validates: Requirements 8.6**

### Property 33: Marketplace Listing Query Filtering

*For any* marketplace query with filters (produce_type, location, availability), all returned listings SHALL match ALL specified filter criteria, and no listings that fail any filter SHALL be included in results.

**Validates: Requirements 9.3**

### Property 34: Price Recommendation Generation

*For any* produce listing creation, the system SHALL generate a price recommendation that includes a recommended_price value, a price_range tuple, and a confidence score between 0 and 1.

**Validates: Requirements 9.4**

### Property 35: Listing Status Transition on Sale

*For any* produce listing, when a sale is recorded, the listing status SHALL transition from 'active' to 'sold', the sale_price and sale_date SHALL be recorded, and the listing SHALL no longer appear in active listing queries.

**Validates: Requirements 9.6**

### Property 36: Quality Grade Validation

*For any* produce listing with a quality_grade field, the grade SHALL be one of the predefined valid grades ('A', 'B', 'C'), and invalid grades SHALL be rejected during listing creation.

**Validates: Requirements 9.7**

### Property 37: Drought Condition Alert Generation

*For any* land parcel where weather data shows rainfall below 50% of normal for a 30-day period, the system SHALL generate a water stress alert with risk_type 'drought' for that parcel.

**Validates: Requirements 10.1**

### Property 38: Heat Threshold Alert Generation

*For any* land parcel where temperature forecasts exceed the heat threshold for the parcel's primary tree species, the system SHALL generate a heat stress alert for that parcel.

**Validates: Requirements 10.2**

### Property 39: Climate Alert Notification Delivery

*For any* generated climate risk alert, the system SHALL send notifications to the affected farmer via all configured channels (SMS, mobile app), and each notification SHALL include the risk type and recommended mitigation actions.

**Validates: Requirements 10.4, 10.5**

### Property 40: Multi-Risk Prioritization

*For any* land parcel with multiple concurrent climate risks, the system SHALL assign priority levels to alerts such that higher severity risks and risks with greater tree survival impact receive higher priority values.

**Validates: Requirements 10.6**

### Property 41: Soil Health Index Range

*For any* soil sample data entered for a land parcel, the calculated Soil_Health_Index SHALL be an integer in the range [0, 100], where the score is derived from pH, organic matter, nitrogen, phosphorus, and potassium levels.

**Validates: Requirements 11.2**

### Property 42: Soil Analysis Recommendations

*For any* soil health analysis, the system SHALL generate at least one recommendation for soil amendments or organic inputs based on the nutrient levels and pH.

**Validates: Requirements 11.3**

### Property 43: Water Stress Index Calculation

*For any* land parcel with available rainfall and soil moisture data, the system SHALL calculate a Water_Stress_Index value between 0 and 100, where higher values indicate greater water stress.

**Validates: Requirements 12.1**

### Property 44: High Water Stress Recommendations

*For any* land parcel with a Water_Stress_Index exceeding 70 (critical threshold), the system SHALL generate irrigation recommendations specific to the parcel's tree species and growth stages.

**Validates: Requirements 12.2**

### Property 45: Water Availability Prediction

*For any* seasonal rainfall pattern analysis, the system SHALL generate a water availability prediction for the next 90 days, including a predicted availability value and confidence interval.

**Validates: Requirements 12.3**

### Property 46: Species Water Requirements in Predictions

*For any* water stress prediction for a land parcel, the calculation SHALL incorporate the water requirements of the tree species present on the parcel and their current growth stages.

**Validates: Requirements 12.4**

### Property 47: Replanting Recommendation Generation

*For any* land parcel where tree mortality is recorded, the system SHALL generate replanting recommendations that include at least one recommended species, optimal planting density, and expected carbon sequestration over 10 years.

**Validates: Requirements 13.1, 13.4**

### Property 48: Environmental Factors in Replanting

*For any* replanting recommendation, the recommended species SHALL be selected based on the parcel's current soil health, water availability, and climate conditions, ensuring species suitability.

**Validates: Requirements 13.2**

### Property 49: High-Survival Species Filtering

*For any* replanting recommendation in a region with historical survival data, all recommended species SHALL have historical survival rates exceeding 85% in that region.

**Validates: Requirements 13.3**

### Property 50: Species Ranking by Carbon and Economic Value

*For any* replanting recommendation with multiple species options, the species SHALL be ranked in descending order by a composite score combining expected carbon sequestration potential and economic value.

**Validates: Requirements 13.6**

### Property 51: Language Preference Validation

*For any* farmer profile creation or update, the selected preferred_language SHALL be one of the system's supported languages, and unsupported language codes SHALL be rejected.

**Validates: Requirements 14.1**

### Property 52: Interface Localization

*For any* user session, all interface elements (labels, buttons, messages) SHALL be displayed in the user's preferred_language as specified in their profile.

**Validates: Requirements 14.2**

### Property 53: Notification Localization

*For any* notification sent to a user, the notification content SHALL be in the recipient's preferred_language, and if translation is unavailable, SHALL fall back to English and log the missing translation.

**Validates: Requirements 14.3, 14.4**

### Property 54: Language Change Immediate Effect

*For any* user language preference update, the interface SHALL reflect the new language immediately in the current session without requiring logout or page reload.

**Validates: Requirements 14.6**

### Property 55: Offline Mode Activation

*For any* mobile application instance, when network connectivity is lost (detected by failed ping or connection timeout), the app SHALL automatically enable offline mode and allow data capture operations.

**Validates: Requirements 15.1**

### Property 56: Offline Data Storage with Metadata

*For any* data captured while in offline mode, the system SHALL store the data locally with a timestamp, sync_status field set to 'pending', and a unique local identifier.

**Validates: Requirements 15.2**

### Property 57: Sync Conflict Detection

*For any* offline data synchronization, if the server detects conflicts (e.g., same entity modified both offline and online), the system SHALL flag the conflict for manual resolution and SHALL NOT automatically overwrite either version.

**Validates: Requirements 15.4**

### Property 58: Offline Storage Threshold Alert

*For any* mobile application instance, when offline data storage exceeds 80% of allocated space, the system SHALL display an alert prompting the user to synchronize data.

**Validates: Requirements 15.6**

### Property 59: Data Encryption at Rest

*For any* farmer personal data (name, phone_number, contact_info) stored in the database, the data SHALL be encrypted using AES-256 encryption, and decryption SHALL require proper authentication.

**Validates: Requirements 17.1**

### Property 60: Password Hashing Security

*For any* user password stored in the system, the password SHALL be hashed using bcrypt with a minimum of 12 rounds, and plaintext passwords SHALL NEVER be stored.

**Validates: Requirements 17.3**

### Property 61: Data Access Audit Logging

*For any* access to sensitive data (farmer profiles, carbon calculations, payment transactions), the system SHALL create an audit log entry containing user_id, timestamp, data_type, and record_id.

**Validates: Requirements 17.4**

### Property 62: Authentication Token Expiration

*For any* issued authentication token (JWT or session token), the expiration time SHALL be set to no more than 24 hours from issuance, and expired tokens SHALL be rejected.

**Validates: Requirements 17.5**

### Property 63: Account Lockout on Failed Attempts

*For any* user account, if 5 or more failed login attempts occur within a 15-minute window, the system SHALL temporarily lock the account and send a notification to the user.

**Validates: Requirements 17.6**

### Property 64: Carbon Token Creation Audit

*For any* carbon token creation, the system SHALL create an immutable audit log entry containing token_id, parcel_id, farmer_id, co2_tons, verification_date, auditor_id, and timestamp.

**Validates: Requirements 18.1**

### Property 65: Data Modification Change Tracking

*For any* modification to carbon sequestration data or biomass estimates, the system SHALL record a change log entry with previous_value, new_value, user_id, timestamp, and modification_reason.

**Validates: Requirements 18.2**

### Property 66: Audit Log Query Filtering

*For any* audit log query with filters (date_range, user_id, action_type, parcel_id), all returned log entries SHALL match ALL specified filters.

**Validates: Requirements 18.3**

### Property 67: Suspicious Pattern Flagging

*For any* carbon calculation record that is modified more than 3 times within a 24-hour period, the system SHALL flag the record for auditor review and generate an alert.

**Validates: Requirements 18.4**

### Property 68: Complete Provenance Chain

*For any* carbon token, when a Climate_Auditor requests verification history, the system SHALL provide the complete provenance chain from tree planting records through satellite imagery, biomass estimates, carbon calculations, to the final token sale.

**Validates: Requirements 18.5**

### Property 69: Dashboard Metrics Completeness

*For any* Rural Administrator dashboard request, the displayed data SHALL include at minimum: total_farmers, total_land_area_hectares, total_trees_planted, and total_carbon_credits_issued.

**Validates: Requirements 19.1**

### Property 70: Hierarchical Metric Aggregation

*For any* dashboard metrics aggregated at a specific level (village, block, district, state), the aggregated values SHALL equal the sum of all child-level values within that geographic boundary.

**Validates: Requirements 19.2**

### Property 71: Time-Series Date Range Filtering

*For any* time-series data query with a date range [start_date, end_date], all returned data points SHALL have timestamps within the specified range, and the minimum granularity SHALL be one month.

**Validates: Requirements 19.3**

### Property 72: Performance Report Completeness

*For any* generated performance report, the report SHALL include tree_survival_rate, average_carbon_sequestration_per_hectare, and farmer_income_statistics as required metrics.

**Validates: Requirements 19.4**

### Property 73: Multi-Format Export Support

*For any* data export request, the system SHALL generate the export in both PDF and CSV formats, and both files SHALL contain the same data content with format-appropriate structure.

**Validates: Requirements 19.5**

### Property 74: Image Compression with Quality Preservation

*For any* image displayed in the mobile application, the system SHALL compress the image to reduce file size, and the compressed image SHALL maintain a quality score of at least 70/100 on standard image quality metrics.

**Validates: Requirements 20.3**

### Property 75: Form Input Validation

*For any* form submission in the mobile application, the system SHALL validate all input fields against their defined constraints (data type, range, format), and SHALL reject submissions with invalid data while providing specific error messages.

**Validates: Requirements 20.6**



## Error Handling

### Error Classification

**Client Errors (4xx):**
- 400 Bad Request: Invalid input data, validation failures
- 401 Unauthorized: Missing or invalid authentication token
- 403 Forbidden: Insufficient permissions for operation
- 404 Not Found: Requested resource does not exist
- 409 Conflict: Resource conflict (e.g., duplicate registration)
- 429 Too Many Requests: Rate limit exceeded

**Server Errors (5xx):**
- 500 Internal Server Error: Unexpected server-side error
- 502 Bad Gateway: External service failure (satellite API, blockchain)
- 503 Service Unavailable: Service temporarily down or overloaded
- 504 Gateway Timeout: External service timeout

### Error Response Format

All API errors SHALL return a consistent JSON structure:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Land parcel boundaries must form a closed polygon",
    "details": {
      "field": "boundaries",
      "constraint": "closed_polygon",
      "provided_points": 2,
      "required_points": 3
    },
    "request_id": "uuid",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Error Handling Strategies

**Validation Errors:**
- Validate all inputs at API boundary before processing
- Return specific error messages indicating which field failed and why
- For batch operations, return partial success with list of failed items

**External Service Failures:**
- Implement circuit breaker pattern for satellite imagery APIs
- Retry with exponential backoff (max 3 attempts)
- Fall back to cached data when available
- Queue requests for later processing if service is down

**Database Errors:**
- Wrap all database operations in transactions
- Implement automatic retry for transient failures (connection timeouts)
- Log all database errors with query context
- Return generic error to client, detailed error in logs

**Blockchain Errors:**
- Implement idempotency for token minting operations
- Store pending transactions with retry mechanism
- Verify transaction success before updating local state
- Provide transaction hash for manual verification if automated verification fails

**Offline Sync Conflicts:**
- Detect conflicts by comparing timestamps and version numbers
- Present both versions to user for manual resolution
- Provide merge suggestions based on field-level comparison
- Log all conflict resolutions for audit trail

### Logging Strategy

**Log Levels:**
- ERROR: System errors requiring immediate attention
- WARN: Potential issues that don't block operations
- INFO: Important business events (farmer registration, token minting)
- DEBUG: Detailed diagnostic information

**Structured Logging:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "carbon-token-service",
  "operation": "mintToken",
  "user_id": "uuid",
  "farmer_id": "uuid",
  "parcel_id": "uuid",
  "error_code": "BLOCKCHAIN_TIMEOUT",
  "message": "Failed to mint carbon token after 3 retries",
  "stack_trace": "...",
  "request_id": "uuid"
}
```

### Monitoring and Alerts

**Critical Alerts:**
- Blockchain transaction failures
- Payment processing failures
- Database connection pool exhaustion
- API error rate > 5%
- Satellite imagery processing queue > 1000 pending jobs

**Performance Alerts:**
- API response time p95 > 2 seconds
- Database query time p95 > 500ms
- Satellite imagery processing time > 10 minutes per parcel
- Mobile app crash rate > 1%

## Testing Strategy

### Dual Testing Approach

The GramCarbon platform requires both **unit testing** and **property-based testing** to ensure comprehensive correctness:

**Unit Tests:**
- Verify specific examples and edge cases
- Test integration points between components
- Validate error handling for known failure scenarios
- Test UI components and user interactions
- Cover specific business logic branches

**Property-Based Tests:**
- Verify universal properties across all inputs
- Test system behavior with randomized data
- Ensure invariants hold under all conditions
- Validate mathematical properties (calculations, aggregations)
- Test round-trip operations (serialization, encryption)

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property-based tests verify general correctness across the input space.

### Property-Based Testing Configuration

**Framework Selection:**
- Python: Hypothesis (recommended for backend services)
- TypeScript/JavaScript: fast-check (for mobile app and web dashboard)

**Test Configuration:**
- Minimum 100 iterations per property test (due to randomization)
- Configurable seed for reproducibility
- Shrinking enabled to find minimal failing examples
- Timeout: 60 seconds per property test

**Property Test Tagging:**
Each property-based test MUST include a comment tag referencing the design document property:

```python
# Feature: gramcarbon-platform, Property 1: Unique Identifier Generation
@given(st.lists(st.text(min_size=1), min_size=10, max_size=100))
def test_unique_farmer_ids(farmer_names):
    """For any list of farmer names, all generated farmer IDs must be unique."""
    farmer_ids = [create_farmer_profile(name).farmer_id for name in farmer_names]
    assert len(farmer_ids) == len(set(farmer_ids)), "Duplicate farmer IDs detected"
```

### Test Coverage Requirements

**Backend Services:**
- Unit test coverage: Minimum 80% line coverage
- Property test coverage: All 75 correctness properties implemented
- Integration test coverage: All external API integrations
- End-to-end test coverage: Critical user journeys

**Mobile Application:**
- Unit test coverage: Minimum 70% line coverage
- UI component test coverage: All interactive components
- Offline sync test coverage: All sync scenarios
- Property test coverage: Data validation and calculation properties

**AI/ML Models:**
- Model accuracy: Biomass estimation R² > 0.85 on test set
- NDVI calculation: Validated against ground-truth measurements
- Anomaly detection: Precision > 0.80, Recall > 0.75
- Model versioning: All models tagged with version and training date

### Testing Environments

**Development:**
- Local PostgreSQL, MongoDB, Redis instances
- Mock satellite imagery API
- Mock blockchain (Ganache or similar)
- Mock payment gateway

**Staging:**
- Cloud-hosted databases (scaled-down)
- Real satellite imagery API with test account
- Blockchain testnet (Goerli or similar)
- Payment gateway sandbox

**Production:**
- Full-scale cloud infrastructure
- Production satellite imagery API
- Blockchain mainnet
- Production payment gateway

### Continuous Integration

**CI Pipeline:**
1. Lint and format check (Black, ESLint)
2. Unit tests (pytest, Jest)
3. Property-based tests (Hypothesis, fast-check)
4. Integration tests
5. Security scanning (Bandit, npm audit)
6. Build Docker images
7. Deploy to staging (on main branch)

**Test Execution Time:**
- Unit tests: < 5 minutes
- Property tests: < 15 minutes
- Integration tests: < 10 minutes
- Total CI pipeline: < 30 minutes

### Test Data Management

**Synthetic Data Generation:**
- Use Faker library for farmer profiles
- Generate realistic GPS coordinates within India
- Create synthetic satellite imagery for testing
- Generate time-series data with realistic patterns

**Test Data Isolation:**
- Each test suite uses isolated database schemas
- Cleanup after each test run
- No shared state between tests
- Parallel test execution supported

### Performance Testing

**Load Testing:**
- Simulate 10,000 concurrent users
- Test API endpoints under sustained load
- Measure response times at p50, p95, p99
- Identify bottlenecks and resource constraints

**Stress Testing:**
- Push system beyond normal capacity
- Test auto-scaling behavior
- Verify graceful degradation
- Test recovery after overload

**Satellite Imagery Processing:**
- Test processing pipeline with 1,000 parcels
- Measure throughput (parcels processed per hour)
- Test queue management under high load
- Verify no data loss during processing

### Security Testing

**Penetration Testing:**
- SQL injection attempts
- XSS and CSRF attacks
- Authentication bypass attempts
- Authorization escalation attempts

**Vulnerability Scanning:**
- Automated dependency scanning (Snyk, Dependabot)
- Container image scanning
- Infrastructure scanning (AWS Inspector)
- Regular security audits (quarterly)

### Acceptance Testing

**User Acceptance Testing (UAT):**
- Farmer onboarding workflow
- Tree planting and tracking
- Carbon credit verification and sale
- Payment receipt and confirmation
- Climate alert reception and response

**Field Testing:**
- Deploy to pilot region (1-2 districts)
- Onboard 100-500 farmers
- Monitor system performance and user feedback
- Iterate based on real-world usage patterns

## Deployment Strategy

### Infrastructure as Code

All infrastructure SHALL be defined using Terraform or CloudFormation:
- VPC and networking configuration
- Database instances and replication
- Kubernetes cluster configuration
- Load balancers and auto-scaling groups
- Monitoring and logging infrastructure

### Deployment Pipeline

**Deployment Stages:**
1. Build and test (CI pipeline)
2. Deploy to staging
3. Run smoke tests
4. Manual approval gate
5. Blue-green deployment to production
6. Health checks and monitoring
7. Rollback if health checks fail

### Database Migrations

**Migration Strategy:**
- Use Alembic (Python) or Flyway for schema migrations
- All migrations must be reversible
- Test migrations on staging before production
- Zero-downtime migrations using online schema changes
- Backup database before major migrations

### Monitoring and Observability

**Metrics Collection:**
- Application metrics: Request rate, error rate, latency
- Business metrics: Farmers registered, trees planted, carbon credits issued
- Infrastructure metrics: CPU, memory, disk, network
- Custom metrics: Satellite imagery processing time, blockchain transaction time

**Dashboards:**
- System health dashboard (Grafana)
- Business metrics dashboard
- Cost monitoring dashboard
- Security monitoring dashboard

**Alerting:**
- PagerDuty integration for critical alerts
- Slack notifications for warnings
- Email notifications for daily summaries
- Escalation policies for unacknowledged alerts

## Future Enhancements

### Phase 2 Features

**Water Credit Marketplace:**
- Track water conservation efforts
- Tokenize water savings
- Create marketplace for water credits

**Soil Carbon IoT Integration:**
- Deploy soil sensors for real-time monitoring
- Measure soil carbon sequestration
- Integrate sensor data with AI models

**AI-Powered Microinsurance:**
- Predict crop failure risks
- Offer parametric insurance products
- Automate claim processing based on satellite data

**Biodiversity Tokenization:**
- Track biodiversity metrics (species count, habitat quality)
- Create biodiversity credits
- Integrate with global biodiversity markets

### Scalability Roadmap

**Year 1:** 10,000 farmers, 50,000 hectares
**Year 2:** 100,000 farmers, 500,000 hectares
**Year 3:** 1,000,000 farmers, 5,000,000 hectares

### Technology Evolution

**AI Model Improvements:**
- Incorporate LiDAR data for better biomass estimation
- Use deep learning for species identification from imagery
- Implement federated learning for privacy-preserving model training

**Blockchain Evolution:**
- Explore layer-2 solutions for lower transaction costs
- Implement cross-chain bridges for broader market access
- Investigate zero-knowledge proofs for privacy

**Mobile App Evolution:**
- Add AR features for tree identification
- Implement voice-based data entry for low-literacy users
- Add offline AI models for instant feedback
