# 演習C

[トップページに戻る](../)

## 目次

- [演習C](#演習c)
  - [目次](#目次)
  - [１細胞RNA-seqについて](#１細胞rna-seqについて)
  - [演習Cでは、何をやるの？](#演習cでは何をやるの)
    - [プラナリア？](#プラナリア)
    - [どんなデータ？](#どんなデータ)
  - [１細胞RNA-seqって？](#１細胞rna-seqって)
    - [１細胞RNA-seq解析の基本的な流れ](#１細胞rna-seq解析の基本的な流れ)
  - [演習Cの内容](#演習cの内容)
    - [[演習C] 1. データをダウンロードする](#演習c-1-データをダウンロードする)
    - [[演習C] 2. 今回使用するノートブックをダウンロードする](#演習c-2-今回使用するノートブックをダウンロードする)
  - [基本課題C-1](#基本課題c-1)
  - [発展課題C-1](#発展課題c-1)
    - [発展課題C-1の内容](#発展課題c-1の内容)
    - [発展課題C-1の進め方](#発展課題c-1の進め方)
    - [発展課題C-1の提出様式の指定](#発展課題c-1の提出様式の指定)
  - [発展課題C-2](#発展課題c-2)

## １細胞RNA-seqについて

[演習C スライド](https://github.com/bioinfo-tsukuba/FY2022-EB62104-Bioinformatics/raw/main/%E6%BC%94%E7%BF%92C/%E6%BC%94%E7%BF%92C.pdf)

## 演習Cでは、何をやるの？

- プラナリアの１細胞RNA-seqデータを解析します

### プラナリア？

- 切っても再生する生き物
- https://en.wikipedia.org/wiki/Schmidtea_mediterranea

![](img/2021-02-06-15-04-22.png)

### どんなデータ？

- 元論文
  - Comparative transcriptomic analyses and single-cell RNA sequencing of the freshwater planarian Schmidtea mediterranea identify major cell types and pathway conservation https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1498-x
  - freshwater planarian *Schmidtea mediterranea*
  - Drop-seq
- プラナリアの scRNA-seq データ解析
  - https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE115280
  - cluster markers https://figshare.com/articles/Additional_file_4/6852896
  - https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1498-x

## １細胞RNA-seqって？

スライド「演習C.pdf」参照

### １細胞RNA-seq解析の基本的な流れ

今回は１細胞RNA-seq解析でよく使われる [Seurat （すーら）](https://satijalab.org/seurat/) というパッケージを使用します。

Drop-seqという１細胞RNA-seqの場合、どの細胞がどんな細胞型かがわからない。そこで、個々の細胞がどんな細胞かを「遺伝子発現のみから」明らかにする必要がある。この作業を細胞型アノテーション（Cell type annotation）と呼びます。

- 1. 遺伝子発現のカウント行列を読み込む
  - 行が遺伝子数 x 列が細胞、各要素がある遺伝子のある細胞でのリードカウントになっている
- 2. 品質の低い細胞をフィルターする
  - 実験手法の制約から全ての細胞が良い品質のデータに変換されているわけではない
  - ここでは、 `nFeature_RNA` （ある細胞で検出された遺伝子の数）、 `nCount_RNA` (ある細胞のリードカウントの合計) が低い細胞を除く（高い細胞だけを選ぶ）
- 3. 発現量データを正規化する
  - 細胞ごとにリードカウントの合計値が違う場合、元のカウントを細胞間で比較しても意味がない
    - 遺伝子発現量は「割合」に近いイメージ
  - そこで、細胞間で遺伝子発現量を比較できるように、カウントデータを正規化する
- 4. 高変動遺伝子（highly variabe genes) を抽出する
  - 全ての遺伝子の発現量が重要なわけではない
  - 「個々の細胞の細胞型の違い・多様性」を見分けるためには、「細胞間で発現量が異なる遺伝子」を見なければならない
  - そこで「細胞間で発現量が大きく変動している遺伝子」を抽出する
    - この際、遺伝子によっては「ノイズ」のように変動するものもあるため、統計学的に有意に高い変動を示す遺伝子を抽出することが重要
- 5. 発現量データをスケーリングする
  - 細胞のアノテーションの前段階として、細胞のクラスタリングを行いたい
    - 計算処理の高速化や計測ノイズをならす意味がある
  - 細胞のクラスタリングには、遺伝子たちを変数として使用する
  - この際、このままでクラスタリングすると、発現量が大きい遺伝子の影響が大きくなる
  - そのため、遺伝子間での発現量のスケールを揃える（スケーリング）ことが必要となる
- 6. PCA（主成分分析）を用いて次元削減を行う
  - クラスタリングの前に次元圧縮をすることで、データの多様性をなるべく損ねずに効率的にクラスタリングができる
  - PCAはデータの多様性をなるべく損ねずに次元圧縮できる
- 7. 細胞をクラスタリングする
- 8. 各クラスターに特徴的な遺伝子群を探す
- 9. 各クラスターがどんな細胞型かを類推する
  - 遺伝子機能の知識が不足している場合は、オーソログの情報を使うと良い

## 演習Cの内容

### [演習C] 1. データをダウンロードする

Jupyter Hub のターミナルで、以下を実行する：

```bash
# ダウンロードする
wget ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM3173nnn/GSM3173562/suppl/GSM3173562_Lakshmipuram_NCBI_processeddata.txt.gz

# ダウンロードしたファイルを data というディレクトリへ移動する
mv GSM3173562_Lakshmipuram_NCBI_processeddata.txt.gz data/

# 解凍する
gunzip data/GSM3173562_Lakshmipuram_NCBI_processeddata.txt.gz

# 行数を調べる ( 51563 data/GSM3173562_Lakshmipuram_NCBI_processeddata.txt と表示されればOK )
wc -l data/GSM3173562_Lakshmipuram_NCBI_processeddata.txt

# 中身を見てみる ( q を押すと戻れる )
less data/GSM3173562_Lakshmipuram_NCBI_processeddata.txt
```

### [演習C] 2. 今回使用するノートブックをダウンロードする

Jupyter Hub のターミナルで、以下を実行する：

```bash
wget https://www.dropbox.com/s/7v3szc3nku52ajj/planarian_single_cell.ipynb
```

----
----
----
----
----

## 基本課題C-1

- `planarian_single_cell.ipynb` を Jupyter Hub 上で開き、上から１つずつセルを実行せよ。
  - その際、ノートブックに含まれる説明書きをよく読むこと。
- Jupyter Hub上のメニューから `File > Download as > Notebook (.ipynb)` とすることで、実行結果をJupyter notebook形式でダウンロードできるので、それを manaba で提出せよ

## 発展課題C-1

### 発展課題C-1の内容

- 各クラスターに特異的な遺伝子群がどのような機能を持つ遺伝子かを調べ、レポートとしてまとめよ
  - 少なくとも３つのクラスターについて調べよ

### 発展課題C-1の進め方

Jupyter Hub のターミナルで、以下を実行する：

```bash
wget https://www.dropbox.com/s/wti9npcy409rmrv/human_ortholog_subset.tsv

mv human_ortholog_subset.tsv data/

wget https://www.dropbox.com/s/sw7x30vjb8i1doy/Advanced-C-1.ipynb
```

次に、 `Advanced-C-1.ipynb` を Jupyter Hub 上で開き、すでにあるコードを参考にしつつ、書いていく。

### 発展課題C-1の提出様式の指定

- 様式は以下の通りにする。様式が満たされていない場合は０点とする。
  - ファイル形式：Word
  - ページ数：２ページ以内（ギリギリまで書いた方が成績が良いという意味ではない、例えば２ページでも満点になりうる）
  - フォント：サイズは 10ポイント、そのほかは自由
  - 構成：
    - タイトル
    - 氏名
    - 学籍番号
    - 目的（本課題の目的を書く）
    - 方法
    - 結果（どんな遺伝子が多かったか）
    - 考察（結果がこれまで）
    - 結論（目的を達成できたかを結果に基づいて書く）

## 発展課題C-2

> ※ これは余力のある人だけチャレンジしてみてください

- この論文 <https://doi.org/10.1016/j.bbrc.2020.03.044> ではヒトの13の組織において ACE2など SARS-COV2 の感染に関連する受容体の発現を調査している。
- Table S1 (Excelファイル) を参考に好きな組織の１細胞RNA-seqデータをダウンロードし、 [planarian_single_cell.ipynb](planarian_single_cell.ipynb) を参考に、データ前処理・解析を行い、ACE2遺伝子の発現量が高い細胞があるかを調べよ
- Jupyter Hub上のメニューから `File > Download as > Notebook (.ipynb)` とすることで、実行結果をJupyter notebook形式でダウンロードできるので、それを manaba で提出せよ

<!-- # 演習B 追記

## 実行済みファイルについて

- 実行済みのノートブックもアップロードしました
  - [planarian_single_cell_Executed.ipynb](planarian_single_cell_Executed.ipynb)
- [planarian_markers.rds](planarian_markers.rds) をRで読み込むことで、細胞クラスター特異的遺伝子群を用いた解析を実施できます
- また、B01からB16の結果の `planarian` も公開しました（ファイルサイズが大きいため、figshareというサービスにおいています）
  - [https://figshare.com/articles/dataset/planarian_B01-B16_rds/13726531](https://figshare.com/articles/dataset/planarian_B01-B16_rds/13726531)


## 課題について

## R で途中の結果までを保存する `saveRDS()` について

- 途中の結果を保存することができます
  - fileのところは好きに名前をつけられます

```R
# 保存
saveRDS(planarian, file = "planarian_B09.rds")

# 途中の結果を読み込んでそこからやり直す
planarian <- readRDS("planarian_B09.rds")
``` -->
