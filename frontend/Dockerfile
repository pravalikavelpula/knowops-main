FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

EXPOSE 5173

# The "dev" script already binds port 5173 on 0.0.0.0.
CMD ["npm", "run", "dev"]
