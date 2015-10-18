( Example 17.      Guess a Number )

( Require CHOOSE from Lesson16.txt )

DECIMAL

: GetNumber ( -- n )
        BEGIN
                CR ." Enter a Number: " ( show message )
                QUERY BL WORD NUMBER?   ( get a string )
        UNTIL                           ( repeat until a valid number )
        ;

( With this utility instruction, we can write a game 'Guess a Number.' )

: InitialNumber ( -- n , set up a number for the player to guess )
        CR CR CR ." What limit do you want?"
        GetNumber                       ( ask the user to enter a number )
        CR ." I have a number between 0 and " DUP .
        CR ." Now you try to guess what it is."
        CR
        CHOOSE                          ( choose a random number )
        ;                               ( between 0 and limit )

: Check ( n1 -- , allow player to guess, exit when the guess is correct )
        BEGIN   CR ." Please enter your guess."
                GetNumber
                2DUP =                  ( equal? )
                IF      2DROP           ( discard both numbers )
                        CR ." Correct!!!"
                        EXIT
                THEN
                OVER <
                IF      CR ." Too low."
                ELSE    CR ." Too high!"
                THEN    CR
        0 UNTIL                         ( always repeat )
        ;

: Greet ( -- )
        CR CR CR ." GUESS A NUMBER"
        CR ." This is a number guessing game.  I'll think"
        CR ." of a number between 0 and any limit you want."
        CR ." (It should be smaller than 32000.)"
        CR ." Then you have to guess what it is."
        ;

: GUESS ( -- , the game )
        Greet
        BEGIN   InitialNumber                   ( set initial number)
                Check                           ( let player guess )
                CR CR ." Do you want to play again? (Y/N) "
                KEY                             ( get one key )
                32 OR 110 =                     ( exit if it is N or n )
        UNTIL
        CR CR ." Thank you.  Have a good day."  ( sign off )
        CR
        ;


\ Type 'GUESS' will initialize the game and the computer will entertain
\ a user for a while.  Note the use of the indefinite loop structure:

\        BEGIN <repeat-clause> [ f ] UNTIL

\ You can jump out of the infinite loop by the instruction EXIT, which
\ skips all the instructions in a Forth definition up to ';', which
\ terminates this definition and continues to the next definition. )

flush
