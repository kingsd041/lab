# DevOps 工具鉴宝之 Atlassian Jira

从 DevOps 角度，体验 Atlassian 全家桶核心用户场景和产品功能，了解大型企业的需求管理和研发工具链落地实施。

## jira background (5 min)

- product introduction 
- user case 

## Jira configuration story (20 min - yy)

- plan 
- process 
- result 

## jira delivery story (20 min)

### requirement management (yy)

- product roadmap (gantt)
- PRD in confluence => user story => backlog 
- scrum board (kanban)
- active sprint
- kanban 
- close sprint (1 released, 1 undone)
- retrospective

### development integration (toby)

- select user story as example
- create feature branch
- code push & code review in bitbucket (jira in progress)
- Bitbucket pipeline (provided by atlassian)
  - build
  - test 
  - scan 
  - publish
  - deploy
- merge to master branch (jira revolved)
- CD triggered in master branch, go to staging environment
- pause prod deployment & gating
- start regression testing in staging (mention testing plugin)
- release management => change ticket
- validate ticket and go production
- close jira

### deployment management (yy)