( Example 4.      Repeated Patterns )

\ FOR     [ index -- ]          Set up loop given the index.
\ NEXT    [ -- ]                Decrement index by 1.  If index<0, exit.    
\                               If index=limit, exit loop; otherwise
\                               Otherwise repeat after FOR.
\ R@      [ -- index ]          Return the current loop index. 

DECIMAL

VARIABLE WIDTH                  ( number of asterisks to print )

: ASTERISKS ( -- , print n asterisks on the screen, n=width )
        WIDTH @                 ( limit=width, initial index=0 )
        FOR ." *"               ( print one asterisk at a time )
        NEXT                    ( repeat n times )
        ;

: RECTANGLE ( height width -- , print a rectangle of asterisks )
        WIDTH !                 ( initialize width to be printed )
        FOR     CR
                ASTERISKS       ( print a line of asterisks )
        NEXT
        ;

: PARALLELOGRAM ( height width -- )
        WIDTH !
        FOR     CR R@ SPACES    ( shift the lines to the right )
                ASTERISKS       ( print one line )
        NEXT
        ;

: TRIANGLE ( width -- , print a triangle area with asterisks )
        FOR     CR
                R@ WIDTH !      ( increase width every line )
                ASTERISKS       ( print one line )
        NEXT
        ;

\ Try the following instructions:

\        3 10 RECTANGLE
\        5 18 PARALLELOGRAM
\        12 TRIANGLE  

flush
