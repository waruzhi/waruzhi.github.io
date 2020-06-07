---
title: git branch model总结
date: 2020-06-06 23:46:10
tags: best-practise
---
刚到新公司的时候，发现大家的git分支和部署环境有点混乱，大家没有一个统一的标准，经常会因此分支混乱导致一些不必要的错误。因为之前工作中也用过各种各样的分支管理模型，但是一直没有好好梳理一下，所以就趁机稍微研究了一下，把调研的总结记录一下，并且最后写一下我们最终选择的方案是什么。

# 常见的git分支策略
### GitHub Flow
GitHub Flow是最简单粗暴的flow。它只有一个核心的master分支和一堆其他分支。每次要加新feature或者要fix一个bug，那么就从master拉出一个新分支，然后经过commit，test，pull request，deploy，最后merge进master。GitHub Flow的优点是简单，缺点是太简单，如果有多套部署环境，或者有强的release版本的需求，它都不能支持。
{% asset_img github_flow.png %}
### Git Flow
Git Flow比GitHub Flow复杂很多，引入了几种新的类型的branch。
* develop 有点类似上面的master，所有经过充分测试和review的commit都会合到这个分支。
* master 像develop的精简版，每个commit都有tag，对应develop上面多个commit。通过master可以很方便的找到某个版本的全部代码。
* feature-* 字面意思，都从develop拉出，然后合回develop。
* hotfix-* 如果线上发现问题需要hotfix，需要用到hotfix分支，都从master拉出，然后合回master和develop。
* release-* 每次develop的commit合进master之前需要拉出一个release分支，然后做一些version bump之类的操作，然后合回master和develop。这个分支看起来比较鸡肋，但是其实用处很大，一个场景是如果develop上面有多个在测试的feature，但是这次release并不想把它们都发出去，就可以在release分支上面只抽取出想要的feature。

Git Flow还是很全面的，我们最终的方案也基本上参照了它的操作。不过它的复杂性也令很多项目望而却步。
{% asset_img git_flow.png %}
### GitLab Flow
GitLab Flow强调的是不那么复杂，但是又可以支持多种环境和记录release版本。它的主分支还是master分支，一般会部署在staging环境，如果要部到production环境的话，需要从master提到production的pr。如果还有其他环境，比如pp，那么会在master和production之间再添加一个pp分支。至于release分支的话是为了记录某一个版本的release，如果需要release一个版本，只要从master拉出一个分支就可以。
{% asset_img gitlab_flow_environment_branches.png %}
{% asset_img gitlab_flow_release_branches.png %}

# 我们项目的选择
我们最终选择的方案大部分参照了git flow的实现，具体做法是
* master分支对应git flow的develop。因为一般项目默认的主分支是master，所以这样可以避免混淆。
* release分支对应git flow的master。在release分支上，每一个commit都有tag，对应一个发布版本。
* feature分支和hotfix分支和git flow一致。
* pre-release分支对应git flow的release分支。每次要release一个版本之前，都会从master拉出一个pre-release分支，然后在上面bump version，有时候也会有一些小的bug fix和配置改变。我们使用maven的release plugin来帮我们做version bump。
* environment分支。我们的环境除了staging和production以外，可能还会有pp、qa、load、simulation。为了应该各种环境，我们计划参照Gitlab Flow，每个环境分支对应那个环境实际部署的版本。但是在实际操作过程中，过多的分支会给开发人员带来很大的负担，最好能配合好用的git相关工具。所以我们暂时没有使用这些分支，而是还是主要使用master和feature分支。

目前来看，这个分支策略还有一些欠缺，比如hotfix和pre-release每次merge都得merge进release和master两个分支，一不小心就容易忘记。还有环境分支的使用上现在还没有找到合适的工具。之后随着我们的策略的更新，再来进一步更新吧。

参考资料：
1. https://guides.github.com/introduction/flow/
2. https://nvie.com/posts/a-successful-git-branching-model/
3. https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow
4. https://docs.gitlab.com/ee/topics/gitlab_flow.html#production-branch-with-gitlab-flow
