# Computor Controled Found (CCF) - How to participate

## Using the Qubic CLI
You can use the Qubic CLI to create proposals or cast your votes.

### Compile Qubic CLI
Please refer to the Qubic CLI GIT repo: https://github.com/qubic/qubic-cli

### Create a proposal
Create a static web content (it should not be able to alter it after you sent the proposal.)
You can use the Qubic GIT repo if you whish, you can send us a pull request to this repo with your proposal. Put your proposal in the folder `CCF-Funding-Request` and name it `yyyy-mm-dd-name-of-your-request.md`.

If you use Github, please use the `commmit` url. (Explained here: https://docs.github.com/en/repositories/working-with-files/using-files/getting-permanent-links-to-files)

A CCF Proposal can be made by any valid Qubic Address.

> [!Note]
> The cost to create a CCF Proposal is 1mio (1'000'000) Qubic.

#### Create Command

the command to be use: `-ccfsetproposal`

The format of the payload is: `"Transfer|[NumberOfVoteOptions]:[TargetAddress],[Amount1...7]|[URL]"`

**Example to create a computor proposal**
`./qubic-cli -seed <YOURSEED>  -nodeip <ANYNODEIP> -ccfsetproposal "Transfer|2:EQOTMBNINEBMWFTBFTHTMWWLKHNBBAUHJNHUEQKDOCRTCLQBTDCMFOFHNNPJ,8000|https://github.com/qubic/proposal/blob/main/CCF-Funding-Requests/2024-10-13-qvault-smart-contract.md"`

this proposal has two options and refeerence to a github commit page.
Option 0: No
Option 1: 8000 Qubic

### Check the active proposals
the command to be use: `-ccfgetproposals`
Example: 
```
#> ./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -ccfgetproposals active
Received list of 1 proposals
Proposal:
        proposalIndex = 0
        proposer = VGIWRRNVVRRXSEASPENCIMVNANPCFHAASZVPBIEFLCRYHWSZYSGXHNSBYPVN
        url = https://github.com/qubic/proposal/blob/290f9e7d9f6a39d0245d108101e1e062638e5c1f/CCF-Funding-Requests/2024-10-13-qvault-smart-contract.md
        type = Transfer with 2 options
        destination address = AVXAJKOXPJJYPGKJSRAOSSCOXSNAYLQZPKKUFBDZFFHCYZPBROWKFGRCQCUL
                vote value 0 = no transfer
                vote value 1 = 5500000000
        epoch = 131
        tick = 16645515
```

> note the `proposalIndex` (0) this will be used later as reference.

### Cast a vote
the command to be use: `-ccfvote`

> [!NODE]
> Only the 676 Computors from the current epoch are allowed to vote

For this, you need the index of the proposal and the index of the option you want to vote for.

Example: 
```
./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -ccfvote 0 0
                                                          ^ first number is the proposal index
                                                            ^ second number is the option to vote for 0,1,2,3,4,5,6,7
```

### Get result of voting
the command to be use: `-ccfgetresults`
```
#> ./qubic-cli -seed <YOURSEED> -nodeip <ANYNODEIP> -ccfgetresults 0
Querying data of proposal 0 ...
Proposal:
        proposalIndex = 0
        proposer = VGIWRRNVVRRXSEASPENCIMVNANPCFHAASZVPBIEFLCRYHWSZYSGXHNSBYPVN
        url = https://github.com/qubic/proposal/blob/290f9e7d9f6a39d0245d108101e1e062638e5c1f/CCF-Funding-Requests/2024-10-13-qvault-smart-contract.md
        type = Transfer with 2 options
        destination address = AVXAJKOXPJJYPGKJSRAOSSCOXSNAYLQZPKKUFBDZFFHCYZPBROWKFGRCQCUL
                vote value 0 = no transfer
                vote value 1 = 5500000000
        epoch = 131
        tick = 16645515
Proposal voting results:
        proposal index = 0
        proposal tick = 16645515
        total votes = 1 / 676
        votes for option 0 = 0
        votes for option 1 = 1
        most voted option = 1 (total votes not sufficient for acceptance of proposal)
```

for general acceptance rules, please refer to: https://docs.qubic.org/learn/proposals#the-voting-basics--rules
             



