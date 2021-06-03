## Omotesando.rb #63

- - -

## 月単位でイテレーションする

---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* [趣味で気になった bugs.ruby のチケットをブログにまとめてる](https://secret-garden.hatenablog.com/archive/category/bugs.ruby)
* [令和時代の Ruby 基礎文法最速マスター](https://secret-garden.hatenablog.com/entry/2020/12/01/232816) 書いたので気になる人は見てね！

---

## 月単位でイテレーションする

---

#### 背景
- - -

* 特定の Date ~ Date 間で日、月、年の日付を取得したかった
* 日、年の日付はシュッと取得できた
* 月はちょっと難しかった
* 今回はどうやって月単位でイテレーションする事ができたのかの思考を追ってみる話

---

#### 前提条件
- - -

* 日付は1日で固定
  * `2020/04/01` みたに1日で固定する
  * `2020/04/14` みたいな日付は考慮しない
* 年をまたぐ可能性もあるよ
  * `2020/04/01 ~ 2022/04/01` 間
* 次の月は `Date#next_month` で取得できる
    ```ruby
    require "date"
    puts Date.parse("2020/01/01").next_month
    # => 2020-02-01
    ```

---

#### Range#step でできないか考えてみる
- - -

* Date は Range として扱うことができる
* [Range#step]() で特定の日単位でイテレーションできる
    * ただし、月の日数は一定ではないのでこれで対応するのは難しい

```ruby
require "date"

first = Date.parse("2020/01/01")
last  = Date.parse("2020/01/04")
p (first..last).map(&:to_s)
# => ["2020-01-01", "2020-01-02", "2020-01-03", "2020-01-04"]

# 10日ごとにイテレーションする
first = Date.parse("2020/01/01")
last  = Date.parse("2020/02/01")
p (first..last).step(10).map(&:to_s)
# => ["2020-01-01", "2020-01-11", "2020-01-21", "2020-01-31"]
```

---


#### Enumerator.new で自前でイテレーションする
- - -

* [Enumerator.new](https://docs.ruby-lang.org/ja/latest/method/Enumerator/s/new.html) で任意のイテレーションを作る事ができる

```ruby
enum = Enumerator.new { |y|
  # y << を呼ぶたびにイテレーションのブロックを呼び出す
  # これは 1〜3 をブロックに渡すような例
  y << 1
  y << 2
  y << 3
}
enum.each { |it|
  p it
}
# => 1
# 2
# 3
```

* これを利用して月ごとにイテレーションする

↓↓↓↓↓↓↓↓↓

>>>

```ruby
require "date"

first = Date.parse("2020/01/01")
last  = Date.parse("2020/04/01")

# 開始の月から終了の月までをイテレーションする
enum = Enumerator.new { |y|
  date = first
  while date <= last
    y << date
    date = date.next_month
  end
}

p enum.map(&:to_s)
# => ["2020-01-01", "2020-02-01", "2020-03-01", "2020-04-01"]
```

## いけるやん！！！

---

#### Enumerator.produce を使う
- - -

* Enumerator.new を使うと実現できるがやや冗長
* そこで [Enumerator.produce](https://docs.ruby-lang.org/ja/latest/method/Enumerator/s/produce.html) という便利メソッドを使う
* これは与えられたブロックを呼び出し続ける Enumerator を返す

```ruby
# 0, 2, 4, 6, 8... と無限に続ける Enumerator を定義
nums = Enumerator.produce(0) { |it| it + 2 }
# take を使って先頭から4つの要素を取り出す
pp nums.take(4)
# => [0, 2, 4, 6]
```

```ruby
# ランダムな要素を返す Enumerator を定義
rand = Enumerator.produce { rand(10) }

# 10個のランダムな要素を取り出す
pp rand.take(10)
# => [1, 1, 6, 3, 4, 0, 1, 9, 1, 1]
```

↓↓↓↓↓↓↓↓↓

>>>

* これを利用して月単位のイテレーションも定義する

```ruby
require "date"

first = Date.parse("2020/01/01")
last  = Date.parse("2020/04/01")

# これで開始日から次の月まで永遠とイテレーションする Enumerator を定義する
# これだけだと無限ループになる
enum = Enumerator.produce(first) { |it| it.next_month }

# 試しに先頭5月分を取得する
puts enum.take(5)
# => 2020-01-01
# 2020-02-01
# 2020-03-01
# 2020-04-01
# 2020-05-01
```

---

#### [Enumerable#take_while](https://docs.ruby-lang.org/ja/latest/method/Enumerable/i/take_while.html) で終了条件を指定する
- - -

* Enumerator.produce だと無限ループになる
* [Enumerable#take_while](https://docs.ruby-lang.org/ja/latest/method/Enumerable/i/take_while.html) を使うことで特定の条件までの要素を取得

```ruby
require "date"

first = Date.parse("2020/01/01")
last  = Date.parse("2020/04/01")

# take_while の条件になるまでの値を取得する
result = Enumerator.produce(first, &:next_month).take_while { |date| date <= last }
puts result
# => 2020-01-01
# 2020-02-01
# 2020-03-01
# 2020-04-01
```

## 完成！！！

---


#### まとめ
- - -

* Enumerator.produce べんり



---

### ご清聴
### ありがとうございました

