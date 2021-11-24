## Revert用法

### 主要用途
revert 用于线上回滚代码

### 原理
- revert可以回滚指定提交，并产生一个新的提交 `eg: revert_commit_id`
- 由于revert之后产生了相反的提交，原来的提交会丢失，所以要回复revert的提交，可以把`revert_commit_id`再次revert

### 与reset区别
- [ ] 待完成
> 引用资料

### 实际如何操作

#### 开发背景

##### 主要分支
  - main (线上主要分支)
  - develop (测试分支)
  - feature (开发分支)

***
#### 情况1：feature分支合并到develop分支
  - 可以直接在feature上进行bugfix然后重新合并到develop分支
  - 也可以参考下面的revert方法

#### 情况2：feature分支合并到了main分支（上线完成）;发现feature分支存在bug需要回滚代码

- 因为featue分支是通过merge的方式合并到main分支，所以会生成一个新的`merge_commit_id`

- 此时找到该`merge_commit_id`在gitlab上使用revert功能回退代码
  ##### 新建一个revert分支`revert-62021261`，再合并到main分支生成一个`revert_62021261_merge_commit_id` （代码回滚成功）
    - 恢复提交
      ```
      // 切换到main
      
      git checkout main
      git pull
      
      git log // 找到revert_62021261_merge_commit_id
      git revert -m 1 revert_62021261_merge_commit_id // 由于是通过新建revert-62021261合并生成的revert提交 需要参数 -m 1 
      // 中间可能存在冲突，解决玩冲突重新 git add . / git commit -m 'ci: conflict'
      
      // 此时已经恢复到merge_commit_id状态
      
      git checkout featue
      // fixbug
      git add .
      git commit -m 'fix: bugfix'
      
      git chekout main
      git merge feature // 有冲突解决冲突
      git push
      ```
      > 如果是revert的是一个合并节点`mgere_commit`，该`merge_commit`就有两个parent, revert时git无法知道是恢复到哪个分支，所以会报错
      > `git revert -m 1 merge_commit`
      > 1代表当前分支，2代表合并过来的分支
  ##### 没有创建新的分支，直接生成了revert_commit_id（代码回滚成功）
    - 恢复提交（使用main分支）
      ```
      // 切换到main
      
      git checkout main
      git pull
      
      git log // 找到revert_commit_id
      git revert revert_commit_id // 相比较生成新分支的megre不需要 -m 1
      // 中间可能存在冲突，解决玩冲突重新 git add . / git commit -m 'ci: conflict'
      
      // 此时已经恢复到merge_commit_id状态
      
      git checkout featue
      // fixbug
      git add .
      git commit -m 'fix: bugfix'
      
      git chekout main
      git merge feature // 有冲突解决冲突
      git push
      ```
     - 恢复提交（使用feature分支）
       ```
       // 本地
       git checkout main
       git pull
       
       git chekout feature
       git merge main
       git log // 找到revert_commit_id
       
       git revert revert_commit_id
       // 恢复到了原来的状态
       
       // fixbug
       git add .
       git commit -m 'fix: bugfix'
       git push
       
       ```
       再次merge到main分支就可以了
       
   ##### 直接在gitlab操作
   
  1.找到`merge_commit_id`使用revert回滚代码（不要使用新建`revert-828929292`分支的方式）生成一个先的`revert_commit_id`（回滚成功)
  
  2.要想恢复之前的提交 找到`revert_commit_id`的提交，使用revert回滚代码（不使用新建`revert-12773612`分支的方式）生成一个`revert_revert_commit_id` （恢复成功）

### 参考文章
- [git如何回滚一次错误的操作](https://juejin.cn/post/6844903647390744589)
- [代码被revert,你pull了master,代码没了,咋找回来?](https://juejin.cn/post/6856039543628693517)
