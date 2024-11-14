

## メソッド名によって処理速度の違いがあるのか？

```csharp
public class SendEventBenchmark : BenchmarkRunnerBase
{
    public void LongMethod()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            SendCustomEvent("Aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa");
        }
        _stopwatch.Stop();
        
        OnComplete();
    }
    
    public void LongMethod2()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            SendCustomEvent(nameof(Aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa));
        }
        _stopwatch.Stop();
        
        OnComplete();
    }
    
    public void LongMethod3()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            Aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa();
        }
        _stopwatch.Stop();
        
        OnComplete();
    }
    
    public void ShortMethod()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            SendCustomEvent("A");
        }
        _stopwatch.Stop();
        
        OnComplete();
    }
    
    public void ShortMethod2()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            SendCustomEvent(nameof(A));
        }
        _stopwatch.Stop();
        
        OnComplete();
    }
    
    public void ShortMethod3()
    {
        _stopwatch.Restart();
        for (int i = 0; i < _iterations; i++)
        {
            A();
        }
        _stopwatch.Stop();
        
        OnComplete();
    }

    private int _test = 0;
    public void Aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ()
    {
        _test++;
    }
    
    public void A()
    {
        _test++;
    }

}
```

| 処理内容                          | 平均時間 (ms) | 最小時間 (ms) | 最大時間 (ms) | １回あたりの平均時間 (ms) |
|-----------------------------------|---------------|---------------|---------------|---------------------------|
| SendCustomEvent("ShortMethod")     | 60.989        | 60.489        | 61.741        | 0.000610                  |
| SendCustomEvent(nameof(ShortMethod)) | 60.922        | 60.191        | 62.011        | 0.000609                  |
| 直接メソッド実行 ShortMethod()                  | 36.680        | 35.885        | 37.699        | 0.000367                  |
| SendCustomEvent("LongMethod")     | 108.892       | 107.997       | 109.732       | 0.001089                  |
| SendCustomEvent(nameof(LongMethod)) | 108.422       | 107.611       | 109.559       | 0.001084                  |
| 直接メソッド実行 LongMethod()                 | 36.272        | 35.762        | 36.874        | 0.000363                  |
