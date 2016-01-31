# Introduction #

Parallel Spectral Clustering must be run in linux 64bit environment with mpich2-1.2.
**NOTE**: there are some problems on 32bit platform and I haven't got time to figure out yet.

# Installation #

  * Parallel Infrastructure is based on MPI. Firstly mpich2 must be installed before running. You can download it from http://www.mcs.anl.gov/research/projects/mpich2/.
    * Install mpich2
      * Download mpich2-1.2.tar.gz
      * Extract it to a path
      * `./configure`
      * `make`
      * `sudo make install`
      * After installing, you will find some binaries and scripts in $PATH. Test by running `mpd` to see if it exists
    * Create password file `~/.mpd.conf` with access mode 600 (rw-------) in home directory. The file should contain a single line `MPD_SECRETWORD=PASSWORD`. Because you may have many  machines, you must do this on each machine.
      * `touch ~/.mpd.conf`
      * `chmod 600 ~/.mpd.conf`
      * `echo "MPD_SECRETWORD=anypassword" > ~/.mpd.conf`
    * Pick one machine as the master and startup mpd(mpi daemon)
      * `mpd --daemon --listenport=55555`
    * Other machines act as slave and must connect to the master
      * `mpd --daemon -h serverHostName -p 55555`
    * Check whether the environment is setup successfully: on master, run `mpdtrace`, you will see all the slaves. If no machines show up, you must check your mpi setup and refer to mpich2 user manual.
  * Download [Parallel Spectral Clustering](http://pspectralclustering.googlecode.com/files/pspectralclustering-beta.tar.gz) package and extract and run `make` in its directory. You will see four binaries generated `compute_distance`/`distance_to_similarity`/`evd`/`kmeans`. We use mpich2 builtin compiler mpicxx to compile, it is a wrap of g++.
# Prepare datafile #
  * Our datafile is stored using a sparse representation, with one instance per line. Each line is a list of features, of the form featureID:featureValue. It is like `<index1>:<value1> <index2>:<value2> ...` Feature index must be in increasing order.
  * Example: Suppose there are two elements, each with two features. The first one is Feature0: 1, Feature1: 2; The second one is Feature0: 100, Feature1: 200. Then the datafile would look like:
```
0:1    1:2
0:100  1:200
```
  * You could also use our sample data file in `testdata/data.txt`. The data file must be put in a shared directory so that every machine could access it.

# Run #
## Step 1: Compute Similarity ##
  * Flags:
    * `--t_nearest_neighbor`: the number of nearest neighbors to keep for the distance matrix. Usually, 100 is enough.
    * `--input`: the input datafile
    * `--output`: the output datafile
  * Sample: `mpiexec -n 32 ./compute_distance --t_nearest_neighbor 10 --input data.txt --output distance.txt`
  * Note: this step is very time consuming. Try to use as many machines as possible if data set is large.
## Step 2: distance\_to\_similarity ##
  * Flags:
    * `--input`: the input distance file
    * `--output`: the output similarity file
  * Sample: `mpiexec -n 2 ./distance_to_similarity --input distance.txt --output similarity.txt`
  * Note: this step is fast. A few machines will be enough.
## Step 3: evd ##
  * Flags:
    * `--input`: the input similarity file
    * `--eigenvalues_output`: the file to output eigenvalues
    * `--eigenvectors_output`: the file to output eigenvectors
    * `--eigenvalue`: the number of clusters desired
    * `--eigenspace`: the eigenspace to compute eigenvalues. Emperically, set this to 2\*eigenvalue or 3\*eigenvalue. Refer to arpack users' guide for details.
    * `--arpack_iterations`: The maximum number of arnoldi iterations to compute eigenvalues. Default 300 usually works.  Refer to arpack users' guide for details.
    * `--arpack_tolerance`: the stopping criterion to stop eigenvalues computing iterations. Default is 0 which is actually not 0 but a very small value related to machine precision. Default usually works . If you wish to make it converge faster and just need approximated eigenvalues, you could set this to a larger value such as 0.0001.
  * Sample: `mpiexec -n 10 ./evd --eigenvalue 100 --eigenspace 300 --input similarity.txt --eigenvalues_output eigenvalues.txt --eigenvectors_output eigenvectors.txt`
  * Note: this step is time consuming and requires communication between distributed computers,  so you need to choose reasonable number of machines. For example: using tens of machines for 300000 dataset is enough.
## Step 4: kmeans ##
  * Flags:
    * `--input`: the eigenvectors input file
    * `--output`: the clustering result
    * `--num_clusters`: the number of clusters. Should be the same with --eigenvalue in evd step.
    * `--kmeans_loop`: the maximum number of loops for kmeans. Default 100 usually works.
    * `--kmeans_threshold`: the stopping criterion for kmeans. Default 1e-3 usually works.
    * `--initialization_method`: `orthogonal_centers` or `random`. Generally, `orthogonal_centers` works better.
  * Sample: `mpiexec -n 4 ./kmeans --num_clusters 100 --input eigenvectors.txt --output result.txt`
  * Note: this step is faster than evd,  so you need to choose reasonable number of machines. For example: using 10 machines for 300000 dataset is enough.
# Check results #
  * Each line is the cluster index assigned to the corresponding instance. The order is the same as the input instance's order.