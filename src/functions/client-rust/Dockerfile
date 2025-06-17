# Use the official Rust image as the base
FROM --platform=$BUILDPLATFORM rust:latest AS builder

# Install Zig for cross-compilation (recommended way)
RUN apt-get update && apt-get install -y wget xz-utils && \
    wget -q https://ziglang.org/download/0.13.0/zig-linux-x86_64-0.13.0.tar.xz && \
    tar -xf zig-linux-x86_64-0.13.0.tar.xz -C /opt && \
    rm zig-linux-x86_64-0.13.0.tar.xz && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Add Zig to PATH
ENV PATH="/opt/zig-linux-x86_64-0.13.0:${PATH}"

# Install cargo-lambda for building AWS Lambda functions
RUN cargo install cargo-lambda

# Add ARM64 target for cross-compilation
RUN rustup target add aarch64-unknown-linux-gnu

# Set the working directory
WORKDIR /app

# Copy the workspace root files first
COPY Cargo.toml ./
COPY Cargo.lock ./

# Create the directory structure for the workspace member
RUN mkdir -p src/functions/client-rust

# Copy the function's Cargo.toml and source code
COPY src/functions/client-rust/Cargo.toml ./src/functions/client-rust/
COPY src/functions/client-rust/src ./src/functions/client-rust/src

# Build the Lambda function for ARM64
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap
