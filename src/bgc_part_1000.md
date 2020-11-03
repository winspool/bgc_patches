<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Types II

We're used to `char`, `int`, and `float` types, but it's now time to
take that stuff to the next level and see what else we have out there in
the types department!

## Signed and Unsigned Integers

So far we've used `int` as a _signed_ type, that is, a value that can be
either negative or positive. But C also has specific _unsigned_ integer
types that can only hold positive numbers.

These types are prefaced by the keyword `unsigned`.

``` {.c}
int a;           // signed
signed int a;    // signed
signed a;        // signed, "shorthand" for "int" or "signed int", rare
unsigned int b;  // unsigned
unsigned c;      // unsigned, shorthand for "unsigned int"
```

Why? Why would you decide you only wanted to hold positive numbers?

Answer: you can get larger numbers in an unsigned variable than you can
in a signed ones.

But why is that?

You can think of integers being represented by a certain number of
_bits_^["Bit" is short for _binary digit_. Binary is just another way of
representing numbers. Instead of digits 0-9 like we're used to, it's
digits 0-1.]. On my computer, an `int` is represented by 64 bits.

And each permutation of bits that are either `1` or `0` represents a
number. We can decide how to divvy up these numbers.

With signed numbers, we use (roughly) half the permutations to represent
negative numbers, and the other half to represent positive numbers.

With unsigned, we use _all_ the permutations to represent positive
numbers.

On my computer with 64-bit `int`s using [flw[two's
complement|Two%27s_complement]] to represent unsigned numbers, I have
the following limits on integer range:

|Type|Minimum|Maximum|
|:-|-:|-:|
|`int`|`-9,223,372,036,854,775,808`|`9,223,372,036,854,775,807`|
|`unsigned int`|`0`|`18,446,744,073,709,551,615`|

Notice that the largest positive `unsigned int` is approximately twice
as large as the largest positive `int`. So you can get some flexibility
there.

## Character Types

Remember `char`? The type we can use to hold a single character?

``` {.c}
char c = 'B';

printf("%c\n", c);  // "B"
```

I have a shocker for you: it's actually an integer.

``` {.c}
char c = 'B';

// Change this from %c to %d:
printf("%d\n", c);  // 66 (!!)
```

Deep down, `char` is just a small `int`, namely an integer that uses
just a single byte of space, limiting its range to...

Here the C spec gets just a little funky. It assures us that a `char` is
a single byte, i.e. `sizeof(char) == 1`. But then in C99 §3.6¶3 it goes
out of its way to say:

> A byte is composed of a contiguous sequence of bits, _the number of
> which is implementation-defined._

Wait---what? Some of you might be used to the notion that a byte is 8
bits, right? I mean, that's what it is, right? And the answer is,
"Almost certainly."^[The industry term for a sequence of exactly,
indisputably 8 bits is an _octet_.] But C is an old language, and
machines back in the day had, shall we say, a more _relaxed_ opinion
over how many bits were in a byte. And through the years, C has retained
this flexibility.

But assuming your bytes in C are 8 bits, like they are for virtually all
machines in the world that you'll ever see, the range of a `char` is...

---So before I can tell you, it turns out that `char`s might be signed
or unsigned depending on your compiler. Unless you explicitly specify.

In many cases, just having `char` is fine because you don't care about
the sign of the data. But if you need signed or unsigned `char`s, you
_must_ be specific:

``` {.c}
char a;           // Could be signed or unsigned
signed char ab    // Definitely signed
unsigned char c;  // Definitely unsigned
```

OK, now, finally, we can figure out the range of numbers if we assume
that a `char` is 8 bits and your system uses the virtually universal
two's complement representation for signed and unsigned^[In general, f
you have an $n$ bit two's complement number, the signed range is
$-2^{n-1}$ to $2^{n-1}-1$. And the unsigned range is $0$ to $2^{n-1}$.].

So, assuming those constraints, we can finally figure our ranges:

|`char` type|Minimum|Maximum|
|:-|-:|-:|
|`signed char`|`-128`|`127`|
|`unsigned char`|`0`|`255`|

And the ranges for `char` are implementation-defined.

Let me get this straight. `char` is actually a number, so can we do math
on it?

Yup! Just remember to keep things in the range of a `char`!

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char a = 10, b = 20;

    printf("%d\n", a + b);  // 30!

    return 0;
}
```

What about those constant characters in single quotes, like `'B'`? How
does that have a numeric value?

The spec is also hand-wavey here, since C isn't designed to run on a
single type of underlying system.

But let's just assume for the moment that your character set is based on
[flw[ASCII|ASCII]] for at least the first 128 characters. In that case,
the character constant will be converted to a `char` whose value is the
same as the ASCII value of the character.

That was a mouthful. Let's just have an example:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char a = 10;
    char b = 'B';  // ASCII value 66

    printf("%d\n", a + b);  // 76!

    return 0;
}
```

This depends on your execution environment and the [flw[character set
used|List_of_information_system_character_sets]]. One of the most
popular character sets today is [flw[Unicode|Unicode]] (which is a
superset of ASCII), so for your basic 0-9, A-Z, a-z and punctuation,
you'll almost certainly get the ASCII values out of them.

## More Integer Types: `short`, `long`, `long long`

So far we've just generally been using two integer types:

* `char`
* `int`

and we recently learned about the unsigned variants of the integer
types. And we learned that `char` was secretly a small `int` in
disguise. So we know the `int`s can come in multiple bit sizes.

But there are a couple more integer types we should look at, and the
_minimum_ minimum and maximum values they can hold.

Yes, I said "minimum" twice. The spec says that these types will hold
numbers of _at least_ these sizes, so your implementation might be
different. The header file `<limits.h>` defines macros that hold the
minimum and maximum integer values; rely on that to be sure, and _never
hardcode or assume these values_.

|Type|Minimum Bits|Minimum Bytes|Minimum Value|Maximum Value|
|:-|-:|-:|-:|-:|
|`char`|8|1|-127 or 0|127 or 255^[Depends on if a `char` defaults to `signed char` or `unsigned char`]|
|`signed char`|8|1|-127|127|
|`short`|16|2|-32767|32767|
|`int`|16|2|-32767|32767|
|`long`|32|4|-2147483647|2147483647|
|`long long`|64|8|-9223372036854775807|9223372036854775807|
|`unsigned char`|8|1|0|255|
|`unsigned short`|16|2|0|65535|
|`unsigned int`|16|2|0|65535|
|`unsigned long`|32|0|44294967295|
|`unsigned long long`|64|8|0|9223372036854775807|

There is no `long long long` type. You can't just keep adding `long`s
like that. Don't be silly.

> Two's complement fans might have noticed something funny about those
> numbers. Why does, for example, the `signed char` stop at -127 instead
> of -128? Remember: these are only the minimums required by the spec.
> Some number representations (like [flw[sign and
> magnitude|Signed_number_representations#Signed_magnitude_representation]])
> top off at ±127.

Let's run the same table on my 64-bit, two's complement system and see
what comes out:

|Type|My Bits|My Bytes|Minimum Value|Maximum Value|
|:-|-:|-:|-:|-:|
|`char`|8|1|-128|127^[My `char` is signed.]|
|`signed char`|8|1|-128|127|
|`short`|16|2|-32768|32767|
|`int`|32|4|-2147483648|2147483647|
|`long`|64|8|-9223372036854775808|9223372036854775807|
|`long long`|64|8|-9223372036854775808|9223372036854775807|
|`unsigned char`|8|1|0|255|
|`unsigned short`|16|2|0|65535|
|`unsigned int`|32|4|0|4294967295|
|`unsigned long`|64|8|0|18446744073709551615|
|`unsigned long long`|64|8|0|18446744073709551615|

That's a little more sensible, but we can see how my system has larger
limits than the minimums in the specification.

So what are the macros in `<limits.h>`?

|Type|Min Macro|Max Macro|
|:-|:-|:-|
|`char`|`CHAR_MIN`|`CHAR_MAX`|
|`signed char`|`SCHAR_MIN`|`SCHAR_MAX`|
|`short`|`SHRT_MIN`|`SHRT_MAX`|
|`int`|`INT_MIN`|`INT_MAX`|
|`long`|`LONG_MIN`|`LONG_MAX`|
|`long long`|`LLONG_MIN`|`LLONG_MAX`|
|`unsigned char`|`0`|`UCHAR_MAX`|
|`unsigned short`|`0`|`USHRT_MAX`|
|`unsigned int`|`0`|`UINT_MAX`|
|`unsigned long`|`0`|`ULONG_MAX`|
|`unsigned long long`|`0`|`ULLONG_MAX`|

Notice there's a way hidden in there to determine if a system uses
signed or unsigned `char`s. If `CHAR_MAX == UCHAR_MAX`, it must be
unsigned.

Also notice there's no minimum macro for the `unsigned`
variants---they're just `0`.