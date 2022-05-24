# syncmap

syncmap.Map is sync.Map using generics.

syncmap.Map has methods that are same as sync.Map. e.g. `Store`, `Load`, `LoadOrStore`, `LoadAndDelete`, `Delete`, `Range`

## Usage

### Import
```go
import "github.com/shiolier/syncmap"
```

### Init
```go
// empty map
smap := &syncmap.Map[string, int]{}
// empty map
smap := syncmap.NewMap[string, int](nil)
// OR map[K]V => *syncmap.Map[K, V]
m := map[string]int{"foo": 100, "bar": 200}
smap := syncmap.NewMap(m)
```

### Store (thread safe)
```go
smap.Store("baz", 300)
```

### Load (thread safe)
```go
foo, ok := smap.Load("foo")
if !ok {
	fmt.Println("foo did not exist")
}
```

### LoadOrStore (thread safe)
```go
qux, loaded := smap.LoadOrStore("qux", 400)
if loaded {
	fmt.Println("qux exists")
} else {
	fmt.Println("store the value because qux did not exist")
}
```

### LoadAndDelete (thread safe)
```go
qux, loaded := smap.LoadAndDelete("qux")
if loaded {
	fmt.Println("qux exists")
} else {
	fmt.Println("qux did not exist")
}
```

### Delete (thread safe)
```go
smap.Delete("baz")
```

### Range (thread safe)
```go
smap.Range(func(key string, value int) bool {
	fmt.Printf("%s: %d\n", key, value)
	return true
})
```

### Supports json.Marshal and json.Unmarshal
```go
// json.Marshal
js, err := json.Marshal(smap)
if err != nil {
	panic(err)
}
// Print json string
fmt.Println(string(js))

// empty map
smap = syncmap.NewMap[string, int](nil)
// json.Unmarshal
if err := json.Unmarshal(js, smap); err != nil {
	panic(err)
}
```

#### When using your own type
```go
// Key
type MyKey struct {
	Num int
	Str string
}

// MarshalJSKey returns the json key string
func (k MyKey) MarshalJSKey() (string, error) {
	return fmt.Sprintf("%d:%s", k.Num, k.Str), nil
}

// UnmarshalJSKey parses the json key string
func (k *MyKey) UnmarshalJSKey(keystr string) error {
	ss := strings.Split(keystr, ":")
	num, err := strconv.Atoi(ss[0])
	if err != nil {
		return err
	}
	k.Num = num
	k.Str = strings.Join(ss[1:], ":")
	return nil
}

// Value
type MyValue struct {
	// can use json tag
	Foo int `json:"foo"`
	Bar string
}
```

```go
// Init
smap := syncmap.NewMap[MyKey, MyValue](nil)

// Store
smap.Store(MyKey{
	Num: 111,
	Str: "ABC",
}, MyValue{
	Foo: 222,
	Bar: "DEF",
})
smap.Store(MyKey{
	Num: 333,
	Str: "GHI:JKL",
}, MyValue{
	Foo: 444,
	Bar: "MNO",
})

// Marshal
js, err := json.Marshal(smap)
if err != nil {
	fmt.Fprintln(os.Stderr, err)
	return
}
// Print json string
fmt.Println(string(js))
// {"111:ABC":{"foo":222,"Bar":"DEF"},"333:GHI:JKL":{"foo":444,"Bar":"MNO"}}

// empty map
smap = syncmap.Map[MyKey, MyValue]{}
// Unmarshal
if err := json.Unmarshal(js, smap); err != nil {
	fmt.Fprintln(os.Stderr, err)
	return
}

// Range
smap.Range(func(key MyKey, value MyValue) bool {
	fmt.Printf("%v: %v\n", key, value)
	return true
})
/*
	{111 ABC}: {222 DEF}
	{333 GHI:JKL}: {444 MNO}
*/
```