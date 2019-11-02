.. _selfmedicate:

Self-Medicate
=============

We've developed a tool called `"Self-Medicate" <https://github.com/nre-learning/antidote-selfmedicate>`_ that
is designed to run the Antidote platform on your laptop. The primary use case for doing this is to work on
the :ref:`curriculum <contrib-curriculum>` locally before pushing to a branch.

The preferred method for running self-medicate is through :ref:`Vagrant <selfmedicate-vagrant>`. Please first follow those
instructions for starting this environment, and when you have a working SSH connection to a properly provisioned
virtual machine with ``vagrant ssh``, resume here.

Starting Self-Medicate
----------------------

Within this repo, the ``selfmedicate.sh`` script is our one-stop shop for managing the development environment. This script
interacts with minikube for us, so we don't have to. Only in rare circumstances, such as troubleshooting
problems, should you have to interact directly with minikube. This script has several subcommands:

.. CODE::

    ./selfmedicate.sh -h
    Usage: selfmedicate.sh <subcommand> [options]
    Subcommands:
        start    Start local instance of Antidote
        reload   Reload Antidote components
        stop     Stop local instance of Antidote
        resume   Resume stopped Antidote instance

    options:
    -h, --help                show brief help

To initially start the selfmedicate environment, use the ``start`` subcommand, like so:

.. CODE::

    ./selfmedicate.sh start

The output of this script should be fairly descriptive, but a high-level overview of the four tasks
accomplished by the ``selfmedicate`` script in this stage is as follows:

1. ``minikube`` is instructed to start a Kubernetes cluster with a variety of optional arguments that
   are necessary to properly run the Antidote platform
2. Once a basic Kubernetes cluster is online, some additional infrastructure elements are installed. These
   include things like Multus and Weave, to enable the advanced networking needed by lessons.
3. Platform elements like ``syringe`` and ``antidote-web`` are installed onto the minikube instance.
4. Common and large images, like the ``vqfx`` and ``utility`` images
   (specified in ``PRELOADED_IMAGES``) are pre-emptively downloaded to the
   minikube instance, so that you don't have to wait for these to download when you try to spin up a lesson.
5. Once all the above is done, the script will ask for your sudo password so it can automatically add an entry
   to ``/etc/hosts`` for you. Once this is done, you should be able to access the environment at the URL
   shown.

.. WARNING::

    Each of these steps are performed in sequence, and will wait for everything to finish before moving on to the
    next step. This script is designed to do as much work as possible up-front, so that your development experience
    can be as positive as possible. As a result, the first time you run this command can take some time. BE PATIENT.
    Also note that if you destroy your minikube instance, you'll need to redo all of the above. If you want to just
    temporarily pause your environment, see the section below on the ``stop`` and ``resume`` subcommands.

The below screenshot shows this command in action, for your reference. You should see more or less the same thing
on your environment.

.. image:: /images/selfmedicate.png

Once this is done, the environment should be ready to access at the URL shown by the script.

Iterating on Lessons
--------------------

One of the biggest use cases for running ``selfmedicate`` is to provide a local instance of the antidote platform for
building and testing curriculum contributions. As was briefly mentioned in the ``start`` section above, the ``selfmedicate``
script takes care of mapping the files on your local filesystem into minikube and again into the Syringe pod to ensure
it sees the lessons you're working on.

This means you can work on lessons all on your local machine without having to bother editing environment variables or
committing your content to a Git repository.

Once you have a working antidote installation according to the previous section, you'll notice that the web portal shows the lessons
as they existed when you initially started the platform. If you want to apply any changes you've made locally, you need to run the
``reload`` subcommand of the ``selfmedicate`` script:

.. code::

    ./selfmedicate.sh reload

This command will take care of restarting Syringe, so that it can reload the content you've changed on your filesystem.

Pausing and Resuming Environment
--------------------------------

As mentioned above, if you destroy the minikube environment, you'll need to perform the ``start`` command all over again.
However, it would be nice to be able to stop the environment temporarily, and resume later without installing everything
over again from scratch.

Fortunately, the ``stop`` and ``resume`` subcommands do just this for us. To stop/pause the environment, run:

.. code::

    ./selfmedicate.sh stop

To resume, run:

.. code::

    ./selfmedicate.sh resume

The ``resume`` command is important to run, since this re-executes minikube with the optional arguments needed
by Antidote, so make sure to use this, rather than trying to use ``minikube start`` directly.

Troubleshooting Self-Medicate
-----------------------------

The vast majority of all setup activities are performed by the ``selfmedicate`` script. The idea is that this
script shoulders the burden of downloading all the appropriate software and building is so that you can
quickly get to focusing on lesson content. However, issues can still happen. This section is meant to direct
you towards the right next steps should something go wrong and you need to intervene directly.

.. warning::

    The ``selfmedicate`` script is designed to make it easy to configure a local minikube environment
    with everything related to Antidote installed on top. However, you'll always be well-served by
    becoming familiar with ``minikube`` or even Kubernetes itself so that you are more able to troubleshoot
    the environment when things go wrong.

Before asking for help, please first answer these questions:

- Have you read this page and the :ref:`Self-Medicate Vagrant <selfmedicate-vagrant>` doc in its entirety?
- Have you run a ``git pull`` on the latest ``master`` branch of the Self-Medicate and Curriculum repos?
- Have you looked at the :ref:`Common Self-Medicate Issues <common-selfmedicate-issues>` below?

If you answered "yes" to all of these questions, please `open an issue on the 
selfmedicate repository <https://github.com/nre-learning/antidote-selfmedicate/issues/new>`_ and ensure you
follow the instructions provided there.

The self-medicate tool comes with a ``debug`` subcommand which runs a series of commands for extracting information
that will be needed for any troubleshooting activity. To run this subcommand and place all output in a file called
``selfmedicatedebug.txt``, run the below:

.. CODE::

    ./selfmedicate.sh debug > selfmedicatedebug.txt

When opening an issue on the Self-Medicate repository linked above, this information will be required to
move forward with troubleshooting any issues, so please `create a new Github Gist <https://gist.github.com/>`_
and paste the link into the relevant spot in your issue. **Please do not paste the debug contents into a
Github Issue directly**.

.. _common-selfmedicate-issues:

Common Self-Medicate Issues
---------------------------

.. NOTE::

    All of the steps below assume a working :ref:`Vagrant <selfmedicate-vagrant>` environment, and a working
    persistent connection to this virtual machine via ``vagrant ssh``.

Cannot Connect to the Web Front-End
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It's likely that the pods for running the Antidote platform aren't running yet. Try getting the current pods:

.. code::

    ~$ kubectl get pods
    NAME                                        READY   STATUS    RESTARTS   AGE
    antidote-web-99c6b9d8d-pj55w                2/2     Running   0          12d
    nginx-ingress-controller-694479667b-v64sm   1/1     Running   0          12d
    syringe-fbc65bdf5-zf4l4                     1/1     Running   4          12d

You should see something similar to the above. The exact pod names will be different, but you should see the same
numbers under the ``READY`` column, and all entries under the ``STATUS`` column should read ``Running`` as above.

In some cases the ``STATUS`` column may read ``ContainerCreating``. In this case, it's likely that the images for each pod
is still being downloaded to your machine. You can verify this by "describing" the pod that's not ``Ready`` yet:

.. code::

    kubectl describe pods -n=kube-system kube-multus-ds-amd64-ddxqc
    Name:               kube-multus-ds-amd64-ddxqc
    ....truncated....
    Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  19s   default-scheduler  Successfully assigned kube-system/kube-multus-ds-amd64-ddxqc to minikube
    Normal  Pulling    17s   kubelet, minikube  pulling image "nfvpe/multus:latest"

In this example, we're still waiting for the image to download - the most recent event indicates that the image is being pulled.
The ``selfmedicate.sh`` script has some built-in logic to wait for these downloads to finish before moving to the next step,
but in case that doesn't work, this can help you understand what's going on behind the scenes.

If you're seeing something else, it's likely that something is truly broken, and you likely won't be able to get the environment
working without some kind of intervention. Please `open an issue on the antidote-selfmedicate repository <https://github.com/nre-learning/antidote-selfmedicate/issues/new>`_
with a full description of what you're seeing.

Lesson Times Out While Loading
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's say you've managed to get into the web front-end, and you're able to navigate to a lesson, but the lesson just
hangs forever at the loading screen. Eventually you'll see some kind of error message that indicates the lesson timed
out while trying to start.

This can have a number of causes, but one of the most common is that the images used in a lesson failed to download within
the configured timeout window. This isn't totally uncommon, since the images tend to be fairly large, and on some internet
connections, this can take some time.

There are a few things you can try. For instance, ``kubectl describe pods <pod name>``, as used in the previous section,
can tell you if a given pod is still downloading an image.

We can also use docker commands to check the list of docker images that have been successfully pulled:

.. code::

    docker image list
