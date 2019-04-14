# Rust_Microcontainer
Dockerfile and Brief walkthrough on how to deploy tiny rust containers. Uses the template rocket app as a demo

# Building Tiny rust binary containers 
Here we will quickly walk through how to construct a tiny docker container with only a rust binary in 
it. This only covers the construction of the container, but in production we would likely be a bit more
clever about the configuration of the container doing things like setting proper permissions, and using
arguments to get variable names. 

# The setup 
Basically the build goes as follows
```
[YOUR MACHINE] --copy rust files to--> [BUILD CONTAINER]
[BUILD CONTAINER] --loads all necessary libraries (like musl) and compiles binary--
[BUILD CONTAINER] --copy rust binary to new container --> [DEPLOY CONTAINER]
[DEPLOY CONTAINER] --entrypoint via rust binary--
```
We want to end up with a "scratch" container with nothing other than a 

# Two critical things
1) You must install musl-tools so you can use the musl library for the compilation of the rust binary
2) You must install a target x86_64-unknown-linux-musl (or _32) so rust can compile the standalone binary

# Alright, lets walk through the Dockerfile
All the steps are already done for you in the dockerfile in this directory. To run it copy the Dockerfile 
into the root directory of your project and change the following lines: 
```
COPY ./hello_world ---> COPY ./<PROJECT SOURCE DIRECTORY> 
```
This copies your source code into the container. Next: 
```
WORKDIR /hello_world ---> WORKDIR /<YOUR_PROJECT_SOURCE_DIR>
```
This cd's you into the source directory in the container. This is recommended over using `RUN cd /hello_world`.
Now, change in the deploy container section
```
COPY --from=build /<SOURCE_DIR_NAME>/target/x86_<64 or 32>-unknown-linux-musl/release/<PROJECT_NAME> /
```
Also change the port you want to use by editing 
```
EXPOSE 8000 --> EXPOSE <YOUR PORT> 
```
Finally change the entrypoint to access the binary you just added to the deploy container
```
ENTRYPOINT ["/hello_world"]--> ENTRYPOINT ["/<PROJECT_NAME>"]
```

# Great! Now lets build your container!
We are going to run the container with the name "rust/hello:v1" and have it serve on the port 8000 on the host
machine. To do this run the following. First, build the container from your source directory containing the Dockerfile
```
docker build -t rust/hello:v1 .
```
Now this will take a while. Once it's built you run the image with the following
```
docker container run -p 8000:8000 --rm rust/hello:v1
```
