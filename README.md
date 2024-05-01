# Sudachi synonym dictionary for DoqueDB

The Sudachi synonym dictionary for DoqueDB is built from the Sudachi synonym
dictionary source (v20240408) published by WAP Tokushima Laboratory of AI and NLP,
as a variant expansion dictionary for DoqueDB.
It can be used to perform synonym searches from DoqueDB SQL.  
For more information on the Sudachi synonym dictionary, see:
  [SudachiDict/docs/synonyms.md](https://github.com/WorksApplications/SudachiDict/blob/develop/docs/synonyms.md)

## Search example

Here is an example of synonym search using the Sudachi synonym dictionary.  
It is assumed that DoqueDB has been installed and a sample database using partial
data from the Aozora Bunko has been created in advance
(the "Execute examples" in [README.md](https://github.com/DoqueDB/doquedb/blob/master/README.md)
of DoqueDB has been executed up to ./setup.sh).

### Installation

Copy the variant expansion dictionaries in the "dict" directory to the DoqueDB
execution environment and restart DoqueDB. (You don't need to keep the original
dictionaries in the destination as they are for sample purposes.)

```
# cp dict/expStrStrWrd-Sudachi.dic /var/lib/DoqueDB/data/unadic/norm/expStrStrWrd.dic
# cp dict/expStrStrApp-Sudachi.dic /var/lib/DoqueDB/data/unadic/norm/expStrStrApp.dic
# /var/lib/DoqueDB/bin/doquedb restart
```

### Running search

Let's search for "前途(future)" and its synonyms in the text of Aozora Bunko's works.
The EXPAND\_SYNONYM function is used for variant expansion.  
Save the query below to sample.sql.

```
SELECT title, lastName, firstName,
    KWIC(content FOR 30 ENCLOSE WITH '<<<' AND '>>>')
  FROM AozoraBunko
  WHERE content CONTAINS EXPAND_SYNONYM('前途')
  LIMIT 10;
```

Execute the query with sqli.

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

The synonyms of "前途" such as "未来" and "行く末" are expanded by the following
source entries of the Sudachi synonym dictionary.
The word "將來" can be searched as "将来" using DoqueDB normalization.

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

## Building procedure

This section explains how to build the Sudachi synonym dictionary for DoqueDB
from the latest sources.

### Preparation

1. Please prepare the following materials.
Work will be done in the DoqueDB source environment.
See [BUILDING\_PROCEDURE\_ja.md](https://github.com/DoqueDB/doquedb/blob/master/BUILDING_PROCEDURE_ja.md)
in DoqueDB for details.

* [DoqueDB source code](https://github.com/DoqueDB/doquedb/)
* gcc 4.8 or gcc 11.4
* perl-open

2. Download the latest
[synonyms.txt](https://github.com/WorksApplications/SudachiDict/blob/develop/src/main/text/synonyms.txt)
as a raw file from the Sudachi synonym dictionary site and place it in
una/1.0/resource/src-data/norm.

3. Place the header files needed to build the dictionary.
When using gcc 4.8, read O114-64 as O48-64.

```
$ cd una/1.0
$ ../../common/tools/build/mkconfdir O114-64
$ c.O114-64
$ make conf-r
$ make installh-r
```

4. Build the dictionary.

```
$ cd ../resource
$ mkdir work
$ cd work
$ make -f ../tools/make/make-tools install
$ export LD_LIBRARY_PATH=`pwd`/../tools/bin:$LD_LIBRARY_PATH
$ make -f ../tools/make/make-norm-sudachi
```

With the above steps, expStrStrWrd-sudachi.dic and expStrStrApp-sudachi.dic
are created under work/nwork.
By placing these files as expStrStrWrd.dic and expStrStrApp.dic under unadic/norm,
the DoqueDB normalizer resource directory, they can be used as a variant expansion
dictionary.

## Licenses

The Sudachi synonym dictionary for DoqueDB is published under
the Apache License 2.0 like Sudachi.  
For copyright information, see LICENSE/NOTICE.txt.

## Acknowledgment

We would like to express our gratitude to WAP Tokushima Laboratory of AI and NLP,
which has published many useful and advanced resources in the field of natural
language processing.
