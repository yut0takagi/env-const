# Python 初期文法まとめ（超入門チートシート）

はじめて Python に触れる方向けに、最初に知っておくと便利な文法と例をコンパクトにまとめました。実際に手を動かしながら `python` の対話モードや VS Code で試してください。

---

## 変数・基本型・演算子
```python
x = 10              # int（整数）
y = 3.14            # float（小数）
name = "Alice"      # str（文字列）
flag = True         # bool（真偽）

# 四則演算
3 + 2   # 5
3 - 2   # 1
3 * 2   # 6
3 / 2   # 1.5（常に小数）
3 // 2  # 1   （切り捨て）
3 % 2   # 1   （剰余）
2 ** 3  # 8   （べき）

# 代入の複数同時・入れ替え
a, b = 1, 2
a, b = b, a  # (2, 1)
```

---

## 文字列（str）
```python
s = "hello"
len(s)          # 5
s.upper()       # 'HELLO'
"he" in s      # True

# f文字列（埋め込み）
age = 20
msg = f"{name} is {age} years old"

# 複数行
text = """
line1
line2
"""

# 連結・繰り返し
"py" + "thon"   # 'python'
"ha" * 3        # 'hahaha'
```

---

## コレクション
- list: 可変・順序あり
- tuple: 不変・順序あり
- dict: キーと値のペア
- set: 重複なし集合

```python
# list
nums = [1, 2, 3]
nums.append(4)
nums[0]          # 1
nums[-1]         # 4（末尾）

# tuple
pt = (10, 20)

# dict
user = {"id": 1, "name": "Alice"}
user["name"]        # 'Alice'
user.get("age", 0)  # 無ければ0

# set
colors = {"red", "blue", "red"}  # {'red', 'blue'}
"red" in colors  # True
```

---

## スライス（共通の取り出し方法）
```python
arr = [0, 1, 2, 3, 4, 5]
arr[1:4]    # [1, 2, 3]  (start:stop)
arr[:3]     # [0, 1, 2]
arr[3:]     # [3, 4, 5]
arr[::2]    # [0, 2, 4]  (step)
arr[::-1]   # [5, 4, 3, 2, 1, 0]  反転
```

---

## 制御構文（if / for / while）
```python
x = 10
if x > 5:
    print("big")
elif x == 5:
    print("five")
else:
    print("small")

for n in [1, 2, 3]:
    print(n)

# range: 0..4
for i in range(5):
    print(i)

# while
count = 3
while count > 0:
    count -= 1

# break / continue
for i in range(10):
    if i == 3:
        continue
    if i == 7:
        break
```

---

## 関数（def）
```python
def add(a, b):
    return a + b

# デフォルト引数
def greet(name="world"):
    print(f"hello {name}")

# 可変長引数
def debug(*args, **kwargs):
    print(args, kwargs)

# 型ヒント（任意だが推奨）
from typing import List

def mean(xs: List[float]) -> float:
    return sum(xs) / len(xs)
```

---

## 例外処理（try / except / finally / raise）
```python
try:
    n = int("10")
except ValueError as e:
    print("変換できません", e)
else:
    print("成功", n)
finally:
    print("必ず通る")

# 例外を投げる
raise RuntimeError("想定外！")
```

---

## with文（コンテキストマネージャ）
```python
# ファイルは自動でクローズされる
with open("example.txt", "w", encoding="utf-8") as f:
    f.write("hello")
```

---

## 内包表記（リスト・辞書・集合）
```python
# list comprehension
squares = [x * x for x in range(5)]          # [0,1,4,9,16]

# 条件付き
evens = [x for x in range(10) if x % 2 == 0]

# dict comprehension
mapping = {c: ord(c) for c in "abc"}        # {'a':97, ...}

# set comprehension
uniq = {x % 3 for x in range(10)}            # {0,1,2}
```

---

## クラス（最小限）
```python
class Person:
    def __init__(self, name: str):
        self.name = name

    def hello(self) -> str:
        return f"Hello, {self.name}!"

p = Person("Alice")
p.hello()  # 'Hello, Alice!'
```

### dataclass（値オブジェクトに便利）
```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str

u = User(1, "Alice")
# 自動で __init__/__repr__/__eq__ などが定義される
```

---

## 真偽値・比較のコツ
```python
# None/空文字/空リスト/0 は偽（False）として扱われる
if not []:
    print("empty")

# 同値は ==、同一性は is
x = None
x is None      # True

# 連鎖比較
1 < 2 < 5      # True

# and / or / not は短絡評価
name = "" or "guest"  # 'guest'
```

---

## import（モジュールの読み込み）
```python
import math
math.sqrt(9)  # 3.0

from datetime import date
date.today()

# 別名
import numpy as np  # サードパーティ例（要インストール）
```

---

## よくある落とし穴（覚えておくと安心）
- インデントはスペース4つが基本。タブ混在でエラーになります。
- `list` などの可変オブジェクトをデフォルト引数にしない。
  ```python
  def bad(x=[]):      # NG（共有されてしまう）
      x.append(1)
  def good(x=None):   # OK
      if x is None:
          x = []
  ```
- 浅いコピー/深いコピーの違い。
  ```python
  import copy
  a = [[1], [2]]
  b = a[:]              # 浅いコピー（内側は共有）
  c = copy.deepcopy(a)  # 深いコピー
  ```
- 文字コードは原則 `utf-8` を明示して開く。

---

## さらに進むために（キーワード）
- 仮想環境: `python -m venv .venv` / `pip install -r requirements.txt`
- テスト: `pytest`
- フォーマット/静的解析: `black` / `ruff` / `mypy`
- 非同期: `async/await`、Web: `FastAPI`、データ: `pandas` / `numpy`

---

## ミニ練習
1. 1〜100 の合計を求める関数 `sum_1_to_100()` を `range` と `sum` で実装。
2. 文字列リストから長さ3以上のものだけを小文字にして新しいリストを作る（内包表記）。
3. `User(id:int, name:str)` の `dataclass` を作り、`id` が偶数のユーザーだけを抽出する関数を書いてみる。

```python
# 例（1）
def sum_1_to_100() -> int:
    return sum(range(1, 101))
```

このファイルは最小限のチートシートです。詳細は公式チュートリアル（https://docs.python.org/ja/3/tutorial/）も参照してください。
