( Example 5.      The Theory That Jack Built )
( This example shows you how to build a hiararchical structure in Forth)

DECIMAL

: the           ." the " ;
: that          CR ." That " ;
: this          CR ." This is " the ;
: jack          ." Jack Builds" ;
: summary       ." Summary" ;
: flaw          ." Flaw" ;
: mummery       ." Mummery" ;
: k             ." Constant K" ;
: haze          ." Krudite Verbal Haze" ;
: phrase        ." Turn of a Plausible Phrase" ;
: bluff         ." Chaotic Confusion and Bluff" ;
: stuff         ." Cybernatics and Stuff" ;
: theory        ." Theory " jack ;
: button        ." Button to Start the Machine" ;
: child         ." Space Child with Brow Serene" ;
: cybernatics   ." Cybernatics and Stuff" ;

: hiding        CR ." Hiding " the flaw ;
: lay           that ." Lay in " the theory ;
: based         CR ." Based on " the mummery ;
: saved         that ." Saved " the summary ;
: cloak         CR ." Cloaking " k ;
: thick         IF that ELSE CR ." And " THEN
                ." Thickened " the haze ;
: hung          that ." Hung on " the phrase ;
: cover         IF that ." Covered "
                ELSE CR ." To Cover "
                THEN bluff ;
: make          CR ." To Make with " the cybernatics ;
: pushed        CR ." Who Pushed " the button ;
: without       CR ." Without Confusion, Exposing the Bluff" ;

: rest                                  ( pause for user interaction )
        ." . "                          ( print a period )
        10 SPACES                       ( followed by 10 spaces )
        KEY                             ( wait the user to press a key )
        DROP CR CR CR ;

\ Here are new commands needed
\ KEY     [ -- char ]             Wait for a keystroke, and return the
\                                 ASCII code of the key pressed.
\ DROP    [ n -- ]                Discard the number.
\ SPACE   [ -- ]                  Display a blank.
\ SPACES  [ n -- ]                Display n blanks.
\ IF      [ f -- ]                If the flag is 0, skip the following
\                                 instructions up to ELSE or THEN.  If
\                                 flag is not 0, execute the following
\                                 instructions up to ELSE and skip to
\                                 THEN.
\ ELSE    [ -- ]                  Skip the following instructions
\                                 up to THEN.
\ THEN    [ -- ]                  Terminate an IF-ELSE-THEN structure
\                                 or an IF-THEN structure.

: cloaked cloak saved based hiding lay rest ;

: THEORY
        CR this theory rest
        this flaw lay rest
        this mummery hiding lay rest
        this summary based hiding lay rest
        this k saved based hiding lay rest
        this haze cloaked
        this bluff hung 1 thick cloaked
        this stuff 1 cover hung 0 thick cloaked
        this button make 0 cover hung 0 thick cloaked
        this child pushed
                CR ." That Made with " cybernatics without hung
                CR ." And, Shredding " the haze cloak
                CR ." Wrecked " the summary based hiding
                CR ." And Demolished " the theory rest
        ;

( Type THEORY to start)

flush