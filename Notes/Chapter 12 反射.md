# Chapter 12 Reflection

## Agenda
- 12.1 為何需要Reflection
- 12.2 reflect.Type and reflect.Value
- 12.3 輸出值的Display function
- 12.5 reflect.Value 設置變數
- 12.4 Example: SExpr encode
- 12.6 Example: SExpr decode
- 12.7 存取struct欄位標籤
- 12.8 顯示型別方法
- 12.9 注意事項


## 12.1 為何需要Reflection

撰寫一個方法統一處理並未有共通介面、已知表示、或是在設計函式時尚不存在的值的時候需要使用。
像是撰寫一個Sprintf的function根據輸入的值x回傳該值轉換成string後回傳

```
    type stringer interface {String() string}

    type ttype struct {}

    func (t ttype) String() string {
        return "Hello ttype string"
    }

    func main() {
        a := ttype{}
        fmt.Print(Sprintf(a))
    }

    func Sprintf(x interface{}) string {
        switch x := x.(type) {
        case stringer:
            return x.String()
        case string:
            return x
        case int:
            return strconv.Itoa(x)
        case bool:
            if x {
                return "true"
            }
            return "false"
        default:
            // array, func, chan, func, map, pointer, slice, struct
            return "???"
        }
    }
```
我們缺少了很多型別雖然我們可以增加case來處理但是永遠會處理不完。
像是url.Values的具名型別，底層的型別為map[string][]string但是在選擇case的時候兩個type型別卻不會相同。所以我們需要reflection，他給我們提供了在runtime時檢查type的能力，同時也提供給你在runtime時修改、建立變數、函數和struct的能力

## 12.2 reflect.Type and reflect.Value

* reflect 套件提供兩個重要的型別: Type and Value。對應到了7.5節中型別描述的值。 e.g.,
```
          ___________________                    os.File
    Type |      *os.File     |                  ____________________
         |___________________|                 |                    |
    Value|        *----------|---------------> | fd int = 1 (stdout)|
         |___________________|                 |____________________|

```
 Note: Type為一個interface裡面含有許多方法，而Value為一個struct用來保存型別的值因此我們可以reflect.TypeOf(x) 與 reflect.ValueOf(x) 來取得變數x在runtime時的型別與值。Value 的值也可以通過interface()的function來還原成原來的type。
```
func main() {
	var x interface{} = 3
	t := reflect.TypeOf(x)      // reflect.Type
	fmt.Println(t.String())     // "int"
	fmt.Println(t)              // "int"
	fmt.Printf("%T\n", x)       // "int"
	v := reflect.ValueOf(x)     // reflect.Value
	fmt.Println(v)              // "3"
	fmt.Printf("%v\n", x)       // "3"
	fmt.Println(v.String())     // "<int Value>"
	r := v.Interface()          // interface{}
}
```
Type 與 Value 中都有實作做一個Kind() function。他會回傳一個Kind type 他表達了組成該值的primitive type。
```
func main() {
	v := make(map[string][]string)      // map
	c := url.Values(v)                  // url.Values
	t := reflect.TypeOf(c)              // reflect.Type
	fmt.Println(t)                      // "url.Values"
	fmt.Println(t.Kind())               // "map"
}
```
我們可以修改一下原先的Sprintf函數

```
func main() {
	var x int64 = 1
	var d time.Duration = 1 * time.Nanosecond
	fmt.Println(Sprintf(x))                     // "1"
	fmt.Println(Sprintf(d))                     // "1"
	fmt.Println(Sprintf([]int64{x}))            // "[]int64 0xc0000120a8"
	fmt.Println(Sprintf([]time.Duration{d}))    // "[]time.Duration 0xc0000120d0"
}

func Sprintf(x interface{}) string {
	switch reflect.TypeOf(x).Kind() {
	case reflect.Invalid:
		return "invalid"
	case reflect.Int, reflect.Int8, reflect.Int16,
		reflect.Int32, reflect.Int64:
		return strconv.FormatInt(reflect.ValueOf(x).Int(), 10)
	case reflect.Uint, reflect.Uint8, reflect.Uint16,
		reflect.Uint32, reflect.Uint64, reflect.Uintptr:
		return strconv.FormatUint(reflect.ValueOf(x).Uint(), 10)
	// ...floating-point and complex cases omitted for brevity...
	case reflect.Bool:
		return strconv.FormatBool(reflect.ValueOf(x).Bool())
	case reflect.String:
		return strconv.Quote(reflect.ValueOf(x).String())
	case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
		return reflect.ValueOf(x).Type().String() + " 0x" +
			strconv.FormatUint(uint64(reflect.ValueOf(x).Pointer()), 16)
	default: // reflect.Array, reflect.Struct, reflect.Interface
		return reflect.ValueOf(x).Type().String() + " value"
	}
}
```

## 12.3 輸出值的Display function
* reflect.Value.Index(i) 回傳第i個元素的 reflect.Value值
* reflect.Value.Field(i) 回傳struct第i個field的 reflect.Value值
* reflect.Value.MapIndex(key) 回傳key對應元素的 reflect.Value值
* reflect.Value.Elem() 回傳point的 reflect.Value值

```
func Display(name string, x interface{}) {
	fmt.Printf("Display %s (%T): \n", name, x)
	display(name, reflect.ValueOf(x))
}

func display(path string, v reflect.Value) {
	switch v.Kind() {
	case reflect.Invalid:
		fmt.Printf("%s = invalid\n", path)
	case reflect.Slice, reflect.Array:
		for i := 0; i < v.Len(); i++ {
			display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
		}
	case reflect.Struct:
		for i := 0; i < v.NumField(); i++ {
			fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
			display(fieldPath, v.Field(i))
		}
	case reflect.Map:
		for _, key := range v.MapKeys() {
			display(fmt.Sprintf("%s[%s]", path,
				Sprintf(key.Interface())), v.MapIndex(key))
		}
	case reflect.Ptr:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			display(fmt.Sprintf("(*%s)", path), v.Elem())
		}
	case reflect.Interface:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
			display(path+".value", v.Elem())
		}
	default: // basic types, channels, funcs
		fmt.Printf("%s = %s\n", path, Sprintf(v.Interface()))
	}
}
```


## 12.5 reflect.Value 設置變數

* 不可定址的reflect.Value不可以設定值 (但可定址不一定能設定)
* 通過CanSet() 與 CanAddr() function 來確定reflect.Value 可不可以被更改與被定址
* reflect.ValueOf(&x).Elem() 來取的原先變數的值
* 通過SetBool(), SetInt(), SetString() 來設定值

```
func main() {
	stdout := reflect.ValueOf(os.Stdout).Elem()
	fmt.Println(stdout.Type())                  // os.File 
	fd := stdout.FieldByName("fd")
	fmt.Println(fd.CanAddr(), fd.CanSet())      // True, False

	x := 2
	a := reflect.ValueOf(x)         // 整數x 的reflect.Value (2)
	b := reflect.ValueOf(&x)        // 整數x記憶體位址的reflect.Value (address of x)
	c := b.Elem()                   // 取的指向 x 的描述器
	fmt.Printf("%p\n", &x)          // 0xc000012088
	fmt.Printf("%p\n", &a)          // 0xc000004078
	fmt.Printf("%p\n", &b)          // 0xc000004090
	fmt.Printf("%p\n", &c)          // 0xc0000040a8
	fmt.Printf("0x%x\n", c.Addr())  // 0xc000012088
	x = 3
	fmt.Printf("%d\n", a)           // 2
	fmt.Printf("%d\n", c)           // 3

	c.SetInt(4)
	fmt.Printf("%d\n", x)           // 4

	a.SetInt(6)                     // panic
	d := a.Elem()                   // panic
	fmt.Printf("0x%x\n", d.Addr())  // panic
}
               x                  a
          ___________        ___________ 
    Type |    int    |      |    int    | 
         |___________|      |___________| 
    Value|     x     |      |     x     |
         |___________|      |___________|


                 c                                                               d (x)
          ___________________            *int                            ___________________
    Type |      *int         |          ____________________            |        int        |
         |___________________|         |                    |   Elem()  |___________________|
    Value|        *----------|-------> | address of x       | --------> |         x         |
         |___________________|         |____________________|           |___________________|



```

## 12.4 Example: SExpr encode
SExpr 結構:
* 整數: 42
* 字串: "hello"
* 符號: foo
* 清單: ( 1 2 3)
```
// Marshal encodes a Go value in S-expression form.
func Marshal(v interface{}) ([]byte, error) {
	var buf bytes.Buffer
	if err := encode(&buf, reflect.ValueOf(v)); err != nil {
		return nil, err
	}
	return buf.Bytes(), nil
}

// encode writes to buf an S-expression representation of v.
func encode(buf *bytes.Buffer, v reflect.Value) error {
	switch v.Kind() {
	case reflect.Invalid:
		buf.WriteString("nil")

	case reflect.Int, reflect.Int8, reflect.Int16,
		reflect.Int32, reflect.Int64:
		fmt.Fprintf(buf, "%d", v.Int())

	case reflect.Uint, reflect.Uint8, reflect.Uint16,
		reflect.Uint32, reflect.Uint64, reflect.Uintptr:
		fmt.Fprintf(buf, "%d", v.Uint())

	case reflect.String:
		fmt.Fprintf(buf, "%q", v.String())

	case reflect.Ptr:
		return encode(buf, v.Elem())

	case reflect.Array, reflect.Slice: // (value ...)
		buf.WriteByte('(')
		for i := 0; i < v.Len(); i++ {
			if i > 0 {
				buf.WriteByte(' ')
			}
			if err := encode(buf, v.Index(i)); err != nil {
				return err
			}
		}
		buf.WriteByte(')')

	case reflect.Struct: // ((name value) ...)
		buf.WriteByte('(')
		for i := 0; i < v.NumField(); i++ {
			if i > 0 {
				buf.WriteByte(' ')
			}
			fmt.Fprintf(buf, "(%s ", v.Type().Field(i).Name)
			if err := encode(buf, v.Field(i)); err != nil {
				return err
			}
			buf.WriteByte(')')
		}
		buf.WriteByte(')')

	case reflect.Map: // ((key value) ...)
		buf.WriteByte('(')
		for i, key := range v.MapKeys() {
			if i > 0 {
				buf.WriteByte(' ')
			}
			buf.WriteByte('(')
			if err := encode(buf, key); err != nil {
				return err
			}
			buf.WriteByte(' ')
			if err := encode(buf, v.MapIndex(key)); err != nil {
				return err
			}
			buf.WriteByte(')')
		}
		buf.WriteByte(')')

	default: // float, complex, bool, chan, func, interface
		return fmt.Errorf("unsupported type: %s", v.Type())
	}
	return nil
}

```
## 12.6 Example: SExpr decode

```
type lexer struct {
	scan  scanner.Scanner
	token rune // the current token
}

func (lex *lexer) next()        { lex.token = lex.scan.Scan() }
func (lex *lexer) text() string { return lex.scan.TokenText() }

func (lex *lexer) consume(want rune) {
	if lex.token != want {
		panic(fmt.Sprintf("got %q, want %q", lex.text(), want))
	}
	lex.next()
}

func Unmarshal(data []byte, out interface{}) (err error) {
	lex := &lexer{scan: scanner.Scanner{Mode: scanner.GoTokens}}
	lex.scan.Init(bytes.NewReader(data))
	lex.next() // get the first token
	defer func() {
		// NOTE: this is not an example of ideal error handling.
		// 如果發生 panic 回復並回傳error
		if x := recover(); x != nil {
			err = fmt.Errorf("error at %s: %v", lex.scan.Position, x)
		}
	}()
	read(lex, reflect.ValueOf(out).Elem())
	return nil
}

func read(lex *lexer, v reflect.Value) {
	switch lex.token {
	case scanner.Ident:
		// The only valid identifiers are
		// "nil" and struct field names.
		if lex.text() == "nil" {
			v.Set(reflect.Zero(v.Type())) // 設定 v type的0值 // e.g., int-> 0, bool -> false
			lex.next()
			return
		}
	case scanner.String:
		s, _ := strconv.Unquote(lex.text()) // NOTE: ignoring errors
		v.SetString(s)
		lex.next()
		return
	case scanner.Int:
		i, _ := strconv.Atoi(lex.text()) // NOTE: ignoring errors
		v.SetInt(int64(i))
		lex.next()
		return
	case '(':
		lex.next()
		readList(lex, v)
		lex.next() // consume ')'
		return
	}
	panic(fmt.Sprintf("unexpected token %q", lex.text()))
}

func readList(lex *lexer, v reflect.Value) {
	switch v.Kind() {
	case reflect.Array: // (item ...)
		for i := 0; !endList(lex); i++ {
			read(lex, v.Index(i))
		}

	case reflect.Slice: // (item ...)
		for !endList(lex) {
			item := reflect.New(v.Type().Elem()).Elem()
			read(lex, item)
			v.Set(reflect.Append(v, item))
		}

	case reflect.Struct: // ((name value) ...)
		for !endList(lex) {
			lex.consume('(')
			if lex.token != scanner.Ident {
				panic(fmt.Sprintf("got token %q, want field name", lex.text()))
			}
			name := lex.text()
			lex.next()
			read(lex, v.FieldByName(name))
			lex.consume(')')
		}

	case reflect.Map: // ((key value) ...)
		v.Set(reflect.MakeMap(v.Type()))
		for !endList(lex) {
			lex.consume('(')
			key := reflect.New(v.Type().Key()).Elem()
			read(lex, key)
			value := reflect.New(v.Type().Elem()).Elem()
			read(lex, value)
			v.SetMapIndex(key, value)
			lex.consume(')')
		}

	default:
		panic(fmt.Sprintf("cannot decode list into %v", v.Type()))
	}
}

func endList(lex *lexer) bool {
	switch lex.token {
	case scanner.EOF:
		panic("end of file")
	case ')':
		return true
	}
	return false
}
```

## 12.7 存取struct欄位標籤

```
type datas struct {
	Labels    []string `http:"l"`
	MaxResult int      `http:"max"`
	Exact     bool     `http:"x"`
}

func main() {
	a := datas{}
	v := reflect.ValueOf(a)
	for i := 0; i < v.NumField(); i++ {
		f := v.Type().Field(i)
		fmt.Println(f.Name)				// Labels
		fmt.Println(f.Type)				// []string
		fmt.Println(f.Tag)				// http:"l"
		fmt.Println(f.Tag.Get("http"))  // l
	}
}
```
## 12.8 顯示型別方法

```
ex: func (d Duration) Round(m Duration) Duration {}

func main() {
	a := time.Hour
	v := reflect.ValueOf(a)
	fmt.Println(v.Type())
	fmt.Println(v.NumMethod())
	for i := 0; i < v.NumMethod(); i++ {
		m := v.Method(i)
		fmt.Println("function name: ", v.Type().Method(i).Name)	// function name: Round
		fmt.Println("function type: ", m.Type())		// function type: func(time.Duration) time.Duration
	}
}

```
## 12.9 注意事項

小心使用反射的三個原因:

* 大多數的panic都為runtime時才會發現錯誤，在編譯時期不易發現問題
* 會降低自動化重構工具與分析工具的安全性與準確性，因為無法判別或依靠型別的資訊
* 反射為基礎的函式會比特定型別的程式慢。