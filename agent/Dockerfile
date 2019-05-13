FROM rust:1.34

RUN rustup target add thumbv6m-none-eabi && \
	rustup target add thumbv7em-none-eabihf

RUN apt-get update && \
	apt-get install -y \
      apt-transport-https

COPY ["buildkite-agent.list", "/etc/apt/sources.list.d/"]
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 32A37959C2FA5C3C99EFBC32A79206696452D198 && \
    apt-get update -o Dir::Etc::sourcelist="sources.list.d/buildkite-agent.list" -o Dir::Etc::sourceparts="-" -o APT::Get::List-Cleanup="0" && \
    apt-get -y install buildkite-agent

COPY ["ssh", "/root/.ssh"]

ENV BUILDKITE_BUILD_PATH=/var/lib/buildkite-agent/builds

LABEL buildkite.metadata=rust=1.34,rust-embedded

ENTRYPOINT ["/usr/bin/buildkite-agent"]
CMD ["start"]
