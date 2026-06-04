# Build hyperlight-unikraft VMM (static musl binary)
FROM rust:1.89-bookworm AS vmm-builder
RUN rustup target add x86_64-unknown-linux-musl
RUN apt-get update && apt-get install -y musl-tools && rm -rf /var/lib/apt/lists/*
WORKDIR /build
RUN git clone https://github.com/hyperlight-dev/hyperlight-unikraft.git . \
    && cd host \
    && cargo build --release --target x86_64-unknown-linux-musl --bin hyperlight-unikraft

# Grab the pre-built unikernel kernel + initrd from the published GHCR image
FROM ghcr.io/hyperlight-dev/hyperlight-unikraft/hello-hyperlight-unikraft:latest AS unikernel

# Final image — just the VMM binary + unikernel payload
FROM scratch
COPY --from=vmm-builder /build/host/target/x86_64-unknown-linux-musl/release/hyperlight-unikraft /app/hyperlight-unikraft
COPY --from=unikernel /unikernel/kernel /app/unikernel/kernel
COPY --from=unikernel /unikernel/initrd.cpio /app/unikernel/initrd.cpio
ENTRYPOINT ["/app/hyperlight-unikraft", "/app/unikernel/kernel", "--initrd", "/app/unikernel/initrd.cpio"]
