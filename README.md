# CategoryEncoders
A set of gretl transformers for encoding categorical variables into numeric with different techniques.

The mean, median, pca, low-rank and multinomial logit encoding techniques are inspired by the paper of Athey et al. (2019) entitled ["Sufficient Representations for Categorical Variables"] (https://arxiv.org/abs/1908.09874). Their github project-page can be found [here] (https://github.com/grf-labs/sufrep).

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

# PCA encoding with automatic mean encoding
list Xpca = pca_encode(Xbase, unit)			# means_encoding done per default
print unit Xbase Xpca -o --range=70:80

# PCA encoding without automatic mean encoding
list Xpca_wo_mean = pca_encode(Xbase, unit, , 0)
print unit Xbase Xpca_wo_mean -o --range=70:80

# SVD (low rank) encoding but return only first 2 components
list Xsvd = low_rank_encode(Xbase, unit, 2)
print unit Xbase Xsvd -o --range=70:80

# Multinomial logistic regression coefficients (excluding intercept which is used for estimation)
list Xmnl = mnl_encode(Xbase, unit, 1)
print unit Xbase Xmnl -o --range=70:80
```



# Public Functions

## binary_encode
Binarize data (set feature values to 0 or 1) according to a threshold. Values greater than the threshold map to 1, while values less than or equal to the threshold map to 0. With the default threshold of 0, only positive values map to 1.

### Parameters:
X	-- list, List of series to which to apply the function

threshold -- scalar, Feature values below or equal to this are replaced by 0, above it by 1 (0.0 by default).

suffix -- string, Add a suffix to the returned series' name (default value *_bin*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with binary values. The i-th series name takes the i-th input series' name plus the suffix *_bin* if not differently set.
***

## ohe_encode
Encode categorical discrete integer features of a single series as a one-hot (binary) list of series. The encoder derives the categories based on the unique values of the input series.

### Parameters:
X -- series, Series with discrete values to which to apply the function

drop -- int, Decide whether to drop a resulting series. Per default no resulting series is droppped.
	* For drop=1, the highest value of X is omitted from the encoding.
	* For drop=2, the lowest value of X is omitted from the encoding

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with binary values. The i-th series is named 'Dx_i' where 'x' refers to the name of input series X and 'i' to the i-th distinct value of X.

### Example:
![](/figs/ohe.png)
***

## means_encode
Compute category-wise, as identified by the discrete series G, mean values for each variable in list X.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series refering to unique categories

suffix -- string, Add a suffix to the returned series' name (default value *_mean*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise mean values. The i-th series name takes the i-th input series' name plus the suffix *_mean* if not differently set.

### Example:
![](/figs/means_encoding.png)
***

## median_encode
Compute category-wise, as identified by the discrete series G, median values for each variable in list X.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series refering to unique categories

suffix -- string, Add a suffix to the returned series' name (default value *_mean*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise median values. The i-th series name takes the i-th input series' name plus the suffix *_median* if not differently set..
***

## low_rank_encode
Compute category-wise, as identified by the discrete series G, left-singlular value matrix component by means of Singular Value Decomposition using all information in list X.

### Parameters:
X -- list, List of k series to which to apply the function

G -- series, Discrete series identifying unique groups

num_components -- int, Retrieve only the first n columns of the left-singular value matrix (per default retrieve the first k columns)

mean_encode -- bool, Apply means-encoding on all X elemenets before computing the low-rank components of these mean-encoded series.

suffix -- string, Add a suffix to the returned series' name (default value *_svd*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with group-wise rank component values. The i-th series name takes the i-th input series' name plus the suffix *_svd* if not differently set.

### Example:
![](/figs/low_rank.png)
***

## pca_encode
Works similar to the low_rank_encode() function. Compute category-wise, as identified by the discrete series G, principle components using all information in list X.

### Parameters:
X -- list, List of series to which to apply the function

G -- series, Discrete series identifying unique categories

num_components -- int, Retrieve only the first n principle components (per default retrieve all)

mean_encode -- bool, Apply means-encoding on all X elemenets before computing the principle components of these n-encoded series.

suffix -- string, Add a suffix to the returned series' name (default value *_pca*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with group-wise principle component values. The i-th series name takes the i-th input series' name plus the suffix *_pca* if not differently set.
***

## mnl_encode
Compute category-wise, as identified by the discrete series G, left-singlular value matrix component by means of Singular Value Decomposition using all information in list X.

### Parameters:
X -- list, List of k series to which to apply the function
G -- series, Discrete series identifying unique groups

num_components -- int, Retrieve only the first n columns of the left-singular value matrix (per default retrieve the first k columns)

mean_encode -- bool, Apply means-encoding on all X elemenets before computing the low-rank components of these mean-encoded series.

suffix -- string, Add a suffix to the returned series' name (default value *_mnl*)

verbose -- bool, Print details or not (per default no printout)

### Returns:
List of named series with category-wise rank component values. The i-th series name takes the i-th input series' name plus the suffix *_mnl* if not differently set.

### Example:
![](/figs/mnl_encoding.png)
***


## Changelog:
Version 0.2 (Sept. 2019):
  * Initial version
