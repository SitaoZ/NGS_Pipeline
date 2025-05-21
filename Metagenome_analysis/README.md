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

- 数据库下载

```bash
# 显示可用的数据库
humann_databases

# 建立下载目录
db=/data/zhusitao/pipeline/Metagenome/software/Humann/
mkdir -p ${db}/db_humann3

# 自动下载
# 微生物泛基因组 16 GB
humann_databases --download chocophlan full ${db}/db_humann3
# 功能基因diamond索引 20 GB
humann_databases --download uniref uniref90_diamond ${db}/db_humann3
# 输助比对数据库 2.6 GB
humann_databases --download utility_mapping full ${db}/db_humann3


# 无法自动下载可以手动
wget ftp://download.nmdc.cn/tools/meta/humann3/full_chocophlan.v201901_v31.tar.gz
wget ftp://download.nmdc.cn/tools/meta/humann3/full_mapping_v201901b.tar.gz
wget ftp://download.nmdc.cn/tools/meta/humann3/uniref90_annotated_v201901b_full.tar.gz
# 安装、解压
mkdir -p ${db}/db_humann3/chocophlan
tar xvzf full_chocophlan.v201901_v31.tar.gz -C ${db}/db_humann3/chocophlan
mkdir -p ${db}/db_humann3/utility_mapping
tar xvzf full_mapping_v201901b.tar.gz -C ${db}/db_humann3/utility_mapping
mkdir -p ${db}/db_humann3/uniref
tar xvzf uniref90_annotated_v201901b_full.tar.gz -C ${db}/db_humann3/uniref

```

- 核对数据库
```bash
humann_config --print
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

# 增加线程数

humann_config --update run_modes threads 24

# 修改nucleotide路径
humann_config  --update database_folders nucleotide /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/chocophlan
# HUMAnN configuration file updated: database_folders : nucleotide = /data/zhusitao/project/songLab/01.Metagenome/qxx/06.Meta/05.humann/humann_database_location/chocophlan

# 设置核酸、蛋白和注释库位置
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

- 结果文献
[Output](https://github.com/biobakery/humann?tab=readme-ov-file#output-files)
HUMAnN生成5个结果文件，`Gene families file`, `Reactions file`, `Pathway abundance file`, `MetaPhlAn file`, `intermediate temp output files`
- Gene families
```bash
# Gene Family	$SAMPLENAME_Abundance-RPKs
READS_UNMAPPED        187.0
UniRef50_unknown        150.0
UniRef50_unknown|g__Bacteroides.s__Bacteroides_fragilis 150.0
UniRef50_A6L0N6: Conserved protein found in conjugate transposon	67.0
UniRef50_A6L0N6: Conserved protein found in conjugate transposon|g__Bacteroides.s__Bacteroides_fragilis	57.0
UniRef50_A6L0N6: Conserved protein found in conjugate transposon|g__Bacteroides.s__Bacteroides_finegoldii	5.0
UniRef50_A6L0N6: Conserved protein found in conjugate transposon|g__Bacteroides.s__Bacteroides_stercoris	4.0
UniRef50_A6L0N6: Conserved protein found in conjugate transposon|unclassified	1.0
UniRef50_O83668: Fructose-bisphosphate aldolase	60.0
UniRef50_O83668: Fructose-bisphosphate aldolase|g__Bacteroides.s__Bacteroides_vulgatus	31.0
UniRef50_O83668: Fructose-bisphosphate aldolase|g__Bacteroides.s__Bacteroides_thetaiotaomicron	22.0
UniRef50_O83668: Fructose-bisphosphate aldolase|g__Bacteroides.s__Bacteroides_stercoris	7.0
```

Reactions file 文件HUMAnN v3不再输出

- Pathway abundance
```bash
# Pathway	$SAMPLENAME_Abundance
UNMAPPED	140.0
UNINTEGRATED	87.0
UNINTEGRATED|g__Bacteroides.s__Bacteroides_caccae	23.0
UNINTEGRATED|g__Bacteroides.s__Bacteroides_finegoldii	20.0
UNINTEGRATED|unclassified	12.0
PWY0-1301: melibiose degradation	57.5
PWY0-1301: melibiose degradation|g__Bacteroides.s__Bacteroides_caccae	32.5
PWY0-1301: melibiose degradation|g__Bacteroides.s__Bacteroides_finegoldii	4.5
PWY0-1301: melibiose degradation|unclassified	3.0
PWY-5484: glycolysis II (from fructose-6P)	54.7
PWY-5484: glycolysis II (from fructose-6P)|g__Bacteroides.s__Bacteroides_caccae	16.7
PWY-5484: glycolysis II (from fructose-6P)|g__Bacteroides.s__Bacteroides_finegoldii	8.0
PWY-5484: glycolysis II (from fructose-6P)|unclassified	6.0
```


- MetaPhlAn结果合并
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
seqkit stats result/megahit/contigs/*.contigs.fa > result/megahit/contig_stats


#### metaSPAdes

#### quast
组装质量评估
[软件地址](https://github.com/ablab/quast)


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
