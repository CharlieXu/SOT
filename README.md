[![DOI](https://zenodo.org/badge/57388364.svg)](https://zenodo.org/badge/latestdoi/57388364)
# SOT
This is a repository for a C++11 implementation 
of the surrogate based optimization algorithms in 
[https://github.com/dme65/pySOT](https://github.com/dme65/pySOT). 
SOT is designed for  global deterministic optimization of 
computationally expensive black-box objective functions with 
continuous variables where the number of evaluations is 
limited.

Surrogate optimization algorithms generally consist of four 
components:

1. The optimization problem: All of the available information 
about the optimization problem, e.g., dimensionality, 
variable types, objective function, etc.
2. Surrogate model: Approximates the underlying objective 
function. Common choices are RBFs, Kriging, MARS, etc.
3. Experimental design: Generates an initial set of points 
for building the initial surrogate model
4. Adaptive sampling: Method for choosing evaluations after 
the experimental design has been evaluated.

SOT provides abstract classes that describe how these 
components should be implemented and the SOT strategies 
expect the implementations to use these abstract classes 
as base classes. This is an easy way to guarantee that the 
necessary functionality exists.

SOT currently supports synchronous parallel versions of the algorithms 
given in [1], [2], [3]. Paper [1] introduces the immensely 
popular stochastic optimization algorithm SRBF. Paper 
[2] introduces the DYCORS algorithm that is useful for 
high-dimensional problems. Paper [3] introduces DDS that 
is efficient for separable problems, which doesn't use any 
surrogate model.

## Installation

The easiest way to install SOT is using CMake. SOT is a 
header-only library so no .so is being built, but it comes 
with a set of CMake tests that will be compiled. Installing 
SOT can be done using the following five steps:

``` bash
cmake .
make
make doc
make install
make test
```

The first line generates the necessary makefiles and 
finds all of the SOT dependencies. Armadillo is currently 
the only SOT dependency and it can be built through most 
package managers or from the source available at: 
[http://arma.sourceforge.net](http://arma.sourceforge.net). 
We recommend that you build Armadillo with a fast BLAS library, 
such as OpenBLAS. CMake should be able to find both the library 
and the header files once Armadillo has been built. The second 
line will compile all of the SOT tests. SOT is a header-only 
library so no .so will be generated by make. The third line
is optional and will generate the Doxygen documentation. 
The fourth line will move the header files to the default 
location for include, which is /usr/local/include on 
most UNIX systems. The fifth line will run the compiled 
SOT tests and they should all pass.

Note that the fact that SOT is a header-only library makes 
it possible to include the headers directly into your project.

The SOT tests seem to build without any issues on both 
Ubuntu, Linux, OS X, and on Windows under Cygwin.

## Dependencies

SOT currently depends on BLAS, LAPACK, and Armadillo. You
need Doyxgen if you want to generate the documentation.
Armadillo should be linked with a fast BLAS library for 
maximum speed. More information can be found in the 
Armadillo documentation.

## Developers

* Build Status: <a href="https://travis-ci.org/dme65/SOT">
<img src="https://travis-ci.org/dme65/SOT.svg?branch=master"/></a>
* Code Coverage: 
[![codecov](https://codecov.io/gh/dme65/SOT/branch/master/graph/badge.svg)](https://codecov.io/gh/dme65/SOT)
* Doxygen documentation: 
[https://codedocs.xyz/dme65/SOT/](https://codedocs.xyz/dme65/SOT/)

## Examples

The following example code shows how to run SOT 
with the default methods:

``` cpp

#include <sot.h>
using namespace sot;

int main(int argc, char** argv) {
    int dim = 10;
    int maxEvals = 500; // Evaluation budget
    setSeedRandom(); // Set the SOT seed ramdomly

    std::shared_ptr<Problem> data(std::make_shared<Ackley>(dim));
    std::shared_ptr<ExpDesign> slhd(std::make_shared<SLHD>(2*(dim+1), dim));
    std::shared_ptr<Surrogate> rbf(std::make_shared<CubicRBF>(maxEvals, dim, data->lBounds(), data->uBounds()));
    std::shared_ptr<Sampling> dycors(std::make_shared<DYCORS<>>(data, rbf, 100*dim, maxEvals - slhd->numPoints()));

    Optimizer opt(data, slhd, rbf, dycors, maxeval);
    Result res = opt.run();

    std::cout << "Best value found: " << res.fBest() << std::endl;
    std::cout << "Best solution found: " << res.xBest().t() << std::endl;
}
```

SOT expects shared pointers for the base class objects 
that point to implementations of these base classes. 
Ackley is a popular test problem and Ackley implements 
the base class Problem. The SLHD (symmetric Latin 
hypercube design) is one of the most popular experimental 
designs, the Cubic RBF is a very popular surrogate model, 
and DYCORS is a popular adaptive sampling method. In 
addition to these four objects SOT only needs to know 
the evaluation budget in order to run the optimization 
strategy. The results from the run are returned in a 
separate result class.

## Next features to be added

* More surrogate models
* Support for integer variables
* Support for combining adaptive sampling methods
* Support for ensemble surrogate models

## References

[1] Rommel G Regis and Christine A Shoemaker. A stochastic 
radial basis function method for the global optimization 
of expensive functions. INFORMS Journal on Computing, 
19(4): 497–509, 2007.


[2] Rommel G Regis and Christine A Shoemaker. Combining 
radial basis function surrogates and dynamic coordinate 
search in high-dimensional expensive black-box optimization. 
Engineering Optimization, 45(5): 529–555, 2013.


[3] Tolson, Bryan A., and Christine A. Shoemaker. 
Dynamically dimensioned search algorithm for 
computationally efficient watershed model calibration. 
Water Resources Research 43.1 (2007).
