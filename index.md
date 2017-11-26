## Markov Chain Monte Carlo - intuition for new practitioners

Let us say we make n (say 15) observations of an event (employee height), which is mapped to a number (Random Variable). 
Our belief (called the prior) that the random variable follows a normal distribution that has a mean of 70 and standard deviation is known to be 6. 
We have no knowledge of the distribution that generated the data (The posterior)

The requirement is, say, to generate 1000 samples from the unknown posterior distribution.

Markov Chain Monte Carlo enables sample generation from the posterior distribution, without knowing the parameters of the posterior distribution.

MCMC constructs series of 'steps', where each step represents an update to the beliefs about the distribution the observed data is from. The steps are constructed such that the belief doesn't 'converge' to a single value. However, if you look at the chain of steps themselves, you may be able to observe that a (large enough) sequence of k steps is similar to another k step sequence in its characteristics. Each of these k steps then represents a sample from the unknown posterior distribution.

There are three steps to generating 1 sample and and entry/input condition:

Input condition: you must have a current belief or guess about the unknown (posterior) distribution.

#### Step 1. Update the current guess to a 'proposed' guess using a proposal mechanism. 
The proposal mechanism is a way to perturb the current guess to a different value. 
The perturbation mechanism has nothing to do with the prior belief or the posterior distribution. 
This is a way to generate guesses. 
e.g. one could use a uniform distribution or a normal distribution centered around the current guess to generate the next guess.
for our case, say we use a normal distribution with a mean of 70 (our current guess) and a standard deviation of 3 to generate the proposal. 
Say our proposed mean is 72, generated from this proposal mechanism.

Recall, by perturbing the current guess, we have only generated a proposal for the parameter, 
we have not declared it to an official sample yet. 


#### Step 2. Determine the likelihood of observing the data with current guess and with proposal
When a proposal is generated in step 1, we need to check if it explains the observed data (i.e. our 15 height observations) any better than the current guess. How do we measure the 'better' or 'worse' amd how do we quantify it? 

We need to use a 'likelihood' function, or at least a version that may be proportional to the likelihood function. What is likelihood? It is a very simple concept, but I admit I was stumped when I read it first. Therefore I will take a couple of lines to explain, more knowledgeable practistioners can choose to skip.

***Likelihood function*** is just what it says it is. Likelihood(data) is the probability that "exactly this data" will be observed, given a probability distribution. 

This is how you calculate likelihood function:

*a.* Calculate probability of each observation separately

*b.* Assuming independence betwen all the observations, multiply all the probabilitoes together. That is it!

Note: for our purpose, we do not need to create log likelihoods


Recall that our current guess and the proposal guess represent a parameter for the probability function used in the above calculation. Therefore the likelihood will be different as the guessed parameter changes.

#### Step 3. Proposal Acceptance/Rejection
If the likelihood of the data (being observed) increases with tne new guess, we accept the guess.

If the likelihood of the data (being observed) decreases with tne new guess, we sometimes accept the guess and sometimes accept the guess. A good way to think is, we want to accept worse outcomes with lower probability than less worse outcomes.

so our acceptance function looks like 

p_accept = min(1, (likelihood(data) with proposal)/(likelihood(data) with current guess))

If the proposal is accepted, the proposal is added as a sample from mcmc; the proposal now becomes the current guess and the you can go back to step 1 to generate the next sample.

#### Why does this work?
Why do this series of steps work to generate a sample from the true posterior distribution? After all, we don't see a convergence to the 'actual' distribution. The proposals just seem to be bouncing around! also, how can the proposal be a sample itself?

The answer is, with each mcmc step, the acceptance algorithm (Called the Metropolis-Hastings sampler) tries to push towards a distribution that explains the data better. The operative word here is 'tries'. Since we don't know the true posterior distribution, only accepting proposals that explain the data better wcould move the proposal towards a local convergence, which may lead to biased samples. The exploratory part of the sampler aceepts proposals that worsen the likelihood, but with a lower probability. Here, you can see that the acceptance function is shaping the sample space based on the explanatory power of the sample. One can see that proposal means with lower explanatory power are further from a 'true' mean and have a lower chance of being selected as a sample. This fits in well with the idea that if draws were being done from the true posterior, samples that explain the data better have a higher chance of being drawn and vice-versa.

#### Does my initial guess matter?
Ideally, no. Practically, maybe. If the initial guess is far away from the 'true' value of the parameter, the mcmc may take a longer time to 'converge'. However, it is expected to converge after a while. Since we don't know what convergence will exactly look like, it may be practial to throw away first few (10?, 1000?, 10000?) and take the ones after that


#### Does it matter how far the proposal is from the current guess?
The width of the jump is a tuning question, balancing between quicker converge versus the acceptance probability for a proposals. Ability to make a bigger jump may at times lead to faster convergence, but may lower the ratio of acceptances to proposals made.

