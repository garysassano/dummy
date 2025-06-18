# Use the official Rust image as the base
FROM rust:alpine AS chef

# Install build dependencies
RUN apk add --no-cache musl-dev

# Install cargo-chef for dependency caching
RUN cargo install cargo-chef

# Install Zig for cross-compilation (recommended way)
RUN apk add --no-cache wget xz && \
    wget -q https://ziglang.org/download/0.13.0/zig-linux-x86_64-0.13.0.tar.xz && \
    tar -xf zig-linux-x86_64-0.13.0.tar.xz -C /opt && \
    rm zig-linux-x86_64-0.13.0.tar.xz

# Add Zig to PATH
ENV PATH="/opt/zig-linux-x86_64-0.13.0:${PATH}"

# Install cargo-lambda for building AWS Lambda functions
RUN cargo install cargo-lambda

# Add ARM64 target for cross-compilation
RUN rustup target add aarch64-unknown-linux-gnu

WORKDIR /app

# Planner stage - create recipe for dependencies
FROM chef AS planner
COPY Cargo.toml ./
COPY Cargo.lock ./
COPY src/functions/client-rust/Cargo.toml ./src/functions/client-rust/
# Create dummy source files for workspace structure
RUN mkdir -p src/functions/client-rust/src && \
    echo "fn main() {}" > src/functions/client-rust/src/main.rs
RUN cargo chef prepare --recipe-path recipe.json

# Cacher stage - build dependencies only
FROM chef AS cacher
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json

# Builder stage - build the actual application
FROM chef AS builder
COPY --from=cacher /app/target target
COPY Cargo.toml ./
COPY Cargo.lock ./
COPY src/functions/client-rust/Cargo.toml ./src/functions/client-rust/
COPY src/functions/client-rust/src ./src/functions/client-rust/src

# Build the Lambda function for ARM64
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap
