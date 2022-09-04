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


<br><br><br><br><br><br>

# LINQ
* [デリゲート](#delegate)、[ラムダ式](#lambda)、[拡張メソッド](#extendedMethod)を先に理解するとわかりやすい



# <a id="delegate">デリゲート</a>
* メソッドを格納する型を定義することができる
    * メソッドの型は種類が多いため、intやdoubleのような予約語をメソッドには割り当てることができない。そこで考えられたのがデリゲート。
*  変数にメソッドを格納、引数にメソッドを渡すことが可能
## 基本
* クラスメソッド、インスタンスメソッド両方格納できる
```cs
// 基本
public class Sample
{
    // 引数: void、戻り値: voidのメソッドを格納する
    delegate void MyType();
    public static void Func1()
    {
        Console.WriteLine("hello world");
    }
    public static void Func2()
    {
        Console.WriteLine("good");
    }
    static void Main()
    {
        MyType myType = Func1;
        myType += Func2;
        // 格納したメソッドの一括呼び出し
        myType.Invoke(); // hello worldとgoodが出力
    }
}
```
* デリゲートにはAction型、Func型、Predicate型が用意されている。
    * Action: 任意個の引数を取り、かつ、戻り値のないメソッドを格納する型
    * Func: 任意個の引数を取り、かつ、戻り値のあるメソッドを格納する型
    * Predicate: 任意個の引数を取り、かつ、戻り値の型がBooleanであるようなメソッドを格納する型
```cs
public class Sample
{
    public static void ActionFunc(int n)
    {
        Console.WriteLine("ActionFunc");
    }
    public static int FuncFunc(string s)
    {
        Console.WriteLine("FuncFunc");
        return 0;
    }
    public static bool BoolFunc(int n)
    {
        Console.WriteLine("BoolFunc");
        return true;
    }
    static void Main()
    {
        Action<int> action = ActionFunc;
        // Func delegateのみ<T1, ..., TResult>で定義する
        Func<string, int> func = FuncFunc;
        Predicate<int> predicate = BoolFunc;
        action.Invoke(1);
        func.Invoke("s");
        predicate.Invoke(1);
    }
}
```
* 見る機会は少ないdelegate
    * Predicate<T>型: ListクラスのFindAllメソッドの引数
    * Convertor<TInput, TOutput>型: ListクラスのConvertAllメソッドの引数
    * Comparison<T>型: ListクラスのSortメソッドの引数

## 応用
* 音やエフェクトを〇秒後に実行する処理

```cs
public class DelayGenerator
{
    // publicにしないとGenerateDelay()よりもアクセシビリティのレベルが低くなってしまう
    // publicにするか、Sampleクラスで定義する。
    // PlayBGM()とPlayEffect()を入れられる型
    public delegate void DelayedMethod();
    // DelayedMethod型をActionに変更可能
    public void GenerateDelay(DelayedMethod target, int delayTime)
    {
        Thread.Sleep(delayTime * 1000);
        target.Invoke();
    }
}

public class Sample
{
    private static void PlayBGM()
    {
        Console.WriteLine("Play BGM");
    }
    private static void PlayEffect()
    {
        Console.WriteLine("Play Effect");
    }
    static void Main()
    {
        DelayGenerator delayGenerator = new DelayGenerator();
        delayGenerator.GenerateDelay(PlayBGM, 1);
        delayGenerator.GenerateDelay(PlayEffect, 3);
    }
}
```
* 非同期処理をするときに使えるかな。DBからデータ引っ張ってくる間にフレームだけ先に表示させておくみたいな。



# <a id="lambda">ラムダ式</a>
## ラムダ式の成り立ち
* 仕様: 定義された配列から任意の値が要素と一致する場合にカウントする関数
```cs
static int Count(int num)
{
    var count = 0;
    var numbers = new int[] { 1, 2, 3 };
    foreach (var item in numbers)
    {
        if (num == item) count++;
    }
    return count;
}

```
* 仕様変更: 定義された配列から任意の配列に変更する
```cs
static int Count(int[] numbers, int num)
{
    int count = 0;
    foreach (var item in numbers)
    {
        if (num == item) count++;
    }
    return count;
}
```
* 仕様変更: 「要素と一致する場合にカウントする」条件から任意の条件にしたがってカウントするように変更する。
    * デリゲートを使用する
```cs
// public delegate bool Condition(int value);で自作のデリゲート型を定義してもよい
public static int Count(int[] numbers, Predicate<int> conditions)
{
    int count = 0;
    foreach (var item in numbers)
    {
        // 新しい条件は関数のみ定義すればよい。
        if (conditions(item))
            count++;
    }
    return count;
}
public static bool IsEven(int n)
{
    return n % 2 == 0;
}
public static bool IsOdd(int n)
{
    return n % 2 != 0;
}
public static bool IsMatchFive(int n)
{
    return n == 5;
}
static void Main()
{
    int[] numbers = new int[] { 1, 2, 3, 4, 5 };
    Console.WriteLine(Count(numbers, IsEven));
    Console.WriteLine(Count(numbers, IsOdd));
    Console.WriteLine(Count(numbers, IsMatchFive));
}
```
* 一度しか使わない関数でも、毎回定義するとコード量が多くなってしまい冗長になる問題
    * 匿名関数を使用する
    * delegate (型 引数) { 処理 }の形
```cs
static void Main()
{
    int[] numbers = new int[] { 1, 2, 3, 4, 5 };
    Console.WriteLine(Count(numbers, IsEven));
    Console.WriteLine(Count(numbers, IsOdd));
    Console.WriteLine(Count(numbers, IsMatchFive));
    Console.WriteLine(Count(numbers, delegate (int n) { return n == 2; }));
}
```
* 匿名関数はdelegateキーワードや{}を使っているので初見は分かりにくい
    * より分かりやすくしたラムダ式が登場！
    * ラムダ式はメソッドのためデリゲートの型に格納可能。
```cs
Count(numbers, delegate (int n) { return n == 2; });
// 冗長なラムダ式
// =>（ラムダ演算子：アロー演算子）を使用し左側に引数宣言、右側にメソッドの本体を記述する
Count(numbers, (int n) => { return n == 2; })
// 引数の型は使用される関数の引数で定義されているため省略可能。また引数が一つの場合は()不要:
// 中身が一行のステートメントなら{}とreturn省略可能
Count(numbers, n => n == 2);
```

# <a id="extendedMethod">拡張メソッド</a>