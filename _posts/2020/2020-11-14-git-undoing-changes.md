---
layout: post
title: Git undoing changes.
category: tool
tags: [tool]
excerpt: Git undoing changes.
---

## Three trees    

先来看下整体结构：   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/three_trees.png) 

首先，你需要了解`Git`中存在三棵树，分别是`Working Directory`, `Staging Index`和`Commit History`。  

其中`Working Directory`就是当前的你的工作环境，当你在本地做了任何改动：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/changes_not_staged_for_commit.png)  


此时你就可以使用`add`命令将其添加到`Staging Index`中： 

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/changes_to_be_committed.png)   

然后，你可以继续使用`commit`命令将其添加到`Commit History`中：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/after_commit.png)   


## checkout    

然后，我们来介绍一下`checkout`命令，它的作用是什么？  

第一，它可以用来切换分支。  

命令: `git checkout branch_name`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/switch_branch.png)   

如果切换一个并不存在的分支会怎么样？  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/switch_not_existed_branch.png)   

可以看到，抛出了一个`error`。  

第二，它可以用来创建并切换分支。  

命令: `git checkout -b new_branch_name`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/create_and_switch_branch.png)   

等同于如下命令：  

``` java
git branch c;
git checkout c;
```

第三，它可以撤销在`Working Directory`改动的文件。   

命令: `git checkout -- file_name`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/rollback_file_in_working_directory.png) 


## rm    

接着，我们讨论下`rm`命令，老规矩，为什么要使用它？  

它与`add`属于配套命令，`add`命令负责将本地的改动文件添加到`Staging Index`，  

而它则负责将`Working Directory`和`Staging Index`的改动文件删除。    

命令: `git rm -f file_name`

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/rm_file_name.png) 


## reset    

继续，那么`reset`命令的作用是什么？  

它通过移动`Head`指针来撤销`本地尚未push`的`commit`。  

来看下示意图，`reset`之前：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/before_reset.png) 

`reset`之后：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/after_reset.png) 


首先，我们模拟在本地进行三次`commit`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/add_three_commits.png) 

然后，回退到`bb45266`：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/reset_to_a.png) 

注意喔，`git reset commit_id` 默认使用`mix`模式。  

接着，我们详细讨论一下`reset`的三种模式，分别为`hard`,`soft`和`mixed`。  


先来看下`hard`。  

> Resets the index, working tree and commit history.   
> Any changes to tracked files in the working tree since <commit> are discarded.  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/reset_hard.png) 


再来看下`soft`:  

> Does not touch the index file or the working tree at all (but resets the head to <commit>, just like all modes do).  
> This leaves all your changed files "Changes to be committed".  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/reset_soft.png) 


最后来看下默认模式`mix`：  

> Resets the index and commit history but not the working tree.   
> This is the default action.  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/reset_mix.png) 




## revert    

老规矩，`revert`命令的作用是什么？  

当你想撤销一个或多个已`pushed`的提交时，就可以使用`revert`，注意它并不会删除已有的`commit history`。  

举个例子。  

第一步，分别提交四个`commit`并`push`到远程仓库。  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/revert_preparation.png) 


第二步，撤销其中三个提交。   

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/do_revert.png) 


此时，`Working Directory`中只有剩余一个`commit`的内容，也就是类`C`。     

最后将本地的改动`push`到远程仓库即可：`git push origin branch_name`  


## Reference  

- [How to undo local changes to a specific file [duplicate]](https://stackoverflow.com/questions/31281679/how-to-undo-local-changes-to-a-specific-file){:target="_blank"}    
- [Undoing Commits & Changes](https://www.atlassian.com/git/tutorials/undoing-changes){:target="_blank"}    
- [Git Checkout](https://www.atlassian.com/git/tutorials/using-branches/git-checkout){:target="_blank"}    
- [Git RM](https://www.atlassian.com/git/tutorials/undoing-changes/git-rm){:target="_blank"}    
- [git-reset](file:///D:/Software/Git/mingw64/share/doc/git-doc/git-reset.html){:target="_blank"}    