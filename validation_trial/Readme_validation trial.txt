# QIIME2 is an open-source bioinformatics pipeline. Please, refer to the "Natively installing QIIME2 (https://docs.qiime2.org/2020.6/install/native/)" to install and run the QIIME2 software.


# Typical install time on a normal desktop computer is about 30 minutes.


# metafile, manifest file
metadata_total.txt
manifest_total.txt


# Activating QIIME2
conda activate qiime2-2020.6


# Importing data
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path total/manifest_total.txt --output-path total/total-paired-end-demux.qza --input-format PairedEndFastqManifestPhred33V2
[SampleData-PairedEndSequencesWithQuality, input format-PairedEndFastqManifestPhred33V2]


total/

# Demultiplexing sequences
qiime demux summarize --i-data total-paired-end-demux.qza --o-visualization total-paired-end-demux-summary.qzv


# QIIME2 view
file: total-paired-end-denux-summary.qzv
Interactive Quality Plot --> sequence base


# Sequence quality control and feature table construction
qiime dada2 denoise-paired --i-demultiplexed-seqs total-paired-end-demux.qza --p-trim-left-f 16 --p-trim-left-r 9 --p-trunc-len-f 285 --p-trunc-len-r 226 --o-table total-table-16f285-9r226.qza --o-representative-sequences total-rep-seqs-16f285-9r226.qza --o-denoising-stats total-denoising-stats-16f285-9r226.qza --p-n-threads 127


# Convert qza to qzv
qiime metadata tabulate --m-input-file total-denoising-stats-16f285-9r226.qza --o-visualization total-denoising-stats-16f285-9r226.qzv


# FeatureTable and FeatureData summaries
qiime feature-table summarize --i-table total-table-16f285-9r226.qza --o-visualization total-table-16f285-9r226.qzv --m-sample-metadata-file metadata_total.txt


# Amplicon denoising
qiime feature-table filter-features --i-table total-table-16f285-9r226.qza --p-min-frequency 10 --p-min-samples 2 --o-filtered-table total-table-16f285-9r226-minfreq10-minsamples2.qza
qiime feature-table summarize --i-table total-table-16f285-9r226-minfreq10-minsamples2.qza --o-visualization total-table-16f285-9r226-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_total.txt


# Generate a tree for phylogenetic diversity analyses 
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences total-rep-seqs-16f285-9r226.qza --o-alignment total-aligned-rep-seqs-16f285-9r226.qza --o-masked-alignment total-masked-aligned-rep-seqs-16f285-9r226.qza --o-tree total-unrooted-tree-16f285-9r226.qza --o-rooted-tree total-rooted-tree-16f285-9r226.qza --p-n-threads 127
[pipeline: Build a phylogenetic tree using fasttree and mafft alignment]


# Rarefy table
qiime feature-table rarefy --i-table total-table-16f285-9r226-minfreq10-minsamples2.qza --p-sampling-depth 20125 --o-rarefied-table total-rarefied-table-16f285-9r226-minfreq10-minsamples2.qza


# Generate visual and tabular summaries of a feature table
qiime feature-table summarize --i-table total-rarefied-table-16f285-9r226-minfreq10-minsamples2.qza --o-visualization total-rarefied-table-16f285-9r226-minfreq10-minsamples2.qzv --m-sample-metadata-file metadata_total.txt


#Applies a collection of diversity metrics  to a feature table.
qiime diversity core-metrics-phylogenetic --i-phylogeny total-rooted-tree-16f285-9r226.qza --i-table total-rarefied-table-16f285-9r226-minfreq10-minsamples2.qza --p-sampling-depth 20125 --m-metadata-file metadata_total.txt --output-dir total-core-metrics-results/ --p-n-jobs-or-threads 127


# taxonomic classification (full length classifier usage)
qiime feature-classifier classify-sklearn --i-reads total-rep-seqs-16f285-9r226.qza --i-classifier gg-13-8-99-nb-classifier.qza --o-classification total-taxonomy-16f285-9r226.qza --p-n-jobs 127


# Visualize taxonomy with an interactive bar plot
qiime taxa barplot --i-table total-rarefied-table-16f285-9r226-minfreq10-minsamples2.qza --i-taxonomy total-taxonomy-16f285-9r226.qza --m-metadata-file metadata_total.txt --o-visualization total-taxonomy-barcharts-16f285-9r226.qza


# Sourcetracker
feature-table_validation_abx.txt
abx_full_mapping_full_merged_sourcetracker2.txt
R --slave --vanilla --args -i new/abx/feature-table_validation_abx.txt -m new/abx/abx_full_mapping_full_merged_sourcetracker2.txt -o new/abx/sourcetracker_out_rarefied5k -r 5000 --train_rarefaction 5000 < sourcetracker_for_qiime.r

feature-table_validation_con.txt
con_full_mapping_full_merged_sourcetracker2.txt
R --slave --vanilla --args -i new/cont/feature-table_validation_con.txt -m new/cont/con_full_mapping_full_merged_sourcetracker2.txt -o new/cont/sourcetracker_out_rarefied5k -r 5000 --train_rarefaction 5000 < sourcetracker_for_qiime.r Rarefying training data at 5000

feature-table_validation_fmt.txt
fmt_full_mapping_full_merged_sourcetracker2.txt
R --slave --vanilla --args -i new/fmt/feature-table_validation_fmt.txt -m new/fmt/fmt_full_mapping_full_merged_sourcetracker2.txt -o new/fmt/sourcetracker_out_rarefied5k -r 5000 --train_rarefaction 5000 < sourcetracker_for_qiime.r
