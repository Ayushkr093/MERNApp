FROM node:lts AS deps

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install --legacy-peer-deps

FROM node:lts AS app

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/package.json ./package.json

COPY . .

EXPOSE 5050

RUN useradd -m appuser
USER appuser

CMD ["npm", "start"]
