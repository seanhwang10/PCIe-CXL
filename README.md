# PCI Express (PCIe)

Since CXL depends on the physical interface of PCIe, it is necessary to review the fundamentals of PCIe. I personally have one summer worth of experience with PCIe during my internship at Samsung Electronics.

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
