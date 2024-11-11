# UdonSharpでnullチェックは `== null` ではなく `Utilities.IsValid` を使用しよう

## 概要

Unityではnullチェックをする場合、通常 `== null` `!= null` を使いますが、UdonSharpでは `Utilities.IsValid` を使う方法が提供されています。本記事では、UdonSharpにおける `Utilities.IsValid` を使用することによってどのくらい処理速度を向上させるかについて、実際のベンチマーク結果をもとに解説します。

> [!NOTE]  
> 本記事の内容は、VRChat SDK 3.7.X に基づいています。バージョンアップにより仕様が変更される可能性があります。

## 目次

1. [UdonSharpにおけるnullチェックの基本](#udonsharpにおけるnullチェックの基本)
2. [Utilities.IsValidとは](#-Utilities.IsValid-とは)
3. [ベンチマークによる比較](#ベンチマークによる比較)
4. [他の最適化方法](#他の最適化方法)
5. [まとめ](#まとめ)

---

## UdonSharpにおける nullチェック
null チェックは非常に重要な要素です。特に、動的に変化するオブジェクトが関わるシーンでは、適切に null チェックを行わないとエラーや予期しない動作が発生する可能性があります。

例えば、プレイヤーが設定されている場合に処理を行いたい場合があります。プレイヤーはワールドの途中退出などが起きる可能性があるのでスクリプトが正常に動作し続けれるようするため、nullチェックを行って処理します。また、VRChat固有の仕様によりOnTriggerEnterメソッド等では、プレイヤーとの接触時にCollider otherがnullで実行されることがあるため、処理させないようにします。
> プレイヤーとの接触判定はOnPlayerTriggerEnterなど専用メソッドが用意されています

サンプルコード
```csharp

using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

public class Sample : UdonSharpBehaviour
{
    private VRCPlayerApi _player;
    
    private void Update()
    {
        if (_player != null)
        {
            // 処理
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        //プレイヤーと衝突した場合、otherはnullとなる
        if (other == null)
        {
            return;
        }
        
        var foo = other.GetComponent<Foo>();
        if (foo == null)
        {
            return;
        }

        // 処理
        foo.Bar();
    }
}

```

---

## `Utilities.IsValid` とは

`Utilities.IsValid` は、SDK内で提供されるユーティリティメソッドであり、特定のオブジェクトが有効であるかどうかを確認するために使用されます。このメソッドは、null チェックと同等の役割を果たし、効率的に処理されます。

![スクリーンショット 2024-11-11 150424](https://github.com/user-attachments/assets/989cafa9-2916-44e5-9135-c8147ff4e385)
object != null の場合に true を返します

---

## ベンチマークによる比較

### 検証用コード
```csharp
public void NullCheck_Benchmark()
{
    var test = 0;
    _stopwatch.Restart();
    for (int i = 0; i < _iterations; i++)
    {
        if (_testObjects[i] != null)
        {
            test++;
        }
    }
    _stopwatch.Stop();
}

public void IsValid_Benchmark()
{
    var test = 0;
    _stopwatch.Restart();
    for (int i = 0; i < _iterations; i++)
    {
        if (Utilities.IsValid(_testObjects[i]))
        {
            test++;
        }
    }
    _stopwatch.Stop();
}
```

### ベンチマーク結果

| チェック方法             | 試行回数                   | 平均時間 (ms)               | 処理速度差 (%) |
|-------------------------|---------------------------|-----------------------------|---------------|
| `!= null`               | 100                       | 0.103                       |100%           |
| `Utilities.IsValid`     | 100                       | 0.085                       |121.2%         |
| `!= null`               | 1000                      | 0.831                       |100%           |
| `Utilities.IsValid`     | 1000                      | 0.699                       |118.9%         |
| `!= null`               | 10000                     | 8.144                       |100%           |
| `Utilities.IsValid`     | 10000                     | 6.858                       |118.8%         |

この結果から、`Utilities.IsValid` を使用することで約20%程度パフォーマンスが向上します。

#### Assembly Codeからの考察
出力されるAssembly Codeを比較します。違いはnullチェック処理のみですが、`Utilities.IsValid`はシンプルなコードになっています。

##### != null
```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

public class NullCheck : UdonSharpBehaviour
{
    [SerializeField]
    private GameObject _target;
    
    private void Start()
    {
        var result = _target != null;
    }
}
```
```
.data_start
    .export _target
    __refl_typeid: %SystemInt64, null
    __refl_typename: %SystemString, null
    __intnl_returnJump_SystemUInt32_0: %SystemUInt32, null
    _target: %UnityEngineGameObject, null
    __const_SystemUInt32_0: %SystemUInt32, null
    __const_SystemObject_0: %SystemObject, null
    __lcl_result_SystemBoolean_0: %SystemBoolean, null
    __intnl_UnityEngineObject_0: %UnityEngineObject, null
    __intnl_UnityEngineObject_1: %UnityEngineObject, null
.data_end
.code_start
    .export _start
    _start:
        PUSH, __const_SystemUInt32_0
# 
# NullCheck.Start()
# 
        PUSH, _target
        PUSH, __intnl_UnityEngineObject_0
        COPY
        PUSH, __const_SystemObject_0
        PUSH, __intnl_UnityEngineObject_1
        COPY
        PUSH, __intnl_UnityEngineObject_0
        PUSH, __intnl_UnityEngineObject_1
        PUSH, __lcl_result_SystemBoolean_0
        EXTERN, "UnityEngineObject.__op_Inequality__UnityEngineObject_UnityEngineObject__SystemBoolean"
        PUSH, __intnl_returnJump_SystemUInt32_0
        COPY
        JUMP_INDIRECT, __intnl_returnJump_SystemUInt32_0
.code_end
```

##### Utilities.IsValid
```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

public class IsValidCheck : UdonSharpBehaviour
{
    [SerializeField]
    private GameObject _target;
    
    private void Start()
    {
        var result = Utilities.IsValid(_target);
    }
}
```
```
.data_start
    .export _target
    __refl_typeid: %SystemInt64, null
    __refl_typename: %SystemString, null
    __intnl_returnJump_SystemUInt32_0: %SystemUInt32, null
    _target: %UnityEngineGameObject, null
    __const_SystemUInt32_0: %SystemUInt32, null
    __lcl_result_SystemBoolean_0: %SystemBoolean, null
.data_end
.code_start
    .export _start
    _start:
        PUSH, __const_SystemUInt32_0
# 
# IsValidCheck.Start()
# 
        PUSH, _target
        PUSH, __lcl_result_SystemBoolean_0
        EXTERN, "VRCSDKBaseUtilities.__IsValid__SystemObject__SystemBoolean"
        PUSH, __intnl_returnJump_SystemUInt32_0
        COPY
        JUMP_INDIRECT, __intnl_returnJump_SystemUInt32_0
.code_end
```

---

## nullチェック 結果を保持してパフォーマンス向上

事前に null チェックした結果を bool 値として保持する方法もあります。bool 判定で済むためパフォーマンスが大きく向上しますが、この方法には注意点があります。特に、null チェック対象が動的に変更される場合では、この手法は誤った結果を引き起こす可能性があります。そのため、対象の値が意図していないタイミングに変化しないことが保証されている場合でのみ利用することを推奨します。

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

public class NullCheck : UdonSharpBehaviour
{
    [SerializeField]
    private GameObject _target;
    private bool _existsTarget;
    
    private void Start()
    {
        // 事前にnullチェック
        _existsTarget = Utilities.IsValid(_target);
    }

    private void Update()
    {
        // _targetがnullではない場合
        // if (Utilities.IsValid(_target))
        if (_existsTarget)
        { 
            // 処理
        }
    }
}
```

---

## まとめ

UdonSharpにおいて `nullチェック`を行う際、`Utilities.IsValid` を使用することで、処理速度の向上が期待できます。

