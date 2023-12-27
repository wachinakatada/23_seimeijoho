## メタバーコーディングによる細菌叢解析
### 2022年12月27日 - 28日

伊藤通浩（熱帯生物圏研究センター・分子生命科学研究施設）

2023年12月27日作成

<img src="https://raw.githubusercontent.com/wachinakatada/23_seimeijoho/main/Ito/Figure/ゲノムアセンブリーの流れ.jpg" width="1000">

### 1. ディレクトリをコピーする
形
```
$ cp –r [ディレクトリ名] [コピー先ディレクトリ名]
```
例
```
$ cp -r /mnt/bioInfo/bioInfo2023_share/ito/genome genome
```

### 2. 次世代シーケンスデータの中身を見る
形
```
$ zless [ファイル名]
または
$ less [ファイル名]
```
例
```
$ zless HQ_R1.fastq.gz  
# qのキーを押してストップ
```

### 3. シーケンスデータのリード数を調べる
形
```
$ seqkit stat [ファイル名]
```
例
```
$ seqkit stat HQ_R1.fastq.gz
```

### 3-① Raw dataのquality確認 [fastqc] 

- 解析結果の出力フォルダの事前作成が必要

形
```
$ mkdir [フォルダ名] 
```
例
```
$ mkdir fastqc_HQ_R1 
```

- 次に`fastqc`で解析

形
```
$ fastqc -o [outputのフォルダ名]/ [inputのファイル名] -t [数値]
```
例
```
$ fastqc -o fastqc_HQ_R1/ HQ_R1.fastq.gz -t 2 
```

### 3-② Read1(R1) とRead2 (R2) のmerge [vsearch --fastq_mergepaires]
- `vsearch`

形
```
$ vsearch --fastq_mergepairs [ファイル名 (R1, .fastq.gz)] \
-reverse [ファイル名 (R2, .fastq.gz)] \
--fastqout [マージされたファイル名( .fastq)] \
--fastqout_notmerged_fwd [マージされなかったファイル名 (R1,.fastq)] \
--fastqout_notmerged_rev [マージされなかったファイル名 (R1,.fastq)] \
--threads [数値] --fastq_allowmergestagger --fastq_minovlen [数値] --fastq_maxdiffs [数値]
```

例
```
$ vsearch --fastq_mergepairs HQ_R1.fastq.gz \
-reverse HQ_R2.fastq.gz \
--fastqout merged_HQ.fastq \
--fastqout_notmerged_fwd notmerged_fwd_HQ.fastq \
--fastqout_notmerged_rev notmerged_rev_HQ.fastq \
--threads 2 --fastq_allowmergestagger --fastq_minovlen 10 --fastq_maxdiffs 10
```

### 3-③ Quality control [vsearch --fastxfilter]

形
```
$ vsearch --fastx_filter [ファイル名(.fastq)] \
--fastqout [QCされたファイル名(.fastq)] \
--fastaout [QCされたファイル名(.fasta)] \
--fastq_truncee [数値] --fastq_truncqual [数値] \
```

```
例（c）　#マージされた配列をQC
$ vsearch --fastx_filter merged_HQ.fastq \
--fastqout qc_merged_HQ.fastq \
--fastaout qc_merged_HQ.fasta \
--fastq_truncee 0.5 --fastq_truncqual 10
```
```
例（c-2） #マージされた配列、マージされなかったR1配列、マージされなかったR2配列をまとめてQC
$ vsearch --fastx_filter merged_HQ.fastq \
--fastqout qc_merged_HQ.fastq \
--fastaout qc_merged_HQ.fasta \
--fastq_truncee 0.5 --fastq_truncqual 10 \
&& vsearch --fastx_filter notmerged_fwd_HQ.fastq \
--fastqout qc_notmerged_fwd_HQ.fastq \
--fastaout qc_notmerged_fwd_HQ.fasta \
--fastq_truncee 0.5 --fastq_truncqual 10 \
&& vsearch --fastx_filter notmerged_rev_HQ.fastq \
--fastqout qc_notmerged_rev_HQ.fastq \
--fastaout qc_notmerged_rev_HQ.fasta \
--fastq_truncee 0.5 --fastq_truncqual 10
```

### 3-④ Assembly [SPAdes]
- `SPAdes`

形 (a)・(b)
```
$ spades.py \
-1 [R1のファイル名 (fastq.gz, fastq, fasta)] \
-2 [R2のファイル名 (fastq.gz, fastq, fasta)] \
--only-assembler -k auto -t [数値] (--careful) \
-o [outputのフォルダ名]
# --carefulは任意 (使用推奨、ただし時間とメモリが必要)
```

例 (a)
```
$ spades.py \
-1 HQ_R1.fastq.gz \
-2 HQ_R2.fastq.gz \
--only-assembler -k auto -t 2 --careful \
-o HQ_assembled
```


形 (c)・(d)
```
$ spades.py \
-1 [R1のファイル名 (fastq.gz, fastq, fasta)] \
-2 [R2のファイル名 (fastq.gz, fastq, fasta)] \
--merged [マージされたファイル名(fastq, fasta)]
--only-assembler -k auto -t [数値] --careful \
-o [outputのフォルダ名]
```
例 (c)
```
$ spades.py \
-1 qc_notmerged_fwd_HQ.fasta \
-2 qc_notmerged_rev_HQ.fasta \
--merged qc_merged_HQ.fasta \
--only-assembler -k auto -t 12 --careful \
-o HQ_assembled
```
例 (d)
```
$ spades.py \
-1 notmerged_fwd_HQ.fastq \
-2 notmerged_rev_HQ.fastq \
--merged merged_HQ.fastq \
--only-assembler -k auto -t 2 --careful \
-o HQ_assembled
```

- アセンブルされたデータ（scaffolds.fasta）を一つ上のディレクトリに移す

形
```
$ cp [ファイル名] [コピー先ディレクトリ名]
```
例
```
$ cp scaffolds.fasta /home/[username]/genome
または
$ cp scaffolds.fasta ./..
```

### 3-⑤ Assemblyの評価 [quast]
- `quast`

- 複数のSPAdesのoutputを扱う場合は、outputのscaffolds.fastaの名前の変更を推奨
```
例(c)の場合
scaffolds.fasta を　HQ.scaffold.fasta に。
```

- ファイルの名前の変更

形
```
$ mv [ファイル名] [新ファイル名]
```
例
```
$ mv scaffolds.fasta HQ.scaffolds.fasta
```

- `quast`の実行

形
```
$ quast.py [アセンブルされたファイル名 (.fasta)] -o [outputのフォルダ名]
#outputのフォルダ名を指定しなくても自動的にフォルダは生成されるが、それぞれのフォルダ名の設定を推奨
```
例 (c)
```
$ quast.py HQ.scaffolds.fasta -o HQ_scaffolds

#終了後、outputフォルダをフォルダごと各自のPCに移動させ、各自のPCにてhtmlファイルを参照する
```
