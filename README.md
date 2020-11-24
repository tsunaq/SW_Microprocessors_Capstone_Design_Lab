# SW_Microprocessors_Capstone_Design_Lab
 
## 1. 설계목적
- PIC16F876A를 이용하여 디지털 시계를 제작하기  
- 어셈블리어를 이용한 알고리즘 구현하기

## 2. 시스템 동작 내용(HW/SW)
### 2-1. HW 구성 및 동작

![1](https://user-images.githubusercontent.com/58457978/100061682-c5d22d80-2e71-11eb-9c17-b98691cb2e93.png)   
그림 1 전체 회로 구성 (전면, 후면)

|SW A|SW B|SW C|
|---|---|---|
|RB 3|RB 4|RB 5|  

표 1 스위치와 PIC 칩의 연결도

|RC 7|RC 6|RC 5|RC 4|RC 3|RC 2|RC 1|RC 0|
|---|---|---|---|---|---|---|---|
|dp|g|f|e|d|c|b|a|  

표 2 세그먼트와 PIC 칩의 연결도

|DG 1|DG 2|DG 3|DG 4|
|---|---|---|---|
|RA 3|RA 2|RA 1|RA 0|  

표 3 각 digit와 PIC 칩의 연결도

### 2-2. SW 구성 및 동작
- 전체적인 프로그램은 기본적으로 시간을 생성하는 메인 루틴을 수행하고, 그 사이에 TMR0 overflow 인터럽트가 발생하면 디스플레이(DISP) 할 단자를 결정하며 버튼입력(BTN_CHK)을 확인하여 작동하고자 하는 Flag를 변경한다. 인터럽트가 발생하지 않았을 때에는 지정된 Flag를 확인하여 디스플레이 값을 유지하거나 변경하도록 구성되어있다.

1) 스위치 동작
- 인터럽트 수행 중 버튼입력을 확인하고 일정 시간동안 지속된 버튼입력을 수행하도록 하였다. 

2) 디스플레이
- 인터럽트가 수행되면 FLAG의 하위 2비트를 이용해 4개의 분기로 나누어 출력될 DIGIT을 선택하도록 설계하였다.

3) 시간 구현
- TMR0와 내부 클럭과 프리 스케일러를 이용하여 타이머를 작동시키고, TMR0의 오버플로우를 이용한 인터럽트가 4회 발생시 시간 측정을 위한 카운트를 1 증가시키고, 이 카운트가 122회 반복되면 1초가 증가하게 하였다. 1초 카운트가 10이 되면 10의 자리 초가 증가하고 그 초가 7이 되면 1분의 자리가 1 증가한다. 이를 반복해 24시까지 표현되도록 하였다.

4) 각각의 모드와 작동
- 모드는 가장 기본이 되는 시간표현(시간:분), 초만 표시하는 모드, 시각설정모드가 있다.
시간:분(00:00)을 표시하던 상태에서 초(XX:00)만 표시하는 상태로 다시 현재시각 설정모드로 이동할 수 있다. 각각의 모드들은 SW A의 동작으로 MENU_FLAG를 변화시켜 접근 가능하며 시간설정 모드에서는 SW B를 이용해 DIGIT을 변경하고, SW C를 이용해 각 DIGIT의 인수들을 증가시킬 수 있다.

## 3. 기능 및 동작과정

### 3-1. 인터럽트 및 MENU_FLAG를 이용한 분기

```assembly
   ORG   0004H

   MOVWF   W_TEMP
   
   SWAPF   STATUS,W
   MOVWF   STATUS_TEMP
   MOVLW   B'00000000'
   CALL   DISP
   CALL   BTN_CHK
   SWAPF   STATUS_TEMP,W
   MOVWF   STATUS
   SWAPF   W_TEMP,F
   SWAPF   W_TEMP,W
   BCF   INTCON,2
   
   RETFIE
   
   CHK_FLAG
   BTFSC   MENU_FLAG,0
   GOTO   CLK_MODE      ;CLK MODE OR SETTING
   
   CLK_MODE
   
   BTFSC   MENU_FLAG,7
   GOTO   CLK_MODE_SECOND
   BTFSC   MENU_FLAG,4
   GOTO   SETTING_MODE_1
```

### 3-2. 디스플레이

```assembly
DISP
   BTFSS   FLAG,0
   GOTO   SUB1
   GOTO   SUB2
   
SUB1
   BTFSS   FLAG,1
   GOTO   DISP1
   GOTO   DISP3

SUB2
   BTFSS   FLAG,1
   GOTO   DISP2
   GOTO   DISP4
            
DISP1
   ;BTFSC   CLK_SET_FLAG,2
   ;CALL   BLINK

   MOVF   DISP_BUF_2,W
   CALL   CONV
   BSF   PORTA,0
   BSF   PORTA,2
   BSF   PORTA,3
   MOVWF   PORTC
   BCF   PORTA,1   
   BSF   FLAG,0
   BCF   FLAG,1
   
   RETURN


DISP2
   MOVF   DISP_BUF_1,W
   CALL   CONV
   BSF   PORTA,1
   BSF   PORTA,2
   BSF   PORTA,3
   MOVWF   PORTC
   BCF   PORTA,0
   BCF   FLAG,0
   BSF   FLAG,1
   INCF   INT_CNT
   RETURN
   
DISP3
   MOVF   DISP_BUF_4,W
   CALL   CONV
   BSF   PORTA,0
   BSF   PORTA,1
   BSF   PORTA,2
   MOVWF   PORTC
   BCF   PORTA,3
   BSF   FLAG,0
   BSF   FLAG,1
   RETURN

DISP4
   MOVF   DISP_BUF_3,W
   CALL   CONV
   BSF   PORTA,0
   BSF   PORTA,1
   BSF   PORTA,3
   MOVWF   PORTC
   
   BTFSS   MENU_FLAG,4
   CALL   BLINK

   
   BCF   PORTA,2
   BCF   FLAG,0
   BCF   FLAG,1
   RETURN
   
   BLINK
   MOVLW   .0
   SUBWF   INT_CNT,W
   BTFSC   STATUS,ZF
   CALL   DOT
   
   MOVLW   .61
   SUBWF   INT_CNT,W
   BTFSC   STATUS,ZF
   CALL   DOT
   
   RETURN
```

### 3-3. 버튼 확인 및 MENU_FLAG 변경

```assembly
BTN_CHK
      MOVF   PORTB,W
      ANDLW   B'00111000'
      SUBLW   B'00111000'
   BTFSC   STATUS,ZF
      RETURN      
      MOVLW   .50
      SUBWF   BTNDEL,W
      BTFSS   STATUS,ZF
      GOTO   SW_VALID
      CLRF   BTNVALID
      MOVF   PORTB,W
      ANDLW   B'00111000'
      MOVWF   BTNVALID
      GOTO   SW_VALID
   
SW_VALID
      DECFSZ   BTNDEL,F
      RETURN

      MOVLW   .100
      MOVWF   BTNDEL

     MOVF   PORTB,W
      ANDLW   B'00111000'
      SUBWF   BTNVALID,W
      BTFSS   STATUS,ZF
      RETURN
   
      MOVF   BTNVALID,W
      SUBLW   B'00110000'   ;A
      BTFSC   STATUS,ZF
      GOTO   SW_A

      MOVF   BTNVALID,W
      SUBLW   B'00101000'   ;B
      BTFSC   STATUS,ZF
      GOTO   SW_B

      MOVF   BTNVALID,W
      SUBLW   B'00011000'   ;C
      BTFSC   STATUS,ZF
      GOTO   SW_C

;      MOVF   BTNVALID,W
;      SUBLW   B'00001000'   ;B+C
;      BTFSC   STATUS,ZF
;      GOTO   SW_BC
;      RETURN



SW_A
   BTFSC   MENU_FLAG,0
   CALL   SW_A_SUB1
   
   
   CALL   SELECT_MODE
   RETURN

SW_A_SUB1
   ;GOTO   SW_A_SUB1_SUB
   MOVLW   .1
   BTFSC   MENU_FLAG,7
   MOVLW   .2
   BTFSC   MENU_FLAG,4
   MOVLW   .0
      
   RETURN
      
   
SW_B
   BTFSC   MENU_FLAG,0
   CALL   SW_B_SUB1
      
   CALL   SELECT_MODE
   RETURN

SW_B_SUB1
   BTFSC   MENU_FLAG,4
   MOVLW   .3
   RETURN
   
SW_C
   BTFSC   MENU_FLAG,0
   CALL   SW_C_SUB1
   
   CALL   SELECT_MODE
   RETURN
   
SW_C_SUB1
   BTFSC   MENU_FLAG,4
   MOVLW   .4
   RETURN
   
SELECT_MODE
      CALL   SELECT_DETAIL
      ANDLW   B'11111111'
      MOVWF   MENU_FLAG
      RETURN
      
SELECT_DETAIL
      ANDLW   0FH         
      ADDWF   PCL,F   
      RETLW   B'00000001'   ;0
      RETLW   B'10000001'   ;1 초모드
      RETLW   B'00010001'   ;2 시간설정
      RETLW   B'00110001'   ;3 자리변경
      RETLW   B'01010001'   ;4 인수증가
```

### 3-4. 시간생성
```assembly
M_LOOP
   MOVLW   .122
   SUBWF   INT_CNT,W
   BTFSS   STATUS,ZF
   GOTO   CHK_FLAG
   
CK_LOOP
   CLRF   INT_CNT
   INCF   D_1SEC
   MOVLW   .10
   SUBWF   D_1SEC,W
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   
T_10S
   CLRF   D_1SEC
   INCF   D_10SEC
   MOVLW   .6
   SUBWF   D_10SEC,W
   BTFSS   STATUS,ZF
   GOTO   M_LOOP

T_1M   
   CLRF   D_10SEC
   INCF   D_1MIN
   MOVLW   .10
   SUBWF   D_1MIN,W
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   
T_10M
   CLRF   D_1MIN
   INCF   D_10MIN
   MOVLW   .6
   SUBWF   D_10MIN,W
   BTFSS   STATUS,ZF
   GOTO    M_LOOP

T_1H   
   CLRF   D_10MIN
   INCF   D_1HOUR,F
   BSF   PORTA,5
   CALL   DELAY
   BCF   PORTA,5
   BCF   STATUS,ZF
   MOVLW   .10
   SUBWF   D_1HOUR,W
   BTFSC   STATUS,ZF
   GOTO   T_10H
   MOVLW   .2
   SUBWF   D_10HOUR,W
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   MOVLW   .4
   SUBWF   D_1HOUR,W
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   
T_10H
   CLRF   D_1HOUR
   INCF   D_10HOUR,F
   MOVLW   .3
   SUBWF   D_10HOUR,W
   BTFSC   STATUS,ZF
   CLRF   D_10HOUR
   GOTO   M_LOOP
```

### 3-5. 현재시각 설정
```assembly
SETTING_MODE_1
   BTFSC   SET_FLAG,0
   GOTO   SETTIME_1
   BSF   SET_FLAG,0
      GOTO   SETTIME_1
   

SETTIME_1

   BTFSC   MENU_FLAG,7
   GOTO   CLK_MODE
   
   MOVF   D_1MIN,W
   MOVWF   DISP_BUF_1

   MOVF   D_10MIN,W
   MOVWF   DISP_BUF_2

   MOVF   D_1HOUR,W
   MOVWF   DISP_BUF_3

   MOVF   D_10HOUR,W
   MOVWF   DISP_BUF_4
   
   
   BTFSC   MENU_FLAG,5   ;select dgit
   CALL   SETTING_SELECT

   BTFSC   MENU_FLAG,6   ;inc dgit
   GOTO   INC_DIGIT
   
   GOTO   M_LOOP
   
SETTING_SELECT   ;변경
   MOVLW   .33
   MOVWF   TEMP1
   BCF   MENU_FLAG,5
   RLF   CLK_SET_FLAG,F
   BTFSS   CLK_SET_FLAG,4
   RETURN
   MOVLW   .1
   MOVWF   CLK_SET_FLAG
   RETURN

INC_DIGIT ; 0~9, 0~5, 0~2
   MOVLW   .44
   MOVWF   TEMP2
   BCF   MENU_FLAG,6
   BTFSC   CLK_SET_FLAG,0;1분
   GOTO   INC_10
   BTFSC   CLK_SET_FLAG,1;10분
   GOTO   INC_6
   BTFSC   CLK_SET_FLAG,2;1시
   GOTO   INC_10_2
   BTFSC   CLK_SET_FLAG,3;10시
   GOTO   INC_3
   GOTO   M_LOOP
INC_10 ;0~9
   INCF   D_1MIN,F
   MOVF   D_1MIN,W
   SUBLW   .10
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   CLRF   D_1MIN
   GOTO   M_LOOP
INC_6 ;0~5
   INCF   D_10MIN,F
   MOVF   D_10MIN,W
   SUBLW   .6
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   CLRF   D_10MIN
   GOTO   M_LOOP
INC_10_2 ;0~9
   INCF   D_1HOUR,F
   MOVF   D_1HOUR,W
   SUBLW   .10
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   CLRF   D_1HOUR
   GOTO   M_LOOP
INC_3 ;0~2
   INCF   D_10HOUR,F
   MOVF   D_10HOUR,W
   SUBLW   .3
   BTFSS   STATUS,ZF
   GOTO   M_LOOP
   CLRF   D_10HOUR
   GOTO   M_LOOP
```

