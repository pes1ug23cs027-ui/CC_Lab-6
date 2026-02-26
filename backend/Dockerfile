FROM ubuntu:22.04

WORKDIR /app

# Install dependencies
RUN apt-get update && apt-get install -y g++ python3

# Copy source
COPY app.cpp .

# Compile the app (optional if you want the binary)
RUN g++ app.cpp -o app

# Expose port 8080
EXPOSE 8080

# Start a simple HTTP server on 8080
CMD ["python3", "-m", "http.server", "8080"]
