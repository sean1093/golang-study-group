# Chapter 4 組合型別

## 4.1 陣列

- 固定長度
- 所有元素型別相同

### 宣告陣列

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
