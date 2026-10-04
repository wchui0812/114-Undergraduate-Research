# 多模態感測融合之 3D 邊框回歸精度及其環境強健性敏感度分析

## 專題資訊

- **專題名稱：** 多模態感測融合之 3D 邊框回歸精度及其環境強健性敏感度分析
- **指導教授：** 陳彥霖
- **執行期間：** 114 學年度第 1 學期至第 2 學期

## 實驗流程影片

[觀看專題實驗流程影片](https://drive.google.com/file/d/1Ib2KKiDGocCwQISwvw792eJ4G7akzr4r/view?usp=sharing)

本專題以自動駕駛感知為應用情境，使用 BEVFusion 比較 **Camera-only** 與 **Camera + LiDAR** 兩種 3D 物件偵測設定，並分析相機影像受到雜訊與低亮度退化時，模型偵測表現的變化。

> 本專題使用 nuScenes-mini 與官方預訓練 checkpoint 進行 evaluation，未重新訓練模型。結果用於初步觀察模型在指定影像退化條件下的相對變化，不代表完整 nuScenes benchmark 成績。

## 研究動機

相機可提供豐富的色彩、紋理與語意資訊，但影像可能受到低光照或感測器雜訊影響。LiDAR 則提供三維幾何與距離資訊。本研究關注：當相機影像品質下降時，融合 LiDAR 的模型是否能維持較穩定的 3D 物件偵測能力。

## 研究目標

- 建立 BEVFusion 在 nuScenes-mini 上的資料前處理、推論與評估流程。
- 比較 Camera-only 與 Camera + LiDAR 模型在乾淨影像下的基準表現。
- 建立 Gaussian noise 與低亮度（Dark）影像退化條件。
- 使用 nuScenes detection metrics 分析整體偵測表現與 3D 邊框相關誤差。
- 觀察 LiDAR 幾何資訊對相機影像退化的補償效果。

## 使用技術

- **模型架構：** BEVFusion
- **資料集：** nuScenes-mini
- **深度學習與偵測框架：** PyTorch、MMDetection3D、MMCV
- **GPU 擴充：** CUDA extensions
- **實驗環境：** Google Colab GPU
- **評估指標：** mAP、NDS、mATE、mASE、mAOE、mAVE、mAAE

## 實驗設計

模型使用 BEVFusion 官方提供的預訓練 checkpoint，分別以 Camera-only 及 Camera + LiDAR 設定執行評估。除了 clean baseline，本研究針對 camera image 建立下列退化條件；實驗中 LiDAR 與其他非相機資料維持原始狀態：

- **Gaussian noise s5：** 對相機影像加入高斯雜訊。
- **Dark s3、Dark s5：** 降低相機影像亮度，以模擬低亮度或曝光不足情境。

建立退化資料後，需確認 nuScenes info pickle 中的影像路徑指向處理後的影像，確保模型實際讀取 corrupted images。

## 實驗結果

| 影像條件 | Camera-only mAP | Camera + LiDAR mAP | Camera-only NDS | Camera + LiDAR NDS |
|---|---:|---:|---:|---:|
| Clean | 0.3349 | 0.5746 | 0.3828 | 0.5803 |
| Gaussian noise s5 | 0.0249 | 0.5276 | 0.0385 | 0.5583 |
| Dark s3 | 0.1373 | 0.5501 | 0.2064 | 0.5690 |
| Dark s5 | 0.0000 | 0.5167 | 0.0000 | 0.5547 |

在本次實驗設定下，Camera-only 模型於高斯雜訊與嚴重低亮度條件下出現明顯性能下降；Camera + LiDAR 模型在相同相機退化條件下仍維持較高的 mAP 與 NDS。結果顯示，融合模型在這些測試條件下較穩定，與 LiDAR 幾何資訊可能提供額外空間線索的假設一致。

以上為 nuScenes-mini 上的初步結果。由於資料規模有限，且兩種設定使用的模型與 checkpoint 不同，分數差異不能單獨視為 LiDAR 對性能提升的因果證明。

## 評估指標簡介

- **mAP：** 衡量物件偵測準確度。
- **NDS：** nuScenes 綜合指標，納入偵測與多項誤差指標。
- **mATE：** 平移／位置誤差。
- **mASE：** 尺寸誤差。
- **mAOE：** 方向誤差。
- **mAVE：** 速度誤差。
- **mAAE：** 屬性誤差。

## 研究限制與後續方向

- nuScenes-mini 場景與樣本數有限，結果適合用於初步趨勢分析；後續可擴充至完整 nuScenes。
- 本次只測試 Gaussian noise 與低亮度退化；可加入模糊、雨霧、遮擋等情境。
- 本次使用官方預訓練模型進行 evaluation，未進行 corruption-aware training。
- 後續可控制模型訓練與評估條件，並探索感測器品質估計或自適應融合方法。

## 實驗環境

專題報告記錄的主要環境如下：

| 項目 | 設定 |
|---|---|
| 平台 | Google Colab |
| GPU | A100 / T4 |
| Python | 3.8 |
| PyTorch | 1.10.1 |
| CUDA | 11.3 |
| 主要框架 | BEVFusion、MMDetection3D、MMCV |

BEVFusion 相依套件及 CUDA extensions 對版本有要求；重現實驗時，請依專案原始環境與安裝文件設定相容版本。

## 參考文獻

1. Z. Liu et al., “BEVFusion: Multi-Task Multi-Sensor Fusion with Unified Bird's-Eye View Representation,” IEEE International Conference on Robotics and Automation, 2023.
2. H. Caesar et al., “nuScenes: A multimodal dataset for autonomous driving,” IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2020.
3. S. Xie et al., “Benchmarking and Improving Bird's Eye View Perception Robustness in Autonomous Driving,” IEEE Transactions on Pattern Analysis and Machine Intelligence, 2025.
