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
  - 拉取分支时，自动更新 Jira Story 卡片到 In Progress 状态
  - PR Merged时，自动更新 Jira Story 卡片到 Resolved 状态
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
- Jira Service Management：变更管理：
- 在CD中，到Production部署时，会自动创建线上变更申请单；
- CD关联的代码仓库对应的服务变更的Approvers收到审批提醒
- Service Approvers审批通过
- CD生产环境部署继续
- CD生成环境部署完成（成功），Jira Story卡片自动更新到 Done 状态