# Ember - Elastic Malware Benchmark for Empowering Researchers
aka Endgame Malware BEnchmark for Research

## Overview

The **EMBER** dataset provides a large collection of features extracted from over 1.1 million binary files. The dataset includes:

- **900k Training Samples**:
  - 300k Malicious
  - 300k Benign
  - 300k Unlabeled

- **200k Testing Samples**:
  - 100k Malicious
  - 100k Benign

Additionally, the **Ember/Endgame** team has released open-source code to extract features from additional binaries.

The dataset is called "Elastic" because it is large enough to cover various use cases, making it versatile for malware detection research.

## Models

### Malconv
- **MalConv** is a state-of-the-art deep learning model designed for malware detection. It uses raw binary data to train the model.
  
### Gradient Boosted Tree Model
- A **Gradient Boosted Tree** trained on the **EMBER** dataset using **LightGBM** (without any hyperparameter tuning) outperforms **MalConv** in terms of performance.

---

## Why Ember Was Needed

- **Lack of Similar Datasets**: No other comparable dataset was available for malware detection.
- **File Distribution Restrictions**: Sharing a large number of malicious files is prohibited due to copyright laws. VirusShare and VX Heaven provide benign files, but their distribution is also restricted.
- **Long Data Labeling**: Labeling malware data manually takes a lot of time, so having pre-labeled datasets like EMBER saves researchers a lot of effort.

---

## The PE File Format

**PE (Portable Executable)** is the file format used for Windows executables, DLLs, and fonts. The format is defined by several key components:

### Headers
- **COFF Header**: Contains architecture details, file type, and section count.
- **Optional Header**: Includes linker version, code/data segment sizes, and pointers to imports/exports.
- **Data Directories**:
  - Export/Import functions
  - Debug info
  - Resources (e.g., icons, menus)
  - Relocation tables

### Sections
- **.text**: Contains executable code.
- **.data**: Stores read-only data.
- **.bss**: Holds uninitialized data.
- **.reloc**: For address relocation when a preferred memory address isn't available.
- **.tls**: Thread-local storage (can be exploited in malware).

### Custom Sections
- Malware or packers (e.g., UPX) can create custom sections. A packer can unpack itself in memory when executed.

---

## Static PE Malware Detection

Running malware to detect it is neither fast nor safe, which is why **static analysis** of PE files is preferred.

However, static analysis faces challenges due to techniques used to hide or obfuscate the malware. Some historical techniques for malware detection include:

- **Schlutz et al.**: Extracted features like imported functions and byte sequences, then labeled samples via McAfee scanner.
- **Kolter et al.**: Used byte-level n-grams and text processing techniques (e.g., tf-idf) for malware analysis.
- **Shafiq et al.**: Focused on a minimal set of 7 features from the PE header, showing that many malware samples could be identified using just those key elements.
- **Saxe and Berlin**: Used advanced techniques like byte entropy histograms, processed via deep neural networks.

### More Classifiers:
1. **PE-Miner**
   - **Objective**: To create a machine-learning malware detector with:
     - True Positive Rate (TPR): Over 99%
     - False Positive Rate (FPR): Less than 1%
     - Speed: Comparable to signature-based scanners.
   - **Dataset**:
     - **Benign Files**: 1,447 files (details never published).
     - **Malicious Files**: 10,339 samples from VX Heaven and 5,586 samples from Malfease.
   - **Features**: 189 features extracted from the PE file.
   - **Challenges**: Lack of a public dataset prevents proper comparative studies.

2. **Adobe Malware Classifier**
   - **Objective**: Classify malware using a simplified feature set (7 features).
   - **Model**: Decision tree classifier.
   - **Challenges**: The benign dataset heavily consisted of Windows binaries, leading to a bias toward distinguishing non-Windows from Windows files rather than classifying malicious vs. benign files.

3. **Microsoft Malware Classification Challenge (2015)**
   - **Objective**: Provide a benchmark dataset for malware classification.
   - **Dataset**: ~20K malicious samples from 9 malware families.
   - **Features**: Disassembly and bytecode.
   - **Challenges**: The dataset lacked benign files and was difficult to reproduce due to reliance on IDA Pro.

---

## Ember File Structure

The **Ember** dataset contains several key components:

1. **File Information**
   - **SHA-256 Hash**: A unique identifier for each file, allowing researchers to trace back to the original binary or find additional metadata through services like VirusShare or VirusTotal.
   - **Coarse Time Information**: Indicates the approximate month the file was first observed.

2. **Labels**
   - `0`: Benign file.
   - `1`: Malicious file.
   - `-1`: Unlabeled file (for semi-supervised learning research).

3. **Feature Groups**
   The dataset contains eight groups of raw features, including:
   - **General File Information**: File size, PE header info, number of imported/exported functions, and more.
   - **Header Information**: COFF and Optional header details.
   - **Imported Functions**: Libraries and functions parsed from the import address table.
   - **Section Information**: Section name, size, entropy, and characteristics.
   - **Format-Agnostic Histograms**: Byte and entropy histograms.
   - **String Information**: Statistics on printable strings and specific patterns like URLs, registry keys, and file paths.

4. **Research Flexibility**
   - **Raw Features**: Provided for interpretability and custom feature engineering.
   - **Feature Vectorization**: Code included to convert raw features into numeric vectors for machine learning models.

5. **Temporal Splits**
   The dataset is split into training and test sets based on time, simulating real-world malware evolution. This helps test models’ robustness to generational changes.

6. **Source Validation**
   - Benign files are verified on VirusTotal, with no malware detection at the time of collection.
   - Malicious files are verified on VirusTotal with detection by more than 40 vendors.

---

## Experiments

### 1. Building the Model
The EMBER dataset provides raw features that need to be converted into a fixed-size vector (dimension 2351). Feature hashing is used to map strings and other data into numerical values.

### 2. Training the Model
The **LightGBM** library is used to train the Gradient Boosted Decision Tree model. The model is trained with default settings (100 trees, 31 leaves per tree) and takes about 3 hours to train on a MacBook Pro.

### 3. Model Performance
The model achieves an **ROC AUC score** of over 0.9991, indicating excellent performance. At a detection rate above 92.99%, the false positive rate is below 0.1%. When the false positive rate is kept under 1%, the model achieves a detection rate above 98.2%.

### 4. Comparing with Other Models
- **J48 Model**: Showed poor performance with high false positives and negatives.
- **MalConv**: This deep learning model performed slightly worse than LightGBM, with an ROC AUC of 0.99821, and took much longer to train (10 days).

### 5. Key Takeaways
- The **LightGBM** model with parsed features is more efficient and effective than more complex models like **MalConv**, showing that domain-specific features (e.g., file imports/exports) are more useful for this task than using raw binary data alone.

