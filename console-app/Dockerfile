# Restore dependencies and build the app
FROM microsoft/dotnet:2.0-sdk AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build -c Release

# Publish the app based on the build step
FROM build as publish
RUN dotnet publish -c Release -o /app

# Run tests
# FROM publish AS test
# RUN dotnet test <some test project>

# Package the published code with the dotnet runtime image
FROM microsoft/dotnet:2.0-runtime AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "console-app.dll"]