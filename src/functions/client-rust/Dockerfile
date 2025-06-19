ARG RUST_VERSION=1.85.0
FROM lukemathwalker/cargo-chef:latest-rust-${RUST_VERSION} AS chef

# Add cross-compilation targets
RUN set -eux; \
    rustup toolchain install stable; \
    rustup target install x86_64-unknown-linux-gnu aarch64-unknown-linux-gnu;

# Install Zig for cross-compilation
ARG ZIG_VERSION=0.14.0
RUN curl -fsSL "https://ziglang.org/download/${ZIG_VERSION}/zig-linux-$(uname -m)-${ZIG_VERSION}.tar.xz" | \
    tar -xJ -C /opt && mv /opt/zig-linux-* /opt/zig
ENV PATH="/opt/zig:${PATH}"

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
RUN cargo lambda build --release --arm64 --package client-rust

# Export stage - copy the built binary to a scratch image
FROM scratch AS export
COPY --from=builder /app/target/lambda/client-rust/bootstrap /bootstrap