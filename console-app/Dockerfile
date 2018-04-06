FROM microsoft/dotnet:2.0-sdk AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build -c Release
RUN dotnet publish -c Release -o /app

FROM microsoft/dotnet:2.0-runtime AS final
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "console-app.dll"]