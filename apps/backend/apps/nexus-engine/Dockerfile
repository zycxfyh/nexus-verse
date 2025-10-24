# Dockerfile for Individual NestJS Service (V2 - Simplified Paths)

# --- Stage 1: Builder ---
# 使用一个完整的 Node 镜像，以便 pnpm 能处理二进制依赖
FROM node:20 AS builder
WORKDIR /app

# 设置 pnpm
RUN npm install -g pnpm@9.6.0

# 关键：我们只复制构建依赖所需的最少文件
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json tsconfig.json ./

# 拷贝各个工作区的 package.json
COPY apps/frontend/package.json ./apps/frontend/
COPY apps/backend/apps/creation-agent/package.json ./apps/backend/apps/creation-agent/
COPY apps/backend/apps/logic-agent/package.json ./apps/backend/apps/logic-agent/
COPY apps/backend/apps/narrative-agent/package.json ./apps/backend/apps/narrative-agent/
COPY apps/backend/apps/nexus-engine/package.json ./apps/backend/apps/nexus-engine/
# 如果有 packages/* 目录，也需要像上面一样拷贝
# COPY packages/shared-types/package.json ./packages/shared-types/

# 安装所有生产环境的依赖。--filter . 确保只安装实际用到的
RUN pnpm install --prod --frozen-lockfile --filter .

# 现在，拷贝所有源代码
COPY . .

# 重新安装所有依赖（包括 devDependencies）以进行构建
RUN pnpm install --frozen-lockfile

# 只构建当前服务及其依赖
# ARG SERVICE_NAME 会由 Earthfile 传入
ARG SERVICE_NAME
RUN pnpm turbo run build --filter=${SERVICE_NAME}...

# 从完整构建中提取纯净的生产部署包
RUN pnpm --filter=${SERVICE_NAME} deploy /deploy

# --- Stage 2: Final Image ---
FROM node:20-slim
WORKDIR /app
ENV NODE_ENV=production

# 从 builder 阶段拷贝最终产物
COPY --from=builder /deploy .

CMD ["node", "dist/main.js"]