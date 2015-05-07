# Rooted tree example

This examples evaluates the log-likelihood of the rooted tree presented in the
figure below, by creating a custom post-order traversal that drives the
likelihood computation.

![rooted tree](https://github.com/xflouris/asssets/raw/master/libpll/images/rooted.png)

## Explanation of the example

The program first instantiates a partition using the function call

`partition = pll_create_partition(5, 4, 4, 6, 1, 8, 4, 1, PLL_ATTRIB_ARCH_SSE);`

The parameters of the function (in the order passed) indicate
* the number of tip sequences that will be used for this partition, 
* the extra number of Conditional Likelihood Vectors (CLVs) that should be allocated apart from those created for the tip sequences (typically, number of tips minus one for rooted trees), 
* number of states in the dataset (for instance 4 for nucleotide datasets, 20 for aminoacid datasets),
* the length of the alignment, i.e. the number of sites at the tip sequences,
* how many different substitution models we want to have at one time, 
* the number of probability matrices that should be allocated (typically 2 times the number of tip sequences minus 2),
* number of discrete rate categories (rate heterogeneity),
* number of scale buffers to be allocated,
* attributes that specify what kind of hardware acceleration should be used.

For a more detailed explanation of the function arguments refer to the documentation.

Model parameters are set with the function calls 

`pll_set_frequencies(partition, 0, frequencies);`

`pll_set_subst_params(partition, 0, subst_params, 6);`

`pll_set_category_rates(partition, rate_cats);`


The CLVs at tips are set by calling, for example, 

`pll_set_tip_states(partition, 0, pll_map_nt, "WAAAAB");`

This function sets the sequence for tip 0 (first tip). Note that it is
necessary to specify a map for converting the sequence to partials. The map is
essentially a 256-long array of  `unsigned int` values that map each ASCII
character to a number. The set bits of the number indicate which CLV entries
will be set to 1.  For nucleotide data, the predefined map `pll_map_nt` may be
used, which accounts for degenerate (ambiguous) base-calls as well. Also note
that CLVs for tips are numbered from 0 to _n-1_, where _n_ is the number of tips.
For inner node CLVs you may use any number starting from _n_ upto _n+x-1_, where
_x_ is the second parameter of `pll_create_partition(...)`.

Next, we compute the transition probability matrices for each unique branch
length in the tree by calling the function

`pll_update_prob_matrices(partition, 0, matrix_indices, branch_lengths, 5);`

and which computes the probability matrices at indices `matrix_indices` from
the corresponding branch lengths specified in `branch_lengths`. The last
argument indicates the size of the two arrays. Note that the function will the
probability matrices for all available rate categories. For more information on
this function check the documentation.

The next step is to create a traversal descriptor for driving the likelihood
computation. This is done by allocating a `pll_operation_t` structure which we
fill with information on how to compute each CLV corresponding to an internal
node.

`operations[0].parent_clv_index    = 5;`

`operations[0].child1_clv_index    = 0;`

`operations[0].child2_clv_index    = 1;`

`operations[0].child1_matrix_index = 0;`

`operations[0].child2_matrix_index = 0;`

`parent_clv_index` indicates which CLV we want to update/compute, using the
CLVs specified by `child1_clv_index` and `child2_clv_index`. The probability
matrices used for each child CLV is specified by `child1_matrix_index` and
`child2_matrix_index`, respectively.  Note that the node indices in the tree
illustration directly correspond to the indices used in the example program.

Now we can use the created `pll_operation_t` structure to compute the CLVs by
calling

`pll_update_partials(partition, operations, 4);`

where the last parameter (4) sets the number of operations to be carried in
sequential order (0 to 3).

Finally, we obtain the  log-likelihood of the dataset by a call to

`logl = pll_compute_root_loglikelihood(partition,8,0);`

where the last two parameters specify the

* index of the CLV to be used for integrating the log-likelihood over sites,
* the index of the frequency array to use.

Before exiting, we dispose of the allocated memory by calling

`pll_destroy_partition(partition);`

## Instructions to compile

Before proceeding with the compilation, make sure that `pll.h` is accessible,
by copying it to the current example directory, or modifying the line

`#include "pll.h"`

You will also need to make the shared object `libpll.so` accessible by either
placing it in your system's library directory (typically `/lib` or
`/usr/local/lib`), or by copying it to the current example directory.

Now you may compile the example by running the included Makefile using

`make`

## Instructions to run

Make absolutely sure that the shared object `libpll.so` is accessible by either
placing it in your system's library directory (typically `/lib` or
`/usr/local/lib`). If it is not, you may copy it to a directory (for example
$PWD), and run the command

`export LD_LIBRARY_PATH=$PWD`

Now, run the example by changing to the directory of the compiled file and
typing:

`./unrooted`