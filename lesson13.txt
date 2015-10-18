( Example 13.      Square Root )

\ There are many ways to take the square root of an integer.  The
\ special routine here was first discovered by Wil Baden.  Wil
\ used this routine as a programming challenge while attending
\ a FORML Conference in Taiwan, 1984.

\ This algorithm is based on the fact that the square of n+1 is equal
\ to the sum of the square of n plus 2n+1.  You start with an 0 on
\ the stack and add to it 1, 3, 5, 7, etc., until the sum is greater
\ than the integer you wished to take the root.  That number when
\ you stopped is the square root.

: SQRT ( n -- root )
        65025 OVER U<                   ( largest square it can handle)
        IF DROP 255 EXIT THEN           ( safety exit )
        >R                              ( save sqaure )
        1 1                             ( initial square and root )
        BEGIN                           ( set n1 as the limit )
                OVER R@ U<              ( next square )
        WHILE
                DUP 2* 1+           	( n*n+2n+1 )
                ROT + SWAP
                1+                      ( n+1 )
        REPEAT
        SWAP DROP
        R> DROP
        ;

flush
