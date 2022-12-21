# CH9 共用變數並行性

## Agenda
- 9.1 Race Condition
- 9.2 Mutex
- 9.3 RWMutex
- 9.4 記憶體同步
- 9.5 Lazy Initialization: sync.Once
- 9.6 競爭檢查器
- 9.7 Example: Concurrent Non-Blocking Cache
- 9.8 Goroutines and Threads

## 9.1 Race Condition
- 無法確認兩個goroutine中的事件先後順序時, 稱為並行
- 若一函式被併行呼叫後結果能維持正確, 稱為並行性安全
- 函式被併行呼叫未能產生正確結果有很多原因, 此章節專門討論race condition
- race condition發生在多個並行的goroutine存取同一個變數且至少有一個寫入時
```
package bank
var balance int
func Deposit(amount int) { balance = balance + amount }
func Balance() int { return balance }
```
```
// Alice:
go func() {
    bank.Deposit(200)
    fmt.Println("=", bank.Balance())
}()

// Bob:
go bank.Deposit(100)
```
### 避免方法

不要寫入共用變數

把會發生race condition的code放在同一個goroutine (monitor pattern)
```
// Package bank provides a concurrency-safe bank with one account.
package bank

var deposits = make(chan int) // send amount to deposit
var balances = make(chan int) // receive balance
func Deposit(amount int) { deposits <- amount }
func Balance() int { return <-balances }

func teller() {
    var balance int // balance is confined to teller goroutine
    for {
        select {
            case amount := <-deposits:
                balance += amount
            case balances <- balance:
        }
    }
}

func init() {
    go teller() // start the monitor goroutine
}
```

允許多個goroutine存取同一個變數, 但要用互斥鎖

## 9.2 Mutex 互斥鎖

用容量為1的channel當lock
```
var (
    sema = make(chan struct{}, 1) // a binary semaphore guarding balance
    balance int
)

func Deposit(amount int) {
    sema <- struct{}{} // acquire token
    balance = balance + amount
    <-sema // release token
}

func Balance() int {
    sema <- struct{}{} // acquire token
    b := balance
    <-sema // release token
    return b
}
```

用sync.Mutex

```
import "sync"

var (
    mu sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount
    mu.Unlock()
}

func Balance() int {
    mu.Lock()
    b := balance
    mu.Unlock()
    return b
}
```

傳統上被mutex保護的變數要緊接著宣告在mutex之後

用defer來確保mutex最後會被unlock, defer成本較高, 但是對併行程式來說可讀性重要於提早最佳化

```
func Balance() int {
    mu.Lock()
    defer mu.Unlock()
    return balance
}
```

## 9.3 RWMutex
- 可多個goroutine同時讀取, 但寫入時只能單一goroutine
- RWMutex更耗資源, 用於常常處於競爭狀態, 且讀取的需求較多的情況

```
var mu sync.RWMutex
var balance int

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount
    mu.Unlock()
}

func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```

## 9.4 記憶體同步
- 多處理器的架構下, 各個處理器有自己的cache, cache不一定會更新到最新狀態, 不同的編譯緝或CPU會有不同的行為
- 用mutex或是channel可以確保資料被更新到各個cache裡

## 9.5 Lazy Initialization: sync.Once

sync.Once確保函式只會被執行一次

```
var loadIconsOnce sync.Once
var icons map[string]image.Image

func loadIcons() {
    icons = make(map[string]image.Image)
    icons["spades.png"] = loadIcon("spades.png")
    icons["hearts.png"] = loadIcon("hearts.png")
    icons["diamonds.png"] = loadIcon("diamonds.png")
    icons["clubs.png"] = loadIcon("clubs.png")
}

// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

## 9.6 競爭檢查器
- 檢查是否有多個goroutine在沒有同步化操作下寫入共用變數
- 加上 -race 參數
```
$ go test -run=TestConcurrent -race -v gopl.io/ch9/memo1
=== RUN TestConcurrent
...
WARNING: DATA RACE
Write by goroutine 36:
runtime.mapassign1()
~/go/src/runtime/hashmap.go:411 +0x0
gopl.io/ch9/memo1.(*Memo).Get()
~/gobook2/src/gopl.io/ch9/memo1/memo.go:32 +0x205
Previous write by goroutine 35:
runtime.mapassign1()
~/go/src/runtime/hashmap.go:411 +0x0
gopl.io/ch9/memo1.(*Memo).Get()
~/gobook2/src/gopl.io/ch9/memo1/memo.go:32 +0x205
...
Found 1 data race(s)
FAIL gopl.io/ch9/memo1 2.393s
```

## 9.7 Example: Concurrent Non-Blocking Cache

用互斥鎖

```
type entry struct {
    res result
    ready chan struct{} // closed when res is ready
}

func New(f Func) *Memo {
    return &Memo{f: f, cache: make(map[string]*entry)}
}

type Memo struct {
    f Func
    mu sync.Mutex // guards cache
    cache map[string]*entry
}

func (memo *Memo) Get(key string) (value interface{}, err error) {
    memo.mu.Lock()
    e := memo.cache[key]
    if e == nil {
        // This is the first request for this key.
        // This goroutine becomes responsible for computing
        // the value and broadcasting the ready condition.
        e=&entry{ready: make(chan struct{})}
        memo.cache[key] = e
        memo.mu.Unlock()
        e.res.value, e.res.err = memo.f(key)
        close(e.ready) // broadcast ready condition
    } else {
        // This is a repeat request for this key.
        memo.mu.Unlock()
        <-e.ready // wait for ready condition
    }
    return e.res.value, e.res.err
}

func httpGetBody(url string) (interface{}, error) {
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}

m := memo.New(httpGetBody)

for url := range incomingURLs() {
    start := time.Now()
    value, err := m.Get(url)
    if err != nil {
        log.Print(err)
    }
    fmt.Printf("%s, %s, %d bytes\n", url, time.Since(start), len(value.([]byte)))
}

```

用monitor


```
// Func is the type of the function to memoize.
type Func func(key string) (interface{}, error)

// A result is the result of calling a Func.
type result struct {
    value interface{}
    err error
}

type entry struct {
    res result
    ready chan struct{} // closed when res is ready
}

// A request is a message requesting that the Func be applied to key.
type request struct {
    key string
    response chan<- result // the client wants a single result
}

type Memo struct{ requests chan request }

// New returns a memoization of f. Clients must subsequently call Close.
func New(f Func) *Memo {
    memo := &Memo{requests: make(chan request)}
    go memo.server(f)
    return memo
}

func (memo *Memo) Get(key string) (interface{}, error) {
    response := make(chan result)
    memo.requests <- request{key, response}
    res := <-response
    return res.value, res.err
}

func (memo *Memo) Close() { close(memo.requests) }

func (memo *Memo) server(f Func) {
    cache := make(map[string]*entry)
    for req := range memo.requests {
        e := cache[req.key]
        if e == nil {
            // This is the first request for this key.
            e=&entry{ready: make(chan struct{})}
            cache[req.key] = e
            go e.call(f, req.key) // call f(key)
        }
        go e.deliver(req.response)
    }
}

func (e *entry) call(f Func, key string) {
    // Evaluate the function.
    e.res.value, e.res.err = f(key)
    // Broadcast the ready condition.
    close(e.ready)
}

func (e *entry) deliver(response chan<- result) {
    // Wait for the ready condition.
    <-e.ready
    // Send the result to the client.
    response <- e.res
}

func httpGetBody(url string) (interface{}, error) {
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}

m := memo.New(httpGetBody)

for url := range incomingURLs() {
    start := time.Now()
    value, err := m.Get(url)
    if err != nil {
        log.Print(err)
    }
    fmt.Printf("%s, %s, %d bytes\n", url, time.Since(start), len(value.([]byte)))
}

```

## 9.8 Goroutines and Threads

可成長的stack

- stack是一個記憶體區塊, 用來存暫停中的function的區域變數
- os thread的stack通常固定2MB, 對單一goroutine來說太大, 對一次會開幾百個goroutine的狀況又不夠用
- goroutine的stack大小為動態的, 一開始為2KB, 可以依需求長大至1GB

goroutine的排程

- os thread切換需要做full context switch, 很慢
- go用m:n排程, gorouting切換不需要底層context switch, 較快
- 用 GOMAXPROCS 決定要用幾個thread, 預設值為cpu數量

goroutine沒有識別名稱

- go不希望開發者直接去存取goroutine, 所以故意設計為沒有識別名稱


