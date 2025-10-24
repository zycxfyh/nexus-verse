# Earthfile V28 - The Final, Parallel-Optimized Version
VERSION 0.8

# =========================================
# 🟢 基础阶段：所有服务的共同祖先
# =========================================
deps:
    FROM node:20
    WORKDIR /app
    RUN npm config set registry https://registry.npmmirror.com
    RUN npm install -g pnpm@9.6.0
    RUN pnpm config set registry https://registry.npmmirror.com
    COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json tsconfig.json ./
    COPY apps/frontend/package.json ./apps/frontend/
    COPY apps/backend/apps/nexus-engine/package.json ./apps/backend/apps/nexus-engine/
    COPY apps/backend/apps/creation-agent/package.json ./apps/backend/apps/creation-agent/
    COPY apps/backend/apps/logic-agent/package.json ./apps/backend/apps/logic-agent/
    COPY apps/backend/apps/narrative-agent/package.json ./apps/backend/apps/narrative-agent/
    RUN pnpm install --frozen-lockfile
    # 关键优化：缓存 pnpm store，为完全离线构建做准备
    RUN pnpm store prune
    SAVE IMAGE +deps

# =========================================
# 🏗️ 构建阶段：源代码编译
# =========================================
builder:
    FROM +deps
    COPY . .
    SAVE IMAGE +builder

# =========================================
# 🚀 服务打包阶段 (保持不变)
# =========================================

nexus-engine-image:
    FROM +builder
    RUN pnpm turbo run build --filter=nexus-engine...
    RUN pnpm --filter=nexus-engine deploy /deploy
    SAVE ARTIFACT /deploy AS service-files
    FROM node:20-slim
    WORKDIR /app
    ENV NODE_ENV=production
    COPY +nexus-engine-image/service-files .
    CMD ["node", "dist/main.js"]
    SAVE IMAGE nexus-verse/nexus-engine:latest

creation-agent-image:
    FROM +builder
    RUN pnpm turbo run build --filter=creation-agent...
    RUN pnpm --filter=creation-agent deploy /deploy
    SAVE ARTIFACT /deploy AS service-files
    FROM node:20-slim
    WORKDIR /app
    ENV NODE_ENV=production
    COPY +creation-agent-image/service-files .
    CMD ["node", "dist/main.js"]
    SAVE IMAGE nexus-verse/creation-agent:latest

logic-agent-image:
    FROM +builder
    RUN pnpm turbo run build --filter=logic-agent...
    RUN pnpm --filter=logic-agent deploy /deploy
    SAVE ARTIFACT /deploy AS service-files
    FROM node:20-slim
    WORKDIR /app
    ENV NODE_ENV=production
    COPY +logic-agent-image/service-files .
    CMD ["node", "dist/main.js"]
    SAVE IMAGE nexus-verse/logic-agent:latest

narrative-agent-image:
    FROM +builder
    RUN pnpm turbo run build --filter=narrative-agent...
    RUN pnpm --filter=narrative-agent deploy /deploy
    SAVE ARTIFACT /deploy AS service-files
    FROM node:20-slim
    WORKDIR /app
    ENV NODE_ENV=production
    COPY +narrative-agent-image/service-files .
    CMD ["node", "dist/main.js"]
    SAVE IMAGE nexus-verse/narrative-agent:latest

frontend-image:
    FROM +builder
    RUN pnpm turbo run build --filter=frontend
    SAVE ARTIFACT /app/apps/frontend/dist AS frontend-files
    FROM nginx:stable-alpine
    COPY +frontend-image/frontend-files /usr/share/nginx/html
    COPY apps/frontend/nginx.conf /etc/nginx/conf.d/default.conf
    SAVE IMAGE nexus-verse/frontend:latest

# =========================================
# 👑 聚合目标：一键构建所有服务
# =========================================
all:
    BUILD +nexus-engine-image
    BUILD +creation-agent-image
    BUILD +logic-agent-image
    BUILD +narrative-agent-image
    BUILD +frontend-image