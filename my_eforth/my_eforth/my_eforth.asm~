; TITLE Atmega328 eForth

.nolist
	.include "m328Pdef.inc"
.list

;===============================================================
;	328eForth v2.20, Chen-Hanson Ting, July 2011
;		Fix error, quit, 2/ and ?stack
;
;	328eForth v2.10, Chen-Hanson Ting, March 2011
;	Adapted from 
;		86se4th.asm by Richard Haskell 
;		Amforth by Matthias Trute
;	Assembled with AVR Studio 4 from Atmel
;	-Subroutine threaded model
;	-Uniform byte addressing for flash, RAM and registers
;	-Ping-pong block buffers for optimal flash programming
;	-FORTH interpreter & tools are in NRWW flash
;	-FORTH compiler & user extension are in RWW flash
;	-No interrupt, no multitasking 
;	-turnkey capability
;	-Case insensitive
;	-9600 baud, 1 start, 8 data, no parity, 1 stop bit
;	ANS FORTH compatible, but not compliant.
;
;	Subroutine threaded eForth; Version. 1.0, 1991 
;	by Richard E. Haskell
;	Dept. of Computer Science and Engineering
;	Oakland University
;	Rochester, Michigan 48309
;
;	eForth 1.0 by Bill Muench and C. H. Ting, 1990
;	Much of the code is derived from the following sources:
;	8086 figForth by Thomas Newman, 1981 and Joe smith, 1983
;	aFORTH by John Rible
;	bFORTH by Bill Muench
;
;	The goal of this implementation is to provide a simple eForth Model
;	which can be ported easily to many 8, 16, 24 and 32 bit CPU's.
;	The following attributes make it suitable for CPU's of the '90:
;
;	small machine dependent kernel and portable high level code
;	subroutine threaded code
;	single code dictionaries
;	each word record has a link field, a name field and a code field
;	simple terminal and file interface to host computer
;	aligned with the proposed ANS Forth Standard
;	easy upgrade path to optimize for specific CPU
;	easy mixing of Forth and assembly language
;	all assembly language tools can be used directly
;
;	You are invited to implement this Model on your favorite CPU and
;	contribute it to the eForth Library for public use. You may use
;	a portable implementation to advertise more sophisticated and
;	optimized version for commercial purposes. However, you are
;	expected to implement the Model faithfully. The eForth Working
;	Group reserves the right to reject implementation which deviates
;	significantly from this Model.
;
;	Representing the eForth Working Group in the Silicon Valley FIG Chapter.
;	Send contributions to:
;
;	Dr. Chen-Hanson Ting
;	156 14th Avenue
;	San Mateo, CA 94402
;	(650) 571-7639
;	ting@offete.com
;
;===============================================================

;; Version control

.EQU	VER	=	2	;major release version
.EQU	EXT	=	2	;minor extension

;; Constants

.EQU	COMPO	=	$040	;lexicon compile only bit
.EQU	IMEDD	=	$080	;lexicon immediate bit

.EQU	BASEE	=	16		;default radix

.EQU	BKSPP	=	8		;back space
.EQU	LF		=	10		;line feed
.EQU	CRR		=	13		;carriage return

.EQU	RETT	=	$9508
.EQU	CALLL	=	$940E

;; Memory allocation for ATmega328P, all byte addresses
;
;	Flash memory
;	$0		Reset and interrupt vectors, RWW section
;	$100	Initial values for variables
;	$200	Start of compiler and user words
;	$7000	Start of interpreter words, NRWW section
;	$7FFF	End of flash memory
;
;	RAM memory
;	$0		CPU and I/O registers
;	$100	Variables
;	$120	Free RAM memory
;	$160	Initial PAD
;	$4F0	Top of data stack
;	$700	Terminal input buffer
;	$7F0	Top of return stack
;	$800	Flash buffer 0
;	$880	Flash buffer 1
;	$8FF	End of RAM memory

.EQU	RPP		=	$7F0	;start of return stack (RP0)
.EQU	TIBB	=	$700	;terminal input buffer (TIB)
.EQU	UPP		=	$100	;start of user area (UP0)
.EQU	SPP		=	$6F0	;start of data stack (SP0)

;;	Flash programmming

.EQU	BUF0	=	$800
.EQU	BUF1	=	$880
.EQU	NEWER	=	$11C	;flash pointer
.EQU	OLDER	=	$11E	;flash pointer
; buffer pointer word format:	dirty,page_addr,cell_addr,buf?

;; Initialize assembly variable

.SET _LINK		=	0		;init a null link

;	Compile a code definition header.

.MACRO	CODE				;;LEX,NAME 
	.DW		_LINK*2			;;link pointer
	.SET _LINK	=	pc		;;link points to a name string
	.DB		@0,@1
	.ENDM

;	Colon header is identical to code header.

.MACRO	COLON				;;LEX,NAME,LABEL
	.DW		_LINK*2			;;link pointer
	.SET _LINK	=	pc		;;link points to a name string
	.DB		@0,@1
	.ENDM

;; Macros defined by amForth

.DEF zerol = r2
.DEF zeroh = r3
.DEF temp4 = r14
.DEF temp5 = r15
.DEF temp0 = r16
.DEF temp1 = r17
.DEF temp2 = r18
.DEF temp3 = r19
.DEF temp6 = r20
.DEF temp7 = r21
.DEF tosl = r24
.DEF tosh = r25

.macro loadtos
	ld tosl, Y+
	ld tosh, Y+
.endmacro

.macro savetos
	st -Y, tosh
	st -Y, tosl
.endmacro

.macro in_
.if (@1 < $40)
  	in @0,@1
.else
  	lds @0,@1
.endif
.endmacro

.macro out_
.if (@0 < $40)
  	out @0,@1
.else
  	sts @0,@1
.endif
.endmacro

.macro readflashcell
	lsl zl
	rol zh
	lpm @0, Z+
	lpm @1, Z+
.endmacro

.macro writeflashcell
	lsl zl
	rol zh
.endmacro

;; Main entry points and COLD start data

	.CSEG
	.ORG	0
	JMP		ORIG

	.ORG	$80		;byte address $100, copy to ram on boot, 
					;saved from ram for turnkey system
	
UZERO:	
	.DW		HI*2	;'BOOT
	.DW		0		;reserved
	.DW		BASEE	;BASE
	.DW		0		;tmp
	.DW		0		;SPAN
	.DW		0		;>IN
	.DW		0		;#TIB
	.DW		TIBB	;TIB
	.DW		INTER*2	;'EVAL
	.DW		0		;HLD
	.DW		LASTN	;CONTEXT pointer
	.DW		CTOP	;CP
	.DW		DTOP	;DP
	.DW		LASTN	;LAST
	.DW		$6F00	;PTR0 to BUF0
	.DW		$6F81	;PTR1 to BUF1
ULAST:

	.ORG	$3800	;byte address $7000
ORIG:	
	in_ 	r10, MCUSR
	clr 	r11
	clr 	zerol
	clr 	zeroh
	out_ 	MCUSR, zerol
	; init return stack pointer
	ldi 	xl,low(RPP)
	out_ 	SPL,xl
	ldi 	xh,high(RPP)
	out_ 	SPH,xh
	; init parameter stack pointer
	ldi 	yl,low(SPP)
	ldi 	yh,high(SPP)
	; jump to Forth starting word
	jmp 	COLD

;; Device dependent I/O

;   ?RX	( -- c T | F )
;	Return input character and true, or a false if no input.

	CODE	4,"?KEY"
QRX:
QKEY:
	savetos
	clr 	tosl
	clr 	tosh
	movw	tosl,zerol
	in_		xl,UCSR0A
	sbrs	xl,7
	ret
	in_		tosl,UDR0
	savetos
	ser		tosl
	ser		tosh
	ret

;   TX!	( c -- )
;	Send character c to the output device.

	CODE	4,"EMIT"
EMIT:
TXSTO:	
	in_		xl,UCSR0A
	sbrs	xl,5
	rJMP	TXSTO
	out_	UDR0,tosl
	loadtos
	ret

;   !IO	( -- )
;	Initialize the serial I/O devices.

;	CODE	3,"!IO"
STOIO:
	ldi		xl,$33	;19200 baud
;	ldi		xl,$66	;9600 baud
	out_	UBRR0L,xl
	clr		xl
	out_	UBRR0H,xl
	ldi		xl,$18	;enable TX and RX
	out_	UCSR0B,xl
	ldi		xl,6	;8 data bits
	out_	UCSR0C,xl
	RET

;; The kernel

;   doLIT	( -- w )
;	Push an inline literal.

;	CODE	COMPO+5,"doLIT"
DOLIT:
	savetos
	pop		zh
	pop		zl
	readflashcell tosl,tosh
	ror		zh
	ror		zl
	push	zl
	push	zh
	ret

;   next	( -- )
;	Run time code for the single index loop.

;	CODE	COMPO+4,"next"
DONXT:
	POP		zh	;ret addr
	POP		zl	;
	pop		xh	;count
	pop		xl
	sbiw	xl, 1
	brge	NEXT1
	adiw	zl,1
	push	zl
	push	zh
	ret
NEXT1:	
	push	xl	;push count back
	push	xh	
	readflashcell	xl,xh
	push	xl
	push	xh
	ret

;   ?branch	( f -- )
;	Branch if flag is zero.

;	CODE	COMPO+7,"?branch"
QBRAN:
	pop		zh
	pop		zl
	or		tosl, tosh
	loadtos
	breq	BRAN1
	adiw	zl,1
	push	zl
	push	zh
	ret

;   branch	( -- )
;	Branch to an inline address.

;	CODE	COMPO+6,"branch"
BRAN:	
	pop		zh
	pop		zl
BRAN1:	
	readflashcell xl,xh
	push	xl
	push	xh
	ret

;   EXECUTE	( b -- )
;	Execute the word at ca=b/2.

	CODE	7,"EXECUTE"
EXECU:
	asr		tosh	;b/2
	ror		tosl
	push	tosl
	push	tosh
	loadtos
	ret

;   EXIT	( -- )
;	Terminate current colon word.

	CODE	4,"EXIT"
EXIT:
	pop		xh
	pop		xl
	ret

;   !	( w a -- )
;	Pop the data stack to memory.

	CODE	1,"!"
STORE:
	movw 	zl, tosl
	loadtos
	std 	Z+1, tosh
	std 	Z+0, tosl
	loadtos
	RET

;   @	( a -- w )
;	Push memory location to the data stack.

	CODE	1,"@"
AT:
	movw 	zl, tosl
	ld 		tosl, z+
	ld 		tosh, z+
	RET

;   I@	( a -- w )
;	Push flash memory cell to the data stack.

	CODE	2,"I@"
IAT:
	RCALL	DOLIT
	.DW		NEWER
	RCALL	BUFQ	;n a new?
	RCALL	QBRAN	;if a=new, fetch n in new_buf
	.DW		IAT1	;else, a=old?
	RCALL	DOLIT	;n a a old
	.DW		OLDER
	RCALL	BUFQ	;n a old?
	RCALL	QBRAN	;if a=old, fetch n in old_buf
	.DW		IAT2	
	movw 	zl, tosl	;else, fetch from flash
	lpm		tosl, z+
	lpm		tosh, z+
	RET
IAT1:
	RCALL	DOLIT
	.DW		NEWER
	RJMP	IAT3
IAT2:
	RCALL	DOLIT
	.DW		OLDER
IAT3:
	RCALL	BUFAT
	RJMP	AT

;   IC@	( a -- w )
;	Push flash memory byte to the data stack.

	CODE	3,"IC@"
ICAT:
	RCALL	DOLIT
	.DW		NEWER
	RCALL	BUFQ	;n a new?
	RCALL	QBRAN	;if a=new, fetch n in new_buf
	.DW		ICAT1	;else, a=old?
	RCALL	DOLIT	;n a a old
	.DW		OLDER
	RCALL	BUFQ	;n a old?
	RCALL	QBRAN	;if a=old, fetch n in old_buf
	.DW		ICAT2	
	movw 	zl, tosl	;else, fetch from flash
	clr 	tosh
	lpm 	tosl, Z
	RET
ICAT1:
	RCALL	DOLIT
	.DW		NEWER
	RJMP	ICAT3
ICAT2:
	RCALL	DOLIT
	.DW		OLDER
ICAT3:
	RCALL	BUFAT
	RJMP	CAT

;	CODE	6,"BUFFER"	; ptr -- buf
BUFFER:
	RCALL	DOLIT
	.DW		$1
	RCALL	ANDD
	RCALL	QBRAN
	.DW		BUF_1
	RCALL	DOLIT
	.DW		BUF1
	RET	
BUF_1:
	RCALL	DOLIT
	.DW		BUF0
	RET

;	CODE	6,"BUF?"	; a new/old -- f
BUFQ:
	RCALL	AT
	RCALL	OVER
	RCALL	XORR
	RCALL	DOLIT
	.DW		$7F80
	RCALL	ANDD
	RET

;	CODE	6,"BUF@"	; a new/old -- buuf_addr
BUFAT:
	RCALL	AT
	RCALL	BUFFER
	RCALL	SWAPP
	RCALL	DOLIT
	.DW		$7F
	RCALL	ANDD
	RJMP	XORR

;   I!	( w a -- )
;	Store w to flash memory byte location.

	CODE	2,"I!"
ISTOR:				;a=new?
	RCALL	DOLIT
	.DW		NEWER
	RCALL	BUFQ	;n a a new_ptr
	RCALL	QBRAN	;if a=new, store n in new_buf
	.DW		ISTOR5	;else, a=old?
;
	RCALL	DOLIT	;n a a old
	.DW		OLDER
	RCALL	BUFQ	;n a a old_ptr
	RCALL	QBRAN	;if a=old, switch ptrs, store n in new_buf
	.DW		ISTOR4	;else, flush old_buf

	RCALL	DOLIT	;n a old
	.DW		OLDER
	RCALL	AT	;n a old_ptr 
	RCALL	DOLIT	;n a dirty?
	.DW		$8000
	RCALL	ANDD
	RCALL	QBRAN	;if not dirty, go read flash data into old_buf
	.DW		ISTOR2	;else, flush old_buf to flash

ISTOR1:	RCALL	FLUSH_OLD
ISTOR2:	RCALL	READ_FLASH
ISTOR3:	RCALL	UPDATE_OLD
ISTOR4:	RCALL	SWITCH
ISTOR5:	RJMP 	UPDATE_NEW

;	CODE	5,"FLUSH"	; --
FLUSH_OLD:
	RCALL	DOLIT	;old
	.DW		OLDER
	RCALL	AT	;old_ptr
	RCALL	DUPP	;old_ptr old_ptr
	RCALL	DOLIT
	.DW		$7F80
	RCALL	ANDD	;old_ptr flash_addr 
	RCALL	DUPP	;old_ptr flash_addr flash_addr
	RCALL	ERASE	;old_ptr flash_addr
;
	RCALL	SWAPP	;flash_addr old_ptr 
	RCALL	BUFFER	;flash_addr buf
	RCALL	SWAPP	;buf flash_addr
	RJMP	WRITE	

;	CODE	4,"@OLD"	;a -- a
READ_FLASH:	;read new flash data into old_buf
	RCALL	DOLIT	;a old
	.DW		OLDER
	RCALL	AT		;a old_ptr
	RCALL	BUFFER	;a buf
	RCALL	OVER	;a buf a
	RCALL	DOLIT
	.DW		$7F80
	RCALL	ANDD	;a buf flash_addr
	RCALL	SWAPP	;a flash_addr buf
	RJMP	READ	;a

;	CODE	4,"!OLD"	;a --
UPDATE_OLD:			;preserve buf? bit
	RCALL	DUPP	;a a
	RCALL	DOLIT	;
	.DW		$7F80
	RCALL	ANDD	;a page_addr
	RCALL	DOLIT
	.DW		OLDER	;a page_addr old
	RCALL	SWAPP	;a old page_addr
	RCALL	OVER	;a old page_addr old
	RCALL	AT	;a old page_addr old_ptr
	RCALL	DOLIT
	.DW		$1
	RCALL	ANDD	;a old page_addr buf?
	RCALL	ORR	;a old updates_old_ptr
	RCALL	SWAPP	;a old_ptr old
	RJMP	STORE	;a

;	CODE	6,"SWITCH"	; --
SWITCH:	
	RCALL	DOLIT	;old
	.DW		OLDER
	RCALL	AT		;old_ptr
	RCALL	DOLIT	;old_ptr new
	.DW		NEWER
	RCALL	AT		;old_ptr new_ptr
	RCALL	DOLIT	;old_ptr new_ptr old
	.DW		OLDER
	RCALL	STORE	;old_ptr
	RCALL	DOLIT	;old_ptr new
	.DW		NEWER
	RJMP	STORE	; 
	
;	CODE	4,"!NEW"	;n a --
UPDATE_NEW:			;write data to new buufer, set dirty bit
	RCALL	DOLIT	;n a 7e
	.DW		$7E
	RCALL	ANDD	;n disp
	RCALL	DOLIT	;n disp new
	.DW		NEWER
	RCALL	AT		;n disp new_ptr
	RCALL	BUFFER	;n disp buf
UPDAT1:
	RCALL	ORR		;n buff_addr
	RCALL	STORE	;update word in new_buf

	RCALL	DOLIT	;set dirty bit in newer
	.DW		NEWER
	RCALL	DUPP	;newer newer
	RCALL	AT		;newer new_ptr
	RCALL	DOLIT
	.DW		$8000
	RCALL	ORR		;newer new_ptr_dirty
	RCALL	SWAPP
	RJMP	STORE	;new buf is dirty now

;	EMPTY-BUFFERS ( -- )
	CODE	5,"FLUSH"

EMPTY_BUF:
	RCALL	EMPTY_OLD
	RCALL	SWITCH
	RCALL	EMPTY_OLD
	RJMP	SWITCH

;	EMPTY_OLD	;flush old buffer if it is dirty

EMPTY_OLD:
	RCALL	DOLIT	;old
	.DW		OLDER
	RCALL	AT		;old_ptr 
	RCALL	DUPP	;old_ptr old_ptr
	RCALL	DOLIT	;
	.DW		$8000
	RCALL	ANDD	;old_ptr dirty?
	RCALL	QBRAN	;if not dirty, exit
	.DW		EMPTY_1	;else, flush old_buf
;
	RCALL	DOLIT	;old_ptr
	.DW		$7FFF
	RCALL	ANDD	;old_ptr, dirty bit cleared
	RCALL	DOLIT
	.DW		OLDER
	RCALL	STORE	;old_ptr flash_addr
	RJMP	FLUSH_OLD
EMPTY_1:
	RJMP	DROP

;   C!	( c b -- )
;	Pop the data stack to byte memory.

	CODE	2,"C!"
CSTOR:
	movw 	zl, tosl
	loadtos
	st 		Z, tosl
	loadtos
	RET

;   C@	( b -- c )
;	Push byte memory location to the data stack.

	CODE	2,"C@"
CAT:
	movw 	zl, tosl
	clr 	tosh
	ld 		tosl, Z
	RET

;   R>	( -- w )
;	Pop the return stack to the data stack.

	CODE	COMPO+2,"R>"
RFROM:
	savetos
	pop		xh
	pop		xl
	pop 	tosh
	pop 	tosl
	push 	xl
	push 	xh
	RET

;   R@	( -- w )
;	Copy top of return stack to the data stack.

	CODE	2,"R@"
RAT:
	savetos
	pop		xh
	pop		xl
	pop 	tosh
	pop 	tosl
	push 	tosl
	push 	tosh
	push 	xl
	push 	xh
	RET

;   >R	( w -- )
;	Push the data stack to the return stack.

	CODE	COMPO+2,">R"
TOR:
	pop		xh
	pop		xl
	push 	tosl
	push 	tosh
	push 	xl
	push 	xh
	loadtos
	RET

;   SP@	( -- a )
;	Push the current data stack pointer.

;	CODE	3,"SP@"
SPAT:
	savetos
	movw	tosl, yl
	RET

;   SP!	( a -- )
;	Set the data stack pointer.

;	CODE	3,"SP!"
SPSTO:
	movw 	yl, tosl
	loadtos
	RET

;   DROP	( w -- )
;	Discard top stack item.

	CODE	4,"DROP"
DROP:
	loadtos
	RET

;   DUP	( w -- w w )
;	Duplicate the top stack item.

	CODE	3,"DUP"
DUPP:
	savetos
	RET

;   SWAP	( w1 w2 -- w2 w1 )
;	Exchange top two stack items.

	CODE	4,"SWAP"
SWAPP:
	movw 	xl, tosl
	ld		tosl,Y+
	ld		tosh,Y+
	st 		-Y, xh
	st 		-Y, xl
	RET

;   OVER	( w1 w2 -- w1 w2 w1 )
;	Copy second stack item to top.

	CODE	4,"OVER"
OVER:
	savetos
	ldd 	tosl, Y+2
	ldd 	tosh, Y+3
	RET

;   0<	( n -- t )
;	Return true if n is negative.

	CODE	2,"0<"
ZLESS:
	tst 	tosh
	movw 	tosl, zerol
	brge 	ZLESS1
	sbiw 	tosl,1
ZLESS1:
	RET

;   AND	( w w -- w )
;	Bitwise AND.

	CODE	3,"AND"
ANDD:
	ld 		xl, Y+
	ld 		xh, Y+
	and 	tosl, xl
	and 	tosh, xh
	RET

;   OR	( w w -- w )
;	Bitwise inclusive OR.

	CODE	2,"OR"
ORR:
	ld 		xl, Y+
	ld 		xh, Y+
	or 		tosl, xl
	or 		tosh, xh
	RET

;   XOR	( w w -- w )
;	Bitwise exclusive OR.

	CODE	3,"XOR"
XORR:
	ld 		xl, Y+
	ld 		xh, Y+
   	eor 	tosl, xl
	eor 	tosh, xh
	RET

;   UM+	( u u -- udsum )
;	Add two unsigned single numbers and return a double sum.

	CODE	3,"UM+"
UPLUS:
	ld 		xl, Y+
	ld 		xh, Y+
	add 	tosl, xl
	adc 	tosh, xh
	savetos
	clr		tosh
	clr		tosl
	rol		tosl
	RET

;; System and user variables

;   doVAR	( -- a )
;	Run time routine for VARIABLE and CREATE.

;	CODE	COMPO+5,"doVAR"
DOVAR:
	savetos
	pop 	zh
	pop 	zl
	readflashcell tosl,tosh
	RET

;   'BOOT	( -- a )
;	Storage of application address.

	COLON	5,"'BOOT"
TBOOT:
	RCALL	DOVAR
	.DW		UPP

;   BASE	( -- a )
;	Storage of the radix base for numeric I/O.

	COLON	4,"BASE"
BASE:
	RCALL	DOVAR
	.DW		UPP+4

;   tmp	( -- a )
;	A temporary storage location used in parse and find.

	COLON	3,"TMP"
TEMP:
	RCALL	DOVAR
	.DW		UPP+6

;   SPAN	( -- a )
;	Hold character count received by EXPECT.

	COLON	4,"SPAN"
SPAN:
	RCALL	DOVAR
	.DW		UPP+8

;   >IN	( -- a )
;	Hold the character pointer while parsing input stream.

	COLON	3,">IN"
INN:
	RCALL	DOVAR
	.DW		UPP+10

;   #TIB	( -- a )
;	Hold the current count in and address of the terminal input buffer.

	COLON	4,"#TIB"
NTIB:
	RCALL	DOVAR
	.DW		UPP+12

;   'TIB	( -- a )
;	Hold the current count in and address of the terminal input buffer.

	COLON	4,"'TIB"
TTIB:
	RCALL	DOVAR
	.DW		UPP+14

;   'EVAL	( -- a )
;	Execution vector of EVAL.

	COLON	5,"'EVAL"
TEVAL:
	RCALL	DOVAR
	.DW		UPP+16

;   HLD	( -- a )
;	Hold a pointer in building a numeric output string.

	COLON	3,"HLD"
HLD:
	RCALL	DOVAR
	.DW		UPP+18

;   CONTEXT	( -- a )
;	A area to specify vocabulary search order.

	COLON	7,"CONTEXT"
CNTXT:
	RCALL	DOVAR
	.DW		UPP+20

;   CP	( -- a )
;	Point to the top of the code dictionary.

	COLON	2,"CP"
CPP:
	RCALL	DOVAR
	.DW		UPP+22

;   DP	( -- a )
;	Point to the free RAM space.

	COLON	2,"DP"
DPP:
	RCALL	DOVAR
	.DW		UPP+24

;   LAST	( -- a )
;	Point to the last name in the name dictionary.

	COLON	4,"LAST"
LAST:
	RCALL	DOVAR
	.DW		UPP+26

;; Common functions

;   2*	( n -- n )
;	Multiply tos by cell size in bytes.

	COLON	2,"2*"
CELLS:
	lsl		tosl
	rol		tosh
	ret

;   2/	( n -- n )
;	Divide tos by cell size in bytes.

	COLON	2,"2/"
TWOSL:
	asr		tosh
	ror		tosl
	ret

;   ALIGNED	( b -- a )
;	Align address to the cell boundary.

;	COLON	7,"ALIGNED"
ALGND:
	adiw	tosl,1
	andi	tosl,254
	ret

;   BL	( -- 32 )
;	Return 32, the blank character.

	COLON	2,"BL"
BLANK:
	savetos
	ldi		tosl,32
	clr		tosh
	ret

;   ?DUP	( w -- w w | 0 )
;	Dup tos if its is not zero.

	COLON	4,"?DUP"
QDUP:
    mov 	temp0, tosl
    or 		temp0, tosh
    breq 	QDUP1
    savetos
QDUP1:
	RET

;   ROT	( w1 w2 w3 -- w2 w3 w1 )
;	Rot 3rd item to top.

	COLON	3,"ROT"
ROT:
    movw 	temp0, tosl
    ld 		temp2, Y+
    ld 		temp3, Y+ 
    loadtos
    st 		-Y, temp3
    st 		-Y, temp2
    st 		-Y, temp1
    st 		-Y, temp0
	RET

;   2DROP	( w w -- )
;	Discard two items on stack.

	COLON	5,"2DROP"
DDROP:
	loadtos
	loadtos
	ret

;   2DUP	( w1 w2 -- w1 w2 w1 w2 )
;	Duplicate top two items.

	COLON	4,"2DUP"
DDUP:
	RCALL	OVER
	RJMP	OVER

;   +	( w w -- sum )
;	Add top two items.

	COLON	1,"+"
PLUS:
    ld 		temp0, Y+
    ld 		temp1, Y+
    add 	tosl, temp0
    adc 	tosh, temp1
	RET

;   NOT	( w -- w )
;	One's complement of tos.

	COLON	6,"INVERT"
INVER:
    com 	tosl
    com 	tosh
	ret

;   NEGATE	( n -- -n )
;	Two's complement of tos.

	COLON	6,"NEGATE"
NEGAT:
	RCALL	INVER
	adiw	tosl,1
	ret

;   DNEGATE	( d -- -d )
;	Two's complement of top double.

	COLON	7,"DNEGATE"
DNEGA:
	RCALL	INVER
	RCALL	TOR
	RCALL	INVER
	RCALL	DOLIT
	.DW	1
	RCALL	UPLUS
	RCALL	RFROM
	RJMP	PLUS

;   -	( n1 n2 -- n1-n2 )
;	Subtraction.

	COLON	1,"-"
SUBB:
    ld 		temp0, Y+
    ld 		temp1, Y+
    sub 	temp0, tosl
    sbc 	temp1, tosh
    movw 	tosl, temp0
	ret

;   ABS		( n -- n )
;	Return the absolute value of n.

	COLON	3,"ABS"
ABSS:
	RCALL	DUPP
	RCALL	ZLESS
	RCALL	QBRAN
	.DW	ABS1
	RJMP	NEGAT
ABS1:	
	RET

;   =	( w w -- t )
;	Return true if top two are equal.

	COLON	1,"="
EQUAL:
	RCALL	XORR
	RCALL	QBRAN
	.DW		EQU1
	RCALL	DOLIT
	.DW		0
	RET
EQU1:
	RCALL	DOLIT
	.DW		-1
	RET

;   U<	( u u -- t )
;	Unsigned compare of top two items.

	COLON	2,"U<"
ULESS:
	RCALL	DDUP
	RCALL	XORR
	RCALL	ZLESS
	RCALL	QBRAN
	.DW		ULES1
	RCALL	SWAPP
	RCALL	DROP
	RJMP	ZLESS
ULES1:
	RCALL	SUBB
	RJMP	ZLESS

;   <	( n1 n2 -- t )
;	Signed compare of top two items.

	COLON	1,"<"
LESS:
	RCALL	DDUP
	RCALL	XORR
	RCALL	ZLESS
	RCALL	QBRAN
	.DW		LESS1
	RCALL	DROP
	RJMP	ZLESS
LESS1:
	RCALL	SUBB
	RJMP	ZLESS

;   MAX	( n n -- n )
;	Return the greater of two top stack items.

	COLON	3,"MAX"
MAX:
	RCALL	DDUP
	RCALL	LESS
	RCALL	QBRAN
	.DW		MAX1
	RCALL	SWAPP
MAX1:
	RJMP	DROP

;   MIN	( n n -- n )
;	Return the smaller of top two stack items.

	COLON	3,"MIN"
MIN:
	RCALL	DDUP
	RCALL	SWAPP
	RCALL	LESS
	RCALL	QBRAN
	.DW		MIN1
	RCALL	SWAPP
MIN1:
	RJMP	DROP

;   WITHIN	( u ul uh -- t )
;	Return true if u is within the range of ul and uh. ( ul <= u < uh )

	COLON	6,"WITHIN"
WITHI:
	RCALL	OVER
	RCALL	SUBB
	RCALL	TOR
	RCALL	SUBB
	RCALL	RFROM
	RJMP	ULESS

;; Divide

;   UM/MOD	( udl udh un -- ur uq )
;	Unsigned divide of a double by a single. Return mod and quotient.

	COLON	6,"UM/MOD"
UMMOD:
    movw 	temp4, tosl
    ld 		temp2, Y+
    ld 		temp3, Y+
    ld 		temp0, Y+
    ld 		temp1, Y+
;; unsigned 32/16 -> 16r16 divide
  ; set 	loop counter
    ldi 	temp6,$10
UMMOD1:
    ; shift left, saving high bit
    clr 	temp7
    lsl 	temp0
    rol 	temp1
    rol 	temp2
    rol 	temp3
    rol 	temp7
  ; try subtracting divisor
    cp 		temp2, temp4
    cpc 	temp3, temp5
    cpc 	temp7,zerol
    brcs 	UMMOD3
UMMOD2:
    ; dividend is large enough
    ; do the subtraction for real
    ; and set lowest bit
    inc 	temp0
    sub 	temp2, temp4
    sbc 	temp3, temp5
UMMOD3:
    dec  	temp6
    brne 	UMMOD1
UMMOD4:
    ; put remainder on stack
    st 		-Y,temp3
    st 		-Y,temp2
    ; put quotient on stack
    movw 	tosl, temp0
	ret

;   M/MOD	( d n -- r q )
;	Signed floored divide of double by single. Return mod and quotient.

	COLON	5,"M/MOD"
MSMOD:
	RCALL	DUPP
	RCALL	ZLESS
	RCALL	DUPP
	RCALL	TOR
	RCALL	QBRAN
	.DW	MMOD1
	RCALL	NEGAT
	RCALL	TOR
	RCALL	DNEGA
	RCALL	RFROM
MMOD1:	
	RCALL	TOR
	RCALL	DUPP
	RCALL	ZLESS
	RCALL	QBRAN
	.DW	MMOD2
	RCALL	RAT
	RCALL	PLUS
MMOD2:	
	RCALL	RFROM
	RCALL	UMMOD
	RCALL	RFROM
	RCALL	QBRAN
	.DW	MMOD3
	RCALL	SWAPP
	RCALL	NEGAT
	RCALL	SWAPP
MMOD3:	
	RET

;   /MOD	( n n -- r q )
;	Signed divide. Return mod and quotient.

	COLON	4,"/MOD"
SLMOD:
	RCALL	OVER
	RCALL	ZLESS
	RCALL	SWAPP
	RJMP	MSMOD

;   MOD	( n n -- r )
;	Signed divide. Return mod only.

	COLON	3,"MOD"
MODD:
	RCALL	SLMOD
	RJMP	DROP


;   /	( n n -- q )
;	Signed divide. Return quotient only.

	COLON	1,"/"
SLASH:
	RCALL	SLMOD
	RCALL	SWAPP
	RJMP	DROP

;; Multiply

;   UM*	( u u -- ud )
;	Unsigned multiply. Return double product.

	COLON	3,"UM*"
UMSTA:
    movw 	temp0, tosl
    loadtos
    ; low bytes
    mul 	tosl,temp0
    movw 	zl, r0
    clr 	temp2
    clr 	temp3
    ; middle bytes
    mul 	tosh, temp0
    add 	zh, r0
    adc 	temp2, r1
    adc 	temp3, zeroh
	mul 	tosl, temp1
	add 	zh, r0
	adc 	temp2, r1
	adc 	temp3, zeroh
	mul 	tosh, temp1
	add 	temp2, r0
	adc 	temp3, r1
	movw 	tosl, zl
	savetos
	movw 	tosl, temp2
	ret

;   *	( n n -- n )
;	Signed multiply. Return single product.

	COLON	1,"*"
STAR:
	RCALL	MSTAR
	RJMP	DROP

;   M*		( n n -- d )
;	Signed multiply. Return double product.

	COLON	2,"M*"
MSTAR:
	RCALL	DDUP
	RCALL	XORR
	RCALL	ZLESS
	RCALL	TOR
	RCALL	ABSS
	RCALL	SWAPP
	RCALL	ABSS
	RCALL	UMSTA
	RCALL	RFROM
	RCALL	QBRAN
	.DW	MSTA1
	RCALL	DNEGA
MSTA1:	
	RET

;   */MOD	( n1 n2 n3 -- r q )
;	Multiply n1 and n2, then divide by n3. Return mod and quotient.

	COLON	5,"*/MOD"
SSMOD:
	RCALL	TOR
	RCALL	MSTAR
	RCALL	RFROM
	RJMP	MSMOD

;   */	( n1 n2 n3 -- q )
;	Multiply n1 by n2, then divide by n3. Return quotient only.

	COLON	2,"*/"
STASL:
	RCALL	SSMOD
	RCALL	SWAPP
	RJMP	DROP

;; Miscellaneous

;   >CHAR	( c -- c )
;	Filter non-printing characters.

;	COLON	5,">CHAR"
TCHAR:
	RCALL	DUPP
	RCALL	BLANK
	RCALL	DOLIT
	.DW		$7F
	RCALL	WITHI
	RCALL	QBRAN
	.DW		TCHAR1
	RET
TCHAR1:	
	RCALL	DROP
	RCALL	DOLIT
	.DW		'_'
	RET


;   DEPTH	( -- n )
;	Return the depth of the data stack.

	COLON	5,"DEPTH"
DEPTH:
	RCALL	SPAT
	RCALL	DOLIT
	.DW		SPP-2
	RCALL	SWAPP
	RCALL	SUBB
	RJMP	TWOSL

;   PICK	( ... +n -- ... w )
;	Copy the nth stack item to tos.

	COLON	4,"PICK"
PICK:
	ADIW	TOSL,1
	RCALL	CELLS
	RCALL	SPAT
	RCALL	PLUS
	RJMP	AT

;; Memory access

;   +!	( n a -- )
;	Add n to the contents at address a.

	COLON	2,"+!"
PSTOR:
	RCALL	SWAPP
	RCALL	OVER
	RCALL	AT
	RCALL	PLUS
	RCALL	SWAPP
	RJMP	STORE

;   COUNT	( b -- b +n )
;	Return count byte of a string and add 1 to byte address.

	COLON	5,"COUNT"
COUNT:
	movw	zl, tosl
	ld		temp0, z+
	movw	tosl, zl
	savetos
	mov		tosl, temp0
	clr		tosh
	ret

;   ICOUNT	( b -- b +n )
;	Return count byte of a string and add 1 to byte address.

	COLON	6,"ICOUNT"
ICOUNT:
	RCALL	DUPP
	adiw	tosl,1
	RCALL	SWAPP
	RJMP	ICAT

;   HERE	( -- a )
;	Return the top of the code dictionary.

	COLON	4,"HERE"
HEREE:
	RCALL	DPP
	RJMP	AT

;   PAD	( -- a )
;	Return the address of the text buffer above the code dictionary.

	COLON	3,"PAD"
PAD:
	RCALL	HEREE
	RCALL	DOLIT
	.DW		$40
	RJMP	PLUS

;   TIB	( -- a )
;	Return the address of the terminal input buffer.

	COLON	3,"TIB"
TIB:
	RCALL	NTIB
	ADIW	TOSL,2
	RJMP	AT

;   @EXECUTE	( a -- )
;	Execute vector stored in address a.

	COLON	8,"@EXECUTE"
ATEXE:
	RCALL	AT
	RCALL	QDUP	;?address or zero
	RCALL	QBRAN
	.DW		EXE1
	RCALL	EXECU	;execute if non-zero
EXE1:
	RET				;do nothing if zero

;   CMOVE	( b1 b2 u -- )
;	Copy u bytes from b1 to b2.

	COLON	5,"CMOVE"
CMOVE:
	RCALL	TOR
	RJMP	CMOV2
CMOV1:
	RCALL	TOR
	RCALL	COUNT
	RCALL	RAT
	RCALL	CSTOR
	RCALL	RFROM
	ADIW	TOSL,1
CMOV2:
	RCALL	DONXT
	.DW		CMOV1
	RJMP	DDROP

;	UPPER	( c -- c' )
;	Change character to upper case

;	COLON	5,"UPPER"
UPPER:
	RCALL	DUPP
	RCALL	DOLIT
	.DW		$61
	RCALL	DOLIT
	.DW		$7B
	RCALL	WITHI
	RCALL	QBRAN
	.DW		UPPER1
	RCALL	DOLIT
	.DW		$5F
	RCALL	ANDD
UPPER1:
	RET

;   UMOVE	( a b u -- )
;	Copy u bytes from b1 to b2, changing to upper case.

;	COLON	5,"UMOVE"
UMOVE:
	RCALL	TOR
	RJMP	UMOV2
UMOV1:
	RCALL	TOR
	RCALL	COUNT
	RCALL	UPPER
	RCALL	RAT
	RCALL	CSTOR
	RCALL	RFROM
	ADIW	TOSL,1
UMOV2:
	RCALL	DONXT
	.DW		UMOV1
	RJMP	DDROP

;   FILL	( b u c -- )
;	Fill u bytes of character c to area beginning at b.

	COLON	4,"FILL"
FILL:
	RCALL	SWAPP
	RCALL	TOR
	RCALL	SWAPP
	RJMP	FILL2
FILL1:
	RCALL	DDUP
	RCALL	CSTOR
	ADIW	TOSL,1
FILL2:
	RCALL	DONXT
	.DW		FILL1
	RJMP	DDROP

;; Numeric output, single precision

;   DIGIT	( u -- c )
;	Convert digit u to a character.

;	COLON	5,"DIGIT"
DIGIT:
	RCALL	DOLIT
	.DW		9
	RCALL	OVER
	RCALL	LESS
	RCALL	DOLIT
	.DW		7
	RCALL	ANDD
	RCALL	PLUS
	RCALL	DOLIT
	.DW		'0'
	RJMP	PLUS

;   EXTRACT	( n base -- n c )
;	Extract the least significant digit from n.

;	COLON	7,"EXTRACT"
EXTRC:
	RCALL	DOLIT
	.DW		0
	RCALL	SWAPP
	RCALL	UMMOD
	RCALL	SWAPP
	RJMP	DIGIT

;   <#	( -- )
;	Initiate the numeric output process.

	COLON	2,"<#"
BDIGS:
	RCALL	PAD
	RCALL	HLD
	RJMP	STORE

;   HOLD	( c -- )
;	Insert a character into the numeric output string.

	COLON	4,"HOLD"
HOLD:
	RCALL	HLD
	RCALL	AT
	SBIW	TOSL,1
	RCALL	DUPP
	RCALL	HLD
	RCALL	STORE
	RJMP	CSTOR

;   #	( u -- u )
;	Extract one digit from u and append the digit to output string.

	COLON	1,"#"
DIG:
	RCALL	BASE
	RCALL	AT
	RCALL	EXTRC
	RJMP	HOLD

;   #S	( u -- 0 )
;	Convert u until all digits are added to the output string.

	COLON	2,"#S"
DIGS:
DIGS1:
	RCALL	DIG
	RCALL	DUPP
	RCALL	QBRAN
	.DW		DIGS2
	RJMP	DIGS1
DIGS2:
	RET

;   SIGN	( n -- )
;	Add a minus sign to the numeric output string.

	COLON	4,"SIGN"
SIGN:
	RCALL	ZLESS
	RCALL	QBRAN
	.DW		SIGN1
	RCALL	DOLIT
	.DW		'-'
	RCALL	HOLD
SIGN1:	RET

;   #>	( w -- b u )
;	Prepare the output string to be TYPE'd.

	COLON	2,"#>"
EDIGS:
	RCALL	DROP
	RCALL	HLD
	RCALL	AT
	RCALL	PAD
	RCALL	OVER
	RJMP	SUBB

;   str		( w -- b u )
;	Convert a signed integer to a numeric string.

;	COLON	3,"str"
STR:
	RCALL	DUPP
	RCALL	TOR
	RCALL	ABSS
	RCALL	BDIGS
	RCALL	DIGS
	RCALL	RFROM
	RCALL	SIGN
	RJMP	EDIGS

;   HEX		( -- )
;	Use radix 16 as base for numeric conversions.

	COLON	3,"HEX"
HEX:
	RCALL	DOLIT
	.DW	16
	RCALL	BASE
	RJMP	STORE

;   DECIMAL	( -- )
;	Use radix 10 as base for numeric conversions.

	COLON	7,"DECIMAL"
DECIM:
	RCALL	DOLIT
	.DW	10
	RCALL	BASE
	RJMP	STORE

;; Numeric input, single precision

;   DIGIT?	( c base -- u t )
;	Convert a character to its numeric value. A flag indicates success.

;	COLON	6,"DIGIT?"
DIGTQ:
	RCALL	TOR
	RCALL	DOLIT
	.DW		'0'
	RCALL	SUBB
	RCALL	DOLIT
	.DW		9
	RCALL	OVER
	RCALL	LESS
	RCALL	QBRAN
	.DW		DGTQ1
	RCALL	DOLIT
	.DW		7
	RCALL	SUBB
	RCALL	DUPP
	RCALL	DOLIT
	.DW		10
	RCALL	LESS
	RCALL	ORR
DGTQ1:
	RCALL	DUPP
	RCALL	RFROM
	RJMP	ULESS

;   NUMBER?	( a -- n T | a F )
;	Convert a number string to integer. Push a flag on tos.

	COLON	7,"NUMBER?"
NUMBQ:
	RCALL	BASE
	RCALL	AT
	RCALL	TOR
	RCALL	DOLIT
	.DW		0
	RCALL	OVER
	RCALL	COUNT
	RCALL	OVER
	RCALL	CAT
	RCALL	DOLIT
	.DW		'$'
	RCALL	EQUAL
	RCALL	QBRAN
	.DW		NUMQ1
	RCALL	HEX
	RCALL	SWAPP
	adiw	tosl,1
	RCALL	SWAPP
	sbiw	tosl,1
NUMQ1:
	RCALL	OVER
	RCALL	CAT
	RCALL	DOLIT
	.DW		'-'
	RCALL	EQUAL
	RCALL	TOR
	RCALL	SWAPP
	RCALL	RAT
	RCALL	SUBB
	RCALL	SWAPP
	RCALL	RAT
	RCALL	PLUS
	RCALL	QDUP
	RCALL	QBRAN
	.DW		NUMQ6
	sbiw	tosl,1
	RCALL	TOR
NUMQ2:
	RCALL	DUPP
	RCALL	TOR
	RCALL	CAT
	RCALL	BASE
	RCALL	AT
	RCALL	DIGTQ
	RCALL	QBRAN
	.DW		NUMQ4
	RCALL	SWAPP
	RCALL	BASE
	RCALL	AT
	RCALL	STAR
	RCALL	PLUS
	RCALL	RFROM
	adiw	tosl,1
	RCALL	DONXT
	.DW		NUMQ2
	RCALL	DROP
	RCALL	RAT
	RCALL	QBRAN
	.DW		NUMQ3
	RCALL	NEGAT
NUMQ3:
	RCALL	SWAPP
	RJMP	NUMQ5
NUMQ4:
	RCALL	RFROM
	RCALL	RFROM
	RCALL	DDROP
	RCALL	DDROP
	RCALL	DOLIT
	.DW		0
NUMQ5:
	RCALL	DUPP
NUMQ6:
	RCALL	RFROM
	RCALL	DDROP
	RCALL	RFROM
	RCALL	BASE
	RJMP	STORE

;; Basic I/O

;   KEY	( -- c )
;	Wait for and return an input character.

	COLON	3,"KEY"
KEY:
KEY1:
	RCALL	QRX
	RCALL	QBRAN
	.DW		KEY1
	RET

;   SPACE	( -- )
;	Send the blank character to the output device.

	COLON	5,"SPACE"
SPACE:
	RCALL	BLANK
	RJMP	EMIT

;   CHARS	( +n c -- )
;	Send n characters to the output device.

;	COLON	5,"CHARS"
CHARS:
	RCALL	SWAPP
	RCALL	TOR
	RJMP	CHAR2
CHAR1:
	RCALL	DUPP
	RCALL	EMIT
CHAR2:
	RCALL	DONXT
	.DW		CHAR1
	RJMP	DROP

;   SPACES	( +n -- )
;	Send n spaces to the output device.

	COLON	6,"SPACES"
SPACS:
	RCALL	BLANK
	RJMP	CHARS

;   TYPE	( b u -- )
;	Output u characters from b.

	COLON	4,"TYPE"
TYPES:
	RCALL	TOR
	RJMP	TYPE2
TYPE1:
	RCALL	COUNT
	RCALL	TCHAR
	RCALL	EMIT
TYPE2:
	RCALL	DONXT
	.DW		TYPE1
	RJMP	DROP

;   ITYPE	( b u -- )
;	Output u characters from b.

	COLON	5,"ITYPE"
ITYPES:
	RCALL	TOR
	RJMP	ITYPE2
ITYPE1:
	RCALL	ICOUNT
	RCALL	TCHAR
	RCALL	EMIT
ITYPE2:
	RCALL	DONXT
	.DW		ITYPE1
	RJMP	DROP

;   CR	( -- )
;	Output a carriage return and a line feed.

	COLON	2,"CR"
CR:
	RCALL	DOLIT
	.DW		CRR
	RCALL	EMIT
	RCALL	DOLIT
	.DW		LF
	RJMP	EMIT

;   do$	( -- a )
;	Return the address of a compiled string.

;	COLON	COMPO+3,"do$"
DOSTR:
	RCALL	RFROM	;ra
	RCALL	RFROM	;ra a
	RCALL	DUPP	;ra a a
	RCALL	DUPP	;ra a a a
	movw	zl,tosl
	readflashcell	tosl,tosh
	clr		tosh	;ra a a count
	RCALL	TWOSL
	RCALL	PLUS
	ADIW	TOSL,1	;ra a a' 
	RCALL	TOR	;ra a
	RCALL	SWAPP	;a ra
	RCALL	TOR	;a
	RCALL	CELLS	;byte address
	RET

;   $"|	( -- a )
;	Run time routine compiled by $". Return address of a compiled string.

;	COLON	COMPO+3,'$'
;	.DB		'"','|'
STRQP:
	RCALL	DOSTR
	RET				;force a call to do$

;   ."|	( -- )
;	Run time routine of ." . Output a compiled string.

;	COLON	COMPO+3,'.'
;	.DB		'"','|'
DOTQP:
	RCALL	DOSTR
	RCALL	ICOUNT
	RJMP	ITYPES

;   .R		( n +n -- )
;	Display an integer in a field of n columns, right justified.

	COLON	2,".R"
DOTR:
	RCALL	TOR
	RCALL	STR
	RCALL	RFROM
	RCALL	OVER
	RCALL	SUBB
	RCALL	SPACS
	RJMP	TYPES

;   U.R	( u +n -- )
;	Display an unsigned integer in n column, right justified.

	COLON	3,"U.R"
UDOTR:
	RCALL	TOR
	RCALL	BDIGS
	RCALL	DIGS
	RCALL	EDIGS
	RCALL	RFROM
	RCALL	OVER
	RCALL	SUBB
	RCALL	SPACS
	RJMP	TYPES

;   U.	( u -- )
;	Display an unsigned integer in free format.

	COLON	2,"U."
UDOT:
	RCALL	BDIGS
	RCALL	DIGS
	RCALL	EDIGS
	RCALL	SPACE
	RJMP	TYPES

;   .		( w -- )
;	Display an integer in free format, preceeded by a space.

	COLON	1,"."
DOT:
	RCALL	BASE
	RCALL	AT
	RCALL	DOLIT
	.DW	10
	RCALL	XORR	;?decimal
	RCALL	QBRAN
	.DW	DOT1
	RJMP	UDOT
DOT1:	
	RCALL	STR
	RCALL	SPACE
	RJMP	TYPES

;   ?	( a -- )
;	Display the contents in a memory cell.

	COLON	1,"?"
QUEST:
	RCALL	AT
	RJMP	DOT

;; Parsing

;   parse	( b u c -- b u delta ; <string> )
;	Scan string delimited by c. Return found string and its offset.

;	COLON	5,"parse"
PARS:
	RCALL	TEMP
	RCALL	STORE
	RCALL	OVER
	RCALL	TOR
	RCALL	DUPP
	RCALL	QBRAN
	.DW		PARS8
	SBIW	TOSL,1
	RCALL	TEMP
	RCALL	CAT
	RCALL	BLANK
	RCALL	EQUAL
	RCALL	QBRAN
	.DW		PARS3
	RCALL	TOR
PARS1:
	RCALL	BLANK
	RCALL	OVER
	RCALL	CAT	;skip leading blanks ONLY
	RCALL	SUBB
	RCALL	ZLESS
	RCALL	INVER
	RCALL	QBRAN
	.DW		PARS2
	ADIW	TOSL,1
	RCALL	DONXT
	.DW		PARS1
	RCALL	RFROM
	RCALL	DROP
	RCALL	DOLIT
	.DW		0
	RCALL	DUPP
	RET
PARS2:
	RCALL	RFROM
PARS3:
	RCALL	OVER
	RCALL	SWAPP
	RCALL	TOR
PARS4:
	RCALL	TEMP
	RCALL	CAT
	RCALL	OVER
	RCALL	CAT
	RCALL	SUBB	;scan for delimiter
	RCALL	TEMP
	RCALL	CAT
	RCALL	BLANK
	RCALL	EQUAL
	RCALL	QBRAN
	.DW		PARS5
	RCALL	ZLESS
PARS5:
	RCALL	QBRAN
	.DW		PARS6
	ADIW	TOSL,1
	RCALL	DONXT
	.DW		PARS4
	RCALL	DUPP
	RCALL	TOR
	RJMP	PARS7
PARS6:
	RCALL	RFROM
	RCALL	DROP
	RCALL	DUPP
	ADIW	TOSL,1
	RCALL	TOR
PARS7:
	RCALL	OVER
	RCALL	SUBB
	RCALL	RFROM
	RCALL	RFROM
	RJMP	SUBB
PARS8:
	RCALL	OVER
	RCALL	RFROM
	RJMP	SUBB

;   PARSE	( c -- b u ; <string> )
;	Scan input stream and return counted string delimited by c.

;	COLON	5,"PARSE"
PARSE:
	RCALL	TOR
	RCALL	TIB
	RCALL	INN
	RCALL	AT
	RCALL	PLUS	;current input buffer pointer
	RCALL	NTIB
	RCALL	AT
	RCALL	INN
	RCALL	AT
	RCALL	SUBB	;remaining count
	RCALL	RFROM
	RCALL	PARS
	RCALL	INN
	RJMP	PSTOR

;   .(	( -- )
;	Output following string up to next ) .

	COLON	IMEDD+2,".("
DOTPR:
	RCALL	DOLIT
	.DW		')'
	RCALL	PARSE
	RJMP	TYPES

;   (	( -- )
;	Ignore following string up to next ) . A comment.

	COLON	IMEDD+1,"("
PAREN:
	RCALL	DOLIT
	.DW		')'
	RCALL	PARSE
	RJMP	DDROP

;   \	( -- )
;	Ignore following text till the end of line.

	COLON	IMEDD+1,"\\"
BKSLA:
	RCALL	DOLIT
	.DW		$D
	RCALL	PARSE
	RJMP	DDROP


;   CHAR	( -- c )
;	Parse next word and return its first character.

	COLON	4,"CHAR"
CHARR:
	RCALL	BLANK
	RCALL	PARSE
	RCALL	DROP
	RJMP	CAT

;   TOKEN	( -- a ; <string> )
;	Parse a word from input stream and copy it to name dictionary.

;	COLON	5,"TOKEN"
TOKEN:
	RCALL	BLANK
	RCALL	PARSE
	RCALL	DOLIT
	.DW		31
	RCALL	MIN
	RCALL	HEREE
	RCALL 	DDUP
	RCALL	CSTOR
	RCALL 	DDUP
	RCALL	PLUS
	ADIW	TOSL,1
	RCALL	DOLIT
	.DW		0
	RCALL	SWAPP
	RCALL	CSTOR
	ADIW	TOSL,1
	RCALL	SWAPP
	RCALL	UMOVE
	RJMP	HEREE

;   WORD	( c -- a ; <string> )
;	Parse a word from input stream and copy it to code dictionary.

	COLON	4,"WORD"
WORDD:
	RCALL	PARSE
	RCALL	HEREE
	RCALL 	DDUP
	RCALL	CSTOR
	RCALL 	DDUP
	RCALL	PLUS
	ADIW	TOSL,1
	RCALL	DOLIT
	.DW		0
	RCALL	SWAPP
	RCALL	CSTOR
	ADIW	TOSL,1
	RCALL	SWAPP
	RCALL	CMOVE
	RJMP	HEREE

;; Dictionary search

;   NAME>	( na -- ca )
;	Return a code address given a name address.

	COLON	5,"NAME>"
NAMET:
	RCALL	ICOUNT
	RCALL	DOLIT
	.DW		$1F
	RCALL	ANDD
	RCALL	PLUS
	RJMP	ALGND

;   SAME?	( b a u -- b a f \ -0+ )
;	Compare u bytes in two strings. Return 0 if identical.

;	COLON	5,"SAME?"
SAMEQ:
	RCALL	TWOSL
	RCALL	TOR
	RJMP	SAME2
SAME1:
	RCALL	OVER
	RCALL	RAT
	RCALL	CELLS
	RCALL	PLUS
	RCALL	AT
	RCALL	OVER
	RCALL	RAT
	RCALL	CELLS
	RCALL	PLUS
	RCALL	IAT
	RCALL	SUBB
	RCALL	QDUP
	RCALL	QBRAN
	.DW		SAME2
	RCALL	RFROM
	RJMP	DROP
SAME2:
	RCALL	DONXT
	.DW		SAME1
	RCALL	DOLIT
	.DW		0
	RET

;   find	( a va -- ca na | a F )
;	Search a vocabulary for a string. Return ca and na if succeeded.

;	COLON	4,"find"
FIND:
	RCALL	SWAPP
	RCALL	DUPP
	RCALL	CAT
	RCALL	TEMP
	RCALL	STORE
	RCALL	DUPP
	RCALL	AT
	RCALL	TOR
	ADIW	TOSL,2	;va a+2 --
	RCALL	SWAPP	;a+2 va --
FIND1:
	RCALL	DUPP
	RCALL	QBRAN
	.DW		FIND6
	RCALL	DUPP
	RCALL	IAT
	RCALL	DOLIT
	.DW		$FF3F
	RCALL	ANDD
	RCALL	RAT
	RCALL	XORR
	RCALL	QBRAN
	.DW		FIND2
	ADIW	TOSL,2	;a+2 va+2 --
	RCALL	DOLIT
	.DW		-1
	RJMP	FIND3
FIND2:
	ADIW	TOSL,2	;a+2 va+2 --
	RCALL	TEMP
	RCALL	AT
	RCALL	SAMEQ
FIND3:
	RJMP	FIND4
FIND6:
	RCALL	RFROM
	RCALL	DROP
	RCALL	SWAPP
	SBIW	TOSL,2
	RJMP	SWAPP
FIND4:
	RCALL	QBRAN
	.DW		FIND5
	SBIW	TOSL,4
	RCALL	IAT
	RJMP	FIND1
FIND5:
	RCALL	RFROM
	RCALL	DROP
	RCALL	SWAPP
	RCALL	DROP
	SBIW	TOSL,2
	RCALL	DUPP
	RCALL	NAMET
	RJMP	SWAPP

;   NAME?	( a -- ca na | a F )
;	Search all context vocabularies for a string.

;	COLON	5,"NAME?"
NAMEQ:
	RCALL	CNTXT
	RCALL	AT
	RJMP	FIND

;; Terminal response

;   ^H	( bot eot cur -- bot eot cur )
;	Backup the cursor by one character.

;	COLON	2,"^H"
BKSP:
	RCALL	TOR
	RCALL	OVER
	RCALL	RFROM
	RCALL	SWAPP
	RCALL	OVER
	RCALL	XORR
	RCALL	QBRAN
	.DW		BACK1
	RCALL	DOLIT
	.DW		BKSPP
	RCALL	EMIT
	SBIW	TOSL,1
	RCALL	BLANK
	RCALL	EMIT
	RCALL	DOLIT
	.DW		BKSPP
	RCALL	EMIT
BACK1:
	RET

;   TAP	( bot eot cur c -- bot eot cur )
;	Accept and echo the key stroke and bump the cursor.

;	COLON	3,"TAP"
TAP:
	RCALL	DUPP
	RCALL	EMIT
	RCALL	OVER
	RCALL	CSTOR
	adiw	tosl,1
	ret

;   kTAP	( bot eot cur c -- bot eot cur )
;	Process a key stroke, CR or backspace.

;	COLON	4,"kTAP"
KTAP:
	RCALL	DUPP
	SBIW	TOSL,CRR
	RCALL	QBRAN
	.DW		KTAP2
	SBIW	TOSL,BKSPP
	RCALL	QBRAN
	.DW		KTAP1
	RCALL	BLANK
	RJMP	TAP
KTAP1:
	RJMP	BKSP
KTAP2:
	RCALL	DROP
	RCALL	SWAPP
	RCALL	DROP
	RJMP	DUPP

;   accept	( b u -- b u )
;	Accept characters to input buffer. Return with actual count.

;	COLON	6,"accept"
ACCEP:
	RCALL	OVER
	RCALL	PLUS
	RCALL	OVER
ACCP1:
	RCALL	DDUP
	RCALL	XORR
	RCALL	QBRAN
	.DW		ACCP4
	RCALL	KEY
	RCALL	DUPP
	RCALL	BLANK
	RCALL	SUBB
	RCALL	DOLIT
	.DW		$5F
	RCALL	ULESS
	RCALL	QBRAN
	.DW		ACCP2
	RCALL	TAP
	RJMP	ACCP3
ACCP2:
	RCALL	KTAP
ACCP3:
	RJMP	ACCP1
ACCP4:
	RCALL	DROP
	RCALL	OVER
	RJMP	SUBB

;   EXPECT	( b u -- )
;	Accept input stream and store count in SPAN.

	COLON	6,"EXPECT"
EXPEC:
	RCALL	ACCEP
	RCALL	SPAN
	RCALL	STORE
	RJMP	DROP

;   QUERY	( -- )
;	Accept input stream to terminal input buffer.

	COLON	5,"QUERY"
QUERY:
	RCALL	TIB
	RCALL	DOLIT
	.DW		80
	RCALL	ACCEP
	RCALL	NTIB
	RCALL	STORE
	RCALL	DROP
	RCALL	DOLIT
	.DW		0
	RCALL	INN
	RJMP	STORE

;; Error handling


;   ERROR	( a -- )
;	Return address of a null string with zero count.

;	COLON	5,"ERROR"
ERROR:
	RCALL	SPACE
	RCALL	COUNT
	RCALL	TYPES
	RCALL	DOLIT
	.DW		$3F
	RCALL	EMIT
ABORT:
	RCALL	CR
	RCALL	EMPTY_BUF
	ldi 	yl,low(SPP)
	ldi 	yh,high(SPP)
	RJMP	QUIT

;   abort"	( f -- )
;	Run time routine of ABORT" . Abort with a message.

;	COLON	COMPO+6,"abort"
;	.DB		'"'
ABORQ:
	RCALL	QBRAN
	.DW		ABOR1	;text flag
	RCALL	DOSTR
	RCALL	ICOUNT	;pass error string
	RCALL	ITYPES
	RCALL	ABORT
	RJMP	QUIT
ABOR1:
	RCALL	DOSTR
	RJMP	DROP

;; The text interpreter

;   $INTERPRET	( a -- )
;	Interpret a word. If failed, try to convert it to an integer.

;	COLON	10,"$INTERPRET"
INTER:
	RCALL	NAMEQ
	RCALL	QDUP	;?defined
	RCALL	QBRAN
	.DW		INTE1
	RCALL	IAT
	RCALL	DOLIT
	.DW		COMPO
	RCALL	ANDD	;?compile only lexicon bits
	RCALL	ABORQ
	.DB		13," compile only"
	RCALL	EXECU
	RET	;execute defined word
INTE1:
	RCALL	NUMBQ
	RCALL	QBRAN
	.DW		INTE2
	RET
INTE2:
	RJMP	ERROR	;error

;   [	( -- )
;	Start the text interpreter.

	COLON	IMEDD+1,"["
LBRAC:
	RCALL	DOLIT
	.DW		INTER*2
	RCALL	TEVAL
	RJMP	STORE

;   .OK	( -- )
;	Display "ok" only while interpreting.

;	COLON	3,".OK"
DOTOK:
	RCALL	DOLIT
	.DW		INTER*2
	RCALL	TEVAL
	RCALL	AT
	RCALL	EQUAL
	RCALL	QBRAN
	.DW		DOTO1
	RCALL	DOTQP
	.DB		2,"ok"
DOTO1:	RJMP	CR

;   ?STACK	( -- )
;	Abort if the data stack underflows.

;	COLON	6,"?STACK"
QSTAC:
	RCALL	DEPTH
	RCALL	ZLESS	;check only for underflow
	RCALL	ABORQ
	.DB		10," underflow"
	RET

;   EVAL	( -- )
;	Interpret the input stream.

	COLON	4,"EVAL"
EVAL:
EVAL1:	RCALL	TOKEN
	RCALL	DUPP
	RCALL	CAT	;?input stream empty
	RCALL	QBRAN
	.DW		EVAL2
	RCALL	TEVAL
	RCALL	ATEXE
;	RCALL	INTER
	RCALL	QSTAC	;evaluate input, check stack
	RJMP	EVAL1
EVAL2:
	RCALL	DROP
	RJMP	DOTOK

;; Shell

;   QUIT	( -- )
;	Reset return stack pointer and start text interpreter.

	COLON	4,"QUIT"
QUIT:
	ldi 	xl,low(RPP)
	out_ 	SPL,xl
	ldi 	xh,high(RPP)
	out_ 	SPH,xh
	RCALL	DOLIT
	.DW		TIBB
	RCALL	TTIB
	RCALL	STORE
QUIT1:
	RCALL	LBRAC	;start interpretation
QUIT2:
	RCALL	QUERY	;get input
	RCALL	EVAL
	RJMP	QUIT2	;continue till error

;; The compiler

;   '	( -- ca )
;	Search context vocabularies for the next word in input stream.

	COLON	1,"'"
TICK:
	RCALL	TOKEN
	RCALL	NAMEQ	;?defined
	RCALL	QBRAN
	.DW		TICK1
	RET				;yes, push code address
TICK1:
	RJMP	ERROR	;no, error

;; Tools

;   DUMP	( a u -- )
;	Dump 128 bytes from ain RAM, in a formatted manner.

	COLON	4,"DUMP"
DUMP:
	RCALL	DOLIT
	.DW		7
	RCALL	TOR		;start count down loop
DUMP1:	RCALL	CR
	RCALL	DUPP
	RCALL	DOLIT
	.DW		5
	RCALL	UDOTR
	RCALL	SPACE
	RCALL	DOLIT
	.DW		15
	RCALL	TOR
DUMP2:
	RCALL	COUNT
	RCALL	DOLIT
	.DW		3
	RCALL	UDOTR
	RCALL	DONXT	;display printable characters
	.DW		DUMP2
	RCALL	SPACE
	RCALL	DUPP
	RCALL	DOLIT
	.DW		16
	RCALL	SUBB
	RCALL	DOLIT
	.DW		16
	RCALL	TYPES
	RCALL	DONXT
	.DW		DUMP1	;loop till done
	RJMP	DROP

;   IDUMP	( a -- )
;	Dump 128 bytes from a in flash, in a formatted manner.

	COLON	5,"IDUMP"
IDUMP:
	RCALL	DOLIT
	.DW		7
	RCALL	TOR	;start count down loop
IDUMP1:
	RCALL	CR
	RCALL	DUPP
	RCALL	DOLIT
	.DW		5
	RCALL	UDOTR
	RCALL	SPACE
	RCALL	DOLIT
	.DW		15
	RCALL	TOR
IDUMP2:
	RCALL	ICOUNT
	RCALL	DOLIT
	.DW		3
	RCALL	UDOTR
	RCALL	DONXT	;display printable characters
	.DW		IDUMP2
	RCALL	SPACE
	RCALL	DUPP
	RCALL	DOLIT
	.DW		16
	RCALL	SUBB
	RCALL	DOLIT
	.DW		16
	RCALL	ITYPES
	RCALL	DONXT
	.DW		IDUMP1	;loop till done
	RJMP	DROP


;   .S	( ... -- ... )
;	Display the contents of the data stack.

	COLON	2,".S"
DOTS:
	RCALL	DEPTH	;stack depth
	RCALL	TOR	;start count down loop
	RJMP	DOTS2	;skip first pass
DOTS1:
	RCALL	RAT
	RCALL	PICK
	RCALL	DOT	;index stack, display contents
DOTS2:
	RCALL	DONXT 
	.DW		DOTS1	;loop till done
	RCALL	DOTQP
	.DB		4," <sp"
	RET

;   >NAME	( ca -- na | F )
;	Convert code address to a name address.

;	COLON	5,">NAME"
TNAME:
	RCALL	TOR
	RCALL	CNTXT
	RCALL	AT	;na
TNAM1:	
	RCALL	DUPP	;na na
	RCALL	QBRAN
	.DW		TNAM2
	RCALL	DUPP	;na na
	RCALL	NAMET	;na ca
	RCALL	RAT	;na ca ca
	RCALL	XORR	;na f
	RCALL	QBRAN
	.DW		TNAM2
	SBIW	TOSL,2	;la
	RCALL	IAT	;na'
	RCALL	BRAN
	.DW		TNAM1
TNAM2:
	RCALL	RFROM	;na or 0
	RJMP	DROP

;   .ID	( na -- )
;	Display the name at address.

;	COLON	3,".ID"
DOTID:
	RCALL	ICOUNT
	RCALL	DOLIT
	.DW		31
	RCALL	ANDD
	RJMP 	ITYPES

;   WORDS	( -- )
;	Display the names in the context vocabulary.

	COLON	5,"WORDS"
WORDS:
	RCALL	CR
	RCALL	CNTXT
	RCALL	AT	;na
WORS1:	
	RCALL	QDUP	;end of list?
	RCALL	QBRAN
	.DW		WORS2
	RCALL	DUPP	;na na
	RCALL	SPACE
	RCALL	DOTID	;display a name
	SBIW	TOSL,2	;la
	RCALL	IAT	;na'
	RCALL	BRAN
	.DW		WORS1
WORS2:
	RET


;; Hardware reset

;   hi	( -- )
;	Display the sign-on message of eForth.

;	COLON	2,"hi"
HI:
;	RCALL	STOIO
	RCALL	CR
	RCALL	DOTQP 	;initialize I/O
	.DB		15,"328eForth v2.20"	;model
	RJMP	CR

;   COLD	( -- )
;	The hilevel cold start sequence.

	COLON	4,"COLD"
COLD:
COLD1:
	RCALL	STOIO	
	RCALL	DOLIT
	.DW		$100
	RCALL	DUPP
	RCALL	READ	;initialize user area
	RCALL	DOLIT	;init older buffer
	.DW		OLDER
	RCALL	AT		;
	RCALL	READ_FLASH
	RCALL	SWITCH
	RCALL	DOLIT	;init newer buffer
	.DW		OLDER
	RCALL	AT		;
	RCALL	READ_FLASH
	RCALL	SWITCH
	RCALL	DDROP
	RCALL	TBOOT
	RCALL	ATEXE
	RJMP	QUIT	;start interpretation

.equ 	PAGESIZEB = PAGESIZE*2 ;PAGESIZEB is page size in BYTES, not words
.def	spmcrval = r20
.def	looplo = r22
.def	loophi = r23

; Page Erase
;	ERASE ( a -- )
;	Erase a page of flash memory

	COLON	5,"ERASE"
ERASE:
	movw	zl,tosl
	loadtos
ERASE_1:
	ldi 	spmcrval, (1<<PGERS) | (1<<SELFPRGEN)
	rcall 	Do_spm
; re-enable the RWW section
	ldi 	spmcrval, (1<<RWWSRE) | (1<<SELFPRGEN)
	rjmp 	Do_spm

; Page Write
; 	WRITE ( ram flash -- )	
; 	transfer data from RAM to Flash page buffer

	COLON	5,"WRITE"
WRITE:
	movw	zl, tosl
	loadtos
	movw	xl, tosl
	loadtos
WRITE_1:
	ldi 	looplo, low(PAGESIZEB) ;init loop variable
Wrloop:
	ld 		r0, X+
	ld 		r1, X+
	ldi 	spmcrval, (1<<SELFPRGEN)
	rcall 	Do_spm
	adiw 	ZL, 2
	subi 	looplo, 2 ;use subi for PAGESIZEB<=256
	brne 	Wrloop
; execute Page Write
	subi 	ZL, low(PAGESIZEB) ;restore pointer
	sbci 	ZH, high(PAGESIZEB) ;not required for PAGESIZEB<=256
	ldi 	spmcrval, (1<<PGWRT) | (1<<SELFPRGEN)
	rcall 	Do_spm
; re-enable the RWW section
	ldi 	spmcrval, (1<<RWWSRE) | (1<<SELFPRGEN)
	rjmp 	Do_spm

; Page Read
; 	READ ( flash ram -- )	
; 	transfer data from Flash to RAM page buffer

	COLON	4,"READ"
READ:
	movw	xl,tosl
	loadtos
	movw	zl,tosl
	loadtos
READ_1:	
; read back and check, optional
	ldi 	looplo, low(PAGESIZEB) ;init loop variable
Rdloop:
	lpm 	r0, Z+
	st 		X+, r0
	subi 	looplo, 1 ;use subi for PAGESIZEB<=256
	brne 	Rdloop
	ret

Do_spm:
; check for previous SPM complete
Wait_spm:
	in 		temp1, SPMCSR
	sbrc 	temp1, SELFPRGEN
	rjmp 	Wait_spm
; SPM timed sequence
	out 	SPMCSR, spmcrval
	spm
	ret

;===============================================================
; Compiler

.org	$100

;   1+	( a -- a )
;	Add 1 to address.

	COLON	2,"1+"
ONEP:
	adiw	tosl,1
	ret

;   1-	( a -- a )
;	Subtract 1 from address.

	COLON	2,"1-"
ONEM:
	sbiw	tosl,1
	ret


;   2+	( a -- a )
;	Add cell size in byte to address.

	COLON	2,"2+"
CELLP:
	adiw	tosl,2
	ret


;   2-	( a -- a )
;	Subtract cell size in byte from address.

	COLON	2,"2-"
CELLM:
	sbiw	tosl,2
	ret

; 	>	( n1 n2 -- flag ) Compare
; 	compares two values (signed)

	COLON	1,">"
GREATER:
	ld 		temp2, Y+
	ld 		temp3, Y+
	cp 		temp2, tosl
	cpc 	temp3, tosh
	rjmp 	DGRE1

; 	D>	( d1 d2 -- flag ) Compare
; 	compares two d values (signed)

	COLON	2,"D>"
DGRE:	
	ld 		temp0, Y+
	ld 		temp1, Y+
	ld 		temp2, Y+
	ld 		temp3, Y+
	ld 		temp4, Y+
	ld 		temp5, Y+
	cp 		temp4, temp0
	cpc 	temp5, temp1
	cpc 	temp2, tosl
	cpc 	temp3, tosh
DGRE1:
	movw 	tosl,zerol
	brlt 	DGRE2
	brbs 	1, DGRE2
	sbiw 	tosl,1
	ret
DGRE2:
	ret

; 	D+	( d1 d2 -- d3) Arithmetics
; 	add double cell values

	COLON	2,"D+"
DPLUS:
	ld 		temp2, Y+
	ld 		temp3, Y+
	ld 		temp4, Y+
	ld 		temp5, Y+
	ld 		temp6, Y+
	ld 		temp7, Y+
	add 	temp2, temp6
	adc 	temp3, temp7
	adc 	tosl, temp4
	adc 	tosh, temp5
	st 		-Y, temp3
	st 		-Y, temp2
	ret

; 	D-	( d1 d2 -- d3 ) Arithmetics
; 	subtract double cell values

	COLON	2,"D-"
DMINUS:
	ld 		temp2, Y+
	ld 		temp3, Y+
	ld 		temp4, Y+
	ld 		temp5, Y+
	ld 		temp6, Y+
	ld 		temp7, Y+
	sub 	temp6, temp2
	sbc 	temp7, temp3
	sbc 	temp4, tosl
	sbc 	temp5, tosh
	st 		-Y, temp7
	st 		-Y, temp6
	movw 	tosl, temp4
	ret

;	ALLOT	( n -- )
;	Allocate n bytes to the code dictionary.

	COLON	5,"ALLOT"
ALLOT:
	CALL	DPP
	JMP		PSTOR

;   IALLOT	( n -- )
;	Allocate n bytes to the code dictionary.

	COLON	6,"IALLOT"
IALLOT:
	CALL	CPP
	JMP		PSTOR

;   ,	( w -- )
;	Compile an integer into the code dictionary.

	COLON	1,","
COMMA:
	CALL	CPP
	CALL	AT
	CALL	DUPP
	CALL	CELLP	;cell boundary
	CALL	CPP
	CALL	STORE
	JMP		ISTOR

;   call,	( ca -- )
;	Assemble a call instruction to ca.

;	COLON	5,"call,"
CALLC:
	CALL	DOLIT
	.DW		CALLL
	CALL	COMMA
	RJMP	COMMA	;328 long call

;   [COMPILE]	( -- ; <string> )
;	Compile the next immediate word into code dictionary.

	COLON	IMEDD+9,"[COMPILE]"
BCOMP:
	CALL	TICK
	CALL	TWOSL
	RJMP	CALLC

;   COMPILE	( -- )
;	Compile the next address in colon list to code dictionary.

	COLON	COMPO+7,"COMPILE"
COMPI:
	CALL	RFROM
	CALL	CELLS
	CALL	DUPP
	CALL	AT
	CALL	COMMA	;compile call instruction
	CALL	CELLP
	CALL	DUPP
	CALL	AT
	CALL	COMMA	;compile address
	CALL	CELLP
	CALL	TWOSL
	CALL	TOR
	RET				;adjust return address

;   LITERAL	( w -- )
;	Compile tos to code dictionary as an integer literal.

	COLON	7,"LITERAL"
LITER:
	CALL	DOLIT
	.DW		DOLIT
	CALL	CALLC
	RJMP	COMMA

;   $,"	( -- )
;	Compile a literal string up to next " .

;	COLON	3,'$'
;	.DB		',','"'
STRCQ:
	CALL	DOLIT
	.DW		'"'
	CALL	WORDD	;move string to code dictionary
	CALL	DUPP
	CALL	CAT
	CALL	TWOSL
	CALL	TOR
STRCQ1:
	CALL	DUPP
	CALL	AT
	CALL	COMMA
	CALL	CELLP
	CALL	DONXT
	.DW		STRCQ1
	JMP		DROP

;; Structures

;   BEGIN	( -- a )
;	Start an infinite or indefinite loop structure.

	COLON	IMEDD+5,"BEGIN"
BEGIN:
	CALL	CPP
	JMP		AT

;   FOR	( -- a )
;	Start a FOR-NEXT loop structure in a colon definition.

	COLON	IMEDD+3,"FOR"
FOR:
	CALL	DOLIT
	.DW		TOR
	CALL	CALLC
	RJMP	BEGIN

;   NEXT	( a -- )
;	Terminate a FOR-NEXT loop structure.

	COLON	IMEDD+4,"NEXT"
NEXT:
	CALL	DOLIT
	.DW		DONXT
	CALL	CALLC
	CALL	TWOSL
	RJMP	COMMA

;   UNTIL	( a -- )
;	Terminate a BEGIN-UNTIL indefinite loop structure.

	COLON	IMEDD+5,"UNTIL"
UNTIL:
	CALL	DOLIT
	.DW		QBRAN
	CALL	CALLC
	CALL	TWOSL
	RJMP	COMMA

;   AGAIN	( a -- )
;	Terminate a BEGIN-AGAIN infinite loop structure.

	COLON	IMEDD+5,"AGAIN"
AGAIN:
	CALL	DOLIT
	.DW		BRAN
	CALL	CALLC
	CALL	TWOSL
	RJMP	COMMA

;   IF	( -- A )
;	Begin a conditional branch structure.

	COLON	IMEDD+2,"IF"
IFF:
	CALL	DOLIT
	.DW		QBRAN
	CALL	CALLC
	CALL	BEGIN
	CALL	DOLIT
	.DW		2
	RJMP	IALLOT

;   AHEAD	( -- A )
;	Compile a forward branch instruction.

;	COLON	IMEDD+5,"AHEAD"
AHEAD:
	CALL	DOLIT
	.DW		BRAN
	CALL	CALLC
	CALL	BEGIN
	CALL	DOLIT
	.DW		2
	JMP		IALLOT

;   REPEAT	( A a -- )
;	Terminate a BEGIN-WHILE-REPEAT indefinite loop.

	COLON	IMEDD+6,"REPEAT"
REPEA:
	CALL	AGAIN
	CALL	BEGIN
	CALL	TWOSL
	CALL	SWAPP
	JMP		ISTOR

;   THEN	( A -- )
;	Terminate a conditional branch structure.

	COLON	IMEDD+4,"THEN"
THENN:
	CALL	BEGIN
	CALL	TWOSL
	CALL	SWAPP
	JMP		ISTOR

;   AFT	( a -- a A )
;	Jump to THEN in a FOR-AFT-THEN-NEXT loop the first time through.

	COLON	IMEDD+3,"AFT"
AFT:
	CALL	DROP
	CALL	AHEAD
	CALL	BEGIN
	JMP		SWAPP

;   ELSE	( A -- A )
;	Start the false clause in an IF-ELSE-THEN structure.

	COLON	IMEDD+4,"ELSE"
ELSEE:
	CALL	AHEAD
	CALL	SWAPP
	JMP		THENN

;   WHILE	( a -- A a )
;	Conditional branch out of a BEGIN-WHILE-REPEAT loop.

	COLON	IMEDD+5,"WHILE"
WHILE:
	CALL	IFF
	JMP		SWAPP

;   ABORT"	( -- ; <string> )
;	Conditional abort with an error message.

	COLON	IMEDD+6,"ABORT"
	.DB		'"'
ABRTQ:
	CALL	DOLIT
	.DW		ABORQ
	CALL	CALLC
	CALL	STRCQ
	RET

;   $"	( -- ; <string> )
;	Compile an inline string literal.

	COLON	IMEDD+2,'$'
	.DB		'"'
STRQ:
	CALL	DOLIT
	.DW		STRQP
	CALL	CALLC
	CALL	STRCQ
	RET

;   ."	( -- ; <string> )
;	Compile an inline string literal to be typed out at run time.

	COLON	IMEDD+2,'.'
	.DB		'"'
DOTQ:
	CALL	DOLIT
	.DW		DOTQP
	CALL	CALLC
	CALL	STRCQ
	RET

;; Name compiler

;   ?UNIQUE	( a -- a )
;	Display a warning message if the word already exists.

;	COLON	7,"?UNIQUE"
UNIQU:
	CALL	DUPP
	CALL	NAMEQ	;?name exists
	CALL	QBRAN
	.DW		UNIQ1
	CALL	DOTQP	;redefinitions are OK
	.DB		7," reDef "	;but the user should be warned
	CALL	OVER
	CALL	COUNT
	CALL	TYPES	;just in case its not planned
UNIQ1:
	JMP		DROP

;   $,n	( na -- )
;	Build a new dictionary name using the string at na.

;	COLON	3,"$,n"
SNAME:
	CALL	DUPP
	CALL	CAT	;?null input
	CALL	QBRAN
	.DW		SNAM2
	CALL	UNIQU	;?redefinition
	CALL	LAST
	CALL	AT
	CALL	COMMA	;compile link 
	CALL	CPP
	CALL	AT
	CALL	LAST
	CALL	STORE	;save new nfa in LAST	
	CALL	DUPP
	CALL	CAT
	CALL	TWOSL	;na count/2
	CALL	TOR
SNAME1:
	CALL	DUPP
	CALL	AT
	CALL	COMMA	;compile name
	CALL	CELLP
	CALL 	DONXT
	.DW		SNAME1
	JMP		DROP
SNAM2:
	CALL	STRQP
	.DB		5," name"	;null input
	JMP		ERROR

;; FORTH compiler

;   $COMPILE	( a -- )
;	Compile next word to code dictionary as a token or literal.

;	COLON	8,"$COMPILE"
SCOMP:
	CALL	NAMEQ
	CALL	QDUP	;?defined
	CALL	QBRAN
	.DW		SCOM2
	CALL	IAT
	CALL	DOLIT
	.DW		IMEDD
	CALL	ANDD	;?immediate
	CALL	QBRAN
	.DW		SCOM1
	JMP		EXECU
SCOM1:
	CALL	TWOSL
	JMP		CALLC
SCOM2:
	CALL	NUMBQ
	CALL	QBRAN
	.DW		SCOM3
	JMP		LITER
SCOM3:
	JMP		ERROR	;error

;   OVERT	( -- )
;	Link a new word into the current vocabulary.

	COLON	5,"OVERT"
OVERT:
	CALL	LAST
	CALL	AT
	CALL	CNTXT
	JMP		STORE

;   ;	( -- )
;	Terminate a colon definition.

	COLON	IMEDD+COMPO+1,";"
SEMIS:
	CALL	DOLIT
	.DW		RETT
	CALL	COMMA
	CALL	LBRAC
	JMP		OVERT

;   ]	( -- )
;	Start compiling the words in the input stream.

	COLON	1,"]"
RBRAC:
	CALL	DOLIT
	.DW		SCOMP*2
	CALL	TEVAL
	JMP		STORE

;   :	( -- ; <string> )
;	Start a new colon definition using next word as its name.

	COLON	1,":"
COLONN:
	CALL	TOKEN
	CALL	SNAME
	JMP		RBRAC

;   IMMEDIATE	( -- )
;	Make the last compiled word an immediate word.

	COLON	9,"IMMEDIATE"
IMMED:
	CALL	DOLIT
	.DW		IMEDD
	CALL	LAST
	CALL	AT
	CALL	IAT
	CALL	ORR
	CALL	LAST
	CALL	AT
	JMP		ISTOR

;; Defining words

;   CREATE	( -- ; <string> )
;	Compile a new array entry without allocating code space.

	COLON	6,"CREATE"
CREAT:
	CALL	TOKEN
	CALL	SNAME
	CALL	OVERT
 	CALL	DOLIT
	.DW		DOVAR
	CALL	CALLC
	CALL	DPP
	CALL	AT
	JMP		COMMA

;   CONSTANT	( n -- ; <string> )
;	Compile a constant.

	COLON	8,"CONSTANT"
CONST:
	CALL	TOKEN
	CALL	SNAME
	CALL	OVERT
 	CALL	DOLIT
	.DW		DOVAR
	CALL	CALLC
	JMP		COMMA

;   VARIABLE	( -- ; <string> )
;	Compile a new variable uninitialized.

	COLON	8,"VARIABLE"
VARIA:
	CALL	CREAT
	CALL	DOLIT
	.DW		2
	JMP		ALLOT

;============================================================================

.EQU	LASTN	=	_LINK*2	;last name address in name dictionary

.EQU	DTOP	=	$140	;next available memory in name dictionary
.EQU	CTOP	=	pc*2	;next available memory in code dictionary



;===============================================================




