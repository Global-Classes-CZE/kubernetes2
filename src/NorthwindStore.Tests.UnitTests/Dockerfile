FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:6.0 AS build
ARG configuration=Release
WORKDIR /src
COPY ["packages/", "packages/"]
COPY ["nuget.config", "."]
COPY ["NorthwindStore.Tests.UnitTests/NorthwindStore.Tests.UnitTests.csproj", "NorthwindStore.Tests.UnitTests/"]
COPY ["NorthwindStore.DAL/NorthwindStore.DAL.csproj", "NorthwindStore.DAL/"]
COPY ["NorthwindStore.BL/NorthwindStore.BL.csproj", "NorthwindStore.BL/"]
RUN dotnet restore "NorthwindStore.Tests.UnitTests/NorthwindStore.Tests.UnitTests.csproj"
COPY . .
WORKDIR "/src/NorthwindStore.Tests.UnitTests"
RUN dotnet build "NorthwindStore.Tests.UnitTests.csproj"

ENTRYPOINT ["dotnet", "test", "NorthwindStore.Tests.UnitTests.csproj", "--logger", "trx;LogFileName=/src/testresults/unittests-results.trx"]
