# ---------- STAGE 1: Build & Publish the app ----------
FROM mcr.microsoft.com/dotnet/sdk:7.0.102 AS builder

WORKDIR /app

# Copy only csproj first to leverage Docker cache
COPY cartservice.csproj .

# Restore dependencies
RUN dotnet restore cartservice.csproj -r linux-musl-x64

# Copy everything else
COPY . .

# Publish the app as a self-contained, single file binary
RUN dotnet publish cartservice.csproj \
    -p:PublishSingleFile=true \
    -r linux-musl-x64 \
    --self-contained true \
    -p:PublishTrimmed=True \
    -p:TrimMode=Link \
    -c Release \
    -o /cartservice \
    --no-restore


# ---------- STAGE 2: Runtime ----------
FROM mcr.microsoft.com/dotnet/runtime-deps:7.0.2-alpine3.16-amd64 AS runtime

WORKDIR /app

# Copy the published output from the builder
COPY --from=builder /cartservice .

# Port and environment setup
EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070

# Use non-root user
USER 1000

ENTRYPOINT ["/app/cartservice"]


# ---------- STAGE 3: Add Health Probe ----------
FROM runtime AS final

USER root

ENV GRPC_HEALTH_PROBE_VERSION=v0.4.14

# Download and install grpc_health_probe binary
RUN wget -qO /bin/grpc_health_probe \
      https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

USER 1000
