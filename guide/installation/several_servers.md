# Several servers

One application can run on several servers behind a load balancer. Nothing a request needs may exist on one server only. ColdIgniter keeps no secret and no session in the memory of the server, but three things must be set up on purpose.

## Sessions

A session that is kept in files exists on one server only. Use one of these:

- the database handler, so every server reads the same sessions, or
- a load balancer with sticky sessions, so a visitor always reaches the same server.

## Encryption

Use the same key and the same cipher on every server. If they differ, one server cannot read what another one encrypted.

The default cipher, AES with a 128-bit key, works on every default install. If you choose a 256-bit key, the JCE unlimited strength policy files must be on every server.

## Cache

The file cache is kept in `writable/cache/`. Each server has its own cache unless this folder is on a shared disk. Both ways work. With a cache per server, one server can still show an old entry after another server has replaced it.
