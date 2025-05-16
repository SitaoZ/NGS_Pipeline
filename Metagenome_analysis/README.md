## 16S

## Metagenome
宏基因组分析是对一个生态系统中的所有微生物DNA进行分析的方法，可以了解微生物的多样性、功能和互作关系。

宏基因组的应用范围：
- 生物多样性研究：通过宏基因组了解不同环境中微生物的的多样性和分析
- 生态学研究：通过宏基因组可以了解微生物在生态系统中的功能，互作关系和生态位。
- 生物技术：宏基因组可以筛选特定的工程微生物。

宏基因组的分析步骤：
- DNA提取与建库
- 高通量测序
- 数据清洗和组装
- 基因注释
- 数据分析
- MAGs binning

  
### 1. fastp
```bash
for sample in aH1_B aH2_A aH2_B aH3_A aH3_B aH4_B aH6_B aH7_B aL1_A aL1_B aL3_B aL4_A aL4_B aL6_A aL6_B aL7_B H1_B H2_A H2_B H3_A H3_B H4_B H5_B H7_A H7_B L1_A L1_B L4_A L4_B L7_A L7_B;
do
fastp -i ../00.data/customer/2025/0303/PN20250123027/MbPL202502259/1_rawdata/${sample}_R1.fq.gz \
      -I ../00.data/customer/2025/0303/PN20250123027/MbPL202502259/1_rawdata/${sample}_R2.fq.gz \
      -o ${sample}_R1.clean.fq.gz \
      -O ${sample}_R2.clean.fq.gz \
      --detect_adapter_for_pe \
      -w 48 \
      -j ${sample}.json \
      -h ${sample}.html
done

```


### 2. remove host 
移除宿主的DNA
```bash
for sample in aH1_B aH2_A aH2_B aH3_A aH3_B aH4_B aH6_B aH7_B aL1_A aL1_B aL3_B aL4_A aL4_B aL6_A aL6_B aL7_B H1_B H2_A H2_B H3_A H3_B H4_B H5_B H7_A H7_B L1_A L1_B L4_A L4_B L7_A L7_B;
do
        bowtie2 -p 48 -x /data/zhusitao/database/animal/Homo_Sapiens/hg38/bowtie2/hg38 -1 ../01.fastp/${sample}_R1.clean.fq.gz -2 ../01.fastp/${sample}_R2.clean.fq.gz | samtools view -bS --threads 20 - > ${sample}.meta.bam
        samtools sort ${sample}.meta.bam -o ${sample}.meta.reads.sorted.bam --threads 24
        bedtools bamtobed -i ${sample}.meta.reads.sorted.bam > ${sample}.meta.reads.sorted.bed
        cat ${sample}.meta.reads.sorted.bed | cut -f 4 | awk -F "/" '{print $1}' | sort | uniq > ${sample}.exclude_list.txt
        seqkit grep -v -f ${sample}.exclude_list.txt -j 16 -i ../01.fastp/${sample}_R1.clean.fq.gz -o ${sample}_R1.clean.remove.fq.gz
        seqkit grep -v -f ${sample}.exclude_list.txt -j 16 -i ../01.fastp/${sample}_R2.clean.fq.gz -o ${sample}_R2.clean.remove.fq.gz
done

```

### 3. kraken2 
reads-based宏基因组分析是指, 不先进行基因组组装, 而直接利用`测序片段`进行分析的方法。该方法通过对短读长进行比对和注释，从中提取功能信息和物种组成，常用于快速评估环境样本中的微生物群落结构和功能潜力。
kraken2是常用的软件之一。
- 数据库下载
```bash
kraken2-build --standard --threads 24 --db ./
```

```bash
for sample in aH1_B aH2_A aH2_B aH3_A aH3_B aH4_B aH6_B aH7_B aL1_A aL1_B aL3_B aL4_A aL4_B aL6_A aL6_B aL7_B H1_B H2_A H2_B H3_A H3_B H4_B H5_B H7_A H7_B L1_A L1_B L4_A L4_B L7_A L7_B;
do
kraken2 --db /data/zhusitao/database/Microbes/kraken2/20250313 \
        --paired --threads 48 \
        --confidence 0.2 \
        --minimum-base-quality 20 \
        --report ${sample}_kraken_taxonomy.txt \
        --output ${sample}_kraken_output.txt \
        --gzip-compressed \
        ../02.bowtie2/${sample}_R1.clean.remove.fq.gz ../02.bowtie2/${sample}_R2.clean.remove.fq.gz
done

```

### 4. humann
humann用于物种和功能注释，是read-based方法，不用组装，直接进行物种注释和功能分析。

[参考1](https://github.com/biobakery/humann)
[参考2](https://zhuanlan.zhihu.com/p/240910229).
- 安装
```bash
pip install humann
# 测试安装正常与否
humann_test
```

- 路径修改
```bash
$ humann_config
# HUMAnN Configuration ( Section : Name = Value )
# database_folders : nucleotide = /home/zhusitao/miniconda3/envs/metaphlan/lib/python3.9/site-packages/humann/data/chocophlan_DEMO
# database_folders : protein = /home/zhusitao/miniconda3/envs/metaphlan/lib/python3.9/site-packages/humann/data/uniref_DEMO
# database_folders : utility_mapping = /home/zhusitao/miniconda3/envs/metaphlan/lib/python3.9/site-packages/humann/data/misc
# run_modes : resume = False
# run_modes : verbose = False
# run_modes : bypass_prescreen = False
# run_modes : bypass_nucleotide_index = False
# run_modes : bypass_nucleotide_search = False
# run_modes : bypass_translated_search = False
# run_modes : threads = 1
# alignment_settings : evalue_threshold = 1.0
# alignment_settings : prescreen_threshold = 0.01
# alignment_settings : translated_subject_coverage_threshold = 50.0
# alignment_settings : translated_query_coverage_threshold = 90.0
# alignment_settings : nucleotide_subject_coverage_threshold = 50.0
# alignment_settings : nucleotide_query_coverage_threshold = 90.0
# output_format : output_max_decimals = 10
# output_format : remove_stratified_output = False
# output_format : remove_column_description_output = False

# 修改nucleotide路径
humann_config  --update database_folders nucleotide /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/chocophlan
# HUMAnN configuration file updated: database_folders : nucleotide = /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/chocophlan

# 修改protein路径
$ humann_config  --update database_folders protein /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/uniref
# HUMAnN configuration file updated: database_folders : protein = /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/uniref
# 修改utility_mapping路径
$ humann_config  --update database_folders utility_mapping /home/zhusitao/miniconda3/envs/metagenome/lib/python3.6/site-packages/humann/data/misc
# HUMAnN configuration file updated: database_folders : utility_mapping = /home/zhusitao/miniconda3/envs/metagenome/lib/python3.6/site-packages/humann/data/misc

```
- 执行
```bash
## 输入文件可以为多种数据形式，包括测序文件及压缩格（fasta, fastq, fasta.gz, fastq,gz) , 比对文件（sam, bam等），还有gene table文件也可以。
## 输出指定路径，运行后一般输出genefamilies，pathcoverage，pathabundance三个文件和比对过程中的中间文件（包括metaphlan的结果）
humann --input $SAMPLE --output $OUTPUT_DIR --threads 24

```
- metaphlan结果合并

```bash
merge_metaphlan_tables.py result/humann/*_metaphlan_bugs_list.tsv | sed 's/_metaphlan_bugs_list//g' | ~/miniconda3/bin/csvtk pretty -t | less
```

- 合并文件
```bash
humann2_join_tables --input $OUTPUT_DIR --output humann2_genefamilies.tsv --file_name genefamilies_relab
```

### 5 contigs-based
使用不同的软件进行组装

#### megahit

#### metaSPAdes


### 6 prodigal

基因预测


### 7 Mmseqs2
去冗余

### 8 salmon

基因表达量定量


### 9 功能注释

eggNOG (COG/KEGG/CAZy)
CARD
dbCAN2
VFDB

### 10 binning分箱

maxbin2
metabat2
