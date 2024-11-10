# UdonSharpでnullチェックは `== null` ではなく `Utilities.IsValid` を使用しよう

## 概要

Unityではnullチェックをする場合、通常 `== null` を使いますが、UdonSharpでは `Utilities.IsValid` を使う方法が提供されています。本記事では、UdonSharpにおける `Utilities.IsValid` を使用することによってどのくらい処理速度を向上させるかについて、実際のベンチマーク結果をもとに解説します。

> [!NOTE]  
> 本記事の内容は、VRChat SDK 3.7.X に基づいています。バージョンアップにより仕様が変更される可能性があります。

## 目次

1. [UdonSharpにおけるnullチェックの基本](#udonsharpにおける-null-チェックの基本)
2. [Utilities.IsValidとは](#-Utilities.IsValid-とは)
3. [ベンチマークによる比較](#ベンチマークによる比較)
4. [他の最適化方法](#他の最適化方法)
5. [まとめ](#まとめ)

---

## UdonSharpにおける `null` チェックの基本


---

## `Utilities.IsValid` とは


---

## ベンチマークによる比較

### コード

### ベンチマーク結果

ベンチマークの結果として、`Utilities.IsValid` は `== null` よりも効率的であることが確認されました。以下はサンプルの結果です：

| チェック方法             | 単純 `null` チェック (ms) | 入れ子 `null` チェック (ms) |
|-------------------------|---------------------------|-----------------------------|
| `== null`               | 50                        | 100                         |
| `Utilities.IsValid`     | 30                        | 60                          |

この結果から、`Utilities.IsValid` の方がネットワーク同期処理においてもより効果的であることが分かります。


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

## 他の最適化方法

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

