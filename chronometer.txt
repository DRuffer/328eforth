\  Chronometer                  WFR 2011-01-27
\ Modified for 328eForth, 23mar11cht

\ must have io-core.txt installed

chop-chrono
marker chop-chrono  \ this forget point

$36 value TIFR1    \ has overlow bit
$43 value GTCCR    \ counter control register
$80 value TCCR1A   \ timer control register, no outputs
$81 value TCCR1B   \ prescaler factor
$84 value TCNT1lo  \ counter low 8 bits
$85 value TCNT1hi  \ counter high 8 bits

0  value  prescale  \ 0= stop; 1..5 is prescale setup

bin

: chrono-hack  \  --- count scale overflow
\ read continuing count
  TCNT1lo  true       RegFrom      \ lo byte
  TCNT1hi  true       RegFrom      \ hi byte
  $8 lshift or                      \ form count into 16 bits
  prescale                         \ include in output
  TIFR1    00000001  RegFrom ;    \  overflow bit

: chrono-stop  \  --- count scale overflow
\ stop chronometer returning its values
  false  TCCR1B c!  \ stop counter via timer control register
  chrono-hack  ;    \ read chronometer values

: chrono-start  \  prescale ---   
\ initialize and start chronometer using Timer 1
  dup 101 > abort" Prescale too high"  to prescale
  GTCCR   10000001  true     RegTo  \ halt counter, clear prescale
  TCCR1A  11110011  false    RegTo  \ no outputs, normal mode
  TCCR1B  11011111  prescale RegTo  \ set prescaler
   false  TCNT1hi    c!     \ must write directly, clear counter hi
   false  TCNT1lo    c!     \ clear counter lo
  TIFR1   00000001  true     RegTo  \ clear overflow bit
  GTCCR   10000000  false    RegTo  \ start prescaler and counter
  ;  

decimal
 
: chrono-norm  \ count scale overflow --- usec_double overflow
               \ normalize into count and microsecond multiplier
               \ for 1 & 2 truncated not rounded up.
   >r >r r@   1 = if  4 rshift 1  else  \  1 usec ..  4.095 msec
         r@   2 = if  1 rshift 1  else  \  1 usec .. 32.767 msec
         r@   3 = if           4  else  \  4 usec ..  0.262 sec
         r@   4 = if          16  else  \ 16 usec ..  1.048 sec
                              64        \ 64 usec ..  4.194 sec
         then then then then   
         um*  r> drop r> ;  \ leave microseconds as a double number

: 1000/mod    \ double --- rem double/1000
   1000 ud/mod ;

: nnn ( n -- ) <# # # # #> TYPE ;
: nnn, ( n -- ) nnn ." ," ;
: n,n,n ( d -- )
 1000/mod 1000/mod DROP
 ?DUP IF . ." ," nnn, nnn ELSE
 ?DUP IF . ." ," nnn ELSE . THEN THEN ;

: chrono-report  \ count scale overflow --- 
\  from chronometer reading display time, observing the scale
   chrono-norm   abort" counter overflow"
   n,n,n ."  usec" ;

: chrono-count  \ count scale overflow ---  
\ from chronometer reading, report elapsed processor clock counts
\   with 484 count bias removed.
   abort" counter overflow"   >r  \ save scale
   r@ 1 = if  1 else  r@ 2 = if 8   else
   r@ 3 = if 64 else  r@ 4 = if 256 else 1024 then then then then
   r> drop   um*  22976 0 d- ( remove overhead cycles )
   n,n,n  ."  clock counts" ;

\ Usage is:    3 chrono-start . . . chrono-stop chrono-report
\ Null time is 484 clock cycles or 30.25 usec.
\ This is the time for chrono-start exit and do-colon of chrono-stop.
\ A test of 20 ms report gives  19.941 msec.
\         1000 ms report gives 995.136 msec.
\         4000 ms report gives   3.980 sec.
\ Adjusting for fixed offset time is .9955 of true time.

\ Read machine cycles: 1 chrono-start . . . chrono-stop chrono-count

\ end of chronometer.txt
flush



