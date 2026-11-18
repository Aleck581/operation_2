# CVD 鑽石薄膜生長模擬

使用 Python 與 Cantera 模擬化學氣相沉積（Chemical Vapor Deposition, CVD）中的鑽石薄膜生長，觀察原子氫（H）含量變化對薄膜生長速率及表面物種覆蓋率的影響。以 Jupyter Notebook 執行計算，將結果輸出為 CSV，並透過 Matplotlib 繪圖。

## 專案檔案

| 檔案 | 說明 |
| --- | --- |
| `Growth_of_diamond_film_using_CVD.ipynb` | 模型設定、參數掃描、CSV 輸出與結果繪圖 |
| `diamond.csv` | 已產生的 20 組模擬結果；執行 Notebook 時會重新寫入 |

## 環境與安裝

Notebook 的 Python 版本紀錄為 **3.12.15**。需要以下套件：

- `cantera`：氣相與表面反應動力學計算。
- `pandas`：讀取與處理 CSV。
- `matplotlib`：繪製生長速率與表面覆蓋率曲線。
- `jupyterlab`：開啟及執行 Notebook。

在專案目錄建立虛擬環境並安裝：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install cantera pandas matplotlib jupyterlab
```

Windows 的虛擬環境啟用指令為：

```powershell
.venv\Scripts\Activate.ps1
```

目前未鎖定套件版本。Notebook 使用 `diamond.yaml` 機制檔，專案目錄未另附此檔案；執行環境需能透過 Cantera 的資料搜尋路徑找到它。

## 執行方式

1. 在專案目錄啟動 JupyterLab：

   ```bash
   jupyter lab
   ```

2. 開啟 `Growth_of_diamond_film_using_CVD.ipynb`，選擇已安裝上述套件的 Python 核心。
3. 依序執行所有儲存格，或使用 **Run All Cells**。
4. 檢視產生的 `diamond.csv` 與兩張圖表：
   - 原子氫含量與鑽石生長速率。
   - 原子氫含量與各表面物種覆蓋率。

`diamond.csv` 以相對路徑讀寫，輸出位置取決於 Notebook 核心的工作目錄。重新執行計算會覆寫同名檔案，需要保留舊結果時請先另存。

## 模擬設定與流程

| 參數 | 設定 |
| --- | --- |
| 反應機制 | `diamond.yaml` |
| 表面相 | `diamond_100` |
| 相鄰相 | `gas`（氣相）、`diamond`（鑽石體相） |
| 氣相溫度 | 1200 K |
| 氣相壓力 | 20 Torr，以 `20 × ct.one_atm / 760` 換算 |
| 掃描點數 | 20 |
| 原子氫調整 | 每次計算前將組成向量中的 H 分量除以 1.4 |
| 表面覆蓋率積分 | 每組條件呼叫 `advance_coverages(10.0)`，積分 10 秒 |

初始氣相組成取自機制檔。程式逐步降低 H 分量，更新氣相狀態，再積分表面覆蓋率，以接近穩態的表面狀態計算鑽石生長速率。各步驟沿用前一步的表面覆蓋率。

生長速率由體相碳 `C(d)` 的淨生成速率計算：

```text
生長速率（m/s）= 碳的淨生成速率 × 鑽石分子量 / 鑽石密度
生長速率（μm/h）= 生長速率（m/s）× 10⁶ × 3600
```

固定積分 10 秒是目前程式的設定，程式未另外檢查是否已達穩態。

## CSV 欄位

每一列對應一組氫含量條件，共 20 列數據。

| 欄位 | 說明 |
| --- | --- |
| `H mole Fraction` | 程式組成向量中的 H 分量 `x[ih]` |
| `Growth Rate (microns/hour)` | 鑽石薄膜生長速率，單位為 μm/h |
| `c6HH`、`c6H*`、`c6*H`、`c6**`、`c6HM`、`c6HM*`、`c6*M`、`c6B` | 各表面物種的覆蓋率，無因次 |

**氫含量欄位的解讀：** Cantera 在設定 `g.TPX` 時會正規化輸入組成，但目前 CSV 寫入的是正規化前的 `x[ih]`。因此，該欄位及圖表橫軸與氣相實際的 H 莫耳分率可能略有差異；若需要記錄正規化後的數值，可將輸出改為 `g.X[ih]`。

## 調整模擬

- 修改 `t` 與 `p`，調整氣相溫度及壓力。
- 修改 `range(20)`，調整掃描點數。
- 修改 `x[ih] /= 1.4` 的除數，調整氫含量遞減幅度。
- 修改 `advance_coverages(10.0)` 的時間，觀察表面覆蓋率對積分時間的敏感性。

修改後建議從頭執行所有儲存格，以重新初始化組成向量及表面狀態；單獨重跑掃描儲存格會從目前狀態繼續降低 H 分量。

## 常見問題

- **找不到 Python 套件：** 確認 Notebook 使用的核心與安裝套件的虛擬環境一致。
- **找不到 `diamond.yaml`：** 確認 Cantera 的資料目錄包含該機制檔，或將有效的機制檔放入工作目錄，再調整 `ct.Interface(...)` 的檔案路徑。
- **找不到 `diamond.csv`：** 先執行輸出 CSV 的儲存格，並確認讀寫時使用相同工作目錄。
