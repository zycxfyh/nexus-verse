# Earthfile for Nexus Verse (V16 - The Final Commander)

VERSION 0.8

# --- 1. 后端服务 ---
# COPY . . 将整个项目作为上下文提供给 BUILD 命令
# BUILD 命令会使用指定目录下的 Dockerfile 进行构建

nexus-engine-image:
    COPY . .
    # --SERVICE_NAME 参数会传递给 Dockerfile 内部的 ARG
    BUILD --SERVICE_NAME=nexus-engine ./apps/backend/apps/nexus-engine
    SAVE IMAGE nexus-verse/nexus-engine:latest

creation-agent-image:
    COPY . .
    BUILD --SERVICE_NAME=creation-agent ./apps/backend/apps/creation-agent
    SAVE IMAGE nexus-verse/creation-agent:latest

logic-agent-image:
    COPY . .
    BUILD --SERVICE_NAME=logic-agent ./apps/backend/apps/logic-agent
    SAVE IMAGE nexus-verse/logic-agent:latest

narrative-agent-image:
    COPY . .
    BUILD --SERVICE_NAME=narrative-agent ./apps/backend/apps/narrative-agent
    SAVE IMAGE nexus-verse/narrative-agent:latest

# --- 2. 前端服务 ---
# 前端的构建逻辑比较简单，直接放在 Earthfile 里
frontend-image:
    # 阶段1: 构建器
    FROM node:20 AS builder
    WORKDIR /app
    RUN npm install -g pnpm@9.6.0
    # 精确拷贝所需文件
    COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json tsconfig.json ./
    COPY apps/frontend/package.json ./apps/frontend/
    COPY apps/backend/apps/creation-agent/package.json ./apps/backend/apps/creation-agent/
    COPY apps/backend/apps/logic-agent/package.json ./apps/backend/apps/logic-agent/
    COPY apps/backend/apps/narrative-agent/package.json ./apps/backend/apps/narrative-agent/
    COPY apps/backend/apps/nexus-engine/package.json ./apps/backend/apps/nexus-engine/
    RUN pnpm install --frozen-lockfile
    COPY . .
    RUN pnpm turbo run build --filter=frontend

    # 阶段2: 最终镜像
    FROM nginx:stable-alpine
    COPY --from=builder /app/apps/frontend/dist /usr/share/nginx/html
    COPY apps/frontend/nginx.conf /etc/nginx/conf.d/default.conf
    SAVE IMAGE nexus-verse/frontend:latest