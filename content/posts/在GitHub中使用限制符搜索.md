---
title: "在GitHub中使用限制符搜索"
date: 2020-04-23T21:13:21+08:00
draft: false
tags: [ "GitHub" ]
categories: [ "技术文档" ]
---
# 在GitHub中使用限制符搜索

## 使用可视界面搜索
使用 [搜索](https://github.com/search) 页面 或 [高级搜索](https://github.com/search/advanced) 页面 搜索 GitHub。

## 使用搜索限定符搜索
### 搜索仓库
- `in:name,description,readme`：通过 `in` 限定符，搜索限制为仓库名称、仓库说明、自述文件内容或这些的任意组合。如果省略此限定符，则只搜索仓库名称和说明。
- `repo:owner/name`：匹配特定仓库名称。
- `user:*USERNAME*`：按特定用户搜索。
- `org:*ORGNAME*`：按特定组织搜索。
- `size:*n*`：按仓库大小搜索，使用>=、<和...等查找匹配特定大小（以千字节为单位）的仓库。
- `followers:*n*`：按关注者数量搜索。
- `forks:*n*`：按复刻数量搜索。
- `stars:*n*`：按星号数量搜索。
- `created:*YYYY-MM-DD*` `pushed:*YYYY-MM-DD*`：按仓库创建或上次更新时间搜索，日期支持`*n*`大于、小于和范围限定符。
- `language:LANGUAGE`：按编写采用的主要语言搜索。
- `topic:*TOPIC*`：按主题搜索。
- `topics:*n*`：按主题数量搜索。
- `license:*LICENSE_KEYWORD*`：按其许可搜索仓库，必须使用按特定许可或许可系列过滤仓库。
- `is:public` `is:private`：匹配公共、私有仓库。
- `mirror:true` `mirror:false`：匹配镜像、非镜像仓库。
- `archived:true` `archived:false`：匹配已存档、未存档仓库。
- `good-first-issues:>n` `help-wanted-issues:>n`：搜索具有最少数量标签为 `help-wanted` 或 `good-first-issue` 议题的仓库。

### 搜索主题
- `is:curated`：将搜索结果范围缩小到社区成员已向其添加额外信息的主题。
- `is:featured`：将搜索结果范围缩小为 GitHub 上具有最多仓库的主题。
- `is:not-curated`：匹配没有额外信息（例如说明或徽标）的主题。
- `is:not-featured`：将搜索结果范围缩小为 GitHub 上未提供的主题。
- `repositories:n`：超过数量的仓库的主题。
- `created:*YYYY-MM-DD*`：按创建时间搜索。
- `topic:`：按主题搜索仓库，查找连接到特定主题的每个仓库。

### 搜索代码
- `in:file,path`：搜索限制为源代码文件的内容、文件路径或两者。
- `repo:owner/name`：匹配特定仓库名称的代码。
- `user:*USERNAME*`：按特定用户搜索代码。
- `org:*ORGNAME*`：按特定组织搜索代码。
- `path:PATH/TO/DIRECTORY`：按文件位置搜索。
- `language:LANGUAGE`：按编写采用的主要语言搜索。
- `size:*n*`：按仓库大小搜索，使用>=、<和...等查找匹配特定大小（以千字节为单位）的代码。
- `filename:FILENAME`：按文件名搜索。
- `extension:EXTENSION`：按文件扩展名搜索。

### 搜索提交
- `author:*USERNAME*` `committer:*USERNAME*` `author-name:*NAME*` `committer-name:*NAME*` `author-email:*EMAIL*` `committer-email:*EMAIL*`：按作者或提交者搜索。
- `author-date:*YYYY-MM-DD*` `committer-date:*YYYY-MM-DD*`：按创作或提交日期搜索，日期支持大于、小于和范围限定符。
- `merge:true` `merge:false`：匹配合并、非合并提交。
- `hash:*HASH*` `parent:*HASH*` `tree:*HASH*`：匹配具有、父项具有、引用树指定 SHA-1 哈希的提交。
- `repo:owner/name`：匹配特定仓库名称的提交。
- `user:*USERNAME*`：按特定用户搜索提交。
- `org:*ORGNAME*`：按特定组织搜索提交。
- `is:public` `is:private`：匹配公共、私有提交。

### 搜索议题和拉取请求
- `type:pr` `type:issue` `is:pr` `is:issue`：仅搜索议题或拉取请求。
- `in:title` `in:body` `in:comments`：按标题、正文或评论搜索。
- `repo:owner/name`：匹配特定仓库名称的议题或拉取。
- `user:*USERNAME*`：按特定用户搜索议题或拉取。
- `org:*ORGNAME*`：按特定组织搜索议题或拉取。
- `state:open` `state:closed` `is:open` `is:closed`：按开放或关闭状态搜索。
- `is:public` `is:private`：匹配公共、私有议题或拉取。
- `assignee:*USERNAME*`：按受理人搜索。
- `mentions:*USERNAME*`：按提及搜索。
- `team:*ORGNAME/TEAMNAME*`：按团队提及搜索。
- `commenter:*USERNAME*`：按评论者搜索。
- `involves:*USERNAME*`：按议题或拉取请求中涉及的用户搜索。
- `linked:pr` `linked:issue` `-linked:pr` `-linked:issue`：按链接的议题或拉取搜索。
- `label:*LABEL*`：按标签搜索。
- `milestone:*MILESTONE*`：按里程碑搜索。
- `project:*PROJECT_BOARD*` `project:*REPOSITORY/PROJECT_BOARD*`：按项目板搜索。
- `status:pending` `status:success` `status:failure`：按提交状态搜索。
- `*SHA*`：按提交 SHA 搜索。
- `head:*HEAD_BRANCH*` `base:*BASE_BRANCH*`：按分支名称搜索。
- `language:*LANGUAGE*`：按语言搜索。
- `comments:*n*`：按评论数量搜索。
- `interactions:*n*`：按交互数量搜索。
- `reactions:*n*`：按反应数量搜索。
- `draft:true` `draft:false`：搜索草稿拉取请求。
- `review:none` `review:required` `review:approved` `review:changes_requested` `reviewed-by:*USERNAME*` `review-requested:*USERNAME*` `team-review-requested:*TEAMNAME*`：按拉取请求审查状态和审查者搜索。
- `created:*YYYY-MM-DD*` `updated:*YYYY-MM-DD*` `closed:*YYYY-MM-DD*` `merged:*YYYY-MM-DD*`：按议题或拉取请求创建、上次更新、关闭或合并的时间搜索。
- `is:merged` `is:unmerged`：基于拉取请求是否已合并搜索。
- `archived:true` `archived:false`：基于仓库是否已存档搜索。
- `is:locked` `is:unlocked`：基于对话是否已锁定搜索。
- `no:label` `no:milestone` `no:assignee` `no:project`：按缺少的元数据搜索。

### 搜索用户
- `type:user` `type:org`：仅搜索用户或组织。
- `user:name` `org:name` `in:login` `in:name` `fullname:firstname lastname` `in:email`：按帐户名、全名或公共电子邮件搜索。
- `repos:*n*`：按用户拥有的仓库数量搜索。
- `location:*LOCATION*`：按位置搜索。
- `language:*LANGUAGE*`：按仓库语言搜索。
- `created:*YYYY-MM-DD*`：按用户帐户创建时间搜索。
- `followers:*n*`：按关注者数量搜索。

### 搜索 wiki
- `repo:owner/name`：匹配特定仓库名称的wiki。
- `user:*USERNAME*`：按特定用户搜索wiki。
- `org:*ORGNAME*`：按特定组织搜索wiki。
- `in:title` `in:body`：在 wiki 页面标题或正文文本中搜索。
- `updated:*YYYY-MM-DD*`：按上次更新日期搜索。

### 在复刻中搜索
- `fork:true`：包含复刻搜索。
- `fork:only`：仅在复刻中搜索。