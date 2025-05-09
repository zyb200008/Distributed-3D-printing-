---

# ChatGPT Web 项目 Docker 操作文档

本文档介绍了如何使用 Docker 和 docker-compose 快速部署 ChatGPT Web 项目，同时包含自定义镜像构建、网络连接、以及 MongoDB 数据库的备份与恢复步骤。

---

## 1. 拉取项目代码

首先，从代码仓库拉取最新版代码：

```sh
git clone <项目仓库地址>
cd <项目目录>
```

> **注意:** 后续所有操作都在项目目录下进行。

---

## 2. 启动/恢复所有容器

如需启动已存在的所有容器：

```sh
docker start $(docker ps -a -q)
```

---

## 3. 使用 docker-compose 启动服务

在项目目录下，首次或更新后重建并启动服务：

```sh
docker-compose up -d
```

---

## 4. 加入自定义 Docker 网络

确保容器与指定网络关联，如 `hulber-network`:

```sh
docker network connect hulber-network chatgptweb
```

---

## 5. 构建自定义镜像

如需集成参数和代码版本信息，执行以下命令构建镜像：

```sh
docker build -t my-chatgptweb:latest \
  --build-arg GIT_COMMIT_HASH=$(git rev-parse HEAD) \
  --build-arg RELEASE_VERSION=v1.0.0 \
  .
```

---

## 6. MongoDB 数据库备份

### 6.1 备份数据

首先在数据库容器内删除旧备份：（**chatgptweb-database**为数据库容器名）

```sh
docker exec chatgptweb-database rm -rf /data/backup
```

然后执行备份操作：

```sh
docker exec chatgptweb-database mongodump -u chatgpt -p mongo123 --authenticationDatabase admin -d chatgpt -o /data/backup
```

将备份拷贝至主机：

```sh
docker cp chatgptweb-database:/data/backup /home/ubuntu/mongo_backup
```

---

## 7. MongoDB 数据库恢复

### 7.1 恢复数据

将本地备份文件夹拷贝回数据库容器：

```sh
docker cp /home/ubuntu/mongo_backup chatgptweb-database:/data/backup
```

在数据库容器中执行恢复命令：

```sh
docker exec chatgptweb-database mongorestore -u chatgpt -p mongo123 --authenticationDatabase admin -d chatgpt /data/backup/chatgpt
```

---

## 8. 常用命令摘要

- 查看所有容器：`docker ps -a`
- 进入容器：`docker exec -it <容器名/ID> bash`
- 查看日志：`docker logs <容器名/ID>`

---

## 9. 注意事项

- 所有操作推荐在 root 或有足够权限账号下执行。
- 请根据实际数据库用户名、密码、数据库名调整相关命令。
- 定期对 MongoDB 数据进行备份并妥善保存备份文件。

---
