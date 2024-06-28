FROM node:12-alpine
WORKDIR /usr/src/app
COPY . ./
RUN yarn --frozen-lockfile && yarn build
EXPOSE 3000
CMD yarn start
