 FROM microsoft/aspnetcore-build-nightly AS builder
 WORKDIR /source

 COPY *.csproj .
 RUN dotnet restore

 COPY . .
 RUN dotnet publish --output /app/ --configuration Release

  FROM microsoft/aspnetcore-nightly
 WORKDIR /app
 COPY --from=builder /app .
 ENTRYPOINT ["dotnet", "app.dll"]