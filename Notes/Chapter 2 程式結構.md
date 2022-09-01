## #1 名稱

- 命名以**字母**或**底線**開頭，後接字母、數字或底線。
  - (O) `_` 或 `_1` 或 `a` 或 `a1` 或 `a_`或 `_a` 或 `a_1` 或 `a1_` 或 `_a1` 或 `_1a`
  - (X) `1` 或 `1_` 或 `1a` 或 `1a_` 或 `1_a`
  - 偏好 camel case 而非底線，例如：htmlExcape、HTMLEscape、escapeHTML。
- case sensitivity：大小寫是有區分的。
  - 例如：HeapSort 和 heapSort 是表示不同的名稱。
- 不能作為變數名稱的英文單字：保留字與預先定義
  - 保留字 (reserved word)：break、case、chan、const、continue、default、defer、else、fallthrough、for、func、go、goto、if、import、interface、map、package、range、return、select、struct、switch、type、var，共 25 個。
  - 預先定義 (pre-declared type)，雖然不做保留而可宣告，但小心產生混淆。
    - 常數：true、false、iota、nil
    - 型別：int、int8、int16、int64、unit、unit8、unit16、unit64、uintptr、float32、float64、complex128、complex64、bool、byte、rune、string、error
    - 函式：make、len、cap、new、append、copy、close、delete、complex、real、img、panic、recover
- 變數的可視範圍：local vs global

  - local：函式內宣告，函式內可見。例如：x 宣告在 main 裡面，helloworld 裡面就看不見。

  ```go
  package main
  import "fmt"

  func main() {
    var x = 5
    fmt.Println(x) // 5
  }

  func helloworld() {
    fmt.Println(x) // undefined
  }
  ```

- global：函式外宣告，所屬 package 的全部檔案都可看見。例如：x 宣告在 package 層級，main 和 helloworld 裡面都可以看見。

  ```go
  package main
  import "fmt"
  var x = 5

  func main() {
    fmt.Println(x) // 5
  }

  func hello(){
    fmt.Println(x) // 5
  }
  ```

- 名稱大寫開頭表示本身 package 外可存取。

## #2 宣告

共 4 種宣告 - var、const、type、func。

### 範例 2-1：輸出水的沸點

宣告 1 個常數 (`boilingF`)、 1 個函式 (`main`)、2 個變數 (`f` 和 `c`)

- global: `boilingF`，函式外宣告，所屬 package 的全部檔案都可看見。
- local：`f` 和 `c` 函式內宣告，函式內可見。
- 函式 `main`：包含名稱、參數清單 (opional，在此沒有)、結果清單 (opional，在此沒有)、函式本體。

```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  var f = boilingF
  var c = (f - 32) * 5 / 9
  fmt.Printf("boiling point = %g F or %gC\n", f, c)
}
```

得到輸出結果。

```
boiling point = 212 F or 100C
```

### 範例 2-2：函式封裝華氏與攝氏溫度轉換邏輯

fToc 函式封裝華氏與攝氏溫度轉換邏輯，以便重複使用。

- input：華氏 (float64)。
- output：攝氏 (float64)。

```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  const freezingF, boilingF = 32.0, 212.0

  fmt.Printf("%g F = %g C\n", freezingF, fToc(freezingF))
  fmt.Printf("%g F = %g C\n", boilingF, fToc(boilingF))
}

func fToc(f float64) float64 {
  return (f - 32) * 5 / 9
}
```

得到輸出結果。

```
32 F = 0 C
212 F = 100 C
```

## #3 變數

### 3-1 變數的宣告

宣告一個變數的通用形式，type 和 expression 可擇一省略。

```go
var name type = expression
```

若省略 type，則由 expression 的運算結果決定型別。

例如：變數 happy 的值是字串 happy，印出型別為 string。

```go
var happy = "happy"
fmt.Printf("happy: %s\n", reflect.TypeOf(happy)) // happy: string
```

若省略 expression，則初始值為該型別的零值，因此 Go 沒有 undefined 值，以減少不預期的錯誤。

例如：變數 hello 的型別是 int，沒有給定初始值，因此初始值會是 int 的零值，也就是數字 0。

```go
var hello int;
fmt.Println(hello) // 0
```

型別的零值。

| 型別 | 數字 | 布林  | 字串 | 介面與參考型別`*` | array 與 struct        |
| ---- | ---- | ----- | ---- | ----------------- | ---------------------- |
| 零值 | 0    | false | ""   | nil               | 所有元素或欄位皆為零值 |

`*`包含 slice、pointer、map、channel、func。

### 3-2 一串變數的宣告

一串變數的宣告 i、j 和 k 為 int。

```go
var i, j, k int

fmt.Printf("i: %s\n", reflect.TypeOf(i))
fmt.Printf("j: %s\n", reflect.TypeOf(j))
fmt.Printf("k: %s\n", reflect.TypeOf(k))
```

得到輸出結果。

```
i: int
j: int
k: int
```

一串變數的宣告 b、f、s 和 result，初始值可為實字值或運算式。

```go
var b, f, s, result = true, 2.3, "four", 2*3

fmt.Printf("b: %s\n", reflect.TypeOf(b))
fmt.Printf("f: %s\n", reflect.TypeOf(f))
fmt.Printf("s: %s\n", reflect.TypeOf(s))
fmt.Printf("result: %s\n", reflect.TypeOf(result))
```

得到輸出結果。

```
b: bool
f: float64
s: string
result: int
```

### 3-3 短變數的宣告

短變數的宣告用於宣告並初始化 local variable。

```go
name := expression
```

改寫[前例](#範例-2-1輸出水的沸點)。

```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  f := boilingF
  c := (f - 32) * 5 / 9
  fmt.Printf("boiling point = %g F or %gC\n", f, c)
}
```

得到輸出結果。

```
boiling point = 212 F or 100C
```

短變數的宣告可同時做變數宣告和指派，但至少有一個新的變數宣告。例如：最初先宣告兩個變數 hello 和 world，然後接下來宣告一個新的變數 hi 並且 assign 值給 world，這樣就最少有宣告一個新的變數。

```go
hello, world := "hello", "world"
hi, world := "hi", "hello world"

fmt.Printf("hello = %s\n", hello) // hello
fmt.Printf("world = %s\n", world) // hello world
fmt.Printf("hi = %s\n", hi) // hi
```

得到輸出結果。

```
hello = hello
world = hello world
hi = hi
```

### 3-4 指標

指標指向某個變數，存的是變數的位址。

範例如下

- x 是一個變數，初始值是 1。
- p 是 x 的位址，其值為 int。
- 經由 `*p` 可間接修改 x 的值為 2。

```go
package main

import "fmt"

func main() {
  x := 1
  p := &x // x 的位址，是 int
  fmt.Printf("p = %d\n", p)

  *p = 2
  fmt.Printf("x = %d\n", x) // x === 2
}
```

得到輸出結果。

```
p = 824633835680
x = 2
```

每次宣告變數都會存在不同的位址。

```go
func f() *int {
  v := 1
  return &v
}

fmt.Println(f() == f()) // false
```

得到輸出結果。

```
false
```

利用變數的位址間接修改其值。

範例如下

- v 是一個變數，初始值是 1。
- `&v` 是 v 的位址，其值為 int。
- `*p` 是指這個位址所存的值，也就是 v 存的值。`*p` 是 v 的別名 (alias)，可讓開發者不用知道變數名稱，即可存取變數。
- 經由 `*p` 可間接遞增 x 的值為 2 和 3。

```go
v := 1
fmt.Println(incr(&v)) // 2
fmt.Println(incr(&v)) // 3

func incr(p *int) int {
  *p++
  return *p
}
```

得到輸出結果。

```
2
3
```

利用 flag 套件改寫命令列參數的設定值。

- `-n`：忽略換行符號
- `-s`：sep 的字串會取代空白字元
- `flag.Bool` 函式會建立新的 bool 型別的變數，取用 3 個參數 - flag 的名稱 (n)、變數的預設值 (false) 和說明訊息。
- `flag.String` 函式會建立新的 string 型別的變數，取用 3 個參數 - flag 的名稱 (s)、變數的預設值 (" ") 和說明訊息。
- 變數 n 與 sep 是 flag 變數的指標，`*n` 與 `*sep` 可存取其值。

```go
package main

import (
  "flag"
  "fmt"
  "strings"
)

var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "seperator")

func main() {
  flag.Parse() // 將預設值更新
  fmt.Print(strings.Join(flag.Args(), *sep)) // 將新的 seperator 帶入空白、組成字串

  if !*n {
    fmt.Println(); // 輸出換行
  }
}
```

執行。

```
$ ./echo a bc def
```

得到輸出結果。

```
a bc def
```

執行，seperator 設定為 `/`。

```
$ ./echo -s / a bc def
```

得到輸出結果。

```
a/bc/def
```

備註：執行 `./echo -s / a bc def`

- `*sep` 的值是 `/`
- `flag.Args()` 的值是 `[a bc def]`
- `strings.Join(flag.Args(), *sep)` 的值是 `a/bc/def`

執行。

```
$ ./echo -n=true a bc def
```

得到輸出結果，不換行。

```
a bc def$
```

執行。

```
./echo -h
```

得到說明訊息。

```
Usage of ./echo:
  -n    omit trailing newline
  -s string
        seperator (default " ")
```

### 3-5 new 函式

利用內建的 new 函式建立變數。

範例如下，`new(T)` 表示以 T 型別建立變數，預設值是 T 的零值，回傳 `*T` 作為位址。

```go
p := new(T)
```

```go
package main

import "fmt"

func main() {
  p := new(int)
  fmt.Println(*p) // 0
  *p = 1;
  fmt.Println(*p) // 1
}
```

得到輸出結果。

```
0
1
```

由以上範例可知，以下兩種表示方法是相同的。

```go
func newInt1() *int {
  return new(int)
}
```

```go
func newInt2() *int {
  var dummy int
  return &dummy
}
```

通常不同變數的位置會不同，但若變數沒有資訊且大小為零，則位置會相同。

```go
arr1 := [1]int{0}
arr2 := [1]int{0}
fmt.Println(arr1 == arr2)
```

得到輸出結果。

```
true
```

### 3-6 變數的生命週期

變數的生命週期是指變數存在的時間。

#### global vs local

- global variable：程式整個執行時間，預設狀況下，global variable 會保存在 heap memory 中。
- local variable：宣告時重新建立，當不可觸及時被回收。**不可觸及**是指有沒有人要引用它。預設狀況下，local variable 會保存在 stack memory 中。

#### heap vs stack

- x 分配在 heap，因為函式回傳後 global 依然可以參考到 x。
- y 分配在 stack，因為函式回傳後，y 即為不可觸及。
- 注意全域變數保存不必要的短生命週期物件指標會阻止 gc 回收記憶體，在效能最佳化時請考慮這一點。

```go
var global *int

func f() {
  var x int
  x = 1
  global = &x
}

func g() {
  y := new(int)
  *y = 1
}
```

## #4 指派

### 4-1 資料組指派

資料組指派是指一次指派多個變數。

#### 範例：用於交換或變數同時出現在指派雙方

```go
x, y = y, x
```

#### 範例：精簡指派

```go
i, j, k = 2, 3, 4
```

#### 範例：指派不要的值給空識別符號

```go
_, err = io.Copy(dst, src)
_, ok = x.(T)
```

[NOTE] 空識別符號 `_` 是一個佔位符，用於賦值操作的時候，丟棄、忽略某個值。

### 4-2 可指派性

如下範例，slice 這種組合型別的實字運算式間接指派每個元素的值。

也就是說，當值與變數的型別是可指派的，才是合法的。

```go
package main

import "fmt"

func main() {
  medals := []string{"gold", "silver", "bronze"}

  fmt.Println(medals[0])
  fmt.Println(medals[1])
  fmt.Println(medals[2])
}
```

得到輸出結果。

```
gold
silver
bronze
```

i 重新指派字串就是不合法的。

```go
i := 1;
i = "hello world";
```

## #5 型別宣告

### 具名型別

- 「具名型別」提供分離底層型別的新的具名型別，可避免與底層型別混淆。
- 利用 `type` 宣告一個新的具名型別，格式：`type name underlying-type`。

### 範例：型別轉換-攝氏與華氏的溫度轉換

說明：

- 定義 2 個型別：Celsius 與 Fahrenheit 作為攝氏與華氏的溫度單位，其底層型別都是 float64。
- Celsius 與 Fahrenheit 是不同的型別，因此不能互相比較或是交互進行運算。若要進行型別轉換，則要使用 `Celsius(t)` 或 `Fahrenheit（t）` 做明確的型別轉換。
- 函式 `CToF` 與 `FToC` 用於兩種單位的計算轉換，分別回傳型別為 Fahrenheit 和 Celsius 的計算結果。

```go
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
  AbsoluteZero Celsius = -273.15
  FreezingC    Celsius = 0
  BoilingC     Celsius = 100
)

func CToF(c Celsius) Fahrenheit {
  return Fahrenheit(c*9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
  return Celsius((f - 32) * 5 / 9)
}
```

說明：

- (1) boilingF 的型別是 boilingF，而 FreezingC 的型別是 Celsius，型別不同，出現編譯錯誤。
- (2) c 與 f 型別不同，出現編譯錯誤。
- (3) 強制轉換 f 的型別為 Celsius，型別轉換後值沒有改變，c 與 f 都是 0，因此相等。

```go
fmt.Printf("%g\n", BoilingC-FreezingC) // 100
boilingF := CToF(BoilingC)
fmt.Printf("%g\n", boilingF-CToF(FreezingC)) // 180
fmt.Printf("%g\n", boilingF-FreezingC) // mismatched types Fahrenheit and Celsius - (1)

var c Celsius
var f Fahrenheit
fmt.Println(c == 0) // true
fmt.Println(f >= 0) // true
fmt.Println(c == f) // mismatched types Celsius and Fahrenheit  - (2)
fmt.Println(c == Celsius(f)) // true - (3)
```

得到輸出結果。

```
100
180
mismatched types Fahrenheit and Celsius
true
true
mismatched types Celsius and Fahrenheit
true
```

### 範例：定義具名型別的方法-加上單位字串

說明：

- 宣告 String 方法，就能控制 fmt 輸出時要怎麼顯示。
- `%g` 印出浮點數，不會呼叫 String。
- `float64` 是轉換為浮點數，不會呼叫 String。

```go
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }

c := FToC(212.0)
fmt.Println(c.String()) // 100°C
fmt.Println("%v\n", c)  // 100°C
fmt.Println("%s\n", c)  // 100°C
fmt.Println(c)          // 100°C
fmt.Println("%g\n", c)  // 100，預計是不會呼叫 String，但實際執行得到 "100°C"
fmt.Println(float64(c)) // 100，不會呼叫 String
```

得到輸出結果。

```
100°C
100°C
100°C
100°C
100
100
```

## #6 套件與檔案

- Golang 的套件等同於其他語言的函式庫或模組，可用於模組化、封裝、個別編譯、重用。
- 透過 `go.mod` 管理 package。
- 使用套件的函式要明確指出套件和函式名，例如：`image.Decode` 或 `utf16.Decode`。
- 匯出套件的識別字要以大寫字母開頭，例如：`PrintHello`。

### 範例：套件管理

```
- hello/
   - main.go
   - go.mod
   - hello/
       - hello.go
```

說明

- 在 hello 底下執行 `go mod init hello`。

```go
module hello

go 1.19
```

- 撰寫 hello package。

```go
package hello

import "fmt"
func PrintHello() {
    fmt.Println("hello world")
}
```

- 使用 hello 的 `PrintHello()` -> 執行 `go run main.go`。

```go
package main

import "hello/hello"

func main(){
    hello.PrintHello()
}
```

得到輸出結果。

```
hello world
```

### 範例：攝氏與華氏的溫度轉換的套件

定義 2 個檔案：`tempconv.go` 和 `conv.go`。

`tempconv.go` 放置型別、常數、方法的宣告。

```go
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
  AbsoluteZero Celsius = -273.15
  FreezingC    Celsius = 0
  BoilingC     Celsius = 100
)

func CToF(c Celsius) Fahrenheit {
  return Fahrenheit(c*9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
  return Celsius((f - 32) * 5 / 9)
}

func (c Celsius) String() string    { return fmt.Sprintf("%g°C", c) }
func (f Fahrenheit) String() string { return fmt.Sprintf("%g°F", f) }
```

`conv.go` 放置轉換函式。

```go
package tempconv

func CToF(c Celsius) Fahrenheit {
  return Fahrenheit(c*9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
  return Celsius((f - 32) * 5 / 9)
}
```

印出攝氏沸點轉成華氏的值。

```go
package main

import (
  "fmt"
  "tempconv/tempconv"
)

func main(){
  fmt.Println(tempconv.CToF(tempconv.BoilingC)) // 212°F
}
```

利用匯入路徑來識別套件，例如：`tempconv/tempconv`，最後一段字串是套件名稱。

### 練習 2-1

對 tempconv 加上型別、常數與函式以處理克耳文單位，克耳文零度等於 -273.15 攝氏度，1K 的溫度等於 1°C 的溫差。

`tempconv.go` 放置型別、常數、方法的宣告。

```go
package tempconv

import "fmt"

type Celsius float64
type KELVIN float64

const (
	AbsoluteZeroK KELVIN = -273.15
)
```

`conv.go` 放置轉換函式。

```go
package tempconv

func CToK(c Celsius) KELVIN {
  return KELVIN(c)
}

func KToC(k KELVIN) Celsius {
  return Celsius(k)
}
```

印出 K 氏絕對零度轉成攝氏的值。

```go
package main

import (
  "fmt"
  "tempconv/tempconv"
)

func main(){
  fmt.Println(tempconv.KToC(tempconv.AbsoluteZeroK)) // -273.15°C
}
```

### 練習 2-2 公斤與磅的轉換

`weightconv.go` 放置型別、常數、方法的宣告，這裡定義大貓和小貓的體重當成兩個常數。

```go
package weightconv

import "fmt"

type Kilo float64
type Pound float64

const (
	SmallCat Kilo = 2
	BigCat Pound = 50
)

func (k Kilo) String() string    { return fmt.Sprintf("%g Kg", k) }
func (p Pound) String() string { return fmt.Sprintf("%g Pound", p) }
```

`conv.go` 放置轉換函式。

```go
package weightconv

// 1 kg = 2.20462262 pound
func KToP(k Kilo) Pound {
  return Pound(k* 2.20462262)
}

// 1 pound = 0.45359237 kg
func PToK(p Pound) Kilo {
  return Kilo(p* 0.45359237)
}
```

印出小貓與大貓的體重轉換。

```go
package main

import (
    "fmt"
    "weightconv/weightconv"
)

func main(){
    fmt.Println(weightconv.KToP(weightconv.SmallCat)) // 2 Kilo = 4.40924524 Pound
    fmt.Println(weightconv.PToK(weightconv.BigCat)) // 50 Pound = 22.6796185 Kg
}
```

### 套件初始化

- 套件初始化的順序：套件層級的變數 -> 解決相依性 -> 宣告順序。
- 套件若有多個檔案，編譯器會先以檔名排序再做初始化。

### 範例：人口計算

說明

- 利用 init 先建立表格 pc，然後再用查表的方式算出 PopCount。
- pc 是一個 array，大小是 256 bytes。
- init 會先初始化以計算結果，算出 pc 內每個元素的值，例如：`pc[0]=0`，`pc[1]=1`，`pc[255]=8`。
- PopCount 計算個別 pc 的值，i.e., 把每個 byte 裡面的數字分別加總。例如：`PopCount(254)` 表示 x 代入 254
  - 若 x = 254，以二進位表示即是 `00000000 00000000 00000000 00000000 00000000 00000000 00000000 11111110`
  - `pc[byte(x>>(0*8))]` 是 `pc[254]`，`pc[byte(x>>(1*8))] ~ pc[byte(x>>(7*8))]` 皆為 `pc[0]`，因此 `int(pc[byte(x>>(0*8))] + pc[byte(x>>(1*8))] +pc[byte(x>>(2*8))] + pc[byte(x>>(3*8))] + pc[byte(x>>(4*8))] + pc[byte(x>>(5*8))] + pc[byte(x>>(6*8))] + pc[byte(x>>(7*8))])` 等於 `pc[254] + pc[0] + pc[0] + pc[0] + pc[0] + pc[0] + pc[0] = 7 + 0 + 0 + 0 + 0 + 0 + 0 + 0 = 7`

```go
package popcount

var pc [256]byte 

func init() {
  for i := range pc {
    pc[i] = pc[i/2] + byte(i&1) // 對 `00000001` 做遮罩
  }
}

func PopCount(x uint64) int {
  return int(pc[byte(x>>(0*8))] +
    pc[byte(x>>(1*8))] +
    pc[byte(x>>(2*8))] +
    pc[byte(x>>(3*8))] +
    pc[byte(x>>(4*8))] +
    pc[byte(x>>(5*8))] +
    pc[byte(x>>(6*8))] +
    pc[byte(x>>(7*8))])
}
```

init 計算 `pc[0]` ~ `pc[255]` 的值。

```bash
i = 0, i/2 = 0, i&1 = 0, pc[0] = 0
i = 1, i/2 = 0, i&1 = 1, pc[1] = 1
i = 2, i/2 = 1, i&1 = 0, pc[2] = 1
i = 3, i/2 = 1, i&1 = 1, pc[3] = 2
i = 4, i/2 = 2, i&1 = 0, pc[4] = 1
i = 5, i/2 = 2, i&1 = 1, pc[5] = 2
i = 6, i/2 = 3, i&1 = 0, pc[6] = 2
i = 7, i/2 = 3, i&1 = 1, pc[7] = 3
i = 8, i/2 = 4, i&1 = 0, pc[8] = 1
i = 9, i/2 = 4, i&1 = 1, pc[9] = 2
i = 10, i/2 = 5, i&1 = 0, pc[10] = 2
i = 11, i/2 = 5, i&1 = 1, pc[11] = 3
i = 12, i/2 = 6, i&1 = 0, pc[12] = 2
i = 13, i/2 = 6, i&1 = 1, pc[13] = 3
i = 14, i/2 = 7, i&1 = 0, pc[14] = 3
i = 15, i/2 = 7, i&1 = 1, pc[15] = 4
i = 16, i/2 = 8, i&1 = 0, pc[16] = 1
i = 17, i/2 = 8, i&1 = 1, pc[17] = 2
i = 18, i/2 = 9, i&1 = 0, pc[18] = 2
i = 19, i/2 = 9, i&1 = 1, pc[19] = 3
i = 20, i/2 = 10, i&1 = 0, pc[20] = 2
i = 21, i/2 = 10, i&1 = 1, pc[21] = 3
i = 22, i/2 = 11, i&1 = 0, pc[22] = 3
i = 23, i/2 = 11, i&1 = 1, pc[23] = 4
i = 24, i/2 = 12, i&1 = 0, pc[24] = 2
i = 25, i/2 = 12, i&1 = 1, pc[25] = 3
i = 26, i/2 = 13, i&1 = 0, pc[26] = 3
i = 27, i/2 = 13, i&1 = 1, pc[27] = 4
i = 28, i/2 = 14, i&1 = 0, pc[28] = 3
i = 29, i/2 = 14, i&1 = 1, pc[29] = 4
i = 30, i/2 = 15, i&1 = 0, pc[30] = 4
i = 31, i/2 = 15, i&1 = 1, pc[31] = 5
i = 32, i/2 = 16, i&1 = 0, pc[32] = 1
i = 33, i/2 = 16, i&1 = 1, pc[33] = 2
i = 34, i/2 = 17, i&1 = 0, pc[34] = 2
i = 35, i/2 = 17, i&1 = 1, pc[35] = 3
i = 36, i/2 = 18, i&1 = 0, pc[36] = 2
i = 37, i/2 = 18, i&1 = 1, pc[37] = 3
i = 38, i/2 = 19, i&1 = 0, pc[38] = 3
i = 39, i/2 = 19, i&1 = 1, pc[39] = 4
i = 40, i/2 = 20, i&1 = 0, pc[40] = 2
i = 41, i/2 = 20, i&1 = 1, pc[41] = 3
i = 42, i/2 = 21, i&1 = 0, pc[42] = 3
i = 43, i/2 = 21, i&1 = 1, pc[43] = 4
i = 44, i/2 = 22, i&1 = 0, pc[44] = 3
i = 45, i/2 = 22, i&1 = 1, pc[45] = 4
i = 46, i/2 = 23, i&1 = 0, pc[46] = 4
i = 47, i/2 = 23, i&1 = 1, pc[47] = 5
i = 48, i/2 = 24, i&1 = 0, pc[48] = 2
i = 49, i/2 = 24, i&1 = 1, pc[49] = 3
i = 50, i/2 = 25, i&1 = 0, pc[50] = 3
i = 51, i/2 = 25, i&1 = 1, pc[51] = 4
i = 52, i/2 = 26, i&1 = 0, pc[52] = 3
i = 53, i/2 = 26, i&1 = 1, pc[53] = 4
i = 54, i/2 = 27, i&1 = 0, pc[54] = 4
i = 55, i/2 = 27, i&1 = 1, pc[55] = 5
i = 56, i/2 = 28, i&1 = 0, pc[56] = 3
i = 57, i/2 = 28, i&1 = 1, pc[57] = 4
i = 58, i/2 = 29, i&1 = 0, pc[58] = 4
i = 59, i/2 = 29, i&1 = 1, pc[59] = 5
i = 60, i/2 = 30, i&1 = 0, pc[60] = 4
i = 61, i/2 = 30, i&1 = 1, pc[61] = 5
i = 62, i/2 = 31, i&1 = 0, pc[62] = 5
i = 63, i/2 = 31, i&1 = 1, pc[63] = 6
i = 64, i/2 = 32, i&1 = 0, pc[64] = 1
i = 65, i/2 = 32, i&1 = 1, pc[65] = 2
i = 66, i/2 = 33, i&1 = 0, pc[66] = 2
i = 67, i/2 = 33, i&1 = 1, pc[67] = 3
i = 68, i/2 = 34, i&1 = 0, pc[68] = 2
i = 69, i/2 = 34, i&1 = 1, pc[69] = 3
i = 70, i/2 = 35, i&1 = 0, pc[70] = 3
i = 71, i/2 = 35, i&1 = 1, pc[71] = 4
i = 72, i/2 = 36, i&1 = 0, pc[72] = 2
i = 73, i/2 = 36, i&1 = 1, pc[73] = 3
i = 74, i/2 = 37, i&1 = 0, pc[74] = 3
i = 75, i/2 = 37, i&1 = 1, pc[75] = 4
i = 76, i/2 = 38, i&1 = 0, pc[76] = 3
i = 77, i/2 = 38, i&1 = 1, pc[77] = 4
i = 78, i/2 = 39, i&1 = 0, pc[78] = 4
i = 79, i/2 = 39, i&1 = 1, pc[79] = 5
i = 80, i/2 = 40, i&1 = 0, pc[80] = 2
i = 81, i/2 = 40, i&1 = 1, pc[81] = 3
i = 82, i/2 = 41, i&1 = 0, pc[82] = 3
i = 83, i/2 = 41, i&1 = 1, pc[83] = 4
i = 84, i/2 = 42, i&1 = 0, pc[84] = 3
i = 85, i/2 = 42, i&1 = 1, pc[85] = 4
i = 86, i/2 = 43, i&1 = 0, pc[86] = 4
i = 87, i/2 = 43, i&1 = 1, pc[87] = 5
i = 88, i/2 = 44, i&1 = 0, pc[88] = 3
i = 89, i/2 = 44, i&1 = 1, pc[89] = 4
i = 90, i/2 = 45, i&1 = 0, pc[90] = 4
i = 91, i/2 = 45, i&1 = 1, pc[91] = 5
i = 92, i/2 = 46, i&1 = 0, pc[92] = 4
i = 93, i/2 = 46, i&1 = 1, pc[93] = 5
i = 94, i/2 = 47, i&1 = 0, pc[94] = 5
i = 95, i/2 = 47, i&1 = 1, pc[95] = 6
i = 96, i/2 = 48, i&1 = 0, pc[96] = 2
i = 97, i/2 = 48, i&1 = 1, pc[97] = 3
i = 98, i/2 = 49, i&1 = 0, pc[98] = 3
i = 99, i/2 = 49, i&1 = 1, pc[99] = 4
i = 100, i/2 = 50, i&1 = 0, pc[100] = 3
i = 101, i/2 = 50, i&1 = 1, pc[101] = 4
i = 102, i/2 = 51, i&1 = 0, pc[102] = 4
i = 103, i/2 = 51, i&1 = 1, pc[103] = 5
i = 104, i/2 = 52, i&1 = 0, pc[104] = 3
i = 105, i/2 = 52, i&1 = 1, pc[105] = 4
i = 106, i/2 = 53, i&1 = 0, pc[106] = 4
i = 107, i/2 = 53, i&1 = 1, pc[107] = 5
i = 108, i/2 = 54, i&1 = 0, pc[108] = 4
i = 109, i/2 = 54, i&1 = 1, pc[109] = 5
i = 110, i/2 = 55, i&1 = 0, pc[110] = 5
i = 111, i/2 = 55, i&1 = 1, pc[111] = 6
i = 112, i/2 = 56, i&1 = 0, pc[112] = 3
i = 113, i/2 = 56, i&1 = 1, pc[113] = 4
i = 114, i/2 = 57, i&1 = 0, pc[114] = 4
i = 115, i/2 = 57, i&1 = 1, pc[115] = 5
i = 116, i/2 = 58, i&1 = 0, pc[116] = 4
i = 117, i/2 = 58, i&1 = 1, pc[117] = 5
i = 118, i/2 = 59, i&1 = 0, pc[118] = 5
i = 119, i/2 = 59, i&1 = 1, pc[119] = 6
i = 120, i/2 = 60, i&1 = 0, pc[120] = 4
i = 121, i/2 = 60, i&1 = 1, pc[121] = 5
i = 122, i/2 = 61, i&1 = 0, pc[122] = 5
i = 123, i/2 = 61, i&1 = 1, pc[123] = 6
i = 124, i/2 = 62, i&1 = 0, pc[124] = 5
i = 125, i/2 = 62, i&1 = 1, pc[125] = 6
i = 126, i/2 = 63, i&1 = 0, pc[126] = 6
i = 127, i/2 = 63, i&1 = 1, pc[127] = 7
i = 128, i/2 = 64, i&1 = 0, pc[128] = 1
i = 129, i/2 = 64, i&1 = 1, pc[129] = 2
i = 130, i/2 = 65, i&1 = 0, pc[130] = 2
i = 131, i/2 = 65, i&1 = 1, pc[131] = 3
i = 132, i/2 = 66, i&1 = 0, pc[132] = 2
i = 133, i/2 = 66, i&1 = 1, pc[133] = 3
i = 134, i/2 = 67, i&1 = 0, pc[134] = 3
i = 135, i/2 = 67, i&1 = 1, pc[135] = 4
i = 136, i/2 = 68, i&1 = 0, pc[136] = 2
i = 137, i/2 = 68, i&1 = 1, pc[137] = 3
i = 138, i/2 = 69, i&1 = 0, pc[138] = 3
i = 139, i/2 = 69, i&1 = 1, pc[139] = 4
i = 140, i/2 = 70, i&1 = 0, pc[140] = 3
i = 141, i/2 = 70, i&1 = 1, pc[141] = 4
i = 142, i/2 = 71, i&1 = 0, pc[142] = 4
i = 143, i/2 = 71, i&1 = 1, pc[143] = 5
i = 144, i/2 = 72, i&1 = 0, pc[144] = 2
i = 145, i/2 = 72, i&1 = 1, pc[145] = 3
i = 146, i/2 = 73, i&1 = 0, pc[146] = 3
i = 147, i/2 = 73, i&1 = 1, pc[147] = 4
i = 148, i/2 = 74, i&1 = 0, pc[148] = 3
i = 149, i/2 = 74, i&1 = 1, pc[149] = 4
i = 150, i/2 = 75, i&1 = 0, pc[150] = 4
i = 151, i/2 = 75, i&1 = 1, pc[151] = 5
i = 152, i/2 = 76, i&1 = 0, pc[152] = 3
i = 153, i/2 = 76, i&1 = 1, pc[153] = 4
i = 154, i/2 = 77, i&1 = 0, pc[154] = 4
i = 155, i/2 = 77, i&1 = 1, pc[155] = 5
i = 156, i/2 = 78, i&1 = 0, pc[156] = 4
i = 157, i/2 = 78, i&1 = 1, pc[157] = 5
i = 158, i/2 = 79, i&1 = 0, pc[158] = 5
i = 159, i/2 = 79, i&1 = 1, pc[159] = 6
i = 160, i/2 = 80, i&1 = 0, pc[160] = 2
i = 161, i/2 = 80, i&1 = 1, pc[161] = 3
i = 162, i/2 = 81, i&1 = 0, pc[162] = 3
i = 163, i/2 = 81, i&1 = 1, pc[163] = 4
i = 164, i/2 = 82, i&1 = 0, pc[164] = 3
i = 165, i/2 = 82, i&1 = 1, pc[165] = 4
i = 166, i/2 = 83, i&1 = 0, pc[166] = 4
i = 167, i/2 = 83, i&1 = 1, pc[167] = 5
i = 168, i/2 = 84, i&1 = 0, pc[168] = 3
i = 169, i/2 = 84, i&1 = 1, pc[169] = 4
i = 170, i/2 = 85, i&1 = 0, pc[170] = 4
i = 171, i/2 = 85, i&1 = 1, pc[171] = 5
i = 172, i/2 = 86, i&1 = 0, pc[172] = 4
i = 173, i/2 = 86, i&1 = 1, pc[173] = 5
i = 174, i/2 = 87, i&1 = 0, pc[174] = 5
i = 175, i/2 = 87, i&1 = 1, pc[175] = 6
i = 176, i/2 = 88, i&1 = 0, pc[176] = 3
i = 177, i/2 = 88, i&1 = 1, pc[177] = 4
i = 178, i/2 = 89, i&1 = 0, pc[178] = 4
i = 179, i/2 = 89, i&1 = 1, pc[179] = 5
i = 180, i/2 = 90, i&1 = 0, pc[180] = 4
i = 181, i/2 = 90, i&1 = 1, pc[181] = 5
i = 182, i/2 = 91, i&1 = 0, pc[182] = 5
i = 183, i/2 = 91, i&1 = 1, pc[183] = 6
i = 184, i/2 = 92, i&1 = 0, pc[184] = 4
i = 185, i/2 = 92, i&1 = 1, pc[185] = 5
i = 186, i/2 = 93, i&1 = 0, pc[186] = 5
i = 187, i/2 = 93, i&1 = 1, pc[187] = 6
i = 188, i/2 = 94, i&1 = 0, pc[188] = 5
i = 189, i/2 = 94, i&1 = 1, pc[189] = 6
i = 190, i/2 = 95, i&1 = 0, pc[190] = 6
i = 191, i/2 = 95, i&1 = 1, pc[191] = 7
i = 192, i/2 = 96, i&1 = 0, pc[192] = 2
i = 193, i/2 = 96, i&1 = 1, pc[193] = 3
i = 194, i/2 = 97, i&1 = 0, pc[194] = 3
i = 195, i/2 = 97, i&1 = 1, pc[195] = 4
i = 196, i/2 = 98, i&1 = 0, pc[196] = 3
i = 197, i/2 = 98, i&1 = 1, pc[197] = 4
i = 198, i/2 = 99, i&1 = 0, pc[198] = 4
i = 199, i/2 = 99, i&1 = 1, pc[199] = 5
i = 200, i/2 = 100, i&1 = 0, pc[200] = 3
i = 201, i/2 = 100, i&1 = 1, pc[201] = 4
i = 202, i/2 = 101, i&1 = 0, pc[202] = 4
i = 203, i/2 = 101, i&1 = 1, pc[203] = 5
i = 204, i/2 = 102, i&1 = 0, pc[204] = 4
i = 205, i/2 = 102, i&1 = 1, pc[205] = 5
i = 206, i/2 = 103, i&1 = 0, pc[206] = 5
i = 207, i/2 = 103, i&1 = 1, pc[207] = 6
i = 208, i/2 = 104, i&1 = 0, pc[208] = 3
i = 209, i/2 = 104, i&1 = 1, pc[209] = 4
i = 210, i/2 = 105, i&1 = 0, pc[210] = 4
i = 211, i/2 = 105, i&1 = 1, pc[211] = 5
i = 212, i/2 = 106, i&1 = 0, pc[212] = 4
i = 213, i/2 = 106, i&1 = 1, pc[213] = 5
i = 214, i/2 = 107, i&1 = 0, pc[214] = 5
i = 215, i/2 = 107, i&1 = 1, pc[215] = 6
i = 216, i/2 = 108, i&1 = 0, pc[216] = 4
i = 217, i/2 = 108, i&1 = 1, pc[217] = 5
i = 218, i/2 = 109, i&1 = 0, pc[218] = 5
i = 219, i/2 = 109, i&1 = 1, pc[219] = 6
i = 220, i/2 = 110, i&1 = 0, pc[220] = 5
i = 221, i/2 = 110, i&1 = 1, pc[221] = 6
i = 222, i/2 = 111, i&1 = 0, pc[222] = 6
i = 223, i/2 = 111, i&1 = 1, pc[223] = 7
i = 224, i/2 = 112, i&1 = 0, pc[224] = 3
i = 225, i/2 = 112, i&1 = 1, pc[225] = 4
i = 226, i/2 = 113, i&1 = 0, pc[226] = 4
i = 227, i/2 = 113, i&1 = 1, pc[227] = 5
i = 228, i/2 = 114, i&1 = 0, pc[228] = 4
i = 229, i/2 = 114, i&1 = 1, pc[229] = 5
i = 230, i/2 = 115, i&1 = 0, pc[230] = 5
i = 231, i/2 = 115, i&1 = 1, pc[231] = 6
i = 232, i/2 = 116, i&1 = 0, pc[232] = 4
i = 233, i/2 = 116, i&1 = 1, pc[233] = 5
i = 234, i/2 = 117, i&1 = 0, pc[234] = 5
i = 235, i/2 = 117, i&1 = 1, pc[235] = 6
i = 236, i/2 = 118, i&1 = 0, pc[236] = 5
i = 237, i/2 = 118, i&1 = 1, pc[237] = 6
i = 238, i/2 = 119, i&1 = 0, pc[238] = 6
i = 239, i/2 = 119, i&1 = 1, pc[239] = 7
i = 240, i/2 = 120, i&1 = 0, pc[240] = 4
i = 241, i/2 = 120, i&1 = 1, pc[241] = 5
i = 242, i/2 = 121, i&1 = 0, pc[242] = 5
i = 243, i/2 = 121, i&1 = 1, pc[243] = 6
i = 244, i/2 = 122, i&1 = 0, pc[244] = 5
i = 245, i/2 = 122, i&1 = 1, pc[245] = 6
i = 246, i/2 = 123, i&1 = 0, pc[246] = 6
i = 247, i/2 = 123, i&1 = 1, pc[247] = 7
i = 248, i/2 = 124, i&1 = 0, pc[248] = 5
i = 249, i/2 = 124, i&1 = 1, pc[249] = 6
i = 250, i/2 = 125, i&1 = 0, pc[250] = 6
i = 251, i/2 = 125, i&1 = 1, pc[251] = 7
i = 252, i/2 = 126, i&1 = 0, pc[252] = 6
i = 253, i/2 = 126, i&1 = 1, pc[253] = 7
i = 254, i/2 = 127, i&1 = 0, pc[254] = 7
i = 255, i/2 = 127, i&1 = 1, pc[255] = 8
```

| #     | PopCount          |
|-------|-------------------|
| x = 0   | PopCount(0) = 0   |
| x = 1   | PopCount(1) = 1   |
| x = 2   | PopCount(2) = 1   |
| x = 3   | PopCount(3) = 2   |
| x = 254 | PopCount(254) = 7 |
| x = 255 | PopCount(255) = 8 |

利用 timestamp 計算花多少時間。

如下範例 `284671 - 284461  = 210 (micro seconds)` (file: popcount.go)。

```go
package main

import (
  "fmt"
  "time"
)

var pc [256]byte

func test() {
  for i := range pc {
    pc[i] = pc[i/2] + byte(i&1)
  }
}

func PopCount(x uint64) int {
  return int(pc[byte(x>>(0*8))] +
    pc[byte(x>>(1*8))] +
    pc[byte(x>>(2*8))] +
    pc[byte(x>>(3*8))] +
    pc[byte(x>>(4*8))] +
    pc[byte(x>>(5*8))] +
    pc[byte(x>>(6*8))] +
    pc[byte(x>>(7*8))])
}

func main() {
  test()
  fmt.Println(time.Now()) // 2022-08-31 14:26:14.284461 +0800 CST m=+0.000095479
  PopCount(255)
  fmt.Println(time.Now()) // 2022-08-31 14:26:14.284671 +0800 CST m=+0.000305969
}
```

### 練習 2-3 以單一運算式改寫 PopCount

我認為單一運算式就是查表的意思，以下略過。

利用 timestamp 計算花多少時間。

如下範例 `215806  - 215604 = 202 (micro seconds)` (file popcount_2_3.go)。

```go
package main

import (
  "fmt"
  "time"
)

var pc [256]byte

func init() {
  for i := range pc {
    pc[i] = pc[i/2] + byte(i&1)
  }
}

func PopCount(x uint64) int {
  count := 0
  
	for i := uint64(0); i < 8; i++ {
		count += int(pc[byte(x>>(i*8))])
	}
	return count
}

func main() {
  fmt.Println(time.Now()) // 2022-08-31 15:30:45.215604 +0800 CST m=+0.000100824
  PopCount(255)
  fmt.Println(time.Now()) // 2022-08-31 15:30:45.215806 +0800 CST m=+0.000302278
}
```

### 練習 2-4 以位移 64 次的方式改寫 PopCount

利用 timestamp 計算花多少時間。

如下範例 `114736 - 114936 = 200 (micro seconds)` (file popcount_2_4.go)。

```go
package main

import (
  "fmt"
  "time"
)

var pc [256]byte

func init() {
  for i := range pc {
    pc[i] = pc[i/2] + byte(i&1)
  }
}

func PopCount(x uint64) int {
  var counter uint64 = 0

  for i := 0; i < 64; i++ {
    counter += (x >> i) & 0x1
  }

  return int(counter)
}

func main() {
  fmt.Println(time.Now()) // 2022-08-31 14:29:03.114736 +0800 CST m=+0.000097190
  PopCount(255)
  fmt.Println(time.Now()) // 2022-08-31 14:29:03.114936 +0800 CST m=+0.000297015
}
```

### 練習 2-5 以 `x&(x-1)` 改寫 PopCount

利用 timestamp 計算花多少時間。

如下範例 `896941  - 896757 = 184 (micro seconds)` (file popcount_2_5.go)。

```go
package main

import (
  "fmt"
  "time"
)

var pc [256]byte

func init() {
  for i := range pc {
    pc[i] = pc[i/2] + byte(i&1)
  }
}

func PopCount(x uint64) int {
  counter := 0

  for i := 0; i < 64; i++ {
    x = x&(x-1)

    if (x != 0) {
      counter++
    }
  }

  return counter
}

func main() {
  fmt.Println(time.Now()) // 2022-08-31 15:38:16.896757 +0800 CST m=+0.000094159
  PopCount(255)
  fmt.Println(time.Now()) // 2022-08-31 15:38:16.896941 +0800 CST m=+0.000278211
}
```

| #     | 花費時間 (micro seconds)          |
|-------|-----|
| 2-3 查表 |  210 |
| 2-4 位移 64 次 | 200 (直接位移 64 bit 所以比較快) |
| 2-5 刪掉最右邊非零 bit | 184 (運算元更少，最快)  |

## #7 範圍

### 名詞解釋

- 範圍 vs 生命週期：
  - 範圍：變數的宣告的範圍是指程式的文字區域，是編譯期的屬性。
  - 生命週期：變數的生命週期是指可被程式參考的時間範圍，是執行期的屬性。
- 語句 (syntatic) 區塊是指包圍在括弧內 (`{...}`) 的陳述，其宣告在括弧外不可見，這是範圍。
- 詞彙 (lexical) 區塊是指變數可存取的 parent scope 或巢狀函式的範圍。

### 範例：詞彙區塊

在自己所在的括弧內找不到值，就會往上一層尋找，順序是 local -> global。

```go
package main

import "fmt"

func f() {}

var g = "g"

func main() {
  f := "f"
  fmt.Println(f) // f，在內層宣告找到
  fmt.Println(g) // g，在外層宣告找到
  fmt.Println(h) // 編譯錯誤，未定義
}
```

得到輸出結果。

```
f
g
undefined: h
```

### 範例：同名但不同範圍的變數

說明：3 個 x 分別代表不同變數，因為其範圍不同而遮蔽上一層的值。

```go
package main

import "fmt"

func main() {
  x := "hello!"

  for i := 0; i < len(x); i++ {
    x := x[i]

    if x != '!' {
      x := x + 'A' - 'a'
      fmt.Printf("%c", x) // HELLO
    }
  }
}
```

改寫 3 個 x 分別為 x、y、z 以便閱讀。

```go
package main

import "fmt"

func main() {
  x := "hello!"

  for i := 0; i < len(x); i++ {
    y := x[i]

    if y != '!' {
      z := y + 'A' - 'a'
      fmt.Printf("%c", z) // HELLO
    }
  }
}
```

### 範例：隱含區塊

說明：

- `for`、`if`、`switch` 除了陳述的本體區塊外，在宣告的地方也建立的隱含的區塊，因此 (2) 和 (3) 都可以用到 (1) 當中 x 的宣告。
- (4) 是 main 的區塊的，看不見 x 和 y。

```go
package main

import "fmt"

func f(a int) int { return a }
func g(b int) int { return b }

func main() {
  a := 1

  if x := f(a); x == 0 {
    fmt.Println(x) // (1)
  } else if y := g(x); x == y {
    fmt.Println(x, y) // (2)
  } else {
    fmt.Println(x, y) // (3)
  }

  fmt.Println(x, y) // (4) undefined: x, undefined: y
}
```

### 範例：宣告順序 vs 範圍

- 套件層級的宣告順序對範圍沒有影響。
- 自身常數或變數的宣告對範圍有影響。

說明：f 的範圍只有在 `if` 之內，因此 if 之內沒有用到而報錯、後面的函式用到也會報錯未定義。

```go
if f, err := os.Open("hello_world.txt"); err != nil { // f declared but not used
  return err
}

f.ReadByte() // undefined: f
f.close()    // undefined: f
```

改寫，讓 f 可以在條件與後續都能存取。

```go
f, err := os.Open("hello_world.txt")

if err != nil {
  return err
}

f.ReadByte()
f.close()
```

再次改寫，讓 f 只限定於 if 區塊內存取。

```go
if f, err := os.Open("hello_world.txt"); err != nil {
  return err
} else {
  f.ReadByte()
  f.close()
}
```

### 範例：避免不當宣告所產生的範圍

說明：由於 cwd 與 err 都沒有在 main 宣告過，因此 `:=` 將其宣告為區域變數，導致 cwd 報錯未使用。

```go
package main

import (
  "log"
  "os"
)

var cwd string

func main() {
  cwd, err := os.Getwd() // cwd declared but not used

  if (err) != nil {
    log.Fatal("os.Getwd failed: %v", err)
  }
}
```

改寫，在最後印出 cwd 的值，讓 cwd 被用到，就不會報錯。

```go
package main

import (
  "log"
  "os"
)

var cwd string

func main() {
  cwd, err := os.Getwd()

  if (err) != nil {
    log.Fatal("os.Getwd failed: %v", err)
  }

  log.Printf("Working directory = %s", cwd) // Working directory = xxx/xxx/xxx
}
```

改寫，捨棄 `:=` 這種會宣告為區域變數的作法，改用 var 分開宣告 err 即可。

```go
package main

import (
  "log"
  "os"
)

var cwd string

func main() {
  var err error

  cwd, err = os.Getwd()

  if (err) != nil {
    log.Fatal("os.Getwd failed: %v", err)
  }
}

```
