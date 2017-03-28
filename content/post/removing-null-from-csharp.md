+++
date = "2017-03-27T17:17:05-06:00"
title = "Removing Null from C#"
draft = false
author = "Eric Andres"
+++

Null references can be a source of subtle bugs in software. `Maybe` is a tool that, while deceptively similar, provides
much greater safety. Let's look at an example of retrieving a document from a database. First, with code that uses null.

<!--more-->

```csharp
public class AccountRepository
{
  public Account Find(string id) { /* ... */ }
}

// ...

var account = accountRepository.Find("user-name");
Console.WriteLine(account.FirstName);
```

By calling the `Find` method on the `AccountRepository`, we get back a reference to an account. Printing the first name works, as long as an account actually exists with the given ID. Otherwise, we'll get the dreaded `NullReferenceException`. To prevent this we
have to add a null check.

```csharp
var account = accountRepository.Find("user-name");
if (account != null) 
{
  Console.WriteLine(account.FirstName);
}
```

It's very easy to forget that null check, or to convince yourself that, in this particular scenario, it could never
happen. `Maybe` makes it impossible to ignore the "no value" case by wrapping the value. Let's look at how `Maybe` (also known as `option`) works in F#.

```
let findAccount (id:string) : Account option = 
  if /* we have an account */
    then Some account
    else None

let accountOption = findAccount "user-name"
match accountOption with
  | Some account -> Console.WriteLine(account.FirstName)
  | None -> Console.WriteLine("Account not found")
```

In F#, it's impossible for an `option` type to be assigned null. The type system simply won't allow it. To provide a value for the `option` we use the `Some` constructor. Otherwise, we use `None`. We can then pattern match the value to see if we have a `Some` or a `None`. Next let's look at a simple implementation of `Maybe` in C#. The basis for this code is from Mark Seeman's [Encapsulation and SOLID](https://app.pluralsight.com/player?course=encapsulation-solid&author=mark-seemann&name=encapsulation-solid-m2-encapsulation&clip=18&mode=live) course on Pluralsight.

```csharp
public struct Maybe<T>
{
  readonly IEnumerable<T> values;

  public static Maybe<T> Some(T value)
  {
    if (value == null)
    {
      throw new InvalidOperationException();
    }

    return new Maybe<T>(new [] { value });
  }

  public static Maybe<T> None => new Maybe<T>(new T[0]);

  private Maybe(IEnumerable<T> values)
  {
    this.values = values;
  }

  public bool HasValue => values != null && values.Any();

  public T Value
  {
    get
    {
      if (!HasValue)
      {
        throw new InvalidOperationException("Maybe does not have a value");
      }

      return values.Single();
    }
  }
}
```

With `Maybe`, we have wrapped an array in an object wrapper. There is significance in using an array, instead of a bare
value and a flag stating where there is a value. Maybe is actually a special case of list types (such as `IEnumerable`
in .Net). Instead of the type containing zero-to-many items, it contains either zero or one. Let's rewrite our example
above using Maybe.

```csharp
public class AccountRepository
{
  public Maybe<Account> Find(string id) 
  {
    if (/* account was found in database */) 
    {
      return Maybe<Account>.Some(account);
    }
    else
    {
      return Maybe<Account>.None;
    }
  }
}

// ...

var account = accountRepository.Find("user-name");
if (account.HasValue) 
{
  Console.WriteLine(account.Value.FirstName);
}
else
{
  Console.WriteLine("No account found");
}
```

Working with the `Maybe` type as is doesn't provide much of an advantage over null references, since we still have to do a value check
with `HasValue`. Let's add a method to make working with it more natural, similar to the `match` expression in F#.

```csharp
public U Case<T, U>(Func<T, U> some, Func<U> none)
{
  return this.HasValue
    ? some(this.Value)
    : none();
}
```

Now, using `Maybe` feels similar to working with a `match` expression.

```csharp
var maybeAccount = accountRepository.Find("user-name");
var message = maybeAccount.Case(
  some: account => account.FirstName,
  none: () => "No account found"
);
Console.WriteLine(message);
```

Moving in this direction makes the code more expressive, focusing more on what output should be produced, instead of checking
for corner cases. But, at the same time, we force the programmer to consider the `None` case, since it's impossible to
call `Case` without a `none` function.

We can also use `Maybe` to code in a functional style. Let's look at a `map` implementation.

```csharp
public Maybe<U> Map<T, U>(Func<T, U> mapper)
{
  return this.HasValue 
    ? Maybe<U>.Some(mapper(this.Value));
    : Maybe<U>.None;
}
```

Now we can continue computation on a result, regardless of whether a value is present.

```
public Maybe<int> Divide (int a, int b)
{
  return b == 0
    ? Maybe<int>.None
    : Maybe<int>.Some(a / b);
}

public Maybe<int> DoAComputation(int a, int b)
{
  return Divide(a, b)
    .Map(x => x * x)
    .Map(x => x + 1);
}

var x = DoAComputation(4, 2); // => Some(5)
var y = DoAComputation(5, 0); // => None
```

## Conclusion

While working with `Maybe` might not feel quite as natural in C# as it does in a language with first class support, it
can greatly reduce the number of null reference exceptions. It also guides code in a more functional direction. Using
`Maybe` in C# code provides a similar paradigm shift for null references that LINQ did for list types. Try out
Pluralsight's implementation at https://github.com/pluralsight/maybe-dotnet.


