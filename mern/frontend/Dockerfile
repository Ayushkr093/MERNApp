# Use a specific Node.js version (18.9.1) for stability
FROM node:18.9.1

# Set the working directory inside the container
WORKDIR /app

# Copy package.json first to optimize Docker cache usage
COPY package.json ./

# Install dependencies with the --legacy-peer-deps flag to handle any peer dependency issues
RUN npm install --legacy-peer-deps

# Expose the port the app runs on (5173 for Vite-based frontend)
EXPOSE 5173

# Copy the rest of the application files
COPY . .

# Start the frontend app using the "dev" script (Vite or another dev server)
CMD ["npm", "run", "dev"]
