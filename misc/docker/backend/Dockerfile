FROM node:20

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./
RUN npm install
# For production RUN npm ci --only=production
RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 3001

#Define the command
CMD [ "node", "index.js" ]