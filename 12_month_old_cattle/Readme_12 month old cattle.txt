# QIIME2 is an open-source bioinformatics pipeline. Please, refer to the "Natively installing QIIME2 (https://docs.qiime2.org/2020.6/install/native/)" to install and run the QIIME2 software.


# Typical install time on a normal desktop computer is about 30 minutes.


# metafile, manifest file
metadata_12m
manifest_12m


# Activating QIIME2
conda activate qiime2-2020.6


# Importing data
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path 12m/manifest_12m.txt --output-path 12m/12m-paired-end-demux.qza --input-format PairedEndFastqManifestPhred33V2
[SampleData-PairedEndSequencesWithQuality, input format-PairedEndFastqManifestPhred33V2]

12m/

# Demultiplexing sequences
qiime demux summarize --i-data 12m-paired-end-demux.qza --o-visualization 12m-paired-end-demux-summary.qzv


# QIIME2 view
file: 12m-paired-end-denux-summary.qzv
Interactive Quality Plot --> sequence base


# Sequence quality control and feature table construction
qiime dada2 denoise-paired --i-demultiplexed-seqs 12m-paired-end-demux.qza --p-trim-left-f 18 --p-trim-left-r 18 --p-trunc-len-f 297 --p-trunc-len-r 241 --o-table 12m-table-18f297-18r241.qza --o-representative-sequences 12m-rep-seqs-18f297-18r241.qza --o-denoising-stats 12m-denoising-stats-18f297-18r241.qza --p-n-threads 127


# Convert qza to qzv
qiime metadata tabulate --m-input-file 12m-denoising-stats-18f297-18r241.qza --o-visualization 12m-denoising-stats-18f297-18r241.qzv


# FeatureTable and FeatureData summaries
qiime feature-table summarize --i-table 12m-table-18f297-18r241.qza --o-visualization 12m-table-18f297-18r241.qzv --m-sample-metadata-file metadata_12m.txt


# Amplicon denoising
qiime feature-table filter-features --i-table 12m-table-18f297-18r241.qza --p-min-frequency 10 --p-min-samples 2 --o-filtered-table 12m-table-18f297-18r241-minfreq10-minsamples2.qza
qiime feature-table summarize --i-table 12m-table-18f297-18r241-minfreq10-minsamples2.qza --o-visualization 12m-table-18f297-18r241-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_12m.txt


# Generate a tree for phylogenetic diversity analyses 
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences 12m-rep-seqs-18f297-18r241.qza --o-alignment 12m-aligned-rep-seqs-18f297-18r241.qza --o-masked-alignment 12m-masked-aligned-rep-seqs-18f297-18r241.qza --o-tree 12m-unrooted-tree-18f297-18r241.qza --o-rooted-tree 12m-rooted-tree-18f297-18r241.qza --p-n-threads 127
[pipeline: Build a phylogenetic tree using fasttree and mafft alignment]


# Rarefy table
qiime feature-table rarefy --i-table 12m-table-18f297-18r241-minfreq10-minsamples2.qza --p-sampling-depth 20293 --o-rarefied-table 12m-rarefied-table-18f297-18r241-minfreq10-minsamples2.qza


# Generate visual and tabular summaries of a feature table
qiime feature-table summarize --i-table 12m-rarefied-table-18f297-18r241-minfreq10-minsamples2.qza --o-visualization 12m-rarefied-table-18f297-18r241-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_12m.txt


# Applies a collection of diversity metrics  to a feature table.
qiime diversity core-metrics-phylogenetic --i-phylogeny 12m-rooted-tree-18f297-18r241.qza --i-table 12m-rarefied-table-18f297-18r241-minfreq10-minsamples2.qza --p-sampling-depth 20293 --m-metadata-file metadata_12m.txt --output-dir 12m-core-metrics-results/ --p-n-jobs-or-threads 127


# Taxonomic classification (full length classifier usage)
qiime feature-classifier classify-sklearn --i-reads 12m-rep-seqs-18f297-18r241.qza --i-classifier gg-13-8-99-nb-classifier.qza --o-classification 12m-taxonomy-18f297-18r241.qza --p-n-jobs 127


# Visualize taxonomy with an interactive bar plot
qiime taxa barplot --i-table 12m-rarefied-table-18f297-18r241-minfreq10-minsamples2.qza --i-taxonomy 12m-taxonomy-18f297-18r241.qza --m-metadata-file metadata_12m.txt --o-visualization 12m-taxonomy-barcharts-18f297-18r241.qza

# Convert qza to qzv
cd 12m-core-metrics-results/
qiime metadata tabulate --m-input-file rarefied_table.qza --o-visualization rarefied_table.qzv


# Compares categories
qiime diversity beta-group-significance --i-distance-matrix 12m-core-metrics-results/unweighted_unifrac_distance_matrix.qza --m-metadata-file metadata_12m.txt --m-metadata-column group --p-method permanova --p-pairwise --p-permutations 999 --o-visualization 12m_permanova_unweighted_pairwise.qzv
qiime diversity beta-group-significance --i-distance-matrix 12m-core-metrics-results/weighted_unifrac_distance_matrix.qza --m-metadata-file metadata_12m.txt --m-metadata-column group --p-method permanova --p-pairwise --p-permutations 999 --o-visualization 12m_permanova_weighted_pairwise.qzv

