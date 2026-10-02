---

name: tsinswreng-csharp-doc-attr

description: 使用\[Doc\] Attribute 爲C#代碼添加註釋

---

## 說明

此skill主要是**形式上的**而非內容上的約束。 至于內容上的約束則看`tsinswreng-write-comment`

## 代碼註釋形式規範

使用`[Doc]`這個Attr爲類型/成員 添加註釋, 不使用常規的斜槓註釋。 你不需要分散注意力去關注`[Doc]`在哪個命名空間下。 你直接用就是了。 `[Doc]`理應是全局直接可用的。 如果不可用 報錯`Doc`未定義, 則先停下來請示用戶, 不要自作主張。

## 爲甚麼用`[Doc]`

- 比xml註釋字數少, 看着清爽
- 方便結合`nameof()`一起用, 如`[Doc(@$"參見{nameof(SomeSymbol)}")]`
  - 看的時候能跳轉
  - 用IDE一鍵重命名符號時能同步重命名
  - 編譯器會確保 引用的符號 存在。

## 添加`[Doc]`註釋的位置

- 類型(class/interface/struct/enum等)
- 方法(不管是不是public)
- 類型的成員(包括字段和訪問器 事件等等、不管是不是public)

由于`[Doc]`是Attr, 不是所有地方都能用Attr。 故對于不能用`[Doc]`的地方 如函數內部實現的註釋, 則用回普通註釋

## 示例

````cs
[Doc($$"""
合併兩字典
#Prm[字典A]
#Prm[字典B]
#TPrm[鍵]
#TPrm[值]
#Rtn[合併後的字典]
#See[{{nameof(IDictionary)}}][說明]
#Throw[{{nameof(Exception1)}}][原因1]
#Throw[{{nameof(Exception2)}}][原因2]
#Eg[
	```cs
	var d1 = new Dict<str, obj?>(){
		["a"] = "A",
		["b"] = "B",
	}
	var d2 = new Dict<str, obj?>(){
		["c"] = "C",
	}
	var R = fn(d1, d2)

	```

	```
	R is
	{
		["a"] = "A",
		["b"] = "B",
		["c"] = "C",
	}
	```
]
#Eg[示例2]
...
""")]
public IDictionary<K,V> Merge<K,V>(IDictionary<K,V> A, IDictionary<K,V> B)
	where K:notnull
{
	//函數實現裏面不能用Attr 所以通常還是寫普通的註釋。
	//如果你實在有 想引用其他符號 的需求 就像下面這樣寫:
	_ = @$"看{nameof(...)}";
}
````

### 說明

- 像`#Xxx[Yyy]`這樣的 使用的是typst語法。中括號內的東西爲內容塊。內容塊內可以換行;
- 標籤說明:
  - Prm: Param
  - TPrm: TypeParam
  - Rtn: Return
  - Eg: e.g.
- 不是每個標籤都要寫, 按需使用標籤即可。
- 需要引用其他符號的時候就用`nameof`、避免硬編碼。
- `#Prm`這類是按參數順序填寫說明。如
  ```
  #Prm[第一個參數的解釋]
  #Prm[第二個參數的解釋]
  ```不是所有參數都有必要加註釋, 像非常常見的, 成爲規範約定的一部分的參數就完全沒必要加了。 像 `CancellationToken` 這種就完全沒必要爲註釋。 不加註釋的時候可以把相應位置的`#Prm[]`的中括號的內容空出來。 如
  ```
  #Prm[]
  #Prm[第二個參數的解釋]
  ````TPrm`亦同理。 **注意是按參數順序填寫解釋、不要再把參數名稱寫上去。**
- `#Eg`也支持寫多個。 主要寫代碼的示例。 建議多寫例子。 允許用`fn`指代本函數。 如在該例中, `Eg`中的`fn`即指代`Merge`

## `nameof`

在Doc內的字符串中引用其他符號時 能用`nameof`的就用`nameof`、禁止硬編碼

其他符號 包括 類型, 成員, 方法, 形參 等等。

錯誤示例

```cs
[Doc(@$"this is for MyClass ......")]
```

正確示例

```
[Doc(@$"this is for {nameof(MyClass)} ......")]
```

### 別名聲名

若`Doc`中的`nameof`太多, 看起來也不美觀。 這種情況允許用別名。 先在開頭定義別名 然後在後文則直接用別名 甚麼時候用別名:

- 同一個符號需多次引用
- 在示例(如Eg)中出現的nameof太多

正確示例:

```cs
[Doc(@$"
let MkRegister = {nameof(ExtnITestFnRegister.MkRegister)}
let Register = {nameof(ITestFnRegister.Register)}
#Eg[
	var R = Node.MkRegister<...>(...).Register;
]
")]
```

錯誤示例:

```cs
var R = Node.{nameof(ExtnITestFnRegister.MkRegister)}<...>(...).{nameof(ITestFnRegister.Register)}
```

## `Doc(...)`中C#多行文本塊的寫法

優先用`@$"..."`。

若此時的`...`中出現了非插值語法的的`{`或`}`字符,

再改用

```cs
$$"""

"""
```

例:

```cs
var a = @$"單行。無 被視作文本的大括號";
var b = @$"多行
無 文本的大括號";

var c = @$"{nameof(SomeSymbol)}。
這裏出現了大括號,
但是是作爲插值的語法的一部分,
不是 文本的大括號
";

var d = $$"""
這是普通的 被視作文本的大括號:
int Add(int a, int b){return a+b;}
這是 被視作插值語法的 大括號:
{{nameof(SomeSymbol)}}
爲了區分 與 避免轉義,
字符串界定符 外面寫了多少個$,
你的插值語法的大括號就要有多少個。
"""
```
