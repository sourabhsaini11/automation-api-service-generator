# Build stage
FROM golang:1.25.5 AS builder

WORKDIR /app

# Copy main onix project
COPY automation-beckn-onix/go.mod automation-beckn-onix/go.sum ./
RUN go mod download

COPY automation-beckn-onix/ .

# Copy temp directory with plugins and validation packages
COPY temp/ ../temp/

# Build the plugins first
WORKDIR /temp/automation-beckn-plugins
RUN chmod +x buildplugins.sh && ./buildplugins.sh /app/plugins

# Build the server (statically linked for alpine)
WORKDIR /app
RUN CGO_ENABLED=1 go build -o server cmd/adapter/main.go

# Runtime stage
FROM debian:bookworm-slim

WORKDIR /app

# Install ca-certificates for HTTPS
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates && rm -rf /var/lib/apt/lists/*

# Copy the binary from builder
COPY --from=builder /app/server .

# Copy .env file
COPY --from=builder /app/.env .env

# Copy plugins directory
COPY --from=builder /app/plugins ./plugins

# Copy schemas directory
COPY --from=builder /app/schemas ./schemas

# Copy temp config and validationpkg
COPY --from=builder /temp/config ./config

# Expose port (adjust if needed)
EXPOSE 8080

# Run the server with config from temp/config
CMD ["./server", "--config=./config/adapter.yaml"]
