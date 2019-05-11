# How to Build ARM-based Docker Image using `docker buildx`?


```
[Captains-Bay]🚩 >  docker buildx --help

Usage:	docker buildx COMMAND

Build with BuildKit

Management Commands:
  imagetools  Commands to work on images in registry

Commands:
  bake        Build from a file
  build       Start a build
  create      Create a new builder instance
  inspect     Inspect current builder instance
  ls          List builder instances
  rm          Remove a builder instance
  stop        Stop builder instance
  use         Set the current builder instance
  version     Show buildx version information 

Run 'docker buildx COMMAND --help' for more information on a command.
```

```
[Captains-Bay]🚩 >  docker buildx ls
NAME/NODE DRIVER/ENDPOINT STATUS  PLATFORMS
default * docker                  
  default default         running linux/amd64, linux/arm64, linux/arm/v7, linux/arm/v6
```

We are currently using the default builder, which is basically the old builder.  Let’s create a new builder, which gives us access to some new multi-arch features.

```
[Captains-Bay]🚩 >  docker buildx create --help

Usage:	docker buildx create [OPTIONS] [CONTEXT|ENDPOINT]

Create a new builder instance

Options:
      --append                 Append a node to builder instead of changing it
      --driver string          Driver to use (eg. docker-container)
      --leave                  Remove a node from builder instead of changing it
      --name string            Builder instance name
      --node string            Create/modify node with given name
      --platform stringArray   Fixed platforms for current node
      --use                    Set the current builder instance
```
   
## Creating a new builder called "testbuilder"
   
```
[Captains-Bay]🚩 >  docker buildx create --name testbuilder
testbuilder
```

```
[Captains-Bay]🚩 >  docker buildx ls
NAME/NODE      DRIVER/ENDPOINT             STATUS   PLATFORMS
testbuilder    docker-container                     
  testbuilder0 unix:///var/run/docker.sock inactive 
default *      docker                               
  default      default                     running  linux/amd64, linux/arm64, linux/arm/v7, linux/arm/v6
[Captains-Bay]🚩 > 
```


## Switching to testbuilder

```
[Captains-Bay]🚩 >  docker buildx use testbuilder
[Captains-Bay]🚩 >  docker buildx ls
NAME/NODE      DRIVER/ENDPOINT             STATUS   PLATFORMS
testbuilder *  docker-container                     
  testbuilder0 unix:///var/run/docker.sock inactive 
default        docker                               
  default      default                     running  linux/amd64, linux/arm64, linux/arm/v7, linux/arm/v6
[Captains-Bay]🚩 > 
```

Here I created a new builder instance with the name mybuilder, switched to it, and inspected it.  Note that --bootstrap isn’t needed, it just starts the build container immediately.  Next we will test the workflow, making sure we can build, push, and run multi-arch images. 

```
[Captains-Bay]🚩 >  docker buildx inspect --bootstrap
[+] Building 22.4s (1/1) FINISHED                                                                                        
 => [internal] booting buildkit                                                                                    22.4s
 => => pulling image moby/buildkit:master                                                                          21.5s
 => => creating container buildx_buildkit_testbuilder0                                                              0.9s
Name:   testbuilder
Driver: docker-container

Nodes:
Name:      testbuilder0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Platforms: linux/amd64, linux/arm64, linux/arm/v7, linux/arm/v6
[Captains-Bay]🚩 >  
```

## Authenticating with Dockerhub

```
[Captains-Bay]🚩 >  docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: ajeetraina
Password: 
Login Succeeded

```


## Cloning the Repository

```
[Captains-Bay]🚩 >  git clone https://github.com/collabnix/docker-cctv-raspbian
Cloning into 'docker-cctv-raspbian'...
remote: Enumerating objects: 47, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (31/31), done.
remote: Total 47 (delta 20), reused 32 (delta 14), pack-reused 0
Unpacking objects: 100% (47/47), done.
```


## Peeping into Dockerfile

```
[Captains-Bay]🚩 >  cat Dockerfile 
FROM resin/rpi-raspbian:latest

RUN apt update && apt upgrade && apt install motion
RUN mkdir /mnt/motion && chown motion /mnt/motion
COPY motion.conf /etc/motion/motion.conf

VOLUME /mnt/motion
EXPOSE 8081
ENTRYPOINT ["motion"]
[Captains-Bay]🚩 >
```


##  Building ARM-based Docker Image

```
[Captains-Bay]🚩 >  docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t ajeetraina/docker-cctv-raspbian --push .

[Captains-Bay]🚩 >  docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t ajeetraina/docker-cctv-raspbian --push .
[+] Building 254.2s (12/21)                                                                                              
[+] Building 2174.9s (22/22) FINISHED                                                                                    
 => [internal] load build definition from Dockerfile                                                                0.1s
 => => transferring dockerfile: 268B                                                                                0.0s
 => [internal] load .dockerignore                                                                                   0.1s
 => => transferring context: 2B                                                                                     0.0s
 => [linux/arm/v7 internal] load metadata for docker.io/resin/rpi-raspbian:latest                                   8.8s
 => [linux/arm64 internal] load metadata for docker.io/resin/rpi-raspbian:latest                                    8.8s
 => [linux/amd64 internal] load metadata for docker.io/resin/rpi-raspbian:latest                                    8.8s
 => [linux/amd64 1/4] FROM docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a99  0.1s
 => => resolve docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a996424ba8a15d5  0.0s
 => [internal] load build context                                                                                   0.1s
 => => transferring context: 27.72kB                                                                                0.0s
 => [linux/arm64 1/4] FROM docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a9  17.3s
 => => resolve docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a996424ba8a15d5  0.0s
 => => sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a996424ba8a15d5f88d055b 2.81kB / 2.81kB                      0.0s
 => => sha256:67a3f53689ffd00b59f9a7b7f8af0da5b36c9e11e12ba43b35969586e0121303 1.56kB / 1.56kB                      2.3s
 => => sha256:ef16d706debb7a56d89bf14958aca2da76bd3d0a42a6762c37302c9527a47d10 305B / 305B                          3.5s
 => => sha256:2734b459704280a5238405384487095392e5414d973554b27545cec0e984a0f2 256B / 256B                          3.1s
 => => sha256:9d5b46a00008070b009b685af5846c225ad170f38768416b63917ea7ac94062d 7.75kB / 7.75kB                      4.4s
 => => sha256:5fb2b6f7daac67a8465e8201c76828550b811619be473ab594972da35b4b7ee7 354B / 354B                          4.4s
 => => sha256:a04aef7b1e2fc4905fea2a685c63bc1eeb94c86149fd1286459206c22794e610 177B / 177B                          4.4s
 => => sha256:3eae71a21b9db8bd54d6a189b7587a10f52d6ffee3d868705a89974fe574c268 234B / 234B                          2.3s
 => => sha256:28f1ee4d4d5aa8bb96b3ba6a5e2177b2b58b595edaf601b9aae69fd02f78a6c6 7.48kB / 7.48kB                      0.0s
 => => sha256:6bddb275e70b0083d76083d01be2c3da11f67f526a123adcc980c5a3260d46e8 51.54MB / 51.54MB                   13.4s
 => => sha256:873755612f304f066db70c4015fdeadc9a81c0e6e25fb1aa833aeba80a7aeffc 229B / 229B                          3.9s
 => => sha256:78ba3f0466312c019467b178339a89120f2dce298d7c3d5e6840b3566749f5c0 250B / 250B                          3.2s
 => => sha256:b98db37cf25231afe68852e2cb21fb8aa465bb7a32ecc9afc6dec100ec0ba9b0 367B / 367B                          3.5s
 => => sha256:6b6c68e7ac8567569cee8da92431637e561e7aef5addb70373401d0887447a00 363B / 363B                          4.4s
 => => unpacking docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a996424ba8a15  3.8s
 => [linux/arm/v7 1/4] FROM docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a9  0.0s
 => => resolve docker.io/resin/rpi-raspbian:latest@sha256:18be79e066099cce5c676f7733a6ccb4733ed5962a996424ba8a15d5  0.0s
 => [linux/amd64 2/4] RUN cat /.resin/deprecation-warning                                                           0.6s
 => [linux/arm/v7 2/4] RUN cat /.resin/deprecation-warning                                                          0.5s
 => [linux/arm64 2/4] RUN cat /.resin/deprecation-warning                                                           0.6s
 => [linux/arm/v7 3/4] RUN apt update && apt upgrade && apt install motion                                        481.2s
 => [linux/amd64 3/4] RUN apt update && apt upgrade && apt install motion                                        1431.5s
 => [linux/arm64 3/4] RUN apt update && apt upgrade && apt install motion                                        1665.5s
 => [linux/arm/v7 4/4] RUN mkdir /mnt/motion && chown motion /mnt/motion                                            0.6s
 => [linux/arm/v7 5/4] COPY motion.conf /etc/motion/motion.conf                                                     0.1s
 => [linux/amd64 4/4] RUN mkdir /mnt/motion && chown motion /mnt/motion                                             0.6s
 => [linux/amd64 5/4] COPY motion.conf /etc/motion/motion.conf                                                      0.1s
 => [linux/arm64 4/4] RUN mkdir /mnt/motion && chown motion /mnt/motion                                             0.6s
 => [linux/arm64 5/4] COPY motion.conf /etc/motion/motion.conf                                                      0.1s
 => exporting to image                                                                                            481.7s
 => => exporting layers                                                                                            19.6s
 => => exporting manifest sha256:55ddd5c67557190344efea7327a5b2f8de0bdc8ba184f856b1086baac6bed702                   0.0s
 => => exporting config sha256:ebb6e951b8f206d258e8552313633c35f4fe4c82fe7a7fcc51475022ae089c2d                     0.0s
 => => exporting manifest sha256:7b6919de7edd7d1be695877f827e7ee6d302d3acfd3f69ed73bf2ffaa4a80632                   0.0s
 => => exporting config sha256:a83b02bf9cbcb408110c4b773f4e5edde04f35851e2a42d4ff1d947c132bed6d                     0.0s
 => => exporting manifest sha256:d379c0f79b6a72a770124bb2ee94d91d9afef031c81ac20ea7a4c51f4f13ddf2                   0.0s
 => => exporting config sha256:86c2e637fe39036d51779c3bb5f800d3e8da14122907c5fd03bdace47b03bb38                     0.0s
 => => exporting manifest list sha256:daec2787002024c07addf56a8099a866d7f1cd85ed8c33818beb64a5a208cd54              0.0s
 => => pushing layers                                                                                             455.4s
 => => pushing manifest for docker.io/ajeetraina/docker-cctv-raspbian:latest                                        6.4s
[Captains-Bay]🚩 >  
[Captains-Bay]🚩 >  

```

That worked well!  The --platform flag told buildx to generate Linux images for Intel 64-bit, Arm 32-bit, and Arm 64-bit architectures. The --push flag generates a multi-arch manifest and pushes all the images to Docker Hub.  

## What is this ImageTools all about?


Let’s use imagetools to inspect what we did. 


```
[Captains-Bay]🚩 >  docker buildx imagetools inspect docker.io/ajeetraina/docker-cctv-raspbian:latest
Name:      docker.io/ajeetraina/docker-cctv-raspbian:latest
MediaType: application/vnd.docker.distribution.manifest.list.v2+json
Digest:    sha256:daec2787002024c07addf56a8099a866d7f1cd85ed8c33818beb64a5a208cd54
           
Manifests: 
  Name:      docker.io/ajeetraina/docker-cctv-raspbian:latest@sha256:55ddd5c67557190344efea7327a5b2f8de0bdc8ba184f856b1086baac6bed702
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/amd64
             
  Name:      docker.io/ajeetraina/docker-cctv-raspbian:latest@sha256:7b6919de7edd7d1be695877f827e7ee6d302d3acfd3f69ed73bf2ffaa4a80632
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm64
             
  Name:      docker.io/ajeetraina/docker-cctv-raspbian:latest@sha256:d379c0f79b6a72a770124bb2ee94d91d9afef031c81ac20ea7a4c51f4f13ddf2
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm/v7
[Captains-Bay]🚩 >
```

