# WEEK 2
- [x]  드림핵
  - 기드라 과정: 코드 패치~
  - 리버싱 과정: 어셈 파트 복습, Exercise 풀이
- [x]  도서 ‘리버싱 핵심원리’
  - 1부 6장~12장: 개인적으로 진행
  - 2부: 스터디 범위
- [x]  문제풀이 과제
  - reversing\.kr - Easy ELF, Easy keygen
  - dreamhack- basic-rev2
  - 나머지는 개인적으로 진행

<br>

## 리버싱 핵심원리
### abex' crackme \#1

파일 실행 시 "Make me think your HD is a CD-Rom" 출력.  
HD가 CD-ROM으로 인식되도록 해야함.

![WEEK-2-cackme1](./img/WEEK2/WEEK-2-crackme1-1.png)

GetDriveType() 함수가 무엇인지 보자.
```
UINT GetDriveTypeA(
  [in, optional] LPCSTR lpRootPathName
);
```
- 드라이브의 루트디렉터리를 전달받은 뒤, 반환 값에 따라 드라이브 유형을 지정함.  
[관련 링크](https://learn.microsoft.com/ko-kr/windows/win32/api/fileapi/nf-fileapi-getdrivetypea)

|반환값|의미|
|-----|-----|
|0|드라이브 유형 확인 불가능|
|1|루트 경로가 잘못됨|
|2|이동식 미디어|
|3|고정 미디어(Hd, 플래시 등)|
|4|네트워크 드라이브|
|5|CD-ROM|
|6|RAM 디스크|

이 문제를 풀기 위해서는 함수 반환 값을 5로 패치해버리면 될 것 같다.   
- `CALL <JMP.&KERNEL32.GetDriveTypeA>` 부터 F8을 눌러가며 확인

![WEEK-2-crackme1](./img/WEEK2/WEEK-2-crackme1-2.png)
- `CALL GetDriveTypeW` 명령어 실행 후 EAX 값이 3으로 변경됨을 확인
- 계속 진행

![WEEK-2-crackme1](./img/WEEK2/WEEK-2-crackme1-3.png)
- EAX 값과 ESI를 비교해서 Equal이면 점프.

![WEEK-2-crackme1](./img/WEEK2/WEEK-2-crackme1-4.png)
- EAX 값 변경

![WEEK-2-crackme1](./img/WEEK2/WEEK-2-crackme1-5.png)
- "your HD is a CO-ROM!" 출력

<br>

### 스택 프레임

스택프레임 어셈블리 구조 
```
PUSH EBP      // 함수의 시작, EBP를 스택에 PUSH (내가 돌아갈곳의 주소가 저장되어있음)
MOV EBP, ESP  //현재 스택포인터 EBP에 저장
...

MOV ESP, EBP  // EBP 값을 ESP에 (=함수 호출 이전의 스택프레임으로 돌아오기 위함)
POP EBP       // 함수 마지막, 스택에서 EBP 값 꺼내서 EBP를 복구함(=함수 호출 이전의 스택 베이스 주소를 갖기위함)
RETN
```

**※ 스택에 복귀주소가 저장될 때, 취약점으로 작용가능함. BUF 통해 복귀주소가 저장된 스택메모리를 다른 값으로 변경 가능하기 때문.**

> `XOR EAX,EAX` : main() 함수의 리턴 값(0) 세팅을 위한 명령어. 같은 값끼리 XOR하면 0이 되는 특징으로, 실행속도가 빨라서 레지스터 초기화 시 많이 사용됨

<br>

### abex' crackme2

- TEST 명령어: 두 오퍼랜드 중 하나가 0이면 ZF = 1로 셋팅  
- JE 명령어: Jump if equal. ZF = 1이면 점프

이거 풀이는 낼 출근하고 적기