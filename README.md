# SiameseCBO
See if we can train a CNN together during a Contextual Bayesian Optimization  
  
Let's consider a simple MNIST printing task. Due to the instability of the instrument, there could be sample drifting during printing, causing distortions in the printed digits. Here we model the drifting velocity as a two dimensional vector along x and y axis, which gradually random walks with the printing going on.  
  
![image](https://github.com/user-attachments/assets/68a3a045-47ce-4e3a-9b8e-248fd987c67e)  
  
To correct the drifting. One can use a CNN to predict the current drift from the observed printed/ground truth image pairs and apply the counter-action to correct the drift. Let's use the data collected so far to test the feasibility of the idea.  
  
![image](https://github.com/user-attachments/assets/80672bf0-7805-4eab-bf00-2ecc98a6ca4b)  
  
It looks like the a Siamese CNN is able to predict the drift velocity (maybe other instrument offsets as well) from the image pairs. There are three worth noting points here:   
1. Until we have collected a sufficient number of training samples (appears to be 500-1000) the model prediction does not make sense and thus useless.  
2. The drift velocity labels provided are highly quantitative. Such labels must require very strong human supervision. It's very inefficient or impractical to generate such labels on experimental data.  
3. For experimental implementation, it's actually not practical to collect the samples with random instrument status. Most of the times, instruments are stably settled at some offsets given the same sample in the same day. It's not practical to wait for other users to mess up the instrument or wait for a few days to collect the next datapoint.  
  
Let's expore an alternative algorithm to solve the issues above:  
  
Here we use a Contextual Bayesian Optimization (CBO) model to solve the problem. The model has a Siamese CNN based embedding network to learn the latent variables (ideally the drift x and y, but let's see) from the image pairs. Then a Gaussian Process layer with four input nodes is used to predict a reward score, where two nodes takes the CNN outputs and the other two represent the counter-actions. The hope is:  
1. In short times or during the initial stages, despite the output of the CNN does not makes sense, some good condition could still be found by the GP layers through BO process. And any data point generated during this process could be used to train the CNN.  
2. Once the CNN is trained, the GP layers do not have to go through a lot of BO steps. Instead, it directly outputs the optimal action.  
  
![Picture2](https://github.com/user-attachments/assets/3d94e3cd-ff3c-4e4a-b4b3-6ac2e711af03)
  
========================================================================  
  
This project is still under development. And the idea above is not fully validated.  
03/14/2025 current stage:  
  
1. The GP layers could be trained with a pre-trained and untrainable CNN and seem to work.  
2. The randomly intialized Siamese CNN overfits crazily when trained together with the GP layers.  
3. Implemented the reward function as the distance to optimal condition at this point. It was supposed to be MSE/SSIM.  
  
03/17/2025 current stage:  
  
1. Achieved the joint training of the Siamese CNN and GP on existing data (no BO): there is still crazy overfitting but not after your have included ~1000 training datapoints. When you don't have enough training data, the CNN overfits by directly learning to map the images to the rewards (mostly ignoring the actions) to get very low loss. Somehow this is doable on a not-big-enough dataset.
2. Implemented the correct reward function as MSE/SSIM. This did not bring any noticeable difference/obstacles in the training behavior.
  
03/24/2025 current stage:  
  
1. Overfitting problem is solved in the 1D demostration by a sampling strategy. Now the algorithm is ready
  
![Demo Animation](https://github.com/lanceyao-research/SiameseCBO/blob/main/animation.gif)  
