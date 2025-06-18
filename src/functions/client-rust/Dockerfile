# Use the pre-built cargo-chef image with Rust (Debian-based for better tooling)
FROM lukemathwalker/cargo-chef:latest-rust-1 AS chef

# Add ARM64 target for cross-compilation
RUN rustup target add aarch64-unknown-linux-gnu

# Install cargo-lambda
RUN curl -fsSL https://cargo-lambda.info/install.sh | sh

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
