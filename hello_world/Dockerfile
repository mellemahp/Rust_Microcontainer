FROM rustlang/rust:nightly-slim AS build
RUN apt-get update; apt-get install -y musl-tools
RUN rustup target install x86_64-unknown-linux-musl
COPY . /hello_world/
WORKDIR /hello_world
RUN cargo build --release --target=x86_64-unknown-linux-musl

FROM scratch AS deploy
COPY --from=build /hello_world/target/x86_64-unknown-linux-musl/release/hello_world /
EXPOSE 8000
ENTRYPOINT ["/hello_world"]
