# microbiome-profiler
This repository contains the scripts necessary to characterize human gut microbiome compositions based on selection criteria by using the Bio.jl package, BLAST+ software package, and the Plotly.jl package.   

## Getting Started
### Generation of Data Sets
Data sets were generated from the [Human Microbiome Project Data Portal](https://portal.hmpdacc.org/) using the following selection criteria:
1. Body Site
   - feces
2. Format
   - FASTQ
3. Type
   - 16s_raw_seq_set
4. Subject Race
   - african_american
   - asian
   - caucasian
   - hispanic_or_latino

To download large data sets, follow this tutorial [here](https://www.youtube.com/watch?v=hbSUBr8yWNY). The [hmp_client](https://github.com/ihmpdcc/hmp_client) can be used to download the specified data files in the manifest file.
#### Limitations
The **hmp_client** has limited options for retrieving data files listed in the manifest file. If the options specified [here](https://github.com/ihmpdcc/hmp_client) do not work for you, **get_files.py** can be run to download the necessary data files by parsing the manifest file.
#### Example
Specify the path to the folder containing your manifest file(s).
```{Python}
directory = 'C:\Users\MyName\Desktop\hmp_data'
```
Run **get_files.py**.
```{Python}
python get_files.py
```

### Setting up Bio.jl 
```{Julia}
Pkg.add("Bio")
```

### Installing BLAST+
BLAST+ software can be used to run BLAST sequence alignment searches locally. The instructions to download BLAST+ can be found [here](https://www.ncbi.nlm.nih.gov/books/NBK279671/).

##### Downloading BLAST database
As our data type is 16S raw sequence data, we used the 16SMicrobial data set. Information on how to download BLAST+ data sets can be found [here](https://www.ncbi.nlm.nih.gov/books/NBK52637/) in the **Test BLAST database** subsection.

### Setting up Plotly.jl
```{Julia}
Pkg.clone("https://github.com/plotly/Plotly.jl")
```
Create a Plotly user account following the instructions [here](https://plot.ly/julia/getting-started/#authentication).
Generate and save the username, password, and API key associated with the Plotly user account. 

## Parsing of FASTQ Files
Using **fastq_parser.jl**, specify the directory path that contains all of the FASTQ files. 

### Example
```{Julia}
init_path = "C:/Users/MyName/Desktop/hmp_fastq_files/"
```
**Note**: For proper execution of **fastq_parser.jl**, ensure a forward slash follows the directory name.

### Run fastq_parser.jl.
```{Julia}
julia fastq_parser.jl
```
## Using BLAST to Create Microbiome Profile
Using **microbiome_blast.jl**, specify the directory path that contains the **fasta_output.txt** generated by **fastq_parser.jl**.

### Example
```{Julia}
sequences = open("C:/Users/MyName/Desktop/fasta_output.txt/", "r")
```
Ensure the parameters passed into the **blastn()** method are to your specifications. 

### Example
```{Julia}
profile = blastn("C:\\Users\\MyName\\Desktop\\query_sequence.txt", "C:\\Users\\MyName\\Desktop\\blastdb\\16SMicrobial", db=true, ["-perc_identity", 98])
```
In this case, we've added a parameter that returns results of a percent identity of 98% or higher. Additional options that can be used in your BLAST search can be found [here](https://biojulia.net/Bio.jl/stable/man/tools/#BLAST-wrapper-1).

### Run microbiome_blast.jl.
```{Julia}
julia microbiome_blast.jl
```

### Data Extraction & Visualization
Run the following Julia script:
```{Julia}
julia extraction_visulaization.jl
```
This script reads in a Text File containing the BLAST results for a given population, parses a specific set of data, restructures it, and sends requests to the Plotly API to visualize the restructured data.

The script performs 5 main functions:
1) For a given population (data stored in the Text File), determine the microbial species present within each sample.
```{Julia}
species_sample = clean(extract(open(data_file)))
```
2) For this population, produce a dictionary representing the frequency count distribution of all microbial genera present.
```{Julia}
genus_FC_sample = count(genusify(species_sample))
```
3) For this population, request the Plotly API to produce and store a bar chart displaying this frequency count distribution.
```{Julia}
createBar(genus_FC, bargraph_name)
```
4) For this population, determine the species-level percentage composition of the three most prominent microbial genera present.
```{Julia}
prominent_genera_pairs = determineOutliers(genus_FC_sample, "Prominent Genera:")
    for prominent_genus_pair in prominent_genera_pairs
        determineComposition(population_name, prominent_genus_pair, species_sample)
    end
```
5) For this poulation, request the Plotly API to produce and store pie charts diplaying each genera's species-level composition.
```{Julia}
createPie(genus_breakdown, population_name * " " * prominent_genus_name * " Composition")
```
All plots (bie charts and pie charts) are publicly viewable at [Plotly](https://plot.ly/organize/home).
