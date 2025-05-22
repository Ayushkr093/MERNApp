# Use a specific LTS version of Node.js for stability
FROM node:lts

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first for optimized layer caching
COPY package.json package-lock.json ./

# Install dependencies with the --legacy-peer-deps flag
RUN npm install --legacy-peer-deps

# Copy the rest of the application files
COPY . .

# Expose the port the app runs on
EXPOSE 5050

# Add a non-root user to run the application for security
RUN useradd -m appuser
USER appuser

# Command to run the application
CMD ["npm", "start"]
