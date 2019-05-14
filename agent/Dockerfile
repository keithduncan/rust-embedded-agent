FROM rust:1.34

RUN rustup toolchain install nightly

RUN rustup target add --toolchain stable thumbv6m-none-eabi && \
	rustup target add --toolchain stable thumbv7em-none-eabihf && \
	rustup target add --toolchain nightly thumbv6m-none-eabi && \
	rustup target add --toolchain nightly thumbv7em-none-eabihf

RUN apt-get update && \
	apt-get install -y \
      apt-transport-https

COPY ["buildkite-agent.list", "/etc/apt/sources.list.d/"]
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 32A37959C2FA5C3C99EFBC32A79206696452D198 && \
    apt-get update -o Dir::Etc::sourcelist="sources.list.d/buildkite-agent.list" -o Dir::Etc::sourceparts="-" -o APT::Get::List-Cleanup="0" && \
    apt-get install -y buildkite-agent
ENV BUILDKITE_BUILD_PATH=/var/lib/buildkite-agent/builds

COPY ["ssh", "/root/.ssh"]

RUN cargo install --git https://github.com/keithduncan/travis-pipeline --tag v0.2

LABEL buildkite.metadata=rust=stable,rust=nightly,rust:version=1.34,rust:embedded,rust:target=thumbv6m-none-eabi,rust:target=thumbv7em-none-eabihf,travis-pipeline,travis-pipeline:version=0.2

ENTRYPOINT ["/usr/bin/buildkite-agent"]
CMD ["start"]
