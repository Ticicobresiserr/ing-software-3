#---Base----
# This build create a staging docker image

FROM node:13.12.0-alpine AS builder
COPY package.json /tmp/
WORKDIR /tmp/
RUN npm install
 

FROM builder
COPY --from=builder /tmp/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD npm run start


