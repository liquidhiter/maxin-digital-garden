---
{"dg-publish":true,"permalink":"/0-notes/regex-practice/","noteIcon":"","created":"2024-01-27T08:01:02.690+01:00","updated":"2024-01-27T08:01:11.850+01:00"}
---

## Lookaround
[reference]: https://www.regular-expressions.info/lookaround.html

### Lookahead
negative lookahead: match something not followed by something else

- example: match a `q` not followed by a `u`
```bash
q(?!u)
```
- syntax: `(?!expr)`
- question: what can be used in the `expr`?
  - seems that normal regex is supported: `b(?!(a){3})`
  - how about nested lookahead?
    - `b(?!(c(?!d)))`
    - seems negation of the negative is the positive ?
