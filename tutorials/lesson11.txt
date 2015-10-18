( Example 11.  Calendars )

( Print monthly calendars for any month in years 1950-2128. )
DECIMAL

VARIABLE JULIAN                         ( 0 is 1/1/1950, good until 2050 )
VARIABLE LEAP                           ( 1 for a leap year, 0 otherwise. )
( 1461 CONSTANT 4YEARS                  ( number of days in 4 years )

: YEAR ( YEAR --, compute Julian date and leap year )
        DUP
        1949 - 1461 4 */MOD             ( days since 1/1/1949 )
        365 - JULIAN !                  ( 0 for 1/1/1950 )
        3 =                             ( modulus 3 for a leap year )
        IF 1 ELSE 0 THEN                ( leap year )
        LEAP !
        DUP 2000 =                      ( 2000 is not a leap year )
        IF 0 LEAP ! THEN
	2000 >				( adjust due to year 2000 )
	IF ELSE -1 JULIAN +! THEN
        ;

: FIRST ( MONTH -- 1ST, 1st of a month from Jan. 1 )
        DUP 1 =
        IF DROP 0 EXIT THEN             ( 0 for Jan. 1 )
        DUP 2 =
        IF DROP 31 EXIT THEN            ( 31 for Feb. 1 )
        DUP 3 =
        IF DROP 59 LEAP @ + EXIT THEN   ( 59/60 for Mar. 1 )
        4 - 30624 1000 */
        90 + LEAP @ +                   ( Apr. 1 to Dec. 1 )
        ;

: STARS 60 FOR 42 EMIT NEXT ;           ( form the boarder )

: HEADER ( -- )                         ( print title bar )
        CR STARS CR 
        ."      SUN     MON     TUE     WED     THU     FRI     SAT"
        CR STARS CR                     ( print weekdays )
        ;

: BLANKS ( MONTH -- )                   ( skip days not in this month )
        FIRST JULIAN @ +                ( Julian date of 1st of month )
        7 MOD 8 * SPACES ;              ( skip colums if not Sunday   )

: DAYS ( MONTH -- )                     ( print days in a month )
        DUP FIRST                       ( days of 1st this month )
        SWAP 1 + FIRST                  ( days of 1st next month )
        OVER - 1 -                      ( loop to print the days )
        1 SWAP                          ( first day count -- )
        FOR  2DUP + 1 -
                JULIAN @ + 7 MOD        ( which day in the week? )
                IF ELSE CR THEN         ( start a new line if Sunday )
                DUP  8 U.R              ( print day in 8 column field )
                1 +
        NEXT
        2DROP ;                         ( discard 1st day in this month )

: MONTH ( N -- )                        ( print a month calendar )
        HEADER DUP BLANKS               ( print header )
        DAYS CR STARS CR ;              ( print days   )

: JANUARY       YEAR 1 MONTH ;
: FEBRUARY      YEAR 2 MONTH ;
: MARCH         YEAR 3 MONTH ;
: APRIL         YEAR 4 MONTH ;
: MAY           YEAR 5 MONTH ;
: JUNE          YEAR 6 MONTH ;
: JULY          YEAR 7 MONTH ;
: AUGUST        YEAR 8 MONTH ;
: SEPTEMBER     YEAR 9 MONTH ;
: OCTOBER       YEAR 10 MONTH ;
: NOVEMBER      YEAR 11 MONTH ;
: DECEMBER      YEAR 12 MONTH ;

\ To print the calender of April 1999, type:
\        2011 APRIL

flush

