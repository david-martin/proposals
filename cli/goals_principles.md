# Abstract

This document will outline the goals of creating a mobile CLI for K8s / OpenShift. It will also outline the technical considerations and design principles that should be used to guide the development of not only the mobile CLI, but any other mobile service CLI.



# Goals

The key goal of the mobile cli, is to provide a simple and intuitive interface to enable and assist developers to adopt and use Open Source mobile services on top of Kubernetes and OpenShift in order to solve their mobile project requirements.


# General design principles and UX considerations

## Output

As this is a CLI, it is important that it remains a programmable grep and pipe friendly interface to allow scripting and execution from external programs. This generally means it should write its output to stdOut and its errors to stdErr. Using a format like JSON allows it to be piped to tools such as [jq](https://stedolan.github.io/jq/) for further use.

However it should default to presenting its output in a human readable format such as a table structure:
```
+-----------------------+-----------------------------+
|         NAME          |             ID              |
+-----------------------+-----------------------------+
| dh-fh-sync-server-apb | dh-fh-sync-server-apb-59dkt |
+-----------------------+-----------------------------+
```

## Arguments and Flags
	
- An argument represents a required parameter needed in order to complete the requested action. 
- A flag represent an optional value needed for additional functionality or to change default behaviour. Flags can be global, as in spanning all commands or specific to a single command. Were appropriate they should have a short and long format.

```
mobile get serviceinstances fh-sync-server --output=json
```
In this example we have  ```servicesinstances``` and ```fh-sync-server``` that are required but the ```--output``` flag is optional to change the default behaviour for how the output is formatted. As much as possible we want to align with the kubectl practices here to keep it a familiar interface.

## Errors

In the case of an error, there should be a clear message printed as the first line of output, the process should then exit with a non zero exit code so then when being called from an external program it is clear when something has failed.
In the case of an exception or unexpected error, a clear error should still be shown as the first line of output, however the stacktrace should also be shown so that it can be used in support tickets and github issues to highlight were the bug is and how it happened.

```
Error: failed to find a service class associated with that name

error: exit status 1
```

## User Instruction
User instruction should have the format outlined below and be shown when the user incorrectly uses a command uses the help command or adds a --help / -h flag to a command:

```
Usage:
  mobile create serviceinstance <serviceName> [flags]
  
Flags:
  -h, --help      help for serviceinstance
      --no-wait   --no-wait will cause the command to exit immediately instead of waiting for the service to be provisioned

Global Flags:
      --namespace string   --namespace=myproject
  -o, --output string      -o=json -o=template (default "table")
```  
  

