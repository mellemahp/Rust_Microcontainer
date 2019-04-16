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
Everything is set up with docker compose and hosts on port 8000. Simply Run the following from the project root folder:
```
docker-compose up
```
That's it! The first time you run this it will take quite a while, but after that if boots up very fast.

Now if you go to `localhost:8000` you should see a "Hello World" message

# Tiny images! 
If we take a look at the actual container size its only 4.77MB!!! That's fantastic. 
```bash
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS              PORTS                    NAMES                                 SIZE
fee29ebeaf94        rust_microcontainer_rest_endpoint   "/hello_world"      23 seconds ago      Up 21 seconds       0.0.0.0:8000->8000/tcp   rust_microcontainer_rest_endpoint_1   0B (virtual 4.77MB)
```
For comparison the container for an alpine [https://hub.docker.com/r/jazzdd/alpine-flask/](flask image) is 81.5MB: 
```
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES               SIZE
70f3eb1d21f4        jazzdd/alpine-flask   "/entrypoint.sh"    29 seconds ago      Up 27 seconds       80/tcp              gracious_diffie     5B (virtual 81.5MB)
```
And this is with only a uWSGI server running in the flask image, no app.py is loaded. I expect that the difference only becomes more stark as the size of the applications grow.


