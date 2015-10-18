\ DUMP utility to print memory                WFR 2011-01-03 
\ Begin by specifying target memory: flash, ram, eeprom
\ Call dump with starting address and byte count

\ dump                 WFR 2011-01-27
\ Modified for 328eForth, 23mar11cht

marker chop-dump

decimal

1 value target  ( 0=flash, 1=ram)  

: flash 0 to target ;

: ram   1 to target ;

: smartC@  ( addr --- cell-contents )
     target if C@ else iC@ then ;

: ?legal ( char --- printable_char from low 8 bits )
   255 and   32 over > over 127 > or   if drop $2E ( . ) then ; 

: .location ( --- print memory bank being accessed)
   target if ."  ram " else ."  flash " then ;

: .alpha ( addr count --- print 8 cells or 16 bytes as ascii characters)
   for aft dup smartC@ ?legal emit 1+ then next drop ;

: .memory ( addr count --- flash prints cells; r/r prints bytes)
   for aft dup smartC@ 3 u.r 1+ then next drop ;

: dump ( addr count ---  form: 1234 xx xx xx xx xx abcdefg)
   .location cr   base @ >r   hex  16 /
   for aft dup 5 u.r dup 16 .memory dup 16 .alpha cr
   16 + then next drop r> base !  ;

\ end of dump utility
flush

