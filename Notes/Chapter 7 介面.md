# Ch7 interface 介面

### Agenda
- 7.1 何謂介面
- 7.2 介面基本用法
- 7.3 空介面
- 7.4 以介面為合約
- 7.5 介面嵌入介面
- 7.6 介面值
- 7.7 型態判斷
- 7.8 以sort.Interface排序
- 7.9 error介面

## 7.1 何謂介面
1. Go的介面是一種**抽象型別**
2. 它是滿足**隱含性的**，無需聲明哪個struct實現了這個介面
3. 可以用來定義方法，更精確的說是它的方法提供什麼行為
4. 利用 interface 可實現泛型、多型的功能，從而可以調用同一個函數名的函數但實現完全不同的功能。

## 7.2 介面基本用法
- 如何宣告

用type來宣告interface

裡面宣告抽象方法
```
type BankAccount interface {
	GetName() string
	GetBalance() int
	Deposit(amount int)
	Withdraw(amount int) error
}
```
可以看到裏面宣告了三個方法，不過現在也只有定義並沒有實踐

- 如何實踐

Golang 中如果自定義型態實現了 interface 的所有方法，那麼它就會認定該自定義型態也是 interface 型態的一種

例如：在這邊宣告一個```EsunAccount```的struct，並且實踐了```GetBalance()``` ```Deposit()``` ```Withdraw```這三個方法，那麼Golang就會自動判定我們實作了```BankAccount```這個介面

```
type EsunAccount struct {
}

func (e *EsunAccount) GetName() string {
	//TODO implement me
}

func (e *EsunAccount) GetBalance() int {
	//TODO implement me
}

func (e *EsunAccount) Deposit(amount int) {
	//TODO implement me
}

func (e *EsunAccount) Withdraw(amount int) error {
	//TODO implement me
}
```
特別注意的是：當我們實踐方法時，都是用指標```(*EsunAccount)```

這個所代表的意思是透過傳遞指標來操控同一個struct實例

那趕緊來看看介面的效益吧
```
//esun.go
type EsunAccount struct {
	balance int
}

func NewEsunAccount() *EsunAccount {
	return &EsunAccount{
		accountName: "esunAccount",
		balance: 0,
	}
}

func (e *EsunAccount) GetName() string {
	fmt.Printf("Account Name= %s\n", e.accountName)
	return e.accountName
}

func (e *EsunAccount) GetBalance() int {
	return e.balance
}

func (e *EsunAccount) Deposit(amount int) {
	e.balance += amount
}

func (e *EsunAccount) Withdraw(amount int) error {
	newBalance := e.balance - amount
	if newBalance < 0 {
		return errors.New("insufficient funds")
	}
	e.balance = newBalance
	return nil
}
```

```
//main.go
func main() {
	esun := NewEsunAccount()
	//存錢
	esun.Deposit(300)
	esun.Deposit(100)
	esun.Deposit(500)

	//領錢
	esun.Withdraw(600)
	//印出帳戶剩多少錢
	fmt.Printf("%s balance: %d\n", esun.GetName(), esun.GetBalance())
}
```
執行結果：
```
esunAccount balance: 300
```
接下來要展現介面真正的力量了

當我們多一個銀行帳號```CtbcAccount```
```
//ctbc.go
type CtbcAccount struct {
	balance int
	fee     int
}

func NewCtbcAccount() *CtbcAccount {
	return &CtbcAccount{
		accountName: "ctbcAccount",
		balance: 0,
		fee:     15,
	}
}

func (c *CtbcAccount) GetName() string {
	fmt.Printf("Account Name= %s\n", c.accountName)
	return c.accountName
}

func (c *CtbcAccount) GetBalance() int {
	return c.balance
}

func (c *CtbcAccount) Deposit(amount int) {
	c.balance += amount
}

func (c *CtbcAccount) Withdraw(amount int) error {
	newBalance := c.balance - amount - c.fee
	if newBalance < 0 {
		return errors.New("insufficient funds")
	}
	c.balance = newBalance
	return nil
}
```
可以看到ctbc這銀行在領錢的時候會多一層手續費，多一個```fee```屬性，並在```Withdraw()```方法的實作內會多扣掉這個```fee```

讓我們回到```main.go```裡面去看，同樣對兩間銀行做存錢領錢的動作，帳戶會剩多少錢

```
//main.go
func main() {

	myAccounts := []BankAccount{
		NewEsunAccount(),
		NewCtbcAccount(),
	}

	for _, account := range myAccounts {
		//存錢
		account.Deposit(100)
		//領錢
		if err := account.Withdraw(50); err != nil {
			fmt.Printf("ERR: %d\n", err)
		}
		//印出帳戶剩多少錢
		fmt.Printf("%s balance: %d\n", account.GetName(), account.GetBalance())
	}
}
```
執行結果：
```
esunAccount balance: 50
ctbcAccount balance: 35
```
也可以這樣寫：
```
//main.go
func ShowAccountName(m BankAccount) {
	m.GetName()
}

func main() {
	var m BankAccount
	esun := NewEsunAccount()
	ctbc := NewCtbcAccount()
	m = esun
	ShowAccountName(m)
	m = ctbc
	ShowAccountName(m)
}
```
執行結果：
```
Account Name= esunAccount
Account Name= ctbcAccount
```
這樣的做法，也展現了介面可以實現了**多型**的行為

## 7.3 空介面
Go程式的interface除了定義型態的行為，本身也是一種型態。而空介面則代表任意型態。

空介面宣告方式：
```
interface{}
```

例如下面的變數```i```的型態為empty interface，所以可以接收任意型態的值。
```
type BankAccount struct {
    accountName string
    balance     int
}

func printValueAndType(i interface{}) { // take empty interface as parameter
    fmt.Printf("value=%v, type=%T\n", i, i)
}

func main() {
    var i interface{} // declare var as empty interface type

    i = "abc"
    printValueAndType(i) // value=abc, type=string

    i = 123
    printValueAndType(i) // value=123, type=int

    i = BankAccount{"ctbc", 1000}
    printValueAndType(i) // value={ctbc 1000}, type=main.BankAccount
}
```
有沒有覺得空介面其實很像```泛型(generics)```

[Go 1.18](https://tip.golang.org/doc/go1.18#generics)加入的泛型(generics)新增了一個關鍵字```any```作為空介面```interface{}```的別名，所以之後改用```any```即可
```
m := make(map[int]any)
m["a"] = 123
m["b"] = "abc"
```
也可以看到最經典最常用標準函式庫的例子

**fmt package**

func Println
```
func Println(a ...any) (n int, err error)
```
所以其實平常用```Println()```印出什麼型別的東西都可以

範例：
```
func main() {
    fmt.Println(123)
    fmt.Println("abc")
    fmt.Println(EsunAccount{"esun", 100})
}
```
執行結果：
```
123
abc
{esun 100}
```

## 7.4 以介面為合約
上面有提到```Println```方法，這邊來看一下```fmt```的三個函式的源碼
```
package fmt
func Fprintf(w io.Writer, format string, a ...any) (n int, err error)

func Printf(format string, a ...any) (n int, err error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```
可以看到```Printf()```和```Sprintf()```都會呼叫```Fprintf()```方法，並以```io.Writer```介面當成```Fprintf```與呼叫方的合約

先來看一下```Writer```這個介面
```
type Writer interface {
	Write(p []byte) (n int, err error)
}
```
一方面合約要求呼叫方要有呼叫```Write```方法的具體型別，另一方面合約保證Fprintf在值符合io.Writer介面時會執行工作

那我們就來追看看被當成```w```傳入的參數```os.Stdout```，果然在```os```底下看到一個函式```Write```
```
// Write writes len(b) bytes to the File.
// It returns the number of bytes written and an error, if any.
// Write returns a non-nil error when n != len(b).
func (f *File) Write(b []byte) (n int, err error) {
        if err := f.checkValid("write"); err != nil {
                return 0, err
        }
}
```
(想要追更底層可以參考[link](https://ithelp.ithome.com.tw/articles/10218227))

所以任何有實作```io.Writer```介面的具體型別都可以當成實作替換的參數

## 7.5 介面嵌入介面
剛剛有提到```Writer```介面
現在再來看一下另外兩個
```
package io

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```
可以在介面裡嵌入另一個介面

來看```io```的另一個interface
```
type ReadWriter interface {
	Reader
	Writer
}
```
直接宣告介面的名稱即可

也可以直接宣告要嵌入介面的方法

舉例：
```
type ReadWriter interface {
	Read(p []byte) (n int, err error)
	Write(p []byte) (n int, err error)
}
```
甚至混合兩種風格
```
type ReadWriter interface {
	Read(p []byte) (n int, err error)
	Writer
}
```

## 7.6 介面值
介面值有分為**型別**和**值**兩個元件

介面零值的型別和值皆為nil

這裡再舉例一個fmt套件庫的重要介面
```
package fmt

type Stringer interface {
	String() string
}
```
String方法用於輸出，接受任何格式的或像**Print函數**印出來為格式化的值

這邊我實作了兩個不同型別```Temp```,```Point```來實作**Stringer**介面
```
type Temp int

func (t Temp) String() string {
	return strconv.Itoa(int(t)) + "°C"
}

type Point struct {
	x, y int
}

func (p *Point) String() string {
	return fmt.Sprintf("%d, %d", p.x, p.y)
}

```
再來我們回到了主題來探討介面值

宣告介面並指派具體的型別，來看看各種情況下的介面型別與值
```
func main() {
	var x fmt.Stringer
	fmt.Printf("value=%v, type=%T\n", x, x) //value=<nil>, type=<nil>

	x = Temp(27)
	fmt.Printf("value=%v, type=%T\n", x, x) //value=27°C, type=main.Temp

	x = &Point{1, 2}
	fmt.Printf("value=%v, type=%T\n", x, x) //value=1, 2, type=*main.Point

	x = (*Point)(nil)
	fmt.Printf("value=%v, type=%T\n", x, x) //value=<nil>, type=*main.Point
}
```

**既然有值，就可以比較**

介面值可以用```==``` ```!=```比較

- 兩介面值若型別相同才可以比較
- 動態值均為nil或是動態型別相同且動態值根據其行為相等則介面值相等時相等時，```==```才成立

範例：
```
func main() {
	//實作AnotherTemp，並且與Temp行為相同
	var x, y fmt.Stringer

	fmt.Printf("%v %v\n", x, y) //印出兩者的介面值
	fmt.Println(x == y)         //型別相同且動態值相同
	fmt.Println("--------------")
	x = Temp(27)
	y = AnotherTemp(27)
	fmt.Printf("%v %v\n", x, y)
	fmt.Println(x == y)  //動態值相同但動態型別不同
	fmt.Println("--------------")
	y = Temp(27) //把y指派給Temp型別
	fmt.Printf("%v %v\n", x, y)
	fmt.Println(x == y) //動態型別和動態值皆相同
}
```
執行結果：
```
<nil> <nil>
true
--------------
27°C 27°C
false
--------------
27°C 27°C
true
```
⚠️特別注意：有些型別完全不能比較(slice, map與函式)

舉例：
```
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: []int型別不能比較
```

## 7.7 型態判斷
我們知道interface的變數裡可以存放任何實現此interface的任意型別

要反向知道變數裡面實際儲存的是什麼型別的物件

語意上類似```x.(T)```，```x```是介面的變數，```T```是要判別的型別

**常見的兩種做法**

- **Comma-ok**

如果在```x```裡面確實儲存了```T```型別的數值，那麼ok回傳 true，否則回傳 false

再沿用剛剛的例子並完整宣告三個實現Stringer interface的型別
```
// #1
type Temp int

func (t Temp) String() string {
	return strconv.Itoa(int(t)) + "°C"
}

// #2
type AnotherTemp int

func (a AnotherTemp) String() string {
	return strconv.Itoa(int(a)) + "°C"
}

// #3
type Point struct {
	x, y int
}

func (p Point) String() string {
	return strconv.Itoa(int(p.x)+int(p.y)) + "°C"
}
```
這裡會宣告三個型別塞進陣列，並用if-else分別用Comma-ok來判斷
```
func main() {
	var list [3]fmt.Stringer

	list[0] = Point{13, 14}
	list[1] = AnotherTemp(27)
	list[2] = Temp(27)

	for index, eachStringer := range list {
		if value, ok := eachStringer.(Temp); ok {
			fmt.Printf("list[%d] is an AnotherTemp and its value is %s\n", index, value)
		} else if value, ok := eachStringer.(AnotherTemp); ok {
			fmt.Printf("list[%d] is a Temp and its value is %s\n", index, value)
		} else if value, ok := eachStringer.(Point); ok {
			fmt.Printf("list[%d] is a Point type and its value is %s\n", index, value)
		}
	}
}
```
執行結果：
```
list[0] is a Point type and its value is 27°C
list[1] is a Temp and its value is 27°C
list[2] is an AnotherTemp and its value is 27°C
```
- **switch**

這裡我把陣列多加第四項，並不塞任何值

所以他會是```Stringer```的空介面
```
func main() {
	//實作AnotherTemp，並且與Temp行為相同
	//var x, y fmt.Stringer
	//
	//fmt.Printf("%v %v\n", x, y) //印出兩者的介面值
	//fmt.Println(x == y)         //型別相同且動態值相同
	//fmt.Println("--------------")
	//x = Temp(27)
	//y = AnotherTemp(27)
	//fmt.Printf("%v %v\n", x, y)
	//fmt.Println(x == y) //動態值相同但動態型別不同
	//fmt.Println("--------------")
	//y = Temp(27) //把y指派給Temp型別
	//fmt.Printf("%v %v\n", x, y)
	//fmt.Println(x == y) //動態型別和動態值皆相同
	var list [4]fmt.Stringer

	list[0] = Point{13, 14}
	list[1] = AnotherTemp(27)
	list[2] = Temp(27)

	for index, eachStringer := range list {
		switch value := eachStringer.(type) {
		case Temp:
			fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
		case AnotherTemp:
			fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
		case Point:
			fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
		default:
			fmt.Printf("list[%d] is of a different type and its value is %v\n", index, value)
		}
	}
}
```
執行結果：
```
list[0] is a Person and its value is 27°C
list[1] is a string and its value is 27°C
list[2] is an int and its value is 27
list[3] is of a different type and its value is <nil>
```

⚠️特別注意：```eachStringer.(type)```語法不能在 switch 外的任何邏輯裡面使用，如果你要在 switch 外面判斷一個型別就使用```comma-ok```

## 7.8 以sort.Interface排序
在很多語言中，排序算法都是和序列資料型別關聯，同時排序函數和具體類型元素關聯

相比之下，Go語言的```sort.Sort```函數不會對具體的序列和它的元素做任何假設

它使用了sort.Interface來指定通用的排序算法和可能被排序到的序列型別之間的合約

此介面通常為slice的排序方式

sort
```
func Sort(data Interface)
```
我們知道一個排序的演算法通常都需要三個東西
1. 排序的長度
2. 表示兩個比較的結果
3. 交換兩個元素的方式

因此```sort.interface```有三個方法：
```
type Interface interface {
	Len() int

	Less(i, j int) bool

	Swap(i, j int)
}
```

為了要對任何序列排序，必須實作這三個方法的型別，最後對該型別的實例套用sort.Sort

以```StringSlice```做範例
```
type StringSlice []string

func (p StringSlice) Len() int {
	return len(p)
}

func (p StringSlice) Less(i, j int) bool {
	return p[i] < p[j]
}

func (p StringSlice) Swap(i, j int) {
	p[i], p[j] = p[j], p[i]
}

func main() {
	names := []string{"Go", "Bravo", "Gopher", "Alpha", "Grin", "Delta"}
	sort.Sort(StringSlice(names))
	fmt.Println(names)
}
```
執行結果：
```
[Alpha Bravo Delta Go Gopher Grin]
```
其實這就是sort套件提供的```Strings```函式的做法：[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/sort/sort.go;l=164)

- **對字串的slice進行排序**

所以上面的呼叫可以簡化成
```
sort.Strings(names)
```

那我們再來看一下其他sort提供的常用函式

- **對整數、浮點數的slice進行排序**

```
func main() {

	// Sorting a slice of Integers
	ints := []int{56, 19, 78, 67, 14, 25}
	sort.Ints(ints)
	fmt.Println(ints)

	// Sorting a slice of Floats
	floats := []float64{176.8, 19.5, 20.8, 57.4}
	sort.Float64s(floats)
	fmt.Println(floats)
}
```
執行結果：
```
[14 19 25 56 67 78]
[19.5 20.8 57.4 176.8]
```
- **對struct的slice進行排序**

介紹完了```string``` ```int``` ```float64s```三個常見型別的slice後，來看一下如何對struct slice進行排序

假設有一個struct```User```並塞一些資料給它
```
type User struct {
	Name string
	Age  int
}

func main() {
	// Sorting a slice of structs by a field
	users := []User{
		{
			Name: "Barney",
			Age:  26,
		},
		{
			Name: "Peter",
			Age:  27,
		},
		{
			Name: "Amy",
			Age:  35,
		},
		{
			Name: "Amanda",
			Age:  16,
		},
		{
			Name: "Steven",
			Age:  28,
		},
	}
}
```
如果要指定依照年齡來排序，可以透過自訂```less```函數並傳給```Slice()```

來看一下sort的```Slice```函式，[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/sort/slice.go;l=18)
```
func Slice(x any, less func(i, j int) bool) {
	...
}
```
實作排序：
```
func main() {
	sort.Slice(users, func(i, j int) bool {
		return users[i].Age < users[j].Age
	})
	fmt.Println("Sorted users by age: ", users)
}
```
執行結果：
```
Sorted users by age:  [{Amanda 16} {Barney 26} {Peter 27} {Steven 28} {Amy 35}]
```
當使用```Slice```函數時，排序不保證穩定：相等的元素可能會和它們原本的順序顛倒

要穩定的排序，請使用```SliceStable()```，[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/sort/slice.go;l=32)
```
func main() {
	sort.SliceStable(users, func(i, j int) bool {
		return users[i].Age < users[j].Age
	})
}
```
- **自訂排序**

要啟用對任何類型集合的自訂排序，你需要定義一個相對應的類型並實作```sort.Interface```
```
type User struct {
	Name string
	Age  int
}

type UsersByAge []User

func (u UsersByAge) Len() int {
	return len(u)
}
func (u UsersByAge) Swap(i, j int) {
	u[i], u[j] = u[j], u[i]
}
func (u UsersByAge) Less(i, j int) bool {
	return u[i].Age < u[j].Age
}
```
```UsersByAge```實作了```sort.interface```介面，最後一步就是把```UsersByAge```當成參數傳入```Sort```函式

```
func main() {
	sort.Sort(UsersByAge(users))
	fmt.Println("Sorted users by age: ", users) //Sorted users by age:  [{Amanda 16} {Barney 26} {Peter 27} {Steven 28} {Amy 35}]
}
```
- **相反順序對slice進行排序**

要以相反順序對slice進行排序，可以使用sort提供的```Reverse```函式，[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/sort/sort.go;l=94)
```
func Reverse(data Interface) Interface
```
不過這裡只能用sort提供的三種類型```StringSlice``` ```IntSlice``` ```Float64Slice```來使用```Reverse()，這些類型實作了 sort package 中定義用來比較和交換的 Interface

範例：
```
func main() {
	// Sorting a slice of Strings
	strs := []string{"Go", "Bravo", "Gopher", "Alpha", "Grin", "Delta"}
	sort.Sort(sort.Reverse(sort.StringSlice(strs)))
	fmt.Println("Sorted strings in reverse order: ", strs)

	// Sorting a slice of Integers
	ints := []int{56, 19, 78, 67, 14, 25}
	sort.Sort(sort.Reverse(sort.IntSlice(ints)))
	fmt.Println("Sorted integers in reverse order: ", ints)

	// Sorting a slice of Floats
	floats := []float64{176.8, 19.5, 20.8, 57.4}
	sort.Sort(sort.Reverse(sort.Float64Slice(floats)))
	fmt.Println("Sorted floats in reverse order: ", floats)
}
```
執行結果：
```
Sorted strings in reverse order:  [Grin Gopher Go Delta Bravo Alpha]
Sorted integers in reverse order:  [78 67 56 25 19 14]
Sorted floats in reverse order:  [176.8 57.4 20.8 19.5]
```
## 7.9 error介面

最常看到的```error```型態，在golang的底層也是interface
```
type error interface {
  Error() string
}
```
它只是一個單一方法並回傳錯誤訊息的介面型別

- **使用 errors.New()**

建構error最簡單的方式是呼叫errors.New，[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/errors/errors.go;l=58)
```
func New(text string) error {
	return &errorString{text}
}
```
它回傳指定錯誤訊息的新error

範例：
```
func checkHandsomeMan(username string) (bool, error) {
	if username != "Barney" {
		return true, errors.New("You are ugly")
	}
	return false, nil
}

func main() {
	if _, err := checkHandsomeMan("Peter"); err != nil {
		fmt.Println(err)
	}
}
```
執行結果：
```
You are ugly
```

- **使用 fmt.Errorf()**

不過呼叫```errors.New```比較少見，因為有```fmt.Errorf```這個方便的包裝函式，它能執行字串格式化
```
func Errorf(format string, a ...any) error
```
這個函式封裝了```errors.New```，也間接實踐了error這個介面，[原始碼](https://cs.opensource.google/go/go/+/refs/tags/go1.19.3:src/fmt/errors.go;l=17)

範例：
```
func checkHandsomeMan(username string) (bool, error) {
	if username != "Barney" {
		return true, fmt.Errorf("%s, you are ugly", username)
	}
	return false, nil
}

func main() {
	if _, err := checkHandsomeMan("Peter"); err != nil {
		fmt.Println(err)
	}
}
```
執行結果：
```
Peter, you are ugly
```

- **自己定義 error structure**
```
// 定義custom error struct
type MyError struct {
	time    string
	message string
}

// 實作Error Interface的方法
func (e MyError) Error() string {
	return fmt.Sprintf("%s\nThe time is:%s", e.message, e.time)
}

// 拋出錯誤的函式
func throw() error {
	return MyError{
		time:    time.Now().Format(time.RFC3339),
		message: "This is error circumstance",
	}
}
```
呼叫run()：
```
func main() {
	if err := throw(); err != nil {
		fmt.Println(err)
	}
}
```
執行結果：
```
This is error circumstance
The time is:2022-11-24T03:58:56+08:00
```
