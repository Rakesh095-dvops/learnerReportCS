
FROM node:20

# Working directory be app
WORKDIR /usr/src/app

COPY package*.json ./


###  Installing dependencies (include dev dependencies for msw) --omit=dev 
RUN npm ci --legacy-peer-deps

### Installing dependencies
#RUN npm install

# copy local files to app folder
COPY . .

EXPOSE 3000

CMD ["npm","start"]
