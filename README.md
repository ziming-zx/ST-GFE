# ST-GFE: Capturing Well Interference with Asynchronous Start

## Citation

Xu, Z., & Leung, J. Y. (2024, September). *Graph-Level Feature Embedding with Spatial–Temporal GCN Method for Interconnected Well Production Forecasting*. In SPE Annual Technical Conference and Exhibition (p. D031S032R006). SPE.

## Introduction

The **ST-Graph-Level Feature Embedding (ST-GFE)** model is designed to account for well interference during production forecasting. The model leverages a GCN to aggregate information from neighbouring wells, providing richer contextual information for downstream tasks. Notably, it handles **asynchronous start times** of neighbouring wells.  

In our experiments, ST-GFE is integrated with the **Masked Encoding-Decoding (MED)** method. This integration offers two advantages:  
1. Predictions are made in a single step, avoiding cumulative error.  
2. The model can be updated in real time as streaming data becomes available.

## Methodology

### Feature Embedding

ST-GFE aggregates neighbouring information using a GCN (Kipf and Welling, 2016). In our setup, **nodes** represent wells, with features including production data, well properties, rock and fluid properties (from wireline logs), and completion parameters. **Edges** represent spatial or operational connections between wells. After aggregation, a global pooling layer transforms the node-level information into a **graph-level embedding vector**.

### Model Structure

In standard GCNs, the production rate of the target well can be diluted or distorted after aggregation, since the aggregated vector represents an average over all nodes. To preserve the target well’s historical information and improve real-time updating, we introduce a separate **self-history encoder**.  

The final model architecture consists of three components:  
- Self-history encoder  
- Neighbourhood encoder  
- Decoder  

These components are illustrated in detail in the accompanying figures.

<img width="765" height="296" alt="image" src="https://github.com/user-attachments/assets/78915e04-9f11-46c2-ab9b-e889df7b6ae9" />

### Production Data Alignment

Incorporating neighbouring well production is challenging due to their differing start times. To properly align time-series data, we standardize by **calendar date** through appropriate padding and masking.  

Each graph centers on a **target well**, with production beginning at time 1 (the datum point) and a prediction span of \( T \). To provide \( T \)-step historical context, we construct an input sequence of length \( 2T \) by prepending an empty segment of length \( T \). Neighbouring wells' time series are aligned relative to this datum point, including up to \( T \) steps of their history.  

Missing values within this \( 2T \)-length sequence are padded with a special placeholder \( \gamma = -999 \), which does not overlap with real production data. These values are masked during training.  
- *(a)*: Production profiles of 10 wells from the same pad.  
- *(b)*: Node features over time, showing how asynchronous start dates are aligned into a uniform matrix with masked entries.

<img width="1056" height="396" alt="image" src="https://github.com/user-attachments/assets/bc401618-45d1-4717-be33-522070950fbf" />

### Real-Time Updating

To enable real-time updating in the MED architecture, the training data is segmented into **input–target pairs**:  
 `
() → (x_1, x_1, ..., x_T);
(x_1) → (x_2, x_3, ..., x_T);
(x_1, x_2) → (x_3, x_4, ..., x_T);
...
(x_1, x_2, ..., x_{T-1}) → (x_T);
`

At step 0, when no production history \( U \) is available, \( x_0 \) is used to initiate predictions. At this point only static and drilling parameters (\( D \) and \( S \)) are valid. Since these segments vary in length, sequences are padded to a uniform size and masked accordingly. (*See accompanying figure.*)

<img width="961" height="268" alt="image" src="https://github.com/user-attachments/assets/c493775a-9f06-42ce-b1da-bdf164323836" />

## Experimental Setup

We use a dataset from the **Central Montney shale gas play**, covering 6,605 wells as of March 2024, including 4,388 active producers (PRISM, 2024). Wells that began production after 2010 and have at least 12 months of history are included.  

- Training set: 2,645 wells producing before Jan 2020.  
- Testing set: 414 wells producing later.  

This split avoids pad-level information leakage.

<img width="1125" height="374" alt="image" src="https://github.com/user-attachments/assets/977ab508-187d-4a31-a409-dc1a63795ed3" />


### Neighbourhood Determination

#### Approach 1: Wellhead Location

We cluster wellhead locations using DBSCAN, with a search radius of 0.001° (~362 ft), identifying 541 pads. To improve accuracy, we calculate the adjacency matrix using the **midpoints of horizontal laterals**, which better represent true spatial proximity. These midpoint-based distances are used in the edge calculations.

<img width="1125" height="455" alt="image" src="https://github.com/user-attachments/assets/4a5d1f22-ef3e-44cc-8ae4-bc384680722b" />


#### Approach 2: Geological Proximity

In some formations, wells from different pads may have parallel and closely spaced trajectories. This method defines neighbourhoods based on wells whose horizontal midpoints fall within 0.005° (~1,810 ft) of the target well’s midpoint.  

This distance is treated as a hyperparameter (0.005° in this study). A sensitivity analysis of this parameter is provided in the paper.

<img width="535" height="428" alt="image" src="https://github.com/user-attachments/assets/45c4f940-1b38-4b7d-a7c3-5d3dad30fed0" />


## Experimental Results

The model forecasts **36 months of production rates** for newly drilled wells.  

We present predicted production curves for 16 randomly selected wells using the ST-GFE-MED model at known history lengths \( l = 0, 5, 10 \) months. Results for both neighbourhood determination approaches are summarized below.

#### Approach 1 results:
<img width="932" height="620" alt="image" src="https://github.com/user-attachments/assets/901b1ac2-2b40-4d74-90fb-b1f0032abc69" />

#### Approach 2 results:
<img width="911" height="606" alt="image" src="https://github.com/user-attachments/assets/0cc6bd94-437d-4c20-b593-ecef0315aceb" />




