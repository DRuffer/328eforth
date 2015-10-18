\  Marker.txt                                      WFR 2010-10-22
\ Defines a word which resets the dictionary when executed. 
\ Due to three memory spaces the traditional FORGET is impractial.
\ Modified for 328eForth, 23mar11cht

\ Extend eForth first, 22mar11cht
: DOES R> LAST @ NAME> 2+ I! ; 
\ specific for subroutine thread code.  see VALUE and MARKER.

: VALUE ( n --, interpret ) VARIABLE DP @ 2- ! DOES R> 2* I@ @ ;
\ compile a pointer in flash, access data in RAM

: [TO] ( n --, interpret ) ' 4 + I@ ! ; 
\ change value while interpreting

: to ( n --, compile ) R> DUP   1+ 2* I@   2+ 2* I@   SWAP 2+ >R   ! ;
\ compiled in colon word to change value at run time

: TRUE ( -- -1 ) FFFF ;
: FALSE ( -- 0 ) 0 ;
: LSHIFT ( n --, logic left shift ) FOR AFT 2* THEN NEXT ;
: RSHIFT ( n --, logic right shift ) FOR AFT 2/ THEN NEXT ;
: ms ( n -- ) FOR AFT $1CB FOR NEXT THEN NEXT ;
: ud/mod ( ud1 n -- rem ud2 ) >R 0 R@ UM/MOD R> SWAP >R UM/MOD R> ; 
: bin ( -- ) 2 base ! ;

\ marker is easier for 328eForth with only one vocabulary, 23mar11cht
: marker ( <new_name> -- )
    last @ dp @ cp @ context @
    create -2 cp +! , , , ,
    does r> 2* dup i@ context !
	2+ dup i@ cp !
	2+ dup i@ dp !
	2+ i@ last ! ;

flush

marker fence

