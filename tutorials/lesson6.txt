( Example 6.      Help )
( How to use Forth interpreter to carry on a dialog )

: question
        CR CR ." Any more problems you want to solve?"
        CR ." What kind ( sex, job, money, health ) ?"
        CR
        ;

: help  CR
        CR ." Hello!  My name is Creating Computer."
        CR ." Hi there!"
        CR ." Are you enjoying yourself here?"
        KEY 32 OR 89 =
        CR
        IF      CR ." I am glad to hear that."
        ELSE    CR ." I am sorry about that."
                CR ." maybe we can brighten your visit a bit."
        THEN
        CR ." Say!"
        CR ." I can solved all kinds of problems except those dealing"
        CR ." with Greece. "
        question
        ;

: sex   CR CR ." Is your problem TOO MUCH or TOO LITTLE?"
        CR
        ;

: too  ;                                ( noop for syntax smoothness )

: much  CR CR ." You call that a problem?!!  I SHOULD have that problem."
        CR ." If it reall y bothers you, take a cold shower."
        question
        ;

: little
        CR CR ." Why are you here!"
        CR ." You should be in Tokyo or New York of Amsterdam or"
        CR ." some place with some action."
        question
        ;

: health
        CR CR ." My advise to you is:"
        CR ."      1. Take two tablets of aspirin."
        CR ."      2. Drink plenty of fluids."
        CR ."      3. Go to bed (along) ."
        question
        ;

: job   CR CR ." I can sympathize with you."
        CR ." I have to work very long every day with no pay."
        CR ." My advise to you, is to open a rental computer store."
        question
        ;

: money
        CR CR ." Sorry!  I am broke too."
        CR ." Why don't you sell encyclopedias of marry"
        CR ." someone rich or stop eating, so you won't "
        CR ." need so much money?"
        question
        ;

: HELP help ;
: H help ;
: h help ;

( Type 'help' to start )

flush
