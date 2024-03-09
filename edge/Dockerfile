ARG VERSION="main"
ARG REVISION="ee7100854c9760c201c9319c76ec5278cfd52b02"

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
    && git clone --branch "${VERSION}" https://github.com/opencontainers/runc.git /go/src/runc \
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

