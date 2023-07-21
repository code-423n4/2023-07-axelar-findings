# Axelar Analysis Report

## Approach taken in evaluating the codebase

- Started with the docs
- Then the codes in scope line by line
- Took a brief look at other contracts not in scope
- Back to the docs to counter some notes I made while going through codebase
- Discussed a few ideas with sponsors to understand their design choice and why I think it's risky or how I can recommend a better method to implement the strategies
- Finally went back to the codebase to sum up my reports

## Anything that you learned from evaluating the codebase

A couple things, to list a few:

- Create3
- The concept of interchain calls
- Understood why _addreses_ are treated as strings in the context to allow non-EVM chains support in the future and that is why they must also be converted to lower strings to avoid inequality issues.

### Refresher on previously acquainted things

Got more understanding of formerly acquainted concepts like:

- Create
- Create2
- Governance
- A few basic solidity concepts

## Any comments for the judge to contextualize your findings

I always approach contests as a learning opportunity and then submit issues I believe should be looked on, I spent around ~ 20 hours spread around 4 days on the contest, so I believe I had understanding of the protocol to a level to submit some findings, I would be waiting for the comments on my findings to find out what I can improve in my reports or why my thought process is wrong.


### Time spent:
20 hours