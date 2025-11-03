ORG 00H
	
; --------------  UART SETUP --------------------
MOV TMOD , #20H
MOV TH1, #-3
MOV SCON, #50H
SETB TR1

MOV A, #'0'
ACALL CHECK_HEX_INP
;- - - - - - - - - Init  - - - - - - - - - - 
MOV A,#38H               ; 5x7 matrix LCD init
ACALL COMMAND_WRITE
MOV A,#0CH               ; Disp ON , cursor OFF
ACALL COMMAND_WRITE
MOV A,#01H               ; Clear LCD
ACALL COMMAND_WRITE
;MOV A,#06H               ; Shift cursor right 
;ACALL COMMAND_WRITE
;----------------FSM setup------------------ 
; R0:  address
MOV R1, #00H ; mode of reception
MOV R2, #00H ; mode of  location pointer
MOV R3, #00H ; number of char recieved
MOV R4, #00H ; location 1
MOV R5, #00H ; location 2


MOV P0, #00H

WAIT_4INP: 
CLR RI
RPT: JNB RI, RPT
MOV A,SBUF

CJNE R1, #01H , SKIP_GET_MODE
	; get data
	CJNE A, #'`', SKIP1 ; complete data : newline / line feed
		DUMP:
		ACALL TOGGLE 
		
		MOV A, R3
		JZ SKIP_all

		MOV A, R4
		ADD A, R5           ; got the address
		ACALL COMMAND_WRITE

		;---------------------------
		MOV R0, #80H
		MOV A, R3
		MOV R4, A
		
		REVERSE_STACK:
		POP acc
		INC R0
		MOV @R0, A
		DJNZ R4, REVERSE_STACK
		;---------------------------

		DUMP_NEXT:
		MOV A, @R0
		ACALL DATA_WRITE
		DEC R0
		DJNZ R3, DUMP_NEXT

		MOV R1, #00H
		MOV R2, #00H
		MOV R3, #00H
		MOV R4, #00H
		MOV R5, #00H
		MOV SP, #08H
		SJMP WAIT_4INP
SKIP1:
		CJNE R2, #00H , SKIP2
		ACALL CHECK_HEX_INP
		CJNE R0,#00H, WAIT_4INP
		;---------------
		CJNE A, #00H, NEXT_LINE
			MOV A, #80H
			SJMP LINE_OK
		NEXT_LINE:
			MOV A, #0C0H
		LINE_OK:
		MOV R4, A ; get location 1
		MOV R2, #01H
		ACALL TOGGLE
		LJMP WAIT_4INP
SKIP2:
		CJNE R2, #01H , SKIP3
		ACALL CHECK_HEX_INP
		CJNE R0,#00H, WAIT_4INP
		;---------------
		MOV R5, A ; get location 2
		MOV R2, #02H
		ACALL TOGGLE
		LJMP WAIT_4INP
SKIP3:
		CJNE R2, #02H, SKIP_all
		CJNE R3, #10H, SKIP_dump1
			SJMP DUMP
		SKIP_dump1:
		ACALL TOGGLE 
		MOV R7, A
		MOV A, R3
		ADD A, R5
		CJNE A, #10H, SKIP_DUMP2
			LJMP DUMP
		SKIP_DUMP2:
		PUSH 7
		INC R3
		LJMP WAIT_4INP
SKIP_all:
		MOV R1, #00H ; mode of reception
		MOV R2, #00H ; mode of  location pointer
		MOV R3, #00H ; number of char recieved
		MOV R4, #00H ; location 1
		MOV R5, #00H ; location 2
		LJMP WAIT_4INP
		
SKIP_GET_MODE: 
	CJNE A, #'~', NEXT1  ;  start storing all data
		MOV R1, #01H
		MOV R2, #00H
		MOV R3, #00H
		MOV R4, #00H
		MOV R5, #00H
		MOV SP, #08H
		ACALL TOGGLE
		LJMP WAIT_4INP
	NEXT1:
		CJNE A, #7FH, NEXT2  ; clear screen upon getting delete
		ACALL CLR_LCD
		MOV R1, #00H
		MOV R2, #00H
		MOV R3, #00H
		MOV R4, #00H
		MOV R5, #00H
		MOV SP, #08H
		LJMP WAIT_4INP
	NEXT2:                   ; garbage value entered
	
		MOV R1, #00H ; mode of reception
		MOV R2, #00H ; mode of  location pointer
		MOV R3, #00H ; number of char recieved
		MOV R4, #00H ; location 1
		MOV R5, #00H ; location 2
		LJMP WAIT_4INP

;- - - - - - - - - Dump - - - - - - - - - - 


;- - - - - - - - -Clear LCD - - - - - - - - 
CLR_LCD: 
MOV A,#01H               ; Clear LCD
ACALL COMMAND_WRITE
RET

;- - - - - - - - - - - - - - - - - - - - - - - - 
;P2 = PORT
;RS = P3.7
;RW = P3.6
;EN = P3.5

COMMAND_WRITE:
ACALL IS_READY
MOV P2,A      ; shift data to port
CLR P3.7      ; RS=0 for CMD
CLR P3.6      ; R/W = 0 to write
SETB P3.5     ; En = 1 
CLR P3.5      ; En = 0: H-> L pulse
RET

DATA_WRITE:
ACALL IS_READY
MOV P2,A        ; shift data to port
SETB P3.7       ; RS=0 for data
CLR P3.6        ; R/W = 0 to write
SETB P3.5       ; En = 1 
CLR P3.5        ; En = 0: H-> L pulse
RET

IS_READY:
SETB P2.7       ; make 7th bit an input port
CLR P3.7        ; RS=0 access CMD reg
SETB P3.6        ; R/W = 1 to read CMD reg
BACK: CLR P3.5       ; En = 0 
ACALL DELAY
SETB P3.5        ; En = 1: L->H pulse 
JB P2.7, BACK    ; stay untill busy flag = 0
RET

DELAY:   
MOV R7,#25H
HERE: DJNZ R7,HERE
RET

TOGGLE:
CJNE R6,#00H, TOGGLE_HIGH
MOV R6, #01H
SETB P0.1 
SJMP TOGGLE_DONE
TOGGLE_HIGH: 
MOV R6, #00H
CLR P0.1
TOGGLE_DONE:
RET

CHECK_HEX_INP:
MOV R7, A
SUBB A, #'0'
JC BAD_NUM
MOV A, R7
SUBB A, #3AH  ; 9= 39H, this way we pass '9'
JNC BAD_NUM
CLR CY
MOV A, R7
SUBB A,#'0'
MOV R0, #00H
SJMP DONE_CHECK_INP

BAD_NUM:
MOV A, R7
SUBB A, #'A'
JC BAD_ALPHA_BIG
MOV A, R7
SUBB A, #'F'    ; don't need to include F
JNC BAD_ALPHA_BIG
CLR CY
MOV A, R7
SUBB A,#'A'
ADD A, #0AH
MOV R0, #00H
SJMP DONE_CHECK_INP

BAD_ALPHA_BIG:
MOV A, R7
SUBB A, #'a'
JC BAD_ALPHA_NUM
MOV A, R7
SUBB A, #'f'   ; don't need to include f
JNC BAD_ALPHA_NUM
CLR CY
MOV A, R7
SUBB A,#'a'
ADD A, #0AH
MOV R0, #00H
SJMP DONE_CHECK_INP

BAD_ALPHA_NUM:
MOV A, #0FFH
MOV R0, #0FFH
MOV R1, #00H ; mode of reception
MOV R2, #00H ; mode of  location pointer
DONE_CHECK_INP:
RET

END
