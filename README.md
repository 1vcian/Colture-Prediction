# Documentazione Applicativo per l'Analisi di Appezzamenti Agricoli

## Indice
1. [Introduzione](#introduzione)
2. [Architettura del Sistema](#architettura-del-sistema)
3. [Componenti Server](#componenti-server)
4. [Componenti Client](#componenti-client)
5. [Flusso di Funzionamento](#flusso-di-funzionamento)
6. [Specifiche Tecniche](#specifiche-tecniche)
7. [API](#api)
8. [Requisiti di Sistema](#requisiti-di-sistema)
9. [Installazione e Configurazione](#installazione-e-configurazione)
10. [Considerazioni sulla Scalabilità](#considerazioni-sulla-scalabilità)

## Introduzione

L'applicativo è una soluzione avanzata per l'analisi di appezzamenti agricoli che combina tecnologie di visione artificiale e apprendimento automatico per identificare e classificare terreni agricoli. Il sistema si compone di una parte server per l'elaborazione delle immagini e una parte client per l'interazione con l'utente.

L'applicativo consente agli utenti di:
- Visualizzare ortofoto di terreni agricoli
- Segmentare automaticamente gli appezzamenti tramite semplici input (punti o box)
- Classificare automaticamente il tipo di coltivazione presente (seminativo, arboreo, uliveto, pascolo, ecc.)
- Ottenere geometrie precise degli appezzamenti identificati

La soluzione è particolarmente utile per agronomi, tecnici agricoli, enti pubblici e privati che operano nel settore agricolo e necessitano di una mappatura e classificazione rapida dei terreni.

## Architettura del Sistema

L'architettura dell'applicativo si divide in due componenti principali:

```
┌────────────────────────────────────────┐      ┌───────────────────────────────────┐
│              SERVER                    │      │             CLIENT                │
│                                        │      │                                   │
│  ┌────────────────┐  ┌───────────────┐ │      │  ┌────────────────────────────┐  │
│  │                │  │               │ │      │  │                            │  │
│  │   Modello di   │  │  Modello di   │ │      │  │    Interfaccia Web con     │  │
│  │ Classificazione│  │ Segmentazione │ │      │  │    OpenLayers             │  │
│  │                │  │ (SAM Large)   │ │      │  │                            │  │
│  └────────────────┘  └───────────────┘ │      │  └────────────────────────────┘  │
│                                        │      │                                   │
│  ┌────────────────────────────────────┐│      │  ┌────────────────────────────┐  │
│  │                                    ││      │  │                            │  │
│  │      API REST per l'elaborazione   ││◄─────┼─►│   Gestione Eventi Input    │  │
│  │      delle richieste client        ││      │  │   e Visualizzazione        │  │
│  │                                    ││      │  │   Risultati                │  │
│  └────────────────────────────────────┘│      │  └────────────────────────────┘  │
└────────────────────────────────────────┘      └───────────────────────────────────┘
```

## Componenti Server

### 1. Modello di Classificazione
- **Funzionalità**: Prende in input un'ortofoto di un appezzamento e restituisce la classificazione del tipo di coltivazione.
- **Tipi di classificazione**:
  - Seminativo
  - Arboreo
  - Uliveto
  - Pascolo
  - Vigneto
  - Frutteto
  - Altro
- **Tecnologia**: Deep Learning CNN (Convolutional Neural Network)
- **Precisione stimata**: >85%

### 2. Modello di Segmentazione
- **Funzionalità**: Segmenta l'area dell'appezzamento a partire dall'ortofoto e da un prompt utente
- **Input accettati**:
  - Punto (coordinate x,y)
  - Bounding box (coordinate x1,y1,x2,y2)
- **Output**: Maschera di segmentazione e confini poligonali dell'appezzamento
- **Tecnologia**: SAM Large (Segment Anything Model) fine-tuned su dataset agricoli
- **Precisione stimata**: IoU >90%

### 3. API REST
- Endpoint per la segmentazione
- Endpoint per la classificazione
- Endpoint per operazioni combinate (segmentazione + classificazione)
- Gestione delle immagini e caching

## Componenti Client

### 1. Visualizzazione Mappa
- **Tecnologia**: OpenLayers
- **Layer primario**: WMS (Web Map Service) per la visualizzazione delle ortofoto
- **Layer secondari**:
  - Layer vettoriale per la visualizzazione dei risultati di segmentazione
  - Layer per l'interazione utente (posizionamento punti, definizione bounding box)

### 2. Interazione Utente
- **Eventi supportati**:
  - Click singolo per posizionare un punto positivo (area da includere)
  - Click con modificatore (es. Shift+Click) per posizionare un punto negativo (area da escludere)
  - Drag per definire un bounding box
- **Controlli**:
  - Pulsanti per la selezione della modalità di interazione
  - Visualizzazione dei risultati di classificazione
  - Esportazione dei risultati (GeoJSON, Shapefile)

### 3. Visualizzazione Risultati
- Panel laterale con informazioni sul risultato della classificazione
- Evidenziazione sulla mappa dell'area segmentata
- Statistiche sull'appezzamento (area, perimetro)

## Flusso di Funzionamento

1. L'utente accede all'interfaccia web e visualizza la mappa con le ortofoto
2. L'utente seleziona un'area di interesse e inserisce un punto o definisce un bounding box
3. Il client invia la richiesta al server con le coordinate e l'estensione dell'ortofoto
4. Il server elabora la richiesta:
   - Il modello di segmentazione identifica i confini dell'appezzamento
   - Il modello di classificazione determina il tipo di coltivazione
5. Il server restituisce al client:
   - La geometria dell'appezzamento segmentato
   - La classificazione della tipologia di terreno
   - Livello di confidenza della classificazione
6. Il client visualizza i risultati sulla mappa e nel pannello informativo

## Specifiche Tecniche

### Server

#### Framework e Linguaggi
- Python 3.9+
- Framework web: FastAPI/Flask
- Gestione modelli: PyTorch 2.0+

#### Specifiche Modello di Classificazione
- Architettura: EfficientNet-B3 o ResNet-50
- Input: Immagine RGB ritagliata sull'area dell'appezzamento
- Output: Vector di probabilità per ciascuna classe
- Fine-tuning: Addestrato su dataset proprietario di immagini agricole

#### Specifiche Modello di Segmentazione
- Architettura: SAM Large fine-tuned
- Input: Immagine RGB + coordinate prompt (punto o box)
- Output: Maschera binaria + poligono vettoriale
- Dimensioni supportate: Fino a 1024x1024 pixel per elaborazione

### Client

#### Framework e Linguaggi
- JavaScript (ES6+)
- HTML5/CSS3
- Framework UI: React o Vue.js

#### Librerie
- OpenLayers 6.x per la visualizzazione della mappa
- Turf.js per elaborazioni geometriche lato client
- React/Vue componenti per l'UI

#### Modalità Responsive
- Supporto per dispositivi desktop (prioritario)
- Adattabilità base per tablet
- UI ottimizzata per interazioni mouse/trackpad

## API

### Endpoint: `/api/segment`

**Request**:
```json
{
  "image_url": "string", // URL dell'ortofoto o identificativo WMS
  "bbox": [x1, y1, x2, y2], // Coordinate dell'area da analizzare
  "prompt_type": "point|box", // Tipo di prompt
  "prompt_data": {
    // Per punto: {x: float, y: float, is_positive: boolean}
    // Per box: {x1: float, y1: float, x2: float, y2: float}
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
  "confidence": 0.95 // Confidence della segmentazione
}
```

### Endpoint: `/api/classify`

**Request**:
```json
{
  "image_url": "string", // URL dell'ortofoto o identificativo WMS
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
    "class": "seminativo|arboreo|uliveto|pascolo|...",
    "confidence": 0.87,
    "alternatives": [
      {"class": "seminativo", "probability": 0.87},
      {"class": "pascolo", "probability": 0.09},
      {"class": "altro", "probability": 0.04}
    ]
  }
}
```

### Endpoint: `/api/analyze`

**Request**:
```json
{
  "image_url": "string", // URL dell'ortofoto o identificativo WMS
  "bbox": [x1, y1, x2, y2], // Coordinate dell'area da analizzare
  "prompt_type": "point|box", // Tipo di prompt
  "prompt_data": {
    // Per punto: {x: float, y: float, is_positive: boolean}
    // Per box: {x1: float, y1: float, x2: float, y2: float}
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
    "class": "seminativo|arboreo|uliveto|pascolo|...",
    "confidence": 0.87,
    "alternatives": [
      {"class": "seminativo", "probability": 0.87},
      {"class": "pascolo", "probability": 0.09},
      {"class": "altro", "probability": 0.04}
    ]
  },
  "statistics": {
    "area_ha": 1.45,
    "perimeter_m": 487.2
  }
}
```

## Requisiti di Sistema

### Server
- CPU: 8+ core (consigliato)
- RAM: 16GB+ (32GB consigliato)
- GPU: NVIDIA con 8GB+ VRAM per inferenza rapida
- Storage: 100GB+ per modelli e cache
- OS: Linux (Ubuntu 20.04 LTS o superiore)

### Client
- Browser moderno: Chrome 80+, Firefox 78+, Edge 85+, Safari 14+
- WebGL supportato
- Connessione Internet stabile (5+ Mbps)
- Risoluzione display: minimo 1366x768px (consigliato 1920x1080px)

## Installazione e Configurazione

### Setup Server
1. Installazione dipendenze Python:
   ```bash
   pip install -r requirements.txt
   ```
2. Download dei modelli pre-addestrati:
   ```bash
   python scripts/download_models.py
   ```
3. Configurazione server:
   ```bash
   cp config.example.yaml config.yaml
   # Modificare le impostazioni nel file config.yaml
   ```
4. Avvio server:
   ```bash
   python server.py
   ```

### Setup Client
1. Installazione dipendenze:
   ```bash
   npm install
   ```
2. Configurazione:
   ```bash
   cp .env.example .env
   # Modificare le impostazioni nel file .env
   ```
3. Build per produzione:
   ```bash
   npm run build
   ```
4. Avvio in modalità sviluppo:
   ```bash
   npm run dev
   ```

## Considerazioni sulla Scalabilità

### Elaborazione Parallela
- Supporto per l'elaborazione di multiple richieste in parallelo
- Sistema di code per gestire picchi di richieste

### Cache dei Risultati
- Caching delle segmentazioni per aree frequentemente analizzate
- Caching delle classificazioni per ottimizzare le prestazioni

### Deployment su Cloud
- Supporto per deployment su infrastrutture cloud (AWS, GCP, Azure)
- Containerizzazione con Docker e orchestrazione con Kubernetes
- Autoscaling basato sul carico

### Ottimizzazioni Modelli
- Possibilità di utilizzare versioni quantizzate dei modelli per inferenza più veloce
- Edge deployment per scenari con connettività limitata
- Strategia di rolling update per aggiornamenti modelli senza downtime
