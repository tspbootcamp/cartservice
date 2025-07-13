# ---------- STAGE 1: Build & Publish ----------
FROM mcr.microsoft.com/dotnet/sdk:7.0.102 AS builder

WORKDIR /app

COPY src/cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-musl-x64

COPY src/. .
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
COPY --from=builder /cartservice .

EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070

USER 1000
ENTRYPOINT ["/app/cartservice"]

# ---------- STAGE 3: Health Probe ----------
FROM runtime AS final

USER root
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.14
RUN wget -qO /bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

USER 1000
