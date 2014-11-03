## logitr

Author:             John Paul Helveston

Date First Written: Sunday, September 28, 2014

Most Recent Update: Sunday, November 02, 2014

# OVERVIEW

logitr estimates multinomial and mixed logit models in R. For mixed logit
models, the program can handle only normal and log-normal heterogeneity
distributions (for now) and only under the assumption of uncorrelated
heterogeneity covariances (i.e. a diagonal heterogeneity covariance matrix).
The program can also weight the results by individual choice situation if
desired. A unique feature is that logitr can estimate models in the preference
space or willingness-to-pay (WTP) space. The program can also be configured to
run a multistart loop with different starting points to search for a global
solution (recommended for WTP space models, which have nonlinear-in-parameters
utility functions that may result in multiple local maxima).

The algorithms used are based those in Kenneth Train's book "Discrete Choice
Methods with Simulation, 2nd Edition (New York: Cambridge University Press,
2009)." Mixed logit models are estimated through maximum simulated likelihood.
The main optimization loop uses the optim function to minimize the negative
log-likelihood function, but optimx can also be used if desired.

============================================================================
BASIC USAGE
============================================================================

Put all the files from logitr_v0.1.zip into a directory and use that directory 
as your working directory. Before using logitr, make sure your choice data file
is a properly setup csv file (see "CHOICE DATA FILE SETUP" below). Set all the 
model inputs and control options using the 'modelSetup.R' file in the main
working directory. Once the modelSetup.R file is complete setup, begin the
model estimation by running the entire modelSetup.R file in R.

============================================================================
MODEL OUTPUT
============================================================================

The program produces a single list object called "model" that stores all the
results. The main results of interest are explained below, but "model" also
contains many other values, including the original data and settings assigned
in the "modelSetup.R" file. All values can be accessed by using the "$" symbol.

MAIN VALUES:
model$bestModel     = The original "model" list obect that optim or optimx
                      produces after the optimization has converged to a
                      solution.
model$pars          = The best set of parameters found.
model$hessian       = A symmetric matrix giving an estimate of the Hessian at
                      the solution found.
model$sds           = The standard error of the best set of parameters found.
model$scaledPars    = The best set of parameters found scaled by the scale
                      factors (same as model$pars if scaledParams = FALSE).
model$scaledHessian = A symmetric matrix giving an estimate of the Hessian at
                      the solution found scaled by the scale factors (same as
                      model$pars if scaledParams = FALSE).
model$scaledSDs     = The standard error of the best set of parameters found
                      scaled by the scale factors (same as model$pars if
                      scaledParams = FALSE).
model$logL          = The log-likelihood function evaluated at the best set of
                      parameters found.
model$nullLogL      = The log-likelihood function evaluated with all
                      parameters set to 0.
model$summary       = A summary table including the parameters, standard
                      errors, t-statistics, and significance codes.
model$startPoints   = A matrix of the different starting points used for each
                      run of the optim algorithm (for multistarts).
model$parMat        = A matrix of the best set of parameters found for each
                      run of the optim algorithm (for multistarts).
model$logLMat       = A matrix of the log-likelihood function evaluated at the
                      best parameters found for each run of the optim
                      algorithm (for multistarts).

OTHER VALUES OF INTEREST:
model$covariateSetup = A data frame that summarizes the covariates used in the
                       model and their distributional assumptions (0=fixed,
                       1=normal, 2=log-normal).
model$numRandom      = The number of randomly distributed covariates.
model$numFixed       = The number of fixed (non-random) covariates.
model$numObs         = The total number of observations.
model$betaNames      = The names of the model covariates (fixed and random).
model$allParNames    = The names of all the model parameters.
model$numBetas       = The number of model covariates (fixed and random).
model$numParams      = The number of model parameters.
model$scaleFactors   = A vector of scaling factors used to scale the data (all
                       ones if scaledParams = FALSE).
model$standardDraws  = A matrix of standard normal draws used during the model
                       simulation.

============================================================================
CHOICE DATA FILE SETUP
============================================================================

The choice data file must be in a csv file format. Each row is an alternative
from a choice occasion faced by an individual. The choice occasions do not
have to be symmetric in that they could each have a different number of
alternatives. The columns can be in any order, but the file MUST have a column
for each of the following variables:
1) A variable that identifies each individual choice maker. This must be a
   sequence of numbers that changes by choice maker and repeats for the same
   choice maker. This is used for the "respondentID" variable in the
   modelSetup.R file.
2) A variable that identifies each individual choice occasion. This must be a
   sequence of numbers that changes by choice occasion and repeats for the
   same choice occasion. This is used for the "observationID" variable in the
   modelSetup.R file.
3) A variable that identifies which alternative was chosen for each choice
   occasion, using 1 and 0 for chosen and not-chosen. This is used for the
   "choice" variable in the modelSetup.R file.
4) A variable that identifies the price of each alternative. This is strictly
   required for a WTP space model, but is otherwise optional.
5) All other columns can be any attributes to be used as model covariates.

Optional weighting variable:
A variable for weighting the data by individual or by choice occasion. The
"weight" of each choice situation will be effectively multiplied by these
numbers. For example, a value of 0.1 for a particular choice occasion would
reduce it's weight by a factor of 10, and a value of 10 would increase it's
weight by a factor of 10.

============================================================================
PROGRAM FILES
============================================================================

modelSetup.R:
Main file for setting up the model. Running this file begins the entire
program and is the ONLY file that needs to be run by the user.

createDataObject.R:
This file takes the user-provided model data and settings and puts them all
into a list called "d" (for "data") which stores all of the data, settings,
variables, and model output throughout the entire model estimation. Keeping a
single object that contains all of the data protects against the accidental
use of global variables and makes passing information and data between
functions much easier.

helperFunctions.R:
Contains a series of helper functions for various purposes, including scaling
the model inputs, making start points, printing headers and model output,
running the main optimization loop, and summarizing the model output.

launch.R:
The main file that runs the entire program in the correct sequence.

logitFunctions.R:
Contains all of the log-likelihood and gradient of the log-likelihood
functions for different model types (MNL vs. MXL) and model spaces (preference
vs. WTP).

makeStartPoints.R:
Makes all the start points for running the model estimation optimization.

optimizor.R:
Runs the main optimization loop.

saveResults.R:
Saves the results (duh!)

setLikelihoodFunctions.R:
Given the user provided settings on the model type and space, this file sets
which log-likelihood and gradient functions to use for estimating the model.

setupVariables:
Prepares all the necessary variables and re-formats the data according to the
model type and space. Stores everything in the "d" list of objects.

summarizeResults.R:
Summarizes all model results and renames the "d" object as "model" for the
user to explore the results.

============================================================================
FILE HIERARCHY
============================================================================

The "modelSetup.R" file calls "launch.R," which begins the program and
controls the sequence of the program steps. "launch.R" first loads all the
functions needed for the program by calling the "helperFunctions.R" and
"logitFunctions.R" files. Once these functions are loaded, "launch.R" then
calls the "checkModelInputs.R" file which runs a series of checks on whether
the user-provided information in "modelSetup.R" is correct. If all is good,
then "launch.R" loads the choice data. "launch.R" then calls
"createDataObject.R" which creates the main "d" object for storing all data
and settings. Once the "d" object is created, "launch.R" calls
"setLikelihoodFunctions.R" to set the correct log-likelihood functions to use
in model estimation. "launch.R" then calls "setupVariables.R" to create the
necessary variables for model estimation based on the user-provided settings
and choice data. "launch.R" then calls "makeStartPoints.R" which simply
creates all the starting points to be used in model estimation. "launch.R"
then starts the main optimization loop by calling the "optimizor.R" file.
After running the optimization loops, "optimizor.R" then calls
"summarizeResults.R" to summarize the results. Finally, "launch.R" calls
"saveResults.R" to save the results as a .Rds file.
