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






在 **MZmine 3** 中分析 Bruker TIMS-TOF Pro 2 生成的数据（通常已转换为 `mzML` 格式）**需要充分利用其离子淌度维度（CCS/1/K0值）**，以显著提升代谢物/脂质鉴定的准确性和异构体分辨能力。以下是关键步骤和针对性设置：

---

## **核心优势：利用离子淌度 (CCS) 维度**
TIMS-TOF Pro 2 的核心价值在于提供**碰撞截面 (CCS)** 信息。在 MZmine 中，这可用于：
1.  **增强峰解卷积/特征检测：** 在保留时间 (RT)、质荷比 (m/z) 基础上，增加 **CCS** 作为第三维分离依据，更好区分共洗脱/同m/z的异构体。
2.  **降低假阳性：** 将实验 CCS 值与数据库/预测值匹配，作为鉴定的强有力佐证。
3.  **分辨异构体：** 区分结构/立体异构体（相同 m/z，不同 CCS）。

---

## **MZmine 3 分析 TIMS-TOF Pro 2 数据工作流程**

### **步骤 1：数据导入与参数设置**
1.  **导入 `mzML` 文件：**
    *   确保转换时**保留了离子淌度信息**（在 MSConvert 中必须使用 `64-bit precision`）。
    *   MZmine 自动识别 `inverse reduced ion mobility (1/K0)` 字段。
2.  **设置原始数据参数 (Raw Data Parameters)：**
    *   **MS level:** 选择 1 和 2。
    *   **m/z range:** 根据实验设定。
    *   **Mobility type:** 确认显示为 `Inverse reduced ion mobility (1/K0)` 或类似。
    *   **Mobility range:** 通常保留默认（全范围），除非有特定范围要求。

### **步骤 2：质量检测 (Mass Detection) - *通常跳过***
*   TIMS-TOF Pro 2 数据在转换时（如用 MSConvert + Peak Picking）**应已中心化 (centroided)**。MZmine 通常可直接使用这些峰，无需再次进行质量检测。如果数据仍是 Profile 模式，需在此步骤进行峰检测。

### **步骤 3：构建离子淌度谱图 (Build Ion Mobility Spectra) - *关键步骤***
*   **目的：** 将连续的淌度漂移数据聚合成离散的离子淌度谱图 (IMS)，类似将连续的质谱扫描聚合成色谱峰。
*   **模块：** `Ion mobility processing` -> `Build ion mobility spectra`
*   **关键参数：**
    *   **Frame merging:** 如何合并连续的 TIMS 帧（通常选 `Sum` 或 `Maximum`）。
    *   **Scoring:** 选择构建 IMS 时评估峰质量的方法（如 `Maximum height`, `Total area`）。
    *   **Noise level:** 设定阈值过滤低强度淌度信号（谨慎设置，避免丢失弱信号）。

### **步骤 4：色谱图构建 (ADAP Chromatogram Builder) - *核心，利用淌度维度***
*   **目的：** 在 RT-m/z-CCS 三维空间中识别色谱峰（特征）。
*   **模块：** `Chromatogram building` -> `ADAP Chromatogram Builder` (推荐，处理复杂数据能力强)。
*   **关键参数 - 优化以利用 CCS:**
    *   **Min group size (scans):** `2` (或 `3`) - 允许在少数扫描中检测到的真实峰。
    *   **Group intensity threshold:** 设置较低值（如 `1000`）以捕获弱峰，后续可过滤。
    *   **Min highest intensity:** `1000 - 5000` (根据数据强度调整)。
    *   **m/z tolerance:** 根据仪器分辨率设置 (e.g., `0.002 m/z` 或 `5 ppm`)。
    *   **✳ 关键 - 利用离子淌度 (CCS 维度):**
        *   **Mobility tolerance:** 设定允许的淌度差异范围以将离子归为同一特征。**这是关键参数！**
            *   单位通常是 `1/K0 * 10^{-3} V*s/cm²` (有时直接显示为 `1/K0`)。
            *   合理起始值： **`0.02 - 0.05`**。需根据数据质量和淌度分辨率优化。太宽会导致共洗脱离子合并，太窄会拆分同一化合物。
        *   **Mobility type:** 确认是 `Inverse reduced ion mobility (1/K0)`。
        *   **Require same mobility?:** 通常 **`Yes`**。强制要求归入同一色谱峰的离子必须有相似的淌度（即在 RT-m/z-CCS 三维空间共迁移）。

### **步骤 5：色谱峰解卷积 (Chromatographic Deconvolution)**
*   **目的：** 分离共洗脱峰（尤其同 m/z 但不同 CCS 的异构体）。
*   **模块：** `Peak deconvolution` -> `Local Minimum Resolver` 或 `Wavelets (ADAP)`。
*   **关键参数：**
    *   **Chromatographic threshold:** `5% - 15%`。
    *   **Search minimum in RT range (scans):** `3 - 5`。
    *   **Min relative height:** `1% - 5%`。
    *   **Min absolute height:** 根据数据设定。
    *   **Min ratio of peak top/edge:** `1.5 - 2`。
    *   **✳ 关键 - 解卷积维度：** 确保算法考虑了 **CCS 维度**。在参数中寻找与 `mobility` 或 `ion mobility` 相关的选项，确保其启用或作为约束条件。

### **步骤 6：同位素分组 (Isotopic Peak Grouper)**
*   识别同一化合物的同位素峰簇。
*   参数设置与常规 LC-MS 分析类似，关注 `m/z tolerance` 和 `RT tolerance`。淌度在此步骤通常不是主要依据，但可设置宽松的 `Mobility tolerance` (如 `0.1`)。

### **步骤 7：峰对齐 (Join Aligner)**
*   将不同样品中的相同特征（RT-m/z-CCS）对齐。
*   **关键参数：**
    *   **m/z tolerance:** `0.002 m/z` 或 `5-10 ppm`。
    *   **RT tolerance:** `0.1 - 0.5 min` (根据色谱分离度)。
    *   **✳ 关键 - 淌度对齐：**
        *   **Mobility tolerance:** **必须设置！** 使用与色谱图构建相同或稍宽的值 (e.g., `0.03 - 0.07`)。这是确保同一化合物在不同样本中通过 **RT-m/z-CCS** 三维信息正确匹配的关键。
        *   **Weight for mobility:** 可赋予较高权重（如 `2`），强调 CCS 匹配的重要性。

### **步骤 8：峰过滤 (Peak Filtering)**
*   根据空白样本、QC 样本、峰形、强度、信号稳定性等过滤假阳性峰。
*   可添加基于 **CCS 值合理性**的过滤（如果已知目标物 CCS 范围）。

### **步骤 9：化合物鉴定 (Compound Identification) - *关键，整合 CCS***
*   **方法：**
    1.  **精确质量匹配：** 利用 `m/z` 搜索数据库 (如 HMDB, LipidMaps, MoNA)。
    2.  **MS/MS 谱图匹配：** 导入 MS2 数据，使用内置或外部工具 (如 SIRIUS+CSI:FingerID, GNPS) 进行谱库匹配或从头预测。
    3.  **✳ 核心优势 - CCS 匹配/过滤：**
        *   **数据库：** 使用包含 CCS 值的数据库 (如部分 HMDB 条目, LipidCCS, MetCCS 预测值)。
        *   **MZmine 操作：**
            *   在鉴定结果中，**添加 CCS 作为关键注释信息**。
            *   **手动/半自动过滤：** 将实验 CCS 值与数据库/预测 CCS 值比较。设定允许偏差（如 **< 2-3%** 相对误差）。显著偏离的候选化合物可降低优先级或排除。
            *   **脚本/自定义模块：** 可使用 MZmine 的脚本功能或开发自定义模块，自动计算 CCS 偏差并打分/排序候选化合物。

### **步骤 10：定量与统计分析**
*   导出包含峰面积/高度的特征表 (含 RT, m/z, CCS)。
*   进行归一化、缺失值填补。
*   使用 MZmine 内置统计模块或导出到外部工具 (如 MetaboAnalyst, R) 进行：
    *   单变量分析 (T-test, ANOVA, Fold Change)。
    *   多变量分析 (PCA, PLS-DA, OPLS-DA) - **可将 CCS 作为附加变量输入**。

### **步骤 11：可视化与解释**
*   **利用淌度维度可视化：**
    *   **Mobilograms (淌度谱图)：** 查看特定 m/z/RT 窗口下的离子淌度分布，识别异构体峰。
    *   **3D 图 (RT vs m/z vs Mobility/Intensity)：** 直观展示特征在三维空间中的分布。
    *   **CCS vs RT / CCS vs m/z 散点图：** 发现不同类别化合物（如脂质类别）的 CCS 分布规律。
*   通路分析 (KEGG, Reactome)、富集分析。

---

## **关键挑战与注意事项**

1.  **CCS 数据库与预测：**
    *   公共 CCS 数据库覆盖度有限。积极查找或使用预测工具 (如 MetCCS, DeepCCS)。
    *   实验条件 (漂移气体、温度、电压) 影响 CCS 值，需确保数据库/预测值与你的实验条件匹配。使用内标校准 CCS 值可提高准确性。
2.  **MZmine 对淌度数据处理成熟度：**
    *   相比 Bruker MetaboScape，MZmine 对 TIMS 数据的深度整合和自动化 CCS 利用仍在发展中。部分步骤 (如基于 CCS 的数据库自动打分过滤) 可能需要手动或脚本辅助。
    *   确保使用 **MZmine 3 的最新版本**，其对离子淌度的支持持续改进。
3.  **参数优化 (尤其淌度容差)：**
    *   `Mobility tolerance` 是平衡灵敏度和特异性的关键。**必须根据仪器淌度分辨率和数据质量仔细优化！** 使用标准品或已知化合物验证。
4.  **异构体分析：**
    *   对于已知存在异构体的化合物 (如脂质、糖类)，重点关注它们在淌度维度的分离情况。比较不同样品间异构体比例的差异可能具有生物学意义。
5.  **DIA (dia-PASEF) 数据处理：**
    *   MZmine 主要设计用于 DDA 和全扫描模式。处理 **DIA (dia-PASEF)** 数据更复杂，需要：
        *   使用支持 DIA 解卷积的算法 (MZmine 相关模块可能仍在开发或实验性)。
        *   或先生成谱图库 (如从 DDA 数据)，再进行靶向提取 (类似 Skyline)。目前 MZmine 对 DIA 的支持不如 Spectronaut/DIA-NN 成熟。

---

## **总结**

在 MZmine 3 中分析 TIMS-TOF Pro 2 数据，**核心在于将离子淌度 (CCS/1/K0) 作为与保留时间 (RT)、质荷比 (m/z) 同等重要的维度，贯穿整个流程：**

1.  **特征检测 (Chromatogram Building)：** 使用 `m/z`, `RT`, `Mobility tolerance` 三维约束。
2.  **峰对齐 (Alignment)：** 使用 `m/z`, `RT`, `Mobility tolerance` 三维匹配。
3.  **化合物鉴定：** 将 **实验 CCS 值** 与数据库/预测值比对，作为关键过滤/验证依据。
4.  **可视化：** 利用淌度谱图、3D 图等揭示异构体信息。

虽然需要更多手动整合 CCS 信息，且对 DIA 支持有限，但 MZmine 3 仍是分析 TIMS-TOF Pro 2 非靶向代谢/脂质数据的强大免费工具。**充分利用 CCS 维度是其获得高质量、高可信度结果的关键，也是区别于普通 LC-MS 分析的核心优势。** 密切留意 MZmine 对离子淌度支持功能的后续更新。对于 Bruker 平台深度整合和自动化 CCS 利用，MetaboScape 仍是更“一站式”的解决方案。
