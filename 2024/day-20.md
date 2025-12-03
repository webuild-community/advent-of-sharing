**Author:** Vũ

---

# ezpkg.io/conveyz: Understanding the Implementation of FConvey

The `FConvey` function is a standout feature of [ezpkg.io/conveyz](https://ezpkg.io/conveyz), designed to simplify focused testing in Go. Unlike the original [smartystreets/goconvey](https://github.com/smartystreets/goconvey) framework, which requires developers to set `FocusConvey` at every nested block level, `FConvey` is smarter. It ensures that focusing on a single block propagates to all relevant nested and parent blocks, streamlining the debugging process. In this article, we dive deep into the implementation of `FConvey`, why the original `FocusConvey` does require manual setting at every level, and how `FConvey` solves the problem.

![favicon](https://olivernguyen.io/images/favicon.png)

<https://olivernguyen.io/w/conveyz/>

![og-image](https://iolivernguyen.github.io/images/ogx.png)

The [`FConvey`](https://pkg.go.dev/ezpkg.io/conveyz#FConvey) function is a standout feature of [ezpkg.io/conveyz](https://ezpkg.io/conveyz), designed to simplify focused testing in Go. Unlike the original [smartystreets/goconvey](https://github.com/smartystreets/goconvey) framework, which requires developers to set `FocusConvey` at every nested block level, `FConvey` is smarter. It ensures that focusing on a single block propagates to all relevant nested and parent blocks, streamlining the debugging process.

In this article, we dive deep into the implementation of `FConvey`, why the original `FocusConvey` does require manual setting at every level, and how `FConvey` solves the problem.

### The motivation behind ezpkg.io/conveyz

Before we dive into the implementation details, let’s first understand the motivation behind the creation of [ezpkg.io/conveyz](https://ezpkg.io/conveyz).

#### Ginkgo and Gomega are great, but verbose

At work, we use [gomega](https://github.com/onsi/gomega) and [ginkgo](https://github.com/onsi/ginkgo) for writing behaviour tests. They are great, but sometimes too verbose. We have to write a lot of boilerplate code to set up the test environment, and it’s hard to focus on the test itself.

Each package has a `suite_test.go` file to set up the test environment, with:

```go
RegisterFailHandler(Fail)
RunSpecs()
```

Each file starts with a `Describe` block then `var` block, then `Context` blocks, then more `var` blocks, then more `Context` blocks, with `BeforeEach` and `AfterEach` blocks inside, then end up with a lot of `It` blocks at the leaf.

Lists of variables are declared at the beginning of the `Describe` and `Context` blocks, then initialized in `BeforeEach` blocks, before being used in `It` blocks, or `JustBeforeEach` blocks.

And even `JustBeforeEach` blocks to keep the execution logic separated.

So the execution is outer blocks to inner blocks:

```text
Describe
  → BeforeEach
    → Context
      → BeforeEach
        → It
```

Oh, no! It needs to jump to `JustBeforeEach` before `It`. Finally, `AfterEach` at the last.

And we have to repeat the same setup for each test file.

Well, you got it!

An example of `profile_suite_test.go`:

```go
// ... example profile_suite_test.go
```

and `user_test.go`:

```go
// ... example user_test.go
```

#### Convey: Focus on the test, not the setup

Compare the above code with the following from [smartystreets/goconvey](https://github.com/smartystreets/goconvey). It’s much simpler and easier to focus on the test itself.

- No need to set up the test environment in a separate file.
- You only need to use `Convey` and `So` to write the test.

```go
Convey("UserService", t, func() {
    Convey("SignUp", func() {
        Convey("valid username & password", func() {
            // ...
        })

        Convey("invalid username", func() {
            // ...
        })
    })
})
```

#### Problems: Incompatible assertions and repeated `FocusConvey`

While the code looks cleaner, there are still some problems:

- With ginkgo, you can focus on a single block by setting `FIt` or `FContext`. Basically, add a single `F` and it will only run that block (and children), skipping others.
- But in goconvey, you have to set `FocusConvey` at every level. It’s not smart enough to propagate the focus to all relevant blocks.

And the assertions are not compatible. You have to change:

- `Ω` to `So`
- `To` to `Should`
- `ToNot` to `ShouldNot`
- `ToNotBeEmpty` to `ShouldNotBeEmpty`
- `To(HaveOccurred())` to `ShouldNotBeNil`
- `To(ContainSubstring("..."))` to `ShouldContainSubstring("...")`

and so on.

We don’t want to maintain two sets of assertions, and it’s hard to convince the team to switch to goconvey or even doing the rewrite.

### Meet ezpkg.io/conveyz

So, I decided to create a new package, [ezpkg.io/conveyz](https://ezpkg.io/conveyz), to solve the above problems. It’s a wrapper around goconvey, with the following features:

- `Ω` to replace `So` and `To` in goconvey, so you can use the same assertions as gomega.
- `FConvey` to focus on a single block and propagate to all relevant blocks while skipping others.

The above code can be rewritten as:

```go
package user_test

import (
    . "ezpkg.io/conveyz"
)

func TestUserService(t *testing.T) {
    Convey("UserService", t, func() {
        Convey("SignUp", func() {
            Convey("valid username & password", func() {
                // ...
            })

            FConvey("invalid username", func() {
                // ...
            })
        })
    })
}
```

### Solution: Make “gomega” `Ω` work with “goconvey”

We can make `Ω` work with goconvey by [creating an adapter](https://github.com/ezpkg/ezpkg/blob/6f91299c405090f3a7289b6e8f591796a43d626e/conveyz/conveyz.go#L81-L84) that implements [gomega.Assertion](https://pkg.go.dev/github.com/onsi/gomega#Assertion):

```go
type assertionAdapter struct {
    actual interface{}
}

func (a assertionAdapter) Should(matcher types.GomegaMatcher, optional ...interface{}) string {
    return failureMessageOrEmpty(a.actual, matcher)
}
```

The adapter will call `convey.So()` with the actual value and the matcher.

It returns empty string if the matcher passes, or the error message if the matcher fails.

When failure, the output will look like this, with a nice error message and a focused stacktrace:

```text
Expected
    <int>: 2
to equal
    <int>: 1
at: user_test.go:42
```

And to make it easier for our engineers, we can [re-export](https://github.com/ezpkg/ezpkg/blob/6f91299c405090f3a7289b6e8f591796a43d626e/conveyz/gomega.go#L18) all the assertions from gomega:

```go
package conveyz

import "github.com/onsi/gomega"

var (
    Equal          = gomega.Equal
    BeNil          = gomega.BeNil
    // ... etc
)
```

So to migrate tests from goconvey to ezpkg.io/conveyz, you don’t need to change the assertions, just do some cleanup:

1. Replace the gomega/ginkgo imports with:

   ```go
   . "ezpkg.io/conveyz"
   ```

2. Replace all `Describe | Context | It` with `Convey`.
3. Remove `BeforeEach` and merge it with the `var` block to simplify the setup code.
4. Replace `AfterEach` with idiomatic Go `defer`.
5. For `JustBeforeEach`, you can simply inline the code right before the assertion. If it needs more logic, you can extract to a function and call it right before the assertion.

This way, we can enjoy the best of both worlds: the simplicity of goconvey with the power of gomega!

But there is still one problem: to focus on a single `Convey` block, you have to set `FocusConvey` at every level, from the outermost `Convey` to the innermost `Convey`. Then delete each one after finishing your tests… 😮‍💨

### Solution: Make `FConvey` smarter

To understand how `ezpkg.io/conveyz` `FConvey` works, let’s first look at how ginkgo’s `FIt` works and why goconvey’s `FocusConvey` requires manual setting at every level.

#### ginkgo: How a single `FIt` can focus on a test

In ginkgo, inside each `Describe | Context` block, you can set `FIt` or `FContext` to focus on that block.

For example, with the following code:

```go
Describe("UserService", func() {
    var (
        userService *UserService
        err         error
    )

    BeforeEach(func() {
        userService = NewUserService()
    })

    Context("SignUp", func() {
        Context("valid username & password", func() {
            BeforeEach(func() {
                // ...
            })

            It("should succeed", func() {
                // ...
            })
        })

        Context("invalid username", func() {
            BeforeEach(func() {
                // ...
            })

            It("should record failure", func() {
                // ...
            })

            FIt("should error (invalid username)", func() {
                // ...
            })
        })
    })
})
```

When you set `FIt("should error (invalid username)")`, it will only exec that test and skip others.

Specifically, it execs in this order (🟢 for exec and 🟡 for skip):

- 🟢 `Describe("UserService")`
  - 🟢 `BeforeEach`
  - 🟢 `Context("SignUp")`
    - 🟢 `BeforeEach`
    - 🟡 skip `Context("valid username & password")`
      - 🟡 skip `BeforeEach` inside
      - 🟡 skip all `It`s inside
    - 🟢 exec `Context("invalid username")`
      - 🟢 `BeforeEach`
      - 🟡 skip `It("should record failure")`
      - ✅ exec `FIt("should error (invalid username)")`
    - 🟡 skip all remaining non-focus `Context`s

It can be able to do that, because when run the `Context("valid username & password")`, it calls `BeforeEach`, but that `BeforeEach` does not actually run any logic inside – it only registers the logic to execute later.

Then there are 2 scenarios:

1. If ginkgo does not encounter any `FIt` (or `PIt` for pending), it will 🟢 execute all the registered logic.
2. If ginkgo encounters an `FIt`, it will only 🟢 execute that `It` (and all logic to that path), but 🟡 skip all other registered logic outside the execution path.

#### But goconvey `FocusConvey` must be set at every level

Because if you only set `FocusConvey` at the leaf level without setting it at the parent level, it will not skip the parent level.

When it runs a parent `Convey` block, it will run all the code inside, including the before-each logic (the `BeforeEach` blocks in previous ginkgo example).

Unlike ginkgo, which registers the before-each logic inside `BeforeEach` blocks without actually execute it, goconvey does not have that privilege: it has to run the `Convey` block to know if there is any `FocusConvey` inside!

So you get idea: no way to know if there is any `FocusConvey` inside without actually running the `Convey` block, including the before-each logic!

#### ezpkg.io/conveyz: Set a single `FConvey` to focus on a test

Okay, now we know why goconvey `FocusConvey` requires manual setting at every level. But how does `ezpkg.io/conveyz` [`FConvey`](https://pkg.go.dev/ezpkg.io/conveyz#FConvey) workaround that limitation? What’s the magic behind it?

Turns out, it’s quite simple: instead of running the `Convey` blocks, **we can first read the source code and find all the `FConvey` (or `SConvey`) inside!** When we know locations of all the `Convey` blocks, we can set the execution focus only on the path to the `FConvey` block, and skip all others. ✅

Now you know the trick, it’s time to see the implementation of `FConvey` in `ezpkg.io/conveyz`:

#### 1. Read the source code for the current package

The first time any `Convey | FConvey | SkipConvey` runs, we need to initialize everything: read the source code and find all `FConvey`.

To read the source code, we can use `runtime.Caller()` to get the current file path:

```go
_, file, _, ok := runtime.Caller(1)
if !ok {
    panic("cannot get caller info")
}
```

Then [read all the test files](https://github.com/ezpkg/ezpkg/blob/ce59fe2d591e0786d4587dbc1e4c50abcf1747f3/conveyz/focus.go#L53-L79) and store into a `fileMap`:

```go
func loadTestFiles(rootDir string) (map[string][]byte, error) {
    files := make(map[string][]byte)

    err := filepath.WalkDir(rootDir, func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }

        if d.IsDir() {
            return nil
        }

        if !strings.HasSuffix(path, "_test.go") {
            return nil
        }

        b, err := os.ReadFile(path)
        if err != nil {
            return err
        }

        files[path] = b
        return nil
    })

    return files, err
}
```