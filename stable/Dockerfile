ARG VERSION="1.4.0"
ARG REVISION="8bd78a9977e604c4d5f67a7415d7b8b8c109cdc4"

FROM golang:alpine AS builder

ARG VERSION
ARG REVISION

ENV GOOS=linux
ENV GO111MODULE=on

RUN apk upgrade && apk --no-cache add \
	ca-certificates \
    git \
    gcc \
    make \
    musl-dev \
    libseccomp-dev \
    libseccomp-static

RUN mkdir -p /go/src/ \
    && git clone --branch "v${VERSION}" https://github.com/opencontainers/runc.git /go/src/runc \
	&& cd /go/src/runc \
    && git reset --hard "${REVISION}" \
    && make static \
	&& cp -vr runc /usr/local/bin/ \
	&& if [$TARGETPLATFORM = "linux/amd64"]; then apk add --no-cache upx && upx --lzma --best /usr/local/bin/runc; fi 

RUN addgroup runc \
    && adduser -G runc -s /bin/sh -D runc

FROM scratch

ARG VERSION
ARG REVISION

ENV RUNC_VERSION="${VERSION}" \
    RUNC_REVISION="${REVISION}"

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/local/bin/runc /usr/local/bin/runc

USER runc

ENTRYPOINT ["/usr/local/bin/runc"]
CMD ["--help"]

LABEL org.opencontainers.image.title="runc" \
    org.opencontainers.image.description="runc in Docker" \ 
    org.opencontainers.image.revision="${REVISION}" \
    org.opencontainers.image.version="${VERSION}"

