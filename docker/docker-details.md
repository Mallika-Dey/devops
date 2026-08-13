## Docker Daemon (dockerd)
Docker daemon is the background service that manage the docker objects like - image, container, network etc. also can communicate with docker registry. 

```bash
You
 │
 │ docker run nginx
 ▼
Docker CLI
 │
 │ Docker API
 ▼
Docker Daemon (dockerd)
 │
 ├── Image management
 ├── Container management
 ├── Network management
 └── Volume management
```

Docker CLI = client; Docker daemon = engine doing the work.

## Docker caching
If the changes only in src/; then docker can reuse the <code>resolve dependencies</code> in next build. 

```bash
# resolve dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package
```