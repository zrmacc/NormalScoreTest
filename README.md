# README
Zachary McCaw  
11/08/2017  

# Package Vignette




## Contents

* [Standard Score Test](#standard-score-test)
* [Kernelized Score Test](#kernelized-score-test)

## Standard Score Test
Consider the linear model:
$$
y = X\alpha + G\beta+\epsilon,\;\; \epsilon\sim N(0,\tau R)
$$
Here $\beta$ is the regression parameter of interest, $\alpha$ is a nuisance regression parameter, $\tau$ is a variance component, and $R$ is a fixed correlation structure. The function `lmScoreTest` provides a score test of $H_{0}:\beta = 0$. In particular, the score statistic is calculated as:
$$
T_{S} = S_{\beta}(\beta=0,\alpha=\tilde{\alpha})'I_{\beta\beta'|\alpha}^{-1}S_{\beta}(\beta=0,\alpha=\tilde{\alpha})'
$$
Here $\tilde{\alpha}$ solves the equation $S_{\alpha}(\beta=0,\alpha) = 0$. A $p$-value for $T_{S}$ is estimated by comparison with the $\chi_{\dim(\beta)}^{2}$ distribution. 

#### Size Estimation
Data including covariates $X$, genotypes $G$, and phenotype $Y$ were simulated for $10^{3}$ subjects. Covariates include age, sex, and the first two principal components of the centered and scaled subject by locus genotype matrix. Genotypes were generated at $10^{3}$ loci on chromosome one. A phenotype with normally distributed residuals was generated independently of genotypes. 

In the following, the genotype matrix is centered, then the score test is applied individually to each locus in $G$. Size is estimated at $\alpha = 0.05$ by averaging an indicator of rejection. 


```r
library(ScoreTest);
# Phenotype vector
Y = ScoreTest::Y;
N = nrow(Y);
# Covariate matrix
X = ScoreTest::X;
# Genotype matrix
G = ScoreTest::G;
ng = ncol(G);
# Minor allele frequencies
maf = apply(G,MARGIN=1,FUN=mean)/2;
# Standardizing genotype matrix
Gs = scale(t(G));
# Apply score test to each column of Gs
Results = sScoreTest(G=Gs,X=X,y=Y);
# Estimated size
cat("Estimated size at alpha=0.05:\n");
p = mean(Results[,"p"]<=0.05);
L = round(p-2*sqrt(p*(1-p)/ng),digits=3);
U = round(p+2*sqrt(p*(1-p)/ng),digits=3);
Out = c("p"=p,"Lower"=L,"Upper"=U);
show(Out);
```

```
## Estimated size at alpha=0.05:
##     p Lower Upper 
## 0.041 0.028 0.054
```

## Kernelized Score Test
The kernelized score test is calculated as:

$$
T_{K} = (y-X\tilde{\alpha})'K(y-X\tilde{\alpha})
$$

Here $\tilde{\alpha}$ is an estimate of $\alpha$ under $H_{0}:\beta =0$, and $K$ is a positive semi-definite matrix. A $p$-value for $T_{K}$ is estimated by comparison with the mixture distribution:

$$
\sum_{k}\lambda_{k}\chi_{1}^{2}(0)
$$

Here $\lambda_{k}$ are the non-zero eigen-values of $K\tilde{\Sigma}$, where:

$$
\tilde{\Sigma} = \text{Var}\left[y-X\tilde{\alpha}\right] = V-VX(X'VX)^{-1}X'V
$$

#### Size Estimation
For $R=5\times10^{2}$ MC replicates, groups of $10$ distinct loci were drawn at random from the centered genotype matrix. A kernelized score test of $H_{0}:\beta_{1}=\cdots=\beta_{10}=0$ was conducted, where $K = GWG'$, and the weighting scheme proposed in `SKAT` was adopted. For comparison, the standard score test of this hypothesis was also assessed. Size is estimated at $\alpha = 0.05$ by averaging an indicator of rejection.


```r
library(foreach);
R = 5e2;
# Down sample subjects
P = foreach(r=1:R,.combine=rbind,.inorder=F) %dopar% {
  # Select 10 loci at random
  Draw = sort(sample(ncol(Gs),size=10,replace=F));
  # Subset genotypes
  Gsub = Gs[,Draw];
  # Standard score test
  p = ScoreTest::ScoreTest(G=Gsub,X=X,y=Y)["p"];
  # Weights
  W = round(diag((dbeta(x=maf[Draw],1,25))^2),digits=6);
  # Remove positions with zero weight
  d = diag(W);
  flag = (d==0);
  Gsub = Gsub[,!flag];
  W = diag(d[!flag]);
  # Kernelized score test
  pk = ScoreTest::kScoreTest(G=Gsub,W=W,X=X,y=Y)$p;
  # Output
  Out = c("Score"=p,"Kernel.p"=pk);
}
```

<img src="Figs/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />
