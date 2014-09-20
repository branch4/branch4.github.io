---
layout: post
title: DynamoDBの本番とテスト環境の分け方
date: 2014-08-31 01:30:00 +0900
comments: true
published: false
author: risterlab
categories: 
 - Oracle
 - SQL
---
  
はい、こにゃにゃちは。[risterlab](http://diary.risterlab.com)です。  
11月にイタリアに行くことになって、今からでもワインを飲めるようになりたーい。  
  
本題はOracleで実行時間の長いSQLをNagiosで監視したので、それについて。  
だいたい１時間以上実行してるSQLなんてろくなことないですよね。  
何かあってから犯人探しをするよりは、事前に怪しいやつを押さえておきたい。そんな人は読んでください。   
![long-elapsed-sql-oracle](http://blog.branch4.pw/images/2014/09/long_elapsed_sql_oracle.jpg)  
photo credit: <a href="https://www.flickr.com/photos/fun_flying/246252433/">Armchair Aviator</a> via <a href="http://photopin.com">photopin</a> <a href="http://creativecommons.org/licenses/by/2.0/">cc</a>  
  
<!-- more --> 
### 実行時間の長いSQLを見つける方法  
----------
  
下記SQLを打てばOK。  

```
select
b.elapsed_time/b.executions,a.sid,a.serial#,a.sql_hash_value,a.machine,a.username,
b.elapsed_time, b.executions,b.disk_reads,b.cpu_time,b.sql_text
from v$session a,v$sql b
where a.sql_address = b.address
and a.sql_hash_value = b.hash_value
and a.status = 'ACTIVE'
and b.elapsed_time/b.executions >= 3600000000
and b.executions != 0
order by b.elapsed_time desc;
```  

意味は  

1. a.status = 'ACTIVE' 
   - 実行中であること  
1. b.elapsed_time/b.executions >= 3600000000
   - 実行時間を実行回数で割った時間が１時間以上のもの  
1. b.executions != 0
   - 実行回数が０回じゃないもの。
   - ０回ということは初めての実行ということで2.の割り算が失敗してエラーになる為除外。  

v$sessionとjoinしてる理由は  
実行しているmachine,usernameがわからないと  
実際どのアプリが実行しているか判断しづらくて対応時に困るため。  
sidやserial#も出しているのは  
いざこのSQLのセッションをkillしようとした時に必要だからです。  

### Nagiosから監視   
----------
  
何か障害が起きた時に上記SQLで怪しいSQLを探すのもありですが、  
障害になる前に危ないSQLはつぶしておきたいところ。  
なので一定時間以上実行されているSQLを検知してメールを送りましょう、ってのをNagiosでやってみました。
DBに置くnrpeで実行するスクリプトはこんな感じ

```
#!/bin/sh

OK=0
CRITICAL=2
UNKNOWN=3
STATUS=${UNKNOWN}

LOGFILE=/usr/local/nagios/log/check_long_elapsed_time.log

error_check(){
	RC=`echo $?`
	if [ $RC != 0 ];then
		STATUS=${CRITICAL}
	fi
}

SQL_NUM="101"
SQLDIR="/usr/local/nagios/sqls"
ORACLE_USER="user"
ORACLE_PASS='password'
ORACLE_HOST="172.20.xxx.xxx"
ORACLE_PORT="1521"
ORACLE_SID="SIDxxx"

sqlplus ${ORACLE_USER}/${ORACLE_PASS}@${ORACLE_HOST}:${ORACLE_PORT}/${ORACLE_SID} @${SQLDIR}/${SQL_NUM}.sql 1>/dev/null
error_check ${SQL_NUM}

if [ ${STATUS} = 2 ];then
	echo "CRITICAL: check_long_elapsed_time.shが失敗しました。"
	exit ${STATUS}
fi

SQLCNT=`cat ${LOGFILE} | wc -l`
LONG_SQLS_FILE="/usr/local/nagios/log/long_sqls.log"
if [ ${SQLCNT} -ge  1 ];then
	echo "WARNING: 1時間以上実行しているSQLが${SQLCNT}つあります"
	echo "[`date`] 実行時間の長いSQL情報"		 		      > ${LONG_SQLS_FILE}
	echo "-----------------------------------------------------"         >> ${LONG_SQLS_FILE}
	while read line;do
		echo "SID = `echo $line | awk '{print $2}'`"                 >> ${LONG_SQLS_FILE}
		echo "SERIAL# = `echo $line | awk '{print $3}'`"             >> ${LONG_SQLS_FILE}
		echo "SQL_HASH_VALUE = `echo $line | awk '{print $4}'`"      >> ${LONG_SQLS_FILE}
		echo "MACHINE = `echo $line | awk '{print $5}'`"             >> ${LONG_SQLS_FILE}
		echo "USERNAME = `echo $line | awk '{print $6}'`"            >> ${LONG_SQLS_FILE}
		echo "合計実行時間 = `echo $line | awk '{print $7}'`(μs)"    >> ${LONG_SQLS_FILE}
		echo "実行回数 = `echo $line | awk '{print $8}'`"            >> ${LONG_SQLS_FILE}
		echo "DISK_READS = `echo $line | awk '{print $9}'`"          >> ${LONG_SQLS_FILE}
		echo "CPU_TIME = `echo $line | awk '{print $10}'`"           >> ${LONG_SQLS_FILE}
		echo "SQL_TEXT = `echo $line | awk '{$1="";$2="";$3="";$4="";$5="";$6="";$7="";$8="";$9="";$10="";print}' | sed -e 's/^          //g'`"	     >> ${LONG_SQLS_FILE}
		echo "-----------------------------------------------------" >> ${LONG_SQLS_FILE}
	done < ${LOGFILE}
	cat ${LONG_SQLS_FILE}
	exit ${CRITICAL}
fi

exit ${STATUS}
```
  
まぁ適当にこんな感じでさっきのSQLにひっかかったものがあれば  
整形してメールに流すようにする。  
あとはnagiosサーバーから１時間に１回実行すればOK.  
これでおばかなSQLは事前に発見できる、はず。  
   
### まとめ  
----------
  
つまりは、  
実行時間の長いSQLを定期的に見て、チューニングしていきましょう。  
ってことです。  
  
