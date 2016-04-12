---

layout:     documentation
title:      "Documentation | eris:cli | eris ls"

---

# eris ls

List all the things eris knows about.

## Synopsis

List all Eris service, chain, and data containers.

The default output shows containers in three sections. The -a flag
adds a few additional informational columns for each container.

The -r flag limits the output to running services or chains only.

The --json flag dumps the container information in the JSON format.

The -f flag specifies an alternative format for the list, using the syntax
of Go text templates. If the fields to be displayed are separated by the 
'\t' tab character, the output will be columnized.

The struct being passed to the template is:

  type Details struct {
    Type      string          // container type
    ShortName string          // chain, service, or data name
    FullName  string          // container name

    Labels map[string]string  // container labels
    Info   *docker.Container  // Docker client library Container info 
  }

The full list of available fields can be observed by issuing
the [eris ls --json] command.

The default [eris ls] output is equivalent to this custom format:

  {{.ShortName}}\t{{asterisk .Info.State.Running}}\t
  {{short .Info.ID}}\t{{short (dependent .ShortName)}}

The are a few helper functions available to prefix the fields with:

  quote       quote the value
  toupper     make the value upper case
  ports       prettify exposed container ports (used as {{ports .Info}})
  short       shorten the container ID (or any other value) to 10 symbols
  asterisk    show the '*' symbol if the value is true, '-' otherwise
  dependent   find a dependent data container for the given service or chain


```bash
eris ls
```

## Examples

```bash
$ eris ls -rf '{{.ShortName}}, {{.Type}}, {{ports .Info}}'
$ eris ls  -f '{{.ShortName}}\t{{.Type}}\t{{.Info.NetworkSettings.IPAddress}}'
$ eris ls  -f '{{.ShortName}}\t{{.Info.Config.Env}}'
```

## Options

```
  -a, --all             show extended output
  -f, --format string   alternate format for columnized output
      --json            machine readable output
  -r, --running         show only running containers
```

## Options inherited from parent commands

```
  -d, --debug            debug level output
  -m, --machine string   machine name for docker-machine that is running VM (default "eris")
  -v, --verbose          verbose output
```

## See Also

* [eris](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/eris/)	 - The Blockchain Application Platform

## Specifications

* [Actions Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/actions_specification/)
* [Chains Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/chains_specification/)
* [Contracts Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/contracts_specification/)
* [Motivation](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/motivation/)
* [Services Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/services_specification/)

