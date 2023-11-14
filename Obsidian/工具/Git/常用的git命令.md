Git 是一个广泛使用的版本控制系统，非常有用于代码的版本管理和协作。以下是一些常用的 Git 命令：

1. **配置 Git**
   - `git config --global user.name "Your Name"`：设置用户名。
   - `git config --global user.email "your_email@example.com"`：设置用户邮箱。

2. **初始化仓库和克隆**
   - `git init`：在当前目录初始化一个新的 Git 仓库。
   - `git clone <repository>`：克隆一个远程仓库。

3. **查看状态和日志**
   - `git status`：查看当前仓库状态。
   - `git log`：查看提交历史。

4. **添加和提交更改**
   - `git add <file>`：添加文件到暂存区。
   - `git commit -m "commit message"`：提交暂存区的更改。

*** 
<details>
<summary> git add` 和 `git commit` 详细用法 </summary>
</details>

### 1. `git add`

`git add` 命令用于将更改（新文件、修改过的文件）添加到暂存区（staging area），准备进行提交。这是提交流程的第一步。

- **添加单个文件**：`git add <file_name>`
  - 例如，`git add file.txt` 会将名为 `file.txt` 的文件添加到暂存区。

- **添加多个文件**：`git add <file1> <file2> ...`
  - 例如，`git add file1.txt file2.txt` 会将 `file1.txt` 和 `file2.txt` 添加到暂存区。

- **添加目录**：`git add <directory>`
  - 例如，`git add my_folder/` 会将 `my_folder` 目录下的所有更改添加到暂存区。

- **添加所有更改**：`git add .` 或 `git add -A`
  - `git add .` 添加当前目录下所有的更改到暂存区。
  - `git add -A` 添加整个工作目录的所有更改到暂存区。

- **添加忽略的文件**：`git add -f <file>`
  - 如果某个文件被 `.gitignore` 忽略，使用 `git add -f file.txt` 可以强制将该文件添加到暂存区。

### 2. `git commit`

`git commit` 命令用于将暂存区的更改提交到仓库。这是创建新历史记录点的过程，记录了谁在何时做了什么更改。

- **简单提交**：`git commit -m "commit message"`
  - 使用 `-m` 参数后跟提交信息来进行提交。例如：`git commit -m "fix bug #123"`。

- **多行提交信息**：`git commit`
  - 如果不加 `-m` 参数，Git 会打开文本编辑器供您写入更长的提交信息，支持多行。

- **添加更改并提交**：`git commit -am "commit message"`
  - 如果是对已追踪（tracked）文件的修改，可以使用 `-am` 来同时添加并提交。

- **修改最后一次提交**：`git commit --amend`
  - 如果需要修改最后一次提交（例如，更改提交信息或添加遗漏的更改），可以使用 `--amend` 选项。这会替换上一次的提交。

- **跳过暂存区直接提交**：`git commit -a`
  - 对于已经追踪的文件，`-a` 选项会让 Git 自动将它们添加到暂存区并进行提交。

请注意，合理地使用 `git add` 和 `git commit` 对于维护清晰、可读的版本历史非常重要。每次提交时，应该确保提交信息清晰地描述了更改的内容和目的。

***


5. **分支管理**
   - `git branch`：列出所有本地分支。
   - `git branch <branch_name>`：创建新分支。
   - `git checkout <branch_name>`：切换到指定分支。
   - `git merge <branch>`：合并指定分支到当前分支。

6. **拉取和推送更改**
   - `git pull`：从远程仓库拉取最新更改并合并到当前分支。
   - `git push`：将本地分支的更新推送到远程仓库。

7. **撤销操作**
   - `git checkout -- <file>`：撤销指定文件的更改（未暂存的）。
   - `git reset HEAD <file>`：从暂存区移除文件。
   - `git revert <commit>`：撤销指定的提交。

8. **查看更改**
   - `git diff`：查看未暂存的文件更改。
   - `git diff --staged`：查看暂存的文件更改。

9. **远程仓库操作**
   - `git remote add <name> <url>`：添加新的远程仓库。
   - `git remote -v`：查看远程仓库信息。

10. **标签操作**
    - `git tag`：列出所有标签。
    - `git tag <tagname>`：创建新标签。

这些只是 Git 命令的基础，Git 的功能非常强大且复杂，可以根据不同的开发需求进行各种操作。如果你是 Git 的新手，建议从这些基本命令开始学习，并逐渐掌握更高级的功能。