pspectralclustering is a parallel C++ implementation of Parallel Spectral Clustering.  We are expecting to present a highly optimized parallel implemention of all the steps of spectral clustering.  We use [PARPACK](http://www.caam.rice.edu/~kristyn/parpack_home.html) as underlying eigenvalue decomposition package and [F2C](http://www.netlib.org/f2c/) to compile fortran code.

If you wish to publish any work based on pspectralclustering, please cite our paper as:

The bibtex format is
```
@article{Chen11,
  author = {Wen-Yen Chen and Yangqiu Song and Hongjie Bai and Chih-Jen Lin and Edward Y. Chang},
  title = {Parallel Spectral Clustering in Distributed Systems},
  journal = {IEEE Transactions on Pattern Analysis and Machine Intelligence},
  volume = {33},
  number = {3},
  pages = {568-586},
  year = {2011}
}  
```

You can also download our paper [here](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=5444877&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D5444877).

If any problems using it, please send mail to pspectralclustering@googlegroups.com