# CLARK SLURM Submission Scripts User Guide

This directory contains submission scripts for running CLARK virus detection and classification on SLURM clusters.

## Default Configuration

All scripts are pre-configured with the following default paths:
- **CLARK Database Directory**: `/scratch/sp96859/Meta-genome-data-analysis/Apptainer/databases/CLARK_db`
- **CLARK Software Directory**: `/scratch/sp96859/Meta-genome-data-analysis/Apptainer/CLARK/CLARK`

If your paths are different, you can override the default values by setting environment variables:
```bash
export CLARK_DB_DIR=/your/custom/db/path
export CLARK_DIR=/your/custom/clark/path
```

## Script List

### 1. `slurm_build_virus_db.sh` - Build Virus Database

**Purpose**: Download and build CLARK virus database on cluster

**Usage**:
```bash
# Direct submission (database and software directories are already default set)
sbatch slurm_build_virus_db.sh

# If you need to use different directories, you can set environment variables to override defaults:
# export CLARK_DB_DIR=/your/custom/db/path
# export CLARK_DIR=/your/custom/clark/path
```

**Resource Requirements**:
- CPU: 8 cores
- Memory: 64GB
- Time: 24 hours

### 2. `slurm_virus_classification.sh` - Single-end Sequencing Virus Classification

**Purpose**: Virus classification for single-end sequencing data

**Usage**:
```bash
# Set input/output parameters (database and software directories are already default set)
export INPUT_FILE=/path/to/sample.fastq
export OUTPUT_PREFIX=virus_results
export OUTPUT_DIR=/path/to/results
export K_MER=20  # Recommended 20-21 for virus detection
export MODE=0    # 0=full, 1=default, 2=express

# Submit job
sbatch slurm_virus_classification.sh
```

**Resource Requirements**:
- CPU: 32 cores
- Memory: 128GB
- Time: 72 hours

### 3. `slurm_virus_classification_paired.sh` - Paired-end Sequencing Virus Classification

**Purpose**: Virus classification for paired-end sequencing data

**Usage**:
```bash
# Set input/output parameters (database and software directories are already default set)
export INPUT_R1=/path/to/sample_R1.fastq
export INPUT_R2=/path/to/sample_R2.fastq
export OUTPUT_PREFIX=virus_results
export OUTPUT_DIR=/path/to/results
export K_MER=20
export MODE=0

# Submit job
sbatch slurm_virus_classification_paired.sh
```

**Resource Requirements**:
- CPU: 32 cores
- Memory: 128GB
- Time: 72 hours

### 4. `slurm_estimate_abundance.sh` - Abundance Estimation

**Purpose**: Abundance estimation for CLARK classification results

**Usage**:
```bash
# Set input/output parameters (database and software directories are already default set)
export INPUT_CSV=/path/to/results/virus_results.csv
export OUTPUT_DIR=/path/to/results

# Optional parameters
export CONFIDENCE_THRESHOLD=0.80  # Confidence threshold
export GAMMA_THRESHOLD=0.03       # Gamma threshold
export ABUNDANCE_THRESHOLD=2       # Abundance threshold (%)
export HIGH_CONFIDENCE=true        # Use only high confidence assignments
export MPA_FORMAT=false            # Output MetaPhlAn format

# Submit job
sbatch slurm_estimate_abundance.sh
```

**Resource Requirements**:
- CPU: 8 cores
- Memory: 32GB
- Time: 4 hours

### 5. `slurm_virus_pipeline.sh` - Complete Virus Detection Pipeline

**Purpose**: One-stop virus classification and abundance estimation

**Usage**:
```bash
# Single-end sequencing (database and software directories are already default set)
export INPUT_FILE=/path/to/sample.fastq
export OUTPUT_PREFIX=virus_results
export OUTPUT_DIR=/path/to/results
export K_MER=20
export MODE=0
export ESTIMATE_ABUNDANCE=true
export GAMMA_THRESHOLD=0.03

sbatch slurm_virus_pipeline.sh

# Paired-end sequencing
export INPUT_R2=/path/to/sample_R2.fastq
sbatch slurm_virus_pipeline.sh
```

**Resource Requirements**:
- CPU: 32 cores
- Memory: 128GB
- Time: 72 hours

### 6. `slurm_batch_virus_classification.sh` - Batch Processing

**Purpose**: Batch processing of multiple samples

**Usage**:
```bash
# Set input/output parameters (database and software directories are already default set)
export INPUT_DIR=/path/to/input_samples
export OUTPUT_DIR=/path/to/results
export K_MER=20
export MODE=0
export ESTIMATE_ABUNDANCE=true

# Submit job
sbatch slurm_batch_virus_classification.sh
```

**Description**:
- Script automatically detects single-end or paired-end files
- Single-end files: Finds `*.fastq`, `*.fq`, `*.fa`, `*.fasta`
- Paired-end files: Finds `*_R1*` and corresponding `*_R2*` files

**Resource Requirements**:
- CPU: 32 cores
- Memory: 128GB
- Time: 72 hours (adjust based on number of samples)

## Parameter Description

### Required Environment Variables

- `CLARK_DB_DIR`: CLARK database directory path (default: `/scratch/sp96859/Meta-genome-data-analysis/Apptainer/databases/CLARK_db`, can be overridden by environment variable)
- `CLARK_DIR`: CLARK software installation directory path (default: `/scratch/sp96859/Meta-genome-data-analysis/Apptainer/CLARK/CLARK`, can be overridden by environment variable)
  - This directory should contain the following script files:
    - `classify_metagenome.sh`
    - `set_targets.sh`
    - `estimate_abundance.sh`
    - `CLARK` (executable file)
    - Other CLARK-related scripts

### Optional Environment Variables

#### Classification Parameters
- `INPUT_FILE`: Input file path (single-end)
- `INPUT_R1`: Input file R1 path (paired-end)
- `INPUT_R2`: Input file R2 path (paired-end)
- `OUTPUT_PREFIX`: Output file prefix
- `OUTPUT_DIR`: Output directory
- `K_MER`: k-mer size (default 31, recommended 20-21 for virus detection)
- `MODE`: Execution mode (0=full, 1=default, 2=express, default 0)

#### Abundance Estimation Parameters
- `CONFIDENCE_THRESHOLD`: Confidence threshold (e.g.: 0.80)
- `GAMMA_THRESHOLD`: Gamma threshold (e.g.: 0.03)
- `ABUNDANCE_THRESHOLD`: Abundance threshold percentage (e.g.: 2)
- `HIGH_CONFIDENCE`: Whether to use only high confidence assignments (true/false)
- `MPA_FORMAT`: Whether to output MetaPhlAn format (true/false)

## Usage Examples

### Example 1: Complete Workflow

```bash
# Step 1: Build database (only need to run once)
# Database and software directories are already default set, just submit directly
sbatch slurm_build_virus_db.sh

# Wait for database build to complete...

# Step 2: Run virus classification and abundance estimation
export INPUT_FILE=/data/samples/sample1.fastq
export OUTPUT_PREFIX=sample1_virus
export OUTPUT_DIR=/data/results
export K_MER=20
sbatch slurm_virus_pipeline.sh
```

### Example 2: Batch Processing Multiple Samples

```bash
# Prepare input directory, put all FASTQ files in it
mkdir -p /data/input_samples
cp *.fastq /data/input_samples/

# Set parameters and submit (database and software directories are already default set)
export INPUT_DIR=/data/input_samples
export OUTPUT_DIR=/data/results
export K_MER=20
sbatch slurm_batch_virus_classification.sh
```

### Example 3: Classification Only, Calculate Abundance Later

```bash
# First, run classification
export INPUT_FILE=/data/sample.fastq
export OUTPUT_PREFIX=sample_virus
sbatch slurm_virus_classification.sh

# Wait for classification to complete...

# Then calculate abundance
export INPUT_CSV=/data/results/sample_virus.csv
export GAMMA_THRESHOLD=0.03
sbatch slurm_estimate_abundance.sh
```

### Example 4: Using Actual Data Files

**Long-read Data (Single-end)**:
```bash
# Set input/output parameters
export INPUT_FILE=/scratch/sp96859/Meta-genome-data-analysis/Apptainer/data/long_reads/llnl_66d1047e.fastq.gz
export OUTPUT_PREFIX=llnl_66d1047e_virus
export OUTPUT_DIR=/scratch/sp96859/Meta-genome-data-analysis/Apptainer/results/long_reads
export K_MER=20
export MODE=0
export ESTIMATE_ABUNDANCE=true
export GAMMA_THRESHOLD=0.03

# Submit job (complete pipeline: classification + abundance estimation)
sbatch slurm_virus_pipeline.sh
```

**Short-read Data (Paired-end)**:
```bash
# Set input/output parameters
export INPUT_FILE=/scratch/sp96859/Meta-genome-data-analysis/Apptainer/data/short_reads/llnl_66ce4dde_R1.fastq.gz
export INPUT_R2=/scratch/sp96859/Meta-genome-data-analysis/Apptainer/data/short_reads/llnl_66ce4dde_R2.fastq.gz
export OUTPUT_PREFIX=llnl_66ce4dde_virus
export OUTPUT_DIR=/scratch/sp96859/Meta-genome-data-analysis/Apptainer/results/short_reads
export K_MER=20
export MODE=0
export ESTIMATE_ABUNDANCE=true
export GAMMA_THRESHOLD=0.03

# Submit job (complete pipeline: classification + abundance estimation)
sbatch slurm_virus_pipeline.sh
```

**Tip**: You can also check the `submit_my_data_example.sh` file for more configuration examples.

## Resource Adjustment Recommendations

### Adjust Resources Based on Data Size

**Small Dataset** (< 10GB):
- CPU: 16 cores
- Memory: 64GB
- Time: 24 hours

**Medium Dataset** (10-50GB):
- CPU: 32 cores
- Memory: 128GB
- Time: 72 hours

**Large Dataset** (> 50GB):
- CPU: 64 cores
- Memory: 256GB
- Time: 120 hours

### Adjust Memory Based on k-mer Size

- k=20-21: Relatively low memory requirements
- k=31: Default, balanced memory and performance
- k>31: Increased memory requirements

## Output File Description

### Classification Results
- `<OUTPUT_PREFIX>.csv`: Main classification result file

### Abundance Estimation Results
- `<OUTPUT_PREFIX>_abundance.csv`: Abundance estimation results
- If using `--mpa` option, MetaPhlAn format file will be generated

## Troubleshooting

### 1. Database Does Not Exist Error

**Problem**: `Error: Virus database does not exist`

**Solution**: First run `slurm_build_virus_db.sh` to build the database

### 2. Out of Memory

**Problem**: Job was killed (OOM)

**Solution**: Increase `--mem` parameter, or use smaller k-mer value (k=20)

### 3. Timeout

**Problem**: Job timed out

**Solution**: Increase `--time` parameter, or use express mode (MODE=2)

### 4. Cannot Find CLARK Scripts

**Problem**: `Error: Cannot find classify_metagenome.sh script`

**Solution**: 
- Ensure `CLARK_DIR` environment variable is set correctly
- `CLARK_DIR` should point to CLARK installation directory containing `classify_metagenome.sh`, `set_targets.sh`, etc.
- Check if these script files exist in that directory: `ls $CLARK_DIR/classify_metagenome.sh`

## Notes

1. **Database Build**: Virus database only needs to be built once and can be reused
2. **k-mer Selection**: For virus detection, recommended to use k=20 or 21 for higher sensitivity
3. **Memory Requirements**: CLARK memory requirements depend on k-mer size and database size
4. **Parallel Processing**: For multiple samples, you can use batch script or submit multiple jobs separately
5. **Result Checking**: After job completion, check `.out` and `.err` files to confirm success

## Contact and Support

For questions, please refer to:
- CLARK Official Documentation: http://clark.cs.ucr.edu
- CLARK GitHub: https://github.com/rouni001/CLARK
- CLARK User Community: https://groups.google.com/g/clarkusers
