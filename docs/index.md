## 1. Bulk RNA-seq Issue Description
* The NFCORE_RNASEQ:RNASEQ:ALIGN_STAR:STAR_ALIGN step is notably slow when running bulk RNA-seq analysis on data from Genomics hubs AGR000166 and AGR000152. This slow operation causes the entire RNA-seq pipeline to fail, with errors reported as "Host EC2 (instance i-0486fb5f354ce15b1) terminated" or "Job attempt duration exceeded timeout.”

Runs | Sequencer | Library Type | Reference Genome | Avg. FASTQ Size 
------------| ------------ | ------------- | ------------ | ------------ 
AGR000152 | Novaseq 6000 | mRNA-seq  | GRCm39 | 3.5 GB
AGR000162 | Novaseq 6000  | mRNA-seq | GRCm39 | 1.6 GB

## Cause of the error
* Alignement Error: 
    * It's possible that the issue stems from the samples not aligning properly with the reference genome.
* Reference Genome:
    * It's pssible that the issue stems from the use inappropriate sample.
* Spike in:
    * It's pssible that the issue stems from the merge ERCC spike in with reference genome.
* Total RNA-seq mix with mRNA-seq:
    * It's pssible that the issue stems from the two mapped very slow runs are from the same data as the total RNA-seq mix.
* Pipeline updating
    * It's pssible that the issue stems from pipline updated.
* Read Length
    * It is possible that this problem stems from the fact that the mRNA needs to use a reads length of 101/101.


## Investigation Results
 * Research Design

Alignement Error | Reference Genome | Spike in | Total RNA-seq mix with mRNA-seq | Pipeline updating | Read Length
------------ | ------------- | ------------ | ------------ | ------------ | ------------ 
By downloading the trimmed fastq file and running it on personal EC2 by using the same command as nextflow | Alignment using GRCm38 as a reference genome | Do not add ERCC spike-in, use only GRCm39 as reference | Use the runs of the previous mix total RNA-seq (AGR000105) and mRNA-seq (AGR000108) to reanalyze | Use the former pipeline version (3.11.1) for analysis | Because normally, for mRNA, the reads length we would use is 101/101. but in this analysis, we are using 151/151, if I generate the reads length of 101/101 fastq file directly in the demux step, and then use the newly generated fastq for analysis |

* Research Results

Alignement Error | Reference Genome | Spike in | Total RNA-seq mix with mRNA-seq | Pipeline updating | Read Length
------------ | ------------- | ------------ | ------------ | ------------ | ------------ 
The downloaded fastq can run alignment on personal EC2 normally but it is also very slow | Despite the replacement of the reference genome, the alignment is still very slow | Despite not adding ERCC spike in, the comparison is still very slow | AGR000108(mRNAseq) can be analyzed normally | Despite using an older version of the rna-seq pipeline, the alignment is still very slow | After trimming the reads length of fastq to 101/101, RNA-seq can be analyzed normally. But once we also used 151/151 reads length to analyze novaseq mRNA data (AGR000177[GRCm39], AGR000108[GRCh38]), the previous results showed that they could be analyzed normally and very fast |

* Runs used for surveys

| Request ID | Read length | Reference | Spike in | RNAseq type | Run results | Sequencer | Description                                                                                                                             |
| ---------- | ----------- | --------- | -------- | ----------- | ----------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| AGR000105  | 151/151     | GRCm39    | No       | Total RNA   | Success     | Novaseq   | Previous run have mixed total RNA-seq and mRNA-seq                                                                                      |
| AGR000108  | 151/151     | GRCh38    | No       | mRNA        | Success     | Novaseq   |
| AGR000166  | 151/151     | GRCm39    | Yes      | mRNA        | Failed      | Novaseq   | Failed run: Alignement error, reference genome, Spike in, and pipeline updating                                                         |
| AGR000177  | 100/100     | GRCm39    | No       | mRNA        | Success     | Novaseq   | Runs mixed with total RNA-seq but can alignmenet fast when the reads length is 151/151. This run is also the same batch with AGR000166 |
| AGR000177  | 151/151     | GRCm39    | No       | mRNA        | Success     | Novaseq   |
| AGR000152  | 151/151     | GRCm39    | Yes      | mRNA        | Failed      | Novaseq   | Failed run: Alignement error, reference genome, Spike in, and pipeline updating                                                         |
| AGR000166  | 101/101     | GRCm39    | Yes      | mRNA        | Success     | Novaseq   | Trimmed the fastq into 101/101, to test the does read length influence the alignement reads                                             |



* STAR command example: 
```
STAR \
    --genomeDir star \
    --readFilesIn input1/<sample_name_read1.fq.gz> input2/<sample_name_read2.fq.gz> \
    --runThreadN 12 \
    --outFileNamePrefix sample_name. \
     \
    --sjdbGTFfile GRCm39_ERCC92.gtf \
    --outSAMattrRGline 'ID:sample_name'  'SM:sample_name'  \
    --quantMode TranscriptomeSAM --twopassMode Basic --outSAMtype BAM Unsorted --readFilesCommand zcat --runRNGseed 0 --outFilterMultimapNmax 20 --alignSJDBoverhangMin 1 --outSAMattributes NH HI AS NM MD --quantTranscriptomeBan Singleend --outSAMstrandField intronMotif
```

## Summary

* Before Trimmed (example: 17807)
```
                                 Started job on |	Jun 06 02:44:50
                             Started mapping on |	Jun 06 08:27:47
                                    Finished on |	Jun 06 16:28:35
       Mapping speed, Million of reads per hour |	3.02

                          Number of input reads |	24166036
                      Average input read length |	273
                                    UNIQUE READS:
                   Uniquely mapped reads number |	19877682
                        Uniquely mapped reads % |	82.25%
                          Average mapped length |	268.68
                       Number of splices: Total |	22378269
            Number of splices: Annotated (sjdb) |	22365606
                       Number of splices: GT/AG |	22197559
                       Number of splices: GC/AG |	147882
                       Number of splices: AT/AC |	17590
               Number of splices: Non-canonical |	15238
                      Mismatch rate per base, % |	0.20%
                         Deletion rate per base |	0.00%
                        Deletion average length |	1.68
                        Insertion rate per base |	0.00%
                       Insertion average length |	1.33
                             MULTI-MAPPING READS:
        Number of reads mapped to multiple loci |	2680963
             % of reads mapped to multiple loci |	11.09%
        Number of reads mapped to too many loci |	34149
             % of reads mapped to too many loci |	0.14%
                                  UNMAPPED READS:
  Number of reads unmapped: too many mismatches |	0
       % of reads unmapped: too many mismatches |	0.00%
            Number of reads unmapped: too short |	1549093
                 % of reads unmapped: too short |	6.41%
                Number of reads unmapped: other |	24149
                     % of reads unmapped: other |	0.10%
                                  CHIMERIC READS:
                       Number of chimeric reads |	0
                            % of chimeric reads |	0.00%

```

* After Trimmed (example: 17807)
```
Started job on |	Jun 06 11:22:11
                             Started mapping on |	Jun 06 11:39:03
                                    Finished on |	Jun 06 12:12:18
       Mapping speed, Million of reads per hour |	43.60

                          Number of input reads |	24159587
                      Average input read length |	198
                                    UNIQUE READS:
                   Uniquely mapped reads number |	19744219
                        Uniquely mapped reads % |	81.72%
                          Average mapped length |	196.93
                       Number of splices: Total |	15770903
            Number of splices: Annotated (sjdb) |	15765405
                       Number of splices: GT/AG |	15648274
                       Number of splices: GC/AG |	101550
                       Number of splices: AT/AC |	14109
               Number of splices: Non-canonical |	6970
                      Mismatch rate per base, % |	0.17%
                         Deletion rate per base |	0.00%
                        Deletion average length |	1.62
                        Insertion rate per base |	0.00%
                       Insertion average length |	1.31
                             MULTI-MAPPING READS:
        Number of reads mapped to multiple loci |	2784616
             % of reads mapped to multiple loci |	11.53%
        Number of reads mapped to too many loci |	41540
             % of reads mapped to too many loci |	0.17%
                                  UNMAPPED READS:
  Number of reads unmapped: too many mismatches |	0
       % of reads unmapped: too many mismatches |	0.00%
            Number of reads unmapped: too short |	1578930
                 % of reads unmapped: too short |	6.54%
                Number of reads unmapped: other |	10282
                     % of reads unmapped: other |	0.04%
                                  CHIMERIC READS:
                       Number of chimeric reads |	0
                            % of chimeric reads |	0.00%
```
* Read base pair compairsion

| Total Bases after mapped | Total Bases before mapped | unmapped bases |
| ------------------------ | ------------------------- | -------------- |
| 3026331473               | 3654905019                | 628573546     |
| 2217107204               | 2444671569                | 227564366      |

* **Based on this investigation, we think it might cause by a locus (at the end of the reads) which has very small exons and appears to be highly expressed in the data.**

## Solution
* running STAR with ``--seedPerWindowNmax 30``
     * Through adding ``--seePerWindowNmax`` the speed increase by a factor  
     * Using ``--seePerWindowNmax`` will reduce the mapping sensitivity
          * ``--seePerWindowNmax 30 `` increase speed by a factor of 10, but lose ~1%
          * Please refrence to https://github.com/alexdobin/STAR/issues/1414 for more details.
