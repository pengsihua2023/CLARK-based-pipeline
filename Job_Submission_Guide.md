# CLARK Virus Classification Job Submission Guide

## Quick Submission for Long-read Data Classification

### Method 1: Using Quick Submission Script (Recommended)

Execute the following commands on the cluster:

```bash
# 1. Navigate to script directory
cd /path/to/CLARK  # Replace with your actual path

# 2. Run submission script
bash run_long_read_classification.sh
```

The script will automatically:
- Set all necessary environment variables
- Check if input file exists
- Create output directory
- Submit job to SLURM

### Method 2: Manual Submission

If you want to manually set parameters, you can use:

```bash
# Set environment variables
export INPUT_FILE="/scratch/sp96859/Meta-genome-data-analysis/Apptainer/data/long_reads/llnl_66d1047e.fastq.gz"
export OUTPUT_PREFIX="llnl_66d1047e_virus"
export OUTPUT_DIR="/scratch/sp96859/Meta-genome-data-analysis/Apptainer/CLARK/run_CLARK/results_long"
export K_MER=20
export MODE=0

# Submit job
sbatch slurm_virus_classification.sh
```

## Job Configuration Information

- **Job Name**: CLARK_Virus_Classification
- **Partition**: bahl_p
- **CPU Cores**: 32
- **Memory**: 128GB
- **Time Limit**: 72 hours
- **Input File**: llnl_66d1047e.fastq.gz
- **Output Directory**: results_long
- **k-mer Size**: 20

## Post-Submission Operations

### Check Job Status
```bash
squeue -u $USER
```

### View Job Details
```bash
squeue -j <job_id> -l
```

### Cancel Job
```bash
scancel <job_id>
```

### View Output Logs

After job completion, view output and error logs:
```bash
# View standard output
cat CLARK_Virus_Classification_<job_id>.out

# View error log
cat CLARK_Virus_Classification_<job_id>.err
```

## Result File Location

After classification is complete, result files will be saved at:
```
/scratch/sp96859/Meta-genome-data-analysis/Apptainer/CLARK/run_CLARK/results_long/llnl_66d1047e_virus.csv
```

## Notes

1. Ensure input file path is correct
2. Ensure CLARK database is built (at `/scratch/sp96859/Meta-genome-data-analysis/Apptainer/databases/CLARK_db/viruses`)
3. Ensure sufficient storage space for results
4. Jobs may take a long time, please be patient

