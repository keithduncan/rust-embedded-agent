FROM buildpack-deps:stretch

RUN apt-get update && \
	apt-get install -y \
      apt-transport-https \
      python-pip

COPY ["buildkite-agent.list", "/etc/apt/sources.list.d/"]
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 32A37959C2FA5C3C99EFBC32A79206696452D198 && \
    apt-get update -o Dir::Etc::sourcelist="sources.list.d/buildkite-agent.list" -o Dir::Etc::sourceparts="-" -o APT::Get::List-Cleanup="0" && \
    apt-get install -y buildkite-agent
ENV BUILDKITE_BUILD_PATH=/var/lib/buildkite-agent/builds

COPY ["ssh", "/root/.ssh"]

RUN pip install awscli~=1.16 --upgrade

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
        rustup target remove --toolchain $toolchain x86_64-unknown-linux-gnu || true; \
    done

ENTRYPOINT ["/usr/bin/buildkite-agent"]
CMD ["start"]
