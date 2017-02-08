## Intro to CI/CD Hands-on Lab

For this hands-on lab, we're going to build a simply CI/CD pipeline using GitHub and Docker Hub and Play with Docker. We will...

- Host a project in GitHub
- Configure the project to build and deploy automatically to Docker Hub
- Run a deployment stack locally
- Update the app
- Roll out the update


## Requirements

- A GitHub account
- A Docker Hub account
- Docker 1.13+ installed locally
- Git installed locally




## Setting up the GitHub project

You're welcome to use whatever project you want, but feel free to fork my [mikesir87/cats](https://github.com/mikesir87/cats).

1. Go to https://github.com/mikesir87/cats
2. Click the "Fork" button at the top-right corner.
3. Clone the repository locally (doesn't matter SSH vs HTTP)
   ```bash
   git clone [your repository path]
   ```


## Configuring an Automated Build on Docker Hub

1. Login/create an account at https://hub.docker.com/
2. In the top-right corner, hit the **Create** link and hit the **Create Automated Build** option in the dropdown.
3. Choose the GitHub option for your build's source.
4. If you haven't yet, you will be prompted to link your GitHub account to Docker Hub.
5. Once back on Docker Hub, navigate to the repository you setup earlier.
6. Configure the repository namespace/name as desired. This will be the name in which you will pull the image. For example, if I set the namespace to *mikesir87* and name to *cats*, the image would be found at **mikesir87/cats**.


## Trigger the first build

Docker Hub is now configured to receive webhook notifications whenever you push code to your repository on GitHub. However, it doesn't trigger build once setup.  So... let's do that!

1. In your project, go to the **Build Settings** tab.
2. Click the **Trigger** button.
3. Then, go to the **Build Details** tab. You should see that the build has been queued.
4. You can drill into the build to see the console output and progress.
5. Once done, you should see a build with the tag **latest**


## Launch a stack with your newly built image

In the `mikesir87/cats` repository is a `docker-compose-stack.yml` file that we will use to launch a new stack.

1. Modify the `docker-compose-stack.yml` to point to your new image... `mikesir87/cats` to `your_namespace/your_image_name`
2. Open up https://play-with-docker.com/
3. Create a new instance by hitting the **New Instance** in the left corner.
4. Create a `docker-compose-stack.yml` file and paste in the contents of your updated `docker-compose-stack.yml`
   ```bash
   vi docker-compose-stack.yml
   [Press 'i' to enter INSERT mode; paste contents; hit ESC; type ':x'; press enter]
   ```
   You may have to re-align the contents
5. Start a Swarm manager
   ```bash
   docker swarm init
   ```
   This will most likely fail with an error stating it doesn't know what address to use.  Copy the `10.0.*` address and run...
   ```bash
   docker swarm init --advertise-addr=[the address]
   ```
   and copy the `docker swarm join` command.
6. Create another instance using the button in the left sidebar.
7. Run the `docker swarm join` command.
8. Feel free to create another instance.
9. On your manager node (has a blue user icon in the list of instances), launch the stack...
   ```bash
   docker stack deploy -c docker-compose-stack.yml cats
   ```
10. After a brief moment, you should see results when checking on the service 
   ```bash
   $ docker service ls
   ID            NAME          MODE        REPLICAS  IMAGE
   c7tfd9aufcxk  cats_cat-app  replicated  3/3       mikesir87/cats:latest
   ```
11. You should notice a _5000_ link at the top of the page, on the same line as the IP. Clicking that will open that instance. Notice that refreshing will rotate through the running containers (yah routing mesh!!).



## Make a code change

1. Make a change to the `templates/index.html`. Feel free to change the body background color to another color.
2. Commit and push the code.
   ```bash
   git add .
   git commit
   git push
   ```
3. If you jump into the Docker Hub **Build Details** panel, you should see a build trigger shortly.


## Update your service
1. In the Play with Docker panel, go to your Swarm Manager (instance with blue user icon).
2. Update the service to use the latest image (swap out `mikesir87/cats` with the name of your image):
   ```bash
   docker service update cats_cat-app --image=mikesir87/cats:latest --force
   ```
3. Wait for the deploy to roll out!  If you refresh, you might get the new version or the old.



## Clean up

1. Simply delete the instances in Play with Docker and close your session.



## How to Improve/Actually Use This

- In most environments, you don't want to just use the `latest` tag. You'll most likely want to create tags for each version (`1.0.0`, `1.0.1`, etc.)
- For projects that you don't public...
  - Use something other than GitHub (with public repos) for your code
  - Use GitLab, Jenkins, Travis, etc. to do the build. All can watch repos or receive webhooks from your repos
  - Use either private Docker Hub repos or another private repo solution (Nexus, AWS ECR, etc., or run your own)

