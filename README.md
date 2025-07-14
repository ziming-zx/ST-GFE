# ST-GFE: Capturing Well Interference at Asyncronize start

## Citation
Xu, Z., & Leung, J. Y. (2024, September). Graph-Level Feature Embedding with Spatial–Temporal GCN Method for Interconnected Well Production Forecasting. In SPE Annual Technical Conference and Exhibition (p. D031S032R006). SPE.

## Introduction
ST-graph-level feature embedding (ST-GFE) is designed for counting well interference while conducting production forecasting. The model uses GCN to aggregate neighboring information and provide more comprehensive contextual information for downstream tasks. Th model allows asyncrinozed start time of the neighbouring wells. In our experiment, ST-GFE is integrated with Masked Encoding Decoding (MED) method, it gives two features to the model 1. the model make prediction in one step, with no accumulative error, 2. it allows real time updating with stream data comes in.

## Methodology
### Feature Embedding
### Model Structure
### Production Data Alignment
### Real Time Updating


## Experiment Setup
The dataset is from the Central Montney shale gas play, with 6,605 wells as of March 2024, including 4,388 active producers (PRISM, 2024). We analyze producers starting after 2010, using practical and widely available features (Table 4). Wells producing before Jan 2020 are used for training (2,645 wells), and later wells for testing (414 wells), avoiding pad-level information leakage. All wells have at least 12 months of production history. Figure 5 shows the training and testing well locations.
### Neighbourhood Determination

## Experiment Results

