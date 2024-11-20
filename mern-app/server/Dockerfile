FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
ENV MONGO_URI=your_mongo_uri_here
CMD ["npm", "start"]







