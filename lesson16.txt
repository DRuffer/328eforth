( Example 16.  Random Numbers )

\ Random numbers are often used in computer simulations and computer
\ games.  This random number generator was published in Leo Brodie's
\ 'Starting Forth'.

DECIMAL
VARIABLE RND                            ( seed )

: RANDOM ( -- n, a random number within 0 to 65536 )
        RND @ 31421 *                   ( RND*31421 )
        6927 +                          ( RND*31421+6926, mod 65536)
        DUP RND !                       ( refresh he seed )
        ;

: CHOOSE ( n1 -- n2, a random number within 0 to n1 )
        RANDOM UM*                      ( n1*random to a double product)
        SWAP DROP                       ( discard lower part )
        ;                                ( in fact divide by 65536 )

\ To test the routine, type

\        100 CHOOSE .
\        100 CHOOSE .
\        100 CHOOSE .

\ and varify that the results are randomly distributed betweem 0 and
\ 99 . 

flush

HERE RND !                              ( initialize seed )



