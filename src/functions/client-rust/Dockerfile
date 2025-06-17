# Use the official Rust image as the base
FROM --platform=$BUILDPLATFORM rust:latest AS builder

# Install cargo-lambda for building AWS Lambda functions
RUN cargo install cargo-lambda

# Set the working directory
WORKDIR /app

# Copy the workspace root files first
COPY ../../Cargo.toml ./
COPY ../../Cargo.lock ./

# Create the directory structure for the workspace member
RUN mkdir -p src/functions/client-rust

# Copy the function's Cargo.toml and source code
COPY Cargo.toml ./src/functions/client-rust/
COPY src ./src/functions/client-rust/src

# Build the Lambda function for ARM64
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap
