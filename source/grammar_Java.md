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