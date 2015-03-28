# Parallel programming on Lisa #
Roughly, parallel computing occurs when a problem is split in different tasks that are executed in
parallel. A program needs to be designed to run in parallel.
Any parallel program makes use of some sort of middle ware that enables communication between
its parallel processes. One of these is  MPI (Message Passing Interface).  On the 64-bit  nodes Lisa
uses a an MPI implementation called Open MPI. Open MPI materializes in a set of libraries,
containing methods for exchanging information between parallel processes in a program.
It is possible to run Octave parallel code on the 64-bit architecture nodes on the  Lisa cluster.1
Although Lisa has a Matlab module installed, only serial jobs can be run, i.e jobs that are executed
on a single node. Of course, one can run several serial jobs in parallel, but a single Matlab program
(job) cannot be split in several processes that  run in parallel.
If you would like to execute parallel programs on Lisa, for the time being you need to run your
coding as Octave. In general, Matlab code is compatible with Octave, but there are a few syntax
differences.  Some of them are  described in this wiki book:
http://en.wikibooks.org/wiki/MATLAB_Programming/Differences_between_Octave_and_MATLA
B


# Short Introduction to Parallel SCEM #
SCEM-UA  is a general purpose global parameter optimization algorithm that provides an estimate
of the most likely parameter set and its underlying posterior probability distribution.
In general, application of global optimization methods to high-dimensional parameter estimation
problems requires the solution of a large number of deterministic model runs. SCEM-UA's
architecture in itself is embarrassingly parallel, i.e it can be decomposed into processes that are
executed in parallel. More exactly, independent model simulations can run on different nodes of a
distributed system. This takes place in two phases using the master-slave paradigm. In the first
phase, the master samples (s) parameter combinations randomly from the prior distribution and
distributes the evaluation of the fitness function for the individuals in the population over the slave
processors. The slaves compute the posterior probabilities and send the results back to the master. In
the second phase, the master sorts the points in decreasing order of the posterior probability
distribution, initializes the starting points of the (k) sequences and builds the corresponding
complexes. Then it generates and distributes the new candidate points to the slaves. Each slave
evolves a different sequence and complex and sends the results to the master. The master unpacks
1As of March 18th 2009, new 64-bit architecture nodes were added to the Lisa cluster. In the end all
nodes will run in AMD64 mode.  On these nodes  OpenMPI is the standard message passing
interface. More details on this topic here:
https://subtrac.sara.nl/userdoc/wiki/lisa/news/amd64-phase1
To log in to the AMD-64 environment, you need to log in using the host name
lisa64.sara.nl.
the complexes back into the set and sorts the points in decreasing order of the posterior probability
distribution, checks the convergence and, if necessary, re-iterates.
Parallel SCEM-UA is the adaptation of the SCEM-UA algorithm implemented in GNU Octave for
multiprocessor distributed systems using  the MPI standard (Open MPI) (enabled in Octave by
MPITB).
If you would like to apply the SCEM algorithm to optimize the parameters of your model, you can
simply do so by providing your model, some initialization parameters, the measurement data and
instructing SCEM where to find them. A detailed description on how to proceed,is provided in the
rest o this document.

# Create an account on the Lisa cluster #

If you do not have  an account on the Lisa cluster, send an e-mail to SARA (hic at sara dot nl) with the
following information:

  * Your name, address, telephone number, e-mail address
  * The faculty and/or subfaculty
  * The project leader (including email address)
  * An estimation of cpu-hours needed (for instance  10 node-hours)
  * A short description of your project (10 lines or less)

# Log in to your Lisa account #

Lisa cluster runs on Linux OS. To connect to your Lisa account from your machine you will need a
SSH client. On Windows, you can use a program called Putty. You can download the latest version
of Putty from http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe.

  1. Run putty.exe
  1. Fill in the host name with lisa64.sara.nl
  1. Make sure SSH is enabled
  1. Click “Open”
  1. Fill in your username and password when prompted.

After the first login, don't forget to change your password, using the passwd command.

# Copying your model files in your Lisa account #

In order to do this this you need a FTP client. On Windows machines you can use WinSCP, which
can be downloaded from http://winscp.net/download/winscp421setup.exe. To install it launch the
executable and follow the steps in the wizard.
To copy the model files to your account:
  1. Launch WinSCP
  1. Login to your FTP account on Lisa:
  1. Click Login (you can also save the session for future use)
  1. Create your model directory in your account
  1. Copy there the files as described in section “Running Parallel SCEM on your model” later in this document.

# Installing Octave and MPITB on Lisa #

Octave with MPITB for the Lisa 64-bit architecture is installed in /home/dgorea/octave.
Very soon it will be available cluster-wide. If you just want to use this installation, you can skip this
section entirely.
However, if you would like to have your own installation, you can follow the following steps to
compile and build octave and MPITB for the 64-bit architecture nodes on Lisa. Note that the $
character denotes your prompt.
  1. Login to your Lisa64 account on lisa64.sara.nl
  1. Download the latest Octave distribution. Replace <x.x.x> with your version, i.e 3.0.3
$
  1. Uncompress the file.
$ tar zxvf octave-<x.x.x>.tar.gz
At this moment you should have a directory octave-<x.x.x> in your home directory.
To check this, type:
$ ls
  1. Create the directory where the octave installation will be (let's call it octave for instance):
$ mkdir octave
    1. Go to the octave-<x.x.x> directory:
$ cd octave-<x.x.x>
  1. Now compile and install Octave in the newly created directory. This usually takes a while
and produces lots of output messages. Just type the following sequence of commands,
replacing <your account> with your Lisa username.
$ .configure –prefix=/home/<your account>/octave
...lots of output
$ make
...lots of output
$ make install
$ make clean
    1. You can delete the downloaded archive and the directory with the distribution if you like:
$ cd ..
$ rm -r octave-<x.x.x>
$ rm octave-<x.x.x>.tar.gz
8. Download the latest MPITB distribution
9. Go to the  octave installation directory
$ cd octave
10.Uncompress the downloaded MPITB here:
$ tar jxvf ~/mpitb-<...>.tar.bz2
You should have now a directory called mpitb. You can check this by typing:
$ ls
11. In order to install MPITB you need to set some environment variables so that MPITB
finds Octave and Open MPI. Just run this:
$ module load openmpi/gnu
$ export PATH=/home/<your account>/octave/bin:$PATH
12.Now you need to compile and install MPITB in that directory, for 64-bit architectures, the
existing version of Open MPI and  Octave. First remove the existing binaries:
$ cd mpitb
$ rm -r DLD**Then create the new installation directory corresponding to the your octave version and the
existing ompi and gcc:
$ mkdir DLD-oc
Create a link to that directory:
$ ln -s DLD-oct<x.x.x>-ompi1.3-gcc4.3.2-x86\_64 DLD
$ cd src
Make sure that Makefile.inc points to the Open MPI makefile:
$ rm Makefile.inc
$ ln -s Makefile.inc.OMPI
Run make:
$ make
....lots of output
You should be done now.**
