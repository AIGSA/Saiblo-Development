# 概览

本文档为 Saiblo 平台的开发维护手册，由「运维手册」和「开发手册」两个部分组成。维护者可以参考「运维手册」从零开始配置 Saiblo 平台并使其功能正常运行。开发者可以参考「开发手册」获得 Saiblo 平台各个部分的技术细节，从而在此基础之上进行二次开发。

Saiblo 平台网站端的目录结构为：

```
├── Dockerfile # Dockerfile，用于编译 Docker 镜像
├── README.md
├── backend # 后端目录
├── config # 一些启动脚本
├── frontend # 前端目录
├── mkdocs # Wiki 目录
└── sources.list # apt 源（在 Dockerfile 中被使用）
```

Saiblo 平台的前端、后端使用的框架分别为：

- 前端：Nuxt.js
- 后端：Django REST Framework，Django Channels

除此之外，平台 Wiki 系统使用 Mkdocs 进行构建，生成静态文档站点。

在运维或二次开发时，应当对这些框架有一个基本的了解。如想详细了解各模块的功能，请参照「运维手册」和「开发手册」的对应部分。