FROM node:24.21.0-trixie

RUN apt update && \
    apt install -y git curl jq unzip groff less && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

CMD ["node"]
