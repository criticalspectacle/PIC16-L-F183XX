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

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/122e9fa0-6715-4261-b296-60602a85db07/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/122e9fa0-6715-4261-b296-60602a85db07/Untitled.png)

In-line packages는 oscillator pins을 완전히 포함하는 단일방면 레이아웃으로 처리 될 수 있다. fine-pitch packages가 항상 핀들과 부품들을 완전히 둘러싸는 것이 가능한 것은 아니다.

application의 routing과 I/O할당을 설계할 때, 인접한 포트 핀과 oscillator 가까이에 있는다른 신호들이 benign(양성?)인지 확인해라(고주파가 없는지?, 짧은 rise and fall 시간, 그외 유사한 노이즈)

## 2.6 Unused I/Os

사용되지 않는 입출력 핀들은 output으로 설정되어야하고 logic low 상태로 구동되어야 한다. 대안으로, 사용하지 않는 핀들에 1 kΩ ~10kΩ 저항을 Vss에 연결해서 출력 로직을 low로 만들어라


