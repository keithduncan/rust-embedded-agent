FROM buildpack-deps:stretch

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='7d610882f67ec4e53a56a8177d7862501a043f80be7ba13b6e325cc9921f23b8' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='3bc5c5fff32113dc20284bd605eb2a6f7070de0c69c742b00b0ca4511a8fbc4c' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='cff57e155046439896004d7eb66fcbebe436f00298b1dacef426aef0a109a866' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='2d82bf50439c6ec74af4a5642a004fe8915921b52d3ec54032e0dc10476718c1' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.18.1/${rustArch}/rustup-init"; \
    wget "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --default-toolchain none; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version;

ARG TOOLCHAIN=stable
RUN rustup toolchain install $TOOLCHAIN && \
	rustup target add --toolchain $TOOLCHAIN thumbv6m-none-eabi && \
	rustup target add --toolchain $TOOLCHAIN thumbv7em-none-eabihf && \
    rustup default $TOOLCHAIN

RUN apt-get update && \
	apt-get install -y \
      apt-transport-https

COPY ["buildkite-agent.list", "/etc/apt/sources.list.d/"]
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 32A37959C2FA5C3C99EFBC32A79206696452D198 && \
    apt-get update -o Dir::Etc::sourcelist="sources.list.d/buildkite-agent.list" -o Dir::Etc::sourceparts="-" -o APT::Get::List-Cleanup="0" && \
    apt-get install -y buildkite-agent
ENV BUILDKITE_BUILD_PATH=/var/lib/buildkite-agent/builds

COPY ["ssh", "/root/.ssh"]

RUN cargo install --git https://github.com/keithduncan/travis-pipeline --tag v0.3

LABEL buildkite.metadata=rust=$TOOLCHAIN,rust:embedded,rust:target=thumbv6m-none-eabi,rust:target=thumbv7em-none-eabihf,travis-pipeline,travis-pipeline:version=0.2

ENTRYPOINT ["/usr/bin/buildkite-agent"]
CMD ["start"]
