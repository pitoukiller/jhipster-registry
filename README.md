# JHipster Registry

This is a fork from a well known repository !

[![Build Status][github-actions-build]][github-actions-url] [![Docker Status][github-actions-docker]][github-actions-url] [![Docker Pulls](https://img.shields.io/docker/pulls/jhipster/jhipster-registry.svg)](https://hub.docker.com/r/jhipster/jhipster-registry/)

This is the [JHipster](https://www.jhipster.tech/) registry service, based on [Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/), [Eureka](https://github.com/Netflix/eureka) and [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/).

Full documentation is available on the [JHipster documentation for microservices](https://www.jhipster.tech/microservices-architecture).

## Deploy to Heroku

Click this button to deploy your own instance of the registry:

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

There are a few limitations when deploying to Heroku.

- The registry will only work with [native configuration](https://www.jhipster.tech/jhipster-registry/#spring-cloud-config) (and not Git config).
- The registry service cannot be scaled up to multiple dynos to provide redundancy. You must deploy multiple applications (i.e. click the button more than once). This is because Eureka requires distinct URLs to synchronize in-memory state between instances.

## Running locally

To run the cloned repository;

- For development run `./mvnw -Pdev,webpack` to just start in development or run `./mvnw` and run `npm install && npm start` for hot reload of client side code.
- For production profile run `./mvnw -Pprod`

[github-actions-build]: https://github.com/jhipster/jhipster-registry/workflows/Build/badge.svg
[github-actions-docker]: https://github.com/jhipster/jhipster-registry/workflows/Docker%20Image/badge.svg
[github-actions-url]: https://github.com/jhipster/jhipster-registry/actions

## HashiCorp Vault Integration

### Development Mode

`JHipster Registry` default integration uses a `vault` server with an in-memory backend. The data shall not be persisted and shall require you to configure secrets after every restart. The in-memory configuration provides an easy way to test out the integration and later switch to the recommended server mode.

- Start vault server docker container:

```shell
docker-compose -f src/main/docker/vault.yml up -d
```

- The default configured root token is `jhipster-registry`. We shall use the default secrets engine backend mounted on the `secrets` path. Configure secrets using either of `ui`, `cli` or `http`.
- Create a new secret sub-path `jhipster-registry/dev` and add the following secret in JSON format. Here `jhipster-registry` refers to the application name and `dev` refers to the development profile. Do follow the same convention to configure secrets of other applications.

```json
{
  "spring.security.user.password": "admin123!@#"
}
```

- Start `JHipster Registry` server in development mode using the following command (skipping execution of test cases):

```shell
./mvnw -DskipTests
```

- After successful start, open `http://localhost:8761/` in a browser. You shall require entering a new password as provided in the above vault configuration.

### Server Mode

`JHipster Registry` also provides configuration to use the native file system as the persistent backend.

- Uncomment the following configurations in [vault.yml](src/main/docker/vault.yml). You can refer [config.hcl](src/main/docker/vault-config/config/config.hcl) to view provided vault server configurations:

```yml
command: server
volumes:
  - ./vault-config/config:/vault/config
  - ./vault-config/logs:/vault/logs
  - ./vault-config/data:/vault/file
```

- Start vault server docker container:

```shell
docker-compose -f src/main/docker/vault.yml up -d
```

- Open `vault` server [`ui`](http://localhost:8200/ui/vault/init) to initialize master key shares. In this guide, we shall enter `1` as the number of key shares and `1` as the key threshold value. Do refer to vault documentation for recommended configuration. Note down the initial `root token` and the `key` and keep it at a safe place. You shall require the `key` to unseal the vault server after a restart.
- Enable secret engine backend `kv` and use `secrets` as the mount path.
- Create a new secret sub-path `jhipster-registry/dev` and add the following secrets in JSON format. Here `jhipster-registry` refers to the application name and `dev` refers to the development profile. Do follow the same convention to configure secrets of other applications.

```json
{
  "spring.security.user.password": "admin123!@#"
}
```

- In this guide, we shall use the `token` authentication mechanism to retrieve secrets from the `vault` server. Update `bootstrap.yml` to specify `root token` in place of default dev token.

```yaml
vault:
  authentication: token
  token: jhipster-registry # In server mode, provide a token having read access on secrets
```

- Start `JHipster Registry` server in development mode using the following command (skipping execution of test cases):

```shell
./mvnw -DskipTests
```

- After successful start, you shall require entering a new password as provided in vault.
