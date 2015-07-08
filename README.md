# java-node-component

Docker container for building a Java project that runs an application server on 8080 but also requires Node.js.

The purpose of this base image is to speed up builds. Docker will cache `RUN git clone ...`, so, for simplicity, ensuring the container builds with the latest code requires the use of `docker build --no-cache ...`. That's slow if you start from a Java base image and reinstall Git and Maven every time.

This base image therefore provides those common steps so you can run with `--no-cache` and still minimise build time.

## Example component Dockerfile

```
from carboni.io/java-node-component

# Consul check

WORKDIR /etc/consul.d
RUN echo '{"service": {"name": "my-component", "port": 8080, "check": {"script": "curl http://localhost:8080 >/dev/null 2>&1", "interval": "10s"}}}' > my-component.json

# Check out code from Github

WORKDIR /usr/src
RUN git clone https://github.com/username/my-repo.git .
# Select branch
RUN git checkout release

# Build the component

RUN npm install
RUN mvn install -DskipTests

# Add command to the entry point script

RUN echo "java -jar target/*-jar-with-dependencies.jar" >> container.sh
