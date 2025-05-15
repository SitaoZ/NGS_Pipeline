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

  
### 1. 


### 2.

### 3.


### 4. humann
[参考](https://github.com/biobakery/humann)
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
