# QIIME2 is an open-source bioinformatics pipeline. Please, refer to the "Natively installing QIIME2 (https://docs.qiime2.org/2020.6/install/native/)" to install and run the QIIME2 software.


# Typical install time on a normal desktop computer is about 30 minutes.


# metafile, manifest file
metadata_preliminary
manifest_preliminary


# Activating QIIME2
conda activate qiime2-2020.6


# Importing data
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path preliminary/manifest_preliminary.txt --output-path preliminary/preliminary-paired-end-demux.qza --input-format PairedEndFastqManifestPhred33V2
[SampleData-PairedEndSequencesWithQuality, input format-PairedEndFastqManifestPhred33V2]

cd khs/fmt/preliminary/qiime2


# Demultiplexing sequences
qiime demux summarize --i-data preliminary-paired-end-demux.qza --o-visualization preliminary-paired-end-demux-summary.qzv


# QIIME2 view
file: preliminary-paired-end-denux-summary.qzv
Interactive Quality Plot --> sequence base


# Sequence quality control and feature table construction
qiime dada2 denoise-paired --i-demultiplexed-seqs preliminary-paired-end-demux.qza --p-trim-left-f 14 --p-trim-left-r 9 --p-trunc-len-f 266 --p-trunc-len-r 220 --o-table preliminary-table-14f266-9r220.qza --o-representative-sequences preliminary-rep-seqs-14f266-9r220.qza --o-denoising-stats preliminary-denoising-stats-14f266-9r220.qza --p-n-threads 127


# Convert qza to qzv
qiime metadata tabulate --m-input-file preliminary-denoising-stats-14f266-9r220.qza --o-visualization preliminary-denoising-stats-14f266-9r220.qzv


# FeatureTable and FeatureData summaries
qiime feature-table summarize --i-table preliminary-table-14f266-9r220.qza --o-visualization preliminary-table-14f266-9r220.qzv --m-sample-metadata-file metadata_preliminary.txt


# Amplicon denoising
qiime feature-table filter-features --i-table preliminary-table-14f266-9r220.qza --p-min-frequency 10 --p-min-samples 2 --o-filtered-table preliminary-table-14f266-9r220-minfreq10-minsamples2.qza
qiime feature-table summarize --i-table preliminary-table-14f266-9r220-minfreq10-minsamples2.qza --o-visualization preliminary-table-14f266-9r220-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_preliminary.txt


# Generate a tree for phylogenetic diversity analyses 
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences preliminary-rep-seqs-14f266-9r220.qza --o-alignment preliminary-aligned-rep-seqs-14f266-9r220.qza --o-masked-alignment preliminary-masked-aligned-rep-seqs-14f266-9r220.qza --o-tree preliminary-unrooted-tree-14f266-9r220.qza --o-rooted-tree preliminary-rooted-tree-14f266-9r220.qza --p-n-threads 127
[pipeline: Build a phylogenetic tree using fasttree and mafft alignment]


# Rarefy table
qiime feature-table rarefy --i-table preliminary-table-14f266-9r220-minfreq10-minsamples2.qza --p-sampling-depth 28147 --o-rarefied-table preliminary-rarefied-table-14f266-9r220-minfreq10-minsamples2.qza


# Generate visual and tabular summaries of a feature table (rarefy confirmation)
qiime feature-table summarize --i-table preliminary-rarefied-table-14f266-9r220-minfreq10-minsamples2.qza --o-visualization preliminary-rarefied-table-14f266-9r220-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_preliminary.txt


# Applies a collection of diversity metrics  to a feature table.
qiime diversity core-metrics-phylogenetic --i-phylogeny preliminary-rooted-tree-14f266-9r220.qza --i-table preliminary-rarefied-table-14f266-9r220-minfreq10-minsamples2.qza --p-sampling-depth 28147 --m-metadata-file metadata_preliminary.txt --output-dir preliminary-core-metrics-results/ --p-n-jobs-or-threads 127


# Taxonomic classification (full length classifier usage)
qiime feature-classifier classify-sklearn --i-reads preliminary-rep-seqs-14f266-9r220.qza --i-classifier gg-13-8-99-nb-classifier.qza --o-classification preliminary-taxonomy-14f266-9r220.qza --p-n-jobs 127


# Visualize taxonomy with an interactive bar plot
qiime taxa barplot --i-table preliminary-rarefied-table-14f266-9r220-minfreq10-minsamples2.qza --i-taxonomy preliminary-taxonomy-14f266-9r220.qza --m-metadata-file metadata_preliminary.txt --o-visualization preliminary-taxonomy-barcharts-14f266-9r220.qza


# Convert qza to qzv
cd preliminary-core-metrics-results/
qiime metadata tabulate --m-input-file rarefied_table.qza --o-visualization rarefied_table.qzv


# Sourcetracker
feature-table_preliminary.txt
preliminary_full_mapping_full_merged_sourcetracker2.txt

/home/qiime/miniconda3/lib64/R/bin/exec/R --slave --vanilla --args -i feature-table_preliminary.txt -m preliminary_full_mapping_full_merged_sourcetracker2.txt -o sourcetracker_out