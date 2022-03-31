# Initialize a new build stage and set the Base Image 
FROM mcr.microsoft.com/dotnet/aspnet:6.0-focal AS base
# Set the working directory
WORKDIR /app
# Expose the port 5000
EXPOSE 5000
# Sets the environment variable
ENV ASPNETCORE_URLS=http://+:5000

# Creates a non-root user with an explicit UID and adds a permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-dotnet-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

# Initialize the new build stage and set the Build Image
FROM mcr.microsoft.com/dotnet/sdk:6.0-focal AS build
# Set the working directory for subsequent COPY and RUN commands
WORKDIR /src
# Copy the project file to the container filesystem /src folder
COPY ["src/sample-app/SampleAPI.csproj", "src/sample-app/"]
# Execute dotnet restore to restore dependencies specified in the .csproj file
RUN dotnet restore "src/sample-app/SampleAPI.csproj"
# Copy everything to the container filesystem
COPY . .
# Set the working directory for subsequent RUN commands
WORKDIR "/src/src/sample-app"
# Execute project build with all of its dependencies
RUN dotnet build "SampleAPI.csproj" -c Release -o /app/build

# Pick up where a previous build stage left off
FROM build AS publish
# Publish the application and its dependencies to a folder for deployment
# UseAppHost=false to disable the generation of the native executable.
RUN dotnet publish "SampleAPI.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Pick up where a previous base stage left off
FROM base AS final
# Set the working directory for the subsequent COPY command
WORKDIR /app
# Copy from the publish image /app/publish directory to the container filesystem
COPY --from=publish /app/publish .
# Specify the command executed when the container is started
ENTRYPOINT ["dotnet", "SampleAPI.dll"]