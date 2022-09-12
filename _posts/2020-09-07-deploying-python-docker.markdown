---
title: "Deploying Python Code to Docker, the Easy Way"
header:
  image: /assets/images/2020-09-07-deploying-python-docker/docker.png
toc: true
toc_sticky: true
categories:
  - Guides
tags:
  - Programming
---

After teaching myself Python through a few projects, I can finally 24/7 host them on something other than an old laptop: my new virtulization server (post coming soonish). I heard a few good things about docker, so I spun up a headless ubuntu server VM and installed docker with portainer. However, I ran into one big problem. All the guides I could find online about deploying python to docker all either involve single-file programs or programs with "proper python project structure" and let's face it, no one does that.

Enter, this guide. Below will be a super easy, basically copy-paste way to deploy basically and super bad python code you could possibly write to a docker container. For this example, I will be using a discord bot project that you can find [here](https://github.com/Jellayy/CheeseBot).

## Make a requirements.txt file:

Okay, I know I said this would work for any 2am slapped together code, and while you can use this method without a requirements file, its a lot more tedious and a requirements file is super easy to make so there's not really any excuse to not have one.

Hopefully, you had enough sense to at least use some sort of virtual python enviornment to work in. If so, simply go to the root directory of your project and use:

{% highlight bash %}
$ pip freeze > requirements.txt
{% endhighlight %}

## Dockerfile:

Here's where the real meat is. Setting up our dockerfile "correctly" is what's going to allow us to run basically anything with absolutely no regard for project structure whatsoever. Now we both know why you're here, I'm not gonna sit here using a paragraph to explain each line in this file. I you really want an explination for some reason, I'll add comments before each line. But for those of you who are here to just copy paste the code blocks here you go, just make sure to change the path to the file you want to run when the container starts.

{% highlight dockerfile %}
# Tell Docker that you're using python 3
# Change to whatever version you need here: https://hub.docker.com/_/python
FROM python:3

# Make the directory where your code will sit in the container
WORKDIR /app

# Install dependencies from your requirements file
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy your code to the directory we made
COPY ./ .

# Tell python to use the directory we made as the root directory
ENV PYTHONPATH /app

# Run your program
# Don't forget to change the second argument to your starting file's path
CMD ["python", "./change_me.py"]
{% endhighlight %}

## Deploying:

Since we put all of our work into making the most forgiving dockerfile ever, this part is pretty easy.

First we have to build our image so that it can be run in a container:

{% highlight bash %}
$ docker build -t IMAGE-NAME PATH/
{% endhighlight %}

Then if your image successfully builds its as simple as a:

{% highlight bash %}
$ docker run IMAGE-NAME
{% endhighlight %}

You can add any extra arguments to your run command as needed for exposing ports, etc. However, if you know what you're doing there, you probably don't need to be reading this article.
