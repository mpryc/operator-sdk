# Build the manager binary
FROM --platform=$BUILDPLATFORM golang:1.18 as builder
ARG TARGETARCH

WORKDIR /workspace
# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum
# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
RUN go mod download

# Copy the go source
COPY . .

# Build
RUN GOOS=linux GOARCH=$TARGETARCH make build/ansible-operator

# Final image.
# TODO(asmacdo) update GH action to set this
FROM quay.io/operator-framework/ansible-operator-2.11-preview-base:master-80435707c04072729ce39b776ca9f1755f000d2e

ENV HOME=/opt/ansible \
    USER_NAME=ansible \
    USER_UID=1001

# Ensure directory permissions are properly set
RUN echo "${USER_NAME}:x:${USER_UID}:0:${USER_NAME} user:${HOME}:/sbin/nologin" >> /etc/passwd \
  && mkdir -p ${HOME}/.ansible/tmp \
  && chown -R ${USER_UID}:0 ${HOME} \
  && chmod -R ug+rwx ${HOME}

WORKDIR ${HOME}
USER ${USER_UID}

COPY --from=builder /workspace/build/ansible-operator /usr/local/bin/ansible-operator

ENTRYPOINT ["/tini", "--", "/usr/local/bin/ansible-operator", "run", "--watches-file=./watches.yaml"]
