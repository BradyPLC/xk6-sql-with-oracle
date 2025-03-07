# xk6-sql

This is a [k6](https://github.com/grafana/k6) extension using the
[xk6](https://github.com/grafana/xk6) system. This extension is a fork of [grafana/xk6-sql](https://github.com/grafana/xk6-sql) which was extended by including the [Go Driver for Oracle](https://github.com/godror/godror) which allows connecting to an Oracle database too.

Supported RDBMSs: `mysql`, `postgres`, `sqlite3`, `sqlserver`, `oracle`. See the [tests](tests)
directory for examples.

## Build

To build a `k6` binary with this plugin, first ensure you have the prerequisites:

- [Go toolchain](https://go101.org/article/go-toolchain.html)
- If you're using SQLite or Oracle, a build toolchain for your system that includes `gcc` or
  another C compiler. On Debian and derivatives install the `build-essential`
  package. On Windows you can use [tdm-gcc](https://jmeubank.github.io/tdm-gcc/).
  Make sure that `gcc` is in your `PATH`.
- Git

Then:

1. Install `xk6`:
  ```shell
  go install go.k6.io/xk6/cmd/xk6@latest
  ```

2. Build the binary:
  ```shell
  xk6 build --with github.com/stefnedelchevbrady/xk6-sql-with-oracle
  ```

  If you're using SQLite or Oracle, ensure you have a C compiler installed (see the
  prerequisites note) and set `CGO_ENABLED=1` in the environment:
  ```shell
  CGO_ENABLED=1 xk6 build --with github.com/stefnedelchevbrady/xk6-sql-with-oracle
  ```

  On Windows this is done slightly differently:
  ```shell
  set CGO_ENABLED=1
  xk6 build --with github.com/stefnedelchevbrady/xk6-sql-with-oracle
  ```

## Runtime
If you're using Oracle, make sure you have downloaded the Basic or Basic Light instant client for oracle from [here](https://www.oracle.com/database/technologies/instant-client/downloads.html). After extracting the content of the client make sure to set the `ORACLE_HOME` environment variable on your system and make it point to the Oracle instant client directory. Additionally, add the directory to the `PATH` variable. Keep in mind that some column types of Oracle have specifics so make sure you check the [caveats section of the Go driver for Oracle docs](https://github.com/godror/godror#caveats).

## Development
To make development a little smoother, use the `Makefile` in the root folder. The default target will format your code, run tests, and create a `k6` binary with your local code rather than from GitHub.

```bash
make
```
Once built, you can run your newly extended `k6` using:
```shell
 ./k6 run tests/oracle_test.js
 ```

## Example

```javascript
// script.js
import sql from 'k6/x/sql';

const db = sql.open('oracle', `user="myuser" password="mypass" connectString="127.0.0.1:1521/mydb"`);

export function setup() {
  db.exec(`CREATE TABLE IF NOT EXISTS keyvalues (
           id integer PRIMARY KEY AUTOINCREMENT,
           key varchar NOT NULL,
           value varchar);`);
}

export function teardown() {
  db.close();
}

export default function () {
  db.exec("INSERT INTO keyvalues (key, value) VALUES('plugin-name', 'k6-plugin-sql');");

  let results = sql.query(db, "SELECT * FROM keyvalues;");
  for (const row of results) {
    console.log(`key: ${row.key}, value: ${row.value}`);
  }
}
```

Result output:

```shell
$ ./k6 run script.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: /tmp/script.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0000] key: plugin-name, value: k6-plugin-sql        source=console

running (00m00.1s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.0s/10m0s  1/1 iters, 1 per VU

    █ setup

    █ teardown

    data_received........: 0 B 0 B/s
    data_sent............: 0 B 0 B/s
    iteration_duration...: avg=9.22ms min=19.39µs med=8.86ms max=18.8ms p(90)=16.81ms p(95)=17.8ms
    iterations...........: 1   15.292228/s
```

## See also

- [Load Testing SQL Databases with k6](https://k6.io/blog/load-testing-sql-databases-with-k6/)

## Docker
For those who do not have a Go development environment available, or simply want
to run an extended version of `k6` as a container, Docker is an option to build 
and run the application.

The following command will build a custom `k6` image incorporating the `xk6-sql` extension
built from the local source files.
```shell
docker build -t stefnedelchevbrady/k6-for-sql:latest .
```
Using this image, you may then execute the [tests/sqlite3_test.js](tests/sqlite3_test.js) script 
by running the following command:
```shell
docker run -v $PWD:/scripts -it --rm stefnedelchevbrady/k6-for-sql:latest run /scripts/tests/sqlite3_test.js
```
For those on Mac or Linux, the `docker-run.sh` script simplifies the command:
```shell
./docker-run.sh tests/sqlite3_test.js
```
