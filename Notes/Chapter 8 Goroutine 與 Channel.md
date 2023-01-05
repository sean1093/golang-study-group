# Golang ch8 Goroutines and Channels

**8.1. Goroutines**

**8.2. Example: Concurrent Clock Server**

**8.3. Example: Concurrent Echo Server**

**8.4. Channels**

**8.5. Looping in Parallel**

**8.6. Example Concurrent Web Crawler**

**8.7. Multiplexing with select**

**8.8. Example: Concurrent Directory Traversal**

**8.9. Cancellation**

**8.10. Example: Chat Server**

---

Concurrent programming是由數個獨立活動組成的程式。利用concurrency來隱藏I/O操作的latency且使用到多處理器的功能。

Concurrent programming有兩種，一種是communicating sequential processes (CSP)，包含goroutines和channels；另一種是傳統的shared memory multithreading，就是使用threads。

# 8.1. Goroutines

每一個並行執行的活動就是一個goroutine。兩個function可以同時被active。

程式一開始只有一個main function，也叫做main goroutine。main function結束時，所有goroutines會立即被終止且程式結束。只有main function回傳或是程式結束之外，goroutine裡面不能用程式碼去結束另一個goroutine。

建立一個goroutine：

```go
f() // 呼叫f()，等f()回傳
go f() // 建立一個goroutine去呼叫f()，不等f()回傳
```

在main goroutine中計算第45個Fibonacci數是多少：

```go
func main() {
	go spinner(100 * time.Millisecond)
	const n = 45
	fibN := fib(n) // slow
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x-1) + fib(x-2)
}
```

輸出

```go
$ go build spinner/main.go
$ ./main
|
Fibonacci(45) = 1134903170
```

# 8.2. Example: Concurrent Clock Server

Networking是很適合用concurrency的領域，因為很多server需要同時處理多個client的conections且每個client都是彼此獨立的。

Sequential clock server每秒印出目前的時間給client：

ch8/clock1

```go
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		handleConn(conn) // handle one connection at a time
	}
}

func handleConn(c net.Conn) {
	defer c.Close()
	for {
		_, err := io.WriteString(c, time.Now().Format("15:04:05\n"))
		if err != nil {
			return // e.g., client disconnected
		}
		time.Sleep(1 * time.Second)
	}
}
```

輸出

使用nc工具程式讓client連到server：

```go
$ go build clock1/clock.go
$ ./clock &
$ nc localhost 8000
11:41:27
11:41:28
11:41:29
11:41:30
11:41:31
^C
```

使用netcat寫一個read-only的client，用來連到server：

ch8/netcat1

```go
func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	mustCopy(os.Stdout, conn)
}

func mustCopy(dst io.Writer, src io.Reader) { // 一個工具程式用來把輸入的值複製後輸出
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

輸出

```bash
$ go build netcat1/netcat.go
$ ./netcat
12:11:13
12:11:14
12:11:15
12:11:16
12:11:17
12:11:18
^C

$ killall clock
```

```bash

$ ./netcat

12:11:20
12:11:21
12:11:22
12:11:23
12:11:24
^C
```

修改clock server，讓它可同時接收多個client：

ch8/clock2

```go
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn) // handle connections concurrently
	}
```

輸出

```bash
$ go build clock2/clock.go
$ ./clock &
$ go build netcat1/netcat.go
$ ./netcat
12:37:15
12:37:16
12:37:17
12:37:18
12:37:19
^C

$ killall clock
```

```bash

$ ./netcat
12:37:16
12:37:17
12:37:18
12:37:19
12:37:20
^C

```

# 8.3. Example: Concurrent Echo Server

每個connection都使用多個goroutine。

Sequential echo server把輸入印出來，加上大、中、小聲的回音效果。

ch8/rverb1

```go
func echo(c net.Conn, shout string, delay time.Duration) {
	fmt.Fprintln(c, "\t", strings.ToUpper(shout))
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", shout)
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		echo(c, input.Text(), 1*time.Second)
	}
	// NOTE: ignoring potential errors from input.Err()
	c.Close()
}
```

修改netcat client，讓它利用concurrency達到既可傳送input到server也可以複製server的response output到terminal。

ch8/netcat2

```go
func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	go mustCopy(os.Stdout, conn) // 等server輸出response文字到terminal
	mustCopy(conn, os.Stdin) // 把terminal輸入的文字傳給server
}
```

輸出

```bash
$ go build reverb1/reverb.go
$ ./reverb &
$ go build netcat2/netcat.go
$ ./netcat
Hello?
	 HELLO?
	 Hello?
	 hello?
Is there anybody there?
	 IS THERE ANYBODY THERE?
Yooo-hooo!
	 Is there anybody there?
	 is there anybody there?
	 YOOO-HOOO!
	 Yooo-hooo!
	 yooo-hooo!
^D
$ killall reverb
```

問題：第三個的回音要等第二個個回音先處理完，才會有回音。

所以會需要多個goroutine來處理多個回音。

修改reverb server，讓它利用concurrency達到回音重疊的效果。

ch8/reverb2

```go
func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		go echo(c, input.Text(), 1*time.Second) // 建立一個goroutine呼叫echo()印出回音
	}
	// NOTE: ignoring potential errors from input.Err()
	c.Close()
}
```

輸出

```bash
$ go build reverb2/reverb.go
$ ./reverb &
$ go build netcat2/netcat.go
$ ./netcat
Hello?
	 HELLO?
	 Hello?
	 hello?
Is there anybody there?
	 IS THERE ANYBODY THERE?
Yooo-hooo!
	 YOOO-HOOO!
	 Is there anybody there?
	 Yooo-hooo!
	 is there anybody there?
	 yooo-hooo!
^D
$ killall reverb
```

即使處理多個input也不用等前一個input完成，因為reverb server是可處理多個輸入。

# 8.4. Channels

在各個goroutines之間的connections。Channel是一個reference，用built-in make function建立。可指定channel的capacity，使用make function的第二個選擇性的參數代表channel的capacity，capacity大於0即為buffered channel；反之，沒有設定capacity或是設為0都是unbuffered channel。Channel是一個溝通機制，讓一個goroutine傳值給另一個goroutine。零值是nil。

每個channel是為了特定的type的渠道，這個type叫Channel的element type。element type為int的channel，寫成chan int。

建立一個channel：

```go
ch := make(chan int) // ch的type是‘chan int'
```

Communications是包含channel的兩個基本操作send和receive。

```go
ch <- x   // a send statement
x = <-ch  // a receive expression in an assignement statement
<-ch      // a receive statement; result is discarded
```

close操作，表示不再傳值到某個channel的一個flag，若再傳值進去某channel就會panic。在已經close的channel上做receive的操作時，會取出已send的值直到沒有值為止。沒有值之後的receive操作立即完成，並且產生channel的element type之零值，例如：chan int的type，此時零值就是0。

關閉一個channel：

```go
close(ch)
```

## 8.4.1. Unbuffered Channels

這種channel上的send會block到sending goroutine，直到另一個goroutine在同一個channel上執行相對應的receive，此時值可以傳輸在兩個goroutine中。反之，若receive操作先執行，那receiving goroutine會被block直到另一個goroutine執行一個send到相同的channel。又稱為Synchronous channels。

用channel來同步兩個goroutines，可以讓程式等背景goroutine都完成了才結束。

將netcat client改用unbuffered channel來做背景goroutine與main goroutine的同步：

ch8/netcat3

```go
func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	done := make(chan struct{})
	go func() {
		io.Copy(os.Stdout, conn) // NOTE: ignoring errors
		log.Println("done")
		done <- struct{}{} // signal the main goroutine
	}()
	mustCopy(conn, os.Stdin)
	conn.Close()
	<-done // wait for background goroutine to finish
}
```

透過channel傳遞的訊息，有兩個重點：1. 每個訊息都有一個值，用來表示某個發生的時間點的話，稱為event。2. 當event沒有夾帶額外的訊息，這時候就是單純用來做同步，因此一般會使用bool或int來表示，done < -1的寫法更簡短。

## 8.4.2. Pipelines

一個pipeline代表一個goroutine的輸出是另一個goroutine的輸入，而使用channel可以將goroutine連結起來。

一個三階段的pipeline（圖8.1）：

ch8/pipeline1

```go
func main() {
	naturals := make(chan int)
	squares := make(chan int)

	// Counter
	go func() {
		for x := 0; ; x++ {
			naturals <- x // send：放0,1,2,3,...到naturals channel
		}
	}()

	// Squarer
	go func() {
		for {
			x := <-naturals // receive:從naturals channel取出0,1,2,3,...，存到x
			squares <- x * x // send:把平方後的結果放到squares channel
		}
	}()

	// Printer (in main goroutine)
	for {
		fmt.Println(<-squares) // receive:從squares channel取出0,1,4,9,...到無限大
	}
}
```

若這時候將Counter goroutine中的for loop改成有限的的話：

```go
	// Counter
	go func() {
		for x := 0; x < 5; x++ {
			naturals <- x // send：放0,1,2,3,4到naturals channel
		}
	}()
```

會出現以下的錯誤：

```bash
0
1
4
9
16
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/.../gopl.io/ch8/pipeline1/main.go:33 +0xf4

goroutine 19 [chan receive]:
main.main.func2()
	/.../gopl.io/ch8/pipeline1/main.go:26 +0x38
created by main.main
	/.../gopl.io/ch8/pipeline1/main.go:24 +0xe4
```

如果sender知道不會再有值要傳入channel，有用的方式是將這個事實告訴receiver goroutines它們可以停止等待了。因此，需要使用到close操作。

在已經關閉的channel抽乾（drained）之後，也就是最後一個element已被接收後，後續的receive操作可以進行且不會被block，但會產出element type的零值。關閉naturals channel會導致計算平方的for loop變成無只境的接收零值，並且把零值傳給Printer。

沒有辦法直接檢查channel是已經關閉，但有一種接收操作可以產生兩個結果，回傳ok是true表示成功接收；ok是false表示是從已經關閉且抽乾了的channel接收。

修改計算平方的for loop，在naturals channel抽乾時，停止並關閉squares channel：

```go
	// Squarer
	go func() {
		for {
			x, ok := <-naturals
			if !ok {
				break // channel was closed and drained
			}
			squares <- x * x
		}
		close(squares)
	}()
```

更常見的寫法是搭配range的loop。

當Counter goroutine完成了有限的elements，就關閉naturals channel，導致Squarer goroutine的loop也結束並關閉squares channel。最終main goroutine完成了loop且程式也跟個結束。

ch8/pipeline2

```go
func main() {
	naturals := make(chan int)
	squares := make(chan int)

	// Counter
	go func() {
		for x := 0; x < 100; x++ {
			naturals <- x
		}
		close(naturals)
	}()

	// Squarer
	go func() {
		for x := range naturals {
			squares <- x * x
		}
		close(squares)
	}()

	// Printer (in main goroutine)
	for x := range squares {
		fmt.Println(x)
	}
}
```

Channel用完不一定要close，只有在必須告訴receive的goroutine時，才需要呼叫close。Garbage collector判斷channel是否已無法觸及時，不管是否已經關閉都會回收資源。關閉channel的另一個用途是broadcast機制。

## 8.4.3. Unidirectional Channel Types

典型的用法是將channel當成參數傳入function。為了避免誤用並且為了註明channel的意圖，將channel分成兩種單向的channel：1. send-only channel、2. receive-only channel。

```go
chan<- int // a send-only channl of int
<-chan int // a receive-only channel of int
```

修改pipeline程式，使用單向channel type來計算平方。

```go
func counter(out chan<- int) { // out: send-only channel
	for x := 0; x < 100; x++ {
		out <- x
	}
	close(out)
}

func squarer(out chan<- int, in <-chan int) { // out: send-only channel, in: receive-only channel
	for v := range in {
		out <- v * v
	}
	close(out)
}

func printer(in <-chan int) {
	for v := range in {
		fmt.Println(v)
	}
}

func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go counter(naturals)
	go squarer(squares, naturals)
	printer(squares) // 在main goroutine中
}
```

## 8.4.4. Buffered Channels

具有一個存放elements的queue，queue的size是在建立時就決定了。可用built-in functions：1. cap(ch) 2. len(ch)，但是因為concurrency的關係，所以len可能在呼叫後就改變了，因此用途有限。

ch與它refer到的capacity為3的buffered channel：（圖8.2）

```go
ch = make(chan string, 3)
```

把A,B,C三個值send到ch channel不會block到goroutine：

```go
ch <- 'A'
ch <- 'B'
ch <- 'C'
```

但這時候，goroutine再send第四個時就會被block：（圖8.3）

如果receive一個值出來後，channel就變成不空也不滿：（圖8.4）

```go
fmt.Println(<-ch) // 'A'
```

再做兩次以上的receive操作後，channel就會再次變空且第四次會被block。

這個範例中，是在相同的goroutine中做send與receive操作，但真實世界中通常都是在不同的goroutines。新手常會在單一個goroutine中使用buffered channel來當queue使用，但因為channel跟goroutine的排程有高度相關，所以如果沒有另一個goroutine從channel中接收，那sende或是整個程式都有被永遠block的風險。因此，若想簡單的用queue，就用slice。

應用buffered channel，對三個不同位置但內容相同的server發出parallel requests，即使receive到最快回來的response就return，也是沒問題的：

```go
func mirroredQuery() string {
	responses := make(chan string, 3)
	go func() { responses <- request("asia.gopl.io") }()
	go func() { responses <- request("europe.gopl.io") }()
	go func() { responses <- request("americas.gopl.io") }()
	return <-responses // return the quickest response
}

func request(hostname string) (response string) { /* ... */ }
```

但如果使用unbuffered channel的話，另外兩個比較慢的goroutines會卡在一直嘗試send他們的response到沒有goroutine會receive的channel上，這種狀況稱為goroutine leak，是個bug。而goroutine leak是不會被自動回收的，所以確保不需要的goroutine自己終止是很重要的。

使用unbuffered channels多用在保證同步；而使用buffered channel時，操作是解耦的（decoupled）。如果已知傳送的值是有上限的，若沒有分配足夠的buffer capacity會導致程式發生deadlock。

Channel的buffer也會影響到程式的效能。三個師傅（烤蛋糕、裝飾、寫字）來解釋unbuffered channel跟capacity為一的buffered channel。unbuffered的話就要等，或是師傅速度均等且buffer夠大就會很順暢。但是如果前面比後面快，buffer會一直是滿的；而後面比前面快，buffer會一直是空的。這樣的話，有buffer也沒用。解法是在建立另一個在相同channel上運作的goroutine。

# 8.5. Looping in Parallel

常見的concurrency模式是在loop中平行的執行。

製作縮圖的function：

ch8/thumbnail

```go
// ImageFile reads an image from infile and writes
// a thumbnail-size version of it in the same directory.
// It returns the generated file name, e.g. "foo.thumb.jpeg".
func ImageFile(infile string) (string, error) {
	ext := filepath.Ext(infile) // e.g., ".jpg", ".JPEG"
	outfile := strings.TrimSuffix(infile, ext) + ".thumb" + ext
	return outfile, ImageFile2(outfile, infile)
}
```

用一個sequential programming的loop，將filename的list跑一遍，每次就處理一個圖片的縮圖（embarrassingly parallel）：

```go
// makeThumbnails makes thumbnails of the specified files.
func makeThumbnails(filenames []string) {
	for _, f := range filenames {
		if _, err := thumbnail.ImageFile(f); err != nil {
			log.Println(err)
		}
	}
}
```

錯誤的在平行中執行縮圖的操作：

```go
// NOTE: incorrect!
func makeThumbnails2(filenames []string) {
	for _, f := range filenames {
		go thumbnail.ImageFile(f) // NOTE: ignoring errors
	}
}
```

為何跑很快？答案：在完成該做的事之前就return了，因為它將所有goroutines都啟動，但是沒等他們完成。

沒有直接的方式等待一個goroutine完成，但是可以改變由內部的goroutine回報它完成了給外部的goroutine，透過send一個event到內外部共用的channel。

正確的在平行中執行縮圖的操作，內部goroutine已知有多少檔案，外部goroutine只需要計算完所有的event就能return：

```go
// makeThumbnails3 makes thumbnails of the specified files in parallel.
func makeThumbnails3(filenames []string) {
	ch := make(chan struct{})
	for _, f := range filenames {
		go func(f string) {
			thumbnail.ImageFile(f) // NOTE: ignoring errors
			ch <- struct{}{}
		}(f)
	}

	// Wait for goroutines to complete.
	for range filenames {
		<-ch
	}
}
```

如果製作縮圖失敗的話，也要回傳error的錯誤做法：

```go
// makeThumbnails4 makes thumbnails for the specified files in parallel.
// It returns an error if any step failed.
func makeThumbnails4(filenames []string) error {
	errors := make(chan error)

	for _, f := range filenames {
		go func(f string) {
			_, err := thumbnail.ImageFile(f)
			errors <- err
		}(f)
	}

	for range filenames {
		if err := <-errors; err != nil {
			return err // NOTE: incorrect: goroutine leak!
		}
	}

	return nil
}
```

因為errors channel沒有receive goroutine接收值，所以sender會被block，因此導致goroutine leak，這是個微妙的bug。

解法1. 使用足夠的buffered channel，一定可以send到channel。

```go
// makeThumbnails5 makes thumbnails for the specified files in parallel.
// It returns the generated file names in an arbitrary order,
// or an error if any step failed.
func makeThumbnails5(filenames []string) (thumbfiles []string, err error) {
	type item struct {
		thumbfile string
		err       error
	}

	ch := make(chan item, len(filenames))
	for _, f := range filenames {
		go func(f string) {
			var it item
			it.thumbfile, it.err = thumbnail.ImageFile(f)
			ch <- it
		}(f)
	}

	for range filenames {
		it := <-ch
		if it.err != nil {
			return nil, it.err
		}
		thumbfiles = append(thumbfiles, it.thumbfile)
	}

	return thumbfiles, nil
}
```

解法2. 建立另一個goroutine在第一個回傳錯誤的時候抽乾channel。

最終版，回傳所有新的檔案的總byte數：

```go
// makeThumbnails6 makes thumbnails for each file received from the channel.
// It returns the number of bytes occupied by the files it creates.
func makeThumbnails6(filenames <-chan string) int64 {
	sizes := make(chan int64)
	var wg sync.WaitGroup // number of working goroutines
	for f := range filenames {
		wg.Add(1)
		// worker
		go func(f string) {
			defer wg.Done()
			thumb, err := thumbnail.ImageFile(f)
			if err != nil {
				log.Println(err)
				return
			}
			info, _ := os.Stat(thumb) // OK to ignore error
			sizes <- info.Size()
		}(f)
	}

	// closer
	go func() {
		wg.Wait()
		close(sizes)
	}()

	var total int64
	for size := range sizes {
		total += size
	}
	return total
}
```

搭配圖8.5的事件順序圖：垂直線goroutine、細線sleep、粗線activity、箭頭event（表示兩個goroutine同步）

# 8.6. Example Concurrent Web Crawler

改良Ch5.6的以廣度優先順序的web crawler，要使它concurrent就是獨立呼叫craw function來利用I/O平行化（parallelism）。

Concurrent的web crawler：

ch8/crawl1

```go
func crawl(url string) []string {
	fmt.Println(url)
	list, err := links.Extract(url)
	if err != nil {
		log.Print(err)
	}
	return list
}

func main() {
	worklist := make(chan []string) // 改用channel來取代用slice當成的queue

	// Start with the command-line arguments.
	go func() { worklist <- os.Args[1:] }() // 輸入一個URL

	// Crawl the web concurrently.
	seen := make(map[string]bool)
	for list := range worklist {
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				go func(link string) {
					worklist <- crawl(link)
				}(link)
			}
		}
	}
}
```

解決了loop變數抓取的問題，Crawl goroutine拿link當作明確的參數，避免了loop中用到link的function都用到相同的memory address。

解決了deadlock，把URL send到worklist channel必須獨立出一個goroutine，避免掉main goroutine和crawler goroutine都企圖send給對方卻沒有一方receive的情況。

輸出

```bash
$ go build gopl.io/ch8/crawl1/findlinks.go
$ ./crawl1 http://www.gopl.io/
http://www.gopl.io/
https://golang.org/help/

https://golang.org/doc/
https://golang.org/blog/
...
2015/07/15 18:22:12 Get ...: dial tcp: lookup blog.golang.org: no such host
2015/07/15 18:22:12 Get ...: dial tcp 23.21.222.120:443: socket:
                                                        too many open files
...
```

但即使已高度concurrent了，還是有兩個問題：

問題1. DNS查詢失敗。因為程式一下子建立太多連線而超出限制，程式太過parallel。無限制的平行化不是好的做法。

用capacity n的buffered channel來限制平行化，就是設計一個稱為counting semaphore（計數號誌）的concurrency primitive模型。

問題1的解法是修改crawl function，在裡面使用buffered channel，限制最多20個concurrent requests：

ch8/crawl2

```go
// tokens is a counting semaphore used to
// enforce a limit of 20 concurrent requests.
var tokens = make(chan struct{}, 20)

func crawl(url string) []string {
	fmt.Println(url)
	tokens <- struct{}{} // 取得一個token
	list, err := links.Extract(url)
	<-tokens // 釋放token

	if err != nil {
		log.Print(err)
	}
	return list
}
```

問題2. 程式不會終止，即使已找完所有的links。像是當worklist channel是空的且沒有crawl goroutine是active時，會需要有個終止main loop的方式。

問題2的解法1. 是修改main function，用一個counter n紀錄還沒send到worklist channel的數量。

ch8/crawl2

```go
func main() {
	worklist := make(chan []string)
	var n int // number of pending sends to worklist

	// Start with the command-line arguments.
	n++
	go func() { worklist <- os.Args[1:] }()

	// Crawl the web concurrently.
	seen := make(map[string]bool)
	for ; n > 0; n-- {
		list := <-worklist
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				n++
				go func(link string) {
					worklist <- crawl(link)
				}(link)
			}
		}
	}
}
```

問題2的解法2. 用原本Ch5.6的crawl function，沒有使用token，但是建立了20個crawler goroutine去呼叫crawl function，確保同時最多20個requests：

ch8/crawl3

```go
func crawl(url string) []string {
	fmt.Println(url)
	list, err := links.Extract(url)
	if err != nil {
		log.Print(err)
	}
	return list
}

func main() {
	worklist := make(chan []string)  // lists of URLs, may have duplicates
	unseenLinks := make(chan string) // 去除重複的URLs

	// Add command-line arguments to worklist.
	go func() { worklist <- os.Args[1:] }()

	// Create 20 crawler goroutines to fetch each unseen link.
	for i := 0; i < 20; i++ {
		go func() {
			for link := range unseenLinks {
				foundLinks := crawl(link)
				go func() { worklist <- foundLinks }()
			}
		}()
	}

	// The main goroutine de-duplicates worklist items
	// and sends the unseen ones to the crawlers.
	seen := make(map[string]bool) // 限制在main goroutine
	for list := range worklist {
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				unseenLinks <- link // 把沒看過的link加到unseenLinks channel
			}
		}
	}
}
```

# 8.7. Multiplexing with select

就是switch case的概念。

發射火箭前的倒數計時，使用會定期send event並回傳一個channel的time.Tick function：

ch8/countdown1

```go
func main() {
	fmt.Println("Commencing countdown.")
	tick := time.Tick(1 * time.Second) // tick是一個channel，定期會有event被send進去
	for countdown := 10; countdown > 0; countdown-- {
		fmt.Println(countdown)
		<-tick // receive: 從tick channel接收每秒進來的event
	}
	launch()
}

func launch() {
	fmt.Println("Lift off!")
}
```

修改countdown，讓它能在倒數計時過程中按return後abort發射火箭：

ch8/countdon2

```go
func main() {
	// ...略...
	abort := make(chan struct{})
	go func() {
		os.Stdin.Read(make([]byte, 1)) // read a single byte
		abort <- struct{}{}
	}()
	// ...略...
}
```

在loop的每一圈都需要等event抵達兩個channel的其中一個，所以需要一個select statement。case可指定在某個channel上的send或receive操作。

```go
select {
case <-ch1: // 從ch1 channel receive
	// ...
case x := <-ch2: // 從ch2 channel receive後存到x
	// ...use x...
case ch3 <- y: // 把y send到ch3 channel
	// ...
default:
	// ...
}
```

修改countdown，讓它經過10秒且沒有按return就發射火箭。

ch8/countdown2

```jsx
func main() {
	// ...略...
	fmt.Println("Commencing countdown.  Press return to abort.")
	select {
	case <-time.After(10 * time.Second): // 回傳一個channel並且開始一個goroutine用來send一個值到這個channel
		// Do nothing.
	case <-abort: // abort channel有receive到event就執行
		fmt.Println("Launch aborted!")
		return
	}
	launch()
}
```

Non-blocking communication可以用select的default來指定沒有立即處理的溝通要如何處理。

```go
select {
case <-abort:
	fmt.Printf("Launch aborted!\n")
	return
default:
	// do nothing
}
```

# 8.8. Example: Concurrent Directory Traversal

用抽乾的function列出dir目錄下的所有紀錄。

建立一個report，列出指定目錄的disk usage：

ch8/du1

```go
func main() {
	// Determine the initial directories.
	flag.Parse()
	roots := flag.Args()
	if len(roots) == 0 {
		roots = []string{"."}
	}

	// Traverse the file tree.
	fileSizes := make(chan int64)
	go func() { // backgound goroutine
		for _, root := range roots {
			walkDir(root, fileSizes)
		}
		close(fileSizes)
	}()

	// Print the results.
	var nfiles, nbytes int64
	for size := range fileSizes { // main goroutine
		nfiles++
		nbytes += size
	}
	printDiskUsage(nfiles, nbytes)
}

func printDiskUsage(nfiles, nbytes int64) {
	fmt.Printf("%d files  %.1f GB\n", nfiles, float64(nbytes)/1e9)
}

// walkDir recursively walks the file tree rooted at dir
// and sends the size of each found file on fileSizes.
func walkDir(dir string, fileSizes chan<- int64) {
	for _, entry := range dirents(dir) {
		if entry.IsDir() {
			subdir := filepath.Join(dir, entry.Name())
			walkDir(subdir, fileSizes)
		} else {
			fileSizes <- entry.Size() // 把每個檔案大小都放到fileSizes channel
		}
	}
}

// dirents returns the entries of directory dir.
func dirents(dir string) []os.FileInfo {
	entries, err := ioutil.ReadDir(dir)
	if err != nil {
		fmt.Fprintf(os.Stderr, "du1: %v\n", err)
		return nil
	}
	return entries
}
```

輸出

```bash
go build du1/main.go
./main $HOME /usr /bin /
213201 files 62.7 GB
```

# 8.9. Cancellation

廣播一個event到一個channel，並讓多個goroutine可以在事情發生時收到並且事情發生之後還是能知道。

想指示goroutine停止，例如：client已斷線了，這時候server的計算也要停止。

一個goroutine直接中斷另一個goroutine是不行的，因為會讓所有共用變數留在undefined狀態。

廣播機制就是不send值，就close掉channel。

ch8/du4

```go
var done = make(chan struct{})

func cancelled() bool { // 用utility function來poll取消的狀態
	select {
	case <-done:
		return true
	default:
		return false
	}
}
```

# 8.10. Example: Chat Server

多個users互相廣播文字訊息的聊天server：

ch8/chat

```go
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}

	go broadcaster()
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		go handleConn(conn)
	}
}
```

```go
type client chan<- string // an outgoing message channel

var ( // 全域的channel
	entering = make(chan client)
	leaving  = make(chan client)
	messages = make(chan string) // all incoming client messages
)

func broadcaster() { // 廣播氣接收到其中一個client的event就廣播給其他的clients
	clients := make(map[client]bool) // all connected clients
	for {
		select {
		case msg := <-messages: // 廣播訊息到所有client的channel
			// Broadcast incoming message to all
			// clients' outgoing message channels.
			for cli := range clients {
				cli <- msg
			}

		case cli := <-entering:
			clients[cli] = true

		case cli := <-leaving:
			delete(clients, cli)
			close(cli)
		}
	}
}
```

輸出

```bash
go build chat/chat1.go
go build netcat3/netcat.go
```

```bash
./netcat
You are 127.0.0.1:61610 //user1
127.0.0.1:61631 has arrived
Hi!
127.0.0.1:61610: Hi!

127.0.0.1:61631: Hi yourself.
^C

./netcat
You are 127.0.0.1:61819 //user3
127.0.0.1:61631: Welcome
127.0.0.1:61631 has left
```

```bash

./netcat
You are 127.0.0.1:61631  //user2

127.0.0.1:61610: Hi!
Hi yourself.
127.0.0.1:61631: Hi yourself.

127.0.0.1:61610 has left

127.0.0.1:61819 has arrived
Welcome
127.0.0.1:61631: Welcome
^C
```
