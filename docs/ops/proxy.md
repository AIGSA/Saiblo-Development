## 反向代理

在有些情况下，Saiblo 平台以一个端口对外提供服务，所以在 `frontend/nuxt.config.js` 中，我们配置了若干反向代理规则：

```js
proxy: {
  "/api/": {
    target: "http://localhost:8000",
    autoRewrite: true
  },
  "/admin/": {
    target: "http://localhost:8000"
  },
  "/static/": {
    target: "http://localhost:8000"
  },
  "/ws": {
    target: "http://localhost:8000",
    ws: true
  },
  "/judger_proxy/": {
    target: "http://localhost:8000",
    router: function(req) {
      let port = req.url.split("/")[2]
      return "http://localhost:" + port
    },
    pathRewrite: function(path) {
      return path.replace(/\/judger_proxy\/\d+\//, "/")
    },
    ws: true
  }
},
```

- `/api/`：到后端 API 的代理。注意这条规则只会代理到后端 `/api/` 下的地址，从而并不会暴露出后端 `/judger/` 的地址（这部分 API 是测评机的 API）。
- `/admin/`：到后端 Django Admin 的代理
- `/static/`：到后端静态文件的代理
- `/ws/`：到后端 `Websocket` 地址的代理
-  `/judger_proxy/`：用于测评机的端口转发（可不使用），将 `/judger_proxy/port/path` 转发到 `localhost:port/path`