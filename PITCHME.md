## A practical type system for Ruby at Stripe.

Dmitry Petrashko, Paul Tarjan, Nelson Elhage

---

### Summary

- Stripe製Type Checkerの話
- 型情報(type signature)を書く必要がある
- オプショナルな作り
- 速い

---

### Stripe

- オンライン決済サービス |
- Ruby関西でも使っている |

---

### Stripeが使っている言語

- 主要な開発言語はRuby
- Railsは使っていない

lang | %
--- | ---
Ruby | 34
JavaScript | 16
YAML | 10
others | 30

※ others: Scala、HTML、Go

---

### YAMLは言語

---

### Stripeの状況

- 数百万行のコード
- 数百人のエンジニア
- 1日に数千のcommit

---

### Demo

https://sorbet.run/

---

### 特徴

- 型宣言を書いてもらう
- わかりやすいエラーメッセージ
- Rubyに影響を与えない
- 採用しやすい
- 速い

---

### 動き

明確なエラーメッセージ  
エラーが発生した箇所へのリンクもある

---

### Sample: nil

nilになり得る箇所をチェックする

```
foo = array_of_string[0]
foo.empty?
```

=> fooがnilになりうるので通知してくれる

---

### Sample: Dead code

到達しないコードをチェックする

---

### Sample: Dead code - OK

```
if Random.rand > 0.5
  foo = 1
else
  foo = 2
end
```

---

### Sample: Dead code - NG

到達しないコードがあるので通知する

```
if Random.rand
  foo = 1
else
  foo = 2  #=> 到達しない
end
```

---

### Sample: Union types

メソッドを持っているかチェックする  
複数の型が混在していてもチェックしてくれる

---

### Sample: Union types - OK

```
str_or_int = ["1", 2].sample
hash = str_or_int_or_array.succ
```

---

### Sample: Union types - NG

Array.succは存在しないので通知する

```
str_or_int_or_array = ["1", 2, [3]].sample
hash = str_or_int_or_array.succ
```

---

### Declaration

DSLで宣言する

---

### Declaration: Method

メソッドの前に記述する

---

### Declaration: Method - Sample

```
extend T::Helper
...
sig.(
  amount: Integer,
  currency: String,
)
.returns(Stripe::Charge)
def create_charge(amount, currency) do
  ...
end
```

```
create_charge(10_000, :jpy)
#<TypeError: Parameter currency:
  Expected type String, got type Symbol>
```

---

### チェックレベル指定可能

- true: タイプチェックする
- strict:  
    タイプチェックする  
    インスタンス変数の宣言も必要
- strong: タイプを明記していないコードを呼べない

---

### Generics

array、mapなどのコンテナにgenericsを指定できる

```
int_box = Box(Integer).new
```

---

### 6ヶ月運用してみて

手段 | 件数
--- | ---
人間が追加したメソッドの<br />シグニチャ | 3K
型チェックのアノテーション | 150
データベースオブジェクトから<br />生成した宣言 | 240k

---

### code reviewをすり抜けて
### 見つかったバグ - 1

エラーハンドリングのtypo

- NG: JSON::ParseError
- OK: JSON::ParserError

```
begin
  data = JSON.parse(File.read(path))
rescue JSON::ParseError => e
  raise "#{PACKAGE_REL_PATH} contains invalid JSON: #{e}"
end
```

---

### code reviewをすり抜けて
### 見つかったバグ - 2

Pythonばかり書いていたので、  
エラーハンドリングした後、  
コンストラクタを使わずに  
メソッド呼び出しをした

```
if look_ahead_days < 1 || look_ahead_days > 30
  raise ArgumentError('look_ahead_days must be between [_]')
end
```

---

### code reviewをすり抜けて
### 見つかったバグ - 3

nilを返す可能性があるのに  
nil checkをしていない箇所があった

```
app.post '/vi/webhook/:id/update' do
  endpoint = WebhookEndpoint.load(params[:id])
  update_webhook(endpoint, params)
end
```

---

### code reviewをすり抜けて
### 見つかったバグ - 4

static methodでinstance変数を参照していた

---

### code reviewをすり抜けて
### 見つかったバグ - 5

`||` 演算子の誤った使い方  
左辺が常にtruthyだった場合、  
右辺がdead codeになる

---

### Speed

アプリケーション | 処理行数 [lines/sec/cpu]
--- | ---
Sorbe | 10万
Java | 1万
rubocop | 1000

---

### CIとの比較

sorbetだと1インスタンスで数十秒  
CIだと10インスタンス使って10分

---

### 実装

C++で実装していてRubyVMに依存していない  
whitequarkが作ったparserを使っている  
PRごとに実行している

---

### メタプログラミングのサポート - 1

VMに依存しないおかげで  
メタプログラミングのサポートが弱い  
VMにクラス/メソッドが存在しているか  
問い合わせできない

---

### メタプログラミングのサポート - 2

以下のように対策している |

1. メモリにアプリケーションをロード |
2. Rubyのreflection apiを使ってクラス/メソッドが存在しているかを問い合わせ |
3. アノテーションとしてディスクに書き出し

---

### 感想 - 1

- 不安になる気持ちはわかる |
- プログラマに負担をかけないという方針が良い |
- VMに依存しない方針が良い |

---

### 感想 - 2

- 仕組みが知りたかった |
    - C++でDSLを実装… |
    - オーバーヘッド… |
    - アノテーション… |
- もうちょっと詳しい運用方法が知りたかった |
    実行時チェック、静的チェック… |

---

### 感想 - 3

- 動的型付けとは |

**Crystal**

