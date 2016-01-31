# QA #
  1. Is this the code used in the ecml paper?
```
Generally, this is another version different from the version we used in the paper. 
For compute_distance and distance_to_similarity, they are new implementations based on MPICH2 while the ecml paper used mapreduce.
For evd and kmeans, the core algorithm is the same as the paper but data input and output part are a little different.
Also, we do not have timing statistics for all the steps in opensource version. 
```

# Known Problems #
  1. The output phase of compute\_distance and distance\_to\_similarity is not optimized so that outputting data to disk is a bit slow for these two steps.