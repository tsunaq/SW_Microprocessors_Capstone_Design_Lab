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
