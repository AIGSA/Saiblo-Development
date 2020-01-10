## 定时对局抽取

在比赛系统中，对局会根据一定的规则定时自动生成。这利用了 Linux 系统的 `crontab`，故在使用该功能之前需要预先安装 `crontab` 依赖。

确认 `crontab` 启动后，执行以下命令添加定时对局抽取任务

```bash
python manage.py crontab add
```

每间隔 3 分钟便会添加定时对局抽取的任务，如需修改生成对局的间隔，请在 `backend/app/settings.py` 的 CRONJOBS 配置中修改。

可以在 `backend/auth_match.log` 下查看定时对局抽取的日志。