# Flask Web App 实验

这个项目用于头歌的 GitHub Actions 自动测试与部署实验。首次部署必须在头歌服务器上完成；之后推送到 `main` 会先运行测试，成功后通过 SSH 更新服务器。

## 本机检查

```bash
python -m pip install -r requirements.txt
python -m pip install pytest
python -m pytest tests/ -v
```

应用入口是 `src/app.py`，监听 5000 端口，初始页面返回 `Hello, World!`。

## 你需要完成的步骤

1. 在 GitHub 创建**公开的空仓库** `flask-web-app`，不要勾选生成 README、`.gitignore` 或许可证。公开仓库便于头歌服务器无交互拉取代码；如果按课程要求使用私有仓库，还须自行配置服务器读取该仓库的凭据。本项目已在本地提交，在本项目目录运行：

   ```bash
   git remote add origin https://github.com/你的用户名/flask-web-app.git
   git push -u origin main
   ```

   GitHub 登录与推送由你完成。不要把个人访问令牌写进文件或远程仓库地址。

2. 在头歌实验页面的「实验环境」中打开「SSH直连」，记下当前公网 IP、SSH 端口、用户名和密码。在服务器上运行：

   ```bash
   apt update && apt install -y python3 python3-pip git supervisor
   mkdir -p /root/flask-web-app
   cd /root/flask-web-app
   git clone https://github.com/你的用户名/flask-web-app.git .
   python3 -m pip install -r requirements.txt
   service supervisor start
   ```

   若仓库是私有的，需要由你完成服务器上的 GitHub 认证。不要将令牌写入命令或仓库文件。

3. 在服务器复制已准备好的 Supervisor 配置并启动应用：

   ```bash
   cp /root/flask-web-app/deploy/flask-web-app.conf /etc/supervisor/conf.d/flask-web-app.conf
   supervisorctl reread
   supervisorctl update
   supervisorctl status flask-web-app
   curl http://127.0.0.1:5000/
   ```

   状态应为 `RUNNING`，页面内容应为 `Hello, World!`。在头歌「服务预览」里添加内部端口 `5000`，打开生成的外部链接验证。

4. 在本机生成专用 SSH 密钥。Windows 建议在 Git Bash 中运行：

   ```bash
   mkdir -p ~/.ssh
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/touge_flask_deploy -N ""
   ```

   用 `cat ~/.ssh/touge_flask_deploy.pub` 查看公钥。把输出的**完整一行公钥**追加到头歌服务器当前部署用户的 `~/.ssh/authorized_keys`，将 `~/.ssh` 设为 700、`authorized_keys` 设为 600。用 `ssh -i ~/.ssh/touge_flask_deploy -p 实际端口 用户名@实际IP` 验证无需密码登录。不要向任何人发送私钥。

5. 在 GitHub 仓库 `Settings → Secrets and variables → Actions` 中新增：

   | 类型 | 名称 | 内容 |
   | --- | --- | --- |
   | Repository secret | `TOUGE_SSH_PRIVATE_KEY` | 专用私钥的完整内容，包括头尾标记 |
   | Repository secret | `TOUGE_SERVER_IP` | 头歌当前公网 IP |
   | Repository variable（可选） | `TOUGE_SSH_PORT` | 头歌实际 SSH 端口；未设置时使用当前端口 `53647` |
   | Repository variable（可选） | `TOUGE_SSH_USER` | 服务器登录用户；未设置时使用 `root` |

   工作流使用 `/root/flask-web-app` 路径，所以上面的部署配置预期用 root 登录。若平台只能以其他用户登录，需同步修改仓库路径、Supervisor 配置与工作流。服务器 IP 或端口发生变化时，更新对应 GitHub 值。

6. 第一次推送发生在服务器和 GitHub 配置完成前，因此那次 Actions 部署失败是预期的。完成上面配置后，可以在 GitHub 仓库的 `Actions` 页手动运行一次工作流，检查首次自动部署。

7. 在 `src/app.py` 中把 `Hello, World!` 改为 `Hello, NEU!`，并在 `tests/test_app.py` 中修改相同的预期内容。提交并推送 `main`。到 GitHub 仓库的 `Actions` 页等待 `test`、`deploy` 都成功，再刷新头歌服务预览，确认显示 `Hello, NEU!`。

8. 头歌代码文件中如有题目要求填写的模板，请填入你的 GitHub 用户名、仓库 URL、头歌当前公网 IP、SSH 端口、服务器用户名，以及服务预览外部链接。个人访问令牌只能填在头歌指定的私密输入位置；不要提交到 GitHub 或发给助手。

## 常见故障

- `test` 失败：先在本机运行测试，确保应用文本与测试预期一致。
- SSH 失败：确认头歌当前 IP、端口、用户名、公钥与密钥，特别是密钥内容含完整首尾行。
- `deploy` 失败：在服务器运行 `supervisorctl status flask-web-app`，查看 `/root/flask-web-app/app.log`。
- 服务预览无法打开：先用 `curl http://127.0.0.1:5000/` 检查本机服务，再检查头歌是否添加 5000 端口。
