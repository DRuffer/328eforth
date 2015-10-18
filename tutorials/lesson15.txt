( Example 15.      ASCII Character Table )

DECIMAL

: CHARACTER ( n -- )
        DUP EMIT HEX DUP 3 .R
        OCTAL DUP 4 .R
        DECIMAL 3 .R
        2 SPACES
        ;

: LINE ( n -- )
        CR
        5 FOR   DUP CHARACTER
                16 +
        NEXT
        DROP ;

: TABLE ( -- )
        32
        15 FOR  DUP LINE
                1 +
        NEXT
        DROP ;

flush

