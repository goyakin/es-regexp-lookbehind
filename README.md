# RegExp Lookbehind Assertions

Authors: Gorkem Yakin, Nozomu Katō

## Introduction

Lookarounds are zero-width assertions that match a string without consuming anything. ECMAScript has lookahead assertions that does this in forward direction, but the language is missing a way to do this backward which the lookbehind assertions provide. With lookbehind assertions, one can make sure that a pattern is or isn't preceded by another, e.g. matching a dollar amount without capturing the dollar sign.

## High Level API

There are two versions of lookbehind assertions: positive and negative.

Positive lookbehind assertions are denoted as `(?<=...)` and they ensure that the pattern contained within precedes the pattern following the assertion. For example, if one wants to match a dollar amount without capturing the dollar sign, `/(?<=\$)\d+(\.\d*)?/` can be used, matching `'$10.53'` and returning `'10.53'`. This, however, wouldn't match `€10.53`.

Negative lookbehind assertions are denoted as `(?<!...)` and, on the other hand, make sure that the pattern within doesn't precede the pattern following the assertion. For example, `/(?<!\$)\d+(?:\.\d*)/` wouldn't match `'$10.53'`, but would `'€10.53'`.

All regular expression patterns, even unbounded ones, are allowed as part of lookbehind assertions. Therefore, one could, for example, write `/(?<=\$\d+\.)\d+/` to match a dollar amount and capture just the fraction part.

Patterns normally match starting from the leftmost sub-pattern and move on to the sub-pattern on the right if the left sub-pattern succeeds. When contained within a lookbehind assertion, the order of matching would be reversed. Patterns would match starting from the rightmost sub-pattern and advance to the left instead. For example, given `/(?<=\$\d+\.)\d+/`, the pattern would first find a number and ensure first that it is preceded by `.` going backward, then `\d+` starting from `.`, and lastly `$` starting from where `\d+` within the assertion begins. The backtracking direction would also be reversed as a result of this.

## Open Questions

### Matching Direction

Unbounded repetitions require a pattern within a lookbehind assertion to be run backward. Running patterns backward would have two side effects regarding capture groups.

Firstly, capture groups within a lookbehind assertion would have different values compared to when they are outside if the groups have sub-patterns with greedy quantifiers. When within a lookbehind assertion, groups on the right would capture most characters rather than the ones on the left. For example, given `/(?<=(\d+)(\d+))$/` and the string `'1053'`, the second group would capture `'053'` and the first group would be `'1'`. With `/^(\d+)(\d+)/`, on the other hand, the first group would capture `'105'` and the second group would be `'3'`.

The other side effect is how backreferences are resolved. If a backreference together with its corresponding group is within a lookbehind assertion and the backreference is placed to the right of the group, the backreference would match the empty string given that the group wouldn't have been processed at that point. For example, normally `/(a)\1-/` matches `'aa-'`, whereas `/^(?<=(a)\1)-/` would match `'a-'` instead.
