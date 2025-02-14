# Parsing Patterns

In cursornext, there are four basic functions that we would use frequently:

- Move
- Test
- Read
- Backtrack

## Move

To move the cursor, we can use `move`, `next`, `previous` or `setIndex`.

`move` moves the cursor a number of characters.

`next` is an alias of `move` that ensures that the cursor will only move forward. Default argument for move is 1.

`previous` is an alias of `move` that ensures that the cursor will only move backward. Default argument for move is 1.

```ts
const cursor = Cursor.from('Hello, World!')
t.assert(cursor.index === 0)

cursor.move(5)
t.assert(cursor.index === 5)

cursor.next(-1)
t.assert(cursor.index === 5)

cursor.next(2)
t.assert(cursor.index === 7)

cursor.previous(-1)
t.assert(cursor.index === 7)

cursor.previous() // default is 1
t.assert(cursor.index === 6)
```

`setIndex` method lets you specify exact desired position for cursor:

```ts
const cursor = Cursor.from('Hello, World!')

cursor.setIndex(5)
t.assert(cursor.index === 5)

cursor.setIndex(3)
t.assert(cursor.index === 3)
```

It can be very handful when working with regular expression:

```ts
const cursor = Cursor.from('-----1996-----')

const regexResult = cursor.exec(/[0-9]/)
cursor.setIndex(regexResult.index)

t.assert(cursor.startsWith('1996'))
```

## Test

In cursornext, there are three methods that we can use to test a cursor:

- `startsWith`
- `oneOf`
- `exec`

`startsWith` takes an input string and test whether text beginning at the cursor position starts with that given string. For example:

```
1 | Hello, World!
  |        ^
  |        cursor
```

```ts
t.assert(cursor.startsWith('World'))
```

`oneOf` is similar to `startsWith` except that it tests against multiple of strings. If the text beginning at the cursor position starts with any of these, it returns the first string that matches. Otherwise, it returns `null`.

```
1 | ----abcd-----
  |     ^
  |     cursor
```

```ts
t.assert(cursor.oneOf(['bcd', 'cad', 'ab', 'abcd']) === 'ab')
t.assert(cursor.oneOf(['bca', 'bcd']) === null)
```

`exec` executes the text from the cursor position against the given input regular expression. If matches it will return a `RegExpExecArray`; otherwise, it will return `null`:

```
1 | ----1234----
  |   ^
  |   cursor
```

```ts
const regexResult = cursor.exec(/[0-9]+/)

t.assert(regexResult[0] === '1234')
t.assert(regexResult.index === 4)
```

Notice the hat symbol `^` will be used to indicate the cursor position instead of the beginning of the document as common use. This makes `exec` works similar to `startsWith`. For example:

```
1 | ----1234----
  |   ^
  |   cursor
```

```ts
const regexResult = cursor.exec(/^[0-9]+/)

t.assert(regexResult === null)
```

```
1 | ----1234----
  |     ^
  |     cursor
```

```ts
const regexResult = cursor.exec(/^[0-9]+/)

t.assert(regexResult[0] === '1234')
t.assert(regexResult.index === 4)
```

## Read

The most common way to read the document is to create a marker cursor. Then, we can use the `takeUntil` method to read the text between two cursors:

```ts
const cursor = Cursor.from('Hello, World!')

const marker = cursor.clone()
cursor.next(5)

// Read the input from the marker cursor position
// to the current cursor position.
const output = marker.takeUntil(cursor)

t.assert(output === 'Hello')
```

`exec` and `lookahead` can also be used to extract the information from the document:

```ts
const cursor = Cursor.from('Hello, World!')

t.assert(cursor.lookahead(5) === 'Hello')
cursor.next(5)

t.assert(cursor.exec(/[a-zA-Z]+/)[0] === 'World')
```

## Backtrack

Sometimes, when working with complex syntax constraint, you may need to backtrack to the previous position. The following example illustrates how to backtrack using `moveTo` method:

```ts
// Mark the cursor position by cloning the cursor.
const marker = cursor.clone()

// Do something with the cursor!
// ...
// ...

// If some constraint are not sastisfied, backtrack
// the cursor to the marked position.
if (!sastisfied) {
  cursor.moveTo(marker)
}
```
