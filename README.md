#  CFLRU Page Replacement Algorithm

這是一個關於 **CFLRU (Clean-First LRU)** 頁面置換演算法的實作專案。本專案為分組作業，此 Repository 包含了我負責實作的 **CFLRU 演算法核心** 以及使用團隊開發的模擬框架所進行的**實驗數據產出**。

CFLRU 是一種針對快閃記憶體 (Flash Memory) 最佳化的快取演算法，旨在透過優先驅逐乾淨頁面 (Clean Pages) 來減少昂貴的寫入操作 (Flash Writes)。

##  職責與貢獻 (My Contributions)

在本次分組作業中，主要負責以下項目：

1.  **CFLRU 演算法開發 (`algorithm/cflru.py`)**：

      * **核心邏輯實作**：設計並實作 CFLRU 的 Window-based 驅逐策略，優先保留 Dirty Page 以降低 Flash 寫入成本。
      * **動態視窗調整 (Dynamic Tuning)**：實作 Hill Climbing 演算法，根據當前的 Miss Rate 與 Write Cost 動態調整 Window Size，讓演算法能適應不同的 Workload。
      * **效能優化**：使用 `OrderedDict` 確保搜尋候選頁面時維持 $O(W)$ 的效率。

2.  **實驗模擬執行 (Simulation & Analysis)**：

      * 利用組員提供的測試框架 (`simulate_framework.py`) 執行大規模模擬。
      * 分析 Total Cost、Flash Writes 次數與 Miss Rate等等。

##  檔案結構 (File Structure)

```text
CFLRU/
├── algorithm/               
│   ├── cflru.py             # ✨ [My Work] CFLRU 演算法核心實作
│   ├── lru_algo.py          # [Reference] Standard LRU (Baseline)
│   ├── beladys_min_algo.py  # [Reference] Optimal Baseline
│   └── spec.py              # 演算法介面定義
├── simulate_framework.py    # [Tool] 模擬測試框架 (Used for running experiments)
├── utils.py                 # [Tool] Trace 分析工具
├── data_clean.py            # [Tool] 資料清理工具
└── clean_spc.py             # [Tool] SPC 格式轉換工具
```

##  演算法實作細節 (Algorithm Implementation)

我在 `algorithm/cflru.py` 中實作了以下機制：

### 1\. Window-based Eviction

  * 將 LRU List 的尾端視為一個 **Eviction Window** (預設大小為 Capacity 的 25%)。
  * 當 Cache 滿時，演算法會掃描此 Window：
      * 若找到 **Clean Page**：優先驅逐，避免寫入 Flash。
      * 若 Window 內全為 **Dirty Page**：退化為傳統 LRU，驅逐最舊的頁面。

### 2\. Hill Climbing Dynamic Adjustment

為了適應不同的存取模式，我實作了動態調整機制：

  * **監控週期**：每執行 `1000` 次操作後進行一次評估。
  * **成本函數**：計算 `Cost = Read_Miss + 8 * Write_Eviction`。
  * **爬山演算法**：比較當前週期與上一週期的 Cost，若成本上升，則反轉 Window Size 的調整方向（擴大或縮小），自動尋找最佳參數。

##  執行實驗 (Running Simulations)

我使用 `simulate_framework.py` 來驗證演算法效能。

### 1\. 環境準備

需安裝 `tqdm` 以顯示模擬進度：

```bash
pip install tqdm
```

### 2\. 執行指令

確保目錄下有正確格式的 trace CSV 檔案，然後執行：

```bash
python simulate_framework.py
```

### 3\. 實驗參數設定

在模擬過程中，我針對 Trace 的 Working Set Size 設定了不同的 Cache 容量比例進行壓力測試（於 `simulate_framework.py` 的 `main` 區塊中調整）：

```python
# 測試三種不同的容量佔比
ratios = [0.001, 0.01, 0.1] 
```

這能驗證演算法在極端缺乏空間（0.1%）與空間充裕（10%）情況下的適應能力。

-----

## 實驗結果
<img width="472" height="276" alt="image" src="https://github.com/user-attachments/assets/dc15973c-9a8b-4027-982a-0f82f5fa77dd" />

<img width="471" height="288" alt="image" src="https://github.com/user-attachments/assets/d4c2084a-cc2a-4562-8837-b0c04a5f62df" />
