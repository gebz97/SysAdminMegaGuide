#regex #cheatsheet #regexcheatsheet
## 🔤 Character Classes

| Pattern | Description                       |
| ------- | --------------------------------- |
| `.`     | Any character except newline      |
| `\d`    | Digit (0-9)                       |
| `\D`    | Not a digit                       |
| `\w`    | Word character (a-z, A-Z, 0-9, _) |
| `\W`    | Not a word character              |
| `\s`    | Whitespace (space, tab, newline)  |
| `\S`    | Not whitespace                    |
| `\n`    | Newline                           |
| `\t`    | Tab                               |
| `\r`    | Carriage return                   |
## 🎯 Anchors

| Pattern | Description          |
| ------- | -------------------- |
| `^`     | Start of string/line |
| `$`     | End of string/line   |
| `\b`    | Word boundary        |
| `\B`    | Not a word boundary  |
| `\A`    | Start of string      |
| `\Z`    | End of string        |
## 📊 Quantifiers

| Pattern | Description       |
| ------- | ----------------- |
| `*`     | 0 or more         |
| `+`     | 1 or more         |
| `?`     | 0 or 1            |
| `{3}`   | Exactly 3         |
| `{3,}`  | 3 or more         |
| `{3,5}` | 3 to 5            |
| `*?`    | Lazy zero or more |
| `+?`    | Lazy one or more  |
| `??`    | Lazy zero or one  |
## 🎨 Character Sets

| Pattern    | Description          |
| ---------- | -------------------- |
| `[abc]`    | Any of a, b, or c    |
| `[^abc]`   | Not a, b, or c       |
| `[a-z]`    | Any lowercase letter |
| `[A-Z]`    | Any uppercase letter |
| `[0-9]`    | Any digit            |
| `[a-zA-Z]` | Any letter           |
## 🔀 Groups & References

| Pattern        | Description              |
| -------------- | ------------------------ |
| `(abc)`        | Capture group            |
| `(?:abc)`      | Non-capturing group      |
| `(?<name>abc)` | Named capture group      |
| `\1`           | Backreference to group 1 |
| `(abc\|def)`   | Alternation (or)         |
## 👀 Lookarounds

| Pattern    | Description         |
| ---------- | ------------------- |
| `(?=abc)`  | Positive lookahead  |
| `(?!abc)`  | Negative lookahead  |
| `(?<=abc)` | Positive lookbehind |
| `(?<!abc)` | Negative lookbehind |
## 🚩 Flags (Modifiers)

| Flag | Description         |
| ---- | ------------------- |
| `g`  | Global match        |
| `i`  | Case insensitive    |
| `m`  | Multiline           |
| `s`  | Dot matches newline |
| `u`  | Unicode             |
| `y`  | Sticky              |
## 💡 Common Examples
# Email validation
```
^[\w\.-]+@[\w\.-]+\.\w+$
```
# URL matching
```
https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)
```
# Phone number (US)
```
\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}
```
# IP address
```
\b(?:\d{1,3}\.){3}\d{1,3}\b
```
# Date (YYYY-MM-DD)
```
\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])
```
# Strong password
```
^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$
```
# Hex color
```
#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})
```
# HTML tag
```
<\/?[\w\s]*>|<.+[\W]>
```
## 🔧 Escape Sequences

| Pattern | Description                 |
| ------- | --------------------------- |
| `\.`    | Literal dot                 |
| `\\`    | Literal backslash           |
| `\+`    | Literal plus                |
| `\*`    | Literal asterisk            |
| `\?`    | Literal question mark       |
| `\[`    | Literal opening bracket     |
| `\]`    | Literal closing bracket     |
| `\(`    | Literal opening parenthesis |
| `\)`    | Literal closing parenthesis |
| `\{`    | Literal opening brace       |
| `\}`    | Literal closing brace       |