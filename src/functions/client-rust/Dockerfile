# Use the official Rust image as the base
FROM rust:alpine AS builder

# Install dependencies
RUN apk add --no-cache curl xz musl-dev

# Add cross-compilation targets
RUN set -eux; \
    rustup toolchain install stable; \
    rustup target install x86_64-unknown-linux-gnu aarch64-unknown-linux-gnu;

# Install Zig for cross-compilation
RUN curl -fsSL "https://ziglang.org/download/0.13.0/zig-linux-$(uname -m)-0.13.0.tar.xz" | \
    tar -xJ -C /opt && mv /opt/zig-linux-* /opt/zig
ENV PATH="/opt/zig:${PATH}"

# Install cargo-lambda (pre-built binary)
ARG CARGO_LAMBDA_VERSION=v1.8.5
RUN curl -fsSL "https://github.com/cargo-lambda/cargo-lambda/releases/download/${CARGO_LAMBDA_VERSION}/cargo-lambda-${CARGO_LAMBDA_VERSION}.$(uname -m)-unknown-linux-musl.tar.gz" | \
    tar -xz -C /usr/local/bin

# Set working directory
WORKDIR /app

# Copy workspace files
COPY Cargo.toml Cargo.lock ./
COPY src/functions/client-rust ./src/functions/client-rust

# Build the Lambda function
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap