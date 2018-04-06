# Restore dependencies and build the app
FROM microsoft/aspnetcore-build:2.0 AS build
WORKDIR /src
COPY *.csproj .
COPY package.json .
RUN dotnet restore
RUN npm install
COPY . .
RUN dotnet build -c Release

# Publish the app based on the build step
FROM build as publish
RUN dotnet publish -c Release -o /app

# Run tests
# FROM publish AS test
# RUN dotnet test <some test project>

# Package the published code with the dotnet runtime image
FROM microsoft/aspnetcore:2.0 AS final
WORKDIR /app
COPY --from=publish /app .
EXPOSE 80
ENTRYPOINT ["dotnet", "web-app.dll"]