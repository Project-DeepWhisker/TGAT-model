# TGAT Model

## abstract
This notebook implements a Temporal Graph Attention Network (TGAT) using PyTorch Geometric and PyTorch Geometric Temporal.

**Goal:** Achieve higher training efficiency by using a custom temporal neighbor loader for batching and neighbor sampling.

**Structure:**
1. Setup & Configuration
2. Utilities
3. Custom Temporal Loader Implementation
4. Data Loading & Preprocessing
5. Model Definition (TGAT - Adapted for Batch Processing)
6. Training (Using Custom Temporal Loader)
7. Evaluation (Using Custom Temporal Loader)
8. Main Execution

## 環境建置

### 1. 建立 Conda 環境（不包含 PyTorch）
請先在本專案資料夾中執行以下指令：

```bash
conda env create -f environment.yaml
conda activate tgat_model
```

### 2. 安裝與您 CUDA 版本相容的 PyTorch

本環境未預設安裝 PyTorch，請使用者依照自己的 GPU 與驅動狀況安裝對應版本。
可參考 [PyTorch 官網](https://pytorch.org/get-started/locally/) 自動生成安裝指令。


- 無 GPU / CPU-only 使用者：
```bash
pip install torch torchvision torchaudio
```
⚠️ 請務必安裝正確版本，否則模型訓練將無法啟用 GPU 加速。

## 資料集

NSL-KDD
- 原始檔：KDDTrain+_20Percent.txt, KDDTest+.txt
- Notebook 內含 download_file() 函式，可自動下載並放到 ./data/

事件即節點：每一列紀錄 → 節點

邊 (edge_index)：0 → 1 → 2 → … 依時間順序相連；如只剩單筆資料則為空圖

標籤
- binary_attack：正常 (0) / 攻擊 (1)
- attack_label_encoded：多類別（DOS、Probe、R2L、U2R、other_attack…）— 透過 LabelEncoder 映射

## 前處理流程

1. 標籤轉換—`preprocess_labels_event_node()`
   - 攻擊→類別映射 attack_map
   - 產生二元與多類別欄位

2. 特徵處理—`preprocess_features_event_node()`
   - 類別欄：protocol_type, service, flag → OneHotEncoder
   - 數值欄：StandardScaler
   - 常數欄 num_outbound_cmds 自動捨棄

3. 圖組裝—`create_event_node_temporal_data()`
   - x：torch FloatTensor
   - y：torch Long 或 FloatTensor
   - t：事件時間 (序列索引)
   - edge_index：順序邊
