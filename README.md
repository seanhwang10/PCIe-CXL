# Table of Contents

- [Table of Contents](#table-of-contents)
- [PCI Express](#pci-express)
  * [PCIe Basics](#pcie-basics)
  * [Structures - PCIe Layer](#structures---pcie-layer)
    + [Software Layer](#software-layer)
    + [Transaction Layer](#transaction-layer)
    + [Data Link Layer](#data-link-layer)
    + [Physical Layer](#physical-layer)
  * [PCIe data transaction protocol](#pcie-data-transaction-protocol)
    + [Properties of TLP (Transaction Layer Packet)](#properties-of-tlp--transaction-layer-packet-)
  * [Link Training and Status State Machine (LTSSM)](#link-training-and-status-state-machine--ltssm-)
    + [States](#states)
      - [Detect](#detect)
      - [Polling](#polling)
      - [Configuration](#configuration)
      - [Recovery](#recovery)
      - [L0](#l0)
      - [L0s](#l0s)
      - [L1](#l1)
      - [L2](#l2)
      - [Disable](#disable)
      - [Loopback](#loopback)
      - [Hot Reset](#hot-reset)
  * [TLP Routing Mechanism](#tlp-routing-mechanism)
    + [Address Spaces](#address-spaces)
    + [Split Transaction Protocol](#split-transaction-protocol)
    + [Three methods of TLP routing](#three-methods-of-tlp-routing)
  * [Root Complex](#root-complex)
    + [PCIe Root Port](#pcie-root-port)
    + [Switch & Bridge](#switch---bridge)
    + [PCIe Endpoint Device 5](#pcie-endpoint-device-5)
    + [Enumeration](#enumeration)
  * [PCI Configuration Space](#pci-configuration-space)
    + [BDF (Bus, Device, Function)](#bdf--bus--device--function-)
    + [Extended configuration space for PCIe](#extended-configuration-space-for-pcie)
    + [Standardized Registers](#standardized-registers)
  * [PCIe Ordered Set](#pcie-ordered-set)



# PCI Express

Since CXL depends on the physical interface of PCIe, it is necessary to review the fundamentals of PCIe. I personally have one summer worth of experience with PCIe during my internship at Samsung Electronics, working with PCIe 5.0 in NAND SSDs. 

## PCIe Basics

- `Bi-directional connection` that allows both transmitting and receiving. 
  
  - Each link can either transmit or receive data. 
    
    - `link` : Connection between two devices.
  
  - Hence, combination of 2 simplex makes it `Dual-Simplex Connection`. 

- In each `link` there are multiple `lanes`. 
  
  - in forms of : x1, x2, x4, x8, x16, x32 

- Uses `Serial Communication` rather than parallel. 
  
  - Parallel comm has issues such as: 
    
    - Parallel bus needs its `flight time` to be lower than a period of `clock`. In high speed communication, this is very difficult. 
      
      - `Flight Time` : Time takes for a signal to reach RX from TX in a bus. 
    
    - `Clock Skew`, as RX side and TX side has different clock arrival. 

- Uses `Differential Signal` and sends D+ and D-
  
  - Uses the difference between D+ and D- to recognize a signal.
  
  - This helps lower the impact of noise on signal integrity,

## Structures - PCIe Layer

Packet breakdown:

- `TLP` (Transaction Layer Packet)
  
  - Header
  
  - Data
  
  - `ECRC` (End to end CRC)
    
    - `CRC`: Cycle Redundancy Check

- `DLLP` (Data Link Layer Packet)
  
  - All of `TLP` 
  
  - `LCRC` (Link CRC)
    
    - Used by nearby receiver checking for errors. 

- Upon passing `Physical Layer`, `start` and `end` is added to `DLLP`. 

### Software Layer

- Start and destination of all `Requests` made in the PCIe. 

- Not listed in the official PCIe specification. 

### Transaction Layer

- Creates `outbound packets` upon request from software layer. 

- `TLP : Transaction layer packets` 
  
  - Non-posted : RX sends `completion packet` upon receiving TLP. 
  
  - Posted : RX does not need to send `completion` upon receiving TLP. 
  
  - Sending completion packets increases accuracy, but creates more burden as time required for waiting for completion. 

| Request Type         | Posted?    | Acronym |
| -------------------- | ---------- | ------- |
| Memory Read Request  | Non-Posted | MRd     |
| Memory Write Request | Posted     | MWr     |
| IO Read Request      | Non-Posted | IORd    |
| IO Write Request     | Non-Posted | IOWr    |
| Config Read Request  | Non-Posted | CfgRd   |
| Config Write Request | Non-Posted | CfgWr   |
| Message Request      | Posted     | Msg     |

[More TLP details](#TLP (Transaction Layer Packet)) 

### Data Link Layer

- Mainly responsible for`TLP error correction`,`Flow control`,`Link Power Management`
  
  - TLP error correction (Ack, Nak Protocol)
    
    - LCRC (Link CRC) & sequence number is added to TLP
    
    - TLP is buffered into Replay Buffer before data is being transmitted. 
    
    - When RX sends TX Ack/Nack `DLLP` as a signal for data receive, next set of data is sent from the buffer. 
  
  - Flow Control 
    
    - TX can only send TLP to RX when RX's buffer is empty. RX uses `DLLP` to report the status of its buffer to TX. 
    
    - When RX processes all its TLP and remove it from the buffer, it sends `Flow Control Update DLLP`. 
  
  - Power Management 
    
    - DLLP can request the status of Link & System Power

### Physical Layer

Physical Layer is divided into two parts: `Logical` and `Electrical` part. 

- Logical Part 
  
  - Digital Logic 
  
  - Start and End is added to DLLP 
  
  - `Byte Striping`: packets get divided in each lanes. 
  
  - Link Training and Initialization 
    
    - For smooth exchange of packets in the two links, the following options are checked: 
      
      - Link Width
      
      - Link Data Rate
      
      - Bit Lock per Lane
      
      - Symbol lock per lane 
      
      - Lane Reversal 
      
      - Polarity inversion 
      
      - Lane to Lane de-skew (when using multi-lane)

- Electrical Part
  
  - Analog interface 
  
  - AC coupling: capacitors that filters DC signal from AC/DC signal. 
  
  - `OS: Ordered Set`
    
    - Only used in the Physical Layer. 
    
    - Link Training, Entry or Exit low power state
    
    - Clock tolerance compensation
      
      - Compensates the difference of internal clock between RX and TX side. 

## PCIe data transaction protocol

PCIe transaction is divided into 4 categories: 

1. Memory 

2. IO

3. Configuration 

4. Message Transaction 

Transactions are divided into `Non-Posted` and `Posted`. 

### Properties of TLP (Transaction Layer Packet)

- TLP access 4 different `Address Spaces`. 

## Link Training and Status State Machine (LTSSM)

- Sets and resets `links` and `ports` to allow packets to be sent. 
  
  - Physical layer controlled, hardware based processor. 

- Diagram of all states that a `link` can have. 
  
  - `LTSSM` has 11 states, including `Detect`, `Polling`. 
  
  - Each state has its substates 

![ltssm.png](/Users/seahwang/Documents/GitHub/Cisco-Internship/figures/ltssm.png)

### States

Related concepts: 

- `Ordered Set`: Packets used for communication between devices in the physical layer. 

- `Training Sequence` : Ordered sets used for link initialization and training. Acronym : TS. 
  
  - TS1, TS2 

- Cold Reset : Power on for system without power.

- Warm Reset : Reset without system losing power.

#### Detect

- State for both ends to detect & identify the link. 

- Initial state from Boot or Reset 

- `Resets` all logics, ports, registers. 

#### Polling

- State to process / establish the following 
  
  - `Bit lock` : Synchronizes the `clock` for both ends of link. 
    
    - we say a `lock` is secured when sync is made. 
  
  - `Symbol lock` : Finding the boundary of continued bits (Gen1, Gen 2)
    
    - 8b/10b (for Gen 1, Gen 2) 
    
    - PCIe 5.0 uses 128b/130b. 
    
    - (Target data size / Actual required size for data send) 
  
  - `Land Polarity` : Checks if lane is `D+` or `D-`
  
  - `Block Assignment` : Founding the boundary of continued bits  using EIEOS (Gen 3)
  
  - `Lane Data Rate` : Speed of data transfer for each lane. 

#### Configuration

- State to process / establish the following 
  
  - `Lane reversal` : check if lane connection is reversed. 
    
    - for example, [0:3] can be connected as [3:0] in the other end. 
  
  - `Link & Lane number` : Data displayed as `PAD` in `TS` (Training Sequence) gets changed to a negotiated number. 
    
    - Negotiation is done to utilize maximum performance of both devices at the end of a link. 
  
  - `Lane to Lane de-skew` : Corrects the `skew` 
    
    - `Skew` : Parallel bit streams from multiple lanes arriving the RX device at different times. 

#### Recovery

- State that recovers changes made. Processes the following 
  
  - Resets `Bit lock`, `Symbol lock`, `Block alignment`, `Lane to lane de-skew`. 
  
  - Change / Reset the link speed 

- State before entering `Loopback`, `Hot reset`, or `Disabled`. 
  
  - Or when state is changed from `L1` to `L0`. 

#### L0

- State of **successful transaction of data.**

#### L0s

- Substate of `L0`. 

- Saves power while returning to `L0` state can be done fast. 

#### L1

- Saves more power than `L0s`, but returning to `L0` state takes longer. 

- `L1` state can be entered by: 
  
  - ASPM (Active State Power Management)
    
    - When device connected via PCIe is not in use, this policy changes `links` to low power state. 
  
  - PMSW (Power Management Software)
    
    - Can change the device to low power state. 
    
    - D1, D2, D3hot, D3cold 

#### L2

- Maximum power saving mode. 
  
  - Turns the device **OFF** to achieve that. 

#### Disable

- Makes the `TX` ***Electrically Idle*** when `RX` is at ***low impedance state.***

- TS1's `TC (Training Control) disable bit` -> 1 
  
  - Sends 16 of `TC disable bit` set to 1

#### Loopback

- State for testing. 

- Enters the state when TS1's `Loopback bit` is set to 1. 

- Receiver `echos` the received packet back. 
  
  - Sends the same packet back to the TX. 

#### Hot Reset

- Link can be reset by setting `Secondary Bus Reset Bit` as 1. 
  
  - Located in `Bridge Control Register` 

- TS1's TC Hot reset bit -> 1 
  
  - Sends a `TC Hot reset bit` set to 1. 

## TLP Routing Mechanism

Transaction Layer Packets gets transmitted from one link to the other while following TLP routing mechanism. There are 4 different `Address Spaces` that TLP access. 

### Address Spaces

- Memory (Read, Write)
  
  - Transfer data to or from a location in the system memory map. 

- IO (Read, Write)
  
  - Transfer data to or from a location in the system IO map. 

- Configuration (Read, Write)
  
  - Transfer data to or from a location in the configuration space of a PCI-compatible device. 

- Message (Baseline, Vendor-Specific)
  
  - General in-band messaging and event reporting (without consuming memory or IO address resources)

### Split Transaction Protocol

Access to the address spaces in PCIe are accomplished with split-transaction `requests` and `completions`.

- Improvement over earlier bus protocols like PCI.

- Splits and generates two seperate TLP with independent routing for the `Request TLP` and `Completion TLP`. 

- Higher performance, but more overhead
  
  - A link is available for activity in the time between request and completion = BETTER PERFORMANCE
  
  - Requires and generates additional TLP for a single transaction = MORE OVERHEAD 

### Three methods of TLP routing

There are three methods of TLP routing. TLP Variants that targets one of the 4 `address spaces` will use one of the following routing method. 

1. Address Routing 

2. ID Routing

3. Implicit Routing 

| TLP Type                                                                        | Routing Method                                |
| ------------------------------------------------------------------------------- | --------------------------------------------- |
| Memory read (MRd), Memory Read Lock (MRdLk), Memory write (MWr)                 | Address Routing                               |
| IO Read (IORd), IO Write (IOWr)                                                 | Address Routing                               |
| Configuration Read Type 0-1 (CfgRd0-1), Configuration Write Type 0-1 (CfgWr0-1) | ID Routing                                    |
| Message (Msg), Message with Data (MsgD)                                         | Address Routing, ID Routing, Implicit Routing |
| Completion (Cpl), Completion with Data (CplD)                                   | ID Routing                                    |

## Root Complex

- Interface between CPU and PCIe busses. For example:
  
  - Processor interface
  
  - DRAM interface 
  
  - Configurations / chips 

- Logically connects PCIe's hierarchy domain into a single PCIe hierarchy. 

- A single fabric instance is consisted of the following: 
  
  - Rc 
  
  - Multiple Endpoints (I/O devices)
  
  - Switch
  
  - PCIe to PCI/PCI-X Bridge 

![](/Users/seahwang/Documents/GitHub/Cisco-Internship/figures/root%20complex.png)

### PCIe Root Port

- PCIe port inside a root complex. 

- Each Root Port has a divided hierarchy 

- Each hierarchy can be configured with one endpoint, and sub-hierarchy with one or more switch and endpoints. 

![3.png](/Users/seahwang/Documents/GitHub/Cisco-Internship/figures/3.png)

Fig 3.

### Switch & Bridge

- `Switch` is a device that allows a single `Root Port` to connect with multiple devices. It orks as a packet router, and uses the packet address and other routing information to identify the direction of packets. 

- `Switch` can have multiple `Downstream Ports`, and one `Upstream Port`. 

- A `Bridge` provides an interface with other busses. 
  
  - Interface includes PCI, PCI-X or other PCIe bus. 

![4.jpeg](/Users/seahwang/Documents/GitHub/Cisco-Internship/figures/4.jpeg)

Fig 4 

### PCIe Endpoint Device 5

- Endpoint Device is at the bottom or tree topology. 
  
  - And has a single `Upstream Port` 

- Has `Type 0 Configuration Space Header` 

- `Switch` or `Bridge` does NOT serve as a initiator or completer in a bus with PCIe transaction. 
  
  - Endpoint device can act as a `Requester` or `Completer` for Non-PCIe devices. 

- Three types of Endpoint Devices 
  
  - Legacy (PCI Endpoint)
    
    - I/O request and complete possible.
    
    - Can't perform memory transaction over 4G address (32 bit Address Register)
  
  - PCIe Endpoint
    
    - I/O request should not be done. 
    
    - Can perform memory transaction over 4G address. 
  
  - Root Complex Integrated Endpoint 
    
    - Root port inside the `Root Complex` logic. 
    
    - I/O request should not be done 
    
    - No `Power management` and `Hot Plug` 

Fig 5 

### Enumeration

-  Enumerates through BDF of a PCI device (explained below)

- Process of allocating bus numbers and system resources by configuration software. 
  
  - Done after finding system topology 

- in x86 PCIe architecture, `enumeration` takes place on BIOS hardware initialization process. 
  
  - All registers are set before boot loader. 

- System software can re-allocate enumeration. 

## PCI Configuration Space

- PCI bus uses `configuration space` to get the basic information of the connected device when a connection is made. 
  
  - Identifies what type of device is connected 
  
  - What protocol should be used 

- Configuration space includes set of registers such as `Device ID`, `Vendor ID`, `Class Code` etc that includes the device information. 
  
  - In PCIe, `extended configuration space` 
  
  - The registers in `configuration space` are allocated in memory. 

- All PCI devices have `BDF (Bus:Device:Fuction)` number. 
  
  - Scanning all BDFs will allow system to identify the device connected to it. 

- Configuration spae is 256 bytes long 
  
  - Can allocate address for 
    
    - 8 bit PCI bus 
    
    - 5 bit device
    
    - 3 bit function number (BDF)

![6.png](/Users/seahwang/Documents/GitHub/Cisco-Internship/figures/6.png)

Fig 6 BDF 

### BDF (Bus, Device, Function)

There are 

- 256 `Bus` 
  
  - 32 `Device` on each `Bus` 
    
    - 8 `Functions` on each `Devices`

`Bus`: Number of the bus. 

`Device` : Number of a certain device connected to the bus. 

`Function` : Number of functions that each devices can have. 

### Extended configuration space for PCIe

- PCI-X supports up to 4096 bytes. 

- Standard : First 4 bytes = 0x100. 
  
  - This implies for extended configuration space. 

### Standardized Registers

The following is the standard registers in `PCI Type 0 Configuration Space Heder` 

- `Device ID, Vendor ID` Register
  
  - Identifies the Device. 
  
  - 16bit `Vendor ID` is allocated by PCI-SIG. 
  
  - 16bit `Device ID` is set by the Vendor. 

- `Status` Register 
  
  - Used to notify the status such as: 
    
    - Operation of specific functions 
    
    - Errors

- `Command` Register 
  
  - Has `bitmask` that can activate/deactivate options such as: 
    
    - Interrupt Disable 
    
    - Memory Write and Invalidate 
    
    - Bus Master Enable

- `Class Code` Register
  
  - Shows the class (type) of a PCI device. 
  
  - For example: storage, USB controller, Network Card etc. 

- `Header Type` Register
  
  - Sets the layout of the header for remaining 48 bytes (64-16). 
    
    - 10h(16) onwrds in the figure below. 
  
  - Type 1: Root Complex, Switches, Bridges 
  
  - Type 0: Endpoint 

- `Cache Line Size` Register
  
  - Sets the cache line size 
  
  - Not applicable for PCIe 

- `Base Address` Register
  
  - Stores address of memory shared by the Device and OS. 
  
  - For Device Driver which is stored in memory. 

- `Subsystem ID (SSID) & Subsystem Vendor ID (SVID)` 
  
  - Used to distinguish a specific model of device. 
    
    - `Vendor ID` : Chipset Manufacturer 
    
    - `Subsystem Vendor ID` : Card Manufacturer



## PCIe Ordered Set
