# tuneBlolsPLS

## Install
```
devtools::install_github("FujiwaraFuji/tuneBlocksPLS")
```

## Read package
```
library(tuneBlocksPLS)
```

## Example
```
# Read sample dataset
# -------------
# TCGA breast cancer dataset containing mRNA, miRNA, and proteins block
data("breast.TCGA")

# input mRNA and miRNA datasets as X data
X = list(mrna = breast.TCGA$data.train$mrna,
          mirna = breast.TCGA$data.train$mirna)
# responsive variable must be input as matrix having the same rownames as each entry of X list
Y = as.matrix(breast.TCGA$data.train$protein[,3])


# Tuning parameters
# -------------
# set number of component per data set
ncomp <- 2

# design matrix is the same in mixOmics
design = matrix(0.1, ncol = length(X), nrow = length(X), dimnames = list(names(X), names(X)))
diag(design) = 0 # set diagonal to 0s

# vector v is the possible values for the number of selected variables for each component in each block
v <- c(5, 10)
# make.test.keepX makes a list of keepX used in cv.block.spls
# mode is first recommended to be set at 1 (Default)
# when the length of v is n, the length of the set of tested keepX is n^length(X)
test.keepX <- make.test.keepX(X, v, ncomp = ncomp, mode = 1)

# cv.block.spls tuning keepX parameters using (repeated) M-fold cross-validation
# chose "MSE"or "R" for model selection method (Default = "MSE")
# Note the computation time especially when repeating cross validation
res <- cv.block.spls(X, Y, test.keepX = test.keepX, design = design, ncomp = ncomp, nrepeat = 1, fold = 5, eval.mode = "MSE")

# check (average) MSE at the best parameter keepX
# print(res$MSE)
#
list.keepX <- res$list.keepX


# Fit multiblock sPLS method (mixOmics)
# -------------
# fit block.spls method in mixOmics package using estimated optimal keepX parameter
model <- block.spls(X, Y,
                    ncomp = ncomp,
                    keepX = list.keepX,
                    design = design)

# for details of the output, see https://www.rdocumentation.org/packages/mixOmics/versions/6.3.2/topics/block.spls
```
