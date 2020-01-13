# CategoryEncoders

[![Build Status](https://travis-ci.org/atecon/CategoryEncoders.svg?branch=master)](https://travis-ci.org/atecon/CategoryEncoders)

A set of gretl transformers for encoding categorical variables into numeric with different techniques. Many econometric techniques and learning algorithms require categorical data to be transformed into real vectors before it can be used as input. 

The mean, median, pca, low-rank and multinomial logit encoding techniques are inspired by the paper of Athey et al. (2019) entitled ["Sufficient Representations for Categorical Variables"] (https://arxiv.org/abs/1908.09874). Their github project-page can be found [here] (https://github.com/grf-labs/sufrep). The authors propose lower-dimensional real-values representations of categorical variables that are sufficient in the sense that no predictive information is lost.


# Installation and usage
To install this package, run the following commands in gretl:
```gretl
pkg install CategoryEncoders
```
Sample script:
```gretl
include "CategoryEncoders.gfn"

open nc_crime.gdt -q --preserve

# Illustrative discrete 'grouping' variable
series unit = urban
setinfo unit --discrete

# List of regressors
list Xbase = density wcon

# One-hot encoding (aka dummifying)
list OHE = ohe_encode(unit)
print unit OHE -o --range=70:80

# Binarizer
list BIN = binary_encode(Xbase, 1.1)
print Xbase BIN -o --range=70:80

# Mean encoding
list Xpmean = means_encode(Xbase, unit, "some_suffix")
print unit Xbase Xpmean -o --range=70:80

# Median encoding
list Xpmedian = median_encode(Xbase, unit)
print unit Xbase Xpmedian -o --range=70:80

# PCA encoding (means_encoding done per default)
list Xpca = pca_encode(Xbase, unit)
print unit Xbase Xpca -o --range=70:80

# SVD aka 'low rank' encoding (means_encoding done per default)
# Return only first 2 components
list Xsvd = low_rank_encode(Xbase, unit, 2)
print unit Xbase Xsvd -o --range=70:80

# Multinomial logistic regression coefficients (no means_encoding done)
list Xmnl = mnl_encode(Xbase, unit, 1)
print unit Xbase Xmnl -o --range=70:80
```

The returned output is:
```gretl
# print unit OHE -o --range=70:80
             unit   unit_ohe_0   unit_ohe_1

10:7            0            1            0
11:1            1            0            1
11:2            1            0            1
11:3            1            0            1
11:4            1            0            1
11:5            1            0            1
11:6            1            0            1
11:7            1            0            1
12:1            0            1            0
12:2            0            1            0
12:3            0            1            0

# print Xbase BIN -o --range=70:80
          density         wcon  density_bin     wcon_bin

10:7     0.576744      260.138            0            1
11:1     2.461305      231.923            1            1
11:2     2.485584      241.853            1            1
11:3     2.497724      247.788            1            1
11:4     2.523520      262.515            1            1
11:5     2.555387      262.634            1            1
11:6     2.579666      289.087            1            1
11:7     2.602428      313.474            1            1
12:1     1.452381      194.653            1            1
12:2     1.470238      203.073            1            1
12:3     1.468254      227.359            1            1

# print unit Xbase Xpmean -o --range=70:80
                    unit             density                wcon density_some_suffix    wcon_some_suffix

10:7                   0            0.576744             260.138            1.018823            242.1828
11:1                   1            2.461305             231.923            5.150256            281.3701
11:2                   1            2.485584             241.853            5.150256            281.3701
11:3                   1            2.497724             247.788            5.150256            281.3701
11:4                   1            2.523520             262.515            5.150256            281.3701
11:5                   1            2.555387             262.634            5.150256            281.3701
11:6                   1            2.579666             289.087            5.150256            281.3701
11:7                   1            2.602428             313.474            5.150256            281.3701
12:1                   0            1.452381             194.653            1.018823            242.1828
12:2                   0            1.470238             203.073            1.018823            242.1828
12:3                   0            1.468254             227.359            1.018823            242.1828

# print unit Xbase Xpmedian -o --range=70:80
               unit        density           wcon density_median    wcon_median

10:7              0       0.576744        260.138       0.819739       232.2234
11:1              1       2.461305        231.923       5.151138       279.0984
11:2              1       2.485584        241.853       5.151138       279.0984
11:3              1       2.497724        247.788       5.151138       279.0984
11:4              1       2.523520        262.515       5.151138       279.0984
11:5              1       2.555387        262.634       5.151138       279.0984
11:6              1       2.579666        289.087       5.151138       279.0984
11:7              1       2.602428        313.474       5.151138       279.0984
12:1              0       1.452381        194.653       0.819739       232.2234
12:2              0       1.470238        203.073       0.819739       232.2234
12:3              0       1.468254        227.359       0.819739       232.2234

# print unit Xbase Xpca -o --range=70:80
                 unit          density             wcon density_mean_pca    wcon_mean_pca

10:7                0         0.576744          260.138        -0.441375     -2.70617e-14
11:1                1         2.461305          231.923         4.524098      -5.9508e-14
11:2                1         2.485584          241.853         4.524098      -5.9508e-14
11:3                1         2.497724          247.788         4.524098      -5.9508e-14
11:4                1         2.523520          262.515         4.524098      -5.9508e-14
11:5                1         2.555387          262.634         4.524098      -5.9508e-14
11:6                1         2.579666          289.087         4.524098      -5.9508e-14
11:7                1         2.602428          313.474         4.524098      -5.9508e-14
12:1                0         1.452381          194.653        -0.441375     -2.70617e-14
12:2                0         1.470238          203.073        -0.441375     -2.70617e-14
12:3                0         1.468254          227.359        -0.441375     -2.70617e-14

# print unit Xbase Xsvd -o --range=70:80
                 unit          density             wcon density_mean_svd    wcon_mean_svd

10:7                0         0.576744          260.138          0.03924          0.01424
11:1                1         2.461305          231.923          0.04559       -0.1256141
11:2                1         2.485584          241.853          0.04559       -0.1256141
11:3                1         2.497724          247.788          0.04559       -0.1256141
11:4                1         2.523520          262.515          0.04559       -0.1256141
11:5                1         2.555387          262.634          0.04559       -0.1256141
11:6                1         2.579666          289.087          0.04559       -0.1256141
11:7                1         2.602428          313.474          0.04559       -0.1256141
12:1                0         1.452381          194.653          0.03924          0.01424
12:2                0         1.470238          203.073          0.03924          0.01424
12:3                0         1.468254          227.359          0.03924          0.01424

# print unit Xbase Xmnl -o --range=70:80
                 unit          density             wcon density_mean_mnl    wcon_mean_mnl

10:7                0         0.576744          260.138          1.00000         1.000000
11:1                1         2.461305          231.923         14.03720        -0.187684
11:2                1         2.485584          241.853         14.03720        -0.187684
11:3                1         2.497724          247.788         14.03720        -0.187684
11:4                1         2.523520          262.515         14.03720        -0.187684
11:5                1         2.555387          262.634         14.03720        -0.187684
11:6                1         2.579666          289.087         14.03720        -0.187684
11:7                1         2.602428          313.474         14.03720        -0.187684
12:1                0         1.452381          194.653          1.00000         1.000000
12:2                0         1.470238          203.073          1.00000         1.000000
12:3                0         1.468254          227.359          1.00000         1.000000
```


# Public Functions

## binary_encode
Binarize data (set feature values to 0 or 1) according to a threshold. Values greater than the threshold map to 1, while values less than or equal to the threshold map to 0. With the default threshold of 0, only positive values map to 1.

### Parameters:
X	-- list, List of series to which to apply the function

threshold -- scalar, Feature values below or equal to this are replaced by 0, above it by 1 (0.0 by default).

suffix -- string, Add a suffix to the returned series' name (default value "\_bin")

### Returns:
List of named series with binary values. Missing (NA) values of the i-th input series are ignored. The i-th series' name takes the i-th input series' name plus the suffix '\_bin' if not differently set.
***

## ohe_encode
Encode categorical discrete integer features of a single series as a one-hot (binary) list of series. The encoder derives the categories based on the unique values of the input series.

### Parameters:
X -- series, Series with discrete values to which to apply the function

drop -- int, Decide whether to drop a resulting series. Per default no resulting series is droppped.
	* For drop=1, the lowest value of X is omitted from the encoding.
	* For drop=2, the highest value of X is omitted from the encoding

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with binary values. The i-th series is named 'x_ohe_d' where 'x' refers to the name of input series X[i] and 'd' to the d-th distinct value of X. If the the input series is not discrete, includes only missing values (NA) or is a constant, an empty list is returned.

### Example:
![](/figs/ohe.png)
***

## means_encode
Compute category-wise, as identified by the discrete series G, mean values for each variable in list X.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series refering to unique categories

suffix -- string, Add a suffix to the returned series' name (default value "\_mean")

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise mean values. The i-th series name takes the i-th input series' name plus the suffix "\_mean" if not differently set. If G is not a discrete series, an empty list is returned. Missing observations of the i-th series X[i] are interpolated by group-wise mean values.

### Example:
![](/figs/means_encoding.png)
***

## median_encode
Compute category-wise, as identified by the discrete series G, median values for each variable in list X.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series refering to unique categories

suffix -- string, Add a suffix to the returned series' name (default value "\_mean")

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise median values. The i-th series name takes the i-th input series' name plus the suffix "\_median" if not differently set. If G is not a discrete series, an empty list is returned. Missing observations of the i-th series X[i] are interpolated by group-wise median values.
***

## low_rank_encode
Compute category-wise, as identified by the discrete series G, left-singlular value matrix component by means of Singular Value Decomposition using all information in list X. Means encoding, by applying the function mean_encode(), is automatically applied to all members of X, before computing principal components.

### Parameters:
X -- list, List of k series to which to apply the function

G -- series, Discrete series identifying unique groups

num_components -- int, Retrieve only the first n columns of the left-singular value matrix (per default retrieve the first k columns)

suffix -- string, Add a suffix to the returned series' name (default value "\_svd")

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with group-wise rank component values. The i-th series name takes the i-th input series' name plus the suffix "\_svd" if not differently set. If G is not discrete, list X includes less than two members or some series X[i] is a constant, an empty list is returned. If some series X[i] comprises at least some non-missing values, SVD will probably still work.

### Example:
![](/figs/low_rank.png)
***

## pca_encode
Works similar to the low_rank_encode() function. Compute category-wise, as identified by the discrete series G, principle Compute category-wise, as identified by the discrete series G, principle components using all information in list X. Means encoding, by applying the function mean_encode(), is automatically applied to all members of X, before computing principal components.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series identifying unique categories

num_components -- int, Retrieve only the first n principle components (per default retrieve all). 'num_components' cannot exceed the number of list members of X.

suffix -- string, Add a suffix to the returned series' name (default value "\_pca")

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with group-wise principle component values. The i-th series name takes the i-th input series' name plus the suffix "\_pca" if not differently set. If G is not discrete, list X includes less than two members, or any series of X is constant or includes inly missing values, an empty list is returned. If some series X[i] comprises at least some non-missing values, PCA will probably still work.
***

## mnl_encode
Compute category-wise, as identified by the discrete series G, left-singlular value matrix component by means of Singular Value Decomposition using all information in list X. Means encoding, by applying the function mean_encode(), is automatically applied to all members of X, before computing principal components.

### Parameters:
X -- list, List of k series to which to apply the function
G -- series, Discrete series identifying unique groups

num_components -- int, Retrieve only the first n columns of the left-singular value matrix (per	default retrieve the first k columns)

suffix -- string, Add a suffix to the returned series' name (default value "\_mnl")

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise rank component values. The i-th series name takes the i-th input series' name plus the suffix "\_mnl" if not differently set. If G is not discrete, of G has not at least two distinct values, list X includes less than two members or some series X[i] is a constant, an empty list is returned. If some series X[i] comprises at least some missing values, logit type of estimation fails.

### Example:
![](/figs/mnl_encoding.png)
***

# Tests
Unit-tests are stored in the "tests" folder. Run the following shell-script:
```shell
./run_tests.sh
```

## Changelog:
Version 0.3 (Jan. 2020):
 * Bugfix: For mnl_encode(), no means encoding is applied on list X any more as this resulted in perfect collinearity issues.
 * Mean enocoding is done automatically when calling pca_encode() or low_rank_enocde(), and cannot be selected by the user.
 * Drop unused 'verbose' argument from binary_encode().
 * Add several unit-tests.
 * Add additional checks.
 * Code cleanup.

Version 0.2 (Sept. 2019):
  * Initial version
