FROM node:18-alpine

# Working directory be app
WORKDIR /usr/src/app

COPY package*.json package-lock.json ./

###  Installing dependencies (include dev dependencies for msw) --omit=dev 
RUN npm ci --legacy-peer-deps

# copy local files to app folder
COPY . .

EXPOSE 3000

CMD ["npm","start"]
