# Documentation for Agricultural Plot Analysis Application

## Table of Contents
1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [Server Components](#server-components)
4. [Client Components](#client-components)
5. [Operational Flow](#operational-flow)
6. [Technical Specifications](#technical-specifications)
7. [API](#api)
8. [System Requirements](#system-requirements)
9. [Installation and Configuration](#installation-and-configuration)
10. [Scalability Considerations](#scalability-considerations)

## Introduction

The application is an advanced solution for agricultural plot analysis that combines computer vision technologies and machine learning to identify and classify agricultural lands. The system consists of a server component for image processing and a client component for user interaction.

The application allows users to:
- View orthophotos of agricultural lands
- Automatically segment plots using simple inputs (points)
- Automatically classify the type of cultivation present (arable land, arboreal, olive grove, pasture, etc.)
- Obtain precise geometries of the identified plots

The solution is particularly useful for agronomists, agricultural technicians, and public and private entities operating in the agricultural sector that require rapid mapping and classification of lands.

## System Architecture

The application architecture is divided into two main components:

```
┌────────────────────────────────────────┐      ┌───────────────────────────────────┐
│              SERVER                    │      │             CLIENT                │
│                                        │      │                                   │
│  ┌────────────────┐  ┌───────────────┐ │      │  ┌────────────────────────────┐   │
│  │                │  │               │ │      │  │                            │   │
│  │ Classification │  │ Segmentation  │ │      │  │    Web Interface with      │   │
│  │     Model      │  │     Model     │ │      │  │       OpenLayers           │   │
│  │                │  │  (SAM Large)  │ │      │  │                            │   │
│  └────────────────┘  └───────────────┘ │      │  └────────────────────────────┘   │
│                                        │      │                                   │
│  ┌────────────────────────────────────┐│      │  ┌────────────────────────────┐   │
│  │                                    ││      │  │                            │   │
│  │      REST API for processing       ││◄─────┼─►│   Input Event Management   │   │
│  │      client requests               ││      │  │   and Result Visualization │   │
│  │                                    ││      │  │                            │   │
│  └────────────────────────────────────┘│      │  └────────────────────────────┘   │
└────────────────────────────────────────┘      └───────────────────────────────────┘
```

## Server Components

### 1. Classification Model
- **Functionality**: Takes an orthophoto of a plot as input and returns the classification of the cultivation type.
- **Classification types**:
  - Arable land
  - Rice coltures
  - Arboreal
  - Olive grove
  - Pasture
  - Vineyard
  - Orchard
  - Other
- **Technology**: Deep Learning CNN (Convolutional Neural Network)
- **Framework**: Hugging Face Transformers with SafeTensor format
- **Estimated accuracy**: >85%

### 2. Segmentation Model
- **Functionality**: Segments the plot area from the orthophoto and a user prompt
- **Accepted inputs**:
  - Point (x,y coordinates)
  - _Note: In the initial version, only point input is supported_
- **Output**: Segmentation mask and polygonal boundaries of the plot
- **Technology**: SAM Large (Segment Anything Model) fine-tuned on agricultural datasets
- **Framework**: Hugging Face Transformers with SafeTensor format
- **Estimated accuracy**: IoU >90%

### 3. REST API
- Endpoint for segmentation
- Endpoint for classification
- Endpoint for combined operations (segmentation + classification)
- Image management and caching

## Client Components

### 1. Map Visualization
- **Technology**: OpenLayers
- **Primary layer**: WMS (Web Map Service) for orthophoto visualization
- **Secondary layers**:
  - Vector layer for displaying segmentation results
  - Layer for user interaction (point placement)

### 2. User Interaction
- **Supported events**:
  - Single click to place a positive point (area to include)
  - Click with modifier (e.g., Shift+Click) to place a negative point (area to exclude)
  - _Note: In the initial version, only point input is supported_
- **Controls**:
  - Buttons for selecting interaction mode
  - Display of classification results
  - Export of results (GeoJSON, Shapefile)

### 3. Result Visualization
- Side panel with information on classification result
- Highlighting of the segmented area on the map
- Plot statistics (area, perimeter)

## Operational Flow

1. The user accesses the web interface and views the map with orthophotos
2. The user selects an area of interest and inserts a point
3. The client sends the request to the server with the coordinates and the extent of the orthophoto
4. The server processes the request:
   - The segmentation model identifies the plot boundaries
   - The classification model determines the type of cultivation
5. The server returns to the client:
   - The geometry of the segmented plot
   - The classification of the land type
   - Classification confidence level
6. The client displays the results on the map and in the information panel

## Technical Specifications

### Server

#### Frameworks and Languages
- Python 3.9+
- Web framework: FastAPI/Flask
- Model management: Hugging Face Transformers with SafeTensor
- Inference runtime: PyTorch 2.0+ with hardware acceleration support

#### Classification Model Specifications
- Architecture: EfficientNet-B3 or ResNet-50
- Input: RGB image cropped to the plot area
- Output: Probability vector for each class
- Fine-tuning: Trained on proprietary dataset of agricultural images
- Framework: Hugging Face Transformers with SafeTensor format
- Quantization: Supported for optimized inference

#### Segmentation Model Specifications
- Architecture: Fine-tuned SAM Large
- Input: RGB image + prompt coordinates (currently only points supported)
- Output: Binary mask + vector polygon
- Supported dimensions: Up to 1024x1024 pixels for processing
- Framework: Hugging Face Transformers with SafeTensor format
- Model format: SafeTensor for greater execution security

### Client

#### Frameworks and Languages
- JavaScript (ES6+)
- HTML5/CSS3
- UI Framework: React or Vue.js

#### Libraries
- OpenLayers 6.x for map visualization
- Turf.js for client-side geometric processing
- React/Vue components for UI

#### Responsive Mode
- Support for desktop devices (priority)
- Basic adaptability for tablets
- UI optimized for mouse/trackpad interactions

## API

### Endpoint: `/api/segment`

**Request**:
```json
{
  "image_url": "string", // URL of the orthophoto or WMS identifier
  "bbox": [x1, y1, x2, y2], // Coordinates of the area to analyze
  "prompt_type": "point", // In the initial version only "point" is supported
  "prompt_data": {
    // Point format: {x: float, y: float, is_positive: boolean}
  }
}
```

**Response**:
```json
{
  "status": "success|error",
  "segmentation": {
    "type": "Polygon",
    "coordinates": [[[x1, y1], [x2, y2], ... , [xn, yn], [x1, y1]]]
  },
  "confidence": 0.95 // Segmentation confidence
}
```

### Endpoint: `/api/classify`

**Request**:
```json
{
  "image_url": "string", // URL of the orthophoto or WMS identifier
  "geometry": {
    "type": "Polygon",
    "coordinates": [[[x1, y1], [x2, y2], ... , [xn, yn], [x1, y1]]]
  }
}
```

**Response**:
```json
{
  "status": "success|error",
  "classification": {
    "class": "arable|arboreal|olive_grove|pasture|...",
    "confidence": 0.87,
    "alternatives": [
      {"class": "arable", "probability": 0.87},
      {"class": "pasture", "probability": 0.09},
      {"class": "other", "probability": 0.04}
    ]
  }
}
```

### Endpoint: `/api/analyze`

**Request**:
```json
{
  "image_url": "string", // URL of the orthophoto or WMS identifier
  "bbox": [x1, y1, x2, y2], // Coordinates of the area to analyze
  "prompt_type": "point", // In the initial version only "point" is supported
  "prompt_data": {
    // Point format: {x: float, y: float, is_positive: boolean}
  }
}
```

**Response**:
```json
{
  "status": "success|error",
  "segmentation": {
    "type": "Polygon",
    "coordinates": [[[x1, y1], [x2, y2], ... , [xn, yn], [x1, y1]]]
  },
  "seg_confidence": 0.95,
  "classification": {
    "class": "arable|arboreal|olive_grove|pasture|...",
    "confidence": 0.87,
    "alternatives": [
      {"class": "arable", "probability": 0.87},
      {"class": "pasture", "probability": 0.09},
      {"class": "other", "probability": 0.04}
    ]
  },
  "statistics": {
    "area_ha": 1.45,
    "perimeter_m": 487.2
  }
}
```

## System Requirements

### Server
- CPU: 8+ cores (recommended)
- RAM: 16GB+ (32GB recommended)
- GPU: NVIDIA with 8GB+ VRAM for fast inference
- Storage: 100GB+ for models and cache
- OS: Linux (Ubuntu 20.04 LTS or higher)

### Client
- Modern browser: Chrome 80+, Firefox 78+, Edge 85+, Safari 14+
- WebGL supported
- Stable Internet connection (5+ Mbps)
- Display resolution: minimum 1366x768px (1920x1080px recommended)

## Installation and Configuration

### Server Setup
1. Python dependencies installation:
   ```bash
   pip install -r requirements.txt
   ```
2. Hugging Face Transformers installation:
   ```bash
   pip install transformers safetensors
   ```
3. Download pre-trained models from Hugging Face Hub:
   ```bash
   python scripts/download_models_from_hub.py
   ```
4. Server configuration:
   ```bash
   cp config.example.yaml config.yaml
   # Modify settings in config.yaml file
   ```
5. Server startup:
   ```bash
   python server.py
   ```

### Client Setup
1. Dependencies installation:
   ```bash
   npm install
   ```
2. Configuration:
   ```bash
   cp .env.example .env
   # Modify settings in .env file
   ```
3. Production build:
   ```bash
   npm run build
   ```
4. Development mode startup:
   ```bash
   npm run dev
   ```

## Scalability Considerations

### Parallel Processing
- Support for processing multiple requests in parallel
- Queue system for managing request peaks

### Result Caching
- Caching of segmentations for frequently analyzed areas
- Caching of classifications to optimize performance

### Cloud Deployment
- Support for deployment on cloud infrastructures (AWS, GCP, Azure)
- Containerization with Docker and orchestration with Kubernetes
- Autoscaling based on load

### Model Optimizations
- Possibility to use quantized model versions for faster inference
- Edge deployment for scenarios with limited connectivity
- Rolling update strategy for model updates without downtime
