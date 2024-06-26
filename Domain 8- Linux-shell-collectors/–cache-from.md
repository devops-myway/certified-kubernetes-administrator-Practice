https://lipanski.com/posts/speed-up-your-docker-builds-with-cache-from

##### Speed up your Docker builds with –cache-from
Using the Docker cache efficiently can result in significantly faster build times.
In some environments though, like CI/CD systems, individual builds happen independent of each other and the build cache is never preserved.
Every build starts from zero which can be slow and wasteful.
All previously built layers are cached and can be reused.

As long as you’re pushing images to a remote registry, you can always use a previously built image as a cache layer for a new build
ou can achieve this by setting the --cache-from option on the docker build call.
##### How to Leverage the Docker Build Cache
If you build the image the first time, you see that it takes quite some time, in my case 11.3s
[+] Building 11.3s (7/7) FINISHED

you execute it again, and you benefit from the Docker build cache:
Building 1.1s (7/7) FINISHED
the image build is very fast because it can reuse all previously built images.
``````sh
vi Dockerfile

FROM nginx:1.21.6
# Update all packages
RUN apt-get update && apt-get -y upgrade
# Use a custom startpage
RUN echo '<html><bod>My Custom Startpage</body></html>' > /usr/share/nginx/html/index.html

 docker build -t my-custom-nginx .

``````
##### customize your startpage in the Dockerfile
Building 2.1s (7/7) FINISHED
it reused the intense 2nd build step and did not update operation system dependencies.

``````sh
FROM nginx:1.21.6
# Update all packages
RUN apt-get update && apt-get -y upgrade
# Use a custom startpage
RUN echo '<html><bod>New Startpage</body></html>' > /usr/share/nginx/html/index.html

docker build -t my-custom-nginx .

``````
##### How to Use the Docker Build --no-cache Option
``````sh


``````
##### 
``````sh


``````
##### 
``````sh


``````
