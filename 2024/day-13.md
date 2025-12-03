**Author:** Vũ

---

# iter.json: A Powerful and Efficient Way to Iterate and Manipulate JSON in Go

Have you ever needed to modify unstructured JSON data in Go? Maybe you’ve had to delete password and all blacklisted fields, rename keys from `camelCase` to `snake_case`, or convert all number ids to strings because JavaScript does not like `int64`? If your solution has been to unmarshal everything into a `map[string]any` using `encoding/json` and then marshal it back… well, let’s face it, that’s far from efficient! What if you could loop through the JSON data, grab the path of each item, and decide exactly what to do with it on the fly?

![favicon](https://olivernguyen.io/images/favicon.png)

https://olivernguyen.io/w/iter.json/

![og image](https://iolivernguyen.github.io/images/ogx.png)

Have you ever needed to modify unstructured JSON data in Go? Maybe you’ve had to delete password and all blacklisted fields, rename keys from `camelCase` to `snake_case`, or convert all number ids to strings because JavaScript does not like `int64`? If your solution has been to unmarshal everything into a `map[string]any` using `encoding/json` and then marshal it back… well, let’s face it, that’s far from efficient!

What if you could loop through the JSON data, grab the path of each item, and decide exactly what to do with it on the fly?

Yes! I have a good news! With the new iterator feature in Go 1.23, there’s a better way to iterate and manipulate JSON. Meet [ezpkg.io/iter.json](https://ezpkg.io/iter.json) — your powerful and efficient companion for working with JSON in Go.

---

### Hello World!

[ezpkg.io/iter.json](https://ezpkg.io/iter.json) is a Go package that provides a simple and efficient way to iterate over JSON data. It allows you to traverse JSON objects, arrays, and values, and perform various operations on them without fully parsing the data.

At the core, it provides a [`Parse()`](https://pkg.go.dev/ezpkg.io/iter.json#Parse) function and a [`Builder`](https://pkg.go.dev/ezpkg.io/iter.json#Builder) type. The `Parse()` function returns [an iterator](https://pkg.go.dev/iter#hdr-Iterators) that yields each item in the JSON data, while the `Builder` type allows you to build new JSON data dynamically.

Let’s look at some examples of how to use [iter.json](https://ezpkg.io/iter.json) to iterate, build, format, filter, and edit JSON data in Go.

---

#### 1. Iterating JSON

Given that we have an [`alice.json`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/alice.json) file:

```json
// alice.json (example placeholder)
{
  "name": "Alice",
  "age": 30,
  "emails": ["alice@example.com"]
}
```

First, let’s use `for range` [`Parse()`](https://pkg.go.dev/ezpkg.io/iter.json#Parse) to iterate over the JSON file, then print the path, key, token, and level of each item. See [`examples/01.iter`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/01.iter/main.go).

```go
for item := range iterjson.Parse(r) {
    fmt.Println(item.GetPathString(), item.Key, item.Token, item.Level)
}
```

The code will output:

```text
// example output placeholder
$.name name "Alice" 1
$.age age 30 1
$.emails emails [...] 1
$.emails[0] 0 "alice@example.com" 2
```

---

#### 2. Building JSON

Use `Builder` to build a JSON data. It accepts optional arguments for indentation. See [`examples/02.builder`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/02.builder/main.go).

Create a new `Builder` with:

```go
b := iterjson.NewBuilder(prefix, indent)
```

- `Builder.AddRaw(key RawToken, token RawToken)` adds a raw token to the JSON data.
- `Builder.Add(key any, token any)` adds a key-value pair to the JSON data.
- `Builder.Bytes()` returns the JSON data as a byte slice.

It accepts various types, including `string`, `int`, `struct`, `[]byte`, etc.

```go
b.Add("name", "Alice")
b.Add("age", 30)
out := b.Bytes()
fmt.Println(string(out))
```

Which will output the JSON with indentation:

```json
{
  "name": "Alice",
  "age": 30
}
```

---

#### 3. Formatting JSON

You can reconstruct or format a JSON data by sending its key and values to a `Builder`. See [`examples/03.reformat`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/03.reformat/main.go).

The first example minifies the JSON while the second example formats it with prefix `👉` on each line.

```go
b := iterjson.NewBuilder("", "  ")
for item := range iterjson.Parse(r) {
    b.Add(item.Key, item.Value())
}
fmt.Println(string(b.Bytes()))
```

---

#### 4. Adding line numbers

In this example, we add line numbers to the JSON output, by adding a `b.WriteNewline()` before the `fmt.Fprintf()` call. See [`examples/04.line_number`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/04.line_number/main.go).

```go
line := 1
for item := range iterjson.Parse(r) {
    fmt.Fprintf(w, "%4d: ", line)
    line++
    b.WriteNewline()
}
```

This will output:

```text
   1: {
   2:   "name": "Alice",
   3:   "age": 30
   4: }
```

---

#### 5. Adding comments

By putting a `fmt.Fprintf(comment)` between `b.WriteComma()` and `b.WriteNewline()`, you can add a comment to the end of each line. See [`examples/05.comment`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/05.comment/main.go).

```go
b.WriteComma()
fmt.Fprintf(w, " // %s", item.GetPathString())
b.WriteNewline()
```

This will output:

```jsonc
{
  "name": "Alice", // $.name
  "age": 30 // $.age
}
```

---

#### 6. Filtering JSON and extracting values

There are `item.GetPathString()` and `item.GetRawPath()` to get the path of the current item. You can use them to filter the JSON data. See [`examples/06.filter_print`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/06.filter_print/main.go).

Example with `item.GetPathString()` and `regexp`:

```go
re := regexp.MustCompile(`\.password$`)
for item := range iterjson.Parse(r) {
    if re.MatchString(item.GetPathString()) {
        fmt.Println("Found password:", item.Token)
    }
}
```

Example with `item.GetRawPath()` and `path.Match()`:

```go
for item := range iterjson.Parse(r) {
    if ok, _ := path.Match("**/password", item.GetRawPath()); ok {
        fmt.Println("Found password:", item.Token)
    }
}
```

Both examples will output:

```text
Found password: "secret"
```

---

#### 7. Filtering JSON and returning a new JSON

By combining the `Builder` with the option `SetSkipEmptyStructures(false)` and the filtering logic, you can filter the JSON data and return a new JSON. See [`examples/07.filter_json`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/07.filter_json/main.go)

```go
b := iterjson.NewBuilder("", "  ")
b.SetSkipEmptyStructures(false)

for item := range iterjson.Parse(r) {
    if shouldKeep(item) {
        b.Add(item.Key, item.Value())
    }
}

fmt.Println(string(b.Bytes()))
```

This example will return a new JSON with only the filtered fields:

```json
{
  "name": "Alice"
}
```

---

#### 8. Editing values

This is an example for editing values in a JSON data. Assume that we are using number ids for our API. The ids are too big and JavaScript can’t handle them. We need to convert them to strings. See [`examples/08.number_id`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/08.number_id/main.go) and [`order.json`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/examples/order.json).

Iterate over the JSON data, find all `_id` fields and convert the number ids to strings:

```go
for item := range iterjson.Parse(r) {
    if strings.HasSuffix(item.KeyString(), "_id") && item.IsNumber() {
        b.Add(item.Key, strconv.FormatInt(item.Int64(), 10))
    } else {
        b.Add(item.Key, item.Value())
    }
}
```

This will add quotes to the number ids:

```json
{
  "order_id": "1234567890123456789"
}
```

---

### How it parses the JSON data

Thanks to powerful of iterators in Go 1.23, [ezpkg.io/iter.json](https://ezpkg.io/iter.json) is able to process JSON data with minimal lines of code and low overhead.

The core parser logic is contained in 2 files: [`scanner.go`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/scanner.go) and [`parser.go`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/parser.go). Here’s a brief overview of how it works:

- [`NextToken()`](https://pkg.go.dev/ezpkg.io/iter.json#NextToken) pulls the next [`RawToken`](https://pkg.go.dev/ezpkg.io/iter.json#RawToken) from the input.
- [`Parse()`](https://pkg.go.dev/ezpkg.io/iter.json#Parse) is a state machine with a stack. It pulls the next token from the input then processes it based on the current state.
- [`RawToken`](https://pkg.go.dev/ezpkg.io/iter.json#RawToken) is a tagged union with a [`TokenType`](https://pkg.go.dev/ezpkg.io/iter.json#TokenType) and optional raw `[]byte`.

---

#### `NextToken()` pulls the next token from the input

Here’s the core logic of the [`NextToken()`](https://pkg.go.dev/ezpkg.io/iter.json#NextToken) function ([`scanner.go`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/scanner.go)):

```go
func NextToken(r *bufio.Reader) (RawToken, error) {
    // simplified placeholder
    ch, _, err := r.ReadRune()
    if err != nil {
        return RawToken{Type: EOF}, err
    }
    // switch on ch and build RawToken
}
```

---

#### `Scan()` all tokens in a single loop

The [`Scan()`](https://pkg.go.dev/ezpkg.io/iter.json#Scan) function is essentially a single loop to pull the next token from the input each time.

```go
for tok, err := NextToken(r); err == nil; tok, err = NextToken(r) {
    // handle tok
}
```

---

#### `Parse()` is a state machine with a stack

Here’s the core logic of the parser ([`parse.go`](https://github.com/ezpkg/ezpkg/blob/main/iter.json/parser.go)).

It uses a stack to keep track of the current state (path, level) of the JSON data.

It pulls the next token from the input and processes it based on the current state:

- If it’s `[` or `{`, it pushes the current state to the stack.
- If it’s `]` or `}`, it pops the state from the stack.
- Otherwise, it parses “value” or “key: value” depending on the current state.

Here’s how it initializes the stack:

```go
stack := make([]PathItem, 0, 16)
stack = append(stack, PathItem{Key: "$", Index: -1})
```

With the implementation of [`PathItem`](https://pkg.go.dev/ezpkg.io/iter.json#PathItem):

```go
type PathItem struct {
    Key   string
    Index int
}
```

And the `push()`, `pop()`, `advance()` helper functions:

```go
func push(stack []PathItem, item PathItem) []PathItem {
    return append(stack, item)
}

func pop(stack []PathItem) []PathItem {
    return stack[:len(stack)-1]
}

func advance(item *PathItem) {
    item.Index++
}
```

The core state machine code is as follows. Honestly, using `goto` in this case is quite fun:

```go
Parse:
    tok, err := NextToken(r)
    if err != nil {
        break Parse
    }
    switch tok.Type {
    case LBracket, LBrace:
        stack = push(stack, PathItem{Key: "", Index: -1})
        goto Parse
    case RBracket, RBrace:
        stack = pop(stack)
        goto Parse
    default:
        // handle value or key: value
        goto Parse
    }
```

---

### How it builds the JSON data dynamically

`Builder` uses the stream of events from `Parse()` to write JSON incrementally, inserting commas, newlines, indentation, and optional comments or metadata as needed, without ever needing to materialize the full structure in memory.