# Bayesian Mixed Models in Stan and BRMS

## [Resources](https://github.com/julianquandt/brms-intro-SIPS2019)

## Philosophy of Mixed Models
### Scenario
- Evaluate the effect of a new Tx on Y
  - within = pre/post measurement
  - between = experimental/control
- Possible Approaches
  1. t test between conditions at post
  2. t test on the difference between pre and post over samples
  3. ANCOVA with pre measurement as a covariate then between condition effects at post
- Power Scenarios
  1. Pre and post are not correlated. Will have the lowest power for the change score (RMANOVA).
  2. Moderately correlated then RMANOVA is not as good as a t-test or ANCOVA
  3. Highly correlated there is no difference
- RM-ANOVA assumes B2 as 1, so is controlling for something that may have no impact. ANCOVA allows more flexibility as it doesn't assume B2 is 1
- See lords paradox

### Measurement Models and Assumptions
- RM-ANOVA people will aggregate data in condition over trials
  - this assumes a normal distribution and equal reliability of all trials (each trial is given equal weight)
  - Assumes equal variance across subjects  
  - Ignores lower level processes, like learning over time
- Can deal with the above problems by doing a two-step least squares (this approach is used for learning effects).
  - This is not that robust to missing data
  - each subject is also equally weighted
- It would be nice to be able to estimate the contribution of each trial/participant
  - number of trials
  - within subject variance

#### Rubins 8-schools model
- 8 schools get an intervention, each school varies differently
- Could take the mean but this ignores variability and assumes equal treatment effect
- Mixed modeling allows better analysis
  - No pooling each subject has its own effect but this is hard to measure reliably
  - complete pooling should show the same effect
  - Partial pooling finds a compromise between each subjects percuiliarity and that knowledge that all subjects are from the same subject
  a. subject A effect is in the context of what is known
- Partial pooling changes the estimates rather than the CI/alpha value

### Motivation for Multilevel models
- models low level processes
- estimates low order to higher order
- uses other available (i.e. trial number)

## Linear Mixed Models
- have nested groups, data from the same source (i.e. trial) are more similar to each other than data from different sources
  - can be trials in subjects or subjects in groups
- Can improve power by partial pooling from other data

## Bayes v NHST
- Frequentists and Bayesians have different perspectives on parameters
  - frequentist assumes a true parameter (generally 0). What is the true parameter and how does my data compare to this
  - bayes assumes no true parameter value. There is degrees of belief that different values should be applied (distribution instead of point estimate)

### Hypothesis testing v parameter estimation
- Hypothesis testing looks at how incompatable the data are with set parameter (0)
  - requires sampling distribution of test statistic
- parameter estimation asks what estimates should I use to predict y
  - looks at distribution to test how much credibility to assign to different values

### Frequentist Example
- statistical world postulates a true parameter (i.e. H0 = .5)
- draw empirical sample and measure the summary statistics
- draw imaginary samples and ask how many samples give summary statistic for H0 sample
- compare empirical sample to hypothesis data, is the empirical data surprising enough (does it cross alpha)

### Bayesian
- assign prior (degree of belief) to parameter values. At edges (far from best guess) it becomes less likely
- Collect data (gives likelihood) use this to calculate posterior (Posterior = (likelihood x prior) / (average likelihood))
- How does the posterior update, is a zero score impossible or fairly certain?
- base prior
  - even across all values
  - each new sample updates the posterior by combing the prior with the likelihood
  - continue to sample and the posterior distribution will update
  - over time the distribution becomes more peaked and will update beliefs about population distribution.
  - Posterior is used as a prior for the next data point
- likelihood computes for how many ways the data could have occurred

#### Markov Chains
- `lme4` uses maximum likelihood estimator
  - estimate moves until the highest peak is found. This can be a problem when there is a local maximum which goes down before reaching the global maximum
  - tries to account for this by using different starting values
- Metropolis algorithms don't look for the highest value but move around and take the ratio of current position to next position. This means it will move up and down the peak to get to the global peak
  - moves if the height changes from one step to another
  - is very slow, can only move one step at a time. Sometimes very far from the peak so want to make big jumps
  - struggles to find complex patterns
- `STAN` uses hamiltonian MCMC, which makes bigger jumps to find peaks
  - `STAN` is implemented in `Rstan` which is wrapped by `brms`

## `brms` translates `lme4` to `STAN` code and gives Byesian output

# BRMS practical
## Bayesian Regression Models in STAN

## Example 1
### Go/No-go training
- n = 80
- 40 food items before and after training
- people trained to go/no go on food. Food type and condition was randomised
- So we don't know if there a low order relationships with apple compared to a banana
### Maximal Model Approach
- give estimate estimate for all fixed effects across grouping variables
### Practical differences between `brms` and `lme4`
- `brms` gives bayes
  - can specify priors
  - HMC can handle complex models, allows for bigger maximal model
  - can fit multiple model types (binomial, exponential etc.) all pre-implimented just need to specify model family
- HOWEVER `brms` is much slower for simple designs (has to be compiled from R to STAN to C++)
- 
