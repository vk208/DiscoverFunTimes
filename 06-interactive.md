# Running interactive jobs on Discovery

Okay, let us say we want to use a node interactively:

If you want to take advantage of Discovery while also using a graphical software (such as `GaussView`), you need to start by logging in with X11 forwarding:

	$ ssh -X surname.f@discovery2.neu.edu

You can find more details about loading X11 [here](01-logging-in.md).

Remember how we do not want to do anything resource intensive on the log in nodes (discovery2 and discovery4)?
Doing so would cause everyone difficulties, and ruffle some jimmies.
Requesting an interactive job and logging in to a compute node is the solution.

First we need to request a compute node to use.

	$ salloc -N 1  --exclusive -p ht-10g
	salloc: Granted job allocation 649056

This will request exclusive use of a free node on partition `ht-10g`.
(It would make sense to check the queues before choosing a partition, for example `sinfo | grep idle` will show how many idle nodes are in each partition.
	One of the `ser-par-10g-` partitions is usually a good bet.)

To find out which node you were allocated by Slurm:

	$ squeue -u <username>

where <username> is your username, eg. husky.id.
For example, it will look like this:

	         JOBID PARTITION     NAME        USER ST       TIME  NODES NODELIST(REASON)
	        649056    ht-10g     bash  <username>  R       0:08      1 compute-0-006


Then log in to your shiny new compute node (if you did not log in with a `-X`, `$ exit`, then log in again)!
This tells us our job 649056 is running on node compute-0-006.
Then log in to your shiny new compute node!

	$ ssh -X compute-0-006

Sometimes, it may prompt you for a password. Just Ctrl-C and try to `ssh -X <compute-node>` again.
Note that if you didn't use `-X` when sshing in to discovery login node from your computer, you shouldn't use `-X` when sshing from the login node to the compute node.

Now you're on an interactive session - now let's revisit matplotlib!

	[<user_name>@compute-0-006 ~]$


## Using matplotlib
We managed to install matplotlib before in the [python packages tutorial](04-python-packages.md), but can we use it?

First load `ipython` then try `from matplotlib import pyplot as plt` as we are used to doing.

```
$ ipython
Python 3.5.2 (default, Sep 19 2016, 11:10:34)
Type "copyright", "credits" or "license" for more information.

IPython 5.1.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]: from matplotlib import pyplot as plt
```

This will give a long error message, with the clue `# If this fails your Python may not be configured for Tk`.  Googling this will eventually lead you to try the following
, but I'll save you some time. **You have to `exit()` your `ipython` session and start a new one**  (`$ ipython`) because you have to tweak the matplotlib configuration *before* importing pyplot for the first time:

```python
In [1]: import matplotlib

In [2]: matplotlib.use('Agg')

In [3]: from matplotlib import pyplot as plt
```

OK so far - no errors if you did the [python packages tutorial](04-python-packages.md)!
What we did is change the 'back end' that matplotlib uses, because Discovery is not set up with the default 'Tk', but does have 'Agg' ([don't ask me what these are](http://lmgtfy.com/?q=what+are+Tk+and+Agg)). You'll have to do this in any scripts that use matplotlib. Now let's try to make a plot.

```python
In [4]: plt.plot(range(5))
Out[4]: [<matplotlib.lines.Line2D at 0x7fc098b02a90>]
```

OK, but how do we see it?

```python
In [5]: plt.show()
```

Don't see anything? If you had installed an X11 window system and used X11 forwarding (`ssh -X`) then you would have a local window appear.
(This is where you would find out that you need to use -Y instead of -X, but practice makes perfect!)
But when running a script on a remote computer it's usually more helpful to just save the figure to a file and retrieve it later:

```python
In [6]: plt.savefig("my_figure.pdf")

In [7]: exit()
```

Now look for the file:

```
$ ls *.pdf
my_figure.pdf
```

OK - it's there!  You can retrieve it using some SCP or SFTP commands, or preferably a client with a nice GUI (my favorite is currently [Forklift](http://www.binarynights.com/forklift/) but there are many good free ones to choose from. I used [Cyberduck](https://cyberduck.io/) for a while.
FileZilla is commonly mentioned.)

Once you are done with your interactive session, log out of the compute node
by typing `exit` and then release the allocation using `scancel <jobID>`, e.g.:

```
r.west@compute-0-006 ~]$ exit
logout
Connection to compute-0-006 closed.
[r.west@discovery2 ~]$ scancel 649056
salloc: Job allocation 649056 has been revoked.
Hangup
[r.west@discovery2 ~]$
```

## The quick way?

Not sure why this isn't in the instructions provided by research computing for how to use Discovery, but if at the login node you just type:

		$ srun --pty ipython

it will allocate you some resource, wait until it's available (you may experience a pause), then run ipython in "pseudo terminal mode". All your tab completion, syntax highlighting, terminal wizardry etc. seem to work naturally.
We can check which compute node we're running like this:

```
$ srun --pty ipython
Python 3.5.2 (default, Sep 19 2016, 11:10:34)
Type "copyright", "credits" or "license" for more information.

IPython 5.1.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]: import subprocess

In [2]: subprocess.check_output('hostname')
Out[2]: b'compute-0-013\n'
```

We used the python function [`subprocess.check_output`](https://docs.python.org/3/library/subprocess.html#subprocess.check_output) to run the linux program [`hostname`](https://linux.die.net/man/1/hostname) and capture the result, which is `compute-0-013` in this case. But when I `exit()` from iPython I am back on the login node.  You could add other options to the `srun` command to specify the partition, etc. if you want.

---
Next: 7. [Submitting an array of jobs.](07-arrays.md)
