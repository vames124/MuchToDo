
# STAGE 1: Build the application

# We start with an official Go image based on Alpine Linux (a very tiny Linux version)
FROM golang:1.25-alpine AS builder

# We set the working directory inside the container
WORKDIR /app

# COPY DEPENDENCIES FIRST: We copy go.mod and go.sum before our actual code.

COPY go.mod go.sum ./
RUN go mod download

# copy the rest of our application code into the container
COPY . .

# We compile the Go application. 

RUN CGO_ENABLED=0 GOOS=linux go build -o muchtodo-backend ./cmd/api/main.go



# STAGE 2: Create a minimal, secure runtime image

# start entirely fresh with just a barebones Alpine Linux image
FROM alpine:3.19

# Set the working directory for the runtime container
WORKDIR /app

# SECURITY: We create a non-root user. Running apps as the 'root' user inside a 
# container is a security risk. If a hacker breaches the app, they have root access.
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# We copy ONLY the compiled binary from the "builder" stage above. 

COPY --from=builder /app/muchtodo-backend .

# Give our new safe user ownership of the app
RUN chown -R appuser:appgroup /app

# Tell Docker to run all subsequent commands as this safe user
USER appuser

# Expose port 8080 
EXPOSE 8080

CMD ["./muchtodo-backend"]