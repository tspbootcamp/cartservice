# Build stage
FROM mcr.microsoft.com/dotnet/sdk:7.0.102 AS builder
WORKDIR /app

# Copy csproj and restore
COPY src/cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-musl-x64

# Copy the rest of the source code
COPY src/. .

# Publish the app
RUN dotnet publish cartservice.csproj \
    -p:PublishSingleFile=true \
    -r linux-musl-x64 \
    --self-contained true \
    -p:PublishTrimmed=True \
    -p:TrimMode=Link \
    -c Release \
    -o /cartservice \
    --no-restore

# Runtime stage (no gRPC health probe yet)
FROM mcr.microsoft.com/dotnet/runtime-deps:7.0.2-alpine3.16-amd64 AS base
WORKDIR /app
COPY --from=builder /cartservice .
EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070
USER 1000
ENTRYPOINT ["/app/cartservice"]
