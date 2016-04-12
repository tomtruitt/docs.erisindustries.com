---

layout:     documentation
title:      "Documentation | eris:cli | eris clean"

---

# eris clean

Clean up your Eris working environment.

## Synopsis

By default, this command will stop and force remove all Eris containers 
(chains, services, data, etc.) Addtional flags can be used to remove 
the Eris home directory and Eris images. Useful for rapid development 
with Docker containers.

```bash
eris clean
```

## Options

```
  -a, --all          removes everything, stopping short of uninstalling eris
  -c, --containers   remove all eris containers (default true)
      --dir          remove the eris home directory: $HOME/.eris
  -i, --images       remove all eris docker images
  -s, --scratch      remove contents of: $HOME/.eris/scratch (default true)
  -y, --yes          overrides prompts prior to removing things
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

