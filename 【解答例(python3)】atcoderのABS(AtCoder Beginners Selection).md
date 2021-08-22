---
title: 【解答例(python3)】atcoderのABS(AtCoder Beginners Selection)
tags: AtCoder Python algorithm
author: yumenomatayume
slide: false
---
# はじめに
最近、競技プログラミングをやっていなかったので、
頭の体操程度に簡単な問題を解こうと思いました。

以前から[AtCoder](https://atcoder.jp/)のABC(AtCoder Beginner Contest)は解いていましたが、
そういえば[ABS](https://atcoder.jp/contests/abs)(AtCoder Beginners Selection)というAtCoder入門者向けの問題集をやっていなかったので、これを機に全10問解いてみました。

※言語は全てpython3系です。

## PracticeA: Welcome to AtCoder
問題文の通りに解答すれば良し！簡単な入出力処理が出来れば問題ありません。

```python
a = int(input())
b,c = map(int, input().split())
s = str(input())
 
print(a+b+c, s)
```

## ABC086A: Product
こちらも同様、問題文通りに解答します。

```python
a,b = map(int, input().split())
 
sum = a*b
if sum % 2 ==0:
    print("Even")
else:
    print("Odd")
```

## ABC081A: Placing Marbles
同様に、問題文通りに解答します。

```python
s = str(input())
 
print(s.count('1'))
```

## ABC081B: Shift only
ABCのB問題に突入しました。この辺りからすこーしだけ頭を使います。
問題文から何を求めればいいか読み解くと、
各$A_1, \dots, A_N$を素因数分解して、その中で最小の2の冪数を求めれば良いことになります。
[こちら](https://note.nkmk.me/python-prime-factorization/)を参考にして実装しました。

```python
import collections
 
 
def prime_factorize(n):
    a = []
    while n % 2 == 0:
        a.append(2)
        n //= 2
    f = 3
    while f * f <= n:
        if n % f == 0:
            a.append(f)
            n //= f
        else:
            f += 2
    if n != 1:
        a.append(n)
    return a
 
n = int(input())
l = list(map(int, input().split()))
 
prime_list = [ dict(collections.Counter(prime_factorize(i)).items()) for i in l ]
try:
    count_2_list = [ d[2] for d in prime_list ]
    print(min(count_2_list))
except KeyError:
    print(0)
```

コードが長くなってしました。
２の冪乗を求めるので、2進数に変換して処理した方がスマートでしたね。
参考）https://qiita.com/watyanabe164/items/5c26fd9d8d244c5f2483

## ABC087B: Coins
A,B,Cが50以下でしたので、単純に全探索しました。

```python
def equal_x_count(p,q,r,z):
    count = 0
    for i in range(p+1):
        for j in range(q+1):
            for k in range(r+1):
                if 500*i + 100*j + 50*k == z:
                   count += 1
    return count
 
 
a = int(input())
b = int(input())
c = int(input())
x = int(input())
 
print(equal_x_count(a,b,c,x))
```

## ABC083B: Some Sums
数値型で代入したnを文字列型に変換して、
各桁ごとに取り出した文字列を再度数値化して処理しました。

```python
n, a, b = map(int, input().split())
 
count = 0
for i in range(n+1):
    sum_each_digit = sum([ int(j) for j in str(i)])
    if a <= sum_each_digit and sum_each_digit <= b:
        count += i
 
print(count)
```

## ABC088B: Card Game for Two
入力をリストに代入してリストを大きい順にソートし
偶数番目を加算(0,2,4番目)、奇数番目を減算(1,3,5番目)

```python3
n = int(input())
a = list(map(int, input().split()))
 
sorted_a = sorted(a, reverse=True)
alice_list = []
bob_list = []
 
for i in range(n):
    if i % 2 == 0:
        alice_list.append(sorted_a[i])
    else:
        bob_list.append(sorted_a[i])
 
d = sum(alice_list) - sum(bob_list)
 
print(d)
```

## ABC085B: Kagami Mochi
リストで取得したものを集合に変換して、要素数を数えれば簡単ですね。

```python
n = int(input())
d = [int(input()) for _ in range(n)]
 
print(len(set(d)))
```

## ABC085C: Otoshidama
全探索で探していけばいいだけですね。

```python
n, y = map(int, input().split())
 
for i in range(n+1):
    for j in range(n-i+1):
        if 10000*i + 5000*j + 1000*(n-i-j) == y:
            print(i,j,n-i-j)
            exit()
print(-1,-1,-1)
```

## ABC049C: 白昼夢
入力Sの末尾に注目し、一致している箇所があればその末尾を削り、それをSを置く。
これを繰り返す。

```python
s = str(input())
 
while True:
    if s[-5:] == 'dream' or s[-5:] == 'erase':
        s = s[:-5]
    elif s[-6:] == 'eraser':
        s = s[:-6]
    elif s[-7:] == 'dreamer':
        s = s[:-7]
    elif s == '':
        print('YES')
        break
    else:
        print('NO')
        break
```

## ABC086C: Traveling
時刻$t_nに座標($x,$y)にいた場合、時刻$t_(n+1)で座標($x',$y')に移動出来るかどうかを、
1 <= t <= n　の場合で考えてみます。

- 座標($x',$y')に移動出来るまで十分な時刻があるか
- 移動出来る最短の時刻は、移動時間との奇偶が同一であるか
- ２回分の時刻を消費すれば元の位置に戻る

問題文から上記の前提条件を読み解くと、以下のコードで実装できます。

```python
import numpy as np
import sys
 
n = int(input())
l = [ list(map(int, input().split())) for _ in range(n)]
 
l.insert(0, [0,0,0])
 
for i in range(n):
    d = np.array(l[i+1]) - np.array(l[i])
    time, move = d[0], abs(d[1])+abs(d[2])
    if move > time or (time - move) % 2 != 0:
        print('No')
        sys.exit()
 
print('Yes')
```

AtCoderにnumpyが実装されているおかげで、割と簡単に実装できました。

# 最後に
ABSはアルゴリズム的な知識がなくても十分解答することが出来ます。
以上述べたものは解答例なので、もっとスマートにコーディングしたい方は他の模範解答を調べてみてください。

