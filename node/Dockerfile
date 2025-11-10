FROM node:22.21.1-bookworm

RUN apt update && \
    apt install -y git curl jq unzip groff less && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

CMD ["node"]

