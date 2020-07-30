# Lab 2- Adding Value with Custom Docker Images

## Step 3: Run the Docker image

Now that you have built the image, you can run it to see that it works.

1. Run the Docker image

    ```sh
    docker run --name python-$DOCKER_USER -p 5000:5000 -d $DOCKER_USER/python-hello-world:v1
    $ 0b2ba61df37fb4038d9ae5d145740c63c2c211ae2729fc27dc01b82b5aaafa26
    ```

    The `-p` flag maps a port running inside the container to your host. In this case, we are mapping the python app running on port 5000 inside the container, to an external port on your host.

1. curl `http://localhost:5000` to see the results. 

    ```sh
    curl http://localhost:5000
    $ hello world
    ```

    You should see "hello world!" in your terminal.

1. Check the log output of the container.

    If you want to see logs from your application you can use the `docker container logs` command. By default, `docker container logs` prints out what is sent to standard out by your application. Use `docker container ls` to find the id for your running container.

    ```sh
    docker container logs python-$DOCKER_USER 
    $ * Serving Flask app "app" (lazy loading)
    * Environment: production
      WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    172.17.0.1 - - [30/Jul/2019 02:02:46] "GET / HTTP/1.1" 200 
    ```

1. Finally, clean up your image

    ```sh
    docker container rm -f python-$DOCKER_USER
    ```

    The Dockerfile is how you create reproducible builds for your application. A common workflow is to have your CI/CD automation run `docker image build` as part of its build process. Once images are built, they will be sent to a central registry, where it can be accessed by all environments (such as a test environment) that need to run instances of that application. In the next step, we will push our custom image to the public docker registry: the docker hub, where it can be consumed by other developers and operators.

## Step 4: Push to a central registry

1. Push your image to the Dockerhub registry

    Once we have a properly tagged image, we can use the `docker push` command to push our image to the Docker Hub registry.

    ```sh
    docker push $DOCKER_USER/python-hello-world:v1
    $ The push refers to a repository [docker.io/isaias/python-hello-world]
    2bce026769ac: Pushed 
    64d445ecbe93: Pushed 
    18b27eac38a1: Mounted from library/python 
    3f6f25cd8b1e: Mounted from library/python 
    b7af9d602a0f: Mounted from library/python 
    ed06208397d5: Mounted from library/python 
    5accac14015f: Mounted from library/python 
    latest: digest: sha256:508238f264616bf7bf962019d1a3826f8487ed6a48b80bf41fd3996c7175fd0f size: 1786
    ```

    > Notice the "Mounted from..." entries, these are layers that are already in the registry. Only the new layers created in the image build are pushed to the Docker Hub registry.

1. Check out your image on docker hub in your browser

    Navigate to https://hub.docker.com and go to your profile to see your newly uploaded image.

    Now that your image is on Docker Hub, other developers and operations can use the `docker pull` command to deploy your image to other environments.  

    **Note:** Docker images contain all the dependencies that it needs to run an application within the image. This is useful because we no longer have deal with environment drift (version differences) when we rely on dependencies that are install on every environment we deploy to. We also don't have to go through additional steps to provision these environments. Just one step: install docker, and you are good to go.

