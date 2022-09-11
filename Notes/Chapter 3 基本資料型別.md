# Ch3. 基本資料型別

包含：數字，字串和布林。

## 3.1 整數

- 帶正負號的整數有四種大小：8, 16, 32 和 64 位元，分別是 `int8`, `int16`, `int32` 和 `int64`，對應的無正負號型別是 `unit8`, `unit16`, `uint32` 和 `uint64`. 另外還有 `int` 和 `uint` 是特定平台上自然且大小最有效率的有正負號及無正負號整數。
- `rune` 是 `int32` 的同義詞，用來只是該值為 Unicode, 兩者名稱可以交換使用。
- `byte` 為 `uint8` 同義詞，強調該值為原始資料而非數值。
- 無正負號型別 `uintptr`, 它的大小並未指定，但足夠保存指標值的所有位元。只用於低階程式設計，例如 Go 和 C library 或 OS 的介接。
- 帶正負號的數字以 2 補位的形式表示，其最高位元保留給正負號且 n 位元數字值得範圍是 -2<sup>n-1</sup> ~ 2<sup>n-1</sup>-1, Ex.
  - `int8`: -128 ~ 127
  - `uint8`: 0 ~ 255

### 運算子

二進位運算子只有五層優先順序，同一層的運算子以左邊優先：

1. `*`, `/`, `%`, `<<`, `>>`, `&`, `&^`
2. `+`, `-`, `|`, `^`
3. `==`, `!=`, `<`, `<=`, `>`, `>=`
4. `&&`
5. `||`

- `+`, `-`, `*`, `/` 等數學運算子可以用在整數、浮點數和複數，但 `%` 只能用在整數。
- 在 Go 中，餘數的正負號和被除數相同，例如：`-5%3` 和 `-5%-3` 都是 -2.
- `/` 的行為視運算子是否為整數而定，例如：`5.0/4.0 = 1.25`, 但 `5/4 = 1`.
- Overflow: 若數學運算的結果超過結果型別的範圍，則為 Overflow, 容不下的最高位元會被拋棄。如果原始數值是有正負號的型別，則最左為元為1時會變為負數，範例：

    ```go
    var u uint8 = 255
    fmt.Println(u, u+1, u*u)    // 255 0 1

    var i int8 = 127
    fmt.Println(i, i+1, i*i)    // 127 -128 1
    ```

#### 比較運算子

| Operator | Description |
|----------|-------------|
| ==       | 相等        |
| !=       | 不相等      |
| <        | 小於        |
| <=       | 小於或等於  |
| >        | 大於        |
| >=       | 大於或等於  |

- 相同型別的基本型別（布林、數字與字串）都是可以比較的
- 比較運算式的型別為布林

#### 二進位運算子

| Operator | Description        |
|----------|--------------------|
| &        | 位元 AND           |
| \|       | 位元 OR            |
| ^        | 位元 XOR           |
| &^       | 位元清除 (AND NOT) |
| <<       | 左位移             |
| >>       | 右位移             |

範例： `uint8` 位元操作，使用 `Printf` `%b` 來顯示二進位

```go
var x uint8 = 1<<1 | 1<<5
var y uint8 = 1<<1 | 1<<2

fmt.Printf("%08b\n", x)		// 00100010, {1,5} 的集合
fmt.Printf("%08b\n", y)		// 00000110, {1,2} 的集合

fmt.Printf("%08b\n", x&y)	// 00000010, 交集
fmt.Printf("%08b\n", x|y)	// 00100110, 聯集
fmt.Printf("%08b\n", x^y)	// 00100100, 對稱差 {2,5}
fmt.Printf("%08b\n", x&^y)	// 00100000, 差 {5}

for i:= uint(0); i < 8; i++ {
    if x&(1<<i) != 0 {	// 成員測試
        fmt.Println(i)	// 1 5
    }
}

fmt.Printf("%08b\n", x<<1)	// 01000100, {2,6} 的集合
fmt.Printf("%08b\n", x>>1)	// 00010001, {0,4} 的集合
```

### Others

- 不同型別必須有明確的轉換
- 數學與邏輯（除了位移之外）的二進位運算子的運算元必須是相同型別

範例：

- Compile error

    ```go
    var apples int32 = 1
    var oranges int16 = 2
    var compote int = apples + oranges // Compile error
    ```

    Solution:

    ```go
    var compote int = int(apples) + int(oranges)
    ```

- 浮點數轉換

    ```go
    f := 3.141  // float64
    i := int(f)
    fmt.Println(f, i)   // 3.141 3

    f = 1.99
    fmt.Println(int(f)) // 1
    ```

    要避免超過目標型別的轉換，因為他的結果會取決於實作：

    ```go
    f := 1e100
    i := int(f)  // 結果視實作而定
    ```

- 十進位、八進位、十六進位
  - 八進位數字以0開頭，十六進位以0x或0X開頭
  - 可以使用 `%d`, `%o` 或 `%x` 控制底數與格式
  
    ```go
    o := 0666
    fmt.Printf("%d %[1]o %#[1]o\n", o)      // 438 666 0666

    x := int64(0xdeadbeef)
    fmt.Printf("%d %[1]x %#[1]x %#[1]X", x) // 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
    ```

- `rune`
  - 單引號字元，ex. `'a'`
  - 以 `%c` 或 `%q`（加引號）輸出

    ```go
    ascii := 'a'
    unicode := '国'
    newline := '\n'

    fmt.Printf("%d %[1]c %[1]q\n", ascii)   // 97 a 'a'
    fmt.Printf("%d %[1]c %[1]q\n", unicode) // 22269 国 '国'
    fmt.Printf("%d %[1]q\n", newline)       // 10 '\n'
    ```

## 3.2 浮點數

包含 `float32` 和 `float64`.

- `float32`
  - 最大值約為 3.4e48, 最小值約為 1.4e-45
  - 提供小數點以下六位數的精確度
- `float64`
  - 最大值約為 1.8e038, 最小值約為 4.9e324
  - 提供小數點以下15位數的精確度
  - 適合大部分用途
- 非常大或非常小的數值建議以科學記號表示，ex. `6.02214129e23`, `6.62606957e-34`
- 通常以 `%g` 輸出，他會選擇適合精確度的最精簡表示，也可以使用 `%e` (指數)或 `%f` (無指數)，這三種修飾詞可以控制欄寬與數值精確度
  
    ```go
    for x := 0; x < 8; x++ {
        fmt.Printf("x = %d e^x = %8.3f\n", x, math.Exp(float64(x)))
    }
    ```

    Output:

    ```text
    x = 0 e^x =    1.000
    x = 1 e^x =    2.718
    x = 2 e^x =    7.389
    x = 3 e^x =   20.086
    x = 4 e^x =   54.598
    x = 5 e^x =  148.413
    x = 6 e^x =  403.429
    x = 7 e^x = 1096.633
    ```

- 特殊值
  - `+Inf`: 正無限大
  - `-Inf`: 負無限大
  - `NaN`: Not a number

    ```go
    var z float64
    fmt.Println(z, -z, 1/z, -1/z, z/z)  // 0 -0 +Inf -Inf NaN
    ```

  - `math.IsNaN` 測試參數是否不是數字，`math.NaN` 回傳這樣的值，但測試特定運算結果是否等於 NaN 是很危險的事，因為與 NaN 的比較永遠是 false:
  
    ```go
    nan := math.NaN()
    fmt.Println(nan == nan, nan < nan, nan > nan)   // false false false
    ```

    如果回傳浮點數結果的韓式有可能失敗，最好獨立回傳該失敗。

### 範例: 浮點數圖形運算

以 SVG 繪製 3D 圖：

```go
package main

import (
	"fmt"
	"math"
)

const (
	width, height = 600, 320            // 畫布尺寸
	cells         = 100                 // 格數量
	xyrange       = 30.0                // 軸範圍 (-xyrange..+xyrange)
	xyscale       = width / 2 / xyrange // x 或 y 單位像素
	zscale        = height * 0.4        // z 單位像素
	angle         = math.Pi / 6         // x, y 角度(30°)
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle) // sin(30°), cos(30°)

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; fill: white; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i+1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j+1)
			dx, dy := corner(i+1, j+1)
			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64) {
	// 找出(i,j)格的點(x,y)
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	// 計算 z 高度
	z := f(x, y)

	// 投射 (x,y,z) 到 2D 畫布 (sx, sy)
	sx := width/2 + (x-y)*cos30*xyscale
	sy := height/2 + (x+y)*sin30*xyscale - z*zscale
	return sx, sy
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y) // 與(0,0)的距離
	return math.Sin(r) / r
}
```

Output:

![3-2](https://i.imgur.com/Z239YrD.png)

## 3.3 複數

包含 `complex64` 和 `complex128`.

- 內建的 complex 函式從實數與虛數建構出複數，而內建的 real 和 imag 函式提取這些元件：

    ```go
    var x complex128 = complex(1, 2)    // 1+2i
    var y complex128 = complex(3, 4)    // 3+4i

    fmt.Println(x*y)                    // -5+10i
    fmt.Println(real(x*y))              // -5
    fmt.Println(imag(x*y))              // 10
    ```

    上面的宣告也可以簡化為：

    ```go
    x := 1 + 2i
    y := 3 + 4i
    ```

- 複數可以使用 `==` 或 `!=` 進行比較，如果兩個複數的實數和虛數都相等，則兩複數相等。
- `math/cmplx` 提供操作複數的函式，例如：
  
    ```go
    fmt.Println(cmplx.Sqrt(-1)) // 0+1i
    ```

### 範例: 產生曼德博碎形 PNG 圖

```go
package main

import (
	"image"
	"image/color"
	"image/png"
	"math/cmplx"
	"os"
)

func main() {
	const (
		xmin, ymin, xmax, ymax = -2, -2, +2, +2
		width, height          = 1024, 1024
	)

	img := image.NewRGBA(image.Rect(0, 0, width, height))
	for py := 0; py < height; py++ {
		y := float64(py)/height*(ymax-ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px)/width*(xmax-xmin) + xmin
			z := complex(x, y)
			// Image point (px, py) represents complex value z.
			img.Set(px, py, mandelbrot(z))
		}
	}
	png.Encode(os.Stdout, img) // NOTE: ignoring errors
}

func mandelbrot(z complex128) color.Color {
	const iterations = 200
	const contrast = 15

	var v complex128
	for n := uint8(0); n < iterations; n++ {
		v = v*v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast*n}
		}
	}
	return color.Black
}

//!-

// Some other interesting functions:

func acos(z complex128) color.Color {
	v := cmplx.Acos(z)
	blue := uint8(real(v)*128) + 127
	red := uint8(imag(v)*128) + 127
	return color.YCbCr{192, blue, red}
}

func sqrt(z complex128) color.Color {
	v := cmplx.Sqrt(z)
	blue := uint8(real(v)*128) + 127
	red := uint8(imag(v)*128) + 127
	return color.YCbCr{128, blue, red}
}

// f(x) = x^4 - 1
//
// z' = z - f(z)/f'(z)
//    = z - (z^4 - 1) / (4 * z^3)
//    = z - (z - 1/z^3) / 4
func newton(z complex128) color.Color {
	const iterations = 37
	const contrast = 7
	for i := uint8(0); i < iterations; i++ {
		z -= (z - 1/(z*z*z)) / 4
		if cmplx.Abs(z*z*z*z-1) < 1e-6 {
			return color.Gray{255 - contrast*i}
		}
	}
	return color.Black
}
```

Output:

![3-3](https://i.imgur.com/9seMKVZ.png)

## 3.4 布林

- 布林值只有兩種可能的值：`true` or `false`.
- `if` 和 `for` 陳述的條件為布林，`==` 與 `<` 等比較運算子也產生布林結果。
- 布林值可以與 `&&` (AND) 和 `||` (OR) 組合，它具有**短路**行為，如果答案已由左運算元決定，則不會進行右運算式，讓如下運算式可安全撰寫：

    ```go
    s != "" && s[0] == 'x'
    ```

- 由於 `&&` 的優先序比 `||` 高，下列形式的條件式不需要括號：

    ```go
    if 'a' <= c && c <= 'z' ||
        'A' <= c && c <= 'Z' ||
        '0' <= c && c <= '9' {
        // do something...
    }
    ```

## 3.5 字串

- 字串是不可變的一系列位元組，可以帶有任何資料，包括值為 0 的位元組，但通常會帶有可讀的文字。
- `len` 函式回傳字串中位元組數量，索引操作 `s[i]` 讀取字串 s 的第 i 個位元組 (`0 <= i < len(s)`)

    ```go
    s := "hello, world"
    fmt.Println(len(s))     // 12
    fmt.Println(s[0], s[7]) // 104 119 ('h' and 'w')

    c := s[len(s)]          // panic
    ```

- 字串的第 i 個位元組不一定是第 i 字元，因為 UTF-8 編碼對非 ASCII 碼位需要兩個或以上的位元組。
- 子字串 `s[i:j]` 可產生從原始字串index i 到 j (不含 j)之間的位元組，結果帶有 j-i 個位元組。若超出範圍或 j < i 則會引發 panic.

    ```go
    fmt.Println(s[0:5]) // hello
    ```

    i, j 運算元省略時，會以預設值 0 和 `len(s)` 代替：

    ```go
    fmt.Println(s[:5])  // hello
    fmt.Println(s[7:])  // world
    fmt.Println(s[:])   // hello, world
    ```

- 可用 `+` 連接兩字串以產生新字串

    ```go
    fmt.Println("goodbye" + s[5:]) // goodbye, world
    ```

- 可用 `==`, `<` 等運算子進行比較，比較是逐位元組進行。
- 字串值不可變：字串直的位元組序不可改變，但可以指派新值給字串變數

    ```go
    s := "left foot"
    t := s
    s += ", right foot"

    fmt.Println(s)  // left foot, right foot
    fmt.Println(t)  // left foot

    // 不可直接修改字串中的位元組
    s[0] = 'L'      // Error
    ```

### 3.5.1 字串實字

- 字串值可以寫成**字串實字**, Ex. `"Hello, 世界"`
- Go 以 UTF-8 編碼，且字串通常以 UTF-8 解譯，我們可以在字串中用 Unicode 碼位，以反斜線 `\` 開頭的跳脫字符可以用來插入任意位元組值。

    常見的 ASCII 控制碼：

    | Code | Description     |
    |------|-----------------|
    | \a   | 警告或響鈴      |
    | \b   | 後退            |
    | \f   | 跳頁            |
    | \n   | 換行            |
    | \r   | carriage return |
    | \t   | Tab             |
    | \v   | 垂直 Tab        |
    | \\'  | 單引號          |
    | \\"  | 雙引號          |
    | \\\  | 反斜線          |

    也可以用八進位或十六進位跳脫引入任意字元組：

    - `\ooo`: 八進位數字，包含三個八進位數字 (0~7)，但不能超過 `\377` (對應一個字元的範圍)
    - `\xhh`: 十六進位數字，h 表示十六進位數字

- 使用 ` 反引號取代雙引號，跳脫字符不會被處理，會採用實際內容，也可以將字串分多行。

### 3.5.2 Unicode

- 檔案是 Unicode, 它集合世上所有撰寫系統，加上各種符號與控制碼，並為每一個字符指派一個 Unicode 碼位，在 Go 中稱為 `rune`.
- 第八版定義 100 個以上的語言超過 120,000 個字元碼位，保存單一 `rune` 的自然資料型別為 `int32`, Go 也是使用它。我們也可以將一系列的 `rune` 以一系列的 `int32` 值來表示。
- 在這種被稱為 UTF-32 或 UCS-4 的表示中，每個 Unicode 碼位的大小均為 32 位元，但由於大部分電腦上可讀的文字都是每個字元只要 8 位元或一個位元組的 ASCII 而比所需佔用了更多。

### 3.5.3 UTF-8

- UTF-8 是 Unicode 碼位的可變長度位元組編碼。
- 它使用 1 ~ 4 個位元組來表示每個 rune, 但對於 ASCII 字元只使用一個位元，一般的 rune 也只使用 2 或 3 個位元組。
- 此編碼的 rune 的第一個位元組的高位位元表示後面有多少個位元組
  - 高位 0 表示 7 位元的 ASCII, 每個 rune 只佔用一個位元組，與傳統 ASCII 相同
  - 高位 110 表示該 rune 佔用 2 個位元組，第二個位元組以 10 開頭

    | Code                                | Description                   |
    |-------------------------------------|-------------------------------|
    | 0xxxxxxx                            | runes 0-127 (ASCII)           |
    | 110xxxxx 10xxxxxx                   | 128-2047 (值 < 128 未使用)    |
    | 1110xxxx 10xxxxxx 10xxxxxx          | 2048-65535 (值 < 2048 未使用) |
    | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx | 65536-0x10ffff (其他值未使用) |

- 可變長度編碼使得字串的第 n 個字元無法直接以索引存取，但 UTF-8 有許多特性可以補償。
  - 它的編碼緊湊，與 ASCII 相容，並自我同步：不超過三個位元組就可以找到字元的開頭位置。
  - 它也是前綴碼，所以可以從左邊開始解碼而沒有混編或前置作業。
  - rune 的編碼不會有其他子串列或其他序列，所以可以只靠搜尋位元組來找尋 rune, 不用擔心前面的內容。
  - 字典位元組排序與 Unicode 碼位順序相同，因此 UTF-8 的排序一樣自然。
  - 它沒有嵌入 NUL 位元組，對使用 NUL 借數字串的程式語言很方便。
- Go 原始檔是以 UTF-8 編碼，且 UTF-8 是 Go 程式偏好的文字操作編碼。
- Go 的字串實字的 Unicode 跳脫讓我們可以用數值碼位值來指定，有兩種形式：(h 代表十六進位數字)
  - `\uhhhh`: 16 位元值
  - `\Uhhhhhhhh`: 32 位元值 (較罕見)
  
    ```go
    // 以下值都相同
    "世界"
    "\xe4\xb8\x96\xe7\x95\x8c"
    "\u4e16\u754c"
    "\U00004e16\U0000754c"

    '世'
    '\u4e16'
    '\U00004e16'
    ```

- 因為 UTF-8 的特性，在許多字串操作時不需要解碼，以下範例：
  - 檢查 Prefix:

    ```go
    func HasPrefix(s, prefix string) bool {
        return len(s) >= len(prefix) && s[:len(prefix)] == prefix
    }
    ```

  - 檢查 Suffix:

    ```go
    func HasPrefix(s, suffix string) bool {
        return len(s) >= len(suffix) && s[len(s) - len(suffix):] == suffix
    }
    ```

  - 檢查子字串:

    ```go
    func Contains(s, substr string) bool {
        for i := 0; i < len(s); i++ {
            if HasPrefix(s[i:], substr) {
                return true
            }
        }
        return false
    }
    ```

- 若真的關心個別 Unicode 字元，則可以使用其他處理方式

    ```go
    import "unicode/utf8"

    s := "Hello, 世界"
    fmt.Println(len(s))                     // 13
    fmt.Println(utf8.RuneCountInString(s))  // 9
    ```

    ```go
    for i := 0; i < len(s); {
        r, size := utf8.DecodeRuneInString(s[i:])   // r: rune, size: UTF-8 所佔的位元組數量
        fmt.Println("%d\t%c\n", i, r)
        i += size
    }
    ```

    不過 Go 的 range 迴圈在套用於字串時會解決執行 UTF-8 解碼。

    ```go
    for i, r := range s {
        fmt.Printf("%d\t%q\t%d\n", i, r, r)
    }

    /* Output:
    0	'H'	72
    1	'e'	101
    2	'l'	108
    3	'l'	108
    4	'o'	111
    5	','	44
    6	' '	32
    7	'世'	19990
    10	'界'	30028
    */
    ```

    計算 rune 數量：

    ```go
    n := 0
    for _, _ = range s {
        n++
    }

    // 省略不需要的變數
    n = 0
    for range s {
        n ++
    }

    // 也能呼叫 utf8.RuneCountInString(s)
    ```

### 3.5.4 字串與位元組 slice

操作字串的四個重要套件：bytes, strings, strconv 和 unicode.

- strings: 提供許多搜尋、替換、比較、截空白、分割與連接字串的函式。
- bytes: 有類似的韓式來操作 `[]bytes` 的位元組 slice, 它與字串有些相同的特性。
- strconv: 提供布林、整數以及浮點數值與字串間的轉換，還有加引號和去飲好的函式。
- unicode: 提供 `IsDigit`, `IsLetter`, `IsUpper` 和 `IsLower` 等函式。
  - `ToUpper` 和 `ToLower` 將 rune 轉換大小寫。(strings 套件也有類似的函式，也稱為 `ToUpper` 和 `ToLower`, 將原始字串的每個字元都做相應的轉換，然後回傳新的字串)

#### 範例: Basename

此範例事實作移除路徑與副檔名，第一版為未使用 library:

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        fmt.Println(basename(input.Text()))
    }
    // NOTE: ignoring potential errors from input.Err()
}

// 移除路徑與副檔名
// e.g., a => a, a.go => a, a/b/c.go => c, a/b.c.go => b.c
func basename(s string) string {
    // 移除最後一個 '/' 與之前的所有東西
    for i := len(s) - 1; i >= 0; i-- {
        if s[i] == '/' {
            s = s[i+1:]
            break
        }
    }
    // 保留最後一個 '.' 之前的所有東西
    for i := len(s) - 1; i >= 0; i-- {
        if s[i] == '.' {
            s = s[:i]
            break
        }
    }
    return s
}
```

使用 strings library:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	input := bufio.NewScanner(os.Stdin)
	for input.Scan() {
		fmt.Println(basename(input.Text()))
	}
	// NOTE: ignoring potential errors from input.Err()
}

// basename removes directory components and a trailing .suffix.
// e.g., a => a, a.go => a, a/b/c.go => c, a/b.c.go => b.c
//!+
func basename(s string) string {
	slash := strings.LastIndex(s, "/") // -1 if "/" not found
	s = s[slash+1:]
	if dot := strings.LastIndex(s, "."); dot >= 0 {
		s = s[:dot]
	}
	return s
}
```

#### 範例: 插入逗號

將整數字串每三個插入逗號：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for i := 1; i < len(os.Args); i++ {
		fmt.Printf("  %s\n", comma(os.Args[i]))
	}
}

// comma inserts commas in a non-negative decimal integer string.
func comma(s string) string {
	n := len(s)
	if n <= 3 {
		return s
	}
	return comma(s[:n-3]) + "," + s[n-3:]
}
```

#### 範例: Printints

- 字串創建後不可變，相對的 byte slice 可以自由修改。
- 字串和 byte slice 之間可以互相轉換：

    ```go
    s := "abc"
    b := []byte(s)
    s2 := string(b)
    ```

  概念上，[]byte(s)轉換會分配保存 s 的位元組拷貝的新位元組陣列並產生參考該陣列的 slice。最佳化的編譯器或許能夠在某些情況下避免分配與複製，但一般來說複製是必要的，以確保 s 的位元組在之後 b 被修改時還能維持不變。使用 string(b) 將 byte slice 轉換回字串也會製作拷貝，以確保結果 s2 不可變。

  為了避免轉換與不必要的記憶體分配，bytes 套件和 strings 套件同時提供了許多實用函數。下面是strings包中的六個函式：

    ```go
    func Contains(s, substr string) bool
    func Count(s, sep string) int
    func Fields(s string) []string
    func HasPrefix(s, prefix string) bool
    func Index(s, sep string) int
    func Join(a []string, sep string) string
    ```

  bytes 套件中也有相對應的函式：

    ```go
    func Contains(b, subslice []byte) bool
    func Count(s, sep []byte) int
    func Fields(s []byte) [][]byte
    func HasPrefix(s, prefix []byte) bool
    func Index(s, sep []byte) int
    func Join(s [][]byte, sep []byte) []byte
    ```

  它們之間唯一差別是字串被轉換為 byte slice.

bytes 套件提供 Buffer 型別以有效率的操控位元組 slice, 一開始 Buffer 開是空的，但是隨著 string, byte 或 []byte 等型別資料的寫入可以動態增長，`bytes.Buffer` 變數不需要初始化，因為它的零值可用：

```go
// Printints demonstrates the use of bytes.Buffer to format a string.
package main

import (
	"bytes"
	"fmt"
)

// 類似 fmt.Sprint(values) 但加上逗號
func intsToString(values []int) string {
	var buf bytes.Buffer
	buf.WriteByte('[')
	for i, v := range values {
		if i > 0 {
			buf.WriteString(", ")
		}
		fmt.Fprintf(&buf, "%d", v)
	}
	buf.WriteByte(']')
	return buf.String()
}

func main() {
	fmt.Println(intsToString([]int{1, 2, 3})) // "[1, 2, 3]"
}
```

將 rune 的 UTF-8 編碼加入到 `bytes.Buffer` 時，最好使用 `bytes.Buffer` 的 `WriteRune` 方法，但 `WriteByte` 對 `[` 和 `]` 等 ASCII 字元也沒問題。

### 3.5.5 字串與數字的轉換

- 數值和字串之間的轉換可透過 `strconv` 套件
- 將整數轉換成字串有以下方法：
  - 透過 `fmt.Sprintf`
  - 使用 `strconv.Itoa`

    ```go
    x := 123
    y := fmt.Sprintf("%d", x)
    fmt.Println(y, strconv.Itoa(x)) // 123 123
    ```
- `FormatInt` 和 `FormatUint` 可在不同的底數上格式化數字：(不同進制)

    ```go
    fmt.Println(strconv.FormatInt(int64(x), 2)) // 1111011
    ```

- `fmt.Printf` 的 `%b`, `%d`, `%u` 和 `%x` 通常比 Format 函式更方便，特別是，特別是在需要包含其他資訊時：

    ```go
    s := fmt.Sprintf("x=%b", x) // x=1111011
    ```

- 要解析字串所代表的數字時，可以使用 `strconv` 的 `Atoi` 或 `ParseInt`, 或無正負號整數的 `ParseUint`:

    ```go
    x, err := strconv.Atoi("123")             // x 是整數
    y, err := strconv.ParseInt("123", 10, 64) // 10 進位，最多 64 位元
    ```

  - `ParseInt` 的第三個參數指定結果必須符合的整數型別，例如：16 表示 `int16`, 0 表示 `int`; 上例表示 y 的型別是 `int64`.
- 有時候也會使用 `fmt.Scanf` 來解析單行中混合字串和數字的輸入，但它沒有彈性，特別是在處理不完整或不規律的輸入時。

## 3.6 常數

### 3.6.1 iota 常數產生器

### 3.6.2 無型別常數
