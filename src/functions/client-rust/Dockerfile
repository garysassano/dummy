# Use the pre-built cargo-chef image with Rust
FROM lukemathwalker/cargo-chef:latest-rust-alpine AS chef

# Add ARM64 target for cross-compilation
RUN rustup target add aarch64-unknown-linux-gnu

# Install cargo-lambda (pre-built binary)
ARG CARGO_LAMBDA_VERSION=v1.8.5
RUN wget "https://github.com/cargo-lambda/cargo-lambda/releases/download/${CARGO_LAMBDA_VERSION}/cargo-lambda-${CARGO_LAMBDA_VERSION}.x86_64-unknown-linux-musl.tar.gz" && \
    tar -xzf "cargo-lambda-${CARGO_LAMBDA_VERSION}.x86_64-unknown-linux-musl.tar.gz" && \
    mv cargo-lambda /usr/local/bin/ && \
    rm "cargo-lambda-${CARGO_LAMBDA_VERSION}.x86_64-unknown-linux-musl.tar.gz"

# Install Zig
ARG ZIG_VERSION=0.14.1
RUN wget "https://ziglang.org/download/${ZIG_VERSION}/zig-linux-x86_64-${ZIG_VERSION}.tar.xz" && \
    tar -xf "zig-linux-x86_64-${ZIG_VERSION}.tar.xz" && \
    mv "zig-linux-x86_64-${ZIG_VERSION}" /usr/local/zig && \
    rm "zig-linux-x86_64-${ZIG_VERSION}.tar.xz"

ENV PATH="/usr/local/zig:${PATH}"

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
