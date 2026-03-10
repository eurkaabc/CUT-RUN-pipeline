# CUT&RUN Pipeline 🧬

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Language: Bash](https://img.shields.io/badge/Language-Bash-green.svg)](https://www.gnu.org/software/bash/)
[![Python: 3.6+](https://img.shields.io/badge/Python-3.6+-blue.svg)](https://www.python.org/)

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

<p align="center">
  <img src="https://github.com/user-attachments/assets/7b39a0a1-51df-4588-9cdb-429555fd6ef6" alt="CUT&RUN workflow" width="420">
</p>




---

## ✨ Pipeline Features

| Feature | Description |
|---------|-------------|
| 🔬 **Quality Control** | Read quality assessment and adapter trimming using `fastp` |
| 🗺️ **Mapping** | Read alignment using `bowtie2` |
| 📦 **BAM Processing** | Sorting, duplicate marking, and filtering |
| 🎯 **Peak Calling** | Statistical peak identification using `MACS2` |
| 📊 **Normalization** | Spike-in calibration using *E. coli* |
| 📈 **Visualization** | BigWig generation for genome browser display |
| 📋 **Statistics** | Mapping and quality metrics summarization |
| 🔁 **Batch Processing** | Convenient wrapper scripts for running all samples |

---

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/eurkaabc/CUT-RUN-pipeline.git
cd CUT-RUN-pipeline
chmod +x pipeline/*.sh
```

### Prepare environment

For example, on our lab server, the pipeline is usually run in the following conda environment:

```bash
conda activate /mnt/sda/Public/Environment/miniconda3/envs/ChIP
```

If the environment has not been created yet on a new server, it can be recreated with:

```bash
conda env create -f environment.yml
conda activate ChIP
```

### Prepare config

Before running the pipeline, create `pipeline/project_paths.sh` and edit it according to your local environment.

Only this file needs to be modified before running the workflow.  
You do **not** need to edit each pipeline script individually.

Example:

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

### Run the workflow

```bash
# 0. Collect raw FASTQ
bash pipeline/collect_fastq.sh

# 1. QC and genome mapping
bash pipeline/run_qc_map_batch.sh

# 2. Collect clean FASTQ and CPM bigWig
bash pipeline/collect_clean_and_bw.sh
```


### Spike-in mapping options

#### Option A. *E. coli* spike-in

```bash
bash pipeline/run_ecoli_batch.sh
bash pipeline/calc_ecoli_ratio.sh
bash pipeline/make_spike_bw.sh
```

#### Option B. *Drosophila melanogaster* (`dm6`) spike-in

```bash
bash pipeline/02_map_dm6.sh /path/to/analysis sample_name yes
```

### Peak calling

```bash
bash pipeline/03_callpeak.sh /path/to/analysis
```

If needed:

```bash
bash pipeline/03_callpeak_spikeIN.sh /path/to/analysis
```



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

⭐ If you find this pipeline useful, please consider giving it a star.
