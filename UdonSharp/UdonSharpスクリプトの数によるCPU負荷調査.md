# UdonSharpスクリプトの数によるCPU負荷について

UdonSharpスクリプトの数が、CPU負荷にどのような影響を与えるかを検証しました。
特に、ライフサイクルメソッドの有無による影響を調査しています。

---

テスト方法は`Instantiate`を使用してスクリプト付きオブジェクトを生成し続け、**FPSが20未満**になるまでに生成できた数を記録して評価します。

## テスト項目
- **NoBehaviorMethod**: ライフサイクルメソッドなし
- **BehaviorMethod**: `Update`を実装
- **BehaviorMethod2**: `Update` `LateUpdate`を実装
- **BehaviorMethod3**: `Update` `LateUpdate` `PostLateUpdate`を実装
- **BehaviorMethodDisable**: `Update` enabled = falseを実行し、処理を行わないようにする
- **BehaviorMethodDisable2**: `Update` `LateUpdate` enabled = falseを実行し、処理を行わないようにする
- **BehaviorEvent**: `SendCustomEventDelayedFrames(nameof(_LoopUpdate), 1, EventTiming.Update)`を常時ループする
- **BehaviorEvent2**: `SendCustomEventDelayedSeconds(nameof(_LoopUpdate), 0.1f, EventTiming.Update)`を常時ループする
  
## ベンチマーク結果

| テスト項目                    |  スポーン数  |        
|------------------------------|-------------|
| **NoBehaviorMethod**         | 500000↑     |
| **BehaviorMethod**           | 75674       |
| **BehaviorMethod2**          | 33566       |
| **BehaviorMethod3**          | 23210       |
| **BehaviorMethodDisable**    | 500000↑     |
| **BehaviorMethodDisable2**   | 500000↑     |
| **BehaviorEvent**            | 40983       |
| **BehaviorEvent2**           | 70276       |

---

## まとめ
- ライフサイクルメソッドを実装しないスクリプトはCPU負荷がほとんどない(500000オブジェクトを生成した状態でもFPSに変化がなかった)
- ライフサイクルメソッドの数によってCPU負荷が高くなる
- `SendCustomEventDelayedFrames(1)`や`SendCustomEventDelayedSeconds(0.1f)`は呼び出しコストが多く常時ループでの使用は負荷が高い
- ライフサイクルメソッドを実装した場合でも、`スクリプトをenabled = false`することでCPU負荷をなくすことができる
