# Implementing Task Scheduler for ARM Cortex-M4 processor.

In this project,

We implemented 4 user tasks that blink the 4 led with different delay. 

The main purpose od this project implementing Task Scheduler. We used SysTick Timer for create Systick Exception every 1ms. A global tick counter variable was used for calculate task delay.

Firstly ve created a stack memory stack for each user task, scheduler and one idle task. Size of the stack that allocated for the tasks can be different at different applications, In this project 1kb space was used for each task.

Secontly, The user tasks work in Thread Mode with PSP and the Scheduler works in Handler Mode with MSP.

Thirdly, We must tell about the meaning of "Context Switching". The Context Switching means saving the context of the task that is exiting, to it's on stack space and retrieving the context of task that will run on the CPU from it's own stack space.

In the Cortex M4 processor, We must save that registers for context switching: {xPSR, PC, LR, R12-R0}. The xPSR, PC, LR, R12, R3-R0 registers save automatically by processor when a exception is happen (The context switching do in exception handler). I call this registers as SF1 (Stack Frame 1). And we must save the rest of the registers(R11-R4) manually. I call this registers as SF2.

Forthly, We must initialize some dumy values to the stack frames(SF1+SF2) for each task for the first time of their execution. At this point we must careful when with xPSR, PC, LR values. The other ones's(R12-R0) value doesn't matter, can be 0.
  The xPSR should be 0x01000000 because of T-bit, T-bit is the 24. bit of PSR and it must always be 1 in ARM Cortex M Processors.
  The PC should be execution location of each task, at thit time it is taskx_handler funtion adresses.
  The LR should carry a speceial value(0xFFFFFFFD) that means "turn to thread mode with using PSP and not using floating point".

Fifthly, In the SysTick Handler we are updating the global_tick_count, unblocking the user tasks that task delay is done, and setting the pending bit of PendSV.

Sixthly, In the PendSV Handler we implementing the context switching operation. First the old task's context must save then the function which decides to next task that wil run must called. As finally, the next task's context must retrieve.

Seventhly, The tasks that do nothing currently must get blocked state and wait for it's global tick count. We can do this with calling the task_delay function. The function update the task's block_count, get the task to block state and calls an another function that sets the pending bit of PendSV.

Eighthly, idle function actually do nothing and it implemented for the times that when all user tasks in the blocked state. It is possible to put the CPU into sleep mode within this function.

Ninethly, We need a alghoritm to decide next task. We implemented Round Robin Methode that mean actualy cahenges the tasks in order. We have a basic alghorithm in this code. Even so, having a look to it can be helpful. It can become little confusing because of the idle task.

Lastly, We must be careful about variables that shared between Thread Mode code and Handler Mode code because of "race conditions". We have a sample of variable like this in task_delay function. Not a good solution but we used that method for this simple application: Disable all interrupt before reaching the shared global data, and enable the interrupts after using it.

This code was written on STM32CubeIDE for STM32F407G-DISC1 board, compiled with GCC arm compiler, and tested.
