# Sudachi同義語辞書 for DoqueDB

Sudachi同義語辞書 for DoqueDBは、
[ワークス徳島人工知能NLP研究所](https://worksapplications.github.io/Sudachi/)が
公開しているSudachi  
同義語辞書ソース(v20240408)を、DoqueDBの異表記展開辞書としてビルドしたものです。  
DoqueDBのSQLから同義語検索を行うために使用することができます。  
Sudachi同義語辞書については以下を参照してください。  
  [SudachiDict/docs/synonyms.md](https://github.com/WorksApplications/SudachiDict/blob/develop/docs/synonyms.md)

## 使ってみる

Sudachi同義語辞書を使った同義語検索の例を示します。  
あらかじめDoqueDBがインストールされ、 青空文庫の一部のデータを使ったサンプル  
データベースが作成されている
(DoqueDBの[README_ja.md](https://github.com/DoqueDB/doquedb/blob/master/README_ja.md)
にある「サンプルの実行」で  
./setup.shまでが実行されている)ものとします。

### インストール

dictディレクトリにある異表記展開辞書をDoqueDBの実行環境にコピーし、DoqueDBを再起動  
します。(元からある異表記展開辞書はサンプルのため、残しておく必要はありません。)

```
# cp dict/expStrStrWrd-Sudachi.dic /var/lib/DoqueDB/data/unadic/norm/expStrStrWrd.dic
# cp dict/expStrStrApp-Sudachi.dic /var/lib/DoqueDB/data/unadic/norm/expStrStrApp.dic
# /var/lib/DoqueDB/bin/doquedb restart
```

### 検索する

青空文庫の作品本文から「前途」とその同義語を検索してみましょう。  
異表記展開のためにEXPAND\_SYNONYM関数を使っています。  
以下のクエリをsample.sqlに保存します。

```
SELECT title, lastName, firstName,
    KWIC(content FOR 30 ENCLOSE WITH '<<<' AND '>>>')
  FROM AozoraBunko
  WHERE content CONTAINS EXPAND_SYNONYM('前途')
  LIMIT 10;
```

sqliでクエリを実行します。

```
$ /var/lib/DoqueDB/bin/sqli -remote localhost 54321 -user root \
    -password doqadmin -code utf-8 -database sampleSqli < sample.sql
{title,lastName,firstName,kwic(content for 30 enclose with '<<<' and '>>>')}
{青春の逆説,織田,作之助,気持になれなかった。（あの人は<<<前途>>>ある高等学校の学生さんだもの}
{海に生くる人々,葉山,嘉樹,入れることができた。それにはペンキで<<<未来>>>派の絵のような模様が、ベタ}
{西湖の屍人,海野,十三,ひとしかった。非常に心配して、<<<行く末>>>をいろいろと思い煩っている}
{海島冒険奇譚　海底軍艦,押川,春浪,覺えてしまうので、武村兵曹は此兒<<<將來>>>は非常に有望な撰手であると}
{思想と風俗,戸坂,潤,なっているが、今いった面白さは<<<将来>>>社会においても止揚されて伝承}
{光と風と夢,中島,敦,それ以外の職業に従っている<<<将来>>>の自分を想像して見ることが不可能}
{新版　放浪記,林,芙美子,ようなものと地図を私にくれた。<<<行く先>>>の私の仕事は、薬学生の助手}
{道標,宮本,百合子,いるでしょう？　大した力だと思うんです。何だか<<<未来>>>は底なしと}
{惜別,太宰,治,聞かせたが、あの最初の講義は、自分の<<<前途>>>を暗示し激励してくれている}
{津軽,太宰,治,財政もまた窮乏の極度に達し、<<<前途>>>暗澹たるうちにも、八代信明、}
$ 
```

「前途」の同義語「未来」「行く末」などは、Sudachi同義語辞書の以下のエントリで展開  
されています。「將來」はDoqueDBの正規化で「将来」として検索できるようになっています。

```
001051,1,0,1,0,0,0,(時間),前途,,
001051,1,0,2,0,0,0,(時間),将来,,
001051,1,0,3,0,0,0,(時間),未来,,
001051,1,0,4,0,0,0,(時間),行く先,,
001051,1,0,5,0,0,0,(時間),行く末,,
001051,1,0,6,0,0,0,(時間),先行き,,
001051,1,0,7,0,0,0,(時間),フューチャー,,
001051,1,0,7,0,0,1,(時間),future,,
```

## ビルド方法

Sudachi同義語辞書 for DoqueDBを最新のソースからビルドする方法を説明します。

### 準備

1. 以下を用意してください。作業はDoqueDBのソース環境で行います。  
詳細についてはDoqueDBの
[BUILDING_PROCEDURE_ja.md](https://github.com/DoqueDB/doquedb/blob/master/BUILDING_PROCEDURE_ja.md)
を参照してください。

* [DoqueDBソース一式](https://github.com/DoqueDB/doquedb/)
* gcc 4.8またはgcc 11.4
* perl-open

2. Sudachi同義語辞書のサイトから最新の
[synonyms.txt](https://github.com/WorksApplications/SudachiDict/blob/develop/src/main/text/synonyms.txt)
をRaw fileとしてダウンロードし、  
una/1.0/resource/src-data/normに置いてください。

3. 辞書のビルドに必要なヘッダファイルを配置します。  
gcc 4.8を使う場合はO114-64をO48-64と読み替えてください。

```
$ cd una/1.0
$ ../../common/tools/build/mkconfdir O114-64
$ c.O114-64
$ make conf-r
$ make installh-r
```

4. 辞書をビルドします。

```
$ cd ../resource
$ mkdir work
$ cd work
$ make -f ../tools/make/make-tools install
$ export LD_LIBRARY_PATH=`pwd`/../tools/bin:$LD_LIBRARY_PATH
$ make -f ../tools/make/make-norm-sudachi
```

以上により、work/nworkの下にexpStrStrWrd-sudachi.dic, expStrStrApp-sudachi.dicが作成  
されます。これらのファイルをDoqueDBの異表記正規化リソースであるunadic/normの下に  
expStrStrWrd.dic, expStrStrApp.dicとして置くことにより、異表記展開辞書として  
利用することができます。

## ライセンス

Sudachi同義語辞書 for DoqueDBは、Sudachiと同じくApache License 2.0で公開されています。  
著作権情報についてはLICENSE/NOTICE.txtをご覧ください。

## 謝辞

自然言語処理の分野で多くの有益かつ高度な資源を公開されているワークス徳島人工知能NLP  
研究所に感謝の意を表します。
