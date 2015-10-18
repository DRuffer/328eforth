\  Audio tone generator                      2010/01/08
\ Modified for 328eForth, 23mar11cht

\ Must have io-core.txt installed

chop-tone
marker chop-tone  \ a forget point

$44 value TCCR0A  \ Timer Counter Control Register A
\ mask  1100 0011
\ set   0100 0010  toggle output, generator mode [14-2, 14-8]

$45 value TCCR0B  \ Timer Control Control Register B
\ mask  0000 1111
\ set   0000 0xxx   WGM01 bit2, xxx is prescaler. [14-8]
\  0= counter off,  1= /1,  2=/8,  3=/64,  4=/256,  5=/1024

$47 value OCR0A  \ Output Comparison Register A
\ mask 1111 1111
\ set  <limit>   limit is 1..255 for counter. [14-9]

$29 value PortD     6 value Tone-out \ PortD bit 6
\ mask 0100 0000
\ set  0100 0000 or true  initialize pin as output

: setup-osc \ prescale limit  ---  limit 1..255, prescale 1..5
   PortD   Tone-out                  PoBiOut \ setup output pin    
   OCR0A   true      rot ( limit   ) RegTo     [ bin ]
   TCCR0A  11000011  01000010        RegTo   \ CTC mode
           00000101  min    0  max    \ form TCCR0B prescale
   TCCR0B  00001111 rot ( prescale ) RegTo ;  \ and tone on  

: tone-off \  ---  end output tone setting prescale to zeros
   TCCR0B  00000111  false RegTo   ;  decimal

78 value Limit   4 value Prescale  \ 400 Hz tone parameters

: Hertz   \ frequency --- load Limit and Prescale 
   $1200 $7A ( 8000000. ) rot ud/mod  \ total scale as rem double-quot
   dup            if ( >16 bits)  1024  5 else 
   over $C000 and if ( >14 bits)   256  4 else 
   over $F800 and if ( >10 bits)    64  3 else 
   over $FF00 and if ( > 8 bits)     8  2 else   1  1  
                     then then then then  
   to Prescale   um/mod to Limit  drop drop ( two remainders ) ;

: tone-on   \ ---  begin tone from fixed presets
    Prescale  Limit  setup-osc ;

: note  \ duration ---    generate timed tone for duration msec.
    tone-on  ms  tone-off   ;

\ End of tone.txt
flush
