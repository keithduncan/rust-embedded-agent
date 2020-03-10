FROM debian:buster-slim

RUN apt-get update && \
	apt-get install -y \
      git-core \
      python-pip \
      wget && \
    rm -rf /var/lib/apt/lists/*

COPY ["ssh", "/root/.ssh"]

ARG AWS_CLI=1.16
RUN pip install awscli~="$AWS_CLI" --upgrade

# Copied from https://github.com/rust-lang/docker-rust

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH
RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='e68f193542c68ce83c449809d2cad262cc2bbb99640eb47c58fc1dc58cc30add' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='7c1c329a676e50c287d8183b88f30cd6afd0be140826a9fbbc0e3d717fab34d7' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='d861cc86594776414de001b96964be645c4bfa27024052704f0976dc3aed1b18' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='89f1f797dca2e5c1d75790c3c6b7be0ee473a7f4eca9663e623a41272a358da0' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.20.2/${rustArch}/rustup-init"; \
    wget "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile=minimal --default-toolchain=none; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version;

ARG TOOLCHAIN="stable nightly"
RUN for toolchain in $TOOLCHAIN; \
    do \
        rustup toolchain install \
            --no-self-update \
            --target=thumbv6m-none-eabi \
            --target=thumbv7em-none-eabihf \
            $toolchain; \
    done

ARG COMMIT
ENV BUILDKITE_ABOUT_IMAGE "keithduncan/rust-embedded-agent\ncommit: $COMMIT\n\n\naws-cli: $AWS_CLI\nToolchain: $TOOLCHAIN\ntarget: thumbv6m-none-eabi,thumbv7em-none-eabihf"
