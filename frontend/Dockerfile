FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# CRA embute REACT_APP_* no bundle em tempo de build. Deixamos vazio por padrão
# para que o front end faça chamadas relativas (ex.: /sales), que o Nginx abaixo
# repassa para o serviço "backend" — assim a imagem não fica presa a um host/porta
# fixo. Pode ser sobrescrito via --build-arg se necessário.
ARG REACT_APP_BACKEND_URL=""
ENV REACT_APP_BACKEND_URL=${REACT_APP_BACKEND_URL}

RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
