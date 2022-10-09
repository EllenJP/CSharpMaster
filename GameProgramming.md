# シングルトン
* そのクラスのインスタンスが1つであることを保証する。
    * チームメンバーが同じインスタンス作ることを防ぐ。
    * DontDestroyOnLoad()で破棄されないオブジェクトを作成する際に、そのオブジェクトが含まれたシーンをロードし、破棄されないオブジェクトが増えることを防ぐ。
* Managerクラスで利用することが多い。
* 課題
    * Starter.Instance.でどこからでも呼び出せるため密結合になる。解決策としてDependency Injectionを調べてみる。
```cs
public sealed class Starter : MonoBehaviour
{
    private static Starter _instance;
    public static Starter Instance => instance;

    void Awake()
    {
        if (_instance)
        {
            Destroy(this);
        }
        else
        {
            _instance = this;
            DontDestroyOnLoad(this.gameObject);
        }
    }
}
```

<br><br><br><br>

# Observerパターン
* 監視者(Observer)と監視対象(Subject)の二つで構成されており、監視者が常に監視対象を監視し続けることは効率が良くないため、監視対象が任意の処理が終わったら監視者に通知するようにするイメージをデザインパターンにしたもの。
* どんな時に使える？
    * ボタンをクリックしたときに呼ばれるコールバック関数(button.onclick.AddListener())
    * 敵が撃破されたら、スコアマネージャーに通知し敵撃破数を加算する
    * 設定された時間が経過したら通知し、スタミナを回復する、
## Actionデリゲートを用いた方法(シンプル)
* 実装が簡単だが、Observer側が破棄された場合はメモリリークの原因となる。
```cs
// Observer1、Subject1ともにGameObjectにアタッチしている。
public class Observer1 : MonoBehaviour
{
    [SerializeField] private Subject1 _subject = null;
    private void Start()
    {
        _subject.OnDestroyed += OnDestroyed;
        Debug.Log(_subject.OnDestroyed.GetInvocationList().Length); // デリゲートに登録されている関数の数を返す。
        Destroy(this.gameObject, 2); // Observerを破棄する。破棄されたObserverが購読しているためメモリリークの原因となる。
    }
    private void OnDestroyed()
    {
        Debug.Log($"{this} is called."); // null is called.が出力される。
    }
}
public class Subject1 : MonoBehaviour
{
    // public event Actionを推奨
    // 今回はObserver1で_subject.OnDestroyed.getInvocationList()を使用しているので使えない。
    public Action OnDestroyed;  // コールバックの登録
    private void Start()
    {
        Destroy(this.gameObject, 6);
    }
    private void OnDisable()
    {
        OnDestroyed?.Invoke();
    }
}
```
* メモリリークを発生させないために下記のやり方を推奨。
## C#に実装されているIObserver<T>, IObservable<T>を用いた方法(複雑)
* UniRxでも同じ考え方をしている。
```cs





```




<br><br><br><br>






# その他
## Inspector上に変数を公開するときは[SerializeField]を使用
* Inspector上では公開したいが、コード上からはアクセス不可にしたい場合に使用する。Inspector上で公開するだけならpublicでも良い。
* https://tech.pjin.jp/blog/2021/12/23/unity-serializefield
## TextMeshProのテキストはTextMeshProUGUI型であり、デフォルトでは日本語未対応
* 下記手順通りにやれば対応できる。
    * https://www.midnightunity.net/textmeshpro-japanese-font/
* Generateするときに10分ぐらい時間がかかる。次からは一度作成したファイルを使用することを推奨。


## セーブデータのJson化
* セーブデータは端末上で管理する場合はPlayerPrefsで対応できる。
    * ミニゲームでは使用機会は少ないと思う。
* サーバ上で管理する場合はJson形式にして保存する。DBに全て保存しておけないのか？
* https://docs.unity3d.com/ja/2018.4/Manual/JSONSerialization.html
```cs
var playerModel = new PlayerModel();
string jsonPlayerModel = JsonUtility.ToJson(playerModel);
Debug.Log(jsonPlayerModel); // {hp=0, mp=0...}みたいな形
// Json形式をPlayerModel型へ復元する。
playerModel = JsonUtility.FromJson<PlayerModel>(jsonPlayerModel);

[SerializeField] // Json化するためにシリアライズ属性が必要
public sealed class PlayerModel
{
    // プロパティをJson化することはできない。
    //[SerializeField] public int Hp { get; set; } = 0;
    //[SerializeField] public int Mp { get; set; } = 0;
    //[SerializeField] public int CurrentStage { get; set; } = 0;

    [SerializeField] public int hp = 0;
    [SerializeField] public int mp = 0;
    [SerializeField] public int currentStage = 0;
}
```

## クラスのデフォルトはsealedに設定し他のメンバーが不用意に継承しないようにする。
```cs
public sealed class Sample {}
```

## ボタン登録
* editor上でonclickフィールドで手軽に設定できるが、UIの階層が深くボタン数が多い場合は、該当箇所の発見やクリック処理関数の設定がめんどくさいくなるのでソースコード上で設定できるようにする。
```cs
[SerializeField] private Button _nextButton;
public void OnNextButton() {}
private void Start() 
{
    _nextButton.onClick.AddListener(OnNextButton)
}
```












# ユーザデータの保存
* たいてい端末内にセーブデータを格納する際はPlayerPrefsやJson形式でサーバ上に保存をする。そのため、毎回呼び出すと時間がかかってしまうので、アプリ終了時やスリープ時で使用し、それ以外はstatic変数に格納して使用する。
```cs
// 呼び出すとき
var currentStamina = Starter.Instance.Stamina;
```
```cs
// Starter.cs
using UnityEngine;

public sealed class Starter : MonoBehaviour
{
    private string _name;
    private int _stamina;
    private int _userRank;
    private static Starter instance;
    public static Starter Instance => instance;
    public string Name { get; set; }
    public int Stamina { get; set; }
    public int UserRank { get; set; }

    void Awake()
    {
        if (instance) Destroy(this);
        else instance = this;
    }
    public void SaveUserDataToLocal() 
    {
        PlayerPrefs.SetString("name", _name);
        PlayerPrefs.SetInt("stamina", _stamina);
        PlayerPrefs.SetInt("userRank", _userRank);
        PlayerPrefs.Save();
    }
    public void LoadPlayerData()
    {
        _name = PlayerPrefs.GetString("name")
        _stamina = PlayerPrefs.GetInt("stamina")
        _userRank = PlayerPrefs.GetInt("userRank")
    }
}
```
* 上記のやり方で気になる点。
    * どこからでもアクセスできてしまう
    * DBとの接続はどんな方法でどのタイミングで行うのか？





# トラブルシューティング
## When attaching to Unity in VisualStudio, I get a build error and cannot attach.
* Cause
    * .csproj configuration file for visual studio was not functioning well.
* Solution
    * I solved the problem by re-creating the .csproj file again in the following way.
    * https://www.youtube.com/watch?v=EYeHReCaOTg
