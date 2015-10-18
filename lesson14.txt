( Example 14.      Radix for Number Conversions )

DECIMAL

( : DECIMAL       10 BASE ! ; )
( : HEX           16 BASE ! ; )
: OCTAL         8 BASE !  ;
: BINARY        2 BASE !  ;

\ Try converting numbers among different radices:

\        DECIMAL 12345 HEX U.
\        HEX ABCD DECIMAL U.
\        DECIMAL 100 BINARY U.
\        BINARY 101010101010 DECIMAL U.

\ Real programmers impress on novices by carrying a HP calculator
\ which can convert numbers between decimal and hexadecimal.  A
\ Forth computer has this calculator built in, besides other functions.

flush
