**Author:** Vũ

---

# Errors, Errors Everywhere: How We Centralized and Structured Error Handling

Handling errors in Go is simple and flexible – yet no structure! It’s supposed to be simple, right? Just return an `error`, wrapped with a message, and move on. Well, that simplicity quickly turns into chaotic as our codebase grows with more packages, more developers, and more “quick fixes” that stay there forever. Over time, the logs are full of “failed to do this” and “unexpected that”, and nobody knows if it’s the user’s fault, the server’s fault, buggy code, or it’s just a misalignment of the stars!

![favicon](https://olivernguyen.io/images/favicon.png)

<https://olivernguyen.io/w/namespace.error/>

![og-image](https://iolivernguyen.github.io/images/ogx.png)

---

Handling errors in Go is simple and flexible – yet no structure!

It’s supposed to be simple, right? Just return an `error`, wrapped with a message, and move on. Well, that simplicity quickly turns into chaotic as our codebase grows with more packages, more developers, and more “quick fixes” that stay there forever. Over time, the logs are full of “failed to do this” and “unexpected that”, and nobody knows if it’s the user’s fault, the server’s fault, buggy code, or it’s just a misalignment of the stars!

Errors are created with inconsistent messages. Each package has it own set of styles, constants, or custom error types. Error codes are added arbitrarily. No easy way to tell which errors may be returned from which function without digging into its implementation!

So, I took the challenge of creating a new error framework. We decided to go with a structured, centralized system using namespace codes to make errors meaningful, traceable, and – most importantly – give us peace of mind!

This is the story of how we started with a simple error handling approach, got thoroughly frustrated as the problems grew, and eventually built our own error framework. The design decisions, how it’s implemented, the lessons learned, and why it transformed our approach to managing errors. I hope that it will bring some ideas for you too!

---

### Go errors are just values

Go has a straightforward way to handle errors: errors are just values. An error is just a value that implements the `error` interface with a single method `Error() string`. Instead of throwing an exception and disrupting the current execution flow, Go functions return an `error` value alongside other results. The caller can then decide how to handle it: check its value to make decision, wrap with new messages and context, or simply return the error, leaving the handling logic for parent callers.

We can make any type an `error` by adding the `Error() string` method on it. This flexibility allows each package to define its own error-handling strategy, and choose whatever works best for them. This also integrates well with Go’s philosophy of composability, making it easy to wrap, extend, or customize errors as required.

---

### Every package needs to deal with errors

The common practice is to return an error value that implements the `error` interface and lets the caller decide what to do next. Here’s a typical example:

```go
// Loading Go code...
// (placeholder for original example)
```

Go provides a handful of utilities for working with errors:

- **Creating errors:** `errors.New()` and `fmt.Errorf()` for generating simple errors.
- **Wrapping errors:** Wrap errors with additional context using `fmt.Errorf()` and the `%w` verb.
- **Combining errors:** `errors.Join()` merges multiple errors into a single one.
- **Checking and handling errors:**  
  - `errors.Is()` matches an error with a specific value,  
  - `errors.As()` matches an error to a specific type, and  
  - `errors.Unwrap()` retrieves the underlying error.

In practice, we usually see these patterns:

- **Using standard packages:** Returning simple errors with `errors.New()` or `fmt.Errorf()`.
- **Exporting constants or variables:** For instance, [go-redis](https://github.com/redis/go-redis/blob/master/internal/pool/pool.go) and [gorm.io](https://github.com/go-gorm/gorm/blob/master/errors.go) define reusable error variables.
- **Custom error types:** Libraries like [lib/pq](https://github.com/lib/pq/blob/master/error.go) or [grpc/status.Error](https://github.com/grpc/grpc-go/blob/master/internal/status/status.go#L207) create specialized error types, often with associated codes for additional context.
- **Error interfaces with implementations:** The [aws-sdk-go](https://github.com/aws/aws-sdk-go/blob/main/aws/awserr/error.go) uses an interface-based approach to define error types with various implementations.
- **Or multiple interfaces:** Like [Docker’s errdefs](https://github.com/SUSE/docker/blob/master/errdefs/defs.go), which defines multiple interfaces to classify and manage errors.

---

### We started with a common approach

In the early days, like many Go developers, we followed Go’s common practices and kept error handling minimal yet functional. It worked well enough for a couple of years.

- Include stacktrace using [pkg/errors](https://github.com/pkg/errors), a popular package at that time.
- Export constants or variables for package-specific errors.
- Use `errors.Is()` to check for specific errors.
- Wrap errors with a new messages and context.
- For API errors, we define error types and codes with Protobuf enum.

#### Including stacktrace with `pkg/errors`

We used [pkg/errors](https://github.com/pkg/errors), a popular error-handling package at the time, to include stacktrace in our errors. This was particularly helpful for debugging, as it allowed us to trace the origin of errors across different parts of the application.

To create, wrap, and propagate errors with stacktrace, we implemented functions like `Newf()`, `NewValuef()`, and `Wrapf()`. Here’s an example of our early implementation:

```go
// Loading Go code...
// (placeholder for Newf, NewValuef, Wrapf implementation)
```

#### Exporting error variables

Each package in our codebase defined its own error variables, often with inconsistent styles.

```go
// Loading Go code...
// (placeholder for error variable examples)
```

```go
// Loading Go code...
// (placeholder for more error variable examples)
```

#### Checking errors with `errors.Is()` and wrapping with additional context

```go
// Loading Go code...
// (placeholder for errors.Is and wrapping example)
```

This helped propagate errors with more detail but often resulted in verbosity, duplication, and less clarity in logs:

```text
// Loading Plain Text code...
// (placeholder for sample log output)
```

#### Defining external errors with Protobuf

For external-facing APIs, we adopted a Protobuf-based error model inspired by [Meta’s Graph API](https://developers.facebook.com/docs/graph-api/guides/error-handling#receiving-errorcodes):

```proto
// Loading Protobuf code...
// (placeholder for protobuf error definition)
```

This approach helped structure errors, but over time, error types and codes were added without a clear plan, leading to inconsistencies and duplication.

---

### And problems grew over time

#### Errors were declared everywhere

Each package defined its own error constants with no centralized system.

Constants and messages were scattered across the codebase, making it unclear which errors a function might return – ugh, is it `gorm.ErrRecordNotFound` or `user.ErrNotFound` or both?

#### Random error wrapping led to inconsistent and arbitrary logs

Many functions wrapped errors with arbitrary, inconsistent messages without declaring their own error types.

Logs were verbose, redundant, and difficult to search or monitor.

Error messages were generic and often didn’t explain what went wrong or how it happened. Also brittle and prone to unnoticed changes.

```text
// Loading Plain Text code...
// (placeholder for noisy log examples)
```

#### No standardization led to improper error handling

Each package handled errors differently, making it hard to know if a function returned, wrapped, or transformed errors.

Context was often lost as errors propagated.

Upper layers received vague `500 Internal Server Errors` without clear root causes.

#### No categorization made monitoring impossible

Errors weren’t classified by severity or behavior: A `context.Canceled` error may be a normal behavior when the user closes the browser tab, but it’s important if the request is canceled because that query is randomly slow.

Important issues were buried under noisy logs, making them hard to identify.

Without categorization, it was impossible to monitor error frequency, severity, or impact effectively.

---

### It’s time to centralize error handling

#### Back to the drawing board

To address the growing challenges, we decided to build a better error strategy around the core idea of **centralized and structured error codes**.

- **Errors are declared everywhere →** Centralize error declaration in a single place for better organization and traceability.
- **Inconsistent and arbitrary logs →** Structured error codes with clear and consistent formatting.
- **Improper error handling →** Standardize error creation and checking on the new `Error` type with a comprehensive set of helpers.
- **No categorization →** Categorize error codes with tags for effective monitoring through logs and metrics.

#### Design decisions

- **All error codes are defined at a centralized place with namespace structure.**

  Use namespaces to create clear, meaningful, and extendable error codes. Example:

  - `PRFL.USR.NOT_FOUND` for “User not found.”
  - `FLD.NOT_FOUND` for “Flow document not found.”

  Both can share an underlying base code `DEPS.PG.NOT_FOUND`, meaning “Record not found in PostgreSQL.”

- **Each layer of service or library must only return its own namespace codes.**

  Each layer of service, repository, or library declares its own set of error codes.

  When a layer receives an error from a dependency, it must wrap it with its own namespace code before returning it.

  For example: When receiving an error `gorm.ErrRecordNotFound` from a dependency, the “database” package must wrap it as `DEPS.PG.NOT_FOUND`. Later, the “profile/user” service must wrap it again as `PRFL.USR.NOT_FOUND`.

- **All errors must implement the `Error` interface.**

  This creates a clear boundary between errors from third-party libraries (`error`) and our internal `Error`s.

  This also helps for migration progress, to separate between migrated packages and not-yet-migrated ones.