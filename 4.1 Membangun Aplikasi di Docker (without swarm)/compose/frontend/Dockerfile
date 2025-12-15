FROM node:16

WORKDIR /usr/src/frontend

# Build arguments for environment variables
ARG NEXT_PUBLIC_API_ENDPOINT=http://localhost:8000/
ARG NEXT_PUBLIC_APP_NAME=Music App

# Set environment variables
ENV NEXT_PUBLIC_API_ENDPOINT=${NEXT_PUBLIC_API_ENDPOINT}
ENV NEXT_PUBLIC_APP_NAME=${NEXT_PUBLIC_APP_NAME}

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
