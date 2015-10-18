( Example 9.  Weather Reporting. )

\ IF-ELSE-THEN structure can be nested.

: WEATHER ( nFarenheit -- )
        DUP     55 <
        IF      ." Too cold!" DROP
        ELSE    85 <
                IF      ." About right."
                ELSE    ." Too hot!"
                THEN
        THEN
        ;

\ You can type the following instructions and get some responses from the
\ computer:

\        90 WEATHER Too hot!
\        70 WEATHER About right.
\        32 WEATHER Too cold.

flush 
