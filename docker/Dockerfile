FROM node:18
WORKDIR /app
COPY ../app-code/package.json .
RUN npm install
COPY ../app-code .
CMD ["npm", "start"]
