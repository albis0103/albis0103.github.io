
# Docker 常用指令

## 映像檔 (Image)

```bash
docker pull <image>:<tag>        # 拉取映像檔
docker images                    # 列出本機映像檔
docker build -t <name>:<tag> .   # 依 Dockerfile 建置
docker rmi <image>               # 刪除映像檔
docker image prune -a            # 清除未使用的映像檔
docker tag <src> <repo>:<tag>    # 加標籤
docker push <repo>:<tag>         # 推送至 registry
```

## 容器 (Container)

```bash
docker run -d --name <name> -p 8080:80 <image>   # 背景執行並映射連接埠
docker run -it --rm <image> bash                 # 互動模式，結束後自動刪除
docker run -v $(pwd):/app -e KEY=val <image>     # 掛載 volume、設定環境變數
docker ps                        # 執行中的容器
docker ps -a                     # 所有容器
docker stop|start|restart <c>    # 停止／啟動／重啟
docker rm <c>                    # 刪除容器（-f 強制）
docker container prune           # 清除已停止的容器
```

## 除錯與檢視

```bash
docker logs -f --tail 100 <c>    # 追蹤日誌
docker exec -it <c> sh           # 進入執行中的容器
docker inspect <c>               # 詳細設定（JSON）
docker stats                     # 資源使用量
docker top <c>                   # 容器內程序
docker cp <c>:/path ./local      # 容器與主機間複製檔案
```

## Volume 與 Network

```bash
docker volume create|ls|rm <v>
docker network create <n>
docker network ls
docker network connect <n> <c>
```

## Docker Compose (v2)

```bash
docker compose up -d --build     # 建置並背景啟動
docker compose down -v           # 停止並移除（含 volume）
docker compose ps
docker compose logs -f <service>
docker compose exec <service> sh
docker compose config            # 驗證並顯示最終設定
```

## 清理

```bash
docker system df                 # 磁碟用量
docker system prune -a --volumes # 清除所有未使用資源（謹慎使用）
```

需要 Dockerfile 最佳實務、多階段建置，或 Cloud Run 部署相關指令，可以再說。