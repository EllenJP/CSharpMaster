# constとreadonly、static readonly使い分け

## const
* コンパイル時に値を埋め込む（展開される）
* 各インスタンスで常に同じ値を取るため、実質的にstatic
    * const: 各インスタンスに全て同じ値
    * static: 一つの変数を各インスタンスが参照
* constはプリミティブ、文字列型のみ定義可能
* https://qiita.com/4_mio_11/items/203c88eb5299e4a45f31
### メリット
* 実行速度が速い

### デメリット
* バージョニング問題
    * 参照されるアセンブリを変更しても参照するアセンブリ側に変更した値が範囲されない。リビルドしなければならない。
* 変更される予定のない値の時に使用する
* 変更される可能性が少しでもあるならstati readonlyを使うことをオススメ。

## readonly
* 実行時まで値は確定しないが、初期化後は不変な値になる。各インスタンスで同じ値になるとは限らない。

* コンストラクタ内で初期化、変更できる
### メリット
* constのように埋め込みでないためバージョニング問題は起こらない
* 読み取り専用だとクライアントに示すことができる
* コンパイラ時では決められない値を実行時に決定することができる
### デメリット
* staticクラスでは使用できない

## static readonly
* static readonlyの値は静的コンストラクタ内で変更、初期化ができる。
* https://docs.microsoft.com/ja-jp/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
```cs
public class Test
{
    private static readonly int value = 1;
    static Test()
    {
        value = 100;
        Console.WriteLine("hello");
    }
    public int Value { get { return value; } }
}

```

### メリット
* constのように埋め込みでないためバージョニング問題は起こらない
* 将来変更される可能性がある場合はconstよりもstatic readonlyを使用する。
* constが使用できない値やその値をコンパイル時に計算できない場合に使用できる

### デメリット
* constの代用としての意味合いが強く、readonlyに重きを置いた使い方が今のところ思いつかない。

<br><br><br><br><br><br>

# 演算子
## 三項条件演算子
```cs
int temp = 25;
var ans = temp < 20 ? "Cold" : "Hot"; // Hot 
```

## Null合体演算子、Null条件演算子
```cs
// Null合体演算子
// aがnullでないならaを代入、aがnullならbを代入
List<int> a = null;
List<int> b = new List<int>();
var b = a ?? b;

// Null合体代入演算子
// aがnullならbを代入する
// int?の?を付けることでnullを許容する意味になる
int? a = null;
int? b = 5;
a ??= b;

// Null条件演算子
public class Calculator
{
    public double SumList(List<double[]> list, int row)
    {
        // listがnullならdouble.NaN
        // list[row]がnullならdouble.NaN
        // list, list[row]どちらもnullでなければSum()を返す。
        return list?[row]?.Sum() ?? double.NaN;
    }
}
static void Main()
{
    List<double[]> list = new List<double[]>()
    {
        new double[]{1, 2, 3},
        null
    };
    Calculator c = new Calculator();
    Console.WriteLine(c.SumList(list, 0)); // 6
    Console.WriteLine(c.SumList(list, 1)); // NaN

}
```

```cs
```


<br><br><br><br><br><br>

# LINQ
* [デリゲート](#delegate)、[ラムダ式](#lambda)、[拡張メソッド](#extendedMethod)を先に理解するとわかりやすい



# <a id="delegate">デリゲート</a>
# <a id="lambda">ラムダ式</a>
# <a id="extendedMethod">拡張メソッド</a>