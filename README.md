# Interrupt Controller
This is a simple 8 input interrupt controller.Written in verilog. Currently it supports two modes, polling and custom priority.
In polling, there are no priorities. All the interrupts arechecked periodically and if any one of them is active then it is serviced.
 
In the custom priority mode, the priorities are set during the initialization phase. The controller then polls the sources in this custom order.

## Operation
After Reset, the controller waits for commands from the master (processor) on the bus. The lowest 2 bits determine the mode of operation.
    01 - Polling mode
    10 - Priority mode.
It keeps waiting for valid input mode.

## Polling
The bus is driven as xxxx_xx01 for exactly 1 clock by processor. Controller knows this is the polling mode. It then enters the polling state where it keeps checking for all interrupt sources in cycle. If any interrupt is found active then intr_out is set to 1. After that, the controller waits for an acknowledgement from the processor. Once the processor gives acknowledgement, exactly for 1 clock cycle on intr_in by a high -> low -> high transition, the controller starts driving the bus with the condition code of 01011_intrID. The intrID part is the ID of the source currently being serviced.

Controller keeps this data on the bus it it receives another acknowledgement from the processor on the intr_in pin in the same mannter (High -> Low -> High).
After that, the controller waits for the confirmation from the processor that the interrupt has been serviced. Processor, once done, sends another acknowledgement on the intr_in pin in the same mannter (High -> Low -> High) along with the condition code of 10100_intrID on the bus. This interrupt ID must match the one sent last by the controller. 

If either the condition code or the intrID does not match then the controller goes to reset state. Else it just checks (polls) the next source and continues in this cycle unless reset.

## Custom Priority
Working of this mode is exactly similar to the polling exept that it does not poll the sources in order.
After reset, if the input is xxxyyy10 then the controller is in custom priority mode. xxx is the source ID of the highest interrupt and yyy is the ID of the second to highest. The processor needs to give 4 such cycles to and give the priorities for all 8 sources.

After that the controller goes on checking from the highest to lowest source to check if any of them is active. If it is then the operation continues similar to the polling mode.
The only things that change are the condition codes that are sent to and from the processor. The controller sends 10011 and the processor acknowledges with 01100.

## Condition Codes

    Polling:
        From Controller     -   01011
        From Processor      -   10100

    Custom Priority
        From Controller     -   10011
        From Processor      -   01100

## Simulation
Code to run the simulation is given below:
```
iverilog pes_intrCntrl.v pes_intrCntrl_tb.v
./a.out
gtkwave pes_intrCntrl_tb.vcd
```
![image](https://github.com/spurthimalode/pes_pes_intrCntrl/assets/142222859/47485fba-4541-4bc0-9fbf-7090b184bcdf)

## Synthesis
Code for the synthesis is given below:
```
$ yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
yosys> read_verilog pes_intrCntrl.v
yosys> synth -top pes_intrCntrl
yosys> dfflibmap -liberty ../my_lib/verilog_model/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys>flatten
yosys>abc -liberty ../my_lib/verilog_model/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```
![image](https://github.com/spurthimalode/pes_intrCntrl/assets/142222859/1dbec793-2fab-4840-a9ec-19267e2a9678)
![image](https://github.com/spurthimalode/pes_intrCntrl/assets/142222859/53497259-19c9-4f1e-92bd-62f245cff3da)

## Gate level Simulation
Code for gate level simulation is given below:
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v pes_intrCntrl.v pes_intrCntrl_tb.v
./a.out
gtkwave pes_intrCntrl_tb.vcd
```
![image](https://github.com/spurthimalode/pes_intrCntrl/assets/142222859/d3efbe7c-74ae-4810-ba74-051c3f140b69)
