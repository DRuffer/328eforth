( Example 3.      FIG, Forth Interest Group )

: center CR ."   *  " ;
: sides  CR ." *   *" ;
: triad1 CR ." * * *" ;
: triad2 CR ." **  *" ;
: triad3 CR ." *  **" ;
: triad4 CR ."  *** " ;
: quart  CR ." ** **" ;
: right  CR ." * ***" ;
: bigT  bar center center center center center center ;
: bigI  center center center center center center center ;
: bigN  sides triad2 triad2 triad1 triad3 triad2 sides ;
: bigG  triad4 sides post right triad1 sides triad4 ;
: FIG  F bigI bigG ;

flush
