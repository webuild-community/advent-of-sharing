**Author:** Vũ

---

# Empower diff for testing in your Go code - Part 1

[Original article](https://olivernguyen.io/w/diff.testing/)

![favicon](https://olivernguyen.io/images/favicon.png)

![og-image](https://iolivernguyen.github.io/images/ogx.png)

---

### A new working day

Suppose that you, a good software engineer, just got hired by an e-commerce business. And you are joining a team to work on the profile service to manage customers and their addresses.

There are a `Customer` and an `Address` struct as follows, with a couple of `gorm` tags for the database schema:

```go
type Customer struct {
    ID         uuid.UUID `gorm:"type:uuid;primaryKey" json:"id"`
    Name       string    `json:"name"`
    Age        int       `json:"age"`
    Gender     string    `json:"gender"`
    Email      string    `gorm:"uniqueIndex" json:"email"`
    Phone      string    `json:"phone"`
    Occupation string    `json:"occupation"`
    Addresses  []Address `gorm:"foreignKey:CustomerID" json:"addresses"`
    Languages  []string  `gorm:"type:text[]" json:"languages"`
}

type Address struct {
    ID uuid.UUID
    // ...
}
```

You look at the method `UpdateCustomerWithAddress()` which to update a customer and their associated addresses. The code looks like this:

```go
func (r *Repository) UpdateCustomerWithAddress(ctx context.Context, customer *Customer) error {
    return r.db.WithContext(ctx).Transaction(func(tx *gorm.DB) error {
        if err := tx.Save(customer).Error; err != nil {
            return err
        }

        for i := range customer.Addresses {
            customer.Addresses[i].CustomerID = customer.ID
        }

        if err := tx.Where("customer_id = ?", customer.ID).
            Delete(&Address{}).Error; err != nil {
            return err
        }

        if len(customer.Addresses) > 0 {
            if err := tx.Create(&customer.Addresses).Error; err != nil {
                return err
            }
        }

        return nil
    })
}
```

So far so good. It looks like a typical method to update a customer and their addresses, uses `gorm` to interact with the database, and wraps the operation in a transaction.

Now you look at the test for this method. The test looks like this:

```go
func TestRepository_UpdateCustomerWithAddress(t *testing.T) {
    db, mock, _ := sqlmock.New()
    gdb, _ := gorm.Open(postgres.New(postgres.Config{
        Conn: db,
    }), &gorm.Config{})

    repo := &Repository{db: gdb}

    customer := &Customer{
        ID:   uuid.New(),
        Name: "John Doe",
        Addresses: []Address{
            {ID: uuid.New(), /* ... */},
        },
    }

    mock.ExpectBegin()
    mock.ExpectQuery(`UPDATE "customers" SET`).
        WithArgs(/* ... */).
        WillReturnRows(sqlmock.NewRows([]string{"id"}).AddRow(customer.ID))

    mock.ExpectExec(`DELETE FROM "addresses" WHERE`).
        WithArgs(customer.ID).
        WillReturnResult(sqlmock.NewResult(1, 1))

    mock.ExpectQuery(`INSERT INTO "addresses"`).
        WithArgs(/* ... */).
        WillReturnRows(sqlmock.NewRows([]string{"id"}).AddRow(customer.Addresses[0].ID))
    mock.ExpectCommit()

    err := repo.UpdateCustomerWithAddress(context.Background(), customer)
    require.NoError(t, err)
    require.NoError(t, mock.ExpectationsWereMet())
}
```

So, the test is using [go-sqlmock](https://github.com/DATA-DOG/go-sqlmock) to mock the database and assert the SQL queries. It looks like a lot of boilerplate code to write and maintain. But at least it works. And by looking at the tests, we know what the actual SQL queries are.

Okay, your onboarding task is to add a couple of new fields to the `Address` struct: `Phone` and `Receiver`. Because your boss told you, as a delivery business service, a customer may want to deliver to different addresses with different receivers and phone numbers.

It’s just a simple change, right? You add the fields to the `Address` struct, and `UpdateCustomerWithAddress()` method should be able to handle it automatically with the magic of `gorm`.

You start all services on local, send a couple of HTTP requests with the new fields, and everything works fine: database schema is updated, data is saved, and retrieved correctly. You are confident that the changes are good to go.

Now, before sending a pull request, you need to update the tests to cover the new fields. It should be a simple fix, and the tests will work again, right?

![sqlmock error](https://olivernguyen.io/w/diff.testing/a1.png)

Ugh, you look at the error message and quickly get lost in the SQL queries. You know that you need to update the SQL queries to include the new fields, but it’s hard to figure out where to put the changes. You try to update the queries, but the tests keep failing. You are frustrated and start to doubt your changes.

Okay, at last, maybe you can copy the SQL queries from the error message and paste them into the test. But it’s not a good practice. You just overrode the actual queries with the new queries. Who knows if your new queries are always correct? How to compare the new queries with the previous queries? And what if the queries change in the future? You need to update the tests again and again. It’s a nightmare!

---

### Finding a better way to test SQL queries

It’s time to find a better way to test the SQL queries. Instead of the messy regexp matching, you need a tool that can compare the actual queries with the expected queries, and show you the differences, so you can quickly identify what’s wrong.

Luckily, `go-sqlmock` has [QueryMatcherOption](https://pkg.go.dev/github.com/DATA-DOG/go-sqlmock#QueryMatcherOption) to allow you to define a custom matcher for the SQL queries. You can use this feature to compare the actual queries with the expected queries, and show the differences in a human-readable format.

Let’s create a new package `sqlmockz`:

```go
package sqlmockz

import "github.com/DATA-DOG/go-sqlmock"

type diffMatcherImpl struct {
    sqlmock.Sqlmock
}

func OptionDiffMatcher() sqlmock.QueryMatcherOption {
    return sqlmock.QueryMatcherOption(&diffMatcherImpl{})
}
```

And the usage will be like this:

```go
db, mock, _ := sqlmock.New(OptionDiffMatcher())
gdb, _ := gorm.Open(postgres.New(postgres.Config{
    Conn: db,
}), &gorm.Config{})

// use mock as usual
```

---

### Implementing the diff matcher

The package [ezpkg.io/diffz](https://ezpkg.io/diffz) provides some useful functions to compare the differences between two strings. Two notable functions are [diffz.ByChar()](https://pkg.go.dev/ezpkg.io/diffz#ByChar) and [diffz.ByLine()](https://pkg.go.dev/ezpkg.io/diffz#ByLine).

In our case of comparing SQL queries, the `ByChar()` function is more suitable, because it can show the added columns. While in case of comparing JSON or YAML in API response, the `ByLine()` function serves better.

Now, it’s time to implement the `Match()` method in the `diffMatcherImpl` struct:

- Use `diffz.ByChar()` to compare the actual SQL with the expected SQL.
- If the actual SQL is different from the expected SQL, print the actual SQL, the expected SQL, and the differences.
- Use [colorz](https://ezpkg.io/colorz) to colorize the output for better readability.

```go
package sqlmockz

import (
    "fmt"

    "ezpkg.io/colorz"
    "ezpkg.io/diffz"
    "github.com/DATA-DOG/go-sqlmock"
)

type diffMatcherImpl struct {
    sqlmock.Sqlmock
}

func (m *diffMatcherImpl) Match(expected, actual string) error {
    if expected == actual {
        return nil
    }

    diff := diffz.ByChar(expected, actual)

    fmt.Println(colorz.Yellow("=== SQL DIFF ==="))
    fmt.Println(colorz.Cyan("Expected:"))
    fmt.Println(expected)
    fmt.Println(colorz.Cyan("Actual:"))
    fmt.Println(actual)
    fmt.Println(colorz.Cyan("Diff:"))
    fmt.Println(diff.String())

    return fmt.Errorf("sql does not match")
}

func OptionDiffMatcher() sqlmock.QueryMatcherOption {
    return sqlmock.QueryMatcherOption(matcherFunc(func(expected, actual string) error {
        m := &diffMatcherImpl{}
        return m.Match(expected, actual)
    }))
}

type matcherFunc func(expected, actual string) error

func (f matcherFunc) Match(expected, actual string) error {
    return f(expected, actual)
}
```

You can now use the `OptionDiffMatcher` in sqlmock:

```go
db, mock, _ := sqlmock.New(OptionDiffMatcher())
```

And the error message will show exactly what’s need to be changed:

![diff output](https://olivernguyen.io/w/diff.testing/a2.png)

The actual SQL is missing the `phone` and `receiver` columns. You can quickly identify the problem and update the test accordingly!

---

### Make it even better with ignore spaces

But you still need to careful put the spaces correctly in the SQL queries:

```go
const expectedSQL = `
UPDATE "addresses"
SET "street" = $1, "city" = $2
WHERE "customer_id" = $3
`
```

It should be better to ignore the spaces when comparing the SQL queries, so you can freely format the SQL queries in a more readable way:

```go
const expectedSQL = `UPDATE "addresses" SET "street"=$1,"city"=$2 WHERE "customer_id"=$3`
```

Yes, there is a sister function [diffz.ByCharZ()](https://pkg.go.dev/ezpkg.io/diffz#ByCharZ) to ignore the spaces when comparing the SQL queries. Just put it in the `Match()` method and you are good to go:

```go
func (m *diffMatcherImpl) Match(expected, actual string) error {
    if expected == actual {
        return nil
    }

    diff := diffz.ByCharZ(expected, actual)

    if !diff.Equal() {
        fmt.Println(colorz.Yellow("=== SQL DIFF (ignore spaces) ==="))
        fmt.Println(colorz.Cyan("Expected:"))
        fmt.Println(expected)
        fmt.Println(colorz.Cyan("Actual:"))
        fmt.Println(actual)
        fmt.Println(colorz.Cyan("Diff:"))
        fmt.Println(diff.String())
        return fmt.Errorf("sql does not match")
    }

    return nil
}
```

---

### Conclusion

By a simple change to the matcher with [diffz.ByCharZ()](https://pkg.go.dev/ezpkg.io/diffz#ByCharZ), you can confidently make changes and update the tests. When there is any change in the SQL queries, the error message will show you exactly what’s need to be updated.

Instead of spending hours struggling with the messy regexp matching, you can now focus on the actual important bit and grab a nice cup of coffee.

Enjoy coding!