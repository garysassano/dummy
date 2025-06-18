# Use the official Rust image as the base
FROM rust:latest AS builder

# Add cross-compilation targets
RUN set -eux; \
    rustup toolchain install stable; \
    rustup target install x86_64-unknown-linux-gnu aarch64-unknown-linux-gnu;

# Install Zig for cross-compilation
RUN apt-get update && apt-get install -y --no-install-recommends xz-utils && \
    curl -fsSL "https://ziglang.org/download/0.13.0/zig-linux-$(uname -m)-0.13.0.tar.xz" | \
    tar -xJ -C /opt && mv /opt/zig-linux-* /opt/zig && \
    apt-get purge -y xz-utils && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/*
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