# Chapter 4 組合型別

## 4.1 陣列

- 固定長度
- 所有元素型別相同

### 宣告陣列

- n: 長度
- type: 元素型態

```go
var array [n]type
```

宣告一個長度為 3 內容元素皆為 int 的 a 陣列

```go
var a [3]int
```

宣告一個長度為 3 內容元素皆為 int 的 a 陣列，並且給予初始值

```go
var a [3]int = [3]int{1, 2, 3}
```

宣告一個 a 陣列，長度型別由初始值決定
```go
var a = [...]int{1, 2, 3}

// 可簡化成：
a := [...]int{1, 2, 3}
```

宣告一個由初始值決定長度的陣列，初始值中 index 3 = 2, index 5 = 1，其餘都會是 int 初始的 0
```go
a := [...]int{3: 2, 5: 1}
fmt.Println(a) // [0 0 0 2 0 1]
```

也可宣告多個維度，舉例來說: 宣告一個二維陣列

```go
a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
```

### 陣列型別

陣列的大小也是型別的一部分，`[3]int` 與 `[4]int` 是不同型別，不能互相指派值，也不能互相比較，在 compiler 就會報錯

compiler (IncompatibleAssign)

```go
a := [3]int{1, 2, 3}
a = [4]int{1, 2, 3, 4}

// cannot use ([4]int literal) (value of type [4]int) as [3]int value in assignment 
```

compiler (MismatchedTypes)

```go
a := [3]int{1, 2, 3}
b := [4]int{1, 2, 3, 4}

fmt.Println(a == b)
// invalid operation: cannot compare a == b (mismatched types [3]int and [4]int) 
```

### 陣列操作

取得陣列長度: `len()` and `cap()`

- len(): 取得 array 或 slice 的長度
- cap(): 取得 array 或 slice 的容量

```go
a := [3]int{}
fmt.Println(len(a), cap(a)) // 3, 3
```


## 4.2 Slice

- 可變的長度序列
- 本身不儲存值，而是儲存到陣列的參考 (reference)
- 三元件：
    - 指標：指向陣列中可以從 slice 存取的第一個元素 (不一定是陣列第一個元素)
    - 長度：Slice 元素數量 (不能超過容量)
    - 容量：Slice 元素最大容量
    - 內建 len, cap 函式可以回傳值

### 宣告

#### 直接建立

可以使用和陣列一樣的宣告方式，或是利用 make 來動態建立 slice

```go
// 和陣列一樣的宣告方式
var a = [...]int{1, 2, 3}
a := [...]int{1, 2, 3}

// 使用 make 動態建立 slice
// len=3 cap=3 default:[0,0,0]
a := make([]int, 3)
```

#### 指定 array 位置

指定 b 這個 slice 把指標這定在 a index=2 的位置
- 修改 b, a 也會被修改

```go
b := a[2:3]
```

### Slice 特性

#### 儲存陣列的參考

傳遞 slice 給函式可以修改底層陣列元素

```go
// reverse reverses a slice of ints in place.
func reverse(s []int) {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
}

// reverse a
a := [...]int{0, 1, 2, 3, 4, 5}
reverse(a[:])
fmt.Println(a) // "[5 4 3 2 1 0]"

// reverse b from index 0 to index 2
b := [...]int{0, 1, 2, 3, 4, 5}
reverse(b[0:3])
fmt.Println(b) // "[2 1 0 3 4 5]"

```

#### 不能比較

slice 不能直接比較
- 不能使用 `==`測試兩個 slice 是否帶有相同元素
- slice([byte]) 可以使用 `btyes.Equal` 來比較
- 其餘型別需自行寫 compare func

測試一個 slice 是否為空
- (O): len(s) == 0
- (X): s == nil

### append 函式

內建的 append 函式可以對 slice 加入元素

```go
var x []int
x = append(x, 1)
fmt.Println(x) // [1]
x = append(x, 2, 3)
fmt.Println(x) // [1 2 3]
x = append(x, x...)
fmt.Println(x) // [1 2 3 1 2 3]
```

append 做了什麼事？
以 `appendInt` 做範例解釋

```go
func appendInt(x []int, y int) []int {
	var z []int
	zlen := len(x) + 1
	if zlen <= cap(x) {
		// There is room to grow.  Extend the slice.
		z = x[:zlen]
	} else {
		// There is insufficient space.  Allocate a new array.
		// Grow by doubling, for amortized linear complexity.
		zcap := zlen
		if zcap < 2*len(x) {
			zcap = 2 * len(x)
		}
		z = make([]int, zlen, zcap)
		copy(z, x) // a built-in function; see text
	}
	z[len(x)] = y
	return z
}
```

#### 使用 slice and append 實作 stack

- 初始為一個空 stack
- 使用 append 來 push 新的值

```go
stack = append(stack, v) // push v
```

- get stack top
```go
top := stack[len(stack)-1] // stack's top
```

- pop element
```go
stack = stack[:len(stack)-1] // pop
```

```go
var stack []int
stack = append(stack, 1, 2, 3)
fmt.Println(stack) // [1 2 3]
top := stack[len(stack)-1]
fmt.Println(top) // 3
fmt.Println(stack) // [1 2 3]
stack = stack[:len(stack)-1]
fmt.Println(stack) // [1 2]
```

## 4.3 map
- 所有的 key 必須是一樣的型別
- 所有的 value 必須是一樣的型別
- key and value 不一定要是一樣的型別

### 建立 map

使用 make 內建函式

```go
// 宣告&建立一個 key 是 string, value 是 int 的 map
ages := make(map[string]int)
```

建立 map with default value

```go
ages := map[string]int{
	"alice": 31,
    "charlie": 34
}

// 同等於使用 make 的方式
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34
```

### map 元素操作

刪除元素

```go
delete(ages, "alice")
```

對 value 操作

```go
// ages 中沒有 bob，會是預設的 int 也會是 0
ages["bob"] = ages["bob"] + 1
```

縮寫形式的指派也可以使用
```go
ages["bob"] += 1
ages["bob"]++
```

map 元素不是變數，不能取得位址

```go
_ = &ages["bob"] // 會編譯錯誤
```

列舉 map 中元素
```go
// 
for name, age := range ages {
	fmt.Printf("%s\t%d\n", name, age) 
}

// 只拿 age 怎麼做?
for age := range ages {
	fmt.Printf("%d\n", age) // fmt.Printf format %d has arg age of wrong type string
}

// 只拿 age 要先忽略前面的 name(key)
for _, age := range ages {
	fmt.Printf("%d\n", age)
}
```

### map 型別零值

map 型別零值為 `nil`

```go
var ages map[string]int
fmt.Println(ages == nil) // "true"
fmt.Println(len(ages) == 0) // "true"

// 如果存值到 nil map 會造成 panic
ages["tom"] = 30 // panic: assignment to entry in nil map
```

因為預設為 0，所以不能用取值來判斷是否有這個 key

```go
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34

age1 := ages["alice"]
fmt.Printf("%d\n", age1)

age2 := ages["bob"]
fmt.Printf("%d\n", age2) // 0, 預設值
```

可改用：
```go
age2, ok := ages["bob"]
if !ok {
	fmt.Print("not a key\n")
} else {
	fmt.Printf("%d\n", age2)
}
```

## 4.4 struct

- 集合資料型別
- 類似 Object

### 宣告與定義

使用 type 和 struct 來做定義

```go
type Employee struct {
	ID         int
	Name       string
	Address    string
	DoB        time.Time
	Position   string
	Salary     int
	ManagerId  int
}

var tom Employee
```

### 存取

可直接對值做存取或是對指標做存取

```go
tom.Salary += 5000

position := &tom.Salary
*position += 5000
```

沒有欄位(值)的 struct 型別稱為空 struct
- 大小為零
- 不帶有資訊

```go
struct{}
```

### 匯出與暴露

struct 名稱首字母小寫

- 這個struct不會被匯出
- 內部所有欄位都不會匯出
- 有首字母大寫的欄位名也不會被匯出

struct 名稱首字母大寫

- struct會被匯出
- 只匯出內部首字母大寫的欄位

### 不具名欄位

有一組定義:

```go
type Point struct {
	X, Y int
}

type Circle struct {
	Center Point
	Radius int
}

type Wheel struct {
	Circle Circle
	Spokes int
}
```

如果要改變 `Wheel` 的欄位:

```go
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20
```

如果宣告沒有名稱的欄位(拿掉Center 和 Circle這兩個名稱)

```go
type Point struct {
	X, Y int
}

type Circle struct {
	Point
	Radius int
}

type Wheel struct {
	Circle
	Spokes int
}
```

這時候要改變 `Wheel` 的欄位就會變簡潔

```go
var w Wheel
w.X = 8
w.Y = 8
w.Radius = 5
w.Spokes = 20
```

但要注意：
- 不能有兩個相同型別的不具名欄位 => 會衝突

## 4.5 JSON

### marshaling

- 從 go 的資料結構轉換成 JSON 被稱為 marshaling，透過 `json.Marshal` 完成
- `json.MarshalIndent` 可以產生整齊縮排的 JSON
- marshaling 使用 struct 的欄位名稱作為 JSON 的物件名稱，且只有匯出的欄位會被 marshal

```go
Year int `json:"released"`
Color bool `json:"color,omitempty"`
// omitempty 表示如果欄位為其型別的零值或是空，就不輸出
```

定義一個 `Movie` 的 struct 與其資料內容:

```go
type Movie struct {
	Title  string
	Year   int  `json:"released"`
	Color  bool `json:"color,omitempty"`
	Actors []string
}

var movies = []Movie{
	{Title: "Casablanca", Year: 1942, Color: false,
		Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
	{Title: "Cool Hand Luke", Year: 1967, Color: true,
		Actors: []string{"Paul Newman"}},
	{Title: "Bullitt", Year: 1968, Color: true,
		Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
	// ...
}
```

直接使用 `Marshal` 來對剛剛的 `movies` 作轉換

```go
data, err := json.Marshal(movies)
if err != nil {
	log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```

上面就可以印出以下的 `JSON`

```json
[{"Title":"Casablanca","released":1942,"Actors":["Humphrey Bogart","Ingrid Bergman"]},{"Title":"Cool Hand Luke","released":1967,"color":true,"Actors":["Paul Newman"]},{"Title":"Bullitt","released":1968,"color":true,"Actors":["Steve McQueen","Jacqueline Bisset"]}]
```

### unmarshaling

- 將 JSON 解碼並產生 Go 的資料結構稱為 unmarshaling，透過 json.Unmarshal 完成
