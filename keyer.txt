\ KEYER.txt   input of iambic key              WFR 2011-01-27 
\ Modified for 328eForth, 23mar11cht

\ must have io-core.txt and tone.txt installed 

chop-keyer   

decimal

marker chop-keyer  ( a forget point)

$23 value PortB   0 value dah-in  1 value dit-in

: setup-key   \  ---  set inputs with pullups
   PortB  dah-in PoBiInPu
   PortB  dit-in PoBiInPu    ;

1 value Last-in  \ dit or dah last sent

: last?  \ n -- n   convert both into last sent inverted
  dup 3 = if drop Last-in  3 xor then
  dup to Last-in ( save for next time)  ; 

: sense-key ( ---  bits true | false   
   PortB 3 RegFrom  3 xor \  read & invert two low bits
   dup  if last? true else drop false then ;

10 value WPM  \ words per minute

: element-delay  1200 * WPM / ms ;

: dit tone-on 1 element-delay tone-off 1 element-delay ;

: dah tone-on 3 element-delay tone-off 1 element-delay ;

: next-character 2 element-delay ;  \ 3 total

: next-word      6 element-delay ;  \ 7 total

: V   dit dit dit dah next-character ;

: demo  \ iambic keyer
    setup-key
    begin 
      sense-key 
        if  1 and if dah else dit then then
    ?key until ;

: sos   dit dit dit next-character dah dah dah next-character
        dit dit dit next-word ;     

flush
