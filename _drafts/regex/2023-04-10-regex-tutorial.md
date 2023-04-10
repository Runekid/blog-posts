---
layout: post
author: Wouter Van Schandevijl
title:  "Regular Expressions: Tutorial"
date:   2023-04-10 00:00:00 +0200
desc: >
  Regex tutorial
categories: productivity
tags: [regex,tutorial,cheat-sheet]
series: regex
toc:
  title: Tutorial
---


TODO: need more/better examples here

EMPLI;TRAVI;PAIDD;PAIFD;REAVC;REMUI;ANNUC;MODUI;IMSTC;PREPN;TYPAC;BASEC;BASEV;INDEX;ORREI;JO99N;JFO9N;RESERVE;BASEV;;
867;77;20211201;20211231;1;0235;2;01;0000000;gMMO;1;1;17,44;000000;90;2C;0{;000;174400;;
867;77;20211201;20211231;2;0236;2;01;0000000;gMMO;1;1;17,44;000000;16;2C;0{;000;174400;;
867;77;20211201;20211231;2;0236;2;05;0000000;gMMO;1;1;17,44;000000;16;0{;0{;000;174400;;

^(\d+);(\d+);\d+;(\d+);(?:[^;]+;){8}([^;]+);.*
$1;$2;$3;$4



TODO: rename this post to "Cheat sheet"

WIP: we zaten hier...
- http://regexlib.com/CheatSheet.aspx?AspxAutoDetectCookieSupport=1
- https://www.debuggex.com/cheatsheet/regex/javascript

On how to match stuff.

Don'ts:
- Do not use regex for
- Html: Use a Html parser
- Xml: Use an xml parser instead

You could still use regex if you just want a small part from the html or xml
and you know how the html/xml will look like *exactly* (for example when it is
generated from a tool and not hand written)


tool regex from book?


lookahead: /aa(?=bb)/
lookbehind: /(?<=aa)bb/
from the position parsed so far look ahead/behind

<!--more-->


# Syntax


| Regex      | Matches                                                 | Remarks
|------------|---------------------------------------------------------|--------
| Literals
| abc        | Literals match themselves. Here: 'abc'
|
| Metacharacters
| ^s         | 's' but only at the start of the input
| e$         | 'e' but only at the end. ^ and $ match a position.
| \\$        | Escape character: Match '$' anywhere in the input
| .          | Match any one character
| \<e        | Starts with e
| e\>        | Ends with e
| \b         | Word boundary
| 
| Character classes
| [az]       | 'a' or 'z'
| [0-9]      | A single digit                                          | \d
| [a-zA-Z]   |
| [a-Z]      | This might also include '[]\\^_`'
| [\n$^-]    | A newline, $, ^ or a hyphen                             |
| [^a\b]     | Everything but 'a' and 'backspace'
|
| Multipliers
| a?         | Zero or one
| a*         | Zero or more
| a+         | One or more
| a{2,5}     | Two to five
| a{4}       | Exactly four
| a{,5}      | Zero to five                                            | a{5,}
|
| Grouping
| (ab\|yz)   | 'ab' or 'yz'.
| (?:ab)     | 'ab' but non-capturing group
{: .table-code}


\Q begin literal sequence
\E end literal sequence
\Qlite[]ral\E --> Only for Perl, PHP, Java, SublimeText, ...

\A Start of string / \Z

| Shorthand | Meaning
|--------|--------
| \d     | [0-9]
| \w     | [a-zA-Z0-9_]
| \s     | Whitespace
| \t     | Tab
| \r     | Carriage return
| \n     | Newline
| \b     | Word boundary
{: .table-code}

Shorthands can be inversed by capitalizing them: `\D` (not a digit)

Also `\v` (vertical tab), `\f` (form feed).  
Inside a character clas `\b` matches backspace.

# Options

modifiers?

i
g --> global and how it affects . and \A \Z vs ^ $
x free form expressions

## Replacement

| Replacement | Description
|-------------|------------
| $1          | First captured group
| $2          | Second captured group
| $$          | A literal $
| $&          | Entire match
| $0 vs $& ???
| $`          | Before matched string
| $'          | After matched string
| $+          | Last matched string
{: .table-code}

Some implementations use `\` instead of `$`.

Use non-capturing groups `(?:)` to keep your backreferences (`$1`, `$2`, ...) in check.  
Or use named groups if supported in your regex implementation.


## Looking around

this(?=must be after) = lookahead
this(?!must not be after)

(?<=must be before)this = lookbehind
(?<!must not be before)this

<(backreference)>\w*</\1>




Change import { Module } -> {Module}
/(import (?:.+)?\{)(?:\s*)(.*)(?:\s+)(\} from ')/ Replace with $1$2$3



Take the "(?<=/)(([^\\/:*?""<>|]+) (?2))(?=/) example for lookbehind/ahead..


Tel: 000-000-000

Tel:\S\d\d\d-

(?:non capturing)

this(?=must be after) = lookahead
this(?!must not be after)

(?<=must be before)this = lookbehind
(?<!must not be before)this

<(backreference)>\w*</\1>

<Name>MyName</Name><LastName>MyLastName</LastName>
<Name>OtherName</Name><LastName>OtherLastName</LastName>
<Name>ThatName</Name>
<Name>MoreName</Name>
<Name>YouKnow</Name>
<Name>EvenMore</Name>
<Name>Name</Name>














[advanced-cheat-sheet]: https://www.cheatography.com/davechild/cheat-sheets/regular-expressions/
