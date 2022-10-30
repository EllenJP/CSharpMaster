# 目次
* [コーディング](#コーディング)
* [constとreadonly、static readonly使い分け](#constとreadonly、staticreadonly使い分け)
* [演算子](#演算子)
* [LINQ](#LINQ)
* [デリゲート](#delegate)
* [ラムダ式](#lambda)
* [拡張メソッド](#extendedMethod)
* [インターフェース](#インターフェース)


<a id=""></a>

# コーディング<a id="コーディング"></a>
## if文の{ }は複数行の時は省略しない
* 中身のステートメントが1つだけなら中括弧はいらないが、1つのステートメントが複数行にまたがるときは中括弧を付けると良い。
* 1行の場合でも中括弧を付けるチームもあるので、適宜合わせる。
```cs
// NG1
var flagA = true;
var flagB = false;
var flagC = true;
if (flagA)
{
    Console.WriteLine("hello0");
}
else
    if (flagB)
    {
        Console.WriteLine("hello1");
    }
    else if (flagC)
    {
        Console.WriteLine("hello2");
    }
    if (flagA) // else文から外れている
    {
        Console.WriteLine("hello3");
    }
    Console.WriteLine("hello4"); // else文から外れている
    Console.WriteLine("hello5"); // else文から外れている

// NG2
if (FlagA)
    Console.WriteLine($"Hello1");
    Console.WriteLine($"Hello2"); // if文には入らない

```


# constとreadonly、static readonly使い分け<a id="constとreadonly、staticreadonly使い分け"></a>
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


<br><br><br>


# 演算子<a id="演算子"></a>
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


<br><br><br>


# LINQ<a id="LINQ"></a>
* [デリゲート](#delegate)、[ラムダ式](#lambda)、[拡張メソッド](#extendedMethod)を先に理解するとわかりやすい
* IEnumerable<T>インターフェースを実装しているオブジェクトに対して操作可能。
    * List, Dictionary, Array, String, etc...
* Where()とSelect()をまず押さえる。
## Where()
* シーケンス（連続するもの）から条件を満たしたものだけを抽出するメソッド（フィルター）
* Where()で対象を絞り込み、Select()で絞り込んだ対象に対して処理を行う。
* IEnumerable<T>を返す。ToArray()やToList()、ToDictionary()をよく使ったりする。
## Select()
* シーケンスから各要素に対して変換処理（射影）を行う。
    * 射影とはコレクションの中から条件に一致した要素を必要なら加工し取り出す処理（わかりにくいので、全ての要素に対して変換処理を行うでよい）
* IEnumerable<T>を返す。ToArray()やToList()、ToDictionary()をよく使ったりする。
* Where()で対象を絞り込み、Select()で絞り込んだ対象に対して処理を行う。 
```cs
List<int> list = new List<int>() { 1, 2, 3, 4 };
var tmp = list.Select(x => x * 10);

// 以下でもコンパイルは通る。使われていないので使わない。
var tmp = list.Select(delegate (int x) { return 10 * x; });
delegate int MyDelegate(int x);
MyDelegate myDelegate = Calc;
var tmp = list.Select(x => Calc(x)).ToArray();
var tmp = list.Select(Calc).ToArray();
var tmp = list.Select(x => myDelegate(x)).ToArray();
static int Calc(int n)
{
    return n * 10;
}
```
## 遅延評価と即時評価
* 即時評価: 評価後は内容は変わらない
* 遅延評価: 内容を取り出す度に評価する
* 基本は小メモリで処理量が少ない遅延評価を使用するが、何度も同じ評価結果を使用する場合はキャッシュするために即時評価にする。
```cs
// 要素追加時の評価結果が異なる。
// 即時評価
List<string> names = new List<string>() { "apple", "blueberry", "cucumber" };
var query = names.Where(s => s.Length <= 5).ToList();
foreach (var item in query) 
    Console.WriteLine(item); // apple
// eggを追加する。
names.Add("egg");
foreach (var item in query) 
    Console.WriteLine(item); // apple

// 遅延評価
List<string> names = new List<string>() { "apple", "blueberry", "cucumber" };
var query = names.Where(s => s.Length <= 5);
foreach (var item in query) 
    Console.WriteLine(item); // apple
// eggを追加する。
names.Add("egg");
foreach (var item in query) 
    Console.WriteLine(item); // apple, egg
```


<br><br><br>


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


<br><br><br>


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


<br><br><br>


# <a id="extendedMethod">拡張メソッド</a>
* 既存の型に新規メソッドを追加することができる。
    * 派生クラスに処理を記載することもできるが、sealed修飾子がついている場合はできない。
    * 使用する場合はインスタンスメソッドの呼び出し方をするが定義するときは静的メソッドの書き方をしているので混乱しないように。
* 拡張メソッドは多用すると、どこに何の拡張機能があるのかチームの混乱を招く。
* 拡張メソッドと既存のメソッドが同名の場合は既存のメソッドが優先される。
    * string.ToString()とか
```cs
// IEnumerableを実装しているstringクラスでReverse()時にIEnumerable<char>を返すのではなく、string型を返したい
static void Main(string[] args)
{
    string str = "hello";
    // str変数がReverseメソッドの第1引数にわたる
    // 注意: string.Revese()は存在しない。stringがIEnumerableインターフェースを実装しているため。
    string revStr = str.Reverse();
}
 // 拡張メソッドを定義するためには静的クラスにする
 public static class StringExtensions 
 {
     // 静的メソッド+第一引数にthis追加+第二引数以降に宣言したものが拡張メソッド呼び出し時の引数となる
     public static string Reverse(this string str) 
     {
         Console.WriteLine("拡張メソッドを呼び出しました");
         if (string.IsNullOrWhiteSpace(str)) return string.Empty;
         char[] chars = str.ToCharArray();
         Array.Reverse(chars);
         return new string(chars);
     }
 }
```
```cs
// コレクション反復列挙子の拡張メソッド
// namespaceはMyExtensionsとかにしておくと良い
public static class EnumerableExtensions // 拡張メソッドを定義するためには静的クラスにする
{
    // <T>はlist<int>とint[]はint、Dictionary<int, string>は<KeyPairValue<int, string>となる
    public static void ForEach<T>(this IEnumerable<T> enumerable, Action<T> action)
    {
        foreach (T item in enumerable)
        {
            action(item);
        }
    }
}

List<int> list = new List<int>() { 1, 2, 3, 4 };
int[] array = new int[] { 5, 6, 7, 8 };
// Dictionary型はkeyAndValuePairs型{[1, "hello"]}がTに入る。
Dictionary<int, string> dic = new Dictionary<int, string>() {
    {1, "hello" },
    {2, "say" },
};

list.ForEach(s => Console.WriteLine(s));
array.ForEach(s => Console.WriteLine(s));
dic.ForEach(s => Console.WriteLine(s));
```
* LINQのSelect()も拡張メソッドで定義されている。
```cs
// Select<TSource, TResult>は、関数で使うジェネリック型を定義しているだけ。
// 「this IEnumerable<TSource> source」よりIEnumerableインターフェースを実装している型でSelect()が呼ばれる
// IEnumerable<T>は列挙する機能を持つため、TSourceには列挙する要素の型が入る。
// 例）list<int>ならTSource = int
// 第二引数（Func<TSource, TResult>）から呼び出し側の引数となる。
public static IEnumerable<TResult> Select<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector);

// public static IEnumerable<int> Select(this IEnumerable<int> source, Func<int, int>)
// x => x * 10: Func<int, int> int型を引数に取り、int型を返す
List<int> list = new List<int>() { 1, 2, 3, 4 };
var tmp = list.Select(x => x * 10);

// Select()が使えない状況
// voidは変数の型として使用できない。つまり、Func<>のTResultには戻り値がvoidの関数は使用できない。エラーとしては、型引数を使い方から推論することができないため。
list.Select(x => Calc(x)); // OK
list.Select(x => VoidFunc(x)); // NG
list.Select(x => Console.WriteLine(x)); // NG
static int Calc(int n)
{
    return n * 10;
}
static void VoidFunc(int n)
{
    return;
}
```
* LINQのWhere()も拡張メソッドで定義されている。
```cs
// Select()とほぼ同じ
// 「Func<TSource, bool> predicate」の箇所が異なる。TSource型を引数にbool値を返すメソッドのみ使える。
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate);
```


<br><br><br>

# インターフェース

# === ここからSOLID Principle ===

# オープンクローズドの原則
* 

## 初期の仕様
* 4人パーティで敵と戦う
* パーティは剣士、魔法使い、騎士、弓使いの4人
* 各キャラクターのコマンドは、こうげき、ぼうぎょ、アイテムの3つ。
## 方針
* キャラクターごとのクラスを作り、パーティ編成のために各クラスのインスタンスをリストに追加して表現したい。
* そのためにObject型でリストを作ればよいと考えがちだが、パーティ全員に同じコマンドを入力したいときに、いちいちキャストしてメソッドを呼び出さなければいけなくなる
```cs
var playerList = new List<Object>()
{
    new SwordMan(),
    new Wizard(),
    new Knight(),
    new Archer(),
};
playerList[0]
```


たたかう継承元の場合にObject型のみしかできない
```cs
// 呼び出すときに
public sealed class SwordMan
{
    private static readonly string Name = "剣士";
    private static readonly int OffensiveAbility = 10;
    public void Attack() { Console.WriteLine($"{Name}は{OffensiveAbility}のこうげきをした。"); }
    public void Defend() { Console.WriteLine($"{Name}はぼうぎょした。"); }
    public void UseItem() { Console.WriteLine($"{Name}はアイテムをつかった。"); }
}
public sealed class Wizard
{
    private static readonly string Name = "魔法使い";
    private static readonly int OffensiveAbility = 10;
    public void Attack() { Console.WriteLine($"{Name}は{OffensiveAbility}のこうげきをした。"); }
    public void Defend() { Console.WriteLine($"{Name}はぼうぎょした。"); }
    public void UseItem() { Console.WriteLine($"{Name}はアイテムをつかった。"); }
}
public sealed class Knight
{
    private static readonly string Name = "騎士";
    private static readonly int OffensiveAbility = 10;
    public void Attack() { Console.WriteLine($"{Name}は{OffensiveAbility}のこうげきをした。"); }
    public void Defend() { Console.WriteLine($"{Name}はぼうぎょした。"); }
    public void UseItem() { Console.WriteLine($"{Name}はアイテムをつかった。"); }
}
public sealed class Archer
{
    private static readonly string Name = "剣士";
    private static readonly int OffensiveAbility = 10;
    public void Attack() { Console.WriteLine($"{Name}は{OffensiveAbility}のこうげきをした。"); }
    public void Defend() { Console.WriteLine($"{Name}はぼうぎょした。"); }
    public void UseItem() { Console.WriteLine($"{Name}はアイテムをつかった。"); }
}

```




親クラスを継承するやり方は、好ましくない。なぜなら

## 仕様の変更が発生！！
* 新規キャラクターの追加：治癒士
* コマンドは同じ

## さらに仕様の変更が発生！！
* 一定確率で、クリティカル（1.5倍）が発生する

```cs



```





