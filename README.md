# yaml-extract

## The Challenge
Your challenge is to develop a tool that extracts a value from a YAML text based on a path expression.
You will need to: 
* Understand the requirements
* Implement the tool
* Document it
* Package it
* Test it
* Deploy it to "production"


## Specification
The yaml-extract tool should get the following inputs:
* A valid YAML text
* A filter expression that represents the value to be extracted:

Example:

* YAML
```yaml
root:
    child1:
        list:
            - element1
            - element2
        listOfdicts:
            - key1: element1
              key2: element2
    child2:
        child2t: "text"
```
* Expression examples

```shell
root.child1.list[0]            ---> "element1"
root.child1.list[1]            ---> "element2"
root.child1.listOfdics[0].key1 ---> "element1"
root.child1.list               ---> ["element1", "element2"]
root.child2                    ---> {"child2t": "text"}
root.child2.child2t            ---> "text"
```

## Implementation details

* The tool should support 2 interaction modes:
    * [CLI](#cli-interaction-mode)
    * [Web API](#web-api-interaction-mode)
* Code should be written with `python3`
    * You could use any Web Framework
    * You could use any IDE you prefer
    * You could use pip or poetry for dependency management
    * Project should be stored within a public git repository (Github.com/Gitlab.com/Bitbucket.com)
    * You will need to write a README on how to build and run the application (Not this Readme file)
      * How to build the project
      * How to run the CLI
      * How to deploy it
      * Pre-Requisites
* Attached are tests/data files for convenience. Copy them to your repository and use them as baseline tests 
* The Web API should be packaged as `Docker image`
* The Web API should run within `docker-compose`
* The Web API should be deployed to a [local K8S cluster](#local-k8s-cluster)
    * Minimum 2 replicas
    * Has a health-check endpoint `/health`
    * Exposed as `http://yaml-extract.test/api/yaml_extract`
    * Helm chart is a bonus, but not required

### Local K8S Cluster
You could use any local K8S cluster that will do job:
* [Docker for Desktop](https://www.docker.com/products/docker-desktop)
* [Minikube](https://minikube.sigs.k8s.io/docs/)
* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
* etc...

Note that you should not push the docker image to an external docker registry...
Each of the clusters supports a way to load the local image to be used within it:
* [KiND - How I Wasted a Day Loading Local Docker Images](https://iximiuz.com/en/posts/kubernetes-kind-load-docker-image/)
* [Minikube - How to use local docker images with Minikube?](https://stackoverflow.com/a/42564211)
* [Minikube - Pushing Images](https://minikube.sigs.k8s.io/docs/handbook/pushing/)


## CLI Interaction mode
The CLI will print the output value to the stdout as json

The following snippets should be used as tests that the program works as expected:

### Help
```
$ yaml-extract --help
Usage: yaml-extract [OPTIONS]
  -f/--file
        The name of a yaml file.
        If the file name is `-`, then it is read from stdin
  -e/--expr
        The expression to extract
  --help
        Display this message and exists

```

### Using a File
```
# Example from file
$ yaml-extract -f tests/data/test.yaml --expr "root.child1.list[0]"
$ element1
```

### Using Standard Input (Stdin)
```
# Example from stdin
$ cat tests/data/test.yaml | yaml-extract \
    -f - \
    --expr "root.child1.list"
$ ["element1", "element2"]
```

## Web API Interaction mode

* Posting to endpoint `/api/yaml_extract`
* Accept `application/json`
* Message structure:

```
{
    "text": "<YAML_TEXT>",
    "expr": "<EXPR>"
}
```

* Returns:

```
{"data": <JSON_RESULT>}
```

The following snippets should be used as tests that the program works as expected:
### App Tests

```
$ curl http://localhost:8888/api/yaml_extract --data-binary "@./tests/data/request.json" -H "Content-Type: application/json"
$ {"data": "element1"}
```
