# ==========================================
# Stage 1: Build & Publish
# ==========================================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /app

# Copy project manifest file first for optimal NuGet layer caching
COPY payment-service.csproj ./

# Restore dependencies for linux-x64 with ReadyToRun enabled upfront
RUN dotnet restore -r linux-x64 /p:PublishReadyToRun=true

# Copy the rest of the application source code
COPY . .

# Compile and publish in Release mode for linux-x64
RUN dotnet publish payment-service.csproj \
    -c Release \
    -o /app/publish \
    --no-restore \
    -r linux-x64 \
    --self-contained false \
    /p:PublishReadyToRun=true

# ==========================================
# Stage 2: Production Runtime
# ==========================================
FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

# Copy compiled binaries and assets from the builder stage
COPY --from=builder /app/publish .

# Run under Microsoft's built-in secure non-root user ($APP_UID)
USER $APP_UID

# Expose port 8085 matching your configuration
EXPOSE 8085

# Bind ASP.NET Core explicitly to port 8085 across all network interfaces
ENV ASPNETCORE_URLS=http://0.0.0.0:8085

# Execute the compiled web service binary
ENTRYPOINT ["dotnet", "payment-service.dll"]