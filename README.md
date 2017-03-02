# Tensorflow ITP Flyby

We will learn how to use Tensorflow to retrain Google's visual recognition model, Inception, to recognize between 5 different dog kinds (and show how you can use the same process to recognize between any visual categories that you want).

## Follow along

Install [Docker](https://download.docker.com/mac/stable/Docker.dmg). 

Open `Docker.dmg`, copy to Applications, and open the app.

Open the Terminal and make sure that Docker is running:

```
$ docker run hello-world
```

If Docker is running properly you should see this:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
 ```
 
 Download the Tensorflow Docker image, which is a Linux image with Tensorflow pre-installed. 
 
 ```
$ docker run -it gcr.io/tensorflow/tensorflow:latest-devel
 ```
Once the image is downloaded, a container with that image will be created and you will automatically enter a shell within it. Confirm that Tensorflow is installed by running the following (it should return with no errors):
```
$ python -c "import tensorflow"
```

You can now exit the container with `CTRL+D`.

Now that we have an image running Tensorflow, we can proceed to train our network to recognize between 5 kinds of dogs (basset, bluetick, borzoi, chihuahua, redbone). 

Clone this repository to get the training dog images:

```
$ cd ~/
$ git clone https://github.com/agermanidis/tf-flyby
```

We need a way to access our dog images inside our Docker container. To do that, we will run docker with the `-v` argument, which will allow us to map a directory in our machine to a directory inside our Docker container container.

```
$ docker run -it -v ~/tf-flyby:/tf-flyby gcr.io/tensorflow/tensorflow:latest-devel
```

Confirm that you can access the directory from within the container:
```
$ ls /tf-flyby/
README.md  bottlenecks  dogs  inception  label_image.py  retrained_graph.pb  retrained_labels.txt
```

Before starting the training, let's ensure that we have the latest version of Tensorflow inside the container:

```
$ cd /tensorflow
$ git pull
```
We're finally ready to run the retraining script to create a model that can recognize between the different dog kinds:

```
$ python tensorflow/examples/image_retraining/retrain.py --bottleneck_dir=/tf_files/bottlenecks --how_many_training_steps 500 --model_dir=/tf_files/inception --output_graph=/tf_files/retrained_graph.pb --output_labels=/tf_files/retrained_labels.txt --image_dir=/tf-flyby/dogs
```

While we wait for the retraining to finish, let's unpack the arguments of the command:
* `bottleneck_dir`: Since we are only training the last layer of our Inception network, which is called the bottleneck, we can safely cache the output of the network for every image up until the last layer, to speed up the training process. This cache is what's stored in `bottleneck_dir`.
* `how_many_training_steps`: Specifies the number of training iterations that our retraining process goes through. Since we have a small training set, we can keep this to a low value of `500`, but the recommended default is `4000`.
* `model_dir`: Specifies where the trained network is stored.
* `output_graph`: Specifies where the specification of the network, i.e. the graph of operations that the network performs, is stored.
* `output_labels`: Specifies where the labels that our network can recognize, which in this case will be our 5 dog kinds (basset, bluetick, borzoi, chihuahua, redbone), will be stored.
* `image_dir`: Specifies the image directory where our training data is stored.

Because of the small training dataset, we can achieve near-perfect accuracy in our model pretty quickly. Still, training a neural network from scratch to solve the same problem would take hours, if not days.

We can now put our newly trained model to use! Copy this cute borzoi to our `tf-flyby` folder...

![](http://www.pupcity.com/_assets/images/breeds/36_m.jpg)

...and run the `label_image.py` script on it:

```
$ python label_image.py 36_m.jpg
```

If everything works properly, you should see results like these:

```
borzoi (score = 0.99230)
basset (score = 0.00277)
bluetick (score = 0.00180)
chihuahua (score = 0.00160)
redbone (score = 0.00153)
```

Congratulations! You trained your first object recognition network. You can now repeat the same process to recognize anything you want. You just need to download a bunch of images for every category (at least 100 per category is recommended), build a directory structure like the one in the `dogs` folder, and run the retraining script.
