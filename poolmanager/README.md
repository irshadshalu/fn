# Running fn in split LB/API/runner mode

## Create certificates

This is a useful article to read for quickly generating mutual TLS certs:

http://www.levigross.com/2015/11/21/mutual-tls-authentication-in-go/

tl;dr: Get this https://github.com/levigross/go-mutual-tls/blob/master/generate_client_cert.go

add IP `127.0.0.1` to the cert by adding the line

```golang
	template.IPAddresses = append(template.IPAddresses, net.ParseIP("127.0.0.1"))
```

somewhere around line 124, and run it with

```bash
go run generate_client_cert.go --email-address a@a.com
```

Tada! Certs.

## Starting the components

### API server

```bash
FN_NODE_TYPE=api ./fnserver
```

### Runner

```bash
mkdir /tmp/runnerdata
FN_DB_URL=sqlite3:///tmp/runnerdata/fn.db FN_MQ_URL=bolt:///tmp/runnerdata/fn.mq FN_NODE_TYPE=pure-runner FN_PORT=8082 FN_NODE_CERT=cert.pem FN_NODE_CERT_AUTHORITY=cert.pem FN_NODE_CERT_KEY=key.pem ./fnserver
```

### LB

```bash
mkdir /tmp/lbdata
FN_DB_URL=sqlite3:///tmp/lbdata/fn.db FN_MQ_URL=bolt:///tmp/lbdata/fn.mq FN_NODE_TYPE=lb FN_PORT=8081 FN_RUNNER_API_URL=http://localhost:8080 FN_NODE_CERT=cert.pem FN_NODE_CERT_AUTHORITY=cert.pem FN_NODE_CERT_KEY=key.pem ./fnserver
```