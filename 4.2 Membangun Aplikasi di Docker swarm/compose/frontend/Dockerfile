FROM node:16

WORKDIR /usr/src/frontend

COPY package*.json ./

# Install all dependencies (including devDependencies) for build
RUN npm install

# Install sharp explicitly (required by Next.js for image optimization)
RUN npm install sharp

COPY . .

# Build the application
RUN npm run build

EXPOSE 3000

CMD [ "npm", "start" ]
