# glm_benchmarks

![CI](https://github.com/Quantco/glm_benchmarks/workflows/CI/badge.svg)

Python package to benchmark GLM implementations. 

[Link to Google Sheet that compares various existing implementations.](https://docs.google.com/spreadsheets/d/1C-n3YTzPR47Sf8M04eEaX4RbNomM13dk_BZaPHGgWXg/edit)

[Link to Google Doc that compares the top contenders for libraries to improve.](https://docs.google.com/document/d/1hjmagUAS-NkUnD1r9Oyc8yL5NpeLHUyxprAIWWaVNAs/edit)

[Link to google doc that discusses some of the optimizations and improvements we have made](https://docs.google.com/document/d/1wd6_bV9OUFjqc9WGtELDJ1Kdv1jrrticivd50POTeqo/edit)


## Running the benchmarks

After installing the package, you should have two CLI tools: `glm_benchmarks_run` and `glm_benchmarks_analyze`. Use the `--help` flag for full details. Look in `src/quantcore/glm/problems.py` to see the list of problems that will be run through each library.

To run the full benchmarking suite, just run `glm_benchmarks_run` with no flags. 

For a more advanced example: `glm_benchmarks_run --problem_name narrow-insurance-no-weights-l2-poisson --library_name sklearn-fork --storage dense --num_rows 100 --output_dir mydatadirname` will run just the first 100 rows of the `narrow-insurance-no-weights-l2-poisson` problem through the `sklearn-fork` library and save the output to `mydatadirname`. This demonstrates several capabilities that will speed development when you just want to run a subset of either data or problems or libraries. 

The `--problem_name` and `--library_name` flags take comma separated lists. This mean that if you want to run both `sklearn-fork` and `glmnet-python`, you could run `glm_benchmarks_run --library_name sklearn-fork,glmnet-python`.

The `glm_benchmarks_analyze` tool is still more a sketch-up and will evolve as we identify what we care about.

Benchmarks can be sped up by enabling caching of generated data. If you don't do this, 
you will spend a lot of time repeatedly generating the same data set. If you are using
Docker, caching is automatically enabled. The simulated data is written to an unmapped
directory within Docker, so it will cease to exist upon exiting the container. If you
are not using Docker, to enable caching, set the GLM_BENCHMARKS_CACHE environment
variable to the directory you would like to write to.

We support several types of matrix storage, passed with the argument "--storage". 
"dense" is the default. "sparse" stores data as a csc sparse matrix. "cat" splits
the matrix into a dense component and categorical components. "split0.1" splits the
matrix into sparse and dense parts, where any column with more than 10% nonzero elements
is put into the dense part, and the rest is put into the sparse part.

## Docker

To build the image, make sure you have a functioning Docker and docker-compose installation. Then, `docker-compose build work`.

To run something, for example: `docker-compose run work glm_benchmarks_run --help`

---
**NOTE FOR MAC USERS**

On MacOS, docker cannot use the "host" `network_mode` and will therefore have no exposed port. To use a jupyter notebook, you can instead start the container with `docker-compose run -p 8888:8888 workmac`. Port 8888 will be exposed and you will be able to access Jupyter.

---

## Library examples:

glmnet_python: see https://bitbucket.org/quantco/wayfairelastpricing/tests/test_glmnet_numerical.py
H2O: https://github.com/h2oai/h2o-tutorials/blob/master/tutorials/glm/glm_h2oworld_demo.py

## Profiling

For line-by-line profiling, use line_profiler `kernprof -lbv src/quantcore/glm/cli_run.py --problem_name narrow-insurance-no-weights-l2-poisson --library_name sklearn-fork`

For stack sampling profiling, use py-spy: `py-spy top -- python src/quantcore/glm/cli_run.py --problem_name narrow-insurance-no-weights-l2-poisson --library_name sklearn-fork`

## Memory profiling

To create a graph of memory usage:
```
mprof run --python -o mprofresults.dat --interval 0.01 src/quantcore/glm/cli_run.py --problem_name narrow-insurance-no-weights-l2-poisson --library_name sklearn-fork --num_rows 100000
mprof plot mprofresults.dat -o prof2.png
```

To do line-by-line memory profiling, add a `@profile` decorator to the functions you care about and then run:
```
python -m memory_profiler src/quantcore/glm/cli_run.py --problem_name narrow-insurance-no-weights-l2-poisson --library_name sklearn-fork --num_rows 100000
```