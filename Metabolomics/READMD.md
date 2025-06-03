在使用 **MSConvert (ProteoWizard)** 将 Bruker TIMS-TOF Pro 2 生成的 `.d` 原始数据目录转换为开放格式 `mzML` 时，**合理设置参数对于去除噪音、保留有效信号、提高后续分析的质量至关重要**。以下是推荐的参数设置策略，特别针对去噪和优化 TIMS 数据：

## 核心原则

1.  **保留必要信息：** 特别是关键的 **TIMS 维度信息**（离子淌度/CCS 值）和高质量的 MS/MS 谱图。
2.  **去除低置信度信号：** 在转换阶段过滤掉明显是仪器或化学噪音的低强度、低质量信号。
3.  **平衡文件大小与信息量：** 过度过滤可能丢失弱信号（如低丰度肽段/代谢物），需根据实验目的调整。

## 推荐 MSConvert 参数设置 (GUI 或命令行)

### 1. 输入/输出
*   **Input:** 选择 Bruker `.d` 目录。
*   **Output:**
    *   **Format:** `mzML` (推荐) 或 `mzXML`。`mzML` 是更现代、信息更全的标准。
    *   **Binary Encoding Precision:** **选择 `64-bit`**。这对于保留 TIMS-TOF Pro 2 产生的高精度质量数和离子淌度值（特别是补偿电压 `inverse reduced ion mobility` `1/K0`）**至关重要**。`32-bit` 精度可能导致信息损失和舍入误差。
    *   **Compression:** `zlib` compression (gzip)。显著减小文件大小，几乎不影响读取速度。
    *   **Write Index:** `Yes`。加速后续软件读取。

### 2. 过滤与处理 (关键去噪步骤)
*   **Peak Picking:**
    *   **这是最重要的去噪步骤！** TIMS-TOF Pro 2 原始数据是`profile mode`（连续模拟信号），包含大量噪音点。
    *   **必须勾选 ✅ `Peak Picking`。**
    *   **Algorithm:** 通常选 `Vendor` (即使用 Bruker 库中的算法，效果最好) 或 `cwt` (连续小波变换，效果也不错，是默认选项)。
    *   **MS Levels:** **勾选 `1` 和 `2`**。对 MS1 和 MS/MS (MS2) 谱图都进行峰检测。
    *   **作用：** 将连续的 profile 数据转换成离散的 `centroided` (峰中心) 数据。这个过程本身就会滤除大量低于噪音阈值的点，只保留被识别为真实离子信号的峰（有峰高、峰宽、面积）。
*   **Precursor Refinement:**
    *   如果数据是 **DDA (PASEF)**，勾选此选项有助于更准确地确定 MS2 谱图对应的母离子信息（母离子 m/z、电荷态、强度）。这对后续数据库搜索很重要。对于 DIA (dia-PASEF)，通常不需要。
*   **Title Maker:** 可选，用于自定义输出谱图的标题。

### 3. 特定于 MS Level 的过滤 (高级去噪)
*   点击 `Filters` 选项卡，添加针对不同 MS 级别的过滤器：
    *   **MS1:**
        *   **`MS Level` = 1**
        *   **`Threshold`:** 设置一个**绝对强度阈值**或**相对强度阈值 (%)** 来过滤 MS1 噪音峰。
            *   *策略：* 查看原始数据中典型噪音区域的强度。一个常见的起点是 **`absolute` 设为 100-500 counts** (根据仪器状态和样品复杂度调整，丰度高的样本可设高些，痕量样本设低些)。或者使用 **`relative` 设为 0.01-0.1%** (保留强度高于最高峰强度 0.01%-0.1% 的峰)。**注意：过度过滤会丢失弱信号！** 如果对后续分析有信心（如DIA-NN/Spectronaut有自己的去噪能力），这里可以设得宽松些（如绝对阈值 50 或相对 0.005%），甚至不设，让下游软件处理。
        *   *(可选) **`m/z Window`:** 如果知道特定 m/z 范围是纯噪音（如聚合物、常见污染物），可以排除。一般不需要。*
    *   **MS2 (MSn):**
        *   **`MS Level` = 2-` (或 `2`)**
        *   **`Threshold`:** **强烈建议设置一个较低的绝对强度阈值**。MS2 谱图中包含大量低强度碎片离子噪音，对鉴定贡献小却增加文件大小和搜索时间。一个保守且常用的设置是 **`absolute` 设为 `100 counts`**。也可以尝试 `50` 或 `200`，根据数据质量调整。相对阈值（如 0.1%）也可行。
        *   *(可选) **`Scan Event` / `Charge State`:** 如果需要，可以基于扫描事件或电荷态过滤，一般不需要。*
*   **TIMS 信息保留:** 确保 **没有过滤器会删除 `inverse reduced ion mobility` (`1/K0`) 或 `compensation voltage` (`CV`) 等字段**。这些是 TIMS 数据的核心，默认转换过程会自动包含它们。

### 4. 其他重要设置
*   **`Verbosity`:** 设置为 `Status` 或 `Progress` 即可，方便查看转换进度和是否有报错。
*   **`Use zlib compression`:** 如前所述，勾选以压缩输出文件。

## 总结推荐配置 (GUI 截图对应项)

| 选项卡        | 设置项                     | 推荐值/操作                                    | 说明                                                                 |
| :------------ | :------------------------- | :--------------------------------------------- | :------------------------------------------------------------------- |
| **Input/Output** | Format                   | `mzML`                                         | 首选格式                                                             |
|               | Binary Encoding Precision | **`64-bit`**                                   | **关键！** 保留高精度 m/z 和淌度值                                   |
|               | Compression              | `zlib` (gzip)                                  | 压缩文件大小                                                         |
|               | Write Index              | `Yes`                                          | 加速后续读取                                                         |
| **Filtering**    | **✅ Peak Picking**       | Algorithm: `Vendor` 或 `cwt`; MS Levels: `1, 2`| **核心去噪步骤！** 将 Profile 转 Centroid，滤除大量噪音点。         |
|               | ✅ Precursor Refinement   | (DDA 数据推荐勾选)                             | 优化 DDA 母离子信息                                                  |
| **Filters**     | *(添加 Filter)*           | `MS Level` = `1`                               | 针对 MS1 谱图                                                        |
|               |                          | + `Threshold`: `absolute` = **`100`** (或 `relative` = `0.05%`) | 滤除 MS1 低强度噪音峰 **(谨慎调整，避免过滤过严)** |
|               | *(添加 Filter)*           | `MS Level` = `2-`                              | 针对所有 MS2 及以上谱图                                              |
|               |                          | + `Threshold`: `absolute` = **`100`**          | **强烈推荐！** 滤除 MS2 低强度碎片噪音，显著减小文件大小和提升搜索效率 |
| **Advanced**    | (通常默认)                |                                                |                                                                      |

## 命令行示例

```bash
msconvert path/to/your_data.d \
  --outdir path/to/output_folder \
  --outfile your_data.mzML \
  --mzML \
  --64 \
  --zlib \
  --filter "peakPicking true 1-" \ # 对所有MS Level做峰检测 (cwt算法是默认)
  --filter "msLevel 1" \          # 选择MS1
  --filter "threshold absolute 100 most-intense" \ # 对MS1应用绝对强度阈值100
  --filter "msLevel 2-" \         # 选择MS2及以上
  --filter "threshold absolute 100 most-intense" \ # 对MS2应用绝对强度阈值100
  --filter "precursorRefine"      # 执行母离子优化 (DDA推荐)
```

## 重要注意事项

1.  **阈值选择是艺术：** `100 counts` 是一个常用起点，但**没有绝对标准**。需要根据：
    *   **样品复杂度：** 复杂样品（如全细胞裂解液）背景噪音高，阈值可稍高；简单样品（如分馏后）或痕量样品阈值应降低。
    *   **仪器状态和校准：** 灵敏度高的仪器噪音可能更低。
    *   **后续软件：** 像 **DIA-NN** 和 **Spectronaut** 内部有强大的去噪和信号提取算法，MSConvert 的阈值可以设得**更宽松一些**（如 MS2 阈值设为 `50` 甚至 `0`，只做峰检测不过滤），让下游软件发挥优势。对于 **MaxQuant** 或传统的搜索引擎（如 Comet, MSFragger），MS2 设置一个合理的阈值（如 `100`）能显著加快搜索速度和降低假阳性。
    *   **实验目的：** 如果目标是寻找极低丰度物质（如磷酸化位点、痕量代谢物），阈值**必须设低**（甚至只做峰检测，不加强度过滤）。
2.  **先小范围测试：** 选择一个有代表性的 `.d` 文件（如包含 QC 样本或中等复杂度的样本），用不同参数（特别是阈值）进行转换。将转换后的 `mzML` 导入下游分析软件（如 DIA-NN, MaxQuant, MS-DIAL）进行快速测试，比较鉴定结果数、定量重复性、文件大小等，找到最佳平衡点。
3.  **不要过度依赖转换阶段去噪：** MSConvert 的过滤是初步的。主要的去噪、特征检测（Feature Finding）、峰提取（Peak Picking）和定量通常在后续专用软件（如 MaxQuant, DIA-NN, Spectronaut, MS-DIAL, MZmine）中进行，这些软件有更复杂的算法。转换阶段的重点是**保证数据质量、保留关键信息（特别是 CCS！）和去除明显无效噪音**。
4.  **CCS 值保留：** 只要使用了 `--64` (`-64` in GUI) 选项并正确进行了峰检测，离子淌度信息（`inverse reduced ion mobility` `1/K0`）会自动包含在 `mzML` 文件中。后续软件（如 MetaboScape, Skyline, DIA-NN, Spectronaut）会利用这些信息计算或匹配 CCS 值。
5.  **软件版本：** 使用**最新版本**的 ProteoWizard (MSConvert)，以确保对 Bruker TIMS 数据格式的最佳支持和 bug 修复。

**遵循以上参数设置（特别是强制峰检测、64位精度和合理的MS2阈值），能有效地在转换阶段去除大量噪音，生成高质量的 `mzML` 文件，为后续 TIMS-TOF Pro 2 数据的深度分析（利用CCS维度！）奠定坚实基础。** 记得根据你的具体样本和下游流程微调阈值。
