# SAMap automated workflow

This is a workflow, built in [Snakemake](https://snakemake.readthedocs.io/en/stable/), to automate the running of [@atarashansky's](https://github.com/atarashansky) [SAMap](https://github.com/atarashansky/SAMap), mapping cell groups between species for arbitrary input data. It will:

 * Split the input transcriptomes and run the BLAST operations required prior to SAMap. This is instead of [map_genes.sh](https://github.com/atarashansky/SAMap/blob/main/map_genes.sh).
 * Generate the initial SAMAP object.
 * Produce the output mapping table and heatmap relating input cell types.

## Installation

### Prerequisites

The only pre-requisite is Conda (see [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/) for starting instructions), and a Conda environment with Snakemake installed. All other software dependencies will be handled by Snakemake. To create that environment:

```
conda create -n snakemake snakemake
```

(substitute [mamba](https://github.com/mamba-org/mamba) for the Conda command above if you have it, which I recommend)

#### Cluster cofiguration

If you have never used Snakemake before and you have access to a cluster, you will also want to set things up so that Snakemake can exploit those resources. This can be done in the Snakemake command on every run, but it's much easier to use 'profiles', which you can find [here](https://github.com/Snakemake-Profiles) for a variety of cluster types. 


## Download workflow 

Download this repository like:

```
cd /path/to/install
git clone git@github.com:ebi-gene-expression-group/samap-workflow.git
```

Where /path/to/install is where you like to install your software.

### PATH variable

The workflow has couple of scripts providing CLI access to SAMAP. Add them to your PATH like:

```
export PATH=/path/to/install/samap-workflow/workflow/scripts:$PATH
```

# Create analysis directory

```
mkdir -p /path/to/my/analsyis
```

## Generate a config.yaml

The workflow operates from a config.yaml which looks like this:

```
blast:
    splits: 1000
    db_exts: [ 'nhr', 'nin', 'nsq', 'nhr', 'nin' ]
    type: nucl
    threads: 16

data:
    hu:
        anndata: hu.h5ad
        transcriptome: Homo_sapiens.GRCh38.cdna.all.99.fa.gz
    mu:
        anndata: mu.h5ad
        transcriptome: Mus_musculus.GRCm38.cdna.all.99.fa.gz

cell_type_field: 'inferred_cell_type_-_ontology_labels'
outdir: 'out'
```

Edit your own config.yaml based on the above, and store it in your analysis directory.

The 'blast' section configures how blast will be run to generate the homology mappings used by SAMap. If you have access to a cluster, setting splits to a high number as in the above example will save you considerable time, with the BLAST operations spread over multiple nodes. 

The key section is 'data'. Create your own species prefixes in place of 'hu' and 'mu' above, and for each set a transcriptome and an input pair of anndata objects.

The 'cell_type_field' tells SAMap which of the columns in .obs from your input objects should be used to define cell types. 'outdir' just specifies where the results go.

## Running

With the above config done, you can execute the workflow:

```
snakemake -s /path/to/install/samap-workflow/workflow/Snakefile
```

Where the path is as described above.

If you want to use a cluster configuration profile as described above the command is:

```
snakemake -s /path/to/install/samap-workflow/workflow/Snakefile --profile lsf
```

(in this example for an LSF cluster).


## Output

Outputs are produced at the location specified in the configuration, and are currently:

```
results
├── hu_mu.celltype_map_heatmap.png
├── hu_mu.celltype_map.tsv
├── hu_mu.run.sam.pkl
├── hu_mu.sam.pkl
├── hu.t2gene
├── maps
│   └── humu
│       ├── hu_to_mu.txt
│       └── mu_to_hu.txt

```

(where 'hu' and 'mu' are replaced by your own species prefixes). BLAST maps are stored under 'maps', the primary cell type mappings and associated heatmap graphic are stored at the top level

You will see some examples under 'example_outputs', for example the heatmap:

![SAMap heatmap](results/example_outputs/hu_mu.celltype_map_heatmap.png)
