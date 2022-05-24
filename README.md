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

## Benchmark

Compare with WrapperMap[K, V] that is sync.Map wrapped with generics

Use [int, int] for type parameters [K, V]

    goos: darwin
    goarch: amd64
    pkg: github.com/shiolier/syncmap
    cpu: Intel(R) Core(TM) i5-3210M CPU @ 2.50GHz
    BenchmarkLoadMostlyHits/*WrapperMap           51769129       23.38 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadMostlyHits/*Map                  49294209       23.71 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadMostlyMisses/*WrapperMap         86073642       13.97 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadMostlyMisses/*Map                54911031       22.43 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadOrStoreBalanced/*WrapperMap       1775521       673.9 ns/op       93 B/op        2 allocs/op
    BenchmarkLoadOrStoreBalanced/*Map              3079636       326.9 ns/op       36 B/op        1 allocs/op
    BenchmarkLoadOrStoreUnique/*WrapperMap         1000000        1168 ns/op      163 B/op        4 allocs/op
    BenchmarkLoadOrStoreUnique/*Map                1765795       645.7 ns/op      113 B/op        2 allocs/op
    BenchmarkLoadOrStoreCollision/*WrapperMap     55630494       20.86 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadOrStoreCollision/*Map            48463934       24.44 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteBalanced/*WrapperMap    56891140       20.51 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteBalanced/*Map           48169279       25.21 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteUnique/*WrapperMap     124337067       9.726 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteUnique/*Map             60827192       19.44 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteCollision/*WrapperMap  121662361       9.720 ns/op        0 B/op        0 allocs/op
    BenchmarkLoadAndDeleteCollision/*Map          59067524       19.58 ns/op        0 B/op        0 allocs/op
    BenchmarkRange/*WrapperMap                      114358       10445 ns/op        0 B/op        0 allocs/op
    BenchmarkRange/*Map                              50613       23094 ns/op        0 B/op        0 allocs/op
    BenchmarkAdversarialAlloc/*WrapperMap          3588988       345.7 ns/op       39 B/op        0 allocs/op
    BenchmarkAdversarialAlloc/*Map                 4619600       268.7 ns/op       26 B/op        0 allocs/op
    BenchmarkAdversarialDelete/*WrapperMap         9616904       138.1 ns/op       17 B/op        0 allocs/op
    BenchmarkAdversarialDelete/*Map               12149572       101.3 ns/op       10 B/op        0 allocs/op
    BenchmarkDeleteCollision/*WrapperMap         100000000       11.41 ns/op        0 B/op        0 allocs/op
    BenchmarkDeleteCollision/*Map                 57604394       21.07 ns/op        0 B/op        0 allocs/op

