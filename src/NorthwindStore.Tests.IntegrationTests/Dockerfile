# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["packages/", "packages/"]
COPY ["nuget.config", "."]
COPY ["NorthwindStore.Tests.IntegrationTests/NorthwindStore.Tests.IntegrationTests.csproj", "NorthwindStore.Tests.IntegrationTests/"]
COPY ["NorthwindStore.DAL/NorthwindStore.DAL.csproj", "NorthwindStore.DAL/"]
COPY ["NorthwindStore.BL/NorthwindStore.BL.csproj", "NorthwindStore.BL/"]
RUN dotnet restore "NorthwindStore.Tests.IntegrationTests/NorthwindStore.Tests.IntegrationTests.csproj"
COPY . .
WORKDIR "/src/NorthwindStore.Tests.IntegrationTests"
RUN dotnet build "NorthwindStore.Tests.IntegrationTests.csproj"

ENTRYPOINT ["dotnet", "test", "NorthwindStore.Tests.IntegrationTests.csproj", "--logger", "trx;LogFileName=/src/testresults/integrationtests-results.trx"]
