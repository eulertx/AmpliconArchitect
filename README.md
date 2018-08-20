*AmpliconArchitect*
Focal oncogene amplification and rearrangements drive tumor growth and evolution in multiple cancer types. Proposed mechanisms for focal amplification include extrachromosomal DNA (ecDNA) formation, breakage-fusion-bridge (BFB) mechanism, tandem duplications, chromothripsis and others. Focally amplified regions are often hotspots for genomic rearrangements. As a result, the focally amplified region my undergo rapid copy number changes and the structure of the focally amplified region may evolve over time contributing to tumor progression. ecDNA originating from distinct genomic regions may recombine to form larger ecDNA elements bringing together multiple oncogenes for simultaneous amplification. Furthermore, ecDNA elements may reintegrate back into the genome to form HSRs. The interchangeability between ecDNA and HSR may allow the tumor adapt to changing environment, e.g. targetted drug application. As a result, understanding the architecture of the focal amplifications is important to gain insights into tumor progression as well as response to treatment. AmpliconArchitect is a tool which can reconstruct the structure of focally amplified regions using whole genome sequence data of a cancer sample.




#=============================================================================================================================================================================================

# Installation:

# AA download (if you have not already cloned this source code):
git clone https://github.com/virajbdeshpande/AmpliconArchitect.git

# Dependencies:
## 1) Python 2.7

## 2) Ubuntu libraries and tools:
sudo apt-get install build-essential python-dev gfortran python-numpy python-scipy python-matplotlib python-pip zlib1g-dev samtools 

## 3) Pysam verion 0.9.0 or higher (https://github.com/pysam-developers/pysam):
sudo pip install pysam

## 4) Mosek optimization tool (https://www.mosek.com/):
wget http://download.mosek.com/stable/8.0.0.60/mosektoolslinux64x86.tar.bz2
tar xf mosektoolslinux64x86.tar.bz2
echo Please obtain license from https://mosek.com/resources/academic-license or https://mosek.com/resources/trial-license and place in $PWD/mosek/8/licenses
echo export MOSEKPLATFORM=linux64x86 >> ~/.bashrc
export MOSEKPLATFORM=linux64x86
echo export PATH=$PATH:$PWD/mosek/8/tools/platform/$MOSEKPLATFORM/bin >> ~/.bashrc
echo export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PWD/mosek/8/tools/platform/$MOSEKPLATFORM/bin >> ~/.bashrc
echo export MOSEKLM_LICENSE_FILE=$PWD/mosek/8/licenses >> ~/.bashrc
cd $PWD/mosek/8/tools/platform/linux64x86/python/2/
sudo python setup.py install #(--user)
cd -
source ~/.bashrc


# Data repositories:
## Download the data repositories. While we include some annotations, we are unable to host some large files in the git repository.
## These may be downloaded from https://drive.google.com/open?id=0ByYcg0axX7udUDRxcTdZZkg0X1k. Thanks to Peter Ulz for noticing incorrect link earlier.
tar zxf data_repo.tar.gz
echo export AA_DATA_REPO=$PWD/data_repo >> ~/.bashrc
source ~/.bashrc
## If you are using only a specific reference, you may delete the folder for the other reference under data_repo to save space.

#==============================================================================================================================================================================================


#==============================================================================================================================================================================================

# AmpliconArchitect reconstruction:

# Preprocessing:

## Input 1: Coordinate sorted bam file:
## Use bwa mem or any other software to align reads to the hg19full.fa in the data_repo human reference to output a bam file.
## Sort the bam file by genomic coordinates and index the coordinate sorted bam file using "samtools index".
## May downsample bamfile using src/downsample.py or in real time when running AmpliconArchitect.py

## Input 2: Amplicon Interval(s):
## Identify interval(s) of interest by using a CNV caller e.g. ReadDepth, but may use others or user defined intervals.
## Recommended to use file AmpliconArchitect/src/read_depth_params for calling variants with ReadDepth.
## Recommended only analyze CN gains > 5x copy count and > 100kbp in size.
## Recommended to filter out intervals with significant overlap with conserved regions.
## Input intervals in bed file format
python src/amplified_intervals.py --bed {read_depth_folder}/output/alts.dat > {outName}.bed

# Amplicon reconstruction
python src/AmpliconArchitect.py --bam <input_bam> --bed <bed file> --out <prefix for output files> --ref hg19/GRCh37
# See below python src/AmpliconArchitect.py --help for other options.


# Output description
## The software generates 4 types of output files
## 1) <outName>_summary.txt:
## This file includes a summary for all amplicons detected
## 
## 2) <outName>_amplicon<id>_graph.txt:
## For each amplicon, breakpoint graph describing sequence edges and breakpoint edges in the graph
## Edge each consists of 2 vertices in the format <chromosome>:<position><orientation>
## e.g. A sequence edge chr1:10001-->chr1:20000+ represents the 10000bp genomic sequence. A breakpoint edge chr1:10001-->chr1:20000+ implies that on chr1, the forward strand from position 20000 is continues onto the forward strand at position 10001 in the 5' to 3' direction to create a 10000bp loop. Breakpoint vertices with position -1 are reserved for source vertex at the end of linear contig.
## 
## 3) <outName>_amplicon<id>_cycle.txt:
## For each amplicon, this file represents the list of structures contained in the amplicon.
## Segments represent genomic intervals (with possible overlaps) and each segment is assigned a unique id.
## Segment id 0 is reserved for source vertices. 
## Cycles are represented as ordered lists of connected segments along with orientation of the segments.
## The last segment connects back to the first segment of the cycle.
## Linear contigs have 0+ and 0- as their first and last segments.
## May be uploaded to web interface for visualization and operations on cycles (See below).
##
## 4) <outName>_amplicon<id>.png:
## A single image file displaying the set of intervals, underlying coverage, segmentation of coverage, discordant read pairs and oncogene annotations.
## May be uploaded to web interface for visualization and operations on cycles (See below).

# Web interface
## URL: genomequery.ucsd.edu:8800
## Choose and upload a cycles files generated by AA.
## Optional but recommended: Upload corresponding png file generated by AA.
## Operations:
## a) Show-hide cyles by name, copy count threshold
## b) Merge cycles with a common segment: Select cycle names and segments rank (NOT the ID displayed) in the order of segments displayed.
##    For closed cycles, first segment rank is 0; for open walks, For open walks, first segment rank is 1 and so on.
## c) Pivot cycle around inverted duplication: Pivot the portion of the cycle that connects two reversed occurences of a duplicated segment without changing any genomic connections.
## d) Undo last edit: Go back to previous state.


# Commandline options
AmpliconArchitect.py --help
## usage: AmpliconArchitect.py [-h] [--bed FILE] [--bam FILE] [--out FILE]
##                             [--cbam FILE] [--cbed FILE] [--extendmode STR]
##                             [--sensitivems STR] [--ref STR]
##                             [--downsample FLOAT]
## 
## Reconstruct Amplicons connected to listed intervals.
## 
## optional arguments:
##   -h, --help          show this help message and exit
##   --bed FILE          Bed file with putative list of amplified intervals
##   --bam FILE          Coordinate sorted BAM file with index mapped to hg19
##                       reference sequence
##   --out FILE          Prefix for output files
##   --cbam FILE         Optional bamfile to use for coverage calculation
##   --cbed FILE         Optional bedfile defining 1000 10kbp genomic windows for
##                       coverage calcualtion
##   --extendmode STR    Values: [EXPLORE/CLUSTERED/UNCLUSTERED]. This determines
##                       how the input intervals in bed file are treated. EXPLORE
##                       : Search for all connected intervals in genome that may
##                       be connected to input intervals. CLUSTERED : Input
##                       intervals are treated as part of a single connected
##                       amplicon and no new connected intervals are added.
##                       UNCLUSTERED : Input intervals are treated as part of a
##                       single connected amplicon and no new connected intervals
##                       are added.
##   --sensitivems STR   Values: [True, False]. Set "True" only if expected copy
##                       counts to vary by orders of magnitude, .e.g viral
##                       integration. Default: False
##   --ref STR           Values: [hg19, GRCh37, None]. "hg19"(default) : chr1, ..
##                       chrM etc / "GRCh37" : '1', '2', .. 'MT' etc/ "None" : Do
##                       not use any annotations. AA can tolerate additional
##                       chromosomes not stated but accuracy and annotations may
##                       be affected. Default: hg19
##   --downsample FLOAT  Values: [-1, 0, C(>0)]. Decide how to downsample the
##                       bamfile during reconstruction. Reads are automatically
##                       downsampled in real time for speedup. Alternatively pre-
##                       process bam file using $AA_SRC/downsample.py. -1 : Do
##                       not downsample bam file, use full coverage. 0 (default):
##                       Downsample bamfile to 10X coverage if original coverage
##                       larger then 10. C (>0) : Downsample bam file to coverage
##                       C if original coverage larger than C

# Visualizing reconstruction:

## Installing dependencies:
## 3) flask:
sudo pip install Flask

## Running the visualization tool locally on port 8000:
export FLASK_APP=episome/web_app.py
flask run --host=0.0.0.0 --port=8000

#==============================================================================================================================================================================================


