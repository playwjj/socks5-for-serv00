# socks5-vmess-reset
在 Serv00 和 CT8 机器上一步到位地安装和配置 SOCKS5 & nezha-agent，并将其用于 cmliu/edgetunnel 项目，帮助解锁 ChatGPT 等服务。通过一键脚本实现代理安装，使用 Crontab 保持进程活跃，并借助 GitHub Actions 实现帐号续期与自动化管理，确保长期稳定运行。

## 如何使用？ [视频教程](https://youtu.be/L6gPyyD3dUw)

### nohup模式
- 一键安装 **新手小白用这个！**
```bash
bash <(curl -s https://raw.githubusercontent.com/playwjj/socks5-vmess-reset/main/install-socks5.sh)
```
----

## Github Actions保活
添加 Secrets.`ACCOUNTS_JSON` 变量
```json
[
  {"username": "cmliusss", "password": "7HEt(xeRxttdvgB^nCU6", "panel": "panel4.serv00.com", "ssh": "s4.serv00.com"},
  {"username": "cmliussss2018", "password": "4))@cRP%HtN8AryHlh^#", "panel": "panel7.serv00.com", "ssh": "s7.serv00.com"},
  {"username": "4r885wvl", "password": "%Mg^dDMo6yIY$dZmxWNy", "panel": "panel.ct8.pl", "ssh": "s1.ct8.pl"}
]
```

## 定时任务配置
在 `check_cron.sh` 脚本中，VMESS_TOKEN 现在从文件 `/home/${USER}/.vmess/token` 中读取。确保该文件存在并包含有效的 token。以下是相关代码片段：

```shell
if [ -e "/home/${USER}/.vmess/config.json" ]; then
  echo "添加vmess重启任务"
  # 检查并添加 web 进程的定时任务
  (crontab -l | grep -F "*/12 * * * * pgrep -x \"web\"") || (crontab -l; echo "*/12 * * * * pgrep -x \"web\" > /dev/null || nohup /home/${USER}/.vmess/web run -c /home/${USER}/.vmess/config.json >/dev/null 2>&1 &") | crontab -

  # 检查 token 文件
  if [ ! -f "/home/${USER}/.vmess/token" ]; then
    echo "错误: token文件不存在于 /home/${USER}/.vmess/token"
    exit 1
  fi
  
  # 读取token
  VMESS_TOKEN=$(cat "/home/${USER}/.vmess/token")
  # 创建 bot 进程的定时任务
  BOT_COMMAND="*/12 * * * * pgrep -x \"bot\" > /dev/null || nohup /home/${USER}/.vmess/bot tunnel --edge-ip-version auto --no-autoupdate --protocol http2 run --token ${VMESS_TOKEN} >/dev/null 2>&1 &"
  
  if (crontab -l | grep -F "*/12 * * * * pgrep -x \"bot\"" >/dev/null 2>&1); then
    echo "bot进程的定时任务已存在"
  else
    echo "添加bot进程的定时任务"
    (crontab -l 2>/dev/null; echo "${BOT_COMMAND}") | crontab -
  fi
fi
```

# 致谢
[RealNeoMan](https://github.com/Neomanbeta/ct8socks)、[k0baya](https://github.com/k0baya/nezha4serv00)、[eooce](https://github.com/eooce)
