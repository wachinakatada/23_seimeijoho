# コマンドライン入門
藤本真悟（医学部・附属実験実習機器センター）  
和智仲是（熱帯生物圏研究センター・西表研究施設）  
作成：2022年12月20日
___

# コマンドラインでよく使う操作

# 1. 計算機の使用状況の確認 top

```
# CPU, メモリ, 実行中のコマンドに関する要約を表示する
top

# topコマンドの実行終了 
q 
```

![top_コマンド](https://user-images.githubusercontent.com/96465701/208613308-e2bdeeb9-fcc2-405e-8e2d-975dbd8df2f9.PNG)

```
# [ユーザー名] が実行しているコマンドのみを表示
top -u [ユーザー名]
```

# 2. 実行中のコマンドの強制終了

```
# 以下のファイルは、メダカのリファレンスゲノムの塩基配列情報
# ファイルが大きいので(約700MB, 7億塩基を含む)、catで開くと終了まで時間がかかる

cat /mnt/bioInfo/bioInfo2022_share/sfujimoto/medaka_HdrR_genome.fasta

# 実行途中で強制終了するには
# キーボードのCtrl, c を同時に押下

```

# 3. 実行中のコマンドの一時停止と再開

```
# 2. と同様にメダカのゲノムを表示
cat /mnt/bioInfo/bioInfo2022_share/sfujimoto/medaka_HdrR_genome.fasta

# 実行途中で一時停止したいときは
# キーボードのCtrl, z を同時に押下
# コマンドが入力可能な状態になる

# 待機中のコマンドも含めたリストを表示
jobs

# 一時停止したコマンドの再開
fg

```

# 4. ネットワーク上からのファイルの取得

SARS-CoV-2 ウイルスのリファレンスゲノム配列をNCBIからダウンロードしてみる
https://www.ncbi.nlm.nih.gov/assembly/GCF_009858895.2

```
# NCBIのwebサイトに接続してfasta をダウンロード
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/009/858/895/GCF_009858895.2_ASM985889v3/GCF_009858895.2_ASM985889v3_genomic.fna.gz

# gzip圧縮されたfastaファイルを読み込みできるように解凍して表示
gunzip GCF_009858895.2_ASM985889v3_genomic.fna.gz
cat GCF_009858895.2_ASM985889v3_genomic.fna

```

Github (https://github.co.jp) では、プログラムや関連するドキュメントが公開されている  
この生命情報科学の講義用に、Github上で公開している資料を一括でコピーしてダウンロードする  

```
# Github上のURLを指定
git clone https://github.com/wachinakatada/22_seimeijoho.git

# ファイルのリストを表示
# 22_seimeijohoというディレクトリが作成されているか確認
ls -l

```

# 5. ファイルを1行ずつ読み込んで内容を変更する

```
# ファイルの内容を表示して確認
cat /mnt/bioInfo/bioInfo2022_share/sfujimoto/textSample.txt

# 数値が5行分含まれる
#   0
#   1
#  10
#  11
# 100


# 同じ行を2回繰り返して表示する
# while read: txtRowという名前の変数として、1行分の情報を保持する
cat /mnt/bioInfo/bioInfo2022_share/sfujimoto/textSample.txt | while read txtRow
do
 # 変数の中身を表示する
 echo $txtRow 
 echo $txtRow
done

# 0 を X に置換した出力結果を表示する
cat /mnt/bioInfo/bioInfo2022_share/sfujimoto/textSample.txt | while read line
do
 echo $line | sed -e 's/0/X/g' 
done


```


