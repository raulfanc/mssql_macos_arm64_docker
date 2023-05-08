## Pre-requisites:
#### 1. Download Docker
To have docker run in the MacOS, need **Docker Daemon**, the easiest way is to download docker desktop, [Docker download](https://docs.docker.com/desktop/install/mac-install/), everytime you open the docker desktop, the daemon will be running in the background.
#### 2. Docker Settings
- (1). arm64 (apple silicon) chip is incompatible with mssql, so need to enable this
- (2). Performance is ok, no need to worry
- (3). open `Docker desktop GUI`--> `setting`--> `Features in development`-->enable `emulation`
![](../pictures/Pasted%20image%2020230508111301.png)

#### 2. Data Management tool
since sql server management tool is not available in MacOS both intel chip and silicon chip, can either use JetBrains's [Data Grip](https://www.jetbrains.com/datagrip/?source=google&medium=cpc&campaign=15022575985&term=datagrip&content=554780841060&gad=1&gclid=Cj0KCQjwmN2iBhCrARIsAG_G2i5jB1TaEJL5W0fGRQOnfrHf5veiiCwqDIww4H24s17-mWuChkiPmWQaAj_eEALw_wcB) or Microsoft's [Azure Data Studio](https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver16&tabs=redhat-install%2Credhat-uninstall)
- **Data Grip** is recommended, with a `Uni email` can use it for free, refer to [link](https://www.jetbrains.com/community/education/#students)
- **Azure Data Studio** is free, but not as user-friendly as **Data Grip**

---

## Set up docker mssql
- For someone hasn't come across Docker before
- This step is to `set up mssql server` but in a container, similar to the scenario of you installing a SQL Server app on your Windows PC and turned it on/off?

#### 1. Pull Docker Images for `mssql2022`
```bash
docker pull mcr.microsoft.com/mssql/server:2022-latest
```
Can have other versions, i.e., `sql server 2019` or `sql server express`, refer to [Docker other images](https://hub.docker.com/_/microsoft-mssql-server)

#### 2. Run Docker container
``` bash
docker run -e ACCEPT_EULA='Y' -e MSSQL_SA_PASSWORD='<your_password>' \
-p <local port>:1433 \
--name sql2022 --hostname sql2022 \
-v mssql:/var/opt/mssql \
-d mcr.microsoft.com/mssql/server:2022-latest
```
need to **change the following parameters** with your docker run command.
- `MSSQL_SA_PASSWORD` is the desired password to login, and to use mssql, the username is `SA`
- `-p` is to map the port to the host's port.
- - `name` is to set the container name in docker
- `-hostname` is optional, can remove this parameter
- `-v` is to map the data to `docker volume container`, here, I created a volume container called `mssql`. **Note: `Mounting` is not available, so I used `Volume Container` to persist data**, more information refer to [MS docs](https://learn.microsoft.com/en-us/azure/azure-sql-edge/configure#use-data-volume-containers) 
- `-d` is to set the container to run on a detached mode, means if the terminal is closed, the server container is still on

#### 3. `Rosetta`
make sure  `Rosetta` is installed to accelerate x86/amd64 binary emulation on Apple Silicon

- Rosetta is installed?
```bash
softwareupdate --list-rosetta
```

- If Rosetta is installed, you'll see a message similar to this:
```bash
Label: Rosetta
Title: Rosetta
Version: 2.0
Size: 4051997
Kind: Optional
Install Prefix: /
Identifier: com.apple.pkg.RosettaUpdate
```

- If Rosetta is not installed, you'll see a message similar to this:
```bash
No updates are available.
```

- install `Rosetta`
```bash
softwareupdate --install-rosetta
```

##### 4. Check the docker is running? 
Go to Docker Desktop GUI
- **Check image**: I am using sql server 2022, you could use other version if you like.
![](../pictures/Pasted%20image%2020230508113308.png)

- **Check container**: turn it on and off. Or, can use bash `docker start sql2022`,  `docker stop sql2022` 
![](../pictures/Pasted%20image%2020230508140446.png)
- **Check volume container**: the data volume to store all data associated with the mssql
![](../pictures/Pasted%20image%2020230508113703.png)


## Data Grip
using Data Grip as an alternative tool if you cannot use `SQL Server Management Studio (SSMS)`
1. Create a Server with mssql
![](../pictures/Pasted%20image%2020230508115214.png)

2. Fill in details to build the connection
![](../pictures/Pasted%20image%2020230508115505.png)

3. create a table and insert some data
use this [`sql`](northwind1.sql) script to create a `northwind1 products` table 

4. Data Grip Interface overview
![](../pictures/Pasted%20image%2020230508115851.png)

---

## Other settings (optional):

- refer to [MS docs](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-docker-container-configure?view=sql-server-ver16&pivots=cs1-bash) or [Doc docs](https://hub.docker.com/_/microsoft-azure-sql-edge)

- change SA_Password
```bash
sudo docker exec -it azuresqledge /opt/mssql-tools/bin/sqlcmd \
   -S localhost -U SA -P "<YourStrong@Passw0rd>" \
   -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
```


## SQL Edge

- docker pull `sql-edge` image
```bash
docker pull mcr.microsoft.com/azure-sql-edge
```

- docker run it (recommended to have the developer edition) - mapped host port: 1433
```bash
sudo docker run \
--cap-add SYS_PTRACE \
-e 'ACCEPT_EULA=1' \
-e 'MSSQL_SA_PASSWORD=<your_password>' \
-p 1433:1433 \
-v sql-edge:/var/opt/mssql \
--name azuresqledge \
-d mcr.microsoft.com/azure-sql-edge
```

Reference: [Development with SQL in containers on macOS](https://devblogs.microsoft.com/azure-sql/development-with-sql-in-containers-on-macos/) by Drew Skwiers-Koballa