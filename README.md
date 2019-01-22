SRS XML Client (SXC)
====================

This is a simple command line interface to the NZ Shared Registry System.

It is designed to allow generation and transmission of all the XML actions
that the registry supports using command line arguments rather than having
to manually write the XML yourself.

Registrars may use that when integrating the SRS into their own systems
as well as for executing any one-off transactions that their system does
not support.

InternetNZ devlopers use the client when testing the SRS.

Using `--dry-run` it is also possible to generate XML than can then be
sent using the `send-xml` action for any feature not supported by the
client itself.

REQUIRED MODULES
----------------

`sxc` uses the following non-standard python modules: pgpy, lxml, requests
and jinja2.

These are all available as Debian packages, to install them run:
`sudo apt-get install python3-pgpy python3-lxml python3-requests python3-jinja2`.

ENVIRONMENT
-----------

The following environment variables can be used:

* `REGISTRY_URL` URL for the SRS, eg:
  `https://srstest.srs.net.nz/srs/registrar`, used when `--url` is not
  specified.

* `REGISTRY_CERTIFICATE` X509 CA certificate that authenticates the HTTPS
  connection, used when `--certificate` is not specified.

* `REGISTRY_KEY` The PGP key used to verify responses from the registry,
  used when `--verify` is not specified.

* `REGISTRAR_ID` Your registrar ID, used when `--registrar-id` is not
  specified.

* `REGISTRAR_KEY` The PGP key to use when signing the request, used when
  `--sign` is not specifed.

USAGE
-----

Usage: `sxc ACTION OPTIONS`. Run `sxc help` to see available
actions.

Each action has a help message that shows all the possible options for that
action and brielf, what they do. Run `sxc help ACTION` to see that help, eg:

```sh
sxc help domain-update
```

Most command line arguments are optional, you should always run the command
with `--dry-run` first to verify that the XML is what you want to send.

You emulate the old sendXML script using `send-xml`.

All requests require a registrar ID which is specified using `--registrar-id`.

SRS URL is specified using `--url`, it is not required if `--dry-run` is
being used.

Both options can be set using enviroment variables, see ENVIROMENT above.

In the following examples, each option is shown on it's own line for readability,
your shell will require everything to be on one line.

An example whois:

```sh
sxc whois
  --url https://srstest.srs.net.nz/srs/registrar
  --registrar-id 500
  --domain example.co.nz
```

Some options parsed by the client are context sensitive, they are sub-options
of the option that preceeded it.

In this example `domain-create`, the sub-options for `--registrant` and
`--nameserver` are context specific.

```sh
sxc domain-create
  --url https://srstest.srs.net.nz/srs/registrar
  --registrar-id 500
  --domain example.co.nz
  --term 12
  --delegate
  --registrant
    --name "Registrant Name"
    --email "registrant.email@example.co.nz"
    --address1 "Registrant Address"
    --city "Registrant City"
    --country NZ
    --phone "64-4-123456789"
 --nameserver
   --name ns1.example.co.nz
   --ip4 "192.168.0.1"
 --nameserver
   --name ns2.example.net.nz
```

For actions that take multiple domains, just provide multiple arguments to
`--domain`.

```sh
sxc domain-update
  --url https://srstest.srs.net.nz/srs/registrar
  --registrar-id 500
  --domain example.co.nz example.org.nz example.net.nz
  --registrant
    --privacy
  --audit "Enable IRPO on registrant"
```

If you don't supply an `--action-id`, one will be automatically generated.

This is an example `domain-details-query` with `--dry-run` being used,
so it just outputs the XML that would have been sent.

```sh
sxc domain-details-query
  --dry-run
  --registrar-id 500    
  --fields term delegate nameservers registered billed-until 
```

Dates only require the day, month and year. You can specifiy the hour, minute,
second and timezone optionally. Unspecified parts of the date are set to 0.

This `get-messages` example will fetch all messages for that particular day.

```sh
sxc get-messages
  --url https://srstest.srs.net.nz/srs/registrar
  --registrar-id 500
  --transaction-from "1/8/2018"
  --transaction-to "1/8/2018 23:59:59"
```

To send your own XML, use `send-xml` as the action.

```sh
sxc send-xml
  --url https://srstest.srs.net.nz/srs/registrar
  --registrar-id 500
  --file request.xml
```

Using environment variables you can omit the `--url` and
`--registrar-id`.

This `registrar-account-query` example will get all transactions for the
specified month.

```sh
export REGISTRY_URL="http://srstest.srs.net.nz/srs/registrar"
export REGISTRAR_ID="500"
sxc registrar-account-query
  --transaction-from "1/8/2018 00:00:00"
  --transaction-to "31/8/2018 23:59:59"
```

TODO
----

* Client help messages can be improved and common options put into
  template blocks (global, registrant, nameserver, dnssec).

* Better validation of command line arguments, `--domain` should only
  allow wild-cards for actions using `DomainNameFilter`

* Add support for multiple handle-id's to be specified for
  `HandleDetailsQry`.

* Add an output filter that converts XML to something more readable
  YAML-like syntax?

* Add XML syntax highlighting.
