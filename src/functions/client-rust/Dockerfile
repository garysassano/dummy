# Use the pre-built cargo-chef image with Rust
ARG RUST_VERSION=1.85.0
FROM lukemathwalker/cargo-chef:latest-rust-${RUST_VERSION} AS chef

# Add cross-compilation targets
RUN set -eux; \
    rustup toolchain install stable; \
    rustup target install x86_64-unknown-linux-gnu aarch64-unknown-linux-gnu;

# Install Zig for cross-compilation
RUN echo "Architecture: $(uname -m)" && \
    curl -fsSL "https://ziglang.org/download/0.13.0/zig-linux-$(uname -m)-0.13.0.tar.xz" | \
    tar -xJ --strip-components=1 --wildcards -C /usr/local/bin '*/zig' && \
    ls -la /usr/local/bin/zig* && \
    zig version

# Install cargo-lambda (pre-built binary)
ARG CARGO_LAMBDA_VERSION=v1.8.5
RUN curl -fsSL "https://github.com/cargo-lambda/cargo-lambda/releases/download/${CARGO_LAMBDA_VERSION}/cargo-lambda-${CARGO_LAMBDA_VERSION}.$(uname -m)-unknown-linux-musl.tar.gz" | \
    tar -xz -C /usr/local/bin

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
RUN echo "=== DEBUG BUILDER STAGE ===" && \
    echo "Architecture: $(uname -m)" && \
    echo "PATH: $PATH" && \
    ls -la /usr/local/bin/ && \
    echo "=== END DEBUG ==="
RUN which zig && zig version
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap

