
# Rest-Benchmark — End-to-end micro-benchmark suite

This repository contains a small benchmark project that implements the same
REST API across three Java variants so you can compare the performance and
observability surface of different frameworks and approaches:

- Variante A — Jersey (JAX‑RS + JPA)
- Variante C — Spring MVC (Spring Boot + JPA)
- Variante D — Spring Data REST (Spring Boot + Spring Data REST + JPA)

The project includes:

- A shared `benchmark-common` module with the JPA entities (`Item`, `Category`).
- Spring Boot applications for variants C and D (repackaged jars in `target/`).
- A Jersey WAR for variant A (under `variante-a-jersey/`).
- Docker Compose stack to run Postgres, Prometheus, Grafana, and InfluxDB.
- JMeter test plans and CSV payloads in the `jmeter/` folder.
- Scripts and SQL to populate the DB (`schema.sql`, `populate.sql`).

This README gives a concise quickstart and includes three example captures as
requested: `capute`, `caputre1` and `caputre2` (placeholders are provided; add
your screenshots into `docs/` with the same names to make them render).

## Quickstart (minimal)

Prerequisites: Java 17, Maven, Docker & Docker Compose, and optionally JMeter.

1. Build the project (from repository root):

```powershell
mvn -DskipTests package -T1C
```

2. Start the observability / DB stack:

```powershell
docker-compose up -d
```

3. Apply schema and populate the DB (once the Postgres container is healthy):

```powershell
docker cp .\schema.sql benchmark-postgres:/tmp/schema.sql
docker exec -i benchmark-postgres psql -U postgres -d benchmark_db -f /tmp/schema.sql

docker cp .\populate.sql benchmark-postgres:/tmp/populate.sql
docker exec -i benchmark-postgres psql -U postgres -d benchmark_db -f /tmp/populate.sql
```

4. Run a variant locally (example: Spring Boot variant C):

```powershell
Start-Process -FilePath 'java' -ArgumentList '-jar','variante-c-springboot-mvc/target/variante-c-springboot-mvc-1.0-SNAPSHOT.jar' -NoNewWindow -PassThru
```

Or run the Jersey WAR in Docker (the provided Dockerfile includes the
Prometheus JMX javaagent):

```powershell
cd variante-a-jersey
docker build -t rest-benchmark-jersey:latest .
docker run -d --name rest-jersey --network rest-benchmark_default -p 8083:8080 -p 9410:9404 -e APP_FETCH_MODE=JOIN rest-benchmark-jersey:latest
```

5. Verify endpoints and metrics

- Spring MVC (C): http://localhost:8080/items
- Spring Data REST (D): http://localhost:8081/items
- Jersey (A): http://localhost:8083/items (if you used the example run command)
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000
- JMX exporter for Jersey (example): http://localhost:9410/metrics

Notes / troubleshooting:

- If you see HTTP 500 on Spring MVC responses when serializing associations, it's
	usually due to lazy-loading outside a session. For benchmarking, enable JOIN
	fetch mode (`APP_FETCH_MODE=JOIN`) or return DTOs. Variante D (Spring Data REST)
	is configured to return bodies on create/update and should respond out of the box.
- The Jersey WAR may include different Jackson versions that clash with the
	container-provided libraries. If you hit a NoSuchFieldError for Jackson, align
	the Jackson/Jersey dependency versions in the `variante-a-jersey/pom.xml`.

## Capture placeholders

Add your screenshots to the repository under `docs/` using these filenames:

- `docs/capute.png` (placeholder for the "capute" image)
- `docs/caputre1.png` (placeholder for "caputre1")
- `docs/caputre2.png` (placeholder for "caputre2")

They will render here once present:

### capute
![capute](docs/capute.png)

### caputre1
![caputre1](docs/caputre1.png)

### caputre2
![caputre2](docs/caputre2.png)

## Where to look next

- `jmeter/` — test plans and payload CSVs to exercise the 4 scenarios (READ-heavy,
	JOIN-filter, MIXED, HEAVY-body).
- `variante-a-jersey/`, `variante-c-springboot-mvc/`, `variante-d-springdata-rest/`
	— application sources and module-specific configuration.
- `prometheus.yml`, `grafana/` — dashboard and scrape configs.

If you want, I can:

1. Add three placeholder PNG files to `docs/` so the images show up immediately.
2. Fix the Jersey Jackson/Jersey classpath conflict and rebuild the WAR.
3. Fix the Spring MVC lazy-serialization issue (apply JOIN fetch in controller or
	 add DTOs) so all three variants respond cleanly for benchmark runs.

Tell me which of those you'd like and I'll proceed.

