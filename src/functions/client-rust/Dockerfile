ARG RUST_VERSION=1.85.0
FROM rust:${RUST_VERSION} AS builder

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