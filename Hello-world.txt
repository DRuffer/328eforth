\ Hello, World                  WFR 2011-01-27
\ Modified for 328eForth, 23mar11cht

\ must have marker.txt installed

marker chop-hello

: hello cr ." Hello World!" ;

: test ." Testing " 1 . 2 . 3 . ."  Anyone out there?" ;

flush

hello

test

chop-hello
