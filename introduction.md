Introduction
================

-   [1 Index](#index)
-   [2 Installation](#installation)
    -   [2.1 Quick](#quick)
    -   [2.2 Manual](#manual)
-   [3 Usage](#usage)
-   [4 Pipeline minimum input data](#pipeline-minimum-input-data)
    -   [4.1 Input data](#input-data)
    -   [4.2 Input files](#input-files)
-   [5 Pipeline output data](#pipeline-output-data)
-   [6 Acknowledgements](#acknowledgements)
-   [7 References](#references)
    -   [7.1 Data source in package and case
        studies](#data-source-in-package-and-case-studies)
    -   [7.2 Methods used in package](#methods-used-in-package)
    -   [7.3 R package compilation](#r-package-compilation)

Copyright (c) 2020
<a href="https://orcid.org/0000-0002-9207-0385">Tyrone Chen
<img alt="ORCID logo" src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png" width="16" height="16" /></a>,
<a href="https://orcid.org/0000-0002-4146-2848">Al J Abadi
<img alt="ORCID logo" src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png" width="16" height="16" /></a>,
<a href="https://orcid.org/0000-0003-3923-1116">Kim-Anh Lê Cao
<img alt="ORCID logo" src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png" width="16" height="16" /></a>,
<a href="https://orcid.org/0000-0003-0181-6258">Sonika Tyagi
<img alt="ORCID logo" src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png" width="16" height="16" /></a>

Code in this package and git repository
<https://gitlab.com/tyagilab/sars-cov-2/> is provided under a [MIT
license](https://opensource.org/licenses/MIT). This documentation is
provided under a [CC-BY-3.0 AU
license](https://creativecommons.org/licenses/by/3.0/au/).

[Visit our lab website here.](https://bioinformaticslab.erc.monash.edu/)
Contact Sonika Tyagi at <sonika.tyagi@monash.edu>.

# 1 Index

-   [Introduction](introduction.html)
-   [Case study 1](case_study_1.html)
-   [Case study 2](case_study_2.html)

# 2 Installation

## 2.1 Quick

You can install this directly as an R package from gitlab:

    install.packages(devtools)
    install_gitlab("tyagilab/sars-cov-2", subdir="multiomics", build_vignettes=TRUE)

The actual script used to run the pipeline is not directly callable but
provided as a separate script.

    # this will show you the path to the script
    system.file("scripts", "run_pipeline.R", package="multiomics")

## 2.2 Manual

Alternatively, clone the git repository with:

    git clone "https://gitlab.com/tyagilab/sars-cov-2.git"

### 2.2.1 Install dependencies

[With `conda`](https://bioconda.github.io/user/install.html):

    conda config --add channels defaults
    conda config --add channels bioconda
    conda config --add channels conda-forge

    install_me="r-argparser r-brio r-colorspace r-diffobj r-dplyr r-ellipsis \
     r-farver r-ggplot2 r-ggrepel r-igraph r-isoband r-matrixStats r-mixOmics \
     r-parallel r-plyr r-rARPACK r-Rcpp r-RcppEigen r-reshape2 r-RSpectra \
     r-stringi r-testthat r-tibble r-tidyr r-utf8 r-vctrs r-zeallot \
     bioconductor-biocparallel"

    conda create -n my_environment install ${install_me}

You can also install dependencies in `R` directly:

    install_me <- c(
      "argparser", "brio", "colorspace", "diffobj", "dplyr", "ellipsis", "farver",
      "ggplot2", "ggrepel", "igraph", "isoband", "matrixStats", "mixOmics",
      "parallel", "plyr", "rARPACK", "Rcpp", "RcppEigen", "reshape2", "RSpectra",
      "stringi", "testthat", "tibble", "tidyr", "utf8", "vctrs", "zeallot",
      "BiocParallel")
    sapply(install_me, install.packages)

# 3 Usage

Load the library.

``` r
library(multiomics)
```

If you installed this pipeline as an `R` package, an example pipeline
script is included. You can find it by running this command in your `R`
environment:

    system.file("scripts", "run_pipeline.R", package="multiomics")

Otherwise, you can find a copy of this script in the public git
repository:
<https://gitlab.com/tyagilab/sars-cov-2/-/raw/master/src/run_pipeline.R>

To inspect the arguments to the script, run this command:

    Rscript run_pipeline.R --help

A minimal script to run the pipeline is shown. [You can also download
this
here](https://gitlab.com/tyagilab/sars-cov-2/-/raw/master/src/test_run.sh).
This may take a few minutes.

    Rscript run_pipeline.R \
       classes.tsv  \
       --dropna_classes FALSE \
       --dropna_prop 0 \
       --data lipidomics.tsv metabolomics.tsv \
       --data_names lipidomics metabolomics \
       --force_unique FALSE \
       --ncpus 2 \
       --diablocomp 0 \
       --linkage 0.1 \
       --diablo_keepx 5 10 \
       --icomp 0 \
       --pcomp 10 \
       --plsdacomp 2 \
       --splsdacomp 2 \
       --splsda_keepx 5 10 \
       --dist_plsda centroids.dist \
       --dist_splsda centroids.dist \
       --dist_diablo centroids.dist \
       --contrib max \
       --outfile_dir ./results/test_run/ \
       --rdata RData.RData \
       --plot Rplots.pdf \
       --args Rscript.sh

# 4 Pipeline minimum input data

## 4.1 Input data

The minimum input data needed is a file of classes and at least two
files of quantitative omics data. Tab separated data is expected by
default.

A small example subset of test data is included in the package for
reference.

``` r
data(two_omics)
names(two_omics)
#> [1] "classes"    "lipidome"   "metabolome"
```

The class information is available as a vector:

``` r
head(two_omics$classes)
#> [1] "More severe" "More severe" "Less severe" "Less severe" "More severe"
#> [6] "More severe"
```

The lipidomics and metabolomics data have 100 matched samples and an
arbitrary number of features.

``` r
sapply(two_omics, dim)
#> $classes
#> NULL
#> 
#> $lipidome
#> [1]  100 3357
#> 
#> $metabolome
#> [1] 100 150
```

``` r
head(two_omics$lipidome[, 1:2])
#>      AC.10.0_RT_6.936 AC.12.0_RT_7.955
#> C1           18.13745         16.84196
#> C10          20.48135         18.06048
#> C100         22.32588         21.30632
#> C101         20.56189         18.84777
#> C102         22.28591         17.98330
#> C103         18.18658         16.97716
head(two_omics$metabolome[, 1:2])
#>      X1.2.Propanediol..2TMS.de X2.3.Dihydroxybutanoic.ac
#> C1                    22.52599                  13.89898
#> C10                   22.63460                  17.85105
#> C100                  22.12956                  13.34028
#> C101                  21.94220                  17.44137
#> C102                  21.87579                  17.88084
#> C103                  21.37599                  14.28262
```

Important notes on input data:

1.  Class information and the **sample order in each omics dataset must
    be identical**.
2.  Feature names in each omics dataset are truncated. Too long names
    may cause issues.
3.  `R` silently replaces all non-alphanumeric characters in feature
    names with `.`.

To work around (2) and (3), you can rename your feature names to a short
alphanumeric ID in your files, and remap them back later.

## 4.2 Input files

For examples of input files, you can see them by running this command:

    list.files(system.file("extdata", package="multiomics"), full.names=TRUE)

If you did not install the `R` package, you can obtain these files from
gitlab:

-   [classes
    information](https://gitlab.com/tyagilab/sars-cov-2/-/raw/master/data/case_study_2/classes_diablo.tsv)
-   [lipidomics
    data](https://gitlab.com/tyagilab/sars-cov-2/-/raw/master/data/case_study_2/data_lipidomics.tsv)
-   [metabolomics
    data](https://gitlab.com/tyagilab/sars-cov-2/-/raw/master/data/case_study_2/data_metabolomics.tsv)

# 5 Pipeline output data

Files output by the pipeline include:

-   a `pdf` file of all plots generated by the pipeline
-   tab-separated `txt` files containing feature contribution weights to
    each biological class
-   tab-separated `txt` file containing correlations between each omics
    data block

[A `RData` object with all input and output is available in the git
repository.](https://gitlab.com/tyagilab/sars-cov-2/-/blob/master/results/case_study_2/RData.RData)
This is not included directly in the `multiomics` package because of
size constraints, and includes data from four omics datasets.

# 6 Acknowledgements

[We thank David A. Matthews](https://orcid.org/0000-0003-4611-8795) for
helpful discussions and feedback. [We thank Yashpal
Ramakrishnaiah](https://orcid.org/0000-0002-2213-8348) for performing an
extended analysis of the primary data. [We thank Melcy
Philip](https://orcid.org/0000-0002-0827-866X) for performing downstream
analysis of the data. This work was supported by the [MASSIVE HPC
facility](www.massive.org.au) and the authors thank the HPC team at
Monash eResearch Centre for their continuous personnel support. This R
package was compiled referring to information from blog posts or books
by [Hilary
Parker](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/),
[Fong Chun
Chan](https://tinyheero.github.io/jekyll/update/2015/07/26/making-your-first-R-package.html),
[Karl Broman](https://kbroman.org/pkg_primer/pages/data.html), [Yihui
Xie, J. J. Allaire, Garrett
Grolemund](https://bookdown.org/yihui/rmarkdown/) as well as [Jenny
Bryan and Hadley Wickham](https://r-pkgs.org/). [We acknowledge and pay
respects to the Elders and Traditional Owners of the land on which our 4
Australian campuses
stand](https://www.monash.edu/indigenous-australians/about-us/recognising-traditional-owners).

# 7 References

## 7.1 Data source in package and case studies

-   [Bojkova, D., Klann, K., Koch, B. et al. Proteomics of
    SARS-CoV-2-infected host cells reveals therapy targets. Nature 583,
    469–472 (2020).
    https://doi.org/10.1038/s41586-020-2332-7](https://doi.org/10.1038/s41586-020-2332-7)
-   [Overmyer, K.A., Shishkova, E., Miller, I.J., Balnis, J., Bernstein,
    M.N., Peters-Clarke, T.M., Meyer, J.G., Quan, Q., Muehlbauer, L.K.,
    Trujillo, E.A. and He, Y., 2021. Large-scale multi-omic analysis of
    COVID-19 severity. Cell
    systems.](https://dx.doi.org/10.1016%2Fj.cels.2020.10.003)

## 7.2 Methods used in package

-   [Amrit Singh, Casey P Shannon, Benoît Gautier, Florian Rohart,
    Michaël Vacher, Scott J Tebbutt, Kim-Anh Lê Cao, DIABLO: an
    integrative approach for identifying key molecular drivers from
    multi-omics assays, Bioinformatics, Volume 35, Issue 17, 1 September
    2019, Pages 3055–3062,
    https://doi.org/10.1093/bioinformatics/bty1054](https://doi.org/10.1093/bioinformatics/bty1054)
-   Butte, A. J., Tamayo, P., Slonim, D., Golub, T. R. and Kohane, I. S.
    (2000). Discovering functional relationships between RNA expression
    and chemotherapeutic susceptibility using relevance networks.
    *Proceedings of the National Academy of Sciences of the USA* *97*,
    12182-12186.
-   Chavent, Marie and Patouille, Brigitte (2003). Calcul des
    coefficients de regression et du PRESS en regression PLS1. *Modulad
    n*, *30* 1-11.
-   Eisen, M. B., Spellman, P. T., Brown, P. O. and Botstein, D. (1998).
    Cluster analysis and display of genome-wide expression patterns.
    *Proceeding of the National Academy of Sciences of the USA* *95*,
    14863-14868.
-   Eslami, A., Qannari, E. M., Kohler, A., and Bougeard, S. (2013).
    Multi-group PLS Regression: Application to Epidemiology. In New
    Perspectives in Partial Least Squares and Related Methods, pages
    243-255. Springer.
-   González I., Lê Cao K.A., Davis M.J., Déjean S. (2012). Visualising
    associations between paired ‘omics’ data sets. *BioData Mining*;
    *5*(1)
-   [H.M. Blalock, A. Aganbegian, F.M. Borodkin, Raymond Boudon,
    Vittorio Capecchi. Path Models with Latent Variables: The NIPALS
    Approach. In International Perspectives on Mathematical and
    Statistical Modeling (1975).
    https://doi.org/10.1016/B978-0-12-103950-9.50017-4](https://doi.org/10.1016/B978-0-12-103950-9.50017-4)
-   Lê Cao, K.-A., Martin, P.G.P., Robert-Granie, C. and Besse, P.
    (2009). Sparse canonical methods for biological data integration:
    application to a cross-platform study. *BMC Bioinformatics* *10*:34
-   [Liquet, B., Cao, K.L., Hocini, H. et al. A novel approach for
    biomarker selection and the integration of repeated measures
    experiments from two assays. BMC Bioinformatics 13, 325 (2012).
    https://doi.org/10.1186/1471-2105-13-325](https://doi.org/10.1186/1471-2105-13-325)
-   Mevik, B.-H., Cederkvist, H. R. (2004). Mean Squared Error of
    Prediction (MSEP) Estimates for Principal Component Regression (PCR)
    and Partial Least Squares Regression (PLSR). *Journal of
    Chemometrics* *18*(9), 422-429.
-   Moriyama, M., Hoshida, Y., Otsuka, M., Nishimura, S., Kato, N.,
    Goto, T., Taniguchi, H., Shiratori, Y., Seki, N. and Omata, M.
    (2003). Relevance Network between Chemosensitivity and Transcriptome
    in Human Hepatoma Cells. *Molecular Cancer Therapeutics* *2*,
    199-205.
-   [Rohart F, Gautier B, Singh A, Lê Cao KA (2017) mixOmics: An R
    package for ‘omics feature selection and multiple data integration.
    PLOS Computational Biology 13(11): e1005752.
    https://doi.org/10.1371/journal.pcbi.1005752](https://doi.org/10.1371/journal.pcbi.1005752)
-   Weinstein, J. N., Myers, T. G., O’Connor, P. M., Friend, S. H.,
    Fornace Jr., A. J., Kohn, K. W., Fojo, T., Bates, S. E.,
    Rubinstein, L. V., Anderson, N. L., Buolamwini, J. K., van Osdol, W.
    W., Monks, A. P., Scudiero, D. A., Sausville, E. A., Zaharevitz, D.
    W., Bunow, B., Viswanadhan, V. N., Johnson, G. S., Wittes, R. E. and
    Paull, K. D. (1997). An information-intensive approach to the
    molecular pharmacology of cancer. *Science* *275*, 343-349.

## 7.3 R package compilation

-   <https://tinyheero.github.io/jekyll/update/2015/07/26/making-your-first-R-package.html>

-   <https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/>

-   <https://kbroman.org/pkg_primer/pages/data.html>

-   Wickham, H., 2015. R packages: organize, test, document, and share
    your code. " O’Reilly Media, Inc.", <https://r-pkgs.org/>.

-   Xie Y (2021). knitr: A General-Purpose Package for Dynamic Report
    Generation in R. R package version 1.31, <https://yihui.org/knitr/>.

-   Xie Y (2015). Dynamic Documents with R and knitr, 2nd edition.
    Chapman and Hall/CRC, Boca Raton, Florida. ISBN 978-1498716963,
    <https://yihui.org/knitr/>.

-   Xie Y (2014). “knitr: A Comprehensive Tool for Reproducible Research
    in R.” In Stodden V, Leisch F, Peng RD (eds.), Implementing
    Reproducible Computational Research. Chapman and Hall/CRC. ISBN
    978-1466561595,
    <http://www.crcpress.com/product/isbn/9781466561595>.
