# Exploring tools

There are a bunch of tools and a bunch of differen things you can do with these. This doc looks at some examples.

## Exploring the shell

When opening the Mongo service, then there are a bunch of different options that can be configured when launching it.

You can specify a DB data path, or a log path, or any other of the many options available.

You can also specify a config file (.cfg) for Mongo to use config options from. When booting up the Mongo instance, you can point it to the config file.

In UNIX systems, you can run Mongo as a background service using `--fork`.

To shut it down, you can connect to the Mongo instance, use the admin DB, and type `db.shutdownServer()`.

The shell can also be configured, but it has less options.

## MongoDB Compass

MongoDB Compass can be used to explore and manage the DB with a UI.