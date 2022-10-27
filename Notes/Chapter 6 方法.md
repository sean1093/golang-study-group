# Chapter 6 方法

## 6.1 方法宣告



```go=
package geometry
import "math"
type Point struct{ X, Y float64 }
// traditional function
func Distance(p, q Point) float64 {
return math.Hypot(q.X-p.X, q.Y-p.Y)
}
// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

Go語言的Receiver是綁定function到特定type成為其method的一個參數。
這裡的 ==參數 p== 就是一個 receiver (接受器)

這樣宣告會有什麼結果?

function加了receiver即成為一個type的method
也就是說, 你可以這樣呼叫

```go=
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(p.Distance(q)) // "5", method call
```
這個是宣告type ==Point== 的method (Point.Distance)
p.Distance 運算是稱作selector, 因為他選擇Point Type的接受器(receiver)的Distance method.


在這個例子裡面，如下原本的call法也是可以繼續使用
因為這是宣告 package-level function called ==geometry.Distance==

```go=
fmt.Println(Distance(p, q)) // "5", function call
```
由於每個Type 有自己的method namespace
所以我們可以對不同Type 使用Distance這個名稱

Slice 型別的例子 (e.g. Path )
這裡是一個 Disatance method 他會算出周長

```go=
// A Path is a journey connecting the points with straight lines.
type Path []Point
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
            sum += path[i-1].Distance(path[i])
        }
    }
    return sum
}
```

```go=
perim := Path{
    {1, 1},
    {5, 1},
    {5, 4},
    {1, 1},
}
fmt.Println(perim.Distance()) // "12"
```

Path.Distance 使用 Point.Distance來計算長度

結論:
一個Type的所有method, 要有獨特的名稱
但是不同Type可以用一樣的method name
好處取名不會落落長

## 6.2 指標接受器方法


```go=
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```
這個method 實際上的名稱為
(*Point).ScaleBy

:::info
如果Point 的任一個method 有 pointer receiver
則Point 的所有 method 都應該要有pointer receiver
即使沒有必要
（書裡面的例子應該是方便作者講解）
:::

```go=
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // "{2, 4}"

p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
fmt.Println(p) // "{2, 4}"

p := Point{1, 2}
(&p).ScaleBy(2)
fmt.Println(p) // "{2, 4}"
```

後面兩個寫法很醜
不過在golang裏面, 如果你的變數p 是 Point type
且你的method是需要*Point 接受器的話
你是可以寫成
```go=
p.ScaleBy(2)
```
也就是說,你本來要寫成像第三個的樣子
但是不用
這是因為 complier 會把變數轉成 &p
不過這只對==變數==有效 (**<font color="#f00">p := Point{1, 2}</font>**)
下面的寫法是不行的, 因為辦法取得暫存器的位址
```go=
Point{1, 2}.ScaleBy(2) // compile error: can't take address of Point literal
```
但是 對於Point.Distance這樣的method, 我們是可以用*Point 接受器的
因為是有辦法從位址取得值 
以下兩個function call 相同 ( pptr := &p)

```go=
pptr.Distance(q) // pptr 是 *Point type, 但是complier 會幫我們插入 *
(*pptr).Distance(q)
```







總結:
合法的方法呼叫運算式
:::success
接受器宣告參數型別（e.g. p := Point{1, 2}, p 是 Point Type）
接受器參數型別（e.g. func (p *Point) ScaleBy(factor float64) , p 接受器參數Type 為 *Point）
:::

|==→==接受器宣告參數型別==↓==接受器參數型別| T | T* |
| - | -| - |
| T        |Point{1, 2}.Distance(q)|  pptr.Distance(q) // implicit (*pptr)   |
| T*       | p.ScaleBy(2) // implicit (&p) |pptr.ScaleBy(2)|








nil 是有效的接受器值, 就像一些函數允許nil points作為參數一樣
nil若為該Type的有意義零值, e.g. map, slice
則其method 容許 nil 指標當參數
當你定義允許nil 作為接受器值的型別方法時, 最好在文件中明確表明這一點

```go=
net/url
package url
// Values maps a string key to a list of values.
type Values map[string][]string
// Get returns the first value associated with the given key,
// or "" if there are none.
func (v Values) Get(key string) string {
    if vs := v[key]; len(vs) > 0 {
        return vs[0]
    }
    return ""
}
// Add adds the value to key.
// It appends to any existing values associated with key.
func (v Values) Add(key, value string) {
    v[key] = append(v[key], value)
}
```


```go=
m := url.Values{"lang": {"en"}} // direct construction
m.Add("item", "1")
m.Add("item", "2")
fmt.Println(m.Get("lang")) // "en"
fmt.Println(m.Get("q")) // ""
fmt.Println(m.Get("item")) // "1" (first value)
fmt.Println(m["item"]) // "[1 2]" (direct map access)
m=nil
fmt.Println(m.Get("item")) // ""
m.Add("item", "3") // panic: assignment to entry in nil map
```

重點在最後一個Get
此時接受器是nil, 這時的結果就像空map
你可以寫成 
```go=
Values(nil).Get("item"))
```
但你不能寫成下面這樣
```go=
nil.Get("item")
```
這樣會無法過complier。因為nil的型別尚未決定（不知道他是Value type)
最後的那行m.Add的調用就會產生一個panic，因為他嘗試更新一個nil map。


## 6.3 嵌入struct 的組合型別
```go=
import "image/color"
type Point struct{ X, Y float64 }
type ColoredPoint struct {
    Point
    Color color.RGBA
}
```

語法捷徑, 可以直接存取Point 裡面的X, Y(4.4.3有教)
```go=
var cp ColoredPoint
cp.X = 1
fmt.Println(cp.Point.X) // "1"
cp.Point.Y = 2
fmt.Println(cp.Y) // "2"
```
就算ColoredPoint 沒有宣告方法, 我們還是有方法可以呼叫Point的method
```go=
red := color.RGBA{255, 0, 0, 255}
blue := color.RGBA{0, 0, 255, 255}
var p = ColoredPoint{Point{1, 1}, red}
var q = ColoredPoint{Point{5, 4}, blue}
fmt.Println(p.Distance(q.Point)) // "5"
p.ScaleBy(2)
q.ScaleBy(2)
fmt.Println(p.Distance(q.Point)) // "10"
```
:::danger
這邊q不是Point (是ColoredPoint), 我們還是得直接選取它
p.Distance(q) // compile error: cannot use q (ColoredPoint) as Point
:::


原因是function 被編譯成如下的形式
: the embedded field instructs the compiler to generate additional wrapper met hods that delegate to the declared methods, equivalent to these:
```go=
func (p ColoredPoint) Distance(q Point) float64 {
    return p.Point.Distance(q)
}
func (p *ColoredPoint) ScaleBy(factor float64) {
    p.Point.ScaleBy(factor)
}
```
不具名欄位(anonymous field)的型別可以是具名型別(named type)的指標
我們可以看到下面例子中可以取用Point 的method



```go=
type ColoredPoint struct {
    *Point
    Color color.RGBA
}
p := ColoredPoint{&Point{1, 1}, red}
q := ColoredPoint{&Point{5, 4}, blue}
fmt.Println(p.Distance(*q.Point)) // "5"
q.Point = p.Point // p and q now share the same Point
p.ScaleBy(2)
fmt.Println(*p.Point, *q.Point) // "{2 2} {2 2}"

```
struct type 可以有很多個 anonymous field(下面的例子2個)
```go=
type ColoredPoint struct {
    Point
    color.RGBA
}
```
然後這個type的值會有Point 的所有method, RGBA的所有方法
還有在ColoredPoint 所宣告的所有方法
遇到p.ScaleBy 之類的selector 怎辦
先找有直接宣告的ScaleBy method, 再來是嵌入ColoredPoint的method
最後是Point 和RGBA這兩個嵌入欄位(embedded fields)的method


****
Methods 只能在 named types (e.g. Point) 和 指向他的指標(*Point)上宣告
但是透過嵌入(embedding)就能夠讓 unnamed struct types 也具有方法

```go=
var (
    mu sync.Mutex // guards mapping
    mapping = make(map[string]string)
)
func Lookup(key string) string {
    mu.Lock()
    v := mapping[key]
    mu.Unlock()
    return v
}
```

The version below is functionally equivalent but groups together the two related variables in a
single package-level variable, **cache**:
```go=
var cache = struct {
    sync.Mutex
    mapping map[string]string
} {
    mapping: make(map[string]string),
}
func Lookup(key string) string {
    cache.Lock()
    v := cache.mapping[key]
    cache.Unlock()
    return v
}
```
新的變數讓跟cache有關的變數名更有意義
由於sync.Mutex欄位嵌入其中
lock 和 unlock 這兩個methods promote 到 **<font color="#f00">unnamed struct type</font>**
讓我們可以用有意義的語法鎖住cache

## 6.4 方法值與運算式

method value, a function that binds a method (Point.Distance) to a specific receiver value p

This function can then be invoked without a receiver value; it needs only the non-receiver arguments.

我們經常選擇一個方法，並且在同一個表達式裡執行，比如常見的p.Distance()形式，實際上將其分成兩步來執行也是可能的。p.Distance叫作“選擇器”，selector會返回一個方法"值"->一個將方法(Point.Distance)绑定到特定接收器variable的函數。這個函數可以不通過指定其接收器即可被使用；即使用時不需要指定接收器(因為已經在之前中指定過了)，只要傳入函數的參數即可：

如下 distanceFromP / scaleP 為 method value
```go=
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance // method value
fmt.Println(distanceFromP(q)) // "5"
var origin Point // {0, 0}
fmt.Println(distanceFromP(origin)) // "2.23606797749979", ;5
scaleP := p.ScaleBy // method value
scaleP(2) // p becomes (2, 4)
scaleP(3) // then (6, 12)
scaleP(10) // then (60, 120)
```

用途:
當API 呼叫 function value 會很有用
正常我們會需要寫成下面的寫法
下面例子, time.AfterFunc function在10秒的延遲後呼叫 **<font color="#f00">function value</font>**

```go=
type Rocket struct { /* ... */ }
func (r *Rocket) Launch() { /* ... */ }
r := new(Rocket)
time.AfterFunc(10 * time.Second, func() { r.Launch() })
```
但改用 **<font color="#f00">method value</font>**, 簡潔很多
```go=
time.AfterFunc(10 * time.Second, r.Launch)
```
方法運算式, 可以透過一般方式呼叫 （distance(p, q)）
Related to the method value is the method expression. When calling a method, as opposed to an ordinary function, we must supply the receiver in a special way using the selector syntax. A ==method expression, written T.f or (*T).f== where T is a type, yields a function value with a regular first parameter taking the place of the receiver, so it can be called in the usual way

```go=
p := Point{1, 2}
q := Point{4, 6}
distance := Point.Distance // method expression
fmt.Println(distance(p, q)) // "5"
fmt.Printf("%T\n", distance) // "func(Point, Point) float64"
scale := (*Point).ScaleBy
scale(&p, 2)
fmt.Println(p) // "{2 4}"
fmt.Printf("%T\n", scale) // "func(*Point, float64)"
```





Method expressions can be helpful when you need a value to represent a choice among several methods belonging to the same type so that you can call the chosen method with many different receivers

同型別的不同方法 很好用

```go=
type Point struct{ X, Y float64 }
func (p Point) Add(q Point) Point { return Point{p.X + q.X, p.Y + q.Y} }
func (p Point) Sub(q Point) Point { return Point{p.X - q.X, p.Y - q.Y} }
type Path []Point
func (path Path) TranslateBy(offset Point, add bool) {
var op func(p, q Point) Point
    if add {
        op = Point.Add
    } else {
        op = Point.Sub
    }
    for i := range path {
        // Call either path[i].Add(offset) or path[i].Sub(offset).
        path[i] = op(path[i], offset)
}
}
```

## 6.5 範例





```go=
// String returns the set as a string of the form "{1 2 3}".
func (s *IntSet) String() string {
    var buf bytes.Buffer
    buf.WriteByte('{')
    for i, word := range s.words {
        if word == 0 {
            continue
        }
        for j := 0; j < 64; j++ {
            if word&(1<<uint(j)) != 0 {
                if buf.Len() > len("{") {
                    buf.WriteByte(' ')
                }
                fmt.Fprintf(&buf, "%d", 64*i+j)
            }
        }
    }
    buf.WriteByte('}')
    return buf.String()
}
```

```go=
var x, y IntSet
x.Add(1)
x.Add(144)
x.Add(9)
fmt.Println(x.String()) // "{1 9 144}"

y.Add(9)
y.Add(42)
fmt.Println(y.String()) // "{9 42}"

x.UnionWith(&y)
fmt.Println(x.String()) // "{1 9 42 144}"
fmt.Println(x.Has(9), x.Has(123)) // "true false"
```
注意
```go=
fmt.Println(&x) // "{1 9 42 144}"
fmt.Println(x.String()) // "{1 9 42 144}"
fmt.Println(x) // "{[4398046511618 0 65536]}"
```


## 6.6 封裝
何謂封裝
A variable or method of an object is said to be encapsulated if it is inaccessible to clients of the object. 

 As a consequence, to encapsulate an object, we must make it a struct.

所以前面的例子才會即使只有一個field 也照樣宣告 struct
```go=
type IntSet struct {
    words []uint64
}
```
s.word只能出現在定義IntSet的套件中


你可以把InSet 定義成slice type
```go=
type IntSet []uint64
```
這樣*s可以在任何套件中被使用

封裝的三個好處
1. 因為用戶不能直接修改object的variables，其只需要關注少量的語句並且只要弄懂少量variables的可能的value即可
2. 隱藏實作的細節, 以免client用到那些會變化的東西（讓設計者有更大的自由來改變implementation而不會破壞API compatibility)

e.g inital 是一個 type [64]byte with an uncapitalized name extra field. 不能匯出
Buffer 的用戶只會感受到perfomance 提升, 對coding不會有任何感受改變
```go=
type Buffer struct {
    buf []byte
    initial [64]byte
/* ... */
}
// Grow expands the buffer's capacity, if necessary,
// to guarantee space for another n bytes. [...]
func (b *Buffer) Grow(n int) {
    if b.buf == nil {
        b.buf = b.initial[:0] // use preallocated space initially
}
    if len(b.buf)+n > cap(b.buf) {
        buf := make([]byte, b.Len(), 2*cap(b.buf) + n)
        copy(buf, b.buf)
        b.buf = buf
    }
}
```


3. 第三個好處 it prevents clients from setting an object’s variables arbitrarily.
e.g. 
下面的Counte Type允許client增加counter variable 的value，並且允許將這個value reset成 0，但是不允許隨便設定這個value
```go=
type Counter struct { n int } 


func (c *Counter) N() int     { return c.n }
func (c *Counter) Increment() { c.n++ }
func (c *Counter) Reset()     { c.n = 0 }
```

when naming a getter method, we usually omit the Get prefix. This preference for brevity extends to all methods, not just field accessors (also Fetch, Find, and Lookup.)

```go=
package log
type Logger struct {
    flags  int
    prefix string
    // ...
}
func (l *Logger) Flags() int
func (l *Logger) SetFlags(flag int)
func (l *Logger) Prefix() string
func (l *Logger) SetPrefix(prefix string)
```

有時封裝不是必要的, 比方說 time.Duration 就reveal int64的毫秒數
讓我們可以拿來做數學運算或是比較的操作
```go=
const day = 24 * time.Hour
fmt.Println(day.Seconds()) // "86400"
```

Type 設計邏輯
| Type |code| 透明 | 原因 |
| -------- | -------- | -------- | -------- |
| geometry.Path    |type Path []Point|Y     |   他的目的就是給一堆點的座標, 預計不會加入新field   |
| IntSet     | type IntSet struct { words []uint64}   | N     |  現在是 []uint64的slice, 但以後也可能用[]uint或是其他更小的type 來表示。也有可能增加額外的 field 來記錄element 數量   |
