# List of Equations

As a quick reference, this is a list of the equations used in the course. If you are just starting this course, don't worry, each of these equations is explained in detail in the appropriate lesson, so you don't have to learn them now - this list is intended as a quick reference when working on later lessons that use the equations, or when studying for exams.

## Active Power

    P = 1/2 * C * V^2 * f * a

## Yield

    Yield = Working Chips / Chips on Wafer

## Fabrication Cost

    Fabrication Cost = Total Cost / Number of Wafers

## Speedup

    Speedup of X over Y = ExeTime(Y) / ExeTime(X)

    Speedup of X over Y = Throughput(X) / Throughput(Y)

    Speedup of X over Y = (Speedup of X over Z) / (Speedup of Y over Z)

## CPU Time

    CPU Time = <# of Instructions in the program> * <cycles per instruction> * <clock cycle time>

## Amdahl's Law

    Overall Speedup = 1 / ((1 - <Fraction Enhanced>) + <Fraction Enhanced> / <Speedup of Enhancement>)

## Execute Time

    Execute Time = <# Instructions> * <CPI> * <Clock Cycle Time>

## CPI With Branch Prediction

    Overall CPI = <Ideal CPI> + (<# of Mispredictions> / <# of Instructions>) * <Misprediction Penalty in Cycles>

## Availability

    Availability = MTTF / (MTTF + MTTR)

## AMAT

    AMAT = Hit Time + <Miss Rate> * <Miss Penalty>

    AMAT = (1 - Miss Rate) * <Hit Time> + <Miss Rate> * <Miss Time>

    (Miss Time = Hit Time + Miss Penalty)

## Miss Penalty with Multi-Level Caches

    L1 Miss Penalty = L2 Hit Time + L2 Miss Rate * L2 Miss Penalty

## Global Hit Rate

    Global Hit Rate = 1 - Global Miss Rate

## Global Miss Rate

    Global Miss Rate = # of Misses in this cache / # of all memory accesses

## Local Hit Rate

    Local Hit Rate = # of Hits / # of Accesses to this cache

## Page Table Size

    Page Table Size = (Virtual Memory / Page Size) * Size of Entry

## RAID0 1 Disk MTTF

    RAID0, 1 Disk MTTF = 1 / failure rate for single disk

## RAID0 N Disk MTTF

    RAID0, N disks MTTF = MTTF of one disk / N

## RAID1 N Disk MTTDL

    RAID1, N Disk MTTDL(N) = MTTF^1 / N

    where N is the number of disks

## RAID4 Read Throughput

    RAID4 Read Throughput = (N - 1) * <Throughput of Individual Disk>

## RAID4 Write Throughput

    RAID4 Write Throughput = 1/2 * <Throughput of Individual Disk>

## RAID4 MTTF

    RAID4 MTTF = (MTTF of 1 disk * MTTF of 1 disk) / (N * (N - 1) * (MTTR of 1 disk))

## RAID4 Data Capacity

    RAID4 Data Capacity = (N - 1) * <Data Capacity of Individual Disk>

## RAID5 Read Throughput

    RAID5 Read Throughput = N * <Throughput of Individual Disk>

## RAID5 Write Throughput

    RAID5 Write Throughput = N/4 * <Throughput of Individual Disk>

## RAID5 Data Capacity

    RAID5 Data Capacity (same as RAID 4) = (N - 1) * <Data Capacity of Individual Disk>
