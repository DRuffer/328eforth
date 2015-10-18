\ Port Input Output for AmForth                  WFR 2011-01-27 
\ loaded as  io-core.txt
\ Modified for 328eForth, 23mar11cht

\ manually begin with chop-io entered

chop-io
marker chop-io  \ a forget point

: mask ( bit# --- port_mask  convert bit to 8 bit mask)
     1 swap lshift  ;

: DDR  \ port --- port' adjust input port# to DDR 
    1+ ;

: Output \ port --- port adjust input port# to output
    2 + ;

: RegFrom \ Reg mask --- value  read masked bits from register 
        \ To read all bits:  PortB true RegFrom -> value
     swap c@  and ;

: RegTo \ Reg mask new ---  write masked new into register
     over and >r   invert over c@ and   r> or  swap c! ;

: PoBiI/O  \ port bit direction --- configure bit in/out
    rot DDR   rot mask   rot RegTo ;

: PoBiOut  \ port bit --- configure as output
    true  PoBiI/O ;

: PoBiRead  \ port bit --- value  read bit value from port
     mask RegFrom ;

: PoBiHi  \ port bit --- set port bit 0..7 high
     swap Output swap mask true  RegTo ;

: PoBiLo   \ port bit --- clear port bit 0..7 low
     swap Output swap mask false RegTo ;

: PoBiIn   \ port bit --- configure as input,  no pull-up
    2dup false PoBiI/O   PoBiLo ( pullup inactive) ;

: PoBiInPu \ port bit --- configure as input with pull-up
    2dup false PoBiI/O   PoBiHi ( pullup active) ; 

\ read bits from register  Reg#         select      RegFrom
\ write bits to register   Reg#         select bits RegTo
\ write 1-bit to register  Reg#   5 mask true       RegTo
\ write 0-bit to register  Reg#   5 mask false      RegTo
\ configure bits as output PortB DDR    select True RegTo
\ write bits to output     PortB Output select bits RegTo
\ configure bit as output  PortB     LED            PoBiOut
\ bit as input with pullup PortB     Switch3        PoBiInPu
\ read bit from port       PortB     Switch3        PoBiRead
\ write 1-bit to port      PortB     LED            PoBiHi
\ write 0-bit to port      PortB     LED            PoBiLo
\ Note, when initializing a 16 bit register, TCNT1 etc. it
\  must be written directly hi/lo not using RegTo.
\  The proper form to clear is:    TCNT1hi false c!
\                         then:    TCNT1lo false c!
\                                  

\ end of io-core.txt
flush
