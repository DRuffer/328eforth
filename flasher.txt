\ FLASHER.txt to Demo LED control              WFR 2011-01-27 

( must have io-core.txt installed )

chop-flasher

marker chop-flasher  ( a forget point)

$23 value PortB  $26 value PortC  $29 value PortD
  5 value LED

: 1-cycle  ( ms_delay ---   flash LED on then off )
    PortB LED PoBiHi   dup ms   PortB LED PoBiLo   ms ;

: many  ( on_time flashes ---  produce controlled LED flashes)
    PortB LED PoBiOut ( set LED pin as output)
    for aft  dup 1-cycle then next drop ;

( use 'many' leading with on-time and # of flashes )

( end of flasher.txt )
flush

