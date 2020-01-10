## Wiki 系统

Saiblo 平台采用 [Mkdocs](https://www.mkdocs.org/) 作为 Wiki 系统，在安装后端 Python 的依赖时，应当已经安装过 Mkdocs 依赖。以下为 `mkdocs/` 文件夹的目录结构

```
mkdocs
├── docs # 文档存放地址
├── mkdocs.yml # 配置文件
└── site # 生成的文件
```

其中 `site` 是 Mkdocs 生成的 Wiki 站点源代码，这需要通过 Mkdocs 构建生成

```bash
mkdocs build
```

我们配置了 `frontend/static/wiki` 指向 `mkdocs/site` 的软连接，在执行 `mkdocs build` 之后前端应能及时看到文档的变化。