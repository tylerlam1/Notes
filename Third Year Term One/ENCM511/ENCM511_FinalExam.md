# ENCM 511

## Abbreviations

* CPU = Central Processing Unit
* CCU = Computer Control Unit
* DMA = Direct Memory Access
* CLLR = Compile, Link, Load, and Run
* PSP = Personal Software Plan
* PIP = Personal Improvement Plan
* LSD = Little Stupid Details
* WIDFI = When in doubt, fake it
* WAIL = Worry about it later
* WAIN = Worry about it now
* PEP = Personal Exam Process
* CC = Condition Code
* PF = Programmable Flag
* PF = Port F
* GPIO = General purpose Input/Output
* JSR = Jump to Subroutine
* RTS = Return to Subroutine
* REB = Logic Lab Remote Extender Board
* PSPP = Practice Safe Programming Practices
* JEADI = Just Enough Additional Development Interfaces
* MVP = Minimum Viable Product
* ICE = In Circuit Emulator
* IRQ = Interrupt Request Line 
* MOSI =  Master out, Slave In
* KIS = Keep it Simple
* COTAAT = Change only one thing at a time
* TTCOS = Time-Triggered Cooperative Operating System
* RETS = Return from subroutine
* TFD = Test First Development
* TLD = Test Last Development
* GQM = Goal, Question, Metric
* EVT = Event Vector Table
* FILO = First In Last Out
* ASTAT = Arithmetic Status
* SPI = Serial Peripheral Interface
* KFC = Keep Fingers Crossed
* PPP = Profound Procrastinated Programming

In order to set up the GP Timer, check out the reference sheet. 

```c
void initialize_And_Start_Libraries() {
    Init_GP_Timer(GPTIMER);
    Start_GP_Timer(GPTIMER);
}
```

Basically, you go ahead and initialize and start the GP Timer. The GP Timer was used in Lab 4 for the temperature sensor.

Here are the differences between the GP Timer, CoreTimer and the WatchDog Timer:
* The CoreTimer acts like an hourglass -- just counts down from a particular time and sends a done signal
* The GP Timer has a connection to the outer world (not sure what that means)
* Watchdog Timer is basically like a CoreTimer, but it can take control of the entire system if needed

Let's look a little bit at the differences between Harvard and Von-Neumann Architecture.

### Harvard Arhitecture and Von-Neumann

Harvard Architecture has TWO address busses,TWO data busses, and TWO Control Busses. It also has a ROM and a RAM unlike the von-neumann architecture.

The Von-Neumann Architecture has ONE address bus, ONE data bus, and ONE Control Bus. The Control Bus sends timing commands to the memory and peripherals and gets acknowledgement signals back.

When you're taking a look at the standard instruction cycle of a microprocessor, you know that these are typically the instructions taken:

Reset -> Fetch -> Decode -> Execute -> Writeback 

By uselining Pipelining, we can speed things up. basically, we can go ahead and fetch another instruction while decoding the first one, fetch another instruction while the first one is executing, and so on. The only caveat is to ensure there is enough memory.

It is important to keep in mind that each stage must be as long as the longest instruction stage. The solution is using RISC (Reduced Instruction Set Processors) which no complex instructions.

On the Von-Neumann processorm, you cannot fetch data and instructions at the same time because there is only one data bus and one address bus. However, the Harvard Processor can perform this process of fetching data and instruction at the same time as there are two data busses and two address buses.

SISD - Single Instruction Single Data
SIMD - Single Instruction, Multiple Data

Basically, one instruction in SIMD does things with R registers with memory location P in first hald cycle of processor clock, and does things will the S (shadow) registers in the second half of the processor clock

At the end, the programmer must combine R and S results. Automatically handled by the optimizing compiler.

MIMD - Multiple Instruction Multiple Data

VILW - Variable Length Instruction Word - To save program memory space, some processors have 16-bit and 32-bit versions of the same COMMON instruction.

### Files Generates

The CrossCore Compiler uses .cpp and .asm files to create .doj files. The Linker uses the .doj and .dlb files to create the .dxe file. While the .dxe is an executable, the .dxe must become a .ldr file to go to the loader in the processor flash memory.

In general, here are the steps:
* The preprocessor takes the source files (.cpp) and converts them into expanded source files. The compiler takes the expanded source files and converts them into .asm.
* The assembler takes the .asm files and converts them into .doj files. The linker takes the .doj and .dlb files to create an .dxe executable to be loaded into flash memory by the loader.

Here are some of the basic differences between a Microprocessor and a Microcontroller.

### MicroProcessor

A Microprocessor itself is completely useless. It needs external peripherals to interact with the outside world. The issue with adding external devices is that there are:
* Many pins - this increase the time of design, and failure rates increase. The cost also increases with all these pins.
* Another issue is that you don't typically need all this flexibility with pins in real life.
* Probably need to redesign and reassemble the same thing over again

### MicroController

In a Microcontroller, you put a limited amount of most commonly used resources inside the chip. This is typically good enough for most applications.

The advantages of the Microcontroller:
* Pin Count down
* Reliability up
* Design time down
* Cost down
* Common software / hardware design environment available from manufacturer

Problems:
* There are two different types of memory (with different speeds when using) - ON CHIP and OFF CHIP.
* External components are still there - which need the DMA capability.
* It can't wait or poll.

## C++

.extern tells the compiler that the definition is an external file. Name mangling is when the compiler messes up a C++ function name when overloading (multiple overloads have the same name). 

Voltaile registers are registers that change in response to hardware, so the software need to take their erratic changing into account. ssync makes reads and writes happen now.

#### ASM

In Blackfin, the distinction between data registers (R registers) and pointer registers (P registers) are made. All R registers are 32 bits, and all P registers are 32 bits. Volatile registers are R0, R1, R2, R3, and non-voltaile reghisters are R4, R5, R6, and R7.

SP points to the top of the stack. FP points to the bottom of the stack.

When changing register values, here are some examples.

```assembly
    R1 = R2;
```
That is for setting registers equal.

Let's add and subtract, and do some bitwise.

```assembly
R0 = R1 & R2;
R0 = R1 | R2;
R0 = ~R1;
```
Please keep in mind you cannot add a pointer and a data register together.

```assembly
R0 = [P0]; //load a 32-bit value from the memory location at P0 into R0
R0 = W[P0]; //store a 16-bit value from the memory location at P0 into R0;
R0 = W[P0](Z); //store a zero-extended unsigned 16 bit value from memory location at P0 into R0
R0 = W[P0](X); //store a sign-extended (signed) 16-bit value from memory location at P0 into R0

R0 = [P0+4]; //this is how you offset
[P0] = R0; // store a 32-bit value from R0 into memory location at P0
W[P0] = R0; //store lower 16-bits from R0 into P0
```

Function arguments are passed into the R0, R1 and R2 registers. Any further arguments needed are placed on the stack.

Return values in asm subroutines are typically R0. If you need more than 32-bits, then use R1.

Here is some boilerplate example.

```assembly

.section program;

.global _readProcessorCycles_ASM;

_readProcessorCycles_ASM:

_readProcessorCycles_ASM.END;
RTS;
```

When calling functions in ASM, you need to use the .extern keyword.

```assembly

.extern _Subroutine_Name;
CALL _Subroutine_Name;

```

When defining function prototypes for assembly functions, please make sure to have the extern "C" in front of the function declaration. This is for something called name-managling, which means that the compiler changes the compiled name to a assembly name. Using *extern "C"* turns it off. 

As in MiPs, if you're calling another subroutine from a subroutine, you need to save the non-volatile registers in the stack. Below is an example of how this is done.

```assembly
[--SP] = FP;
FP = SP;
SP += -16;

[--SP] = R4;
[--SP] = R5;

R5 = 5;

// do stuff

R5 = [SP++];
R4 = [SP++];

SP = FP;
FP = [SP++];
```

## uTTCOS

uTTCOSg is a time triggered cooperative operating system. u emans that uTTCOS has a low memory cost and g is the difference between the 2015 and 2016 versions.

1. Initialize the TTCOS

```cpp
uTTCOSg_Init_Scheduler();
uTTCOSg_AddThread(function, DELAY, THREAD);
uTTCOSg_Start_Scheduler();

while(1) {
    uTTCOSg_DispatchThreads();
}
```

### Some terminology

* Super Loop: Totally adequate in many embedded appliactions where the system only requires basic functionality, no timing issues are present.
* Pre-emptive Scheduler: uses an idle() when not doing anything useful and uses interrupts to wake system up. 
* Co-operative Scheduler: how uTTCOSg works. Has a list of tasks that get run based on the current time and period and delay. Each task has a Run-Me-Now variable so that DispatchTask() will run when it is not zero.

## Interrupts

There are three approaches when reading from an external device.

1. Wait - Wait till device ready. Read the control register and keep waiting until the control value goes from 0 to 1. Processor is always active and you're persistently waiting.
2. Polling - This checks each device sequentially. Ex. hecks device 1 -> if not ready checks 2 -> and so on

An interrupt is basically an alarm clock. It will cause all else to be abandoned and execute an ISR (Interrupt service routine). All the registers changed by the ISR must be saved and restored at the end.

### Software Interrupts

Software interrupts occur when you write to ILAT (which is ready only) focusing a fake interrupt.

This is done using raise. We write to ILAT so long as the IMASK is set appropriately. Software interrupts will occur every time around a loop but a real interrupt will not.

### CoreTimer Interrupts

CoreTimer interrupts are used in assignment 2. 

* Is EVT6 called IVTMR on core timer vector table
* to set EVT table entry y
* *pEVT6 = (void) (*coretimer_ISR) or registerHandler(ik_timer,coretimer_ISR);


## CoreTimer

On all processors is a dedicated timer. The core timer is clocked by the internal processor clock and is typically used as a system tick clock for generating periodic operating system interrupts. CoreTimer works on both the BF533 and BF609 but the simulator period needs to be adjusted - 0x100000 = 5 seconds on real time 609 so much smaller needed on the 533 simulator.

* TCNTL = Basically controls the start/behaviour of the timer (similar to coffeepots control bits)

On the TCNTL - send 0x0007 to make active. May have to reset sticky bit to 0 so tht you can get signal when the timer goes off again.

* TSCALE - storing the scaling value that is one core clock (CCLK) cycle less than the number of cycles between each decrement of the timer count

* TPERIOD - Tolds the timer period. Cannot be written when the timer is running so should be set before the timer is started.

* TCOUNT - holds the current count for the timer. 

## GP Timer

* TIMER_ENABLE(Timer Enable Register)and the TIMER_DISABLE(Timer Disable Register) can be set as the SET and CLR registers for the GP Timer