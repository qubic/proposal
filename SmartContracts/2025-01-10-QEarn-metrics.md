# Proposal to add state variables for additional metrics in QEarn Smart Contract

## Proposal
Allow to add following variables for tracking additional metrics directly in the QEarn Smart Contract to ensure full transparency on epoche level:

>burnedAmount
>boostedAmount
>rewardedAmount

This way we ensure that the full set of metrics is directly calculated and stored in the SC and can be used and displayed wherever needed.

## SC state change
This change requires a state change of the QEarn SC.
Additional actions for Computors will be posted in the #Computor-Operators channel in discord.

## Available Options
> Option 0: no, dont allow

> Option 1: yes, allow

## Code change

```c++
struct StatsInfo {

        uint64 burnedAmount;
        uint64 boostedAmount;
        uint64 rewardedAmount;

    };

    array<StatsInfo, QEARN_MAX_EPOCHS> statsInfo;
```


