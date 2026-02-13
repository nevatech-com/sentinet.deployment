Nevatech maintains a private Docker Hub repository (https://hub.docker.com/repositories/nevatech).

The repository contains the following images:

* **nevatech/sentinet.linux** - based on the official Microsoft ASP.NET Core 10, Debian image.
* **nevatech/sentinet.windows** - based on the official Microsoft ASP.NET Core 10, Windows 2022 Nano-server image.

Both images contains all Sentinet binaries deployed into a single /Sentinet folder, but no configuration files (except for the CLI tool). Ports 80 and 443 are exposed. The entry point is .NET CLI (dotnet) with no parameters. Currently a sample self-signed "CN=host.docker.internal" X.509 certificate is deployed to trusted CA storage.

To run an image you need to provide the name of executable (.dll) corresponding to the running component: "Nevatech.Sentinet.Agent.dll", "Nevatech.Sentinet.Repository.dll", or "Nevatech.Sentinet.Node.dll" as the first argument. You also need to map a local folder to a container folder, place the configuration and all related files into it, and then provide the configuration file path as an environmental variable or query parameter. For example, to start Repository on local ports 7000 and 7001:

```
// Linux
docker run -it --rm
    -p 7001:443 -p 7000:80 
    --mount type=bind,source=combined\config,target=/config 
    --env SENTINET_CONFIG=/config/repository.json 
    --name repository
    nevatech/sentinet.linux:latest Nevatech.Sentinext.Repository.dll

// Windows
docker run -it --rm
    -p 7001:443 -p 7000:80 
    --mount type=bind,source=combined\config,target=C:\config 
    --env SENTINET_CONFIG=C:\config\repository.json 
    --name repository
    nevatech/sentinet.windows:latest Nevatech.Sentinext.Repository.dll
```

In addition to two Sentinet images we also provide a sample pre-configured Sentinet database images with SQL Server for Unix and SQL Server for Windows installations.

* **nevatech/sentinet.database.linux** - based on the official Microsoft SQL Server 2025 for Unix image.
* **nevatech/sentinet.database.windows** - based on Windows Server Core 2022 image with installed SQL Server 2019 Developer Edition.

Both images have port 1433 exposed, and SQL Server is configured to listen on this port.

To start Linux-based image you need to set two enviromental variables: "ACCEPT_EULA=Y" and "MSSQL_SA_PASSWORD=pwd..". For example:

```
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=password" -p 1433:1433 --name database -d nevatech/sentinet.database.linux:latest
```

To start Windows-based image you need to set two enviromental variables: "ACCEPT_EULA=Y" and "sa_password=pwd..". For example:

```
docker run -e "ACCEPT_EULA=Y" -e "sa_password=password" -p 1433:1433 --name database -d nevatech/sentinet.database.windows:latest
```

## Running in Docker Compose

Nevatech Docker Hub repository contains images for running a complete pre-configured Sentinet environment with Docker Compose.

To run Sentinet environment in *Linux containers* on Windows machine:

1. Download and install Docker Desktop: https://www.docker.com/products/docker-desktop/
2. Get the latest Sentinext source code (https://dev.azure.com/nevatech/_git/Sentinet).
3. Install the "config\host.docker.internal.crt" certificate into Windows Trusted Authorities store.
4. Run "run.cmd" script. Docker will download and start 4 containers:
* *database* - runs SQL Server. You can connect to it with SQL Management Studio using "host.docker.internal,6123" address, "sa" username, and "mJ41C-36gQ[P" password.
* *repository* - can be accessed at "https://host.docker.internal:7001/". Username: "sysadmin@test.org", password: "password". Swagger UI is located at "https://host.docker.internal:7001/swagger".
* *agent* - running all tasks
* *node* - has two base addresses registered: "http://host.docker.internal:5000" and "https://host.docker.internal:5001". Periodically heartbeats.
5. To stop the environment hit "Ctrl-C". Containers are stopped, but their state is preserved. To restart, run "run.cmd" again.
6. To remove all containers and start over, run "remove.cmd".

To run Sentinet environment in *Windows containers* execute the same steps below with two differences:

1. Switch Docker Desktop to "Windows containers" mode.
2. Use "run.win.cmd" and "remove.win.cmd" scripts to start and remove containers.
