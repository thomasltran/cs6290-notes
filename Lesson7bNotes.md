# ReOrder Buffer Rev. 2

The ReOrder buffer is found in all modern high performance processors. It is used to ensure correct program outcomes in out-of-order processors.

## Exceptions in Out-of-Order Execution

The major drawback of Tomasulo's algorithm is that it does not handle exceptions very well.

When an exception occurs in OOO processors the instructions after the exception producing instruction have been executed, resulting in incorrect outcomes.

- MUL and SUB waiting for F2
- F6 is 0 (divide by 0), detected in cycle 40
  - PC for div should be saved -> jump to exception handler -> comes back to div instr -> continue executing 2-4
    - but by cycle 40, have already done WR other res for 2-4
      - 1 can use F0 from 3 too, if OS tries to restart instr (i.e. for page fault)
      - if load has page fault -> page from disk -> back to executing load and other instr, but instr after load might've finished executing

<img src="./png/Lesson7bNotes-image.png" alt="alt text" width="650" />

## Branch Mispredictions in OOO Execution

Tomasulo's algorithm does not handle branch mispredictions in OOO processors well.

- div takes 40 cycles, branch takes long time to mispredict
  - have fetched add and updated r3 before realizing it's mispredicted
- instr can complete wr before preceding instr have been verified and realize it shouldn't have been executed

Phantom Exceptions - if a branch is mispredicted a number of instructions are executed before the misprediction is detected. It is possible to generate an exception within these executed instructions, now the processor must recover from an exception that should not have occurred as well as the misprediction.
- mispredicted, exception on the div

<img src="./png/Lesson7bNotes-image-1.png" alt="alt text" width="550" />

Exception handling should not occur until the processor is sure they are not phantom exceptions.

## Correct OOO Execution

- Execute OOO
- Broadcast OOO
- Deposit values in registers In-Order. This can be achieved with a ReOrder Buffer.
  - if we deposit ooo, if one of earlier instr shouldn't have been done (i.e. mispred or exception)
  - means all prev instr finished successfully

A ReOrder Buffer remembers the program order and keeps the results until it is safe to write them.
- tomasulo is ooo for deposit, so we need rob
- instead of writing to reg when res produced, go through rob

## ROB Part 1

The ROB, table of entries, has three fields

- The data being held by the ROB (val produced by instr)
  - when instr finishes and broadcast, write here instead of to reg
- Done bit, which tracks the validity of the data in the value field
  - will give rob to instr before it writes res
- Which register is holding the data
  - which reg to write to

Instructions are kept in program order.

There are two pointers associated with the ROB:

- Commit tells which instruction is to be completed next
- Issue tells where the next issued instruction is to be placed in the ROB

<img src="./png/Lesson7bNotes-image-2.png" alt="alt text" width="550" />

## ROB Part 2

How the ROB interacts with the IQ, RS, Regs, and the RAT.

The steps for Issue when using a ROB

1. Get the instruction from the Instruction Queue
2. Get a free Reservation Station
3. Get the next ReOrder Buffer entry (at the Issue pointer)
  - increment issue ptr at next entry
4. Look up the source registers in the RAT
5. Change the RAT for destination registers to point to the ROB entry (ROB 6) instead of the RS

### Dispatch with the ReOrder Buffer

Steps for dispatch when using a ROB

1. Find results where the tags match.
2. Only instructions where all the inputs are ready are considered for dispatch
3. Pick one instruction for each functional unit
4. Free the reservation station. This is different than Tomasulo's algorithm (happened during broadcast, RS served as name for res).
  - With a ROB the instruction is broadcast with the ROB name rather than the RS name, so the RS can be freed sooner.

- executes and carries the ROB6 tag (name of the res)

<img src="./png/Lesson7bNotes-image-3.png" alt="alt text" width="600" />

- free RS sooner with rob

<img src="./png/Lesson7bNotes-image-4.png" alt="alt text" width="600" />

## ROB 3

### Broadcast with the ReOrder Buffer

- tag for rob entry since RS freed

The steps for broadcasting with a ROB:

1. Capture the result in the waiting Reservation Stations
2. Write the result to the ROB, mark instr as done
  - without rob would update RF/RAT

### Commit with the ReOrder Buffer

- must first commit all prev instr, each cycle check if instr at commit point done and increment
  - take val and write to reg in RF
  - update RAT to the reg in RF

The steps to commit with a ROB

1. Look at only the oldest instruction in the ROB
2. Wait for this instruction to complete
3. Copy the result from the ROB to the register. This means that register writes are occurring in order.
4. Update the RAT with the destination register
5. Free the ROB entry

<img src="./png/Lesson7bNotes-image-5.png" alt="alt text" width="600" />

## Hardware Organization with the ROB

- before rob

<img src="./png/Lesson7bNotes-image-6.png" alt="alt text" width="550" />

- with rob
  - head = issue, tail = commit
  - RAT points to rob entries

<img src="./png/Lesson7bNotes-image-7.png" alt="alt text" width="600" />

The ROB has a head, the next instruction to be issued, and a tail, the next instruction to commit. Entries in between these two are being executed.

The ROB is used to:

- Remember the program order
- Temporarily store an instruction's result
- Serve as the name (tag) for the result
- RS handles the last 2

<img src="./png/Lesson7bNotes-image-8.png" alt="alt text" width="600" />

## Branch Misprediction Recovery Part 1

- predict pgrm doesn't branch, execute the 3 instr (but it's misprediction)
- issue
  - ld
  - issuing branch produces no output
  - issue the mispredictions for add, mul, and div
    - entry for r2 in rat changed from rob3 to rob5 from add to div
- assume ld takes long time (since cache miss)
  - bne, add depends on this so can't execute
  - mul can continue if r3 r4 ready
    - without rob, val could be written to reg and don't know how to undo once misprediction figured out
    - with rob, res goes into rob entry
    - same for div
  - regs still unmodified, only res stored in rob entry at this point

<img src="./png/Lesson7bNotes-image-9.png" alt="alt text" width="600" />

- ld completes, marked as done
  - now can do brnach and add
- commit instr
  - write 700 to r1, point rat to rf r1
  - suppose branch takes longer to figure out than add
    - add completes, val for res produced in rob entry
    - wouldn't have been written to rf even with tomasulo bc renaming for r2 in rat not pointing to rob3
  - branch resolved, figure out misprediction, marked as done
    - annotate misprediction in rob val
    - once commit at branch, realize wrong instr have been getting fetched
    - all instr prior to branch have been committed (updates reflected in rf)
      - none after branch instr have been committed
      - reverse issuing of the mispredicted instr by moving the ptr back to the branch
      - rat contains wrong vals, rewrite each rat entry to its corresponding reg (rf contains correct vals before the commit point)
        - looks like those mispredicted instr never executed

Branches are not committed until they are resolved. This means instructions that come after the branch are also not committed until the branch is committed.

So branch misprediction recovery requires a flush of the ROB, RS, and RAT, then the correct instructions are fetched from the correct address. The ROB is flushed simply by moving the Issue pointer to the same entry as the Commit pointer. The RAT entries are set to point to the correct registers.
- making rat entries point to regs in rf (erase renaming)
- emptying out rob after committing offending branch instr
- empty rs and alus to prevent from broadcasting res, dropped

<img src="./png/Lesson7bNotes-image-10.png" alt="alt text" width="600" />

## ROB and Exceptions

Two problems with exceptions:

1. OOO instructions - When an exception occurs the ROB has not committed, so everything can be flushed and the exception handler can be loaded.
- in tomasulo, add writes long before div by 0 exception
- when div reaches commit point and it had been marked as exception, flush everything after including div and jump to exception handler
  - everything before committed, everything after not (same for load page fault); resume point for exception handler

2. Phantom Exceptions - Since nothing is committed after the branch, any misprediction results in a flush of the instructions, including any exceptions, and a load of the correct instructions.
- in tomasulo, predict branch not taken, execute div -> get exception before branch gets resolved
  - mark div exception in rob, when commit reaches branch, figure out misprediction and jump to label instead
    - haven't committed anything after the branch, so flush the div

<img src="./png/Lesson7bNotes-image-11.png" alt="alt text" width="650" />

Treat the exception the same as any other result -> delay the actual handling of the instruction until the commit.

Committed instructions cannot be "uncommitted".

- from ps'er pov
  - first did branch -> div -> add -> resolve branch -> undo div -> jump to label
- from pgrm'er perspective
  - never sees the div bc it's not committed, only the add
    - never sees execution of wrong path instr, on exception never sees instr that shouldn't have been executed

<img src="./png/Lesson7bNotes-image-12.png" alt="alt text" width="600" />

- instr in prgrm order, status and instr
  - committed: executed pipeline, res committed from rob to rf
  - executing: left rs, res hasn't been broadcasted on bus
  - done: got rs, executed, but result hasn't been committed
- div has exception
  - try to commit div, sees commits, flush everything after, jump to exception handler


## RAT Updates on Commit

- say instr have all finished, res in rob, time to commit
  - careful of the renames, point to latest instr in rob for the rat entry
- commit -> deposit res in reg, even if rat doesn't point to the rob entry for the reg
  - if not latest renamed val in rat, free rob entry
  - if yes, point the rat entry to the register

- Commit instruction
  - The result is put in the registers
  - Check the RAT. If the RAT entry is not the latest RAT update, nothing needs to be done to the RAT. If it is the latest update, change the value in the RAT to correspond to the correct register.

Registers are always up-to-date, which improves exception handling.
- make entire rat point to regs, content in rf exactly content up to the commit point in rob

The values are updated on commit only if renaming occurs (changing entries from rob to regs).

<img src="./png/Lesson7bNotes-image-13.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-14.png" alt="alt text" width="600" />

- rob example cycles 1 2
  - 1. find rs and rob entry, fill in rs vals from arf too, point rat entry to rob entry
    - can dispatch same cycle, beginning executing next yccle
  - 2. exe, at cycle 42 write res in rob, free rs
    - issue mul

<img src="./png/Lesson7bNotes-image-15.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-16.png" alt="alt text" width="600" />

- rob examples cycles 3 4
  - 3. free rs for mul, issue add
  - 4. free rs and execute the add (write in 5)
    - issue mul, fill r1 and r3 with rob 2 and 3, rename r1 with rob4
      - use vals of rat for operands before rename

<img src="./png/Lesson7bNotes-image-17.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-18.png" alt="alt text" width="600" />

- rob examples cycles 5 6
  - 5. issue sub
    - write instr 3
      - mark rob as done
      - fill in vals during broadcast
  - 6. issue add, mark r1 in rat to rob6
    - can't commit instr 3 until the prior instrs committed

<img src="./png/Lesson7bNotes-image-19.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-20.png" alt="alt text" width="600" />

- rob examples cycles 13 24
  - 13. rob2 write done, mark rob entry done, broadcast
    - rob4 can dispatch in cycle 13 now
  - 14. free rs and execute rob4 instr, write in cycle 24
  - 24. rob4 intr gets broadcasted and mark in rob as done
    - rob5 instr gets dispatched

- rob example cycles 25 43
  - 25. execute rob5 instr and free rs
  - 26. broadcast rob5 instr, mark rob entry done
  - 42. broadcast rob1 instr, mark rob entry done
    - rob6 instr can dispatch
  - 43. execute rob 6 instr and free rs
    - commit rob1 instr
      - update r2 in rf with res
      - look at rat entry for r2, see if it's pointing to the entry
        - yes, point rat entry to r2 in rf
      - clear rob1 entry

<img src="./png/Lesson7bNotes-image-21.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-22.png" alt="alt text" width="600" />

- rob example cycles 44 48
  - 44. broadcast rob6 instr res
    - commit rob2 instr
      - update r1 in rf
      - rat for r1 doesn't point to rob2
      - clear rob2 entry
  - 45. commit rob3 instr
    - update r3 in rf
    - point r3 rat to r3 rf
    - clear rob3 entry
  - 46. commit rob4 instr
    - update r1 in rf
    - clear rob4 entry
  - 47. commit rob5 instr
    - update r4 in rf
    - point r4 rat to r4 in rf
    - clear rob5 entry
  - 48. commit rob6
    - update r1 in rf
    - point r1 rat to r1 in rf
    - clear rob6 entry

<img src="./png/Lesson7bNotes-image-23.png" alt="alt text" width="600" />

- assume same assumptions (capture results and dispatch same cycle, last cycle of exe complete before result can be broadcasted)

<img src="./png/Lesson7bNotes-image-24.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-25.png" alt="alt text" width="600" />

- at beginning of cycle 2, rs for the div already freed (from dispatch)

- at beginning of cycle 4, rs for instr 3 is cleared when it executes

<img src="./png/Lesson7bNotes-image-26.png" alt="alt text" width="600" />

- which rs fields and rat entry for the instr 4 mul?
  - read rat entries for r1 and r2 before modifying rat for the rob4 -> r1 rat

<img src="./png/Lesson7bNotes-image-27.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-28.png" alt="alt text" width="600" />

- cycle 5 can we dispatch instr or wr? name should be rob entry
  - can't dispatch yet, rob 3 wr res
    - mark rob3 entry done, broadcast with rob3 tag
    - since rob5 sub filled by rob3, can dispatch this now in the same cycle
      - figure out wr res happens, enabling the dispatch

<img src="./png/Lesson7bNotes-image-29.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-30.png" alt="alt text" width="600" />

- cycle 6
  - issue the instr 6 add
  - rob2 wr, mark rob as done, and broadcast res
- cycle 7
  - rob wr, mark rob as done, and broadcast res
- what cycle do we dispatch rob4?
  - waiting for rob1 broadcasted in cycle 12
  - can capture and dispatch same cycle, execute in following cycle

<img src="./png/Lesson7bNotes-image-31.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-32.png" alt="alt text" width="600" />

- what else happens in cycle 13
  - rob 4 and 6 exe
  - rob 1 commit

<img src="./png/Lesson7bNotes-image-33.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-34.png" alt="alt text" width="600" />

- rob1 commits
  - deposit 4 to r2
  - check if rat r2 points to rob1, yes
    - free rat entry, point rat r2 to rf

<img src="./png/Lesson7bNotes-image-35.png" alt="alt text" width="600" />

 - rf for r1 updated to rob2 res
  - rob2 entry not pointed to by r1 in rat, don't update just clear rob entry

<img src="./png/Lesson7bNotes-image-36.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-37.png" alt="alt text" width="600" />

- cycle 15
  - commit rob 3
- when does last instr rob6 commit

<img src="./png/Lesson7bNotes-image-38.png" alt="alt text" width="600" />

<img src="./png/Lesson7bNotes-image-39.png" alt="alt text" width="600" />

## rob timing example

- track without going thru whole process
  - free rs on broadcast, not dispatch (i.e. for certainty if doing speculation)
  - issue, capture, dispatch in same cycle
- assume a lot of rob entries, 2 div/mul and 3 add/sub rs
  - e.g. rs for div busy between cycles 1 and 42

- instr 4 mul doesn't have rs until cycle 14
  - also have to wait for instr 2 and 3, broadcasted in cycles 13 and 5
  - dispatch same cycle, exe next

- instr 5 sub has to be issued in order

- make comments for deps and rs

<img src="./png/Lesson7bNotes-image-40.png" alt="alt text" width="600" />

- quiz

<img src="./png/Lesson7bNotes-image-41.png" alt="alt text" width="600" />

- both rs for div/mul freed in cycle 4
- can broadcast add + mul same cycle, commit too

<img src="./png/Lesson7bNotes-image-42.png" alt="alt text" width="600" />

- cycle 7
  - the instr 4 mul and instr 5 sub are diff execution units, can do same cycle (remember pipelining for same execution unit too)
    - otherwise if same unit would have to wait
- cycle 10
  - can commit sub too
- cycle 11
  - can commit the add

<img src="./png/Lesson7bNotes-image-43.png" alt="alt text" width="600" />


## Unified Reservation Station

- couldn't issue instr even if other rs was free
- since inorder issue, could block the other rs

Unified reservation stations: combine the reservation stations for the Adder and the Multiplier.
- All of the reservation stations are in one large array, so the reservation stations can be used when necessary
  - they do not have to wait for a specific set of RS.

The down side of the unified RS is the increased complexity in hardware
  - The hardware must be able to determine the correct arithmetic unit for each entry.
  - only needed to check if add or mul was available before
    - each cycle for dispatch have to go through and check for add and mul to pick

<img src="./png/Lesson7bNotes-image-44.png" alt="alt text" width="650" />

## Superscalar

- before only issuing 1 instr/cycle

A superscalar processor must be able to:

- Fetch more than 1 instruction per cycle
  - 8 bytes instead of 4 bytes for 1 -> 2 instr
- Decode more than 1 instruction per cycle
- Issue more than 1 instruction per cycle
  - still need inorder, consecutive
- Dispatch more than 1 instruction per cycle
- Broadcast more than 1 instruction per cycle
  - multiple buses, rs compares with multiple buses (proportional cost)
- Commit more than 1 instruction per cycle

Weakest link: the processor is limited by the slowest task. 
- If all of the tasks, except Decode, can perform 4 instructions per cycle and Decode can only perform 2 instructions per cycle, then the processor is limited to 2 instructions per cycle.

## Terminology Confusion

Most research papers will use:

- Issue (allocate, dispatch): obtain rs and rob entry
- Dispatch (execute, issue, dispatch): instr selected for exe
- Commit (complete, retire, graduate): committed

Processor Designers will often give these different names

## Out of Order

What is really out of order?

Fetch, Decode, Issue are all in order - this ensures dependencies are done in order
- instr in rs waiting for execution now

Execute and Write can be done out of order
- executing in order of data deps

Commit is done in-order

<img src="./png/Lesson7bNotes-image-45.png" alt="alt text" width="650" />

- in an ooo ps'er
  - bc we commit inorder, rest of pipeline (release rob entry) also in order
  

<img src="./png/Lesson7bNotes-image-46.png" alt="alt text" width="600" />