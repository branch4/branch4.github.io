---
layout: post
title: "Resize Temp File of Oracle on RDS"
date: 2014-06-15T23:20:00-07:00
comments: true
published: true
categories:
 - aws
---

どもども。[saorister](http://portfolio.risterlab.com)です。  
野菜大好き、刺繍大好き、将来は書道の先生になりたーい、そんなインフラエンジニアです。  
(統一性なさすぎw)  

今回は私がブログ記事担当！  
ということで、  
この前ちと困ったのに、あんまりネットにヒントがなかったことをひとつご紹介。  

###DBのディスク空き容量が!!!
![DBのディスク空き容量が!!!](http://blog.branch4.pw/images/2014/06/figure_graph_down.png)  

RDSのOracleさんでTEMP表領域サイズがめっちゃでっかくなっちゃって  
空き容量がほとんどなくなっちゃったよぉぉぉぉ！！！  
  
パニック！！！  
  
って時の対処法を。  
  
<!-- more -->  
  
### 起きたこと
----------

1. どこかのおばかさんがとんでもSQLを投げた  
1. コアDB(RDS)のディスク容量があと10%以下よーというアラートがCloudWatchから飛んできて異常に気づく  
1. 怪しいセッションを発見。すぐにkill!KILL!!キル！！！  
1. とりあえずぐんぐん下がっていくディスクの空き容量のグラフはもうほぼ０のところでぎりぎり止まった。  はぁ。  
1. ひとまずディスク容量増やそうぜ！このためのRDSじゃないか！ってことでディスク容量を10%増量。(10%以上じゃないとあげられないのよ)  
  
退屈な毎日だとは思っていたけどこんなスリルは求めてない。  
  
### 増えたTemp領域を小さくする
----------
とんでもSQLってのはselect文だったわけだけど  
むちゃくちゃなjoinをしてTEMP表領域(一時領域)が膨れ上がっちゃったわけですね。  
こわいねー。  
で、慌ててセッションkillしたから、その一時領域は開放されずそのまま残っちゃった。  
ので、これを小さくしてあげないといけない。  
「oracle temp領域 resize」「oracle 一時領域 縮小」  
とかで検索するとやりかたはいっぱいでてきます。  
だけどそこに書いてあるこのコマンド  
  
`SQL> alter database tempfile ~`  
  
ってのはRDSじゃ権限がなくてできないんですよ。  
はい、困った。  
  
ってわけで先に結論。  
  
####RDSのoracleのtemp領域(一時領域)を縮小する方法
1.TEMP表領域を新規作成します。  
  
```
SQL> CREATE TEMPORARY TABLESPACE TEMP02;
Tablespace created.
```
  
2.デフォルトのTEMP表領域を1.で作成したものに変更します。  
  
```
SQL> exec rdsadmin.rdsadmin_util.ALTER_DEFAULT_TEMP_TABLESPACE('TEMP02')
PL/SQL procedure successfully completed.
```
  
3.元のTEMP表領域を削除します。  
  
```
SQL> DROP TABLESPACE temp;
Tablespace dropped.
```
  
##実際にやったこと Try & Error編
上で書いた結論はたった３つだけど、そこにたどり着くまでの試行錯誤も書いちゃうよ。  
まず膨れ上がったのがTEMP表領域だったことの確認はこのSQLで。  

```
SELECT dt.file_name, dt.tablespace_name,
to_char(dt.bytes / 1024, '99999990.000') file_kbytes,
 to_char(t.bytes_cached / 1024, '99999990.000') used_kbytes,
 to_char(t.bytes_cached / dt.bytes * 100, '990.00') || '%' capacity
FROM sys.dba_temp_files dt, v$temp_extent_pool t, v$tempfile v
WHERE t.file_id(+)= dt.file_id AND dt.file_id = v.file#;
```

こんな状態でした。  
![temp30GB](http://blog.branch4.pw/images/2014/06/temp_30GB_masking.png)
  
で、単純に縮小しようとしてもできません。
  
`SQL> alter tablespace temp resize 100M `
  
```
SQLエラー: ORA-03297: ファイルには、リクエストしたRESIZE値を超える使用中のデータが含まれています。
03297. 00000 -  "file contains used data beyond requested RESIZE value"
*Cause:    Some portion of the file in the region to be trimmed is
           currently in use by a database object
*Action:   Drop or move segments containing extents in this region prior to
           resizing the file, or choose a resize value such that only free
           space is in the trimmed.
```
  
こう言われちゃいますね。使ってるのよって。  
じゃあ、ってことでオンプレのoracleで普通やる縮小方法を試してみると。  
  
`SQL> alter database tempfile '/path/to/temp/file1.tmp' resize 10M;`  
-> はいalter databaseはRDSでうてませーん。権限ありませーん。
  
じゃあお決まりの、新しくTEMP表領域を小さいのつくって、置き換える作成へ。  
  
`SQL> create temporary tablespace TEMP02 size 10M; `  
-> だめ。これも権限ないって怒られる。  
でもこれはファイルサイズを決める権限がないってことらしく、これなら  
通った。  
`SQL> create temporary tablespace TEMP02; `  
  
さっきのSQLで確認するとtmpファイルが２つでてきて、  
新しいものは100MBでできていることがわかるはず。  
  
よし、ここでデフォルトのTEMP表領域を置き換える。  
そうalter databaseじゃなくてexec rdsadminを使ってね！  
  
```
SQL> exec rdsadmin.rdsadmin_util.ALTER_DEFAULT_TEMP_TABLESPACE('TEMP02')
PL/SQL procedure successfully completed.
```
  
よーしこれで用無しになったおデブちゃんを削除だ！  
  
```
SQL> DROP TABLESPACE temp;
Tablespace dropped.
```
  
ぐっばい、おでぶちゃん。  
勝利の美酒に浸りながらCloudWatchで空き容量が一気にもどってるのを見ようとしたら  
なんと！  
グラフ微動だにせず、、、、  
むしろ追加したTEMP表領域の分の100MB分減っとるやないかーい！  
くそ、結局再起動しないと消えへんオチかい。  
と調べてたら、こっちにするべきだったみたい。  
  
```
SQL> drop tablespace <tablespace_name> including contents and datafiles;
```
  
これじゃないとファイルも一緒に削除してくれないとな。  
もー先に教えてよー。時既に遅しじゃーん。まぁ今日は寝よ。  
ってことで次の日CloudWatchのグラフ見たらびっくり。  
半日遅れでディスクの空き増えてるよ。  
  
![CloudWatch ディスク空きグラフ](http://blog.branch4.pw/images/2014/06/CloudWatch_storagefreespace_edit.png)
  
なんでやねん！  
  
答えは単純でOracleのマイナーバージョンアップをONにしてると  
指定した時間にパッチをあててアップデートしてくれてるんですな。  
そのときに再起動がかかってると。  
そのマイナーバージョンアップの指定してる時間を見ると、  
  
![Minor VersionUP Widdow](http://blog.branch4.pw/images/2014/06/minorvirsionup_widdow.png)
  
UTCでAM1:00-1:30だから+9時間で朝の10時半ごろ、、、  
グラフみるとビンゴ！！  
そうゆうことですか、、、  
  
なんだかなぁ。すっきりせんなぁ。  
  
##あとがき
ちなみに上で書いてるのはテストDBでやったこと。    
事情があってなかなかこの作業させてもらえなくてねぇ。  
だから本番でやるときはちゃんとincluding contents and datafilesもつけて  
ファイル削除してやろうと思うんです。  
その結果はまた続きとして書く予定です。  
  
ではでは、今日はこんなところで。  
