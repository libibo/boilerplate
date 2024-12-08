---
title: BATファイルの書き方・パターン集
date: 2019-10-08
tags: ["サンプル", "チュートリアル"]
excerpt: これは抜粋です。
---

## （０）BATプログラミングの特徴

BATのメリット：  
- ファイルシステムの一括処理に便利  
- プロセスの起動処理を簡素化できる（起動引数などをまとめて保管しておける）  

BATのデメリット：  
- 構造型プログラムが書けない，極めて書きづらい   
 →本エントリで，各種の構文を理解すべし。サブルーチンやfor文の使いようがキモ。  
- 複雑な処理は困難   
 →別のプログラムを呼び出して処理させるべし。cscriptでWSF・WSHを呼び出すのもよし。mshtaでワンライナーを記述するのもよし。  

BATでできない難しいことを，無理やりBATでコーディングする必要はない。  

BATの「得意分野」はBATでコーディングし，なおかつ，BATの「守備範囲外」の事柄であれば，BATから別のプログラムを呼び出せばよいのだ。  

BATはプログラムの起動が得意なのだから，多いに外部プログラムに頼って良いのだ。  
むしろ，「外部プログラムの呼び出し方」を，順番にうまく制御する役目を担うのがBATである。  

## （１）BATファイルの雛型

### （１−１）冒頭と末尾のテンプレート
BATの最初と最後に記述する内容は，だいたい決まっている。  
冒頭と末尾は，便利なので暗記してしまおう。  

~~~dos
@echo off

rem
rem このバッチの説明
rem

rem 設定事項
set HOGE="変数の値"

rem このバッチが存在するフォルダをカレントに
pushd %0\..
cls


〜処理〜


pause
exit
~~~

まず冒頭の説明。
- コマンドの先頭に@をつければ，バッチ実行時に，そのコマンド自体は表示されなくなる。
- echo offとすれば，実行しているコマンド自体の表示を全面的にOFFにできる。  
  →全ての行の先頭に「@」と書かなくて済む。
- remはコメント行。冒頭には，バッチの目的などを記載する。各行では，各行の処理内容を説明する。
- setは，環境変数の定義。フォルダのパスとかを定義する。あとから変更しやすいよう，バッチの先頭部分にまとめて記述する。
- pushdは，ディレクトリの移動。 
  - cdコマンドだと，異なるドライブに移動できない。
  - cd /dとすれば，異なるドライブに移動できるが，UNCパス上でネットワークの共有フォルダ上を移動できない。
  - pushdは，ドライブの違いや，ネットワークの違いを気にせず，必ず移動できる。
  - したがって，cdではなく，常にpushdコマンドを使うのが無難。なぜなら，このバッチがどこで実行されるのか，前もって知ることができないから。
- %0 は，このバッチファイルのファイルパス。 
  - %0\.. とすれば，このバッチファイルが存在するフォルダを指す。
  - したがって，バッチの存在ずるフォルダをカレントディレクトリにすることができる。
- clsは，画面の消去。前回のコマンドの実行結果などを削除して，まっさらな画面でバッチの実行を開始できる。

ここまでで１セットである，と考えよう。

次に，バッチの末尾の説明。  

- pauseコマンドは，何かキーを押すまで待機する。バッチの終了を目視でよく確認した上で，次に進む事ができる。 
  - 突然ウィンドウが消えてしまうと，処理が成功したのか，失敗したのか，知ることができないから。
- exitでバッチの終了。

この雛型を利用したバッチのサンプル：  

bat中でforループをネストし，サブルーチンを呼び出して，条件付きファイル検索の結果を一斉コピーしよう　（ファイル名の重複防止機能付き）  

下記のような要望がある。
- 特定のフォルダツリーの中から，batファイルで，Excelファイルを抽出したい。
- サブディレクトリのフォルダ名は，スペースを含む場合がある。
- 抽出対象のExcelファイルは，ファイル名の先頭に特定の「先頭ID」が付与されている。その先頭IDはリストにしてある。（＝条件付きファイル検索）
- 抽出後に，ファイル名がダブる場合もあり得るので，上書きせず別ファイルとなるように保存したい。

この要望に応えるようなbatファイルをコーディングすると，  
下記のようなプログラミング・テクニックを一気に使う事になる。  
- コマンドの実行結果を行ごとに引数に取るようなforループ。（bashでよくあるパターン）
- forループのネスト制御構造。
- 引数を渡したサブルーチンの呼び出し。
- 変数のローカル化。
- ファイルの存在判定のif文。
- ループ制御構造にカウンタを導入し，条件を満たすまでの間，整数を加算する。

~~~dos
@echo off

rem ファイルを一斉コピーするバッチ
rem 
rem ・コピー元のフォルダ内から，特定の拡張子のファイルを再帰的に検索。
rem ・コピー対象のファイル名の先頭に付与されているIDは，リストとして存在。
rem ・コピー先のファイル名が重複した場合，ファイル名の末尾に数値を付与して別名保存。
rem 
rem coded for the answer of: http://q.hatena.ne.jp/1319868767


rem 変数定義
set IDLIST=移動対象の先頭IDのリスト.txt
set FROM_DIR="コピー元"
set TO_DIR="コピー先"
set EXCEL_FILE_EXT="xls"

rem このバッチが存在するフォルダをカレントに
pushd %0\..

rem カレントを変数に保持
for /F "usebackq" %%i in (`cd`) do (
  set BAT_DIR="%%i"
)

rem 開始
echo 「%FROM_DIR%」内のファイルを全チェックします。
pushd "%FROM_DIR%"

rem 先頭IDごとに
for /f "usebackq" %%n in (`type %BAT_DIR%\%IDLIST%`) do (
  echo --------- 先頭IDが「%%n」であるようなファイルを移動します。 --------- 
  
  rem 発見したファイルごとに
  for /f "usebackq delims=" %%m in (`dir /s /b %%n*.%EXCEL_FILE_EXT%`) do (
    echo 移動対象ファイル「%%m」を発見しました。コピーします。
    
    rem ※「%%~nxm」は，拡張子込みのファイル名を表す。
    rem   http://fpcu.on.coocan.jp/dosvcmd/batch.htm#param
    
    rem コピー実行用のサブルーチンにファイル名と拡張子などを渡す
    call :ROUTINE_COPY_ONE_FILE  "%%m"  "%%~nm"  "%%~xm"  %BAT_DIR%  %TO_DIR%
    
      rem ※ネストしたforループの内側でgotoするとエラーになる。要サブルーチン
      rem   http://fpcu.on.coocan.jp/dosvcmd/bbs/log/cat3/for_in_do/4-1324.html
  )
)

rem 終了
echo --------- 全コピーが完了しました。コピー結果： --------- 
dir /b %BAT_DIR%\%TO_DIR%
echo -------------------------------------------------------- 

pause
exit



rem --------------------- サブルーチン ---------------------


rem コピー実行用のルーチン
:ROUTINE_COPY_ONE_FILE

  rem ルーチンの引数を取得
  setlocal
  set FILE_FULLPATH=%1
  set FILE_BASENAME=%2
  set FILE_EXT=%3
  set BAT_DIR=%4
  set TO_DIR=%5
    rem ※サブルーチン中でsetlocalすれば，
    rem   呼び出し元に影響なくローカル変数を使える
    rem   http://d.hatena.ne.jp/iroiro123/20110331/1301567381

  echo ---「%FILE_FULLPATH%」のコピー処理開始：

  rem 通常のコピー実行
  if exist %BAT_DIR%\%TO_DIR%\%FILE_BASENAME%%FILE_EXT% (
    echo 既にコピー済みです。コピー先ファイル名を変えて再試行します・・・
    goto SETUP_COPY_BY_NEW_NAME
  ) else (
    echo 未コピーです。
    copy %FILE_FULLPATH% %BAT_DIR%\%TO_DIR%\
    goto END_OF_ONE_COPY
  )
    rem ※複数行に渡るIF文の書き方
    rem   http://fpcu.on.coocan.jp/dosvcmd/bbs/log/cat3/if/4-0906.html


rem 重複するファイル名が存在する場合の初期化
:SETUP_COPY_BY_NEW_NAME

  rem 重複するファイルの末尾につける数値
  set /a DUP_FILE_COUNT=2
    rem ※算術変数は遅延展開され加算が可能
    rem   http://fpcu.on.coocan.jp/dosvcmd/bbs/log/cat3/4-0873.html
  
  goto BEGIN_COPY_BY_NEW_NAME


rem 重複を避けて新規ファイル名を生成する
:CREATE_NEW_NAME_BY_INCREMENT
    
  rem echo 数値をインクリメントします。
  set /A DUP_FILE_COUNT=%DUP_FILE_COUNT%+1


rem 新規ファイル名でコピーを試みる
:BEGIN_COPY_BY_NEW_NAME
  
  if exist %BAT_DIR%\%TO_DIR%\%FILE_BASENAME%%DUP_FILE_COUNT%%FILE_EXT% (
    echo %DUP_FILE_COUNT%回目：失敗。ファイルが存在するため，再試行します・・・
    
    rem 名前をつけ直し
    goto CREATE_NEW_NAME_BY_INCREMENT
  ) else (
    echo %DUP_FILE_COUNT%個目は未コピーです。
    
    rem コピー実行
    rem echo 新規ファイル名：%BAT_DIR%\%TO_DIR%\%FILE_BASENAME%%DUP_FILE_COUNT%%FILE_EXT%
    copy %FILE_FULLPATH% %BAT_DIR%\%TO_DIR%\%FILE_BASENAME%%DUP_FILE_COUNT%%FILE_EXT%
    goto END_OF_ONE_COPY
  )


rem ファイル名がどうなったにせよ，コピーは終了した
:END_OF_ONE_COPY
  echo ---「%FILE_FULLPATH%」のコピー処理終了。

exit /b
  rem サブルーチンからの脱出時には値も返せる
  rem http://logicalerror.seesaa.net/article/125905911.html
~~~

上記プログラムはEXCEL_FILE_EXT変数の値でExcelファイルの拡張子を変更できる。  
テスト時にはxlsではなくxlsxとして動作確認している。  

事前の状態：  

~~~dos
D:\temp\battest>tree /f
D:.
│  移動.bat
│  移動対象の先頭IDのリスト.txt
│
├─コピー元
│  │  hoge.xlsx
│  │  移動対象1_fuga.xlsx
│  │
│  ├─a
│  │      fuga.xlsx
│  │      移動対象1_fuga.xlsx
│  │      移動対象2_hoge.xlsx
│  │
│  └─スペースの テスト
│          fuga.xlsx
│          移動対象1_fuga.xlsx
│          移動対象2_hoge2.xlsx
│
└─コピー先


D:\temp\battest>type 移動対象の先頭IDのリスト.txt
移動対象1
移動対象2
~~~

もし，もう一度実行すると，番号がかぶらないように別ファイルとしてもう一度まとめてコピーし直される。  

