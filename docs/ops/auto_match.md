## 定时对局抽取

在比赛系统中，对局会根据一定的规则自动生成，需要利用 `crontab` 来添加定时任务：

```bash
python manage.py crontab add
```

这就会添加定时对局抽取的任务，可以在 `backend/auth_match.log` 下查看自动对局生成的日志。