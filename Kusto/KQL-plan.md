# Working plan for adding support for KQL (Kusto)

## Overview

- KQL (Kusto) in front of clickhouse; Guardium Insights Application
- Application uses KQL as front-end (instead of SQL)
- KQL=>converter (PoC – rudimentary translation) =>SQL=>ClickHouse – converter is sub-optimal.
- QRadar team took SQL and ran it thru the db2 optimizer which produced much better performance.
- Proposal is to put the optimizer in ClickHouse – longer term feature
- Further proposal is for ClickHouse to support KQL natively.

KQL Documentation:

https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/

According to the [Kusto Query Language Classification](https://github.com/microsoft/Kusto-Query-Language/blob/master/doc/LanguageMap.md), 
The language is classified to three flavors:
* **Core:** The minimal language syntax that supports the majority of the data query scenarios.
* **Full:** The full language syntax.
* **Out of scope:** language construct that are not considered to be relevant to language implementors, either because they are legacy or because their semantic is not formal.


We only consider about the core and full parts. The part of **Out of scope** will be ignored.Here are the functionalities of core and full.

- [Core Kusto Query Language](KQL-core.md)
- [Full Kusto Query Language](KQL-full.md)



```
a Minimum Viable Product (MVP) is when you do the least possible work in order to test your idea/hypothesis/assumptions while measuring a specific set of "metrics" in order to learn what people actually want you to solve/build.

Building a Minimum Viable Product (MVP) is a process for avoiding the development of products that customers do not want. The idea is to rapidly build a minimum set of features that is enough to deploy the product and test key assumptions about customers’ interactions with the product
``` 


## The project will work in several phases.  
- Phase 1:  
    - Building a Minimum Viable Product (MVP):
    Implement the minimum features need to be support


under the branch KQL-phase1, this is to be used as main branch for phase 1

developers need to created a new sub branch from this branch , e.g : KQL-phase1_Yong.
so the code change, test scripts, docs need to be workd on the sub branch, the merge to the branch KQL-phase1  



- Phase 2:
    - Implement the other features 

- Phase 3:
    - Implement the rest features in part of Core

- Phase 4:
    - Implement the rest features in part of Full


    
Because of the complexity of KQL language, in each phase , the task need to be split to sub-tasks (PRs)


## Work process
In each phase, we need to do the following works:

- Step 1. Create Analysis and design documentation
    - Requirements  
        Clarify which KQL features supported in current phase
    - Design  
        Describe how the KQL features implemented. explore the pipeline of query parse, plan, and execution in ClickHouse
    - Test Plan  
         Include test cases, type of tests and how to test be performed
    - Document  
        The document on how the KQL will be used to end user
- Step 2. Implementation and Test  
    - Make changes on existing components or add new components  
    - Create test script to run KQL  
    - Run tests described in the test plan.  
- Step 3. Code review  
    - Doing peer code review inside the team before raise PR to OSS. 
- Step 4. Raise PR to OSS, include:  
   - Code changes  
   - Test scripts  
   - End user documents  


## Phase 1 Tasks:
We need to build a MVP version and provide a more detailed proposal to the OS community and/or a PoC to demonstrate how it would work.

Documents need to be used as proposal for discussing:
- Requirements  
    [phase1-KQL-requirements](phase1-KQL-requirements.md)
- Design  
    [phase1-KQL-design](phase1-KQL-requirements.md)
- Test Plan   
    [phase1-KQL-testplan](phase1-KQL-testplan.md)


## Phase 2 Tasks:
 TBD
## Phase 3 Tasks:
TBD
## Phase 4 Tasks:  
TBD
