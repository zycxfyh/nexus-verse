# 阶段1：构建前端
FROM node:20 AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json tsconfig.json ./
COPY apps/frontend/package.json ./apps/frontend/
RUN npm install -g pnpm@9.6.0
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm turbo run build --filter=frontend

# 阶段2：部署到 nginx
FROM nginx:stable-alpine
COPY --from=builder /app/apps/frontend/dist /usr/share/nginx/html
COPY apps/frontend/nginx.conf /etc/nginx/conf.d/default.conf
