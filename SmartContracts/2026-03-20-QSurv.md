# Proposal to include QSurv Smart Contract – Decentralized AI-Verified Survey Platform

## Proposal
Deploy the QSurv smart contract to Qubic mainnet as part of the TOP3 Hack Nostromo hackathon prize distribution. QSurv is a trustless survey platform that combines AI-powered answer verification with blockchain-based escrow payments, enabling creators to incentivize authentic responses while eliminating fake participation through real-time validation.

### Available Options
- **Option 0:** no, don't allow
- **Option 1:** yes, allow

---

## What is QSurv
QSurv addresses a critical problem in online surveys: **fake and low-quality responses**. Traditional survey platforms have no way to verify answer authenticity, resulting in wasted funds and unreliable data.

QSurv solves this by combining smart contract escrow, AI verification, blockchain transparency, and decentralized storage into a single trustless platform — enabling survey creators to deposit funds that are automatically distributed only upon AI-verified valid responses.

**Contract Index:** 26

---

## QSurv Key Features

| **Feature** | **Description** |
|-------------|-----------------|
| **Trustless Payments** | Escrow-based fund distribution — no middleman required |
| **AI Validation** | Google Gemini 2.0 Flash verifies responses in real-time, eliminating fake answers |
| **Double-Payout Prevention** | On-chain tracking of paid respondents prevents duplicate payouts |
| **Slot Reuse** | Inactive survey slots are recycled, maximizing contract capacity |
| **Abort & Refund** | Survey creators can abort active surveys and reclaim remaining funds |
| **Burn Mechanism** | Half of platform fees are burned to sustain execution fee reserve |
| **Staking Tiers** | Tiered bonus system (0–30%) rewards committed participants |
| **IPFS Storage** | Decentralized, permanent survey data storage via Pinata |
| **Instant Finality** | Leverages Qubic's fast transaction processing |
| **Refund on Failure** | Invocation rewards are refunded on validation failures |

---

## Reward Distribution Model

```
Total Reward per Response = rewardPerRespondent
├── Base Reward:      40% → Respondent
├── Referral:         20% → Referrer (or oracle if none)
├── Staking Bonus:    Tier-dependent → Respondent
├── Platform Fee:     Tier-dependent → Split: 50% burned / 50% oracle
│
│   Tier 0 (default): bonus  0%, platform  1%
│   Tier 1:           bonus 10%, platform  3%
│   Tier 2:           bonus 20%, platform  5%
│   Tier 3:           bonus 30%, platform 10%
│
└── Leftover on survey completion → Refunded to creator
```

---

## Security Features

| **Feature** | **Description** |
|-------------|-----------------|
| **Oracle-Controlled Payouts** | Only the verified oracle can trigger fund distribution |
| **Genesis Oracle** | Oracle address is hardcoded at INITIALIZE to prevent sniper attacks |
| **Double-Payout Prevention** | `paidRespondents` array tracks who has been paid per survey |
| **Escrow Protection** | Funds locked until AI verification passes |
| **Input Validation** | All inputs validated before state changes; invocation rewards refunded on failure |
| **Balance Checks** | Prevents overdraft and ensures fund availability |
| **Auto-Deactivation** | Surveys automatically close when respondent limit is reached |
| **Creator Abort** | Survey creators can abort and reclaim remaining balance at any time |
| **Burn Fee** | Half of platform fee is burned to sustain the contract's execution fee reserve |

---

## Technical Implementation

### Technology Stack

| **Component** | **Technology** |
|---------------|----------------|
| Frontend | Next.js 15, Tailwind CSS, Framer Motion |
| Smart Contracts | C++ (Qubic QPI) |
| Blockchain | Qubic Network |
| AI Engine | Google Gemini 2.0 Flash |
| Database | SQLite via Prisma ORM |
| Storage | IPFS (Pinata) |
| Wallet | MetaMask Snap integration |

### Contract Interface

**Functions (read-only):**

| **ID** | **Function** | **Description** |
|--------|-------------|-----------------|
| 1 | `getSurvey` | Query survey details by ID |
| 2 | `getSurveyCount` | Get total number of surveys |

**Procedures (state-changing):**

| **ID** | **Procedure** | **Description** |
|--------|--------------|-----------------|
| 1 | `createSurvey` | Create new survey with escrow funds |
| 2 | `payout` | Distribute rewards after AI verification (oracle only) |
| 3 | `setOracle` | Set or update oracle address |
| 4 | `abortSurvey` | Creator aborts survey, remaining balance refunded |

### Source Code

See the PR: https://github.com/qubic/core/pull/771

```c++
using namespace QPI;

// ============================================
// QSurv - Trustless Survey Platform
// Decentralized survey creation with escrow and AI-verified payouts
// ============================================

// Constants
constexpr uint32 QSURV_MAX_SURVEYS = 1024;
constexpr uint32 QSURV_MAX_RESPONDENTS_PER_SURVEY = 128;
constexpr uint32 QSURV_IPFS_HASH_SIZE = 64;

// Forward declaration for state expansion
struct QSURV2
{
};

struct QSURV : public ContractBase
{

    // ============================================
    // STRUCTS
    // ============================================
    struct Survey
    {
        uint64 surveyId;
        id creator;
        uint64 rewardAmount;
        uint64 rewardPerRespondent;
        uint32 maxRespondents;
        uint32 currentRespondents;
        uint64 balance;
        Array<uint8, QSURV_IPFS_HASH_SIZE> ipfsHash;
        Array<id, QSURV_MAX_RESPONDENTS_PER_SURVEY> paidRespondents; // track who has been paid
        bit isActive;
    };

    // ============================================
    // STATE (Persistent Storage)
    // ============================================
  public:
    struct StateData
    {
        Array<Survey, QSURV_MAX_SURVEYS> _surveys;
        uint32 _surveyCount;
        id _oracleAddress;
    };

    // ============================================
    // SYSTEM PROCEDURES
    // ============================================
  public:
    INITIALIZE()
    {
        // Establish the Genesis Oracle to prevent sniper attacks
        // ID: BOWFPORUCOPUOCNOIBDNSSRHQYCAXCRHGKBUKCMZJCZBHQVUUWWLAGIFWGVN
        state.mut()._oracleAddress = ID(_B, _O, _W, _F, _P, _O, _R, _U, _C, _O, _P, _U, _O, _C, _N, _O, _I, _B, _D, _N,
                                        _S, _S, _R, _H, _Q, _Y, _C, _A, _X, _C, _R, _H, _G, _K, _B, _U, _K, _C, _M, _Z,
                                        _J, _C, _Z, _B, _H, _Q, _V, _U, _U, _W, _W, _L, _A, _G, _I, _F);
    }

    BEGIN_EPOCH()
    {
        // Called at the beginning of each epoch
    }

    END_EPOCH()
    {
        // Called at the end of each epoch
    }

    BEGIN_TICK()
    {
        // Called before processing transactions in a tick
    }

    END_TICK()
    {
        // Called after processing transactions in a tick
    }

    // ============================================
    // INPUT/OUTPUT STRUCTS
    // ============================================

    // --- CreateSurvey ---
    struct createSurvey_input
    {
        uint64 rewardPool;
        uint32 maxRespondents;
        Array<uint8, 64> ipfsHash;
    };

    struct createSurvey_output
    {
        uint64 surveyId;
        bit success;
    };

    struct createSurvey_locals
    {
        uint32 i;
        uint32 index;
        Survey tempSurvey;
    };

    // --- Payout ---
    struct payout_input
    {
        uint64 surveyId;
        id respondentAddress;
        id referrerAddress;
        uint8 respondentTier;
    };

    struct payout_output
    {
        uint64 amountPaid;
        uint64 bonusPaid;
        uint64 referralPaid;
        bit success;
    };

    struct payout_locals
    {
        uint32 index;
        bit found;
        bit isDuplicate;
        uint64 totalReward;
        uint64 baseReward;
        uint64 referralReward;
        uint64 platformFee;
        uint64 burnFee;
        uint64 oracleFee;
        uint64 bonus;
        uint64 totalSpent;
        uint32 i;
        Survey tempSurvey;
    };

    // --- GetSurvey (Read-only) ---
    struct getSurvey_input
    {
        uint64 surveyId;
    };

    struct getSurvey_output
    {
        uint64 surveyId;
        id creator;
        uint64 rewardAmount;
        uint64 rewardPerRespondent;
        uint32 maxRespondents;
        uint32 currentRespondents;
        uint64 balance;
        bit isActive;
        bit found;
    };

    struct getSurvey_locals
    {
        uint32 i;
    };

    // --- GetSurveyCount (Read-only) ---
    struct getSurveyCount_input
    {
    };

    struct getSurveyCount_output
    {
        uint32 count;
    };

    // --- AbortSurvey (Creator only) ---
    struct abortSurvey_input
    {
        uint64 surveyId;
    };

    struct abortSurvey_output
    {
        bit success;
    };

    struct abortSurvey_locals
    {
        uint32 i;
        uint32 index;
        bit found;
        Survey tempSurvey;
    };

    // --- SetOracle (Admin) ---
    struct setOracle_input
    {
        id newOracleAddress;
    };

    struct setOracle_output
    {
        bit success;
    };

    // ============================================
    // USER PROCEDURES (State-Modifying)
    // ============================================

    PUBLIC_PROCEDURE_WITH_LOCALS(createSurvey)
    {
        // Validation checks - refund invocation reward on failure
        if (input.maxRespondents == 0 || input.maxRespondents > QSURV_MAX_RESPONDENTS_PER_SURVEY)
        {
            qpi.transfer(qpi.invocator(), qpi.invocationReward());
            return;
        }
        if (input.rewardPool == 0)
        {
            qpi.transfer(qpi.invocator(), qpi.invocationReward());
            return;
        }

        // Verify invocation reward matches rewardPool
        if ((uint64)qpi.invocationReward() < input.rewardPool)
        {
            qpi.transfer(qpi.invocator(), qpi.invocationReward());
            return;
        }

        // Scan for an empty (inactive) slot to reuse
        locals.i = QSURV_MAX_SURVEYS; // sentinel: no slot found yet
        for (locals.index = 0; locals.index < state.get()._surveyCount; locals.index++)
        {
            if (!state.get()._surveys.get(locals.index).isActive)
            {
                locals.i = locals.index;
                break;
            }
        }

        // If no inactive slot found, try to allocate a new one
        if (locals.i == QSURV_MAX_SURVEYS)
        {
            if (state.get()._surveyCount >= QSURV_MAX_SURVEYS)
            {
                qpi.transfer(qpi.invocator(), qpi.invocationReward());
                return; // All slots occupied and active, truly full
            }
            locals.i = state.get()._surveyCount;
            state.mut()._surveyCount++;
        }

        // Create new survey in the found/allocated slot
        locals.tempSurvey.surveyId = locals.i + 1;
        locals.tempSurvey.creator = qpi.invocator();
        locals.tempSurvey.rewardAmount = input.rewardPool;
        locals.tempSurvey.maxRespondents = input.maxRespondents;
        locals.tempSurvey.rewardPerRespondent = QPI::div(input.rewardPool, (uint64)input.maxRespondents);
        locals.tempSurvey.balance = input.rewardPool;
        locals.tempSurvey.currentRespondents = 0;
        locals.tempSurvey.isActive = 1;

        // Zero out paid respondents list for safety (especially on slot reuse)
        for (locals.index = 0; locals.index < QSURV_MAX_RESPONDENTS_PER_SURVEY; locals.index++)
        {
            locals.tempSurvey.paidRespondents.set(locals.index, NULL_ID);
        }

        // Copy IPFS hash using Array's set method
        for (locals.index = 0; locals.index < QSURV_IPFS_HASH_SIZE; locals.index++)
        {
            locals.tempSurvey.ipfsHash.set(locals.index, input.ipfsHash.get(locals.index));
        }

        // Commit state changes
        state.mut()._surveys.set(locals.i, locals.tempSurvey);

        output.surveyId = locals.i + 1;
        output.success = 1;
    }

    PUBLIC_PROCEDURE_WITH_LOCALS(payout)
    {
        // Security: Oracle-only execution
        if (qpi.invocator() != state.get()._oracleAddress)
        {
            return;
        }

        // Find survey by ID using found flag pattern
        for (locals.i = 0; locals.i < state.get()._surveyCount; locals.i++)
        {
            if (state.get()._surveys.get(locals.i).surveyId == input.surveyId)
            {
                locals.index = locals.i;
                locals.found = 1;
                break;
            }
        }

        if (!locals.found)
        {
            return;
        }

        // Copy state to local variable for reading and modification
        locals.tempSurvey = state.get()._surveys.get(locals.index);

        // Validation checks
        if (!locals.tempSurvey.isActive)
        {
            return;
        }
        if (locals.tempSurvey.currentRespondents >= locals.tempSurvey.maxRespondents)
        {
            return;
        }
        if (locals.tempSurvey.balance < locals.tempSurvey.rewardPerRespondent)
        {
            return;
        }

        // Double-payout prevention: check if respondent was already paid
        for (locals.i = 0; locals.i < locals.tempSurvey.currentRespondents; locals.i++)
        {
            if (locals.tempSurvey.paidRespondents.get(locals.i) == input.respondentAddress)
            {
                locals.isDuplicate = 1;
                break;
            }
        }
        if (locals.isDuplicate)
        {
            return;
        }

        // Calculate reward splits using QPI::div (no / operator allowed)
        locals.totalReward = locals.tempSurvey.rewardPerRespondent;

        // Minimum guaranteed 40%, fixed referral 20%
        locals.baseReward = QPI::div(locals.totalReward * 40, 100ULL);
        locals.referralReward = QPI::div(locals.totalReward * 20, 100ULL);

        // Staking bonus tier system and fair oracle incentivization
        if (input.respondentTier == 1)
        {
            locals.bonus = QPI::div(locals.totalReward * 10, 100ULL);      // 10%
            locals.platformFee = QPI::div(locals.totalReward * 3, 100ULL); // 3%
        }
        else if (input.respondentTier == 2)
        {
            locals.bonus = QPI::div(locals.totalReward * 20, 100ULL);      // 20%
            locals.platformFee = QPI::div(locals.totalReward * 5, 100ULL); // 5%
        }
        else if (input.respondentTier == 3)
        {
            locals.bonus = QPI::div(locals.totalReward * 30, 100ULL);       // 30%
            locals.platformFee = QPI::div(locals.totalReward * 10, 100ULL); // 10%
        }
        else // Tier 0
        {
            locals.bonus = 0;
            locals.platformFee = QPI::div(locals.totalReward * 1, 100ULL); // 1%
        }

        locals.totalSpent = locals.baseReward + locals.bonus + locals.referralReward + locals.platformFee;

        // Split platform fee: half burned to sustain execution fee reserve,
        // half goes to oracle as operational compensation
        locals.burnFee = QPI::div(locals.platformFee, 2ULL);
        locals.oracleFee = locals.platformFee - locals.burnFee;

        // Execute fund transfers
        qpi.transfer(input.respondentAddress, locals.baseReward + locals.bonus);

        if (input.referrerAddress != NULL_ID)
        {
            qpi.transfer(input.referrerAddress, locals.referralReward);
        }
        else
        {
            qpi.transfer(state.get()._oracleAddress, locals.referralReward);
        }

        qpi.transfer(state.get()._oracleAddress, locals.oracleFee);
        qpi.burn(locals.burnFee); // Replenish execution fee reserve

        // Record this respondent to prevent double-payout
        locals.tempSurvey.paidRespondents.set(locals.tempSurvey.currentRespondents, input.respondentAddress);

        // Update state in local variable. Only deduct what was ACTUALLY spent!
        locals.tempSurvey.balance = locals.tempSurvey.balance - locals.totalSpent;
        locals.tempSurvey.currentRespondents++;

        if (locals.tempSurvey.currentRespondents >= locals.tempSurvey.maxRespondents)
        {
            locals.tempSurvey.isActive = 0;
            // Refund leftover balance to creator
            if (locals.tempSurvey.balance > 0)
            {
                qpi.transfer(locals.tempSurvey.creator, locals.tempSurvey.balance);
                locals.tempSurvey.balance = 0;
            }
        }

        // Commit modifications back to state
        state.mut()._surveys.set(locals.index, locals.tempSurvey);

        output.success = 1;
        output.amountPaid = locals.baseReward;
        output.bonusPaid = locals.bonus;
        output.referralPaid = locals.referralReward;
    }

    PUBLIC_PROCEDURE_WITH_LOCALS(abortSurvey)
    {
        for (locals.i = 0; locals.i < state.get()._surveyCount; locals.i++)
        {
            if (state.get()._surveys.get(locals.i).surveyId == input.surveyId)
            {
                locals.index = locals.i;
                locals.found = 1;
                break;
            }
        }

        if (!locals.found)
        {
            return;
        }

        locals.tempSurvey = state.get()._surveys.get(locals.index);

        // Only creator can abort
        if (qpi.invocator() != locals.tempSurvey.creator)
        {
            return;
        }

        // Must be active
        if (!locals.tempSurvey.isActive)
        {
            return;
        }

        // Refund remaining balance to creator
        if (locals.tempSurvey.balance > 0)
        {
            qpi.transfer(locals.tempSurvey.creator, locals.tempSurvey.balance);
        }

        // Zero out completely to reclaim slot safely
        locals.tempSurvey.isActive = 0;
        locals.tempSurvey.balance = 0;
        locals.tempSurvey.currentRespondents = locals.tempSurvey.maxRespondents;

        state.mut()._surveys.set(locals.index, locals.tempSurvey);
        output.success = 1;
    }

    PUBLIC_PROCEDURE(setOracle)
    {
        // Only allow setting oracle if not already set, or by current oracle
        if (state.get()._oracleAddress == NULL_ID || qpi.invocator() == state.get()._oracleAddress)
        {
            state.mut()._oracleAddress = input.newOracleAddress;
            output.success = 1;
        }
    }

    // ============================================
    // USER FUNCTIONS (Read-Only)
    // ============================================

    PUBLIC_FUNCTION_WITH_LOCALS(getSurvey)
    {
        for (locals.i = 0; locals.i < state.get()._surveyCount; locals.i++)
        {
            if (state.get()._surveys.get(locals.i).surveyId == input.surveyId)
            {
                output.surveyId = state.get()._surveys.get(locals.i).surveyId;
                output.creator = state.get()._surveys.get(locals.i).creator;
                output.rewardAmount = state.get()._surveys.get(locals.i).rewardAmount;
                output.rewardPerRespondent = state.get()._surveys.get(locals.i).rewardPerRespondent;
                output.maxRespondents = state.get()._surveys.get(locals.i).maxRespondents;
                output.currentRespondents = state.get()._surveys.get(locals.i).currentRespondents;
                output.balance = state.get()._surveys.get(locals.i).balance;
                output.isActive = state.get()._surveys.get(locals.i).isActive;
                output.found = 1;
                return;
            }
        }
    }

    PUBLIC_FUNCTION(getSurveyCount) { output.count = state.get()._surveyCount; }

    // ============================================
    // REGISTER USER FUNCTIONS AND PROCEDURES
    // ============================================
    REGISTER_USER_FUNCTIONS_AND_PROCEDURES()
    {
        // Functions (Read-only queries)
        REGISTER_USER_FUNCTION(getSurvey, 1);
        REGISTER_USER_FUNCTION(getSurveyCount, 2);

        // Procedures (State-modifying)
        REGISTER_USER_PROCEDURE(createSurvey, 1);
        REGISTER_USER_PROCEDURE(payout, 2);
        REGISTER_USER_PROCEDURE(setOracle, 3);
        REGISTER_USER_PROCEDURE(abortSurvey, 4);
    }
};
```

---

## Links

- **GitHub Repository**: https://github.com/IanLaFlair/QSurv
- **Smart Contract Source**: https://github.com/IanLaFlair/QSurv/blob/main/contracts/QSurv.h
- **Core PR**: https://github.com/qubic/core/pull/771

---

## Hackathon Context

This proposal is submitted as part of the **TOP3 Hack Nostromo** hackathon prize distribution process. QSurv was developed to demonstrate practical utility of Qubic's smart contract capabilities for real-world applications in the survey and market research industry.

---

*Submitted by: IanLaFlair*
*Date: March 20, 2026*
*Hackathon: TOP3 Hack Nostromo*
*Contract Index: 26*
