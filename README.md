# CUT&RUN Pipeline 🧬

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Language: Bash](https://img.shields.io/badge/Language-Bash-green.svg)](https://www.gnu.org/software/bash/)
[![Python: 3.9+](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)

A practical shell-based bioinformatics pipeline for analyzing CUT&RUN (Cleavage Under Targets and Release Using Nuclease) data.

This workflow supports:

- raw FASTQ collection
- quality control and genome mapping
- clean FASTQ extraction
- spike-in mapping
- spike-in normalization
- bigWig generation
- peak calling
- mapping statistics summary

This repository reflects a real-world CUT&RUN workflow used on Linux servers for paired-end sequencing data.

---

## 📋 Overview

CUT&RUN is a chromatin profiling method for mapping protein-DNA interactions with high resolution and low background. This repository provides a practical shell-based analysis workflow from raw FASTQ files to processed BAM files, normalized bigWig tracks, and peak calls.

The current implementation is designed for personal or lab use in a Linux environment.

## 🧪 What is CUT&RUN?

CUT&RUN is an in situ method for profiling specific protein-DNA complexes using a target-specific primary antibody together with Protein A/Protein G-micrococcal nuclease (pAG-MNase). Cells are typically immobilized on Concanavalin A-coated magnetic beads, permeabilized, and incubated with a primary antibody that recognizes a histone mark, transcription factor, or cofactor of interest.

The pAG-MNase fusion protein is then recruited to the antibody-bound chromatin region. After calcium ion (Ca2+) activation, MNase cleaves DNA near the target sites, releasing chromatin fragments into the supernatant. These DNA fragments can then be purified and used for downstream qPCR or next-generation sequencing (NGS).

Purified DNA is subsequently used for library construction, genome alignment, signal track generation, and peak calling.

CUT&RUN can be used for profiling histone marks, transcription factors, and other chromatin-associated proteins, and is widely used as an efficient alternative to ChIP-seq in many applications.


<p align="center">
  <img src="https://github.com/SitaoZ/CUT-RUN-snakemake/assets/29169319/052e8504-6a29-4e65-9b79-9978efed0016" />
</p>



---

## ✨ Pipeline Features

| Feature | Description |
|---------|-------------|
| 🔬 **Quality Control** | Read quality assessment and adapter trimming using `fastp` |
| 🗺️ **Mapping** | Read alignment using `bowtie2` |
| 📦 **BAM Processing** | Sorting, duplicate marking, and filtering |
| 🎯 **Peak Calling** | Statistical peak identification using `MACS2` |
| 📊 **Normalization** | Optional spike-in calibration using *E. coli* or `dm6` |
| 📈 **Visualization** | BigWig generation for genome browser display |
| 📋 **Statistics** | Mapping and quality metrics summarization |
| 🔁 **Batch Processing** | Helper scripts for common batch steps on our lab server |

---

## 🚀 Usage Guide

### 1. Prepare environment

For example, on our lab server, the pipeline is usually run in the following conda environment:

```bash
conda activate /mnt/sda/Public/Environment/miniconda3/envs/ChIP
```

If the environment has not been created yet on a new server, it can be recreated with:

```bash
conda env create -f environment.yml
conda activate ChIP
```

### 2. Prepare config

Before running the pipeline, please first place the `pipeline/` folder in your working directory or on the server.

Then edit:

```bash
pipeline/project_paths.sh
```

Only this file needs to be modified before running the workflow.  
You do **not** need to edit each pipeline script individually.

For example, on our lab server, `project_paths.sh` may look like this:

```bash
cat > pipeline/project_paths.sh <<'EOF'
RAW_SOURCE_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260206Cut/rawdata"
PROJECT_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260206Cut/analysis"
PIPELINE_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/pipeline"
CONDA_ENV="/mnt/sda/Public/Environment/miniconda3/envs/ChIP"

THREADS=12
BIN_SIZE=50
PAIR_MODE="yes"
EOF
```

### 3. Run the basic workflow

```bash
# 0. Collect raw FASTQ
bash pipeline/collect_fastq.sh

# 1. QC and genome mapping
bash pipeline/run_qc_map_batch.sh

# 2. Collect clean FASTQ and CPM bigWig
bash pipeline/collect_clean_and_bw.sh
```

If you specifically want CPM-normalized bigWig tracks, run:

```bash
bash pipeline/run_qc_map_batch_CPM.sh
```

In practice, the raw/original version is often used first for initial track inspection, while the CPM version is used for downstream comparison and visualization.

### 4. Prepare spike-in reference

Spike-in mapping depends on the experimental design.  
Different spike-in species require different reference genomes and alignment indexes.

On our lab server, commonly used spike-in references are already available:

- **E. coli**: `/mnt/sda/Public/Database/Ecoli`
- **Drosophila melanogaster (`dm6`)**: `/mnt/sda/Public/Database/Drosophila_melanogaster`

If the required spike-in reference is already available on the server, you can directly proceed to the mapping step below.

A compressed backup package `Ref(spike_in)` is also provided for transfer to a new server.

If you use the reference files from `Ref(spike_in)` on another server, you only need to modify the reference/index path in the corresponding mapping script.

For example:

- update the reference/index path in `02_map_ecoli.sh` for *E. coli*
- update the reference/index path in `02_map_dm6.sh` for `dm6`

<details>
<summary><strong>If the spike-in reference is not available on your server, click here to download and prepare it</strong></summary>

#### Download and build `dm6` spike-in reference

```bash
mkdir -p reference/dm6
cd reference/dm6

wget https://hgdownload.soe.ucsc.edu/goldenPath/dm6/bigZips/dm6.fa.gz
wget https://hgdownload.soe.ucsc.edu/goldenPath/dm6/bigZips/dm6.chrom.sizes

gunzip -c dm6.fa.gz > dm6.fa
bowtie2-build dm6.fa dm6
```

Alternatively, you can use the compressed backup package `Ref(spike_in)`, which contains both *E. coli* and `dm6` references.

</details>

### 5. Spike-in mapping options

Choose the appropriate spike-in mapping script according to your experimental design:

- **Option A. *E. coli* spike-in**
- **Option B. *Drosophila melanogaster* (`dm6`) spike-in**

<details>
<summary><strong>Option A. E. coli spike-in</strong></summary>

If you are using the prebuilt *E. coli* reference on our lab server, you can run the spike-in mapping directly.

If you are running this workflow on another server and downloaded the reference by yourself, please update the following lines in `02_map_ecoli.sh` first:

```bash
ref="/mnt/sda/Public/Database/Ecoli"
chip="/mnt/sda/Public/Environment/miniconda3/envs/ChIP/bin"
```

Then run:

```bash
bash pipeline/02_map_ecoli.sh <analysis_path> <sample_name> yes
```

For batch processing, one simple example is:

```bash
conda activate /mnt/sda/Public/Environment/miniconda3/envs/ChIP

outdir=/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/analysis
pipe=/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/pipeline/02_map_ecoli.sh

samples=$(ls $outdir/2.cleandata/*_clean.R1.fq.gz | sed 's/_clean.R1\.fq\.gz$//' | xargs -n1 basename | sort -u)

for sample in $samples; do
  echo "=== Processing: $sample ==="
  sh "$pipe" "$outdir" "$sample" yes
done
```

If needed, you can then continue with downstream spike-in normalization scripts such as:

```bash
bash pipeline/calc_ecoli_ratio.sh
bash pipeline/make_spike_bw.sh
```

</details>

<details>
<summary><strong>Option B. Drosophila melanogaster (dm6) spike-in</strong></summary>

If you are using the prebuilt `dm6` reference on our lab server, you can run the spike-in mapping directly.

If you are running this workflow on another server and downloaded the reference by yourself, please update the following lines in `02_map_dm6.sh` first:

```bash
ref="/mnt/sda/Public/Database/Drosophila_melanogaster"
chip="/mnt/sda/Public/Environment/miniconda3/envs/ChIP/bin"
```

Then run:

```bash
bash pipeline/02_map_dm6.sh <analysis_path> <sample_name> yes
```

For example:

```bash
bash pipeline/02_map_dm6.sh ./analysis sampleA yes
```

For batch processing, one simple example is:

```bash
conda activate /mnt/sda/Public/Environment/miniconda3/envs/ChIP

outdir=/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/analysis
pipe=/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/pipeline/02_map_dm6.sh

samples=$(ls $outdir/2.cleandata/*_clean.R1.fq.gz | sed 's/_clean.R1\.fq\.gz$//' | xargs -n1 basename | sort -u)

for sample in $samples; do
  echo "=== Processing: $sample ==="
  sh "$pipe" "$outdir" "$sample" yes
done
```

</details>

### 6. Peak calling

Before running peak calling, prepare the following file in the analysis directory:

```text
callpeak.sample.info
```

This file defines the treatment/control pairs used for MACS2 peak calling.

Format:

```text
treatment1    control1    label1
treatment2    control2    label2
```

The corresponding BAM files should already exist under:

```text
analysis/01_qc_map/<sample>/<sample>.final.bam
```

For example, on our lab server, one way to generate this file is:

```bash
python - <<'PY'
rows = [
    ["GZ26005976-WT1-WT1a_combined", "GZ26005978-IgG1-IgG1_combined", "WT1a_vs_IgG1"],
    ["GZ26005976-WT1-WT1b_combined", "GZ26005978-IgG1-IgG1_combined", "WT1b_vs_IgG1"],
    ["GZ26005976-WT1-WT1c_combined", "GZ26005978-IgG1-IgG1_combined", "WT1c_vs_IgG1"],
    ["GZ26005977-OE1-OE1a_combined", "GZ26005978-IgG1-IgG1_combined", "OE1a_vs_IgG1"],
    ["GZ26005977-OE1-OE1b_combined", "GZ26005978-IgG1-IgG1_combined", "OE1b_vs_IgG1"],
    ["GZ26005977-OE1-OE1c_combined", "GZ26005978-IgG1-IgG1_combined", "OE1c_vs_IgG1"],
    ["GZ26005979-H3K27Me3-1-H3K27Me3-1_combined", "GZ26005978-IgG1-IgG1_combined", "H3K27Me3_1_vs_IgG1"],
    ["GZ26005980-WT2-WT2a_combined", "GZ26005982-IgG2-IgG2_combined", "WT2a_vs_IgG2"],
    ["GZ26005980-WT2-WT2b_combined", "GZ26005982-IgG2-IgG2_combined", "WT2b_vs_IgG2"],
    ["GZ26005980-WT2-WT2c_combined", "GZ26005982-IgG2-IgG2_combined", "WT2c_vs_IgG2"],
    ["GZ26005981-OE2-OE2a_combined", "GZ26005982-IgG2-IgG2_combined", "OE2a_vs_IgG2"],
    ["GZ26005981-OE2-OE2b_combined", "GZ26005982-IgG2-IgG2_combined", "OE2b_vs_IgG2"],
    ["GZ26005981-OE2-OE2c_combined", "GZ26005982-IgG2-IgG2_combined", "OE2c_vs_IgG2"],
    ["GZ26005983-H3K27Me3-2-H3K27Me3-2_combined", "GZ26005982-IgG2-IgG2_combined", "H3K27Me3_2_vs_IgG2"],
]
out = "/mnt/sda/Public/Project/collabration/AoLab/20260206Cut/analysis/callpeak.sample.info"
with open(out, "w", encoding="utf-8") as f:
    for r in rows:
        f.write("\t".join(r) + "\n")
print(out)
PY
```

Then run:

```bash
bash pipeline/03_callpeak.sh /mnt/sda/Public/Project/collabration/AoLab/20260206Cut/analysis
```

---

## 📝 Notes

- `run_qc_map_batch.sh` is mainly used to generate the original/raw version for initial inspection.
- `run_qc_map_batch_CPM.sh` is mainly used to generate CPM-normalized bigWig tracks for downstream comparison and visualization.
- Peak calling is usually performed against the corresponding IgG control.
- WT and OE are biological groups and are usually compared after each sample has first been called against IgG.

---

---

## 📁 Repository Structure

```bash
CUT-RUN-pipeline/
├── README.md
├── LICENSE
├── .gitignore
├── scripts/
│   ├── 01_qc_map.sh
│   ├── 02_map_ecoli.sh
│   ├── 03_callpeak.sh
│   ├── 03_callpeak_spikeIN.sh
│   ├── bam2bw.sh
│   ├── stat.map.sh
│   ├── collect_fastq.sh
│   ├── run_qc_map_batch.sh
│   ├── collect_clean_and_bw.sh
│   ├── run_ecoli_batch.sh
│   ├── calc_ecoli_ratio.sh
│   ├── make_spike_bw.sh
│   └── q30_plot.py
├── config/
│   └── project_paths.example.sh
├── docs/
│   └── workflow.md
└── LICENSE
```

---

## 📊 Pipeline Workflow

```text
Raw FASTQ Files
      ↓
collect_fastq.sh
      ↓
analysis/1.data
      ↓
┌─────────────────────────────────────┐
│  Step 1: 01_qc_map.sh               │
│  • Quality Control (fastp)          │
│  • Read Trimming                    │
│  • Genome Alignment (bowtie2)       │
│  • BAM Processing                   │
│  • CPM bigWig Generation            │
└─────────────────────────────────────┘
      ↓
collect_clean_and_bw.sh
      ↓
analysis/2.cleandata + analysis/Bw
      ↓
┌─────────────────────────────────────┐
│  Step 2A: 02_map_ecoli.sh           │
│  • E. coli spike-in mapping         │
└─────────────────────────────────────┘
                OR
┌─────────────────────────────────────┐
│  Step 2B: 02_map_dm6.sh             │
│  • dm6 spike-in mapping             │
└─────────────────────────────────────┘
      ↓
spike-in mapping statistics
      ↓
spike-in normalization
      ↓
make_spike_bw.sh
      ↓
┌─────────────────────────────────────┐
│  Step 3: Peak Calling               │
│  • 03_callpeak.sh                   │
│  • 03_callpeak_spikeIN.sh           │
│  • MACS2 peak detection             │
└─────────────────────────────────────┘
      ↓
Peak files + normalized tracks
```


---

## 🛠️ Requirements

### Software Dependencies

- **bash**
- **fastp** (≥0.20.0)
- **bowtie2** (≥2.4.0)
- **samtools** (≥1.12)
- **picard** (≥2.25.0)
- **bedtools** (≥2.29.0)
- **MACS2** (≥2.2.7)
- **deepTools**
- **bedGraphToBigWig**
- **Python** (≥3.6)
- **bc**
- **conda**

### Recommended Environment

```bash
conda activate /home/bing.pan/software/miniconda3/envs/ChIPseq
```

---

## ⚙️ Configuration

Copy and edit:

```bash
cp config/project_paths.example.sh config/project_paths.sh
```

Example:

```bash
#!/usr/bin/env bash

RAW_SOURCE_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260206Cut/rawdata"
PROJECT_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260206Cut/analysis"
PIPELINE_DIR="/mnt/sda/Public/Project/collabration/AoLab/20260112CHIP/pipeline"
CONDA_ENV="/home/bing.pan/software/miniconda3/envs/ChIPseq"

THREADS=12
BIN_SIZE=50
PAIR_MODE="yes"
```

---

## 📖 Detailed Usage

## Step 0. Collect raw FASTQ files

Copy all FASTQ files into the working directory:

```bash
bash scripts/collect_fastq.sh
```

This collects:

```text
rawdata/*.fastq.gz
→ analysis/1.data/
```

---

## Step 1. QC and genome mapping

### Batch mode

```bash
bash scripts/run_qc_map_batch.sh
```

### Manual mode

```bash
bash scripts/01_qc_map.sh ./rawdata ./output sample_name yes
```

### Output files

```bash
output/01_qc_map/sample_name/
├── sample_name_clean.R1.fq.gz
├── sample_name_clean.R2.fq.gz
├── sample_name.final.bam
├── sample_name.final.bam.bai
├── sample_name.CPM.bw
├── sample_name.fastp.json
├── sample_name.fastp.html
├── sample_name.metrics
└── assessment.sh.o
```

---

## Step 2. Collect clean FASTQ and CPM bigWig

```bash
bash scripts/collect_clean_and_bw.sh
```

This collects:

- clean FASTQ → `analysis/2.cleandata/`
- CPM bigWig → `analysis/Bw/`

---

## Step 3. Spike-in mapping

Spike-in mapping can be performed using either an *E. coli* spike-in genome or a *Drosophila melanogaster* (`dm6`) spike-in genome, depending on the experimental design.

### Option A. *E. coli* spike-in

#### Batch mode

```bash
bash scripts/run_ecoli_batch.sh
```

#### Manual mode

```bash
bash scripts/02_map_ecoli.sh ./output sample_name yes
```

#### Output files

```bash
output/02_map_ecoli/sample_name/
├── sample_name.bam
├── sample_name.final.bam
├── sample_name.final.bam.bai
└── assessment.sh.o
```

---

### Option B. *Drosophila melanogaster* (`dm6`) spike-in

#### Manual mode

```bash
bash scripts/02_map_dm6.sh ./output sample_name yes
```

#### Output files

```bash
output/02_map_dm6/sample_name/
├── sample_name.bam
├── sample_name.final.bam
├── sample_name.final.bam.bai
└── assessment.sh.o
```

---

### Notes

- Use **Option A** if your experiment uses *E. coli* spike-in.
- Use **Option B** if your experiment uses *Drosophila melanogaster* (`dm6`) spike-in.
- At present, a batch wrapper is provided for the *E. coli* workflow.
- The `dm6` workflow can be run manually using `scripts/02_map_dm6.sh`.


---

## Step 4. Calculate spike-in mapping ratio

```bash
bash scripts/calc_ecoli_ratio.sh
```

### Output files

```bash
analysis/
├── ecoli_ratio.tsv
└── ecoli_mapped.tsv
```

Example:

```text
sample    ecoli_total_reads    ecoli_mapped_reads    ecoli_ratio    ecoli_percent
A1        1000000              850000                0.850000       85.000%
A2        1200000              780000                0.650000       65.000%
```

---

## Step 5. Generate spike-in normalized bigWig

```bash
bash scripts/make_spike_bw.sh
```

### Method

- count mapped reads from spike-in BAM
- choose the maximum mapped count as reference
- calculate:

```text
scaleFactor = reference_ecoli_mapped / sample_ecoli_mapped
```

- apply scale factor to the main genome BAM using `bamCoverage`

### Output files

```bash
analysis/
├── spike_scaleFactor.tsv
└── 04_bw_spike/
    ├── sample1.spike.bw
    ├── sample2.spike.bw
    └── ...
```

---

## Step 6. Peak calling

### Standard peak calling

```bash
bash scripts/03_callpeak.sh /path/to/analysis
```

### Spike-in related downstream analysis

```bash
bash scripts/03_callpeak_spikeIN.sh /path/to/analysis
```

### Peak calling config file

Prepare `callpeak.sample.info`:

```text
treatment1    control1    Treatment_vs_Control
treatment2    control2    Treatment2_vs_Control
```

### Output files

```bash
output/03_call_peak/treatment1-VS-control1/
├── treatment1-VS-control1_peaks.narrowPeak
├── treatment1-VS-control1_summits.bed
├── treatment1-VS-control1.bdg
└── treatment1-VS-control1.log
```

---

## 📂 Directory Convention

```bash
analysis/
├── 1.data/
├── 2.cleandata/
├── 01_qc_map/
├── 02_map_ecoli/
├── 03_call_peak/
├── 04_bw_spike/
├── Bw/
├── ecoli_ratio.tsv
├── ecoli_mapped.tsv
└── spike_scaleFactor.tsv
```

---

## 🧾 Input Naming Convention

### Raw FASTQ

```text
sample_R1.fastq.gz
sample_R2.fastq.gz
```

### Clean FASTQ

```text
sample_clean.R1.fq.gz
sample_clean.R2.fq.gz
```

---

## 🔍 Quality Metrics

Monitor these key metrics for quality assessment.

### fastp Report

- Q20 Rate: typically >90%
- Q30 Rate: typically >80%
- GC Content: species-dependent
- Adapter contamination: should be reduced after trimming

### BAM Statistics

- Mapping Rate: ideally >70%
- Unique Mapping Rate: ideally >60%
- Duplicate Rate: project-dependent

### Peak Calling Results

- Number of Peaks: depends on antibody and target
- Peak Width: depends on TF vs histone mark
- Signal Strength: should show clear enrichment over control

---

## 🐛 Troubleshooting

### Problem: Low mapping rate

Possible checks:

- inspect `fastp.html`
- confirm reference genome index is correct
- confirm FASTQ naming is correct
- confirm sample name extraction matches files

### Problem: No peaks called

Possible checks:

- verify BAM files exist
- confirm names in `callpeak.sample.info` match actual BAM/sample names
- adjust MACS2 parameters
- verify treatment/control grouping is correct

### Problem: Missing spike-in BAM files

Possible checks:

- make sure `run_ecoli_batch.sh` completed
- confirm clean FASTQ files exist
- verify spike-in reference index is correct

### Problem: Permission denied

```bash
chmod +x scripts/*.sh
```

### Problem: Out of memory

Suggestions:

- reduce thread count in config
- reduce parallel jobs
- run one sample at a time

---

## 📚 Example Workflow

```bash
#!/usr/bin/env bash

# 1. Prepare config
cp config/project_paths.example.sh config/project_paths.sh

# 2. Edit config/project_paths.sh manually

# 3. Run pipeline
bash scripts/collect_fastq.sh
bash scripts/run_qc_map_batch.sh
bash scripts/collect_clean_and_bw.sh
bash scripts/run_ecoli_batch.sh
bash scripts/calc_ecoli_ratio.sh
bash scripts/make_spike_bw.sh
bash scripts/03_callpeak.sh /path/to/analysis
```

---

## 📖 Documentation

- `docs/workflow.md` — workflow summary
- `config/project_paths.example.sh` — example path configuration

---

## 📝 Citation

If you use this pipeline, please cite the relevant tools and methods:

- CUT&RUN Method
- bowtie2
- MACS2
- samtools
- bedtools
- deepTools

---

## 📄 License

This project is licensed under the MIT License.

---

## 👨‍💻 Author

**eurkaabc**

---

## 🤝 Contributing

Contributions are welcome through pull requests or issue reports.

---

## 🙏 Acknowledgments

This pipeline builds on the CUT&RUN and ChIP-seq analysis ecosystem and the excellent open-source bioinformatics tools used throughout the workflow.

---
加上和Chip-seq的区别
###  优点
- 需要更少的上样量。ChIP需要500K cells, 而CUT&RUN最少可低至5K cells。
- 测序深度要求不高。CUT&RUN 每个样本测3-8M read/sample,而ChIP-Seq则需要大于等于30M read/sample。
- 背景噪音显著减少，因为CUN&RUN是目标片段测序。
- 成本减少。因为抗体的使用减少，建库测序的深度减少。
### 缺点
- 由于钙依赖性MNase反应时间不合适，可能导致DNA过度消化。类似的限制存在于现在的ChIP-seq操作中，酶或超声DNA打断必须优化。
- 并不是所有的蛋白质都适用于该方案。根据感兴趣的蛋白质，可能需要摸索实验条件。
- 可能是染色质复合物太大而无法扩散出去，或者蛋白质-蛋白质相互作用保留了被切割的复合物。在这种情况下，需要消化后才能提取总DNA。

⭐ If you find this pipeline useful, please consider giving it a star.
