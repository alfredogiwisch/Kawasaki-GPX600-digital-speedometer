# Kawasaki-GPX600-digital-speedometer
Development of a digital speedometer in a Kawasaki GPX600

I begin the 2010 year with a new development for my motorcycle. A digital panel to add new features giving a new look. Using the power and flexibility of 8 bit microcontrolers it was easy to complete the project development in short time.
Next pictures shows a view of the the digital tachometer, digital gear position indicator and digital speedometer circuit boards. At the same time I add a bar-graph led acting as voltmeter for measuring the charge status of the battery aka the charge voltage.

Latest update 29 March 2013:

Below is the speedometer program in assembler language for everyone who want to build it. The speedometer module use a PIC 16F628 microcontroller, same processor are used on the tachometer board. The counter is set for a sensor on the wheel with a pulse every 60 grads of rotation. The device has two up/down buttons to adjust the counter via GPS.

The 16F628 is backward compatible with previous 16F84 microcontroller.
---------------------------------------------------------------------------------------------------------
LIST      P=16F628, F=INHX8M
        include "P16F628.inc"
        __CONFIG 0x3D50

        ERRORLEVEL    0,    -302    ;Suppress bank selection messages
        ERRORLEVEL    0,    -207    ;Suppress "label found after column 1"


;        ================================
;            Definitions
;        ================================

#define        Disp1        PortA,    0    ;Ones
#define        Disp2        PortA,    1    ;Tens
#define        Disp3        PortA,    2    ;Hundreds
#define        UpBut        PortA,    4    ;Up button
#define        DownBut        PortA,    5    ;Down button


        SegA        Equ    1    ;7-Segment display
        SegB        Equ    7    ;7-Segment display
        SegC        Equ    5    ;7-Segment display
        SegD        Equ    2    ;7-Segment display
        SegE        Equ    4    ;7-Segment display
        SegF        Equ    0    ;7-Segment display
        SegG        Equ    3    ;7-Segment display

        Seg7Disp    Equ    PortB    ;Led 7-segment display

;        === Definitions End ===

;        ================================
;           General Purpose Registers
;        ================================

        cblock    0x20

            delay            ;Delay
            delaya            ;Delay
            delayb            ;Delay

            W_Temp            ;Interrupt
            S_Temp            ;Interrupt
            P_Temp            ;Interrupt

            Disp1Out        ;Display1's byte
            Disp2Out        ;Display2's byte
            Disp3Out        ;Display2's byte

            NumH            ;Convert routine
            NumL            ;Convert routine
            TenK            ;Convert routine
            Thou            ;Convert routine
            Hund            ;Convert routine
            Tens            ;Convert routine
            Ones            ;Convert routine

            CountDelay        ;Measurement delay
            CountDelayL        ;Register to loop delay with

        endc

;        === GPR's End ===


;        ================================
;            Program Start
;        ================================

        org    0x0000
        clrf    status            ;Init status register
        goto    Initialize        ;Start initializing


;        ================================
;              Interrupt
;        ================================

Interrupt    org    0x0004

        movwf    W_Temp            ;Save W register
        swapf    status,    w        ;Swap status to be saved into W
        movwf    S_Temp            ;Save STATUS register
        movfw    PCLATH
        movwf    P_Temp            ;Save PCLATH

        btfss    PIR1,    TMR2IF        ;Was it TMR2 interrupt?
        goto    ExitInterrupt        ;Exit if not
        bcf    PIR1,    TMR2IF        ;Clear flag

TestDisp    btfsc    Disp3            ;Test if Disp3 last updated
        goto    UpdateDisp1        ;If yes (clear) update Disp1 next
        btfsc    Disp1            ;Test if Disp1 last updated
        goto    UpdateDisp2        ;If yes (clear) update Disp2 next
        btfsc    Disp2            ;Test if Disp2 last updated
        goto    UpdateDisp3        ;If yes (clear) update Disp3 next
        bsf    Disp3            ;If none yet updated, start with Disp1
        goto    TestDisp        ;To DispTesting start

UpdateDisp1    movf    Disp1Out,    w    ;Disp1 variable to W
        bcf    Disp3            ;Disp3 OFF
        bcf    Disp2            ;Disp2 OFF (precaution)
        movwf    Seg7Disp        ;W to Outputport
        bsf    Disp1            ;Disp1 ON
        goto    ExitInterrupt        ;Resume main program

UpdateDisp2    movf    Disp2Out,    w    ;Disp2 variable to W
        bcf    Disp1            ;Disp1 OFF
        bcf    Disp3            ;Disp3 OFF (precaution)
        movwf    Seg7Disp        ;W to Outputport
        bsf    Disp2            ;Disp2 ON
        goto    ExitInterrupt        ;Resume main program

UpdateDisp3    movf    Disp3Out,    w    ;Disp3 variable to W
        bcf    Disp2            ;Disp2 OFF
        bcf    Disp1            ;Disp1 OFF (precaution)
        movwf    Seg7Disp        ;W to Outputport
        bsf    Disp3            ;Disp3 ON
        goto    ExitInterrupt        ;Resume main program

ExitInterrupt    movfw    P_Temp
        movwf    PCLATH            ;Restore PCLATH
        swapf    S_Temp,    w
        movwf    STATUS            ;Restore status, restores bank
        swapf    W_Temp,    f
        swapf    W_Temp,    w        ;Restore W
        retfie     

;        === Interrupt End ===


;        ================================
;                Tables
;        ================================

Seg7Number    addwf    pcl,    f        ;To convert 0-9 to corresponding literal
        retlw    b'01001000'    ;0
        retlw    b'01011111'    ;1
        retlw    b'01100001'    ;2
        retlw    b'01010001'    ;3
        retlw    b'01010110'    ;4
        retlw    b'11010000'    ;5
        retlw    b'11000000'    ;6
        retlw    b'01011101'    ;7
        retlw    b'01000000'    ;8
        retlw    b'01010000'    ;9
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null
        retlw    b'00000000'    ;Null

;        === Tables End ===


;        ================================
;             Initializations
;        ================================

Initialize   
        ;= Comparators =
        movlw 0x07             ;Comparators = OFF
        movwf CMCon

        ;= Ports =
        movlw    b'00000000'        ;PNP transistors OFF
        movwf    PortA
        clrf    PortB
        bsf    status, rp0        ;Bank1
        movlw    b'00110000'        ;All outs except RA4, RA5
        movwf    TrisA
        movlw    b'01000000'        ;All outs except T1 Clock(RB6) = IN
        movwf    TrisB
        bcf    status, rp0        ;Bank0

        ;= Timer1 =
        movlw    b'00000110'        ;External clock, stopped
        movwf    T1Con
        clrf    TMR1H            ;Zero timer
        clrf    TMR1L

        ;= Timer2 =
        movlw    b'01111001'        ;Postscale = 16, Prescale = 4
        movwf    T2Con
        clrf    TMR2            ;Zero timer
        bsf    status, rp0        ;Bank1
        movlw    d'62'            ;d'156'            ;62µS*16*4 = 3,968mS Timeout (252Hz Refreshrate)
        movwf    pr2
        bcf    status, rp0        ;Bank0

        ;= Interrupts =
        bsf    status, rp0        ;Bank1
        movlw    b'10001111'        ;Setup option_reg
        movwf    option_reg
        movlw    b'11000000'        ;GIE and Peripheral interrupts = ON
        movwf    IntCon
        movlw    b'00000010'        ;Enable Timer2 Interrupt
        movwf    PIE1
        bcf    status, rp0        ;Bank0

        clrf    Disp1Out        ;To ensure that no rubbish is displayed
        clrf    Disp2Out
        clrf    Disp3Out

        clrf    NumH            ;Ensure that no crap is displayed at startup
        clrf    NumL
        clrf    TenK
        clrf    Thou
        clrf    Hund
        clrf    Tens
        clrf    Ones

        movlw    d'141'
        movwf    CountDelay

        bsf    T2Con,    TMR2On        ;Start Timer2 and the display refreshing
        call    Intro

        goto    Main

;        === Initializations End ===


;        ================================
;             Main Program
;        ================================

Main        clrf    Tmr1H            ;Zero timer
        clrf    Tmr1L
        bsf    T1Con,    Tmr1On        ;Start counting
        call    SpeedDelay        ;Delay for measuring
        bcf    T1Con,    Tmr1On        ;Stop counting

        movf    Tmr1H,    w        ;High bit to w
        movwf    NumH            ;And then to NumH
        movf    Tmr1L,    w        ;Low bit to w
        movwf    NumL            ;And then to NumH

        call    RefreshDisp        ;Update display

        btfss    UpBut            ;Is Up button pressed?
        goto    DelayUp            ;Increase delay
        btfss    DownBut            ;Is Down button pressed?
        goto    DelayDown        ;Decrease delay
        goto    Main            ;Loop


DelayUp        call    Delay10            ;Wait to debounce
        btfsc    UpBut            ;Still pressed?
        goto    Main            ;NO: goto main
        incfsz    CountDelay,    f    ;Increment Delay
        goto    $+2
        decf    CountDelay,    f    ;If rolls over, decrement back to prevoius
        clrf    NumH
        movf    CountDelay,    w    ;Delaycount to W
        movwf    NumL            ;And move into NumL
        call    RefreshDisp        ;Show delay's length
        call    Delay100        ;Wait for 100mS
        btfss    UpBut            ;Is button still pressed?
        goto    $-1            ;Wait for release
        goto    Main            ;And count again

DelayDown    call    Delay10            ;Wait to debounce
        btfsc    DownBut            ;Still pressed?
        goto    Main            ;NO: goto main
        decfsz    CountDelay,    f    ;Decrement Delay
        goto    $+2
        incf    CountDelay,    f    ;If rolls over, increment back to prevoius
        clrf    NumH
        movf    CountDelay,    w    ;Delaycount to W
        movwf    NumL            ;And move into NumL
        call    RefreshDisp        ;Show delay's length
        call    Delay100        ;Wait for 100mS
        btfss    DownBut            ;Is button still pressed?
        goto    $-1            ;Wait for release
        goto    Main            ;And count again


RefreshDisp    call    Convert            ;Convert to BCD

        movf    Ones,    w        ;First digit to w
        call    Seg7Number        ;Fetch port equalent
        movwf    Disp1Out        ;And show on first digit

        movf    Hund,    w        ;Digit3 to W, modify Z
        btfsc    status,    z        ;Test if Disp3 = Zero
        goto    Dis2StripZero        ;Yes, Zero
                        ;Not zero   
Dis2NoStripZero    movf    Tens,    w        ;Second digit to W, and adjust Z bit
        call    Seg7Number        ;No: Fetch port equalent
        movwf    Disp2Out        ;And show on second digit
        goto    Disp3Update        ;Update display3 next

Dis2StripZero    movf    Tens,    w        ;Second digit to W, and adjust Z bit
        btfsc    status,    z        ;Is z set?
        movlw    d'10'            ;Yes: Put 10 in W, this will return Null from table
        call    Seg7Number        ;No: Fetch port equalent
        movwf    Disp2Out        ;And show on second digit

Disp3Update    movf    Hund,    w        ;Third digit to w
        btfsc    status,    z        ;Is z set?
        movlw    d'10'            ;Yes: Put 10 in W, this will return Null from table
        call    Seg7Number        ;No: Fetch port equalent
        movwf    Disp3Out        ;And show on third digit

        return

;        === Main End ===


;        ================================
;               Intro
;        ================================


Intro        bsf    Disp1Out,    SegD    ;Fill whole screen from bottom up
        call    Delay50
        bsf    Disp2Out,    SegD
        call    Delay50
        bsf    Disp3Out,    SegD
        call    Delay50
        bsf    Disp3Out,    SegE
        call    Delay50
        bsf    Disp3Out,    SegC
        call    Delay50
        bsf    Disp2Out,    SegE
        call    Delay50
        bsf    Disp2Out,    SegC
        call    Delay50
        bsf    Disp1Out,    SegE
        call    Delay50
        bsf    Disp1Out,    SegC
        call    Delay50
        bsf    Disp1Out,    SegG
        call    Delay50
        bsf    Disp2Out,    SegG
        call    Delay50
        bsf    Disp3Out,    SegG
        call    Delay50
        bsf    Disp3Out,    SegF
        call    Delay50
        bsf    Disp3Out,    SegB
        call    Delay50
        bsf    Disp2Out,    SegF
        call    Delay50
        bsf    Disp2Out,    SegB
        call    Delay50
        bsf    Disp1Out,    SegF
        call    Delay50
        bsf    Disp1Out,    SegB
        call    Delay50
        bsf    Disp1Out,    SegA
        call    Delay50
        bsf    Disp2Out,    SegA
        call    Delay50
        bsf    Disp3Out,    SegA
        call    Delay50

        bcf    Disp1Out,    SegD    ;Empty whole screen from bottom up
        call    Delay50
        bcf    Disp2Out,    SegD
        call    Delay50
        bcf    Disp3Out,    SegD
        call    Delay50
        bcf    Disp3Out,    SegE
        call    Delay50
        bcf    Disp3Out,    SegC
        call    Delay50
        bcf    Disp2Out,    SegE
        call    Delay50
        bcf    Disp2Out,    SegC
        call    Delay50
        bcf    Disp1Out,    SegE
        call    Delay50
        bcf    Disp1Out,    SegC
        call    Delay50
        bcf    Disp1Out,    SegG
        call    Delay50
        bcf    Disp2Out,    SegG
        call    Delay50
        bcf    Disp3Out,    SegG
        call    Delay50
        bcf    Disp3Out,    SegF
        call    Delay50
        bcf    Disp3Out,    SegB
        call    Delay50
        bcf    Disp2Out,    SegF
        call    Delay50
        bcf    Disp2Out,    SegB
        call    Delay50
        bcf    Disp1Out,    SegF
        call    Delay50
        bcf    Disp1Out,    SegB
        call    Delay50
        bcf    Disp1Out,    SegA
        call    Delay50
        bcf    Disp2Out,    SegA
        call    Delay50
        bcf    Disp3Out,    SegA
        call    Delay50
        clrf    Disp1Out
        clrf    Disp2Out
        clrf    Disp3Out
        call    Delay50

        bsf    Disp3Out,    SegA    ;Roll from top left to down right
        bsf    Disp3Out,    SegF
        call    Delay50
        bcf    Disp3Out,    SegA
        bcf    Disp3Out,    SegF
        bsf    Disp3Out,    SegE
        bsf    Disp2Out,    SegA
        call    Delay50
        bcf    Disp3Out,    SegE
        bcf    Disp2Out,    SegA
        bsf    Disp3Out,    SegD
        bsf    Disp1Out,    SegA
        call    Delay50
        bcf    Disp3Out,    SegD
        bcf    Disp1Out,    SegA
        bsf    Disp1Out,    SegB
        bsf    Disp2Out,    SegD
        call    Delay50
        bcf    Disp1Out,    SegB
        bcf    Disp2Out,    SegD
        bsf    Disp1Out,    SegC
        bsf    Disp1Out,    SegD
        call    Delay50
        bcf    Disp1Out,    SegC
        bcf    Disp1Out,    SegD
        call    Delay50
        return

;        === Intro End ===


;        ================================
;               Delays
;        ================================

SpeedDelay    movf    CountDelay,    w    ;Countdelay to W
        movwf    CountDelayL        ;And W to loop register
        call    Delay255
        call    Delay255
        call    Delay255
SpeedDelayL    call    Delay5            ;Basedelay
        decfsz    CountDelayL,    f    ;Decrement loop register
        goto    SpeedDelayL        ;While not 0 loop
        return                ;When zero, return


Delay255    movlw    0xff            ;delay 255 mS
        goto    d0
Delay100    movlw    d'100'            ;delay 100mS
        goto    d0
Delay50        movlw    d'50'            ;delay 50mS
        goto    d0
Delay10        movlw    d'10'            ;delay 10mS
        goto    d0
Delay5        movlw    0x05            ;delay 5.000 ms (4 MHz clock)
d0        movwf    delay
d1        movlw    0xC7
        movwf    delaya
        movlw    0x01
        movwf    delayb
Delay_0        decfsz    delaya, f
        goto    $+2
        decfsz    delayb, f
        goto    Delay_0
        decfsz    delay    ,f
        goto    d1
        return

;        === Delays End ===


;        ================================
;            Convert routine
;        ================================
;              (from www.piclist.com)
;        GPRs:    NumH
;            NumL    Takes Number in
;            TenK    NumH:NumL and
;            Thou    returns value
;            Hund    in decimal
;            Tens
;            Ones

Convert        swapf    NumH, w
        iorlw    B'11110000'
        movwf    Thou
        addwf    Thou,f
        addlw    0XE2
        movwf    Hund
        addlw    0X32
        movwf    Ones
        movf    NumH,w
        andlw    0X0F
        addwf    Hund,f
        addwf    Hund,f
        addwf    Ones,f
        addlw    0XE9
        movwf    Tens
        addwf    Tens,f
        addwf    Tens,f
        swapf    NumL,w
        andlw    0X0F
        addwf    Tens,f
        addwf    Ones,f
        rlf    Tens,f
        rlf    Ones,f
        comf    Ones,f
        rlf    Ones,f
        movf    NumL,w
        andlw    0X0F
        addwf    Ones,f
        rlf    Thou,f
        movlw    0X07
        movwf    TenK
        movlw    0X0A
Lb1        addwf    Ones,f
        decf    Tens,f
        btfss    3,0
        goto    Lb1
Lb2        addwf    Tens,f
        decf    Hund,f
        btfss    3,0
        goto    Lb2
Lb3        addwf    Hund,f
        decf    Thou,f
        btfss    3,0
        goto    Lb3
Lb4        addwf    Thou,f
        decf    TenK,f
        btfss    3,0
        goto    Lb4
        return

;        === Convert End ===

        end

,        === Program End ==

