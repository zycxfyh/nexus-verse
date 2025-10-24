# 阶段1：构建阶段
FROM node:20 AS builder
WORKDIR /app

# 复制项目依赖
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json tsconfig.json ./
COPY apps/backend/apps/nexus-engine/package.json ./apps/backend/apps/nexus-engine/
COPY apps/frontend/package.json ./apps/frontend/
COPY apps/backend/apps/creation-agent/package.json ./apps/backend/apps/creation-agent/
COPY apps/backend/apps/logic-agent/package.json ./apps/backend/apps/logic-agent/
COPY apps/backend/apps/narrative-agent/package.json ./apps/backend/apps/narrative-agent/

# 安装依赖并构建
RUN npm install -g pnpm@9.6.0
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm turbo run build --filter=nexus-engine...

# 阶段2：运行阶段
FROM node:20-slim
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/apps/backend/apps/nexus-engine/dist ./dist
CMD ["node", "dist/main.js"]
