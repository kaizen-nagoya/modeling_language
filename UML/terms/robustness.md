# Q1 UML関連でロバストネスってなんの役にたつのですか。
A1 by chatGPT

はい。UML関連でいう**ロバストネス（Robustness）**は、かなり実務的には、

> **「要求仕様で書かれたこと」を「実際にソフトウェアとして実現できる構造」へ橋渡しする**

のに役立ちます。

特に小川さんが重視されている**状態遷移・シーケンス・C/C++/組込みソフトウェア**との相性がよい概念です。

### 1. ロバストネス図とは

代表的には、UMLのユースケースを分析するときに、次の3種類の要素を使います。

| 種類 | 意味 | 例 |
|---|---|---|
| **Boundary** | 外部との境界 | 画面、CAN通信、センサI/F |
| **Control** | 処理・制御 | 制御ロジック、ユースケース処理 |
| **Entity** | 情報・対象 | 車両状態、注文、設定値 |

例えば自動車の「ブレーキ要求」を考えると、

```text
[Brake Pedal]
      |
      | Boundary
      v
+----------------+
| BrakeControl   |  Control
+----------------+
      |
      v
[VehicleState]       Entity
      |
      v
[Brake Actuator]     Boundary
```

のように、**「誰が何をするのか」**を整理できます。

---

### 2. 何が嬉しいのか

一番大きいのは、**いきなりクラス設計に飛び込まなくてよい**ことです。

例えば要求仕様に、

> 運転者がブレーキを踏むと、ブレーキ制御を開始し、車両状態に応じて制動力を調整する。

と書かれていたとします。

いきなり、

```text
class BrakeController
class VehicleState
class BrakeActuator
```

と考えると、設計者の思い込みが入りやすい。

ロバストネス分析なら、

```text
運転者
  ↓
Boundary
  ↓
Control
  ↓
Entity
  ↓
Boundary
```

という**責務の流れ**を先に確認できます。

---

### 3. シーケンス図への橋渡しになる

ここが小川さんのUML教育では特に使いやすいところだと思います。

例えば、

```text
      Boundary       Control        Entity
          |             |             |
          | brake()     |             |
          |------------>|             |
          |             | getState()  |
          |             |------------>|
          |             |<------------|
          |             | control()   |
          |             |------------->
```

というロバストネス分析を行った後、

**シーケンス図**

↓

**クラス図**

↓

**C/C++実装**

へ展開できます。

つまり、

```text
要求
 ↓
ユースケース
 ↓
ロバストネス分析
 ↓
シーケンス図
 ↓
クラス設計
 ↓
プログラム
```

という流れを作れます。

---

### 4. 「Control」を入れるのが重要

ロバストネスの面白いところは、**BoundaryとEntityだけではなく、その間にControlを置く**ことです。

例えば、

```text
Sensor ──→ SensorControl ──→ VehicleState
```

とすると、

- Sensor：入力を提供する
- SensorControl：入力を処理する
- VehicleState：状態を保持する

という**責務分担**が見えてきます。

これは組込みでは非常に重要です。

例えば、

```text
CAN受信
   ↓
CAN Driver
   ↓
CAN Message Handler
   ↓
Vehicle State
   ↓
Control Algorithm
   ↓
Actuator
```

という構造にも対応させやすい。

---

### 5. ただし「ロバストネス＝ソフトウェアの堅牢性」ではない

ここは名前が少し紛らわしいです。

UMLでいう **Robustness Analysis** の「Robustness」は、

> **障害に強い・壊れにくいソフトウェア**

という意味の「robustness」とは少し違います。

むしろ、

> **要求と設計の間にある分析モデル**

と理解した方がよいです。

---

### 6. 小川さんの教育なら、かなり使い道があります

例えば20日間のUML＋構造化プログラミング教育なら、

```text
Day 1～        要求・ユースケース
       ↓
               Robustness
       ↓
               Sequence
       ↓
               State Machine
       ↓
               Class
       ↓
               C/Python/Rust
```

という流れにすると、**「なぜクラスが必要なのか」**を学生に説明しやすくなります。

特に、

**「ユースケース → ロバストネス → シーケンス → 状態機械 → クラス → コード」**

という流れは、単にUMLの図を覚えさせるよりも、**要求から実装までの論理的な変換過程**を教える教材として使えます。

そして小川さんが重視されている**タイミング**を考えるなら、ロバストネス図だけで終わらせず、そこから**シーケンス図＋タイミング図＋状態機械図**へ展開するのが特に面白いと思います。

# Q2 PlantUMLでロバストネス図を４つ書いてください。そのうち一つは、論理回路設計の例でお願いします。
A2

もちろんです。PlantUMLでは、ロバストネス図を**Boundary / Control / Entity**の3種類に分けて表現すると分かりやすいです。

以下の4例にしました。

1. ECサイトの注文
2. ATMの現金引き出し
3. 自動車のモータ制御
4. **論理回路設計** — HDL設計を意識した例

特に4番目は、通常の業務システムのロバストネス分析を**論理回路・RTL設計へ応用する**形にしています。

---

## 1. ECサイトの注文

```plantuml
@startuml
title Robustness Diagram - EC Order

left to right direction

actor "Customer" as customer

boundary "Order UI" as ui
control "Order Control" as control
entity "Product" as product
entity "Order" as order
entity "Inventory" as inventory

customer --> ui : place order
ui --> control : order()
control --> product : check product
control --> inventory : check stock
inventory --> control : stock status
control --> order : create order
control --> inventory : decrease stock
control --> ui : order result

@enduml
```
<img width="472" height="258" alt="ec" src="https://github.com/user-attachments/assets/30f6544f-7f59-4b06-b32d-63a987ecaeed" />

ここでは、

```text
Customer
    ↓
Boundary
    ↓
Control
    ↓
Entity
```

という基本構造が見えます。

---

## 2. ATMの現金引き出し

```plantuml
@startuml
title Robustness Diagram - ATM Withdrawal

left to right direction

actor "Customer" as customer

boundary "ATM Screen" as screen
boundary "Card Reader" as reader
control "Withdrawal Control" as control
entity "Bank Account" as account
entity "ATM Cash" as cash

customer --> reader : insert card
reader --> control : card information
control --> screen : request PIN
customer --> screen : enter PIN

screen --> control : PIN
control --> account : authenticate
account --> control : authentication result

control --> screen : request amount
customer --> screen : enter amount
screen --> control : amount

control --> account : check balance
account --> control : balance

control --> cash : dispense
control --> account : debit
control --> screen : transaction complete

@enduml
```
<img width="549" height="161" alt="motor" src="https://github.com/user-attachments/assets/13214b95-819c-4348-9570-8a17e6e1fd5b" />
<img width="554" height="256" alt="atm" src="https://github.com/user-attachments/assets/ee4cd3e0-e718-45e6-9e67-162f519ab394" />

この例では、**BoundaryとEntityの間にControlを置くことで、処理責務が明確になる**のがポイントです。

---

# 3. 自動車のモータ制御

これは組込み系なので、ロバストネス分析との相性がかなり良い例です。

```plantuml
@startuml
title Robustness Diagram - Motor Control

left to right direction

actor "Driver" as driver

boundary "Accelerator Pedal" as pedal
boundary "Motor Sensor" as sensor
boundary "Inverter" as inverter

control "Motor Control" as control
control "Torque Control" as torque

entity "Vehicle State" as vehicle
entity "Motor State" as motor

driver --> pedal : accelerator input

pedal --> control : accelerator position
sensor --> control : motor speed / current

control --> vehicle : read vehicle state
control --> motor : read motor state

control --> torque : calculate torque
torque --> inverter : PWM command

inverter --> motor : drive motor
motor --> sensor : speed / current

@enduml
```

<img width="549" height="161" alt="motor" src="https://github.com/user-attachments/assets/f2d366c7-9d23-43b2-ae62-dfaf222a892f" />


この場合、

| 種類 | 要素 |
|---|---|
| Boundary | アクセル、モータセンサ、インバータ |
| Control | Motor Control、Torque Control |
| Entity | Vehicle State、Motor State |

となります。

ここから、

**Robustness → Sequence → State Machine → Timing → C/C++**

という流れに発展させることができます。

---

# 4. 論理回路設計

ここが少し変わった応用例です。

例えば、

> 2つの入力 `A`, `B` があり、両方が1のとき出力 `Y` を1にする。

つまりAND回路です。

```text
Y = A AND B
```

これをロバストネス的に表現すると、

```plantuml
@startuml
title Robustness Diagram - Logic Circuit Design

left to right direction

boundary "Input A" as A
boundary "Input B" as B

control "AND Logic" as and

entity "Signal State" as state

boundary "Output Y" as Y

A --> state : input A
B --> state : input B

state --> and : A, B

and --> Y : Y = A AND B

@enduml
```

<img width="434" height="183" alt="logic" src="https://github.com/user-attachments/assets/b96a70b5-5c58-4dd5-9adb-873f1d05d636" />


ただし、論理回路設計では、もう少しRTL設計に近づけることもできます。

例えば、

```text
入力
 ↓
組合せ論理
 ↓
レジスタ
 ↓
出力
```

という回路なら、

```plantuml
@startuml
title Robustness Diagram - RTL Logic Design

left to right direction

boundary "Input A" as A
boundary "Input B" as B
boundary "Clock" as CLK

control "Combinational Logic" as comb
control "Sequential Logic" as seq

entity "Register State" as reg

boundary "Output Y" as Y

A --> comb
B --> comb

comb --> seq : next_state

CLK --> seq : clock

seq --> reg : update
reg --> seq : current_state

seq --> Y : output

@enduml
```

となります。

---

## ただし、論理回路では少し注意が必要です

ここは重要です。

**UMLのロバストネス図を、そのまま論理回路設計図として使うわけではありません。**

むしろ、

```text
要求
 ↓
Use Case
 ↓
Robustness
 ↓
状態・データ・信号の整理
 ↓
State Machine
 ↓
Timing Diagram
 ↓
RTL
 ↓
Verilog / VHDL
```

という**上流設計の整理手段**として使うのが面白いと思います。

特に小川さんが以前検討されていた

> **状態遷移図からVerilogを生成する**

という方向なら、ロバストネス図をその前段に置いて、

```text
Boundary
   ↓
Control
   ↓
Entity
   ↓
State Machine
   ↓
Timing
   ↓
Verilog
```

# Q3 九州大学と協力して、論理回路設計でロバストネス図を書くという演習をしたことがあります。最近の題材だと何がよいでしょうか。４つ例を示してください。
A3

はい。九州大学との演習で「論理回路設計にロバストネス図を使う」という実績があるなら、最近の題材では、**単純なAND/OR回路より「複数の状態・入力・時間制約があるもの」**を選ぶと、ロバストネス図の意味が出やすいです。

特に、現在の半導体・自動車・AI・エッジコンピューティングを考えると、次の4題材が面白いと思います。

| 題材 | Boundary | Control | Entity | 発展先 |
|---|---|---|---|---|
| ① AIカメラ画像処理アクセラレータ | Camera / Host | Preprocess / MAC / Postprocess | Feature Map / Weight | RTL・NPU |
| ② 車載ゾーンECU | CAN/Ethernet / Sensor | Gateway / Diagnostic | Vehicle State | AUTOSAR・RTL |
| ③ RISC-V AIエッジSoC | CPU / Sensor / Memory | Scheduler / Accelerator Control | Register / Buffer | SoC・FPGA |
| ④ バッテリ管理回路 | Cell / Current Sensor / Charger | SOC/SOH / Protection | Battery State | BMS・安全設計 |

以下、PlantUMLで具体化します。

---

# ① AIカメラ画像処理アクセラレータ

最近の題材としてかなり分かりやすいです。

カメラから画像を受け取り、

```text
Camera
 ↓
Preprocess
 ↓
CNN / MAC
 ↓
Postprocess
 ↓
Detection Result
```

という処理を論理回路化します。

```plantuml
@startuml
title Robustness Diagram - AI Vision Accelerator

left to right direction

boundary "Camera\nPixel Input" as camera
boundary "Host CPU" as cpu
boundary "Result Output" as output

control "Image Preprocessor" as pre
control "MAC Accelerator" as mac
control "Post Processor" as post

entity "Image Buffer" as image
entity "Weight Memory" as weight
entity "Feature Map" as feature
entity "Detection Result" as result

camera --> pre : pixel stream

pre --> image : normalized image
image --> mac : input feature

weight --> mac : weights
mac --> feature : feature map

feature --> post : feature map
post --> result : detection

result --> output
cpu --> mac : start/configuration

@enduml
```

### 演習のポイント

ここから、

- 画像入力のタイミング
- MAC演算の並列度
- SRAMアクセス
- DMA
- Pipeline
- Clock
- Buffer Full/Empty

などを考えられます。

したがって、

**Robustness Diagram → State Machine → Timing Diagram → RTL**

という流れが非常に作りやすい題材です。

---

# ② 車載ゾーンECU

自動車なら、最近は**ゾーンアーキテクチャ**を題材にすると面白いです。

例えば左前方のゾーンECUが、

- センサ
- CAN/CAN FD
- Automotive Ethernet
- アクチュエータ

をまとめる構成です。

```plantuml
@startuml
title Robustness Diagram - Automotive Zone ECU

left to right direction

boundary "Sensors" as sensor
boundary "CAN / CAN FD" as can
boundary "Automotive Ethernet" as eth
boundary "Actuator" as actuator

control "Zone Gateway" as gateway
control "Signal Processor" as processor
control "Diagnostic Control" as diag

entity "Vehicle State" as vehicle
entity "Diagnostic State" as diagnostic
entity "Signal Buffer" as buffer

sensor --> processor : sensor data

processor --> buffer : processed signal
buffer --> gateway : signal

can --> gateway : CAN message
eth --> gateway : Ethernet message

gateway --> vehicle : update state

vehicle --> processor : vehicle state

gateway --> diag : diagnostic request
diag --> diagnostic : update

gateway --> actuator : control command

@enduml
```

これは小川さんの**AUTOSAR、CAN、Ethernet、ECU**の教材にもつなげやすいです。

さらに、

```text
CAN
 ↓
Gateway
 ↓
Signal
 ↓
State
 ↓
Control
 ↓
Actuator
```

という流れを、**UMLからAUTOSAR Software Componentへ対応付ける**演習にもできます。

---

# ③ RISC-V＋AIアクセラレータSoC

これは半導体設計そのものを題材にできます。

最近なら、

> **RISC-V CPU + AI Accelerator + DMA + SRAM**

という構成を一つの演習課題にするのが面白いです。

```plantuml
@startuml
title Robustness Diagram - RISC-V AI SoC

left to right direction

boundary "Sensor" as sensor
boundary "RISC-V CPU" as cpu
boundary "External Memory" as ext
boundary "Output" as output

control "DMA Controller" as dma
control "AI Accelerator" as ai
control "Interrupt Controller" as irq

entity "SRAM Buffer" as sram
entity "Control Registers" as reg
entity "AI Model" as model
entity "Processing State" as state

sensor --> dma : sensor data
dma --> sram : transfer

cpu --> reg : configuration
reg --> ai : accelerator parameters

sram --> ai : input data
model --> ai : weights

ai --> sram : result
ai --> irq : completion

irq --> cpu : interrupt

sram --> dma : output data
dma --> ext : store result

ext --> output

@enduml
```

この題材の良いところは、**論理回路だけでは終わらない**ことです。

学生に、

> 「CPUで全部計算するのと、専用回路にするのでは何が違う？」

を考えさせられます。

そして、

- 面積
- 消費電力
- レイテンシ
- スループット
- メモリ帯域
- 並列度

という、現在の半導体設計の重要なトレードオフに進めます。

---

# ④ BMS（Battery Management System）

4つ目は、**安全工学まで含められる**題材です。

例えば、

```text
Battery Cell
 ↓
Voltage/Current Sensor
 ↓
Protection Logic
 ↓
MOSFET
```

という回路です。

```plantuml
@startuml
title Robustness Diagram - Battery Management System

left to right direction

boundary "Battery Cells" as cell
boundary "Voltage Sensor" as vs
boundary "Current Sensor" as cs
boundary "Temperature Sensor" as ts

control "Measurement Control" as measure
control "SOC/SOH Estimator" as estimator
control "Protection Logic" as protection
control "Charge/Discharge Control" as charge

entity "Battery State" as battery
entity "Threshold Parameters" as threshold
entity "Fault State" as fault

boundary "MOSFET / Contactor" as switch
boundary "Charger / Inverter" as power

cell --> vs : cell voltage
cell --> cs : current
cell --> ts : temperature

vs --> measure
cs --> measure
ts --> measure

measure --> battery : measurement data

battery --> estimator
estimator --> battery : SOC / SOH

battery --> protection
threshold --> protection

protection --> fault : fault detection
fault --> protection : protection state

protection --> switch : ON/OFF

charge --> switch
charge --> power

battery --> charge : battery state

@enduml
```

これは特に、

**正常状態 → 過充電 → 過放電 → 過電流 → 過温度 → Fault**

という状態機械へ展開できます。

さらにタイミング図を使って、

```text
Voltage
Temperature
Current
       ↓
Detection
       ↓
Protection
       ↓
MOSFET OFF
```

の**安全機能の応答時間**を検討できます。

---

# 4題材を並べると

私なら九州大学との演習を現在版に更新するなら、次の4段階にします。

```text
① AI Accelerator
       ↓
② Automotive Zone ECU
       ↓
③ RISC-V + AI SoC
       ↓
④ BMS Safety Logic
```

それぞれ違う能力を要求できます。

| 題材 | 主な学習テーマ |
|---|---|
| AI Accelerator | データフロー・並列処理 |
| Zone ECU | 通信・分散システム |
| RISC-V SoC | CPUと専用HWの協調 |
| BMS | 状態・故障・安全 |

そして4題材に共通して、

**Boundary → Control → Entity**

を最初に描かせる。

その後、

**Robustness Diagram → Sequence Diagram → State Machine Diagram → Timing Diagram → RTL**

まで進めると、単なる「UMLのお絵描き」ではなく、**要求分析から論理回路設計までを一貫して扱う演習**になります。

特に小川さんの場合、以前の「状態遷移図からVerilogを生成する」というテーマと組み合わせると、**Robustness DiagramをRTL生成の上流モデルにする**という研究テーマまで自然につながります。

という変換パイプラインを作ることができます。

この考え方だと、**「論理回路の機能要求 → UML → RTL」**という、かなり面白い研究テーマにもできます。
