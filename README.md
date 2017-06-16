Firefox Update Server
=====================

A server for Firefox updates.

The server depends on PostgreSQL.

The protocol is inspired by Google's Omaha client+server,
but is modernized and simplified.

Getting started
===============

# Create database and load schema.
`createdb updates`
`psql -f schema.sql updates`

# Load sample data.
`psql -f sample_data.sql updates`

# Build and run.
`cargo run`

The server is now running on port 9999.

Protocol
========

The server expects an HTTP form POST with a JSON document:

```
{
    "product": "Firefox",
    "version": 666,
    "platform": "macOS",
    "locale": "en-US"
}
```

If no updates are available, the server will return the
HTTP code `204 No Content`.

If updates are available, the server will return a list
of available updates along with a request ID:

```
{
    "available_updates": {
        "blocklist": {
            "url": "https://localhost:8080/src/blocklist_v1.json",
            "hash_function": "sha512",
            "hash_value": "abc123",
            "size": 1234,
            "version": "1.0"
        },
        "gecko_media_plugins": {
            "url": "https://localhost:8080/src/gmp_v1.json",
            "hash_function": "sha512",
            "hash_value": "321cba",
            "size": 4321,
            "version": "2.0"
        }
    },
    "request_id": "xyz321"
}
```

The client may send a "completion" POST to the server to
indicate success or failure to download and apply updates:

```
{
    "updates": {
        "blocklist": {
            "status": "success",
            "version": "1.0"
        },
        "gecko_media_plugins": {
            "status": "failed",
            "reason": "bad checksum",
            "version": "2.0",
            "hash_function": "sha512",
            "hash_value": "abc123",
            "size": 1234
        }
    }
    "request_id": "xyz321",
}
```

The client sends back the server-defined request ID, allowing tracking
of individual updates. The server will record these in to the database.
