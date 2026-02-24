FROM node:18-alpine AS build

WORKDIR /app

COPY package.json ./
COPY package-lock.json ./

RUN npm install

COPY . .

RUN npm run build

# EXPOSE 4200

# CMD ["ng", "serve", "--host", "0.0.0.0", "--port", "4200", "--disable-host-check"]

FROM nginx:alpine

COPY --from=build /app/dist/* /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
