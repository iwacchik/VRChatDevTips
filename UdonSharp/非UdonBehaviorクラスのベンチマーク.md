## UdonSharpでクラス構造を作成したときのアクセスベンチマーク結果
さまざまな方法でクラス構造を作成して`$"a: {a}, b: {b}"` の形式で文字列出力するサンプルコードを実行し、アクセス速度を測定しました。以下は、100,000回の文字列出力にかかる時間を比較した結果です。

### 作成するクラス構造
```csharp
public class TestClass{
  public int A;
  public string B;
}
```

### テスト内容
- **Variable_Access**: クラス構造を作成しないて通常の変数宣言を使用。`$"a: {a}, b: {b}"` で直接変数を参照。
- **UdonbehaviorClass_Access**: データクラスとして別のUdonbehaviorクラスを作成。
- **UdonbehaviorClass_Access2**: **UdonbehaviorClass_Access**のの内部処理にgetterを追加し、その中で文字列出力を行う。
- **List_Access**: `DataList` を用い、`list[0]` などのインデックスでデータを参照。
- **Dictionary_Access**: `DataDictionary` を用い、`dict["a"]` などのキーでデータを参照。
- **MyDarkClass_Access**: [闇クラス](https://power-of-tech.hatenablog.com/entry/2024/11/02/181430)を使用し、プロパティ `A` および `B` にアクセス。
- **MyClass_Access**: U#1.2の非UdonSharpBehaviourクラスを利用。
- **MyClass_Access2**: **MyClass_Access**の内部処理にgetterを追加し、その中で文字列出力を行う。

### ベンチマーク結果
| テスト項目                      | Median (ms) | Min (ms) | Max (ms) | 相対性能 (%) |
|-------------------------------|-------------|----------|----------|--------------|
| **Variable_Access**           | 59.2113     | 58.066   | 65.105   | 100.00       |
| **UdonbehaviorClass_Access**| 139.54925   | 134.638  | 144.867  | 42.43        |
| **UdonbehaviorClass_Access2**| 138.11045   | 135.696  | 158.013  | 42.89        |
| **List_Access**               | 97.7863     | 95.907   | 105.921  | 60.55        |
| **Dictionary_Access**         | 124.2012    | 121.536  | 162.402  | 47.67        |
| **MyDarkClass_Access**        | 135.20415   | 132.985  | 143.174  | 43.79        |
| **MyClass_Access**            | 113.73265   | 111.906  | 120.445  | 52.06        |
| **MyClass_Access2**           | 142.58765   | 136.582  | 198.047  | 41.55        |

