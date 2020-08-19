# QIIME2 is an open-source bioinformatics pipeline. Please, refer to the "Natively installing QIIME2 (https://docs.qiime2.org/2020.6/install/native/)" to install and run the QIIME2 software.


# Typical install time on a normal desktop computer is about 30 minutes.


# Expected run time for demo on a "normal" desktop computer is approximately 6 hours.


# metafile, manifest file
manifest_demo_dataset.txt
metadata_demo_dataset.txt


# Activating QIIME2
conda activate qiime2-2020.6


# Importing data
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path DEMO_dataset/manifest_demo_dataset.txt --output-path khs/fmt/DEMO_dataset/demo_dataset-paired-end-demux.qza --input-format PairedEndFastqManifestPhred33V2
[SampleData-PairedEndSequencesWithQuality, input format-PairedEndFastqManifestPhred33V2]


#Demultiplexing sequences
DEMO_dataset/
qiime demux summarize --i-data demo_dataset-paired-end-demux.qza --o-visualization demo_dataset-paired-end-demux-summary.qzv


# QIIME2 view
file: demo_dataset-paired-end-denux-summary.qzv
Interactive Quality Plot --> sequence base


# Sequence quality control and feature table construction
qiime dada2 denoise-paired --i-demultiplexed-seqs demo_dataset-paired-end-demux.qza --p-trim-left-f 31 --p-trim-left-r 11 --p-trunc-len-f 288 --p-trunc-len-r 227 --o-table demo_dataset-table-31f288-11r227.qza --o-representative-sequences demo_dataset-rep-seqs-31f288-11r227.qza --o-denoising-stats demo_dataset-denoising-stats-31f288-11r227.qza --p-n-threads 127


# Convert qza to qzv
qiime metadata tabulate --m-input-file demo_dataset-denoising-stats-31f288-11r227.qza --o-visualization demo_dataset-denoising-stats-31f288-11r227.qzv


# FeatureTable and FeatureData summaries
qiime feature-table summarize --i-table demo_dataset-table-31f288-11r227.qza --o-visualization demo_dataset-table-31f288-11r227.qzv --m-sample-metadata-file metadata_demo_dataset.txt


# Amplicon denoising
qiime feature-table filter-features --i-table demo_dataset-table-31f288-11r227.qza --p-min-frequency 10 --p-min-samples 2 --o-filtered-table demo_dataset-table-31f288-11r227-minfreq10-minsamples2.qza

qiime feature-table summarize --i-table demo_dataset-table-31f288-11r227-minfreq10-minsamples2.qza --o-visualization demo_dataset-table-31f288-11r227-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_demo_dataset.txt


# Generate a tree for phylogenetic diversity analyses 
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences demo_dataset-rep-seqs-31f288-11r227.qza --o-alignment demo_dataset-aligned-rep-seqs-31f288-11r227.qza --o-masked-alignment demo_dataset-masked-aligned-rep-seqs-31f288-11r227.qza --o-tree demo_dataset-unrooted-tree-31f288-11r227.qza --o-rooted-tree demo_dataset-rooted-tree-31f288-11r227.qza --p-n-threads 127
[pipeline: Build a phylogenetic tree using fasttree and mafft alignment]


# Rarefy table
qiime feature-table rarefy --i-table demo_dataset-table-31f288-11r227-minfreq10-minsamples2.qza --p-sampling-depth 248600 --o-rarefied-table demo_dataset-rarefied-table-31f288-11r227-minfreq10-minsamples2.qza


# Generate visual and tabular summaries of a feature table
qiime feature-table summarize --i-table demo_dataset-rarefied-table-31f288-11r227-minfreq10-minsamples2.qza --o-visualization demo_dataset-rarefied-table-31f288-11r227-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_demo_dataset.txt


# Applies a collection of diversity metrics  to a feature table.
qiime diversity core-metrics-phylogenetic --i-phylogeny demo_dataset-rooted-tree-31f288-11r227.qza --i-table demo_dataset-rarefied-table-31f288-11r227-minfreq10-minsamples2.qza --p-sampling-depth 248600 --m-metadata-file metadata_demo_dataset.txt --output-dir demo_dataset-core-metrics-results/ --p-n-jobs-or-threads 127


# taxonomic classification (full length classifier usage)
qiime feature-classifier classify-sklearn --i-reads demo_dataset-rep-seqs-31f288-11r227.qza --i-classifier gg-13-8-99-nb-classifier.qza --o-classification demo_dataset-taxonomy-31f288-11r227.qza --p-n-jobs 127


# Visualize taxonomy with an interactive bar plot
qiime taxa barplot --i-table demo_dataset-rarefied-table-31f288-11r227-minfreq10-minsamples2.qza --i-taxonomy demo_dataset-taxonomy-31f288-11r227.qza --m-metadata-file metadata_demo_dataset.txt --o-visualization demo_dataset-taxonomy-barcharts-31f288-11r227.qza

