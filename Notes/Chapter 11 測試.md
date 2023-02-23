# Chapter 11. Testing

## Agenda

- The go test Tool
- Test Functions
- Coverage
- Benchmark Functions
- Profiling
- Example

## The go test Tool

The `go test` subcommand is a test driver for Go packages that are organized according to certain
conventions. In a package directory, files whose names end with `_test.go` are not part of  the package ordinarily bui lt by `go build` but are a part of it when built by `go test`.

Three kinds of functions in `_test.go`:
- test functions: verify logic
- benchmark function: measure performance
- example function: compiled documentation

## Test Functions

```go
func TestSin(t *testing.T) { /* ... */ }
func TestCos(t *testing.T) { /* ... */ }
func TestLog(t *testing.T) { /* ... */ }
```

Note:
- Test function names must beg in with `Test`; the optional suffix Name must begin with a capital letter.
- The `t` parameter provides methods for reporting test failures and logging additional information.

To-be-tested Function

```go
// Package word provides utilities for word games. 
package word
// IsPalindrome reports whether s reads the same forward and backward.
// (Our first attempt.)
func IsPalindrome(s string) bool {
    for i := range s {
        if s[i] != s[len(s)-1-i] {
            return false
        }
    }
    return true
}
```

Corresponding test file

```go
package word
import "testing"
func TestPalindrome(t *testing.T) {
    if !IsPalindrome("detartrated") {
        t.Error(`IsPalindrome("detartrated") = false`)
    }
    if !IsPalindrome("kayak") {
        t.Error(`IsPalindrome("kayak") = false`)
    }
}
func TestNonPalindrome(t *testing.T) {
    if IsPalindrome("palindrome") {
        t.Error(`IsPalindrome("palindrome") = true`)
    }
}
```

Output

```shell
$ cd $GOPATH/src/gopl.io/ch11/word1
$ go test
ok gopl.io/ch11/word1 0.008s
```

More Tests

```go
func TestFrenchPalindrome(t *testing.T) {
    if !IsPalindrome("été") {
        t.Error(`IsPalindrome("été") = false`)
    }
}
func TestCanalPalindrome(t *testing.T) {
    input := "A man, a plan, a canal: Panama"
	if !IsPalindrome(input) {
		t.Errorf(`IsPalindrome(%q) = false`, input)
    }
}
```

Output

```shell
$ go test
--- FAIL: TestFrenchPalindrome (0.00s)
    word_test.go:28: IsPalindrome("été") = false
--- FAIL: TestCanalPalindrome (0.00s)
    word_test.go:35: IsPalindrome("A man, a plan, a canal: Panama") = false
FAIL
FAIL gopl.io/ch11/word1 0.014s
```

Common flags:
- The `-v` flag prints the name and exe cut ion time of each test in the package
- The `-run` flag , whose argument is a regular expression, causes go test to run only those tests whose function name match es the pattern


Table-driven testing

```go
func TestIsPalindrome(t *testing.T) {
    var tests = []struct {
        input string
        want bool
    }{
        {"", true},
        {"a", true},
        {"aa", true},
        {"ab", false},
        {"kayak", true},
        {"detartrated", true},
        {"A man, a plan, a canal: Panama", true},
        {"Evil I did dwell; lewd did I live.", true},
        {"Able was I ere I saw Elba", true},
        {"été", true},
        {"Et se resservir, ivresse reste.", true},
        {"palindrome", false}, // non-palindrome
        {"desserts", false}, // semi-palindrome
    }
    for _, test := range tests {
        if got := IsPalindrome(test.input); got != test.want {
            t.Errorf("IsPalindrome(%q) = %v", test.input, got)
        }
    }
}

```

Note:
- `t.Errorf` does not cause a panic or stop the exe cut ion of the test, unlike assertion failures in many test frameworks for other languages.
- Use `t.Fatal` or `t.Fatalf` to terminate test execution when needed.
- Test fai ure messages are usually of the form `"f(x) = y, want z"`, where f(x) explains the attempted operation and its input, y is the actual result, and z the expected result.

### Randomized Testing

Randomized testing explores a broader range of inputs by constructing inputs at random.

How?
1. Write an alternative implementation of the function that uses a less efficient but simpler and cle arer algor it hm, and che ck that bot h implementation s give the
same result. 
2. Create input values according to a pattern so that we know what output to expect.

```go
import "math/rand"
// randomPalindrome returns a palindrome whose length and contents
// are derived from the pseudo-random number generator rng.

func randomPalindrome(rng *rand.Rand) string {
    n := rng.Intn(25) // random length up to 24
    runes := make([]rune, n)
    for i := 0; i < (n+1)/2; i++ {
        r := rune(rng.Intn(0x1000)) // random rune up to '\u0999'
        runes[i] = r
        runes[n-1-i] = r
    }
    return string(runes)
}

func TestRandomPalindromes(t *testing.T) {
    // Initialize a pseudo-random number generator.
    seed := time.Now().UTC().UnixNano()
    t.Logf("Random seed: %d", seed)
    rng := rand.New(rand.NewSource(seed))
    for i := 0; i < 1000; i++ {
        p := randomPalindrome(rng)
        if !IsPalindrome(p) {
            t.Errorf("IsPalindrome(%q) = false", p)
        }
    }
}

```

Since randomize d tests are non-deterministic, it is critical that the log of the failing test record sufficient infor mat ion to reproduce the failure.

### Testing a Command

```go
// Echo prints its command-line arguments.
package main

import (
    "flag"
    "fmt"
    "io"
    "os"
    "strings"
)

var (
	n = flag.Bool("n", false, "omit trailing newline")
	s = flag.String("s", " ", "separator")
)

var out io.Writer = os.Stdout // modified during testing

func main() {
	flag.Parse()
	if err := echo(!*n, *s, flag.Args()); err != nil {
		fmt.Fprintf(os.Stderr, "echo: %v\n", err)
		os.Exit(1)
	}
}

func echo(newline bool, sep string, args []string) error {
	fmt.Fprint(out, strings.Join(args, sep))
	if newline {
		fmt.Fprintln(out)
	}
	return nil
}

```

```go
package main
import (
    "bytes"
    "fmt"
    "testing"
)

func TestEcho(t *testing.T) {
    var tests = []struct {
    newline bool
    sep string
    args []string
    want string
    }{
        {true, "", []string{}, "\n"},
        {false, "", []string{}, ""},
        {true, "\t", []string{"one", "two", "three"}, "one\ttwo\tthree\n"},
        {true, ",", []string{"a", "b", "c"}, "a,b,c\n"},
        {false, ":", []string{"1", "2", "3"}, "1:2:3"},
    }
	for _, test := range tests {
		descr := fmt.Sprintf("echo(%v, %q, %q)",
			test.newline, test.sep, test.args)
		out = new(bytes.Buffer) // captured output
		if err := echo(test.newline, test.sep, test.args); err != nil {
			t.Errorf("%s failed: %v", descr, err)
			continue
		}
		got := out.(*bytes.Buffer).String()
		if got != test.want {
			t.Errorf("%s = %q, want %q", descr, got, test.want)
		}
	}
}
```

### White-Box Testing

A black-box test assumes nothing about the package other than what is exposed by its API and specified by its documentation; the package’s internals are opaque.

A white-box test has privileged access to the internal functions and data structures of the package and can make observations and changes that an ordinary client cannot.

```go
package storage

import (
    "fmt"
    "log"
    "net/smtp"
)

func bytesInUse(username string) int64 { return 0 /* ... */ }
// Email sender configuration.
// NOTE: never put passwords in source code!
const sender = "notifications@example.com"
const password = "correcthorsebatterystaple"
const hostname = "smtp.example.com"
const template = `Warning: you are using %d bytes of storage, %d%% of your quota.`

func CheckQuota(username string) {
	used := bytesInUse(username)
	const quota = 1000000000 // 1GB
	percent := 100 * used / quota
	if percent < 90 {
		return // OK
	}
	msg := fmt.Sprintf(template, used, percent)
	auth := smtp.PlainAuth("", sender, password, hostname)
	err := smtp.SendMail(hostname+":587", auth, sender,
		[]string{username}, []byte(msg))
	if err != nil {
		log.Printf("smtp.SendMail(%s) failed: %s", username, err)
	}
}
```

Refactoring for white-box testing...

```go
var notifyUser = func(username, msg string) {
    auth := smtp.PlainAuth("", sender, password, hostname)
    err := smtp.SendMail(hostname+":587", auth, sender, []string{username}, []byte(msg))
    if err != nil {
        log.Printf("smtp.SendEmail(%s) failed: %s", username, err)
    }
}

func CheckQuota(username string) {
    used := bytesInUse(username)
    const quota = 1000000000 // 1GB
    percent := 100 * used / quota
    if percent < 90 {
        return // OK
    }
    msg := fmt.Sprintf(template, used, percent)
    notifyUser(username, msg)
}
```

Now we could write tests...

```go
package storage

import (
    "strings"
    "testing"
)

func TestCheckQuotaNotifiesUser(t *testing.T) {
	var notifiedUser, notifiedMsg string
	notifyUser = func(user, msg string) {
		notifiedUser, notifiedMsg = user, msg
	}
	// ...simulate a 980MB-used condition...
	const user = "joe@example.org"
	CheckQuota(user)
	if notifiedUser == "" && notifiedMsg == "" {
		t.Fatalf("notifyUser not called")
	}
	if notifiedUser != user {
		t.Errorf("wrong user (%s) notified, want %s",
			notifiedUser, user)
	}
	const wantSubstring = "98% of your quota"
	if !strings.Contains(notifiedMsg, wantSubstring) {
		t.Errorf("unexpected notification message <<%s>>, "+
			"want substring %q", notifiedMsg, wantSubstring)
	}
}
```

However, We must modify the test to restore the previous value so that subsequent tests observe no effect, and we must do this on all execution paths, including test failures and panics.

Use `defer`

```go
func TestCheckQuotaNotifiesUser(t *testing.T) {
    // Save and restore original notifyUser.
    saved := notifyUser
    defer func() { notifyUser = saved }()
    // Install the test's fake notifyUser.
    var notifiedUser, notifiedMsg string
    notifyUser = func(user, msg string) {
        notifiedUser, notifiedMsg = user, msg
    }
    // ...rest of test...
}

```

### Writing Effective Tests

Go expects test authors to do most of this work themselves, defining functions to `avoid repetition`, just as they would for ordinary programs.

A good test does not explode on failure but prints a clear and succinct description of the symptom of the problem, and perhaps other relevant facts about the context.

A good test should not give up after one fai lure but should try to report several errors in a single run, since the pattern of fai lures may itself be revealing.

The key to a good test is to start by implementing the concrete behavior that you want and only then use functions to simplify the code and eliminate repetition. Best results are rarely obtained by starting with a library of abstract, generic testing functions.

Bad example 
```go
import (
    "fmt"
    "strings"
    "testing"
)
// A poor assertion function.
func assertEqual(x, y int) {
    if x != y {
    panic(fmt.Sprintf("%d != %d", x, y))
    }
}
func TestSplit(t *testing.T) {
    words := strings.Split("a:b:c", ":")
    assertEqual(len(words), 3)
    // ...
}
```
Good example
```go
func TestSplit(t *testing.T) {
    s, sep := "a:b:c", ":"
    words := strings.Split(s, sep)
    if got, want := len(words), 3; got != want {
        t.Errorf("Split(%q, %q) returned %d words, want %d",
        s, sep, got, want)
    }
    // ...
}
```

### Avoiding Brittle Tests

An application that often fails when it encounters new but valid inputs is called `buggy`; a test that spuriously fails when a sound change was made to the program is called brittle.

The easiest way to avoid brittle tests is to 
1. Check only the properties you care about. 
2. Test your program’s simpler and more stable interfaces in preference to its internal functions. 
3. Be selective in your assertions. 
4. Don’t check for exact string matches, for example, but look for relevant substrings that will remain unchanged as the program evolves. It’s often worth writing a substantial function to distill a complex out put down to its essence so that assertions will be reliable.


## Coverage

By its nature, testing is never complete. As the influential comp uter scientist Edsger Dijkstra
put it, `Testing shows the presence, not the absence of bugs.` No quantity of tests can ever
prov e a package free of bugs. At best, they increase our confidence that the package works
well in a wide range of important scenarios.

```shell
$ go test -run=Coverage -coverprofile=c.out gopl.io/ch7/eval
ok gopl.io/ch7/eval 0.032s coverage: 68.5% of statements
```

```shell
$ go tool cover -html=c.out
```

<img src="/Users/chengying_chang/Trender/projects/golang-study-group/Notes/res/coverage html report.png" width="600"/>

Achieving 100% statement coverage sounds like a noble goal, but it is not usually feasible in practice, nor is it likely to be a good use of effort.

## Benchmark Functions

Benchmarking is the practice of measuring the performance of a program on a fixed workload.

Useful to answer the below questions
- If a function takes 1ms to process 1,000 elements, how long will it take to process 10,000 or a million?
- What is the best size for an I/O buffer?
- Which algorithm performs best for a given job?

```go
import "testing"

func BenchmarkIsPalindrome(b *testing.B) {
    for i := 0; i < b.N; i++ {
        IsPalindrome("A man, a plan, a canal: Panama")
    }
}
```

```shell
$ cd $GOPATH/src/gopl.io/ch11/word2
$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 1000000 1035 ns/op
ok gopl.io/ch11/word2 2.179s
```

The benchmark name’s numeric suffix, 8 here , indicates the value of GOMAXPROCS, which is important for con cur rent benchmarks.

Option 1
```go
n := len(letters)/2
for i := 0; i < n; i++ {
    if letters[i] != letters[len(letters)-1-i] {
        return false
    }
}
return true
```

```shell
$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 1000000 992 ns/op
ok gopl.io/ch11/word2 2.093s
```

Option 2 Better

```go
letters := make([]rune, 0, len(s))
for _, r := range s {
    if unicode.IsLetter(r) {
        letters = append(letters, unicode.ToLower(r))
    }
}
```

```shell
$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 2000000 697 ns/op
ok gopl.io/ch11/word2 1.468s
```

Comparative benchmarks are just regular code. The y typically take the for m of a single
parameterized function, cal le d from several Benchmark functions with different values

```go
func benchmark(b *testing.B, size int) { /* ... */ }
func Benchmark10(b *testing.B) { benchmark(b, 10) }
func Benchmark100(b *testing.B) { benchmark(b, 100) }
func Benchmark1000(b *testing.B) { benchmark(b, 1000) }
```


## Profiling

Benchmarks are useful for measuring the performance of specific operat ions, but when we’re
trying to make a slow program faster, we often have no ide a where to beg in.

When we wish to look carefully at the speed of our programs, the best technique for identifying
the critical code is `profiling`. Profiling is an automated approach to performance measurement
bas ed on sampling a number of profile events during exe cut ion, then extrapolating from
them dur ing a post-processing step; the resulting statistical summary is called a profile.

- A CPU profile identifies the functions whose exe cut ion requires the most CPU time.
- A heap profile identifies the statements responsible for allocating the most memory.
- A blocking profile identifies the operations responsible for blocking goroutines the longest,
  such as system calls, channel sends and receives, and acquisitions of locks.

```shell
$ go test -cpuprofile=cpu.out
$ go test -blockprofile=block.out
$ go test -memprofile=mem.out
```

<img src="/Users/chengying_chang/Trender/projects/golang-study-group/Notes/res/profiling.png" width="600"/>

## Example Functions

```go
func ExampleIsPalindrome() {
    fmt.Println(IsPalindrome("A man, a plan, a canal: Panama"))
    fmt.Println(IsPalindrome("palindrome"))
    // Output:
    // true
    // false
}
```

Example functions serve three purposes.
- Documentation
- Executable tests
- hands-on experimentation
