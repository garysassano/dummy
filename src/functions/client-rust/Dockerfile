# Use the official Rust image as the base
FROM --platform=$BUILDPLATFORM rust:latest AS builder

# Install cargo-lambda for building AWS Lambda functions
RUN cargo install cargo-lambda

# Set the working directory
WORKDIR /app

# Copy the Cargo.toml and Cargo.lock files
COPY Cargo.toml ./

# Copy the source code
COPY src ./src

# Build the Lambda function for ARM64
RUN cargo lambda build --release --arm64

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap
