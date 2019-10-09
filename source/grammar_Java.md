---
title: Javaの文法
date: 2019-10-09
tags: ["サンプル", "チュートリアル"]
excerpt: これは抜粋です。
---

## ブロック

ブロック内には、複数の文を記述することができます。変数の有効範囲（「スコープ」という）は、そのブロック内のみとなります。  

~~~java
public static void main(String[] args) {
    {
        // ブロック内で変数vを宣言
        int v = 31;
        // ブロック内では参照可能
        System.out.println("v = " + v);
    }
    // ブロックを抜けたため、変数vはここでは参照できない
    // System.out.println("v = " + v);
    // メソッド内のみで有効な変数xを宣言
    int x = 0;
    // for文のブロック内でのみ有効な変数yを宣言
    for (int y = 0; y < 2; y++) {
        // for文のブロック内でのみ有効な変数zを宣言
        int z = 0;
        System.out.println(
        "x = " + x++ + ", y = " + y + ", z = " + z++);
    }
    System.out.println("x = " + x); // 参照可
    // for文のブロックを抜けたため、変数y、変数zはここでは参照できない
    // System.out.println("y = " + y);
    // System.out.println("z = " + z);
}
~~~
ブロック内に記述できるのは、「文」と「局所変数宣言文」と「クラス宣言」だけです。  
処理上、ブロックは通常の文とみなされます。そのため、「for」文や「if」文などでよく使われます。  
メソッド内に「for」文や「if」文などと関係のないブロックを記述することも可能です。クラス宣言部の中、メソッドの外にブロックを記述すると初期化ブロックとなります。

## 識別子

コード中にクラス、メソッド、変数、パッケージ、ラベルなどを定義していきます。これらの名前を「識別子」といいます。
文字列をコンソールに表示する
~~~java
System.out.println（"Hello"）;
~~~

- 「System」はクラス名
- 「out」はフィールド名（変数名）
- 「println」はメソッド名  
それぞれ識別子です。

識別子には、次の文字が使えます。
- 半角アルファベットの大文字・小文字
- 半角数字
- 「$」「_ 」
- 読点などの一部例外を除くUnicode文字（全角文字）

一般的にメソッド名、変数名は小文字+数字で単語の境目の部分だけを大文字にします。  
例「getCode」「familyName」など。  

クラス名は、それに加えて先頭の文字を大文字にします。  
例「TranslateText」など。  

ラベルの場合は、すべて大文字で定義します。  
例「OUTER」など。  

パッケージ名は、すべて小文字で定義します。  
例「jp.sample.test」など。  

## 変数

値や、インスタンスを入れる箱のことを「変数」といいます。  
値を入れる変数のことをプリミティブ型変数（基本型変数）、インスタンスを入れる変数のことを参照型変数といいます。  
どちらの場合も「型名 変数名」の形で宣言します。  

~~~java
int val; // プリミティブ型
String str; // 参照型
~~~

## プリミティブ型変数

プリミティブ型とはデータだけを持つ型で、インスタンスではありません。  
次の8つがJavaのプリミティブ型のすべてです。  

- boolean
- char
- byte
- short
- int
- long
- float
- double

## 参照型変数

「インスタンスを入れる変数」の表現は正確ではありません。  
実際には変数内にインスタンスが入っているわけではなく、メモリのどこかにあるインスタンスの場所（インスタンスの参照）を格納している変数です。 
参照型変数の型は、Javaで最初から用意されているクラス、Webで公開されているクラス、これから自分で作るクラスなど膨大な数になります。  

## 変数修飾子

変数の前には、次の修飾子を付けることができます。  
- アクセス指定子
- final
- volatile
- transient
- static

「final」以外の修飾子は、フィールドでのみ使うことができます。  
「final」を付けて宣言した変数には、最初に値を代入した後は他の値を代入することはできません。  

~~~java
final StringBuilder buf = new StringBuilder();
// インスタンス内のデータの変更は可能
buf.append("ABC");
// 他のインスタンスを代入することはできない
// コンパイルエラーになる
buf = new StringBuilder();
~~~

## 代入

宣言した変数には、値を代入することができます。宣言と同時に代入することも可能です。  
一度、宣言したら、基本的にその変数にはその型の値、または、インスタンスしか代入することはできません。  
演算子「=」の左辺にこれから値を設定する変数、右辺に代入する値もしくは変数を記述します。  
左辺にリテラルデータを記述することはできません。  

~~~java
// int型に整数はそのまま代入できる
// 宣言と同時に代入も可能
int val = 123;
// int型に真偽型を代入することはできない
// コンパイルエラーになる
val = false;
~~~

プリミティブ型の場合、変数の中に値を直接、格納します。
~~~java
int val = 780;
char ch = 'あ';
~~~

参照型変数へ代入するときには、リテラルではなく、インスタンスを作成する必要があります。  
ただし、String型やClass型のように、リテラルを使って代入することができるクラスもあります。  

~~~java
// staticメソッドを使ってインスタンスを作成
Integer val = Integer.valueOf(398);
// new演算子を使ってインスタンスを作成
StringBuilder buf = new StringBuilder();
// 文字列リテラルによる初期化
String str = "ABCD";
~~~
変数に格納されるのはインスタンスそのものではなく、インスタンスの場所です。  
もし、次のようなコードを実行した場合、変数「buf1」と「buf2」は同じインスタンスを指すことになります。  

~~~java
StringBuilder buf1 = new StringBuilder();
StringBuilder buf2 = buf1;
~~~

そのため、一方の変数にだけ変更を加えたつもりでも、他方の変数にも影響を与えてしまうので気を付けてください。  
次のコードを実行すると、「ABCXYZ」と表示されてしまいます。  
これは、参照型の変数の中身は他の変数への参照値が入っていて、参照型の変数を代入することはその参照値をコピーするためです。  

~~~java
StringBuilder buf1 = new StringBuilder();
StringBuilder buf2 = buf1;
buf1.append("ABC");
buf2.append("XYZ");
System.out.println(buf1.toString());
~~~

## null

参照型の変数は、インスタンスのある場所を示す変数です。  
変数を宣言しただけでインスタンスを代入しなかった場合は、何もないことを示す特別な値「null」が入っています。  
宣言しただけで初期化（何らかのインスタンスの代入）を行っていない変数を使おうとすると、コンパイルエラーが発生します。  

~~~java
String str;
// 初期化していない変数strを使おうとしているため
// コンパイルエラーになる
System.out.println(str.substring(1, 3));
~~~

「null」は、すべての参照型変数に代入可能です。

~~~java
String str = null;
Integer val = null;
Object obj = null;
~~~

「null」は何もないことを表すので、「null」が代入された変数を使うと「NullPointerException」という例外が発生します。

~~~java
String str = null;
// NullPointerExceptionが発生する
System.out.println(str.substring(1, 3));
~~~

ただし、staticフィールドとstaticメソッドはクラスに紐付くので、例外は発生しません。

~~~java
// 以下は正常に動作する
Integer var = null;
System.out.println(var.valueOf(100));
System.out.println(var.MAX_VALUE);
~~~

## 異なる型への代入

プリミティブ型変数の場合は、宣言した変数の型が代入する値の型より広い型であれば代入することができます。  
広い方から順に、double型、float型、long型、int型、short型、byte型となります。  
ただし、int型の値をfloat型に、long型の値をfloat型やdouble型に代入すると桁落ちする可能性があります。  
また、char型は文字型ですが、実際は、その文字のUnicodeの文字コードの数値（32bit）を保持しているため、int型と互換があります。  

~~~java
// 広い型への代入
byte b = 3;
short s = b;
int i = s;
long l = i;
float f = i;
double d = f;
// char型をint型に代入
char ch = 'A';
i = ch;
~~~
参照型変数の場合は、その型のクラスのサブクラスや、型のインタフェースの実装クラスのインスタンスを代入することができます。

SuperClass.java

~~~java
// スーパークラス
public class SuperClass {
}
~~~
Interface.java

~~~java
// インタフェース
public interface Interface {
}
~~~

SubClass.java
~~~java
// SuperClassのサブクラス
// Interfaceの実装クラス
public class SubClass extends SuperClass implements Interface {
	public static void main(String[] args) {
		// スーパークラス型の変数にサブクラスのインスタンスを代入
		SuperClass obj1 = new SubClass();
		// インタフェース型の変数に実装クラスのインスタンスを代入
		Interface obj2 = new SubClass();
	}
}
~~~

## プリミティブ型のキャスト

プリミティブ型では、狭い型から広い型に代入できる。

~~~java
byte b = 100;
int i = b; // 狭い型の変数を広い型に代入するのはOK
~~~
それとは逆に、広い型から狭い型への代入を行いたい場合は、キャストを使います。  
キャストは、値の前に変換したい型をカッコで囲んで記述します。  

~~~java
int i = 100;
byte b = (byte) i; // int型をbyte型に変換(キャスト)する
~~~

狭い型から広い型への変換は問題ないのですが、その逆は型の範囲を超える値が代入される可能性があります。  
その場合は桁あふれを起こしてしまい、エラーにはなりません。  
次のコードでは、byte型の範囲を超える値を代入しているため、桁あふれが起きます。  

~~~java
int i = 200;
byte b = (byte) i; // int型をbyte型に変換(キャスト)する
System.out.println(b); // 「-56」と表示される
~~~

小数型を整数型に変換すると、端数が切り捨てられます。

~~~java
int i = (int) 4.99;
System.out.println(i); // 「4」と表示される
i = (int) -4.99;
System.out.println(i); // 「-4」と表示される
~~~

また、boolean型とint型のように互換のない型の間でのキャストはできません。

~~~java
int i = (int) true; // コンパイルエラーになる
~~~

オートボクシングを使った広い型から狭い型へのキャストもできません。

~~~java
byte b = (byte) Integer.valueOf(9); // コンパイルエラーになる
int i = Byte.valueOf((byte) 9); // こちらはOK
~~~

## 参照型のキャスト

参照型のキャストも基本的には同じです。スーパークラスからサブクラスへのキャストが可能です。  
まったく関係のないクラスへのキャストはできません。

~~~java
// SuperClassのサブクラス
public class SubClass extends SuperClass {
	public static void main(String[] args) {
		// スーパークラスからサブクラスへのキャストはOK
		SubClass obj1 = (SubClass) new SuperClass();
		// 全く関係の無いクラスへのキャストはコンパイルエラーになる
		SubClass obj2 = (SubClass) Integer.valueOf(9);
	}
}
~~~

## 「ClassCastException」例外

キャストが明らかに不正な場合はコンパイルエラーとなりますが、コンパイル時に判断が付かないようなケースではプログラムの実行時に「ClassCastException」例外が発生します。  
どうしてもキャストを使用しなければならないケースもありますが、できるだけジェネリックスを使うなど、キャストを使わないほうが思わぬミスの防止になります。

~~~java
Object x = new Integer(0);
System.out.println((String)x); // ClassCastExceptionが発生する
~~~
