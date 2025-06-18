# Use the pre-built cargo-chef image with Rust
FROM lukemathwalker/cargo-chef:latest-rust-alpine AS chef

# Add ARM64 target for cross-compilation
RUN rustup target add aarch64-unknown-linux-gnu

# Install cargo-lambda (pre-built binary)
ARG CARGO_LAMBDA_VERSION=v1.8.5
RUN apk add --no-cache wget && \
    wget -O- "https://github.com/cargo-lambda/cargo-lambda/releases/download/${CARGO_LAMBDA_VERSION}/cargo-lambda-${CARGO_LAMBDA_VERSION}.x86_64-unknown-linux-musl.tar.gz" | \
    tar -xz -C /usr/local/bin && \
    rm -rf /var/cache/apk/*

# Install Zig (lightweight version for cargo-zigbuild)
ARG ZIGVER=0.14.0
RUN apk add --no-cache curl xz && \
    curl -L "https://ziglang.org/download/${ZIGVER}/zig-linux-$(uname -m)-${ZIGVER}.tar.xz" | \
    tar -xJ -C /usr/local --strip-components=1 && \
    rm -rf /var/cache/apk/*

WORKDIR /app

# Planner stage - create recipe for dependencies
FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

# Builder stage - build dependencies and application
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
# Build dependencies - this is the caching Docker layer!
RUN cargo chef cook --release --recipe-path recipe.json
# Build application
COPY . .
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap
