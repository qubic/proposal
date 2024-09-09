# General Computor Proposal - How to participate

## Using the Qubic CLI
You can use the Qubic CLI to create proposals or cast your votes.

### Compile Qubic CLI
Please refer to the Qubic CLI GIT repo: https://github.com/qubic/qubic-cli

### Create a proposal
Create a static web content (it should not be able to alter it after you sent the proposal.)
You can use the Qubic GIT repo if you whish, you can send us a pull request to this repo with your proposal.

If you use Github, please use the `commmit` url. (Explained here: https://docs.github.com/en/repositories/working-with-files/using-files/getting-permanent-links-to-files)

#### Create Command

the command to be use: `-gqmpropsetproposal`

The format of the payload is: `"[ProposalTypeClass]|[NumberOfVoteOptions]:[AdditionalProposalData]|[URL]"`

**ProposalTypeClass**
```
[ProposalTypeClass]: Should be one of the following values:
        - GeneralOptions: Normal proposal without automated execution.
                NumberOfVoteOptions is 2, 3, ..., 7, or 8. AdditionalProposalData is unused/empty.
        - Transfer: Propose to transfer/donate an amount to a destination address.
                AdditionalProposalData is comma-separated list of destination identity, amount of option 1,
                amount of option 2, ... (options 0 is no change to current state. In GQMPROP contract, the
                amount is relative in millionth, for example 150000 = 15% and 1000 = 0.1%.
        - Variable: Propose to set variable in contract state to specific value. Not supported yet.
```

**Example to create a computor proposal**
`./qubic-cli -seed <YOURCOMPUTORSEED>  -nodeip <ANYNODEIP> -gqmpropsetproposal "GeneralOptions|2|https://github.com/qubic/proposal/blob/2e25df383dc95b937a3e0c56d11925b664770108/SmartContracts/2024-09-09-CCF.md"`

this proposal has two options and refeerence to a github commit page.

### Check the active proposals
the command to be use: `-gqmpropgetproposals`
Example: 
```
#> ./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -gqmpropgetproposals active
Received list of 1 proposals
Proposal:
        proposalIndex = 0
        proposer = EQOTMBNINEBMWFTBFTHTMWWLKHNBBAUHJNHUEQKDOCRTCLQBTDCMFOFHNNPJ
        url = https://github.com/qubic/proposal/blob/2e25df383dc95b937a3e0c56d11925b664770108/SmartContracts/2024-09-09-CCF.md
        type = GeneralOptions with 2 options (vote value 0 ... 1)
        epoch = 125
        tick = 15796800
```

> note the `proposalIndex` (0) this will be used later as reference.

### Cast a vote
the command to be use: `-gqmpropvote`
If you are on of the 676 Computors you can cast your vote.
For this, you need the index of the proposal and the index of the option you want to vote for.

Example: 
```
./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -gqmpropvote 0 0
                                                              ^ first number is the proposal index
                                                                ^ second number is the option to vote for 0,1,2,3,4,5,6,7
```

### Get result of voting
the command to be use: `-gqmpropgetresults`
```
#> ./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -gqmpropgetresults 0
Proposal:
        proposalIndex = 0
        proposer = EQOTMBNINEBMWFTBFTHTMWWLKHNBBAUHJNHUEQKDOCRTCLQBTDCMFOFHNNPJ
        url = https://github.com/qubic/proposal/blob/2e25df383dc95b937a3e0c56d11925b664770108/SmartContracts/2024-09-09-CCF.md
        type = GeneralOptions with 2 options (vote value 0 ... 1)
        epoch = 125
        tick = 15796800
Proposal voting results:
        proposal index = 0
        proposal tick = 15796800
        total votes = 2 / 676
        votes for option 0 = 2 (no change)
        votes for option 1 = 0
        most voted option = 0 (total votes not sufficient for acceptance of proposal)
```

for general acceptance rules, please refer to: https://docs.qubic.org/learn/proposals#the-voting-basics--rules
             



