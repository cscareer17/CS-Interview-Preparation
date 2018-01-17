An exhaustive list of questions you might be asked if you were to apply as a Data Scientist or a Machine Learning Engineer/Researcher. Having a degree in Machine Learning, I already had a good foundation in this field. I found this list sufficient for most of the interviews.


# Definitions
### Difference between Machine Learning and Data Science?
Data Science is the statistic for Big Data, while the techniques in ML may have application in data science, the main goal is the solve the grand challenge of AI

### Define variance?
How far a set of numbers are from their mean. Define in squared unit while standard deviation is defined in the unit of the input data.

### Difference between probability mass function, density function, distribution function?
* Probability mass function of `X` at `x` is the probability that X takes exactly the value x. It is used for discrete values. `p(x) = P(X  = x)`
* Probability density function of `X`. `X` can take an uncountable number of values. So you must define the density as an integral over an space, and this integral should sum to 1. So `p(X=x)=0`, and it only makes sense when you talk about interval
* Probability distribution function is often referred to the cumulative distribution. For probability mass function, it is the sum of all probability mass function before a point, and for the probability density function, it is integral before an upper bound.

### Statistical anomaly?
Something that does not follow a classical statistic, probability, a wrong outcome

### What is LDA?
It is a probabilistic model that define a text as built of topics, that are a mixture of words. Very human interpretable

### What is Boosting?
Focus new learners on examples that others get wrong. Train learners sequentially. It convert many weak learners into a complex predictor

### Explain the variance bias trade-off?
Variance: How much the model varies if the training set was different?
Bias: How far the averaged model is from the true solution
The generalization error can be seen as a sum of variance and bias. 

### What a kernel is?
See [there](https://stats.stackexchange.com/questions/152897/how-to-intuitively-explain-what-a-kernel-is)

### Law of a large number?
When you repeat an experience a certain number of times, and the numerical outcome mean tend to be very close to the expectation. 

### LDA, PCA, QDA, difference?
* PCA is a linear transformation technique that doesn't use labels to compute the transformation, but instead, use the variance of the data
* LDA is a linear transformation that uses labels, and try to find a subspace that maximizes the class separability. LDA makes the assumption of normally distributed class with equal class covariance
* While QDA doesn't assume an equal class covariance

### Linear Regression vs Logisitic Regression?
Linear regression learns a relation between dependent and independent variables, while logistic regression learns a probability of a categorical event. 

### Central Limit Theorem, What, Proof, for What?
Given the law of large number, as the number of experience can't go to infinity, you can assume that the outcome of the experience will follow a Gaussian curve, and the variance is inversely proportional to the number of experiences. 
Proof: https://en.wikipedia.org/wiki/Central_limit_theorem#Proof_of_classical_CLT

### What is the T-test?
It can be used to test if a sample means significantly differs from a hypothesized value. The statistic of the test is a Student random variable. We can look at the table, and if the value calculated is larger than the 95% quantile, we can reject the null hypothesis
A p-value is a probability that the results from your sample data occurred by chance 

### Pareto Principle or 80/20 percent?
Empirical phenomena. 80% of the effects can be explained by 20% of the causes

### What is the degree of freedom?
The degrees of freedom of a system is the number of parameters of the system that may vary independently

### What is a robust estimator?
A robust estimator is an estimator does not change a lot with respect to a complete outlier. We talk about breakdown point, a value between 0 and 0.5. If the breakdown point is 0, it means that an estimator might be confused by a single point in the dataset containing n points. An example is a sample mean, that might goes to infinity if we have a large outlier. This is not the case of the sampled median that has a BP of 0.5.

### What is the standard error of the mean?
The standard error of a statistic is the standard deviation of its sampling distribution. It is defined as standard deviation \ sqrt(num_observations).
Decreasing the uncertainty in a mean value estimate by a factor of two requires acquiring four times as many observation

# How does it work?
### How do you compute TF-IDF?
Here is the formula:
* `idf_i =` reversed frequence of document: `log`(number of documents / number of documents that contains the term i)
* `tf_i =` frequency of a word (different formulation)
* `tf_idf = idf * tf`

### Explain how works Conditional Random Field?
It learns a join distribution of a sequence of output given a sequence of input. A simpler CRF, the linear chain CRF learns to predict the join distribution as a function of multiple terms:
`p(Y={y_1, y_t, y_T}|X={x_1, x_t, x_T}) =` `\exp([sum_t \text{probability of token_t}] + sum_t \text{probability of token_t given token_{t-1}}) / \text{Partition function overall sequence possibles}`
The first term is computed with a linear transformation of the previous layer output, while the second one is a separate matrix that has entry `[i, j] = p(w_i|w_j)`
The partition function can be computed efficiently with dynamic programming

### Describe how Gradient Boosting works?
* It's really just a framework for iteratively improving any weak learner.
You start with a model `Y = M(x) + error`
You create a new model `error = G(x) + error_2`
You create a new model `error_2 = H(x) + error_3`
You combine theses model `alpha * M(x) + beta G(x) + gamma H(x) + error_3`

### Explain cross-validation?
Simple set of little ball with half red, half green, cross-validation with 2 holdouts, you pick all the red for the first training, and learn `p(red)=1`, `p(green)=0`, then in the second holdout, you pick only the green `p(green)=1`, `p(red)=0`, hence if you average the score you `p(red) = 1/2 = p(green)` so you have reduced the variance, which means your estimate has less variance.

### How PCA works?
* you can perform PCA on the correlation or covariance matrix. The primer one standardizes the data. It is good to use this one when features have different scales
* compute the correlation matrix of the features `XX^T`. 
* Can use SVD to find the eigenvectors, but any matrix can be rewritten as SVD, so `XX^t = DV^2D^t`. The column of `D` contains the eigenvectors, and the value in `V` are the singular values. 
* The eigenvalues = square singular values/num examples
* the principal components are the eigenvectors of the correlation matrix

### What are ROC curves, and Area under the Curve?
It is the plot of the true positive (y-axis) with respect to the false positive (x-axis) for all classification threshold possible (0 to 1)  
True positive: true positives/all positives  
False positive: false positives/all negatives  
A good classifier has a curve that it as high as possible.  
AUC gives a score for this curves is the area of all square that his under the ROC. If the area is 0.5 the classifier is bad (random), and if the `AUC=1`, the classifier is perfect

ROC curves would work even if the classification range (usually from 0 to 1), is calibrated from 0.9 to 1.
AUC represents the probability that the classifier will rank randomly a positive observation higher than a negative observation
If the curve is below the random line, the classifier is worse than random.  

### What is heteroskedasticity?
When you plot the residuals, you can look and see if the standard deviation of your estimate and the sampled point is larger at some place than in others, then you have heteroskedasticity

### Talk about distributions?
* Gaussian, bell-shaped, 95% of the values are in mean_hat + 1.96*std(mean_hat)
* Binomial, probability of a certain binary outcome (running multiple Bernoulli experiences)
* Poisson distribution, given that a certain outcome occurs lambda time in a certain time, what is the probability that it appears x times. This distribution looks like a decreasing exponential if lambda is small, but if lambda becomes large, it looks like a Gaussian of mean lambda and std sqrt(lambda)
* t-Distribution is looking like a normal, except that it has a large tail if there is not a lot of values in the sampling set. When the sampling set becomes larger, the density curve starts to look like a normal distribution. You use the t-distribution when the variance is unknown, or no gaussian assumption has been made on the variance.
* chi-square distribution with `k` degree of freedom is the distribution of a sum of the squares of k independent standard normal random variables

# Theory
### What is the effect on the coefficients of logistic regression if two predictors are highly correlated? What are the confidence intervals of the coefficients?
We can't expect coefficient to make sense because if the model is of the form `Y=X1+X2+noise`, and `X1~=X2`, then we can rewrite  `Y=2X1, Y=-X1+3X2`... so the analysis of the coefficient is not relevant.
In case one predictor is an exact linear combination of another, then if we use the Ordinary Least Square method to find the parameter, we have a matrix whose rank is smaller than the size of the matrix. so we can't invert the matrix, and the solution is unstable.

### What are the assumptions of Regression?
Ordinary Least Squares is a method for estimating the unknown parameters in a linear regression model with the goal of minimizing the sum of the squares between the observed responses in a given dataset and those predicted by a linear function. It is the maximum likelihood estimator under the normality assumption.
OLS assumes:
* Linearity of the observed variables with respect to the true relationship. 
-> Plot residual, trend line

* No autocorrelation: The error term of one observation is independent of any other observation
-> This occurs often in time series. If this assumption, the model is probably misspecified, i.e, model nonlinear

* The sampling size is sufficiently large or the population error term is normal
-> imply error term is normally distributed
-> imply the sampling distribution for the b term will be normal
-> can check this with a histogram of the residual
-> Can use Q-Q plot or Kolmogorov-Smirnov test

* The variance of the error term should be consistent (stationary variance, homoscedastic, not heteroscedastic)

* No correlation between explanatory variables:
-> Plot correlation matrix
-> Compute Variance Inflation Factor. It is a factor for each explanatory variable, that represents how large the standard error for the coefficient compared  to the case where the explanatory variable would be uncorrelated with the other variables


# Visualization
### Difference between histogram and box plot?
A histogram is used to measure frequencies of data group, while box plot gives us information about the mean, the first and third quartiles. The histogram is more useful when there is very little variance among the observed frequencies or very large variance among the frequencies. Histogram helps to see the mods. If there is an average amount of variance, it might be difficult to see on the histogram that it is actually a normal distribution, while the box plot might be easier to see

### What is the Q-Q Plot?
It compares the quantile of a sample data with the quantile of the approximated distribution used to adjust the point. 

# Practical questions
### What are the steps for wrangling and cleaning data before applying machine learning algorithms?
* selecting the attributes,
* removing the outliers,
* decorated features,
* transforming nonnumerical data,
* splitting

### How would you approach a categorical feature with high-cardinality?
* For linear regression, it is not a problem, you will one hot your features, and create a weight for each of those
* If you sample some features, some columns (like Boosting, random forest), yes model might learn on the same variable, hence increasing the correlation between the models

### Why do you use feature selection?
Some features are irrelevant, might confuse the model, might be correlated with others 

### When to use Gaussian Mixture instead of K-Means?
GMM can have unconstrained covariance, which is more flexible than K-means. 

### How do you deal with unbalanced binary classification?
* resample (add or delete)
* cost-sensitive learning
* change the metric, not accuracy, but recall, precision, 
* generate synthetic data: `SMOTE` algorithm for example

### Name and describe three different kernel functions and in what situation you would use each?
Kernel is like similarity measure, so having a good prior knowledge, for example, if the feature should be invariant to translation, rotation
* Linear kernel: `x^T y`
* Polynomial: `(x^Ty + 1)^d`
* Sigmoid: `tanh(ax^Ty + b)`
* RBF: `exp(−gamma|x − y|2)`

### How do you deal with outliers in your data?
Analyze, see box plot, 3 times the interquartile, if you know the underlining distribution...

### What is cross-validation and why would you use it?
If you have small data, then the accuracy of a model might vary a lot given the sample test set, so it's better to do cross-validation with 10 holds out for example so as to reduce variance

### Solve multicollinearity?
* Don't use as many dummy variable as categorical features, but always minus one. 

### How to standardize the data? What is the z score
Given the true population mean, and population standard deviation, z score is equal to 
Z=(X - E(X))/std(X). If the mean or the std are unknown, we need to use the sampled mean and the standard error.

### How to test if a coin is fair? 
* n is the number of experiments
* X is the number of heads, it is the sum of X_i random variables, X_i is a binomial variable(n, p). p will be our bias. 
* We can look at the probability mass function of a binomial variable (such as X).
* Asking if the coin is fair, is looking if p=0.5. H_0 = 0.5. As the number of sample goes to infinity, there should be as much head as tails.
* We do some test on the coin, get 15 heads out of 20 times. We can use the cumulative distribution to compute the probability that we have at least 15 heads P(X >= 15|p=0.5), we found a very small probability, it will be our p-value for the test. It is the probability of getting a value as or more extreme than our data if the null hypothesis is true
* In our case, we only looked up for the appearance of too many heads, but maybe there is too much tail, and in our case, our p-value would be large.  Because of the Central Limit Theorem, we can multiply by two the p_value to remove the cases where there are too many tails. so the final p-value is 2*P(X >= 15|p=0.5). 
* For example, p_value =0.04, so it is smaller than 0.5, so we can say that we have a significant result, it would have been unlikely to see 15 heads or more given our hypothesis that the coin is fair.

### Estimate the bias in my coin?
We can compute the sampled mean very easily. If we want to find the uncertainty of this statistic (the sample mean), we can use the standard error, we can use the formula of a normal distribution, as X is defined as a sum of random variables. so the bias of the coin is `p_hat +- sqrt(p_hat(1-p_hat) / n)`



# Statistical Testing
### Parametric tests
#### Assumption of normality
Parametric tests assume the data to follow a particular distribution. This is the case of the ANOVA, the t-tests, or the regression.

With a One way ANOVA, you can check normality in the residuals
With a repeated measures ANOVA, you can check the residuals at each time point
With a Pearson correlation coefficient, you can check the normality of both variables
With an Independent t-test, you can check ...

### Nonparametric tests
Nonparametric tests make no assumptions about the distribution of the data. They are usually based on ranks or signs rather than the actual data.

### Multiple testing
The Bonferroni adjustment is the most conservative (least likely to lead to rejection of the null). It divides the significance level initially used by the number of tests being carried out and compare the p-value with the new smaller significance. Or you can multiply the p-value by the number of tests being carried out and compare to 0.05.

### List of tests:
#### Mann Whitney/Wilcoxon rank-sum test
* Purpose: Test whether two distributions are the same
* Assumption: 
    * Dependent variable is ordinal or continuous
    * Independent variables are all categorical features with two classes
* Algorithm:
    Compute the rank of the value of two distributions merged. If the sum of the rank of their respective value is the same, then the distribution are probably the same

#### Kruskal Wallis test
* Purpose: Test if the sample has come from a different population. Extension to the Mann Whitney U. 
* Assumption:
    * Dependent variable is ordinal
    * Independent variables are categorical
    * Observations are independents
    * The sample size are similar and all larger than 5
* Algorithm:
    * Compare the medians of the samples
* Interpretation:
If it leads to significant results, then at least one of the samples is different from the other samples.

#### Independent t-test
* Purpose: Test whether the mean of two independent group is significantly different 
* Assumption: 
    * Dependent variable is continuous and normal in each group
    * Independent variables are all categorical features with two classes
    * Variance within each group is equal
* Algorithm

#### Paired t-test
* Purpose: Test if there is a statistically significant difference two experiments. Report the mean difference
* Assumption:
    * Dependent variable is continuous
    * Independent variables are time point
    * Normality paired differences should be normally distributed
* Algorithm:
    The test assesses whether the mean of the paired differences is zero

#### Wilcoxon Signed Rank test
* Purpose: Test if there is a statistically significant difference two experiments. Report the median of the two set of measurements
* Assumption:
    * Dependent variable is ordinal/continuous
    * Independent variables are time/condition

#### One way ANOVA
* Purpose: Generalisation of the paired t-test to 3 or more independent groups. 
* Assumption:
    * Dependent variable is continuous
    * Independent variable is categorical (at least 3 categories)
    * Residuals should be normally distributed
    * Variance within each group is equal
* Hypothesis: Ho: All group means are equal

#### Chi-squared test:
* Purpose: Test the assumption that there is no relationship/association between the two categorical variables
* Assumption:
    * Dependent variable is categorical
    * Independent variable is categorical

#### Pearson Correlation
* Purpose: Measure of correlation
* Assumption:
    * Dependent variable is continous
    * Independent variables are continous
    * Variables are linearly related variable
    * Variables are normally distributed

#### Spearman Rank Correlation
* Purpose: Measure of correlation and non-parametric statistical measure of the monotonic relationship between paired data
* Assumption:
    * No assumption