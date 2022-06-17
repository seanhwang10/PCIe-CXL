# Table of Contents

- [PCI Express (PCIe)](#pci-express--pcie-)
  - [PCIe Basics](#pcie-basics)
  - [Structures - PCIe Layer](#structures---pcie-layer)
    - [Software Layer](#software-layer)
    - [Transaction Layer](#transaction-layer)
    - [Data Link Layer](#data-link-layer)
    - [Physical Layer](#physical-layer)

# PCI Express (PCIe)

Since CXL depends on the physical interface of PCIe, it is necessary to review the fundamentals of PCIe. I personally have one summer worth of experience with PCIe during my internship at Samsung Electronics, working with PCIe 5.0 in NAND SSDs.

## PCIe Basics

- `Bi-directional connection` that allows both transmitting and receiving.
  
  - Each link can only either transmit or receive data.
    
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
    

| Request Type | Posted? | Acronym |
| --- | --- | --- |
| Memory Read Request | Non-Posted | MRd |
| Memory Write Request | Posted | MWr |
| IO Read Request | Non-Posted | IORd |
| IO Write Request | Non-Posted | IOWr |
| Config Read Request | Non-Posted | CfgRd |
| Config Write Request | Non-Posted | CfgWr |
| Message Request | Posted | Msg |

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

//Add more details later.

## Link Training and Status State Machine (LTSSM)

- Sets and resets `links` and `ports` to allow packets to be sent.
  
  - Physical layer controlled, hardware based processor.
    
- Diagram of all states that a `link` can have.
  
  - `LTSSM` has 11 states, including `Detect`, `Polling`.
    
  - Each state has its substates
    

### Related concepts

- `Ordered Set`: Packets used for communication between devices in the physical layer.
  
- `Training Sequence` : Ordered sets used for link initialization and training. Acronym : TS.
  
  - TS1, TS2
    
- Cold Reset : Power on for system without power.
  
- Warm Reset : Reset without system losing power.
  

### Detect

- State for both ends to detect & identify the link.
  
- Initial state from Boot or Reset
  
- `Resets` all logics, ports, registers.
  

### Polling

- State to process the following
  
  - `Bit lock` : Synchronizes the `clock` for both ends of link.
    
    - we say a `lock` is secured when sync is made.
      
  - `Symbol lock` : Finding the boundary of continued bits
    
    - 8b/10b (for Gen 1, Gen 2)
      
    - PCIe 5.0 uses 128b/130b.
      
    - (Target data size / Actual required size for data send)
      
  - `Land Polarity` : Checks if lane is `D+` or `D-`
    
  - `Block Assignment` :
