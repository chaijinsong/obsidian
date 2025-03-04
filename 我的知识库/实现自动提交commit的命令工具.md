### 步骤 1: 创建通用脚本文件

创建一个名为 `git-autocommit` 的文件（无后缀），内容如下：

``` python
#!/usr/bin/env python3

import os
import time
import argparse
import subprocess
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class GitAutoCommitHandler(FileSystemEventHandler):
    def __init__(self, remote, branch, debounce=5):
        self.remote = remote
        self.branch = branch
        self.debounce_seconds = debounce
        self.last_commit_time = 0

    def on_any_event(self, event):
        if time.time() - self.last_commit_time > self.debounce_seconds:
            self.commit_changes()
            self.last_commit_time = time.time()

    def commit_changes(self):
        try:
            # Git 操作三部曲
            subprocess.run(["git", "add", "."], check=True)
            subprocess.run(
                ["git", "commit", "-m", "Auto commit: detected changes"],
                check=True
            )
            subprocess.run(
                ["git", "push", self.remote, self.branch],
                check=True
            )
            print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Changes pushed to {self.remote}/{self.branch}")
        except subprocess.CalledProcessError as e:
            if "nothing to commit" in e.stderr.decode():
                print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] No changes to commit")
            else:
                print(f"[ERROR] Git operation failed: {e}")

def main():
    parser = argparse.ArgumentParser(description="Auto-commit and push directory changes to Git")
    parser.add_argument("--remote", required=True, help="Git remote repository URL (e.g. https://github.com/user/repo.git)")
    parser.add_argument("--branch", default="main", help="Git branch name (default: main)")
    parser.add_argument("--debounce", type=int, default=5, help="Debounce time in seconds (default: 5)")
    args = parser.parse_args()

    # 检查当前目录是否是 Git 仓库
    if not os.path.exists(".git"):
        print("❌ 当前目录不是 Git 仓库！请先运行 git init")
        return

    # 启动监控
    observer = Observer()
    event_handler = GitAutoCommitHandler(args.remote, args.branch, args.debounce)
    observer.schedule(event_handler, path=".", recursive=True)
    observer.start()

    print(f"🔍 开始监控目录: {os.getcwd()}")
    print(f"   Remote: {args.remote}")
    print(f"   Branch: {args.branch}")
    print("按下 Ctrl+C 停止监控")

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

if __name__ == "__main__":
    main()
```

---

### 步骤 2: 安装工具


```bash
# 移动脚本到可执行路径
sudo mv ~/Downloads/git-autocommit /usr/local/bin/

# 赋予执行权限
sudo chmod +x /usr/local/bin/git-autocommit

# 安装依赖
pip3 install watchdog
```


---

### 使用示例

假设你在 `/Users/didi/Documents/Obsidian` 目录下操作：

bash

复制
```bash
cd /Users/didi/Documents/Obsidian

# 首次使用需要初始化 Git 仓库
git init
git remote add origin https://github.com/chaijinsong/obsidian.git

# 启动自动提交工具
git-autocommit --remote origin --branch main
```


---

### 通用化特性

**动态目录**  
自动使用你执行命令时的当前目录作为监控目录和 Git 仓库。
    
2.**参数配置**  
    通过命令行参数指定必要信息：
    
```bash
    git-autocommit --remote <REMOTE_URL> [--branch BRANCH] [--debounce SECONDS]
    
    - `--remote`: 必填，Git 远程仓库地址（支持 SSH/HTTPS）
        
    - `--branch`: 选填，目标分支（默认 `main`）
        
    - `--debounce`: 选填，防抖时间（单位秒，默认 5）
        
3. **自动验证**  
    如果当前目录不是 Git 仓库，会直接报错退出。
    
4. **友好输出**  
    明确显示监控配置和操作状态：
    
    复制
    
    🔍 开始监控目录: /Users/didi/Documents/Obsidian
       Remote: origin
       Branch: main
    按下 Ctrl+C 停止监控
    

---

### 典型工作流

bash

复制

# 进入任意需要监控的目录
cd /path/to/your/project

# 初始化 Git 仓库（如果尚未初始化）
git init
git remote add origin YOUR_REMOTE_URL

# 启动守护进程
git-autocommit --remote origin --branch dev --debounce 10

# 后台运行（关闭终端不中断）
nohup git-autocommit --remote origin --branch main > ~/autocommit.log 2>&1 &

这个工具现在可以在任何 Git 仓库中使用，只需通过参数动态指定远程仓库和分支，完美满足你的通用性需求！