# 贡献指南

### Fork 仓库

进入 [OneITOM/platform-boot](https://github.com/OneITOM/iplatform-boot) 的 [github 页面](https://github.com/apache/servicecomb-website) ，点击右上角按钮 `Fork` 进行 Fork。

![image-20190322145906973](assets/image-20190322145906973.png)

### Clone 你 Fork 的仓库到本机

1. 点击 Clone or download 按钮复制地址

![image-20190322145946882](assets/image-20190322145946882.png)



2. 将代码克隆到本地

```
git clone https://github.com/<your_github_name>/iplatform-boot.git
```

3. 添加 OneITOM/iplatform-boot 远程地址

```
cd iplatform-boot
git remote add upstream https://github.com/OneITOM/iplatform-boot.git
```

4. 查看本地关联的远程地址

```
git remote -v
origin	https://github.com/<your_github_name>/iplatform-boot.git (fetch)
origin	https://github.com/<your_github_name>/iplatform-boot.git (push)
upstream	https://github.com/OneITOM/iplatform-boot.git (fetch)
upstream	https://github.com/OneITOM/iplatform-boot.git (push)
```

### 修改代码并提交

> **注意：**尽量避免直接在master上修改代码，保持一个可持续同步的使用方法应该如下
>
> 1. 同步 OneITOM/iplatform 的 master 到你本地 master
> 2. 在你本地 master 上创建分支
> 3. 在分支上进行修改
> 4. 提交分支到你的 Github
> 5. 进入你的 Github 页面提交 PR
> 6. 等待你的 PR 被 OneITOM/iplatform-boot 合并
> 7. 同步 OneITOM/iplatform 的 master 到你本地 master
> 8. 推送你本地 master 到你的 Github 的 master 

1. 同步 OneITOM/iplatform 的 master 到你本地 master

    ```bash
    git pull upstream master
    ```

2. 在你本地 master 上创建分支

   > 创建完毕后可以通过 git branch 查看当前是在哪个分支，当前工作分支名前会有一个 `*` 号

    ```
    git checkout -b <your_branch_name>
    ```

3. 进行代码修改
4. 提交代码到本地仓库

    ```
    git commit -a -m "<you_commit_message>"
    ```

5. 推送当前分支到你的 Github

   ```bash
   git push origin <your_branch_name>
   ```

### 创建 PR

在浏览器切换到自己的 Github 页面，切换分支到提交的分支 <your_branch_name> ，依次点击 `New pull request`和 `Create pull request` 按钮进行创建，如下图所示





