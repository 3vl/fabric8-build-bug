This repo demonstrates a difference in how `docker build` and `mvn docker:build` handle
environment variables. fabric8 seems to pull any environment variable from the mvn process
that also has a ENV entry in the Dockerfile and imports it into the docker image.


The demo Dockerfile simply has an `ENV INFO=default`

To see how `docker build` handles the situation. Notice that the `docker run` prints
the value `default` which was set in the `Dockerfile` and does not the value `cows` from
the `docker build` process.

```bash
$ INFO=cows docker build -t env .
Sending build context to Docker daemon  51.71kB
Step 1/3 : FROM bash
 ---> a3cae8598d52
Step 2/3 : ENV INFO=default
 ---> Using cache
 ---> ed13f219d3c6
Step 3/3 : CMD echo ${INFO}
 ---> Using cache
 ---> 1749f7a40bf0
Successfully built 1749f7a40bf0
Successfully tagged env:latest

$ docker run -it --rm env
default
```

However, if we build the same image with with fabric8 docker:build notice that the environment
variable `INFO` now has the value of `cows` which appears to have been picked up from
the environment of the maven process.

```bash
$ INFO=cows mvn docker:build
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------< test:env >------------------------------
[INFO] Building env 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- docker-maven-plugin:0.34.1:build (default-cli) @ env ---
[INFO] Building tar: /Users/m_884025/dev/fabric8-build-bug/target/docker/env/tmp/docker-build.tar
[INFO] DOCKER> [env:latest]: Created docker-build.tar in 35 milliseconds
[INFO] DOCKER> [env:latest]: Built image sha256:311ca
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.521 s
[INFO] Finished at: 2021-02-04T13:55:06-05:00
[INFO] ------------------------------------------------------------------------
$ docker run -it --rm env
cows
```

fabric8's docker:build seems to be behaving as thought theres an `--env` parameter for every
environment variable in the building process.
