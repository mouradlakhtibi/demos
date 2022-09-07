### modify a running container website
Clone the Lab’s GitHub Repo
Use the following command to clone the lab’s repo from GitHub (you can click the command or manually type it). and go to dir  called linux_tweet_app.

    git clone https://github.com/mouradlakhtibi/demos

 Modify a running website
When you’re actively working on an application it is inconvenient to have to stop the container, rebuild the image, and run a new version every time you make a change to your source code.

One way to streamline this process is to mount the source code directory on the local machine into the running container. This will allow any changes made to the files on the host to be immediately reflected in the container.

We do this using something called a bind mount.

When you use a bind mount, a file or directory on the host machine is mounted into a container running on the same host.

Start our web app with a bind mount
Let’s start the web app and mount the current directory into the container.

In this example we’ll use the --mount flag to mount the current directory on the host into /usr/share/nginx/html inside the container.

Be sure to run this command from within the linux_tweet_app directory on your Docker host.

 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
 $DOCKERID/linux_tweet_app:1.0
Remember from the Dockerfile, /usr/share/nginx/html is where the html files are stored for the web app.

The website should be running.

Modify the running website
Bind mounts mean that any changes made to the local file system are immediately reflected in the running container.

Copy a new index.html into the container.

The Git repo that you pulled earlier contains several different versions of an index.html file. You can manually run an ls command from within the ~/linux_tweet_app directory to see a list of them. In this step we’ll replace index.html with index-new.html.

 cp index-new.html index.html
Go to the running website and refresh the page. Notice that the site has changed.

If you are comfortable with vi you can use it to load the local index.html file and make additional changes. Those too would be reflected when you reload the webpage. If you are really adventurous, why not try using exec to access the running container and modify the files stored there.

Even though we’ve modified the index.html local filesystem and seen it reflected in the running container, we’ve not actually changed the Docker image that the container was started from.

To show this, stop the current container and re-run the 1.0 image without a bind mount.

Stop and remove the currently running container.

 docker rm --force linux_tweet_app
Rerun the current version without a bind mount.

 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 $DOCKERID/linux_tweet_app:1.0
Notice the website is back to the original version.

Stop and remove the current container

docker rm --force linux_tweet_app
Update the image
To persist the changes you made to the index.html file into the image, you need to build a new version of the image.

Build a new image and tag it as 2.0

Remember that you previously modified the index.html file on the Docker hosts local filesystem. This means that running another docker image build command will build a new image with the updated index.html

Be sure to include the period (.) at the end of the command.

 docker image build --tag $DOCKERID/linux_tweet_app:2.0 .
Notice how fast that built! This is because Docker only modified the portion of the image that changed vs. rebuilding the whole image.

Let’s look at the images on the system.

 docker image ls
You now have both versions of the web app on your host.

 REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
 <docker id>/linux_tweet_app    2.0                 01612e05312b        16 seconds ago      108MB
 <docker id>/linux_tweet_app    1.0                 bb32b5783cd3        4 minutes ago       108MB
 mysql                          latest              b4e78b89bcf3        2 weeks ago         412MB
 ubuntu                         latest              2d696327ab2e        2 weeks ago         122MB
 nginx                          latest              da5939581ac8        3 weeks ago         108MB
 alpine                         latest              76da55c8019d        3 weeks ago         3.97MB
Test the new version
Run a new container from the new version of the image.

 docker container run \
 --detach \
 --publish 80:80 \
 --name linux_tweet_app \
 $DOCKERID/linux_tweet_app:2.0
Check the new version of the website (You may need to refresh your browser to get the new version to load).

The web page will have an orange background.

We can run both versions side by side. The only thing we need to be aware of is that we cannot have two containers using port 80 on the same host.

As we’re already using port 80 for the container running from the 2.0 version of the image, we will start a new container and publish it on port 8080. Additionally, we need to give our container a unique name (old_linux_tweet_app)

Run another new container, this time from the old version of the image.

Notice that this command maps the new container to port 8080 on the host. This is because two containers cannot map to the same port on a single Docker host.

 docker container run \
 --detach \
 --publish 8080:80 \
 --name old_linux_tweet_app \
 $DOCKERID/linux_tweet_app:1.0
View the old version of the website.

Push your images to Docker Hub
List the images on your Docker host.

 docker image ls -f reference="$DOCKERID/*"
You will see that you now have two linux_tweet_app images - one tagged as 1.0 and the other as 2.0.

 REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
 <docker id>/linux_tweet_app    2.0                 01612e05312b        3 minutes ago       108MB
 <docker id>/linux_tweet_app    1.0                 bb32b5783cd3        7 minutes ago       108MB
These images are only stored in your Docker hosts local repository. Your Docker host will be deleted after the workshop. In this step we’ll push the images to a public repository so you can run them from any Linux machine with Docker.

Distribution is built into the Docker platform. You can build images locally and push them to a public or private registry, making them available to other users. Anyone with access can pull that image and run a container from it. The behavior of the app in the container will be the same for everyone, because the image contains the fully-configured app - the only requirements to run it are Linux and Docker.

Docker Hub is the default public registry for Docker images.

Before you can push your images, you will need to log into Docker Hub.

 docker login
You will need to supply your Docker ID credentials when prompted.

 Username: <your docker id>
 Password: <your docker id password>
 Login Succeeded
Push version 1.0 of your web app using docker image push.

 docker image push $DOCKERID/linux_tweet_app:1.0
You’ll see the progress as the image is pushed up to Docker Hub.

 The push refers to a repository [docker.io/<your docker id>/linux_tweet_app]
 910e84bcef7a: Pushed
 1dee161c8ba4: Pushed
 110566462efa: Pushed
 305e2b6ef454: Pushed
 24e065a5f328: Pushed
 1.0: digest: sha256:51e937ec18c7757879722f15fa1044cbfbf2f6b7eaeeb578c7c352baba9aa6dc size: 1363
Now push version 2.0.

 docker image push $DOCKERID/linux_tweet_app:2.0
Notice that several lines of the output say Layer already exists. This is because Docker will leverage read-only layers that are the same as any previously uploaded image layers.

 The push refers to a repository [docker.io/<your docker id>/linux_tweet_app]
 0b171f8fbe22: Pushed
 70d38c767c00: Pushed
 110566462efa: Layer already exists
 305e2b6ef454: Layer already exists
 24e065a5f328: Layer already exists
 2.0: digest: sha256:7c51f77f90b81e5a598a13f129c95543172bae8f5850537225eae0c78e4f3add size: 1363
You can browse to https://hub.docker.com/r/<your docker id>/ and see your newly-pushed Docker images. These are public repositories, so anyone can pull the image - you don’t even need a Docker ID to pull public images. Docker Hub also supports private repositories.

Next Step
Check out the introduction to a multi-service application stack orchestration in the Application Containerization and Microservice Orchestration tutorial.
