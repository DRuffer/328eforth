( Example 10.  Print the multiplication table )

( More examples on FOR-NEXT loops. )

DECIMAL

: ONEROW ( nRow -- )
        CR
        DUP 3 U.R 3 SPACES
        1 11
        FOR     2DUP *
                4 U.R
                1 +
        NEXT
        DROP ;

: MULTIPLY ( -- )
        CR CR 6 SPACES
        1 11
        FOR     DUP 4 U.R 1 +
        NEXT DROP 
        1 11
        FOR     DUP ONEROW 1 +
        NEXT DROP
        ;

( Type MULTIPLY to print the multiplication table )

flush
