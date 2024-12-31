java c
EEE8087
Real Time   Embedded Systems
Worksheet 4. The Time-Slicing StructureThis   week   we   startwork   on   the   central   components   of   an   elementary   real-time   operating   system (RTOS) that divides the processor's time between separate   user tasks.   These   tasks   will   need   to      communicate with the system, and will request its   services   by   means   of a   'software   interrupt'.
Implementation of Software Interrupts
A software   interrupt   is known as a 'trap'.   It   causes the   processor   to   respond   in   a   very   similar   way   as   it does to a   hardware   interrupt, and so allows system   calls from   the   user   program   and   interrupts from   hardware devices to enter   the operating system in   a   consistent   way.
There are   16 trap   instructions available,   numbered 0   to   15, and written
trap   #0
...
trap   #15
Each of the   16 trap instructions may have   its   own   interrupt   service   routine   (ISR). After   pushing   the   PC and SR, the processor then accesses a   table   in   low   memory,   at   address   80H.   As   for   the hardware   interrupts, the table contains a 4-byte value corresponding to the address   of the   ISR for   each software   interrupt. Vectors for the two types of interrupt will   normally   be combined   into a   single   block of   code.      ;interrupt vectors   org$64                ;origin   64Hhvec1dc.lhisr1          ;address   of   hardware   ISR   1hvec2dc.lhisr2            ;   ...   etc   org$80                ;origin   80Hsvec0dc.lsisr0            ;address of software   ISR 0svec1dc.lsisr1            ;   ...   etc
Controlling Interrupts
There   is,   however, an   important difference   between   hardware and   software   interrupts.   Hardware   interrupts are   in order of priority, with 7   being the   highest   priority   and   1   the   lowest.   If two   hardware interrupts occur at the sametime, then the one at the   higher priority will   be   accepted   and   the   other       one will   be kept waiting   until the first   ISR   has completed.   If a   hardware   interrupt occurs shortly   after another one,   but while the   ISR for the first   interrupt   is still   in execution, then the processor will   again   compare the priorities of the two   interrupts.   If the new interrupt   is   of a   higher   priority,   then   it will   interrupt the   lower priority   ISR.   If the new interrupt   is at   a   lower   priority   than   the   currently   executing   ISR,   it will   be   kept waiting   until that   ISR completes.
Software   interrupts do not   behave   in an analogous way. Since   the   processor   can   only   execute   one   instruction at a time,   it would   be   impossible for two software   interrupts to occur at the sametime, and   unless a programmer includes atrap   instruction within   an   ISR, there will   also   be   no   occasions   on which a trap takes   place during the processing   of another trap.   There   is   therefore   no   point   in prioritising the software   interrupts, and all   16 are at the   same   priority. There   is,   however,   the   question of the relative   priority of the hardware and software   interrupts. What   if   a   hardware   interrupt   is   raised at the sametime as the   processor is executing   a   software   interrupt   instruction?   This   is   handled   by assigning all the software   interrupts to priority   level   0.   Processing   of a   software   interrupt   is therefore interruptible   by a   hardware   interrupt at any of the   priority   levels   1   to   7.
Within your system,   however,   regardless of the type of interrupt being   processed,   you will   want   to   prevent the acceptance of any other interrupt. Your system   will   therefore   be   completely   uninterruptible. Once entered,   it will always   run to completion and then   return   to   the   user   task   that         was   running when the interrupt was   raised. You will therefore   need to disable   interrupt acceptance,   the   procedure for which   is explained   now.
Using the simulator, examine the   16-bit status register.   Bits   8,   9   and   10   (labelled   'INT')   hold   a   3-bit   value that   represents the interrupt   priority mask. When an   interrupt   is   accepted, the   mask   is   set   to the   priority level of that   interrupt. A hardware   interrupt will only   be   accepted   if   its   priority   is   greater than the current setting   in the mask.   Normally, the   mask   is set   to   000   (decimal   0)   thereby   allowing the acceptance of any hardware   interrupt.   However,   it will   remain at zero   during   its   response to   a   software   interrupt, since that   is the   priority of these   interrupts, and will thereby allow the   hardware to   interrupt the software   ISR.   If you wish to   prevent this, then the following   instruction,   placed   at the   very start of a software   ISR, sets the mask   to   binary   111   (decimal   7).   Any   hardware   interrupts will   now be disabled, and   held   pending   until the mask   is   returned to   zero.
or                #$0700,sr                            ;disable   hardware   interrupts
The status register will   have   been automatically saved on the stack   at   the   start   of the   interrupt   servicing. On execution of the 'return from exception'   instruction (RTE),   it will   be   restored,   and the   mask   reset to the zero value that   it   held   previously, thereby allowing the acceptance   of any   hardware   interrupt that   might   have   been raised in the meantime   and   is   currently   pending.
If you want to enable hardware   interrupts at any   other time, the following   instruction   will   set   the   mask to zero.
and       #$f8ff,sr                                       ;enable   hardware   interrupts*
Practical Work
Assessment   question
Work in groups of   three on this question.   Your   submission should include   the   following.
Your software, including the RTOS and the test programmes you used to   demonstrate   it.   Submit the source code, not the assembler output listing.
Documentation: a .PDF file is preferable, otherwise   .DOC.
The names and student numbers of   all three group   members should be shown   on   the   software   heading   and   on   the   frontpage   of   the   documentation.
There are therefore two items to   be   submitted:   a   single   software   file,   and the   documentation.   These   items should be placed into a single zipped file, and uploaded to   a   Canvas   submission point to   be   advised.   The submission deadline is   2pm on Friday   17th January,   2025.
Each item will now be described in   detail.
The   RTOS
The work consists of writing a basic time-slicing system,   along the   lines   of the   one   discussed   in   the lecture.   It should allow the execution of several concurrent   user tasks, with   support for task   scheduling and inter-task communication. An outline   programme   is   provided on   Canvas,   but   you   will   write the service   routines (including   reset), the scheduler, and the   user tasks for   each   application.
The user tasks are   located   in   memory, above the system   itself.   Each task   has   its   own   area   of   memory, with the programme code at the lowest   address,   data   above   it,   and   top-of-stack   at   the   next   address above this task's   memory area.   For example, the following task occupies   memory   between      2000H and 2FFFH.   It   has   its code at 2000H, data   at   2C00H,   and   stack   at   the   top   of the   data   area.Address   2000HProgramme code2C00HData3000HTop-of-stack
The system runs in the foreground, and   is   entered following   either a   timer   interrupt,   a   software   interrupt from one of the tasks requesting service, or another   hardware   interrupt.
The following system calls should   be supported   by means of software   interrupts.   They   can   either   each   be allocated to a separate trap   number, or (as   in the   demonstration   system)   they   can   all   be   called on the same trap, with one of the registers   used to   hold   a value   identifying   the   requested         function. Some of the calls also   require additional parameters   in   other   registers.
1.   Create   task
Function:                               A   currently      unused   TCB   is   marked   as   in   use   and   set   up   for   a   new   task.
It   is   placed on the ready list. The   requesting   task   remains   on the   ready   list.   Two
parameters   indicate the start and end of the memory area   occupied   by   the   new   task   and   its   data.
Parameters:                The start   address   of the   new task,
The address of   its top-of-stack.
2. Delete   task
Function:                               The   requesting   task   is   terminated,   its   TCB   is   removed   from   the   list   and
marked as   unused. Any memory allocated to it   is   returned to the   system.
Parameters:                  None.
3.   Wait   mutex
Function:                               If   the   mutex   variable   is   one,   it   is   set   to   zero   and   the   requesting   task   is   placed
back onto the   ready list.   If the mutex   is zero,   the   task   is   placed   onto   the   wait   list, and subsequently transferred back to the   ready   list when   another task            executes a signal mutex.
Parameters:                  None.
4. Signal   mutex
Function:                               If the mutex   variable   is   zero,   and   a   task   is   waiting   on   the   mutex,   then   that   taskis transferred to the ready list and the   mutex   remains   at   zero.   If the   mutex   is   zero   and   notask   is waiting, the mutex   is set to one.   In either   case, the   requesting   task   remains   on the   ready   list.
Par代 写EEE8087 4W Real Time Embedded SystemsR
代做程序编程语言ameters:                  None.
5. Initialise   mutex
Function:                               The   mutex   is   set   to   the   value   0   or   1,   as   specified   in   the   parameter.
Parameters:                  0 or   1.
6.   Wait   time
Function:                               The   requesting   task   is   placed   onto   the   wait   list   until   the   passage   of   the
number of timer interrupts specified   in the   parameter, when   it   is transferred   back to the   ready   list.
Parameters:                  Number of timer intervals to wait.
An additional function   is executed automatically at start-up, or   if the   user   presses the   reset   button.
System reset
Function:                               The   system   is   initialised: all   internal   variables   are   reset,   and   each   TCB   is
marked   as   unused.    A   TCB   for   task   T0 is   then   created,   and   T0   becomes the   running task.
A   practical   RTOS would also   include the following functions. These are not    required   in your   submissions, but they are   mentioned   here for completeness of these   notes.
7.   Wait I/O:
Function:                               The requesting task   is   placed onto the   wait   list,   until   an   interrupt   signifies
completion of an   I/O operation, when the task   is transferred back   to   the   ready   list.
Parameters:                  None.
8. Allocate memory
Function:                               For   tasks   that   require   a   large   amount   of   memory,   it   is   more   efficient   to   allocate   it   as
required when the task   runs. A large area of memory   is therefore   kept   free within   the      system, and   a   block   from   it, of   say   16   kbytes,   is   returned   to   the   requesting   task.   If   the
request   is satisfied, the   requesting task   remains on the   ready list.   If there   is
insufficient free memory available, then the   requesting task   is   put on the wait   list   until memory is   returned when another task   terminates.Parameters:                On return, the   start   address   of the   allocated   memory   is   held   in   the   parameter   register.
The system assumes that a default user task, T0,   is   present.   The   system   runs   this   task   immediately   after a reset.   It will   need to   be located at   a   predetermined   address, which   will   be   coded   into   the reset function.
Your system should   be robust, and deal with errors   in   an   intelligent   way.   For   example,   what   if the   user tries to create more tasks than there   are available   TCBs?
Test   programmes
Test your system using the following   programmes.   You will recognise these   as   modified versions   of   the questions   in   last week's worksheet, this time running   under your   RTOS.
1.                        Testing create task and wait time functions.A stopwatch counts in seconds.   It starts when   button   0   is   pressed   and   stops when   the   button   is   pressed again. The display shows two digits, and should   be   neatly   formatted   with   unused   digits   blanked.   It   is   programmed as follows.
Timer interrupts are set to   100ms. Task 0   starts task   1,   and   both   tasks   run   concurrently.   A   shared      variable      running   is   held   in a memory location, and   is set   by T1   and   read   by   T0.   T0   displays   a   2-   digit value on the 7-segment display,   initialised to zero.   If   running   is set,   T0   increments   the
display, then waits for   10 time intervals, and then   repeats.   If running   is   not   set,   T0   does   nothing.   T1 tests   pushbutton 0.   Each time the   button is   pressed,   running   changes state.
2.                        Testing   initialise   mutex, wait   mutex, and signal mutex functions.
A   radiation monitor contains two devices that each   generate   a   pulse   each time   a   particle   of   ionising   radiation   is detected. The system alternately samples each detector for   100   ms,   and   records   the
count,   a   and b, from each device.   It also keeps   a   running total   of the two   counts,   c   =   a   +   b.   The
system   measures the time since   it started, and after 8 seconds   displays   the   average   count   per
second   between the two detectors, that   is, (2c   / 2) / 8,   or   c   /   8.      (Because   it   monitors   each   detector      for only half the time, there   is an implicit   assumption that   the total for that   detector   is   twice   its   actual   count.)   If at anytime during the 8-second measurement interval   the   total   count   c   exceeds   a   certain   critical value, the system immediately displays a danger warning   by   lighting   the   RH   LED,   and   then continues running for the remainder of the 8 seconds. The   LH   LED   is   used   to   indicate   an   internal
error, as will   be explained   later.
It would   be convenient to indicate a detection   by   pressing   a   button,   but   since we   need   to test   this
system   in real-time, and   it   is   not   possible to press the   buttons   hundreds of times   per   second, we   will   simulate a fast arrival   rate of detection   pulses simply by   programming the   system   to   increment   the            two counters continuously. The programming   is as follows.
Timer interrupts are set to   100ms. Three tasks run   concurrently. After   performing   any   initialisation,   tasks 0 and   1   update the counters, either   a   and   c, or b   and   c,   using   the following   instruction
sequence which   is   repeated continuously.
move.l                                    a,d0
add.l                                    #1,d0
move.l                                    d0,a
move.l                                    c,d0
add.l                                    #1,d0
move.l                                    d0,c
At the end of this sequence, variable   c   is tested to determine whether   it   has exceeded   the   critical   value, and   if so, the   RH   LED   is   lit to indicate   a   danger   condition.
You will   note that variable   c   is   updated by both tasks, which are   liable   to   interfere with   each   other. A   mutex operation   is therefore   used to enforce exclusion and   prevent simultaneous   updates. A call   to      mutex wait should   be   placed   before the three instructions that increment this variable,   and   a   signal call at   their   end.
Task 2 waits for 8 seconds, then displays the   result   c   /   8 on the   7-segment   display.   (The   division
can   be done by shifting; see the notes   for week   1.)   The   result   may   be   shown   in   hexadecimal,   which      avoids doing a conversion to decimals. This task also   checks   that   the   mutex functions   have   worked:   if the final value of    a   + b   -   c   >   1, then there   has   been an   error   and   this   is   indicated   by   lighting   the
LH   LED.
Submitting the test   programmes.
Your test   programmes should   be   included at the end of your   RTOS code.   Place   both   programmes   together, calling them prog1   and   prog2. At the start of the   user code,   put the   following   branch


instructions.


org usrcode
bra prog1
; bra prog2






By commenting out one of these   instructions, as has   been   done for   prog2   above,   it   is   easy to   select the other for running.
This   is a much better arrangement than keeping   two   versions   of the   RTOS,   with   a   different   test   programme in each, or pasting   each test   programme   into the   RTOS whenever   it   is   required.
However, since   both programmes will   be assembled together, you will   have to   ensure that   you   use   different   identifier names within the two   programmes. You could, for example, start   all   labels   and
variable   names within prog1   with 'p1' and those   in    prog2   with 'p2'.
Documentation
This consists of a   user manual.   It will explain your system   to   a   user,   and will   therefore   focus   on   what   it does and how to   use   it.   It will   include a   brief   overview   of   how the   system works   internally,   but   only      to an extent that   is required for the   programmer to use the   system   correctly.   It   should   be   structured   as follows.
1.
A general description, including, for example:
explanation of the principles of multitasking and time-slicing,   and   how they   are   implemented;   description of the memory layout of the   user tasks as   shown   above;
explanation of the startup   behaviour: a   default task   runs, which   may then start other tasks.
2.
An   itemised description of each of the user functions and their   parameters.   Explain   all aspects   of the   behaviour of each function that are of interest to the   user.   For   example, when   a   new task   is   created,      does   it   run   immediately, or is   it   put at the end of the   ready   queue? What   if two   tasks   are waiting   for            the sametime, and so become ready together? Your descriptions   should   also   state   how   long   each          function takes to execute. This time should   be quoted   in terms of instructions,   and   may   be within   a
specified   range, e.g., a particular function might   execute   in   30 -   50   instructions.
3.
A short   note about your two test   programmes.   How do they demonstrate that the   system   is working?   Were there any unresolved   problems with them?
With   normal typeface and spacing   (such as   used here)   it would   be   reasonable to   expect   a   length   of
no   more than five or six pages. Submission   in   .PDF format   is   preferred,   but   .DOC(X)   is   also   acceptable.

         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
