PIC16-L-F183XX Summary
===

2.0 PIC16(L)F183XX 마이크로 컨트롤러 시작하기 가이드 
---

2.1 기본 연결 요구사항
---

8비트 PIC-16(L)F183XX 마이크로 컨트롤러는 개발 진행 전에 최소한의 필수적인 핀이 연결되었는지 확인해야 한다.

- 꼭 연결되어야 할 핀 목록
    - 모든 VDD(전압V+)와 VSS(전압V-) 핀들 (**2.2** 전원공급핀들PowerSupplyPins 참고)
    - MCLR(**2.3** Master Clear핀 참고)핀 (외부 작동 설정 시)
- end application에서 사용 시, 연결되어야 할 핀 목록
    - ICSPCLK/ICSPDAT (**2.4** ICSP 참고와 디버깅 목적으로 사용)
        - ICSP(In Circuit Serial Programming)
        - CLK(Clock line of the serial data interface. This line swings from GND to Vdd and is always driven by the programmer. Data is transferred on the falling edge.)
        - DAT(Serial data line. The serial interface is bi-directional, so this line can be driven by either the programmer or the PIC depending on the current operation. In either case this line swings from GND to Vdd. A bit is transferred on the falling edge of PGC.)
        - 출처: [https://en.wikipedia.org/wiki/In-system_programming](https://en.wikipedia.org/wiki/In-system_programming)
    - OSC1와 OSC2핀(**2.5** External Oscillator 핀들 참고)
- 추가적으로 필요할 수도 있는 핀목록
    - 아날로그 모듈에 외부전압이 사용될 때, VREF+/VREF- 핀 연결 필요

* 최소연결해야하는 핀들 회로도
  
  <img width="300" alt="RasberryPiImagerIcon" src="https://github.com/criticalspectacle/PIC16-L-F183XX/blob/main/img/2-1.png?raw=true">

2.2 전원 공급 핀 Power Supply Pins 
---
2.2.1 디커플링 캐패시터 Decoupling Capacitors

VDD와 VSS같은 파워공급핀들의 모든 짝은 디커플링 캐패시터를 사용해야한다. (*decoupling capacitors은 전원과  GND사이에 붙이는 capacitor로서 전류가 크게(발진, 노이즈)흐르는 것을 막아주는 역할을 하며, coupling되는 걸 막아준다). 모든 VDD와 VSS 핀들은 연결되어 있어야 하고, 플로팅으로 남겨두면 안된다.

- 디 커플링 캐패시터 사용 규칙
    - **캐패시터 종류과 값**: 0.1 마이크로 패럿(100 나노패럿), 10-20V 캐패시터 권장. 캐패시터는 low-ESR(Equivalent Series Resistance) device를 사용하고, 공진주파수(resonance frequency)는 200MHz 보다 높아야 한다. Ceramic capacitor가 권장된다.
    - **프린트 된 회로기판Circuit board에 위치**: 디커플링 캐패시터는 핀에 가능한 가깝게 위치해야 한다. 기기의 같은 면의 보드에 위치하는 것을 추천, 간격이 좁으면, 캐패시터는 via를 사용하는 PCB의 다른 레이어에 위치해도 된다. 근데 어쨌든 핀이랑 캐패시터 사이의 trace length의 길이는 0.25inch(6mm)보다 길면 안된다.
    - **높은 주파수 노이즈 다루기**: 수십 MHz보다 높은 노이즈가 보드에서 발생한다면, 두번째 ceramic 타입의 캐패시터를 디커플링 캐패시터 위에 병렬적으로 추가하자. 두번 째 캐패시터의 값은 0.01 ~ 0.001 마이크로 패럿 사이가 되어야 한다. 요지는 각각 주요한 디커플링 캐패시터 옆에 두번째 캐패시터를 위치하라는 것이다. 높은 속도의 회로 디자인에서, 10년치 용량을 가능한 파워와 그라운드핀에 가깝게 위치하는 걸 고려하자.(예를 들면, 0.1 마이크로 패럿에 병렬적으로 0.001 마이크로 패럿위치하기)
    - **성능 극대화**: 전원공급회로의 보드레이아웃에서, 전원을 작동하고, 먼저 디 커플링 캐패시터로, 그 다음에 디바이스 핀으로 trace가 반환된다. 디 커플링 캐패시터가 먼저 power chain에 있는것을 확인해야한다. 캐패시터랑 파워핀 사이에 trace length를 유지하면서 PCB trace inductance를 감소시키는것이 중요하다.

2.2.2 TANK CAPACITORS

6inch보다 긴 power traces(in PCB)가 있는 보드에서는 마이크로컨트롤러를 같은 집적회로가 local power source를 공급하기 위해 탱크 캐패시터를 사용하는 것이 좋다. 탱크 캐패시터의 값은 기기에 전원공급 소스에 연결되어 있는 trace resistance와 응용 프로그램에서 장치가 끌어들이는 최대 전류를 기반으로 결정된다. 즉, 기기에 허용가능한 전압 하락스펙을 충족하는 탱크 캐패시터를 선택해야한다. 보통 값은 4.7~ 47 마이크로 패럿이 된다. 

2.3 Master Clear(MCLR) Pin
---

MCLR핀은 세 특정한 기기 기능을 제공한다. 

- Device Reset (MCLR이 1일때)
- Device Input Pin(MCLR이 0일때)
- Device Programming and Debugging

만약 프로그램과 디버깅이 요구되지 않는다면, Application은 MCLRE configuration bit를 1에 설정하고, 그 핀을 digital input으로 사용하거나 MCLRE Configiretation bit를 정리한다. 그리고 핀이 internal weak pull-up에 사용되도록 열어둔다. 다른 컴포넌트들에 더해서, 가짜 전압 하락으로 리셋에 대한 응용 프로그램의 저항을 증가시키는것이 도움이 될 수 있습니다. 일반적인 configuration은 2-1에 나와있습니다. 어플리케이션 요구에 따라 다른 회로 디자인이 진행될 수 있다. 

프로그래밍하고 디버깅 하는 동안에, 핀에 추가된 저항과 정전용량capacitance은 반드시 고려되어야 한다. 기기 프로그래머와 디버거는 MCLR 핀을 작동시킨다. 결과적으로 특정한 전압레벨(VIH &VIL)과 빠른 신호 변화는 역으로 영향받아서는 안된다. 그러므로 프로그래밍하고 디버깅작동중에는 프로그래머 MCLR/Vpp 아웃풋은 R1이 MCLR핀으로부터 capacitor(C1)과 분리되기 위해 핀에 직접적으로 연결되어야한다. 

MCLR핀과 관련있는 어떤 컴포넌트라도 핀에 0.25inch(6mm)안에 위치되어야 한다.


## 2.4 ICSP™ Pins

[ICSP?](http://ww1.microchip.com/downloads/en/devicedoc/30277d.pdf) 

ISP(In-System Programming)? 장치가 회로기판에 배치된 이후에 프로그래밍이 가능한 기술 

ICSP?(In-Circuit Serial Programming)? 향상된 ISP기술 

ICSPCLK, ICSPDAT 핀들의 목적:  In-Circuit Serial Programming(ICSP), debugging

ICSP connector와 ICSP pins의 길이는 가능한 짧은 것이 좋다.

ICSP connector가 ESD event를 겪는다면, 100Ω을 넘지않은 몇 십 짜리 옴의 저항값을 가진 직렬저항을 권장한다.

ICSPCLK, ICSPDAT핀의 Pull-up저항, 직령 다이오드, 캐패시터는 programmer/debugger와 장치간의 통신을 방해하지 않는것이 좋다.

만약 개별 부품이 

If such discrete components are an application requirement, they should be isolated from the programmer by resistors between the application and the device pins or removed from the circuit during programming.

또는 개별 장치 Flash 프로그래밍 스펙(capacitive loading 제한, VIH, VIL조건에 대한 정보가 있는 )의  AC/DC 특성과 timing 요구사항 정보를 참조해라

장치 emulation을 위해서는, 장치에 프로그램된 "Communication Channel Select"를 확인해하고, ICSP에서 Microchip으로의 물리적인 연결을 매치해라 

## 2.5 External Oscillator Pins

capacitor의 역할은 전기를 모아서 "충전과 방전"을 반복하는 것

The PIC16(L)F183XX 계열은 2개의 external crystal oscillator 중 하나를 선택적으로 지원한다. : a high-frequency primary oscillator, a low-frequency secondary oscillator. 

둘 중 하나를 EXTOSC = EC(L/M/H) 또는 SOSCBE로 설정해 external digital clock signal을 우회할 수 있다.

oscillator회로는 장치와 같은 면의 보드에 배치되어야 한다. oscillator회로를 회로부품과 핀들사이의 간격이 0.5inch(12mm)이하로 각 oscillator핀들에 가깝게 배치해라.

부하 캐패시터(load capacitor)는 보드의 같은 측면의 오실레이터 옆에 놓여야한다.

주변 회로들로 부터 분리하기 위해서는 oscillator(발진기)회로 주위로 접지된 구리pour?를 사용해라. 접지된 구리pour는 MCU ground에 직접적으로 연결되어야 한다. 어떠한 signal traces나 power traces도 ground pour 안에서 실행하지 마라. 만약 양면 보드를 사용한다면,  crystal 이 위치한 반대쪽 면의 보드에대한 어떠한 traces도 피해라 . 

Layout 제안은 그림과 같다.

<img width="300" alt="Single and Dual-Sided Layouts" src="https://github.com/criticalspectacle/PIC16-L-F183XX/blob/main/img/layout.png?raw=true">

In-line packages는 oscillator pins을 완전히 포함하는 단일방면 레이아웃으로 처리 될 수 있다. fine-pitch packages가 항상 핀들과 부품들을 완전히 둘러싸는 것이 가능한 것은 아니다.

application의 routing과 I/O할당을 설계할 때, 인접한 포트 핀과 oscillator 가까이에 있는다른 신호들이 benign(양성?)인지 확인해라(고주파가 없는지?, 짧은 rise and fall 시간, 그외 유사한 노이즈)

## 2.6 Unused I/Os

사용되지 않는 입출력 핀들은 output으로 설정되어야하고 logic low 상태로 구동되어야 한다. 대안으로, 사용하지 않는 핀들에 1 kΩ ~10kΩ 저항을 Vss에 연결해서 출력 로직을 low로 만들어라


**3.0 향상된 미드레인지 CPU ( 밴치마크 점수가 중간인 cpu)**

이 디바이스 제품군에는 향상된 미드 레인지 8bit CPU 코어가 포함되어 있다.

CPU 에는 48개의 명령이 있다. 인터럽트 기능은 컨텍스트 자동저장기능이 포함되어 있다. 하드웨어 스택의 깊이는 16단계이며 오버플로 및 언더플로 재설정 기능이 있다. 직접, 간접, 상대 주소 지정모드를 사용할 수 있다.

[https://www.geeksforgeeks.org/difference-between-relative-addressing-mode-and-direct-addressing-mode/](https://www.geeksforgeeks.org/difference-between-relative-addressing-mode-and-direct-addressing-mode/)

두개의 FSR은 프로그램과 데이터 메모리를 읽는 기능을 제공한다.

- FSR = 데이터 메모리의 간접주소 지정을 위한 메모리 포인터로 사용되는 16비트 레지스터이다. 간접 주소 지정을 통해 사용자는 지침에 고정된 주소를 지정하지 않고도 데이터 메모리의 위치에 액세스할 수 있습니다.FSR은 RAM에 위치하며 프로그램 제어 하에 조작할 수 있다.

**3.1 자동 인터럽트 컨텍스트 저장**

인터럽트 중에는 특정 레지스터는 자동으로 세도우 레지스터에 저장되며 인터럽트로부터 반환될 때 복원된다.

이것은 스택 공간과 사용자 코드를 절약한다.

=> 8.5 **Automatic Context Saving**

인터럽트를 입력하면 반환 pc 주소가 스택에 저장된다. 또한 다음의 레지스터는 세도우 레지스터에 자동저장된다.

• Wregister	

• STATUSregister(exceptforTOandPD)

• BSRregister	

• FSRregisters	• PCLATHregister

인터럽트 서비스 루틴ISR 을 종료하면 이런 레지스터가 자동으로 복원된다.

ISR 중 이 레지스터에 대한 어떠한 수정사항도 손실된다.

만약 이 레지스터에 대한 어떠한 수정을 원한다면,

해당하는 섀도 레지스터를 수정해야 하며 ISR을 종료할 때 값이 복원됩니다.

섀도 레지스터는 뱅크 31에서 사용할 수 있으며 읽기 및 쓰기 가능합니다.

사용자의 응용 프로그램에 따라 다른 레지스터도 저장해야 할 수 있습니다.

**3.2 오버플로 및 언더플로가 포함된 16 레벨 스택**

이 장치들은 15비트 폭과 16단어 깊이의 하드웨어 스택 메모리를 가지고 있다.

스택 오버플로 또는 언더플로우는 PCON0 레지스터에서 적절한 비트(STKOVF 또는 STKUNF)를 설정하고, 이것을 선택하면 소프트웨어 재설정의 원인이 될 것이다.

=> **4.4 Stack**

모든 디바이스는 16레벨 에 15비트 폭의  하드웨어 스택을 가진다. 스택 공간은 프로그램 또는 데이터 공간의 일부가 아니다. 브랜치에 의한 인터럽트나 CALL 또는 CALLLW 명령이 실행될때 pc는 스택에 푸시된다. 스택은 RETURN, RETLW 또는 RETFIE 명령이 실행될 때 POP됩니다. PCLATH는 PUSH 또는 POP 작업의 영향을 받지 않습니다.

스택은 순환버퍼로 작동하며 STVREN 비트가 ‘0’으로 프로그래밍 된 경우 스택 오버 플로 또는 언더 플로가 발생할때 재설정하지 않는다. 즉, 스택을 16회 푸시한 후 17번째 푸시는 저장된 값을 덮어쓴다. 18번째 푸시는 두번째 푸시를 덮어쓴다.

리셋이 활성화 되었는지에 상관없이 STKOVF 및 STKUNF 플래그 비트는 Overflow/Underflow(오버플로/언더플로)에 설정된다.

컨피그레이션 단어의 STVREN 비트가 ‘1’로 프로그래밍 된 경우, 스택이 16번째 레벨을 초과하거나 첫 번째 레벨을 초과하여 POP되면 장치가 재설정되어 PCON 레지스터에 적절한 비트(STCOFF 또는 STKUNF)를 설정합니다.

Note 1 : PUSH 또는 POP라는 명령 / 니모닉(연상기호)은 없습니다. 이것은 CALL, CALLW, RETURN, RETLW 및 RETFIE 명령의 실행이나 인터럽트 주소로의 벡터링에서 발생하는 동작들이다.

⇒ 4.4.1 ACCESSING THE STACK

스택은 TOSH, TOSL 및 STKPTR 레지스터를 통해 액세스할 수 있습니다. STKPTR은 스택 포인터의 현재 값입니다. TOSH:TOSL 레지스터 페어는 스택의 TOP를 가리킨다. 두 레지스터 모두 읽고 쓰는 것이 가능하다. TOS는 PC의 15비트 크기를 위해 TOSH와 TOSL로 분할됩니다. 스택에 액세스하려면 TOSH:TOSL를 배치하는 STKPTR 값을 조정하고 TOSH:TOSL 읽고 씁니다. STKPTR은 오버플로우 및 언더플로우를 탐지할 수 있는 5비트입니다.

Note : 인터럽트가 활성화된 동안 STKPTR을 수정할 때 주의해야 합니다.

정상적인 프로그램 작동중에, CALL, CALLW 및 Interrupts는 STKPTR을 증가시키고, RETLW, RETFIE는 STKPTR을 감소시킨다. 언제든지 STKPTR는 스택에서 사용가능한 레벨이 얼마나 남았는지 확인할 수 있다.

STKPTR은 항상 스택에서 현재 사용되는 위치를 가리킵니다. 따라서 CALL 또는 CALLW는 STKPTR을 증가시킨 다음 PC를 쓰고 반환하면 PC를 쓴 다음 STKPTR을 감소시킵니다.

<img width="300" alt="Accessing the Stack EX01" src="https://github.com/criticalspectacle/PIC16-L-F183XX/blob/main/img/stackEx01.png?raw=true">

<img width="300" alt="Accessing the Stack EX04" src="https://github.com/criticalspectacle/PIC16-L-F183XX/blob/main/img/stackEx02.png?raw=true">

**3.3 파일 선택 레지스터**

여기에는 두개의 16비트 FSR이 있다. FSR은 모든 파일 레지스터 프로그램 메모리, 그리고 모든 메모리를 위한 하나의 더이터 포인터를 허락하는 데이터 EEPROM을 엑세스 할 수 있다. FRS가 프로그램 메모리를 가리키면, 데이터를 패치하는 INDF 명령에는 하나의 추가적인 인스트럭션 사이클이 있다. 범용 메모리는 이제 80바이트 이상의 큰 연속데이터에 엑세스 하는 능력을 제공하면서 선형적으로 처리될 수 있다.

⇒ 4.5 Indirect Addressing

INDFN 레지스터는 물리적 레지스터가 아닙니다. NDFN 레지스터에 액세스하는 모든 명령은 실제로 파일 선택 레지스터(FSR)에서 지정한 주소로 레지스터에 액세스합니다. 만약 FSRn 주소가 두개의 INDFN 레지스터 중 하나를 지정하면, 읽은 값은 '0'이 되고 쓰기는 발생하지 않는다.(Status 비트는 영향을 받을 수도 있지만)

FSRn 레지스터 값은 FSRnH 및 FSRnL 쌍에 의해 생성됩니다.

FSR 레지스터는 65536 위치의 주소 공간을 허용하는 16비트 주소를 형성한다.

이 위치는 네 개의 메모리 영역으로 나뉩니다.

- Traditional/BankedDataMemory
• LinearDataMemory
• ProgramFlashMemory
• EEPROM

*아래 각각의 설명을 추가해야함

**3.4 명령 집합**

여기에는 향상된 미드레인지 CPU가 CPU의 기능을 지원하기 위한 48가지 명령이 있습니다.

⇒ 34.0 INSTRUCTION SET SUMMARY

각 명령어는 연산 코드(opcode)와 필요한 모든 피연산자를 포함하는 14비트 워드이다. opcode는 세 가지 넓은 범주로 나뉜다.

- ByteOriented
- BitOriented
- LiteralandControl

리터럴 및 컨트롤 범주는 가장 다양한 명령어 형식을 포함한다. (표 34-3 참고)
