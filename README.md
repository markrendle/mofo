# mofo

Everybody seems to be creating serialization formats, so here's mine: Mark's Object FOrmat

## Rationale

JSON is fairly lightweight, but it's still cruftier than it needs to be and doesn't handle dates or some other really useful bits properly.
MOFO reduces the cruft and does additional useful stuff (dates, binaries, typed nulls and arrays, binary representation... and comments).

## RFC

I didn't do CS or anything like that, so I don't know BNF; therefore, the following will be in English.

**MIME type:** `application/mofo`

### Primitives

Six basic primitive data types are supported: String, Number, Date, Boolean, Binary and UUID:

* String elements are surrounded by `$`, as in `$MOFO is awesome$`.
  * A null string is represented by `$$`
  * An actual $ symbol in a string is escaped with a backslash, as in `$Total: \$599.99$`
  * Standard special string characters are represented as usual: `\n`, `\t`, etc
  * Actual backslashes are represented by a double backslash `\\`
* Number elements are surrounded by `#`, as in `#42#`
  * A null number is represented by `##`
  * More explicit number types can be expressed within the delimiters using upper-case suffixes:
     * 8-bit integer, B: `#24B#`
     * 16-bit integer, S: `#1024S#`
     * 32-bit integer, none (default integer type): `#1024#`
     * 64-bit integer, L: `#1024L#`
     * 32-bit single precision floating point, F: `#1024.0F#`
     * 64-bit double precision floating point, none (default floating point type): `#1024.0#`
  * Octal formatting can be denoted using a leading zero: `#02000#`
  * Hex formatting can be denoted using a `0x` prefix and using lower-case a, b, c, d, e, and f: `#0x40B#`, `#0xff23fac2#`, `#0x4aL#`
  * To express constraints, see BADASSERY below
* Date/time values are surrounded by `/` and are in [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601), as in `/2013-08-11T15:17:10Z/`
  * A null date is represented by `//`
  * To specify a Date only, omit the time part of the expression, as in `/2013-08-11/`
  * To specify a Time only, omit the date part of the expression, as in `/15:00:00Z/`
  * All times are to be expressed in UTC
* Boolean values are not delimited; they are represented by the single characters `^` for true and `!` for false
  * A null boolean is represented by a `?`
* Binary data is represented by non-delimited, lower-case hexadecimal digit pairs surrounded by ampersands, as in `&4d4f464f&`.
  * A null binary value is represented by `&&`
  * Base 64 formatting can be denoted by using a leading `+` (see Base 64 encoding): `&+AdsFX2&`
* UUIDs are surrounded by `=`, as in `=01234567-0123-4567-89ab-0123456789ab=`
  * A null UUID is represented by `==`
  * Base 64 formatting can be denoted by using a leading `+` (see Base 64 encoding): `=+ASNFZwEjRWeJqwAAASNFZw=`

#### Base 64 Encoding

Base 64 encoding should be performed consistent with 'base64url' defined in [RFC 4648](http://tools.ietf.org/html/rfc4648), and no padding should be used.

### Structured data

If it ain't broke, don't fix it.

#### Object graphs

* Object graphs are surrounded by `{` and `}`.
* Any text outside the delimiters used for primitive types and the three boolean symbols is considered to be a property name.
  * Property names may not contain the following characters: `$` `#` `/` `^` `!` `?` `&` `{` `}` `[` `]`
  * But they can contain spaces

Example:
````
{First name$Douglas$Last name$Adams$Date of birth/1952-03-11/}
````

#### Object graph schema

The shape of an object graph can be expressed using *Brian's Awesomely Descriptive And Short Schema for MOFO* (suggested by Brian Kardell):
````
{First name$Last name$Date of birth/}
````

To specify a boolean in a schema, use the `?` null signifier.

To indicate that a property may be of any type, use `*`.
#### Embedding BADASS

To include a BADASS declaration within a MOFO object, enclose it in parentheses `()`

Example:
````
{Class$Person$Schema({First name$Last name$Date of birth/Address{House$Street$City$Region$Postcode$Country$}})}
````

#### Lists

* Lists are surrounded by [ and ]
* Elements within lists are naturally delimited
  * Consecutive elements of the same type are split by a single instance of that type's delimiter (or in the case of objects and arrays, their pairs)

Examples:
* Numbers: `[#0#1#1#2#3#5#8#13#21#]`
* Strings: `[$Alice$Bob$Charlie$$Eve$]` (The `$$` represents a null element)
* Dates: `[/1970-01-01/1601-01-01/]`
* Booleans: `[^^^!!?^!]`
* Objects: `[{color$blue$}{color$red$}]`
* Arrays: `[[#4#2#][#6#$x$#7#]]`
* Mixed (string, string, number, boolean, date, null string): `[$Alice$Bob$#42#^/1752-09-14/$$]`

### Whitespace

Whitespace outside the `$` string delimiters and not within a property name is ignored.

### Summary

This JSON:

````
{
  "Name": "Phoenix",
  "Chassis": "Titan V",
  "Drive": "Warp",
  "First launch": "5th April 2063",
  "Thumbnail": ???,
  "Real": false,
  "Crew": [ "Cochrane", "Riker", "La Forge" ]
}
````
  
Is equivalent to this MOFO:

````
{Name$Phoenix$Chassis$Titan V$Drive$Warp$First launch/2063-04-05/Real!Thumbnail&94a2f19094213a6f8241a9408266f957&Crew[$Cochrane$Riker$La Forge$]}
````

Which, since whitespace is insignificant, could also be formatted thusly:

````
{
  Name $Phoenix$ 
  Chassis $Titan V$
  Drive $Warp$
  First launch /2063-04-05/
  Real !
  Thumbnail &94a2f19094213a6f8241a9408266f957&
  Crew [$Cochrane$Riker$La Forge$]
  ©© 
     And it can have comments...
     Comments: Do you speak it MOFO?
     Yes, yes I do.
  ©©
}
````

BADASS/MOFO Schema:

````
{Name$Chassis$Drive$First launch/Real?Thumbnail&Crew[$]}
````

#### More Badassery...

If basic BADASS just isn't enough, there is additional "Extended Region Yields" syntax which might be useful for code generators, basic data validation descriptions for explaining your service, generating empty records 
and other such non-sense...  

BADASSERY, if it is applied, is done so via trailing `<` annotations `>`.  

Annotations are generally range expressions in which either side may be unspecified, type clarity or enumerations:  

 * For numbers this can include:
   * the type - as in `{calories#<u>}` which means that I may take in from 0 up to an unsigned long's worth of calories.
   * a range as in in which either number can be unbound and the first may include a type... `{age<21:100>}` means a value between 21 and 100 (drinking age in the US). 
 * For strings, this can contain:
   * a ranged length - as in  `firstName$<1:32>` (a requied string less than 32 characters)
   * an enumeration of comma separated values, as in `MOFO$<Merely Awesome,The Awesomest>`
 * For dates this can include a range of short dates, as in `birthday/<:2000-1-1>`  (any date before Jan 1, 2000).
 * For arrays this indicates specified size. `{top picks[$]<0:10>}` signals an array of strings containing up to 10 elements.
 * For objects you can express basic variances by simply comma separating descriptions `{favorites[{}<{food$picture&},{song$band$},{movie$actors[$]}>]}`
 * For binary this is the size in MB, it is generally only useful in describing its max size, for example, 500k: {picture&<0:0.5>} 

Any type annotation indicates that a value is not nullable, unless ONLY the the left value is unbounded.  Thus, `{age#}` is nullable, as is `{age#<:99>}`, but `{age#<0:99>}` is not.


**Best. Wire format. Ever.**

