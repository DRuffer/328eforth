( Example 12.      Sines and Cosines )

\ Sines and cosines of angles are among the most often encountered
\ transdential functions, useful in drawing circles and many other
\ different applications.  They are usually computed using floating
\ numbers for accuracy and dynamic range.  However, for graphics
\ applications in digital systems, single integers in the range from
\ -32768 to 32767 are sufficient for most purposes.  We shall
\ study the computation of sines and cosines using the single
\ integers.

\ The value of sine or cosine of an angle lies between -1.0 and +1.0.
\ We choose to use the integer 10000 in decimal to represent 1.0
\ in the computation so that the sines and cosines can be represented
\ with enough precision for most applications.  Pi is therefore
\ 31416, and 90 degree angle is represented by 15708.  Angles
\ are first reduced in to the range from -90 to +90 degrees,
\ and then converted to radians in the ranges from -15708 to
\ +15708.  From the radians we compute the values of sine and
\ cosine.

\ The sines and cosines thus computed are accurate to 1 part in
\ 10000.  This algorithm was first published by John Bumgarner
\ in Forth Dimensions, Volume IV, No. 1, p. 7.

DECIMAL
31415 CONSTANT PI
10000 CONSTANT 10K 

VARIABLE XS                             ( square of scaled angle )

: KN ( n1 n2 -- n3, n3=10000-n1*x*x/n2 where x is the angle )
        XS @ SWAP /                     ( x*x/n2 )
        NEGATE 10K */                   ( -n1*x*x/n2 )
        10K +                           ( 10000-n1*x*x/n2 )
        ;

: (SIN) ( x -- sine*10K, x in radian*10K )
        DUP DUP 10K */                  ( x*x scaled by 10K )
        XS !                            ( save it in XS )
        10K 72 KN                       ( last term )
        42 KN 20 KN 6 KN                ( terms 3, 2, and 1 )
        10K */                          ( times x )
        ;

: (COS) ( x -- cosine*10K, x in radian*10K )
        DUP 10K */ XS !                 ( compute and save x*x )
        10K 56 KN 30 KN 12 KN 2 KN      ( serial expansion )
        ;

: SIN ( degree -- sine*10K )
        PI 180 */                       ( convert to radian )
        (SIN)                           ( compute sine )
        ;

: COS ( degree -- cosine*10K )
        PI 180 */
        (COS)
        ;

\ To test the routines, type:

\        90 SIN .                        10000 
\        45 SIN .                         7070 
\        30 SIN .                         4999 
\         0 SIN .                            0 
\        90 COS .                            0 
\        45 COS .                         7072 
\         0 COS .                         10000 

flush
