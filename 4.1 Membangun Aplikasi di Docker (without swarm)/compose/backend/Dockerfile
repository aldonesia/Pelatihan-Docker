FROM node:18

WORKDIR /usr/src/backend

COPY package*.json ./

RUN npm install --production
RUN apt-get update && \
    apt-get install -y net-tools postgresql-client && \
    rm -rf /var/lib/apt/lists/*


COPY . .

EXPOSE 5000

CMD [ "npm", "run", "start-prod", "--", "0.0.0.0" ]
