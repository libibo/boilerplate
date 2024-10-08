---
title: 暗記するべき１０の必須コマンド
date: 2019-10-08
tags: ["サンプル", "チュートリアル"]
excerpt: これは抜粋です。
---

「ファイル処理系」のコマンド：

（１）ファイルの一括処理：　for 文  
（２）ファイルを検索：　dir  
（３）ファイル内を検索：　findstr  
（４）ファイルの差分を検出：　fc  
（５）フォルダ構造を表示：　tree  

「ネットワーク系」のコマンド：

- （６）ポート番号とプロセス名の対応を知りたい：　netstat 
    - (a) netstat自体にやらせる方法
    - (b) タスクマネージャを見る方法
    - (c) tasklist コマンドを使う方法
- （７）自分のIPアドレスを知りたい：　ipconfig
- （８）サーバが生きているか調査：　ping
- （９）LANで違うセグメントにアクセスしたい：　route
- （１０）IPアドレスとドメイン名を相互変換：　nslookup

## （１）ファイルの一括処理：for文

このフォルダにある全テキストファイルを一斉に開きたい

~~~dos
for %V in ( *.txt ) do %V
~~~
解説：  
- フォルダ中で，正規表現でファイルパターンが *.txt にマッチする全ファイルを検出する。  
- 検出結果のファイル名は %V なる変数に代入される。  
- 全マッチ結果について，do 以下のコマンドが実行される。  
- ファイル名 %V を単独で実行すると，そのファイルがデフォルトアプリケーションで開かれる。  

このフォルダにある全SQLファイルを一斉に実行したい

~~~dos
for %V in ( *.sql ) do psql -d DBNAME -U USERNAME -f %V
~~~

これはPostgreSQLの例。  
パスワードは複数回入力する必要があるが，実行したいSQLは自動的に，順番に選択・実行される。  
データベース名やユーザ名を１ファイルごとに入力し直さないで済む。  

さらに，このような使い方もある。  

このフォルダ以下にある全JPGファイルを一つのフォルダ上にコピーしたい  

~~~dos
for /R %V in (*.jpg) do copy /Y "%V" d:\img\
~~~


このフォルダにある全テキストファイルを一つのファイルに結合したい

~~~dos
for %V in ( *.txt ) do copy temp.txt + %V temp.txt
~~~

解説：  
- copy /Y A B というコマンドは，同名のファイルは上書きしてコピーする事を指す。
- copy A + B C というコマンドは，AとBを結合してCの名で保存する事を指す。
- フォルダから全テキストファイルを抽出し，それぞれ %V に格納
- %V と temp.txt を毎回結合する。

前者の例では，カレントディレクトリ以下の全てのフォルダからJPGファイルを探索し，コピーする。  
同名のファイルを上書きせずに，フォルダ構造を保ってコピーしたい場合は，  
~~~dos
xcopy /S /H *.jpg d:\img\　
~~~
とする。

後者のtemp.txt の中には，全テキストファイルの内容が順番に結合して保存される。  
（※多少，余計な情報が混入する可能性もあるので注意。）  
「ためておいた細切れのメモを，いっぺんに印刷したい！」というような時に役立つだろう。  

ちなみに，temp.txt は空のファイルとして前もって準備しておく必要がある。  
これもおまけでコマンド上で行なってしまおう。  

空のテキストファイルを作成する

~~~dos
echo:> temp.txt
~~~

この場合，改行が1つだけのテキストデータが作成される。  
なお上記で % の記号が出てきたが，バッチファイル中で同じ事をやりたい場合は，% を %% と表記するのを忘れないこと。  

## （２）ファイルを検索：dir
ファイル名をキーにしてファイルを再帰的に検索、フルパスを表示

~~~dos
dir /s /b *.txt

~~~
「再帰的に」というのがポイント。このフォルダ以下の全ての子フォルダ内も検索対象になる。  
オプションもそのまま覚えておきたい。  

## （３）ファイル内を検索：　findstr
ファイル内容の単語をキーにしてファイルの行を検索、ファイル名と行内容を表示  

「hoge」または「foo」という文字列を含むようなテキストファイル  

~~~dos
findstr /n /s "hoge foo" *.txt
~~~

"hoge foo"というつながりを含むようなテキストファイル

~~~dos
findstr /n /s /c:"hoge foo" *.txt
~~~

これも再帰的な検索。  
プロジェクトで修正待ちの箇所を一斉検出する際などに役立つ。  
ソースディレクトリ以下の全phpファイルから，TODO扱いになっている部分を抜き出し，ファイルに保管  

~~~dos
findstr /n /s "TODO" *.php > todo.txt
~~~

「>」の後に保存先ファイル名を書く。  
ファイルのフルパスと，行番号と，行全体が抽出されるので便利。  
TODO抽出のためのバッチとして，ソースディレクトリの最上部に設置しておくのもよいだろう。

（４）ファイルの差分を検出：fc
diffのように2つのファイルを比較し，相違を検出する。/b でバイナリ比較もできる。  

テキストファイルの差分を行数付きで表示

~~~dos
fc /n a.txt b.txt
~~~

## （５）フォルダ構造を表示：tree

ファイルをツリー構造で一覧表示  

~~~dos
tree /f
~~~

それなりにしっかりした木構造の図が出力される。フォルダ内のスナップショットとして利用できるだろう。  

## （６）ポート番号とプロセス名の対応を知りたい：　netstat
「このプログラムは，何番のポートを使ってるのか？」  
「このポートは，どのアプリケーションが占有してるのか？」  
というような時には，下記のコマンドで調べることになる。  

プロセスのポート利用状況を一覧する  

~~~dos
netstat -ano
~~~

すると，
- プロトコル種別
- ローカルアドレスとポート番号
- 通信先アドレスとポート番号
- 通信状態
- プロセス番号(PID)
が表示される。

次に，プロセスIDからアプリケーション名を特定する。これには３つの方法がある。

### (a) netstat自体にやらせる方法

もしも，上のコマンドでオプションに　-b を追加すれば，プロセスIDだけでなくプロセス名をも表示させる事ができる。  

~~~dos
netstat -abno
TCP 0.0.0.0:80 0.0.0.0:0 LISTENING 3212
[apache.exe]
・・・
~~~

のように，リッスンポートの作成に使われたexeファイル名が出てくる。  
この場合，80番ポートを使っているのが，PID=3212のapache.exeだとわかる。  
しかし，この操作は非常に重い。表示されるまで時間がかかる。  

### (b) タスクマネージャを見る方法
コマンドプロンプトを開いているのなら，コマンドライン上から taskmgr と入力するのが速い。  
メニューバーから，表示→列の選択→PID を選択すると，各プロセスごとにPIDが表示される。  

(c) tasklist コマンドを使う方法
これはXP Home Editionでは利用できないコマンドなのだが，使える環境ならば役立つ。  

プロセスIDからプロセス名を特定する  

~~~dos
tasklist /FI "PID eq 2780"
~~~

上のようにすると，2780番のPIDについてアプリケーション名が表示される。  

このようにしてポートの占有状況が分かれば，  
あとは競合状態にあるプログラムを終了させたり，  
あるいは使われていないポート番号を利用するように調整するだけだ。  

ところで，netstatコマンドで表示される情報は複数ページに渡る。  
必要な行だけを抽出して表示させたい時、findstrコマンドとパイプにすればよい。  

~~~dos
netstat -oan | findstr ":80"
~~~

こうすれば，80番ポートを利用しているアプリケーションの情報だけが，行単位で抽出表示される。  
注意：自分だけでなく，相手アプリケーションが80番を利用しているケースも含めて表示される。  


## （７）自分のIPアドレスを知りたい：ipconfig

自分のマシンやリモートログイン先のマシンが，もしDHCP支配下にある場合，IPアドレスは動的に割り当てられる。  
マシンのIPアドレスを知る方法は，下記のコマンドを適当なバッチに保存しておくこと。


自分のIPアドレスを表示する  

~~~dos
@ipconfig
@pause
~~~

pauseは，「続行するには何かキーを押してください . . .」というウェイト表示を出すためのコマンド。  
上記を my_ip.bat などの名前で保存してデスクトップ上に置いておき，ダブルクリックすれば，現在のIPアドレスがわかる。  
何かキーを押せば表示は終わる。恐らく一番手軽な方法。  

## （８）サーバが生きているか調査：　ping

あるサーバにネットワーク経由で接続できるか調査  

~~~dos
ping IPアドレス
~~~

応答がなければ，相手サーバがダウンしているか，途中のネットワーク設定がおかしいか，自分のマシンの設定がおかしいかのどれかだと予測できる。  

LAN上に3台のPCがあって，3台の応答を一括して調べたいとする。この場合，  

~~~dos
for %i in (1,2,3) do ping 192.168.0.%i
~~~

とすれば，  
192.168.0.1  
192.168.0.2  
192.168.0.3  
の3つのアドレスへ順番にpingを送ってくれる。これも必要ならばバッチにしておくと便利。  

なお，pingで相手サーバから応答がない場合は
~~~dos
 tracert IPアドレス 
~~~
と打ちこんでパケットの伝搬経路を確かめる。  
経路の途中で，しかもLAN内で応答が途切れた場合，route コマンドが役に立つかもしれない。

## （９）LANで違うセグメントにアクセスしたい：route

社内LAN上にDBサーバがあり，そこに接続したいという状況を考えよう。  

応答がない場合，例えば次のような原因が考えられる。  
（a）そのDBは「SQL Server 2005」なので，デフォルトでTCPアクセスが拒否される設定のままだった。

MSSQLに関連したこのトラブルはよくある。コマンドライン上では

~~~dos
データソース '〜' への接続を作成できません。
サーバーへの接続を確立中にエラーが発生しました。SQL Server 2005 に接続している場合、既定の設定では SQL Server によるリモート接続が許可されていないために、このエラーが発生した可能性があります。(プロバイダ: 名前付きパイプ プロバイダ、エラー: 40 - SQL Server への接続を開けませんでした)。
~~~

が出る。  
この場合，configuration managerから，外部からの接続を受け付けるように設定変更すればよい。  

こういった「相手サーバ側の問題」の他に，ルータ未設定のために接続できない事もある。  
（b）まだroute add をしていなかった。  

LAN上の異なるセグメントに接続する
自機が接続できるセグメント一覧表示

~~~dos
route print 
~~~

セグメント間を仲介できるように設定

~~~dos
route -p ADD 192.168.220.0(アクセスしたいセグメント)
MASK 255.255.255.0（ネットマスク）
192.168.〜(自分のいるセグメントのデフォルトゲートウェイのIP)
~~~

この設定を行なったのち再度 route print して，自分の接続したいセグメントがDestinationに含まれていればOK。  
-p はpermanentの略で，PCをシャットダウンした後も設定を持続させる事を意味する。printと混同しないこと。

## （１０）IPアドレスとドメイン名を相互変換：nslookup
IPアドレスの正引き，または逆引きの操作。  
Webサイトのアクセス解析を閲覧している際に，リファラがサイト名ではなくIPアドレスになっていて，どこのサイトなのか知りたいというような時に。  

IPアドレスの正引き・逆引き  

~~~dos
nslookup IPアドレスまたはドメイン名
~~~

補足して，ショートカットキーなど操作上の細かいTipsを挙げれば，  

キー系  
- コマンドプロンプトを開きたい際には，Windows + Rキーでcmdと入力すれば早い
- 長いファイル名入力はTABキーで補完
- 過去に入力したコマンドの履歴はF7キーで閲覧
- 入力中の文字は，Escキーで全削除して行頭に戻れる

コマンド系  
- ディレクトリ間の移動は cd だが，ドライブ間の移動だけは直接 D: のようにcdを付けないで入力
- copy con temp.txt で標準入力からテキストファイルを作れる（Ctrl+Zで終了）

