# Paper

* **Title**: Rank Ordered Autoencoders
* **Authors**: Paul Bertens
* **Link**: https://arxiv.org/abs/1605.01749
* **Tags**: Neural Network, autoencoder, rank order
* **Year**: 2016

# Summary

* What
  * Autoencoders typically have some additional criterion that pushes them towards learning meaningful representations.
    * E.g. L1-Penalty on the code layer (z), Dropout on z, Noise on z.
    * Often, representations with sparse activations are considered meaningful (so that each activation reflects are clear concept).
  * This paper introduces another technique that leads to sparsity.
  * They use a rank ordering on z.
  * The first (according to the ranking) activations have to do most of the reconstruction work of the data (i.e. image).

* How
  * Basic architecture:
    * They use an Autoencoder architecture: Input -> Encoder -> z -> Decoder -> Output.
    * Their encoder and decoder seem to be empty, i.e. z is the only hidden layer in the network.
    * Their output is not just one image (or whatever is encoded), instead they generate one for every unit in layer z.
    * Then they order these outputs based on the activation of the units in z (rank ordering), i.e. the output of the unit with the highest activation is placed in the first position, the output of the unit with the 2nd highest activation gets the 2nd position and so on.
    * They then generate the final output image based on a cumulative sum. So for three reconstructed output images `I1, I2, I3` (rank ordered that way) they would compute `final image = I1 + (I1+I2) + (I1+I2+I3)`.
    * They then compute the error based on that reconstruction (`reconstruction - input image`) and backpropagate it.
  * Cumulative sum:
    * Using the cumulative sum puts most optimization pressure on units with high activation, as they have the largest influence on the reconstruction error.
    * The cumulative sum is best optimized by letting few units have high activations and generate most of the output (correctly). All the other units have ideally low to zero activations and low or no influence on the output. (Though if the output generated by the first units is wrong, you should then end up with an extremely high cumulative error sum...)
      * So their `z` coding should end up with few but high activations, i.e. it should become very sparse.
    * The cumulative generates an individual error per output, while an ordinary sum generates the same error for every output. They argue that this "blurs" the error less.
  * To avoid blow ups in their network they use TReLUs, which saturate below 0 and above 1, i.e. `min(1, max(0, input))`.
  * They use a custom derivative function for the TReLUs, which is dependent on both the input value of the unit and its gradient. Basically, if the input is `>1` (saturated) and the error is high, then the derivative pushes the weight down, so that the input gets into the unsaturated regime. Similarly for input values `<0` (pushed up). If the input value is between 0 and 1 and/or the error is low, then nothing is changed.
  * They argue that the algorithmic complexity of the rank ordering should be low, due to sorts being `O(n log(n))`, where `n` is the number of hidden units in `z`.

* Results
  * They autoencode 7x7 patches from CIFAR-10.
  * They get very sparse activations.
  * Training and test loss develop identically, i.e. no overfitting.