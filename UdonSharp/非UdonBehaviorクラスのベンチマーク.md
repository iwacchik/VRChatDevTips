## UdonSharp クラス構造のアクセスベンチマーク結果
UdonSharpで異なるデータ構造やクラスを用いて、`$"a: {a}, b: {b}"` の形式で文字列出力するサンプルコードを実行し、アクセス速度を測定しました。以下は、100,000回の文字列出力にかかる時間を比較した結果です。

### テスト内容
- **Variable_Access**: 通常の変数宣言を使用。`$"a: {a}, b: {b}"` で直接変数を参照
- **List_Access**: `DataList` を用い、`list[0]` などのインデックスでデータを参照
- **Dictionary_Access**: `DataDictionary` を用い、`dict["a"]` などのキーでデータを参照
- **MyDarkClass_Access**: 闇クラスを使用し、プロパティ `A` および `B` にアクセス
- **MyClass_Access**: U#1.2の非UdonSharpBehaviourクラスを利用
- **MyClass_Access2**: **MyClass_Access**の内部処理にgetterを追加し、その中で文字列出力を行う

### ベンチマーク結果
| テスト名                  | Median (ms) | Min (ms) | Max (ms) | |
|---------------------------|-------------|----------|----------|----------|
| **Variable_Access**       | 175.92      | 172.45   | 209.04   |a => `int a` b => `string b`|
| **List_Access**           | 269.57      | 261.79   | 308.10   |a => `list[0]` b => `list[1]`|
| **Dictionary_Access**     | 355.71      | 347.18   | 400.53   |a => `dict["a"]` a; b => `dict["b"]`|
| **DarkClass_Access**    | 385.70      | 375.28   | 422.99   |a => `darkClass.A()` b => `darkClass.B()`|
| **MyClass_Access**        | 282.02      | 279.66   | 324.56   |a => `MyClass.A`; b => `MyClass.B`|
| **MyClass_Access2**        | 356.04      | 346.20   | 416.16   |MyClass内で出力処理 `GetStr() => $"a: {A}, b: {B}"`|


