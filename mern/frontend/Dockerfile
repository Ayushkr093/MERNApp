FROM node:18.9.1 AS deps

WORKDIR /app

COPY package.json ./

RUN npm install --legacy-peer-deps

FROM node:18.9.1 AS dev

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/package.json ./package.json

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev"]
