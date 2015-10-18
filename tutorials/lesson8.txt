( Example 8. Temperature Conversion )

\ Converting temperature readings between Celcius and Farenheit
\ is also an interesting problem.  The difference between temperature
\ conversion and money exchange is that the two temperature scales
\ have an offset besides the scaling factor. 

\ In the following examples, we use these Forth arithmatic operators:

\ +       [ n1 n2 -- n1+n2 ]      Add n1 and n2 and leave sum on stack.
\ -       [ n1 n2 -- n1-n2 ]      Subtract n2 from n1 and leave differrence
\                                 on stack.
\ *       [ n1 n2 -- n1*n2 ]      Multiply n1 and n2 and leave product
\                                 on stack.
\ /       [ n1 n2 -- n1/n2 ]      Divide n1 by n2 and leave quotient on
\                                 stack.
\ */      [ n1 n2 n3 -- n1*n2/n3] Multiply n1 and n2, divide the product
\                                 by n3 and leave quotient on the stack.

: F>C ( nFarenheit -- nCelcius )
        32 -
        10 18 */
        ;

: C>F ( nCelcius -- nFarenheit )
        18 10 */
        32 +
        ;

\ Try these commands 

\ 90 F>C .        shows the temperature in a hot summer day and
\ 0 C>F .         shows the temperature in a cold winter night. 

flush
