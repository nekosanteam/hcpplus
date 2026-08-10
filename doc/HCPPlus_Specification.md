# HCP+ Specification

## 概要

HCP+ (Hierarchical ComPact description Plus) チャートは、HCPチャートの階層的な可読性を維持しながら、
AI支援開発・要件分析・テスト生成・トレーサビリティ管理を目的として拡張した仕様記述DSLである。

設計原則:

- GOAL 中心の要件分解 (GOAL ⇒ サブ GOAL)
  - ACT による実現手段の記述
  - IF/FOR/PARALLEL 等による制御構造表現
- RULE による不変条件の定義
  - ASSERT による判定条件の記述
- TEST による検証内容の提示
- DATA による操作対象のスコープ明示
- EVENT によるイベント駆動表現
- レビュー容易性を優先
- AI による詳細化・検証を支援

---

## 基本要素

### GOAL

GOAL では達成したい状態を表す。

```text
GOAL 注文成立
```

GOAL はサブ GOAL へ分解できる。
詳細で具体的な処理内容は ACT (後述) を用いて記述する。

```text
GOAL 注文成立
    GOAL 在庫確認完了
        ACT 在庫照会
    GOAL 与信確認完了
        GOAL 注文先特定
        GOAL 支払い確認
```

#### 属性:

- Pre

  前提とする条件を記述する。

- Details (任意)

  補足説明となるコメントを記述する。

#### 例:

```text
GOAL 在庫確認完了
    Details:
        予約のバッディングはカート表示時点でチェック済みのため不要。
    Pre:
        注文予定の商品および数量特定済み
```

---

### ACT

GOALを実現する手段。処理内容を記述する。
条件分岐や繰り返しなど、処理内容の詳細をさらに記述することも可能。

```text
ACT 在庫照会
```

#### 属性:

- Pre
- Exception
- Boundary (internal or external。省略時は internal)
- Details
- Context
- Purpose

#### 例:

```text
ACT 与信照会
    Boundary:
        external
    Exception:
        Timeout
        与信NG
```

---

### RULE

RULEでは不変条件(Invariant)を表す。
詳細な判定条件は ASSERT (後述) を用いて記述する。

処理途中を含め、適用スコープ内で常に成立しなければならない。

#### 属性:

- Pre
- When (後述)
- Details

例:

```text
RULE 在庫非負
    在庫数 >= 0
```

### ASSERT

ASSERT は条件が満たされることを具体的に判定する方法を
条件式として記述する。

#### RULE のスコープ

GOAL 配下に記述した RULE は、その GOAL 配下全体へ適用される。

```text
GOAL 在庫管理
    RULE 在庫非負
        ASSERT 在庫数 >= 0
```

GOAL に共有したい RULE は When 属性を使用する。

```text
RULE 顧客ID一意
    When:
        GOAL 顧客管理
    顧客IDは一意
```

---

### DATA

GOAL の中の処理でアクセスするデータを定義する。

#### DATA のスコープ

サブ GOAL (下位) にあるデータはアクセスできない。
同一 GOAL 内または上位の GOAL にあるデータのみアクセス可能である。

### TEST

GOAL に対して BDD 形式で受入条件を記述する。

#### 属性:

BDD 形式の各記述は Given, When, Then で記述する。

- Given
- When
- Then

```text
TEST 正常注文
    Given:
        在庫あり
    When:
        注文する
    Then:
        注文成立
```

---

### EVENT

処理開始契機。

```text
EVENT 注文要求
    Push: Order
```

---

### WANT

強制しない目的 (GOAL) や規則 (RULE) を記述。

---

## 制御構造

### IF

```text
IF 在庫あり
    ACT 注文登録
ELSE
    ACT 在庫不足通知
```

### FOREACH

```text
FOREACH 各商品
    ACT 在庫確認
```

### WHILE

```text
WHILE 未完了
    ACT 再送
```

### PARALLEL

```text
PARALLEL
    ACT 決済
    ACT 在庫引当
JOIN
```

---

## 論理ノード

RULE や条件式で使用する。

```text
NOT
AND
OR
OPTIONAL
```

例:

```text
AND
    在庫あり
    OR
        FIDO認証
        パスワード認証
```

---

## 品質管理

### 必須要件:

- GOAL, RULE, WANT が充足可能であること。

### 主な測定対象:

- Goal Coverage
- Act Coverage
- Rule Coverage
- Test Coverage
- Traceability Coverage
- Review Count
- Review Defect Density
- Verified Goal Ratio
- Verified Rule Ratio
- Risk-weighted Coverage
