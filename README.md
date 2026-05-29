# Pathology Image Classification Using AI

This project implements an AI pipeline for classifying Whole Slide Images (WSIs) into **Tumor** and **Normal** classes. The work uses CPTAC-LSCC as the main dataset and CPTAC-LUAD for external validation.

The pipeline includes dataset preparation, WSI preprocessing, tissue patch extraction, feature extraction using pretrained models, feature caching, and slide-level classification using Multiple Instance Learning (MIL) methods.

## Project Overview

Whole Slide Images are very large pathology images that cannot be processed directly as a single image. Therefore, each slide is divided into smaller tissue patches. These patches are passed through pretrained feature extractors to generate feature embeddings. The embeddings are then grouped per slide and used to train slide-level classifiers.

The general workflow is:

1. Prepare and split the datasets at the patient level.
2. Extract tissue patches from each WSI at 5× magnification.
3. Use pretrained feature extractors to generate patch embeddings.
4. Save extracted features as `.h5` files.
5. Train MIL classifiers using the extracted features.
6. Evaluate the models on the main test set and external validation dataset.

## Implementation Environment

The project was implemented using **Google Colab**. And **Google Drive** was used to store the WSI datasets and extracted feature files. Cloud-based computing was used because WSIs are very large and cannot be efficiently processed on standard personal machines. IBM Aspera was used to download the CPTAC datasets, and Google Drive Desktop was used to sync the downloaded files to Google Drive so they could be accessed from Colab.

## Repository Structure

```text
Collab Notebooks/
│
├── Classifiers/
│   └── Contains the classifier notebooks for training and evaluating the MIL models.
│       The notebooks use the extracted `.h5` feature files as input.
│
├── Datasets/
│   └── Contains notebooks related to dataset preparation, label handling,
│       patient-level splitting, and leakage verification.
│       It also includes the metadata/CSV file used to assign slide labels.
│
└── Feature_Extractors/
    └── Contains notebooks for extracting features from WSIs.
        This folder also includes model-related files such as pretrained weights.

```

## Datasets

The project uses two CPTAC datasets:

* **CPTAC-LSCC**: main dataset used for training, validation, and testing.
* **CPTAC-LUAD**: external validation dataset.

The dataset was split at the **patient level** to avoid patient-level data leakage. This means slides from the same patient were not allowed to appear in more than one split.

Original Dataset Links (IBM Aspera):

   **CPTAC-LSCC**: 
https://faspex.cancerimagingarchive.net/aspera/faspex/public/package?context=eyJyZXNvdXJjZSI6InBhY2thZ2VzIiwidHlwZSI6ImV4dGVybmFsX2Rvd25sb2FkX3BhY2thZ2UiLCJpZCI6IjY3MCIsInBhc3Njb2RlIjoiN2QzZjU0ZDY4Y2JiNjllZTIwZGRkYWI1YWU5N2Q1MDY5MjNlN2ZmNCIsInBhY2thZ2VfaWQiOiI2NzAiLCJlbWFpbCI6ImhlbHBAY2FuY2VyaW1hZ2luZ2FyY2hpdmUubmV0In0=&redirected=true&authenticated=true
   
   **CPTAC-LUAD**: 
https://faspex.cancerimagingarchive.net/aspera/faspex/public/package?context=eyJyZXNvdXJjZSI6InBhY2thZ2VzIiwidHlwZSI6ImV4dGVybmFsX2Rvd25sb2FkX3BhY2thZ2UiLCJpZCI6IjEwOTUiLCJwYXNzY29kZSI6ImZkNDdkZGNmMGZiZTQyNWFlYWFhYmFiNzBjMTAxNzkzODcyZjcxODMiLCJwYWNrYWdlX2lkIjoiMTA5NSIsImVtYWlsIjoiaGVscEBjYW5jZXJpbWFnaW5nYXJjaGl2ZS5uZXQifQ==&redirected=true&authenticated=true

Dataset split links (Google Drive):

   **CPTAC-LSCC**: https://drive.google.com/drive/folders/1xn69OTswxweXWxzVi-TEXTQGWzUlv0lp?usp=sharing
  

   **CPTAC-LUAD**: https://drive.google.com/drive/folders/1fek4SDpOBn5hKtPuIRsgEDk-2XP4XMA1?usp=sharing


## Preprocessing and Patch Extraction

Preprocessing was implemented to preform the following steps:

1. Opens each `.svs` WSI using OpenSlide.
2. Reads the slide magnification metadata.
3. Selects the correct OpenSlide pyramid level for 5× magnification.
4. Creates a thumbnail of the WSI for tissue detection.
5. Applies grayscale thresholding and morphological closing to detect tissue regions.
6. Keeps patches where at least 20% of the area contains tissue.
7. Reads valid tissue patches from the slide.
8. Normalizes patches using ImageNet mean and standard deviation.

Patches were extracted as **512 × 512** pixels at **5× magnification**. For MobileNetV2 and CTransPath, the patches were resized to **224 × 224** because these models require smaller input size.

## Feature Extractors

Three pretrained feature extractors were used.

### KimiaNet

KimiaNet was used as a pathology-specific feature extractor. It is based on DenseNet-121 and outputs a **1024-dimensional feature vector** for each patch.

The pretrained weights were loaded from:

```text
KimiaNetPyTorchWeights.pth
```

### CTransPath

CTransPath was used as a transformer-based pathology feature extractor. It uses a Swin Transformer architecture with a custom convolutional stem. It outputs a **768-dimensional feature vector** for each patch.

The pretrained weights were loaded from:

```text
ctranspath.pth
```

### MobileNetV2

MobileNetV2 was used as a baseline feature extractor. It was loaded from Torchvision using pretrained ImageNet weights. The final classification layer was replaced with an identity layer so the model could output feature embeddings instead of class predictions.

MobileNetV2 outputs a **1280-dimensional feature vector** for each patch.

## Extracted Features

The extracted features are saved as HDF5 `.h5` files. Each `.h5` file represents one slide and contains:

* `features`: patch-level feature embeddings.
* `coords`: original patch coordinates in the WSI.

The extracted features are not stored directly in this repository because they are large. They can be accessed from Google Drive using the links below.

```text
Features/
│
├── CPTAC-LSCC/
│   ├── CPTAC-LSCC_CTransPath_features/
│   │   Link: [https://drive.google.com/drive/folders/1_WGGMtrclyb53HVUGNLyOmaKwLHOKpql?usp=sharing]
│   │
│   ├── CPTAC-LSCC_kimianet_features/
│   │   Link: [https://drive.google.com/drive/folders/1jteOjBIYAYk25ssdqlAvUH37mseJV0zz?usp=sharing]
│   │
│   └── CPTAC-LSCC_MobileNet_features/
│       Link: [https://drive.google.com/drive/folders/17jWtUisEF1zbDZkvX2cEJaoBGOtT6Col?usp=sharing]
│
└── CPTAC-LUAD/
    ├── CPTAC-LUAD_CTransPath_features/
    │   Link: [https://drive.google.com/drive/folders/1vF_f62vfH_19An9i9UoI5Qms-HsnBwxh?usp=sharing]
    │
    ├── CPTAC-LUAD_kimianet_features/
    │   Link: [https://drive.google.com/drive/folders/1Gk2pg5i-sshiDYFRiKAxLQzEsrVcoFxB?usp=sharing]
    │
    └── CPTAC-LUAD_MobileNet_features/
        Link: [https://drive.google.com/drive/folders/13TIQGAZT2iXkVym8aEEiHDOTHsDf4i0f?usp=sharing]
```

## Classifiers

Two Multiple Instance Learning classifiers were implemented.

### AMIL Classifier

The Attention-based Multiple Instance Learning classifier receives a bag of patch embeddings for each slide. It learns attention weights for the patches and uses these weights to create a slide-level representation. This representation is then used for binary classification.

### Max-Pooling MIL Classifier

The Max-Pooling classifier also receives a bag of patch embeddings for each slide. Instead of learning attention weights, it applies max-pooling across patch features to select the strongest feature responses. The pooled slide-level representation is then passed to a classifier.

## Training Setup

The MIL classifiers were trained using:

* Adam optimizer
* Learning rate: `0.0001`
* Weight decay: `1e-5`
* Batch size: `1`
* Maximum epochs: `30`
* Loss function: Binary Cross-Entropy with Logits Loss
* Early stopping based on validation loss
* Best checkpoint selected using validation AUC

To improve reliability, experiments were repeated across **5 random seeds**, and results were reported as mean ± standard deviation.

## How to Use the Project

### 1. Prepare the datasets

Download the CPTAC-LSCC and CPTAC-LUAD datasets using IBM Aspera and organize them into Tumor and Normal folders. Then run the dataset preparation notebooks inside:

```text
Collab Notebooks/Datasets/
```

These notebooks handle label preparation, patient-level splitting, and leakage checking.

### 2. Run feature extraction

Run the feature extractor notebooks inside:

```text
Collab Notebooks/Feature_Extractors/
```

Each feature extractor notebook processes the WSIs and saves the extracted patch embeddings as `.h5` files.

The `.h5` files should be saved into the corresponding feature folders on Google Drive.

### 3. Train classifiers

Run the classifier notebooks inside:

```text
Collab Notebooks/Classifiers/
```

These notebooks load the extracted `.h5` feature files and train the AMIL and Max-Pooling MIL classifiers.

### 4. Evaluate the models

The trained models are evaluated on the CPTAC-LSCC test set and the CPTAC-LUAD external validation set. The final results are reported using metrics such as accuracy, precision, recall, F1-score, AUC, and confusion matrix.

## Notes

* Dataset and Large `.h5` feature files are not included in the GitHub repository, they can be accessed through the provided Google Drive links.
* Pretrained model weights are included or should be placed inside the feature extractor folder before running the notebooks.
* The code was designed mainly for Google Colab and Google Drive.
