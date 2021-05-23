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

<img width="400" alt="RasberryPiImagerIcon" src="/Users/suhyouri/Documents/collaboration/PIC16-L-F183XX/img/2-1.png">

최소연결해야하는 핀들 회로도

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

