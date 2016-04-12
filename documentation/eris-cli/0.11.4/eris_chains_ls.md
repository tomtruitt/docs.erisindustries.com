---

layout:     documentation
title:      "Documentation | eris:cli | eris chains ls"

---

# eris chains ls

Lists everything chain related.

## Synopsis

List chains or known chain definition files.

The -r flag limits the output to running chains only.

The --json flag dumps the container or known files information
in the JSON format.

The -q flag is equivalent to the '{{.ShortName}}' format.

The -f flag specifies an alternate format for the list, using the syntax
of Go text templates. See the more detailed description in the help
output for the [eris ls] command. The struct passed to the Go template
for the -k flag is this

  type Definition struct {
    Name       string       // chain name
    Definition string       // definition file name
  }

The -k flag displays the known definition files. 

```bash
eris chains ls
```

## Examples

```bash
$ eris chains ls -f '{{.ShortName}}\t{{.Info.Config.Image}}\t{{ports .Info}}'
$ eris chains ls -f '{{.ShortName}}\t{{.Info.State}}'
```

## Options

```
  -a, --all             show extended output
  -f, --format string   alternate format for columnized output
      --json            machine readable output
  -k, --known           list all the chain definition files that exist
  -q, --quiet           show a list of chain names
  -r, --running         show running containers
```

## Options inherited from parent commands

```
  -d, --debug            debug level output
  -m, --machine string   machine name for docker-machine that is running VM (default "eris")
  -v, --verbose          verbose output
```

## See Also

* [eris chains](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/eris_chains/)	 - Start, stop, and manage blockchains.

## Specifications

* [Actions Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/actions_specification/)
* [Chains Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/chains_specification/)
* [Contracts Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/contracts_specification/)
* [Motivation](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/motivation/)
* [Services Specification](https://docs.erisindustries.com/documentation/eris-cli/0.11.4/services_specification/)

