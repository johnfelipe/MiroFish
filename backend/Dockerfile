# Backend-only production image (Flask API)
# Build context must be the repository root so that ../locales is available.
FROM python:3.11

# 从 uv 官方镜像复制 uv
COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

WORKDIR /app/backend

# 先复制依赖描述文件以利用缓存
COPY backend/pyproject.toml backend/uv.lock ./

# 安装 Python 依赖（不含开发依赖）
RUN uv sync --frozen --no-dev

# 复制后端源码与共享的多语言文件
COPY backend/ ./
COPY locales /app/locales

ENV PATH="/app/backend/.venv/bin:$PATH"
ENV FLASK_HOST=0.0.0.0
ENV FLASK_PORT=5001
ENV FLASK_DEBUG=false

EXPOSE 5001

CMD ["python", "run.py"]
