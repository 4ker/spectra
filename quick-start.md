---
layout: page
title: Quick Start
---

# Quick Start

### Common Usage

Spectra is designed to calculate a specified number (`k`) of eigenvalues
of a large square matrix (`A`). Usually `k` is much smaller than the size of matrix
(`n`), so that only a few eigenvalues and eigenvectors are computed, which
in general is more efficient than calculating the whole spectral decomposition.
Users can choose eigenvalue selection rules to pick up the eigenvalues of interest,
such as the largest `k` eigenvalues, or eigenvalues with largest real parts,
etc.

To use the eigen solvers in this library, the user does not need to directly
provide the whole matrix, but instead, the algorithm only requires certain operations
defined on `A`, and in the basic setting, it is simply the matrix-vector
multiplication. Therefore, if the matrix-vector product `A * x` can be computed
efficiently, which is the case when `A` is sparse, Spectra
will be very powerful for large scale eigenvalue problems.

### Key steps

There are two major steps to use the Spectra library:

1. Define a class that implements a certain matrix operation, for example the
matrix-vector multiplication `y = A * x` or the shift-solve operation
`y = inv(A - σ * I) * x`. Spectra has defined a number of
helper classes to quickly create such operations from a matrix object.
See the documentation of
[DenseGenMatProd](https://spectralib.org/doc/classSpectra_1_1DenseGenMatProd.html),
[DenseSymShiftSolve](https://spectralib.org/doc/classSpectra_1_1DenseSymShiftSolve.html), etc.
2. Create an object of one of the eigen solver classes, for example
[SymEigsSolver](https://spectralib.org/doc/classSpectra_1_1SymEigsSolver.html)
for symmetric matrices, and
[GenEigsSolver](https://spectralib.org/doc/classSpectra_1_1GenEigsSolver.html)
for general matrices. Member functions
of this object can then be called to conduct the computation and to retrieve the
eigenvalues and/or eigenvectors.

### Solvers

Below is a list of the available eigen solvers in Spectra:

- [SymEigsSolver](https://spectralib.org/doc/classSpectra_1_1SymEigsSolver.html):
For real symmetric matrices
- [GenEigsSolver](https://spectralib.org/doc/classSpectra_1_1GenEigsSolver.html):
For general real matrices
- [SymEigsShiftSolver](https://spectralib.org/doc/classSpectra_1_1SymEigsShiftSolver.html):
For real symmetric matrices using the shift-and-invert mode
- [GenEigsRealShiftSolver](https://spectralib.org/doc/classSpectra_1_1GenEigsRealShiftSolver.html):
For general real matrices using the shift-and-invert mode,
with a real-valued shift
- [GenEigsComplexShiftSolver](https://spectralib.org/doc/classSpectra_1_1GenEigsRealShiftSolver.html):
For general real matrices using the shift-and-invert mode,
with a complex-valued shift
- [SymGEigsSolver](https://spectralib.org/doc/classSpectra_1_1SymGEigsSolver.html):
For generalized eigen solver for real symmetric matrices

### Examples

Below is an example that demonstrates the use of the eigen solver for symmetric
matrices.

~~~
#include <Eigen/Core>
#include <SymEigsSolver.h>  // Also includes <MatOp/DenseSymMatProd.h>
#include <iostream>

using namespace Spectra;

int main()
{
    // We are going to calculate the eigenvalues of M
    Eigen::MatrixXd A = Eigen::MatrixXd::Random(10, 10);
    Eigen::MatrixXd M = A + A.transpose();

    // Construct matrix operation object using the wrapper class DenseSymMatProd
    DenseSymMatProd<double> op(M);

    // Construct eigen solver object, requesting the largest three eigenvalues
    SymEigsSolver< double, LARGEST_ALGE, DenseSymMatProd<double> > eigs(&op, 3, 6);

    // Initialize and compute
    eigs.init();
    int nconv = eigs.compute();

    // Retrieve results
    Eigen::VectorXd evalues;
    if(eigs.info() == SUCCESSFUL)
        evalues = eigs.eigenvalues();

    std::cout << "Eigenvalues found:\n" << evalues << std::endl;

    return 0;
}
~~~

Sparse matrix is supported via the `SparseGenMatProd` class.

~~~
#include <Eigen/Core>
#include <Eigen/SparseCore>
#include <GenEigsSolver.h>
#include <MatOp/SparseGenMatProd.h>
#include <iostream>

using namespace Spectra;

int main()
{
    // A band matrix with 1 on the main diagonal, 2 on the below-main subdiagonal,
    // and 3 on the above-main subdiagonal
    const int n = 10;
    Eigen::SparseMatrix<double> M(n, n);
    M.reserve(Eigen::VectorXi::Constant(n, 3));
    for(int i = 0; i < n; i++)
    {
        M.insert(i, i) = 1.0;
        if(i > 0)
            M.insert(i - 1, i) = 3.0;
        if(i < n - 1)
            M.insert(i + 1, i) = 2.0;
    }

    // Construct matrix operation object using the wrapper class SparseGenMatProd
    SparseGenMatProd<double> op(M);

    // Construct eigen solver object, requesting the largest three eigenvalues
    GenEigsSolver< double, LARGEST_MAGN, SparseGenMatProd<double> > eigs(&op, 3, 6);

    // Initialize and compute
    eigs.init();
    int nconv = eigs.compute();

    // Retrieve results
    Eigen::VectorXcd evalues;
    if(eigs.info() == SUCCESSFUL)
        evalues = eigs.eigenvalues();

    std::cout << "Eigenvalues found:\n" << evalues << std::endl;

    return 0;
}
~~~

And here is an example for user-supplied matrix operation class.

~~~
#include <Eigen/Core>
#include <SymEigsSolver.h>
#include <iostream>

using namespace Spectra;

// M = diag(1, 2, ..., 10)
class MyDiagonalTen
{
public:
    int rows() { return 10; }
    int cols() { return 10; }
    // y_out = M * x_in
    void perform_op(double *x_in, double *y_out)
    {
        for(int i = 0; i < rows(); i++)
        {
            y_out[i] = x_in[i] * (i + 1);
        }
    }
};

int main()
{
    MyDiagonalTen op;
    SymEigsSolver<double, LARGEST_ALGE, MyDiagonalTen> eigs(&op, 3, 6);
    eigs.init();
    eigs.compute();
    if(eigs.info() == SUCCESSFUL)
    {
        Eigen::VectorXd evalues = eigs.eigenvalues();
        std::cout << "Eigenvalues found:\n" << evalues << std::endl;
    }

    return 0;
}
~~~

To compile and run these examples, simply download the source code of Spectra
and Eigen, and let the compiler know about their paths. For example:

~~~
g++ -I/path/to/eigen -I/path/to/spectra/include -O2 example.cpp
~~~
