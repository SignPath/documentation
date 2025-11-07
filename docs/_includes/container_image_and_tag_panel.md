{:.panel.info}
> **FQN and TAG**
>
> <a name="fqn"/> 
> **Fully qualified name (FQN)**
> 
> For images hosted on Docker Hub, the FQN is `docker.io/$namespace/$repository`, e.g. `docker.io/jetbrains/teamcity-server`. 
> 
> If you are using your own registry, specify the value you would use for Docker CLI commands, but without tag or digest values. E.g. when using `docker pull myreg.jfrog.io/> myrepo/myimage:latest`, the FQN would be `myreg.jfrog.io/myrepo/myimage`.
> 
> 
> `$TAG` refers the specific image tag (e.g. `latest`)