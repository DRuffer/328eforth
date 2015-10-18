( Example 7.  Money Exchange )



\ The first example we will use to demonstrate how numbers are
\ used in Forth is a money exchange program, which converts money
\ represented in different currencies.  Let's start with the
\ following currency exchange table:

\        30.55 NT        1 Dollar
\        7.73 HK         1 Dollar
\        6.47 RMB        1 Dollar
\        1 Ounce Gold    1285 Dollars
\        1 Ounce Silver  14.95 Dollars 

DECIMAL

: NT    ( nNT -- $ )    100 3055 */  ;
: $NT   ( $ -- nNT )    3055 100 */  ;
: RMB   ( nRMB -- $ )   100 647 */  ;
: $RMB  ( $ -- nJmp )   647 100 */  ;
: HK    ( nHK -- $ )    100 773 */  ;
: $HK   ( $ -- $ )      773 100 */  ;
: GOLD  ( nOunce -- $ ) 1285 *  ;
: $GOLD ( $ -- nOunce ) 1285 /  ;
: SILVER ( nOunce -- $ ) 1495 100 */  ;
: $SILVER ( $ -- nOunce ) 100 1495 */  ;
: OUNCE ( n -- n, a word to improve syntax )  ;
: DOLLARS ( n -- )      . ;

\ With this set of money exchange words, we can do some tests:

\        5 ounce gold .
\        10 ounce silver .
\        100 $NT .
\        20 $RMB .

\ If you have many different currency bills in your wallet, you
\ can add then all up in dollars:

\        1000 NT 500 HK + .S
\        320 RMB + .S
\         DOLLARS ( print out total worth in dollars 

flush
