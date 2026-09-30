# NAME

Perl::Critic::Policy::RegularExpressions::ProhibitRegexForStringEquality - Compare a string with eq, not with a regex anchored at both ends.

# VERSION

version 0.001

# Perl::Critic::Policy::RegularExpressions::ProhibitRegexForStringEquality

A match of literal text anchored at both ends asks whether a string is that
text, which is `eq`.  An anchored group of literal alternatives asks whether a
string is one of several, which is `any` from [List::Util](https://metacpan.org/pod/List%3A%3AUtil), or a hash lookup.
Either one says what it does in plain words, and runs no pattern:

```perl
if ( $name =~ m/\Afoo\z/ ) { ... }                   # reported
if ( $name eq 'foo' ) { ... }

return if $tool =~ m/\A(?:Skill|Read)\z/;           # reported
return if List::Util::any { $tool eq $_ } qw{Skill Read};
```

## PROHIBITED

```perl
$str =~ m/\Afoo\z/          # eq
$str !~ m/\Afoo\z/          # ne
$str =~ m/^foo\z/           # ^ is the start without /m
$str =~ m/\Afoo\.bar\z/     # an escaped character is still literal text
$str =~ m/\A foo \z/x       # under /x this is the string foo
$str =~ m/\A(?:foo)\z/      # a group of one alternative is still eq
$str =~ m/\A(?:a|b|c)\z/    # any, or a hash lookup
$str !~ m/\A(?:a|b)\z/      # none, or a failed hash lookup
grep { m/\Afoo\z/ } @names  # a match against $_ counts too
```

## ALLOWED

```perl
$str =~ m/\Afoo$/           # $ also matches before a trailing newline
$str =~ m/\Afoo\Z/          # so does \Z
$str =~ m/^foo\z/m          # /m: ^ is the start of any line
$str =~ m/\Afoo\z/i         # /i, which eq cannot do
$str =~ m/\A(foo|bar)\z/    # a capture, which the code may use
$str =~ m/\A(?i:a|b)\z/     # a group with modifiers of its own
$str =~ m/\Afoo|bar\z/      # alternation outside a group anchors each side once
$str =~ m/\Afo+\z/          # a quantifier
$str =~ m/\A[ab]\z/         # a character class
$str =~ m/\A$foo\z/         # interpolation
$str =~ m/\A\z/             # nothing between the anchors, see CAVEATS
$str =~ s/\Afoo\z/bar/      # a substitution
my $rx = qr/\Afoo\z/;       # a compiled regex
split m/\Afoo\z/, $str;     # the pattern that split takes
```

## MODIFIERS

`/i` exempts a pattern, because `eq` is case sensitive.  `fc` on both sides
would do, but that is a rewrite and not a substitution.

`/m` exempts a pattern that starts with `^`, because `^` then matches at the
start of any line.  A pattern that starts with `\A` is reported under `/m` too.

`/g` exempts a pattern, because in scalar context it moves `pos`, which `eq`
does not.

`/s` and `/x` change nothing here: there is no `.` in literal text, and
whitespace under `/x` is not text.

A modifier in scope from a `use re` counts the same as one written on the
match, so a file under `use re '/i'` is exempt throughout.

## CAVEATS

`m/\A\z/` is `eq q{}`, or `!length`, but it is not reported.  The policy asks
for text between the anchors.

A group of one alternative is reported as `eq`, and a group of more than one as
`any`.  Where the alternatives are many, or the test runs in a loop, a hash of
them built once is faster than `any`.

## METHODS

### supported\_parameters

### default\_severity

### default\_themes

### applies\_to

### violates

Standard [Perl::Critic::Policy](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy) interface.  Returns a violation for a match
whose pattern is literal text, or a plain group of literal alternatives,
between a start anchor and `\z`.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
