# SpkVer_iVector is a project of Speaker Verification using i-Vectors

This is as an example of C++ implemenation of deprecated technology of speaker verification using Universal
Background Model, Total Variability matrix and Projection Matrix.

Speaker verification, or authentication, is the task of confirming that the identity of a speaker
is who they purport to be. An early performance breakthrough was to use a Gaussian mixture model and
universal background model (GMM-UBM). One of the main difficulties of GMM-UBM systems involves
intersession variability.  Intersession variability was then compensated for by using
backend procedures, such as linear discriminant analysis (LDA)
and within-class covariance normalization (WCCN), followed by a scoring, such as the cosine
similarity score. In literature was proposed a method to Gaussianize the
i-vectors and therefore make Gaussian assumptions in the PLDA, referred to as G-PLDA or simplified PLDA.
Further described the common While i-vectors were originally proposed for speaker verification,
they have been applied to many problems, like language recognition, speaker diarization,
emotion recognition, age estimation, and anti-spoofing [10]. Recently, deep learning techniques
have been proposed to replace i-vectors with d-vectors or x-vectors [8] [6].

In this example, you develop a simple i-vector system for speaker verification that uses an
LDA-WCCN backend with either cosine similarity scoring.

**Tools:**
0. mingw730_64 (7.3.0), C++ v. 201103
1. Qt Creator 4.15.0, base on Qt 5.15.2 (MSVC 2019, 64 bitowy), version May 4 2021 01:17:10
2. OpenBLAS-0.3.18-x64
3. QCustomPlot 2.1.0
4. Armadillo 10.7
5. eigen-3.4.0
6. HCopy.exe (part of Microsoft HTK lib,, binary to extract MFCC features)

**To start the code in QTgui**
1. create working dir: variable in mainwindow.cpp *corpora_dir* (path to audio files - in this example PTDB-TUG - University of Graz)
2. create working dir: variable in mainwindow.cpp *workspace_dir*, path to dir with subfolder:
	calc 	- to store checkpoints of model and results
	cfg 	- config for HCopy
	feat	- to store mfcc files
	list 	- to store lists (txt) files with paths with audio files
	logs 	- logs
3. download the PTDB-TUG library and store in *corpora_dir*, split data for example: two speakers as a test data in different folder
4. indicate the dir of HCopy binary variable in mainwindow.cpp  *pHCopy_bin*
5. You are ready to start with default parameters of modeling
6. path pToModel (use only for enrollment) is not uptading after training, so it is imposible to train and evaluate at the same time. To evalute after training paste proper path to dir pToModel (for example \calc\21-12-2018_13h29m30s_gmm128_TV100\) or of the last training. Results will be stored in new dir located in calc directory (files scoreFAR, scoreFRR)
7. You can create Detection Error Tradeoff useing DETpy

Dataset PTDB-TUG consists of 20 English native speakers reading 2342 phonetically rich sentences from the TIMIT corpus [7]. Download and extract the data set. The file names contain the speaker IDs. Default parameters of MFCC are located in mfccextractor.h (20 MFCC + delta + deltadelta (MFCC_D_A)) Initialize the Gaussian mixture model (GMM) that will be the universal background model (UBM) in the i-vector system.

**statBaumWelchVector**
Expand the statistics into matrices and center F(s), as described in [3], such that
N(s) is a C F×C F diagonal matrix whose blocks are Nc(s)I (c=1,...C).
F(s) is a C F×1 supervector obtained by concatenating Fc(s)  (c=1,...C). C is the number of components in the UBM. F is the number of features in a feature vector.

**funkTotalVariabilityVector**
In the i-vector model, the ideal speaker supervector consists of a speaker-independent component and a speaker-dependent component. The speaker-dependent component consists of the total variability space model and the speaker's i-vector.
M=m+Tw
M is the speaker utterance supervector
m is the speaker- and channel-independent supervector, which can be taken to be the UBM supervector.
T is a low-rank rectangular matrix and represents the total variability subspace.
w is the i-vector for the speaker

**funkiVectorFromUBM**
Once the total variability space is calculated, estimate the i-vectors as [4]:
w=(I+T′Σ(−1)NT)′T′Σ(−1)F
At this point, you are still considering each training file as a separate speaker.
However, in the next step, when you train a projection matrix to reduce dimensionality and increase
inter-speaker differences, the i-vectors must be labeled with the appropriate, distinct speaker IDs.
Create a cell array where each element of the cell array contains a matrix of i-vectors across files for
a particular speaker.

**Projection Matrix (in trainTVPMiVector.cpp)**
Many different backends have been proposed for i-vectors. The most straightforward and still well-performing one is the combination of linear discriminant analysis (LDA) and within-class covariance normalization (WCCN). Create a matrix of the training vectors and a map indicating which i-vector corresponds to which speaker. Initialize the projection matrix as an identity matrix.

**performaLDAfunk**
LDA attempts to minimize the intra-class variance and maximize the variance between speakers. It can be calculated as outlined in [4]

**performaWCCNfunk**
WCCN attempts to scale the i-vector space inversely to the in-class covariance,
so that directions of high intra-speaker variability are de-emphasized in i-vector comparisons [9].

**G-PLDA**
This algorithm implemented in this example is a Gaussian PLDA as outlined in [13]. In the Gaussian PLDA, the i-vector is represented with the following equation:

ϕij=μ+Vyi+εij
yi∼Ν(0,Ι)
εij∼Ν(0,Λ−1)

where μ is a global mean of the i-vectors, Λ is a full precision matrix of the noise term εij,
and V is the factor loading matrix, also known as the eigenvoices.
Specify the number of eigenvoices to use. Typically numbers are between 10 and 400.

**Enrollment**
Enroll new speakers that were not in the training data set.
First, split the adsEnrollAndVerify audio datastore object into enroll and verify.
Increasing the number of utterances per speaker for enrollment should increase the performance of the system.
Create i-vectors for each file for each speaker in the enroll set using the this sequence of steps:
- Feature Extraction
- Baum-Welch Statistics: Determine the zeroth and first order statistics
- i-vector Extraction
- Intersession compensation

Then average the i-vectors across files to create an i-vector model for the speaker. Repeat the for each speaker.

**Scoring method: curently only CSS**

***Some of variables:***
uint      numTdim   - Total Variability dimmension
uint numOfComponents- number of commponents in GMM
uint numFeatures    - number of MFCC features in one vector

arma::mat normMean  - [1, numFeatures] - mean values of MFCC vectors, equals number of featues
arma::mat normStd   - [1, numFeatures] - std values of MFCC vectors, equals number of featues
arma::mat normYtrans- [numFrames, numFeatures] - MFCC matrix after normalization normZ
arma::mat T         - [numComponents x numFeatures, numTdim] - total variablity matrix
arma::mat ubmMu


std::multimap<std::string, std::string> multimapUBMSpkMFCC{};
+ std::string          "M1" - spk label,
+ std::string          "j/PTDB_TUG/../1_M1_A.mfc"

std::map<std::string, arma::mat> ivectorPerSpk
+ std::string   "M8"  "M1" "M10" - spk label
+ arma::mat [numTdim, numOfFilesOfSpk] [numTdim, numOfFilesOfSpk] [numTdim, numOfFilesOfSpk]

arma::mat ivectorsTrain     -   [numTdim, numOfAllFiles]

std::map<std::string, arma::mat> ivectorPerFile
+ std::string "M10" - spk label
+ arma::mat [numTdim, numOfFilesOfSpk]

std::vector<uint> utterancePerSpeaker

**References**

[1] Reynolds, Douglas A., et al. “Speaker Verification Using Adapted Gaussian Mixture Models.” Digital Signal Processing, vol. 10, no. 1–3, Jan. 2000, pp. 19–41. DOI.org (Crossref), doi:10.1006/dspr.1999.0361.

[2] Kenny, Patrick, et al. “Joint Factor Analysis Versus Eigenchannels in Speaker Recognition.” IEEE Transactions on Audio, Speech and Language Processing, vol. 15, no. 4, May 2007, pp. 1435–47. DOI.org (Crossref), doi:10.1109/TASL.2006.881693.

[3] Kenny, P., et al. “A Study of Interspeaker Variability in Speaker Verification.” IEEE Transactions on Audio, Speech, and Language Processing, vol. 16, no. 5, July 2008, pp. 980–88. DOI.org (Crossref), doi:10.1109/TASL.2008.925147.

[4] Dehak, Najim, et al. “Front-End Factor Analysis for Speaker Verification.” IEEE Transactions on Audio, Speech, and Language Processing, vol. 19, no. 4, May 2011, pp. 788–98. DOI.org (Crossref), doi:10.1109/TASL.2010.2064307.

[5] Matejka, Pavel, Ondrej Glembek, Fabio Castaldo, M.j. Alam, Oldrich Plchot, Patrick Kenny, Lukas Burget, and Jan Cernocky. “Full-Covariance UBM and Heavy-Tailed PLDA in i-Vector Speaker Verification.” 2011 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), 2011. https://doi.org/10.1109/icassp.2011.5947436.

[6] Snyder, David, et al. “X-Vectors: Robust DNN Embeddings for Speaker Recognition.” 2018 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), IEEE, 2018, pp. 5329–33. DOI.org (Crossref), doi:10.1109/ICASSP.2018.8461375.

[7] Signal Processing and Speech Communication Laboratory. Accessed December 12, 2019. https://www.spsc.tugraz.at/databases-and-tools/ptdb-tug-pitch-tracking-database-from-graz-university-of-technology.html.

[8] Variani, Ehsan, et al. “Deep Neural Networks for Small Footprint Text-Dependent Speaker Verification.” 2014 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), IEEE, 2014, pp. 4052–56. DOI.org (Crossref), doi:10.1109/ICASSP.2014.6854363.

[9] Dehak, Najim, Réda Dehak, James R. Glass, Douglas A. Reynolds and Patrick Kenny. “Cosine Similarity Scoring without Score Normalization Techniques.” Odyssey (2010).

[10] Verma, Pulkit, and Pradip K. Das. “I-Vectors in Speech Processing Applications: A Survey.” International Journal of Speech Technology, vol. 18, no. 4, Dec. 2015, pp. 529–46. DOI.org (Crossref), doi:10.1007/s10772-015-9295-3.

[11] D. Garcia-Romero and C. Espy-Wilson, “Analysis of I-vector Length Normalization in Speaker Recognition Systems.” Interspeech, 2011, pp. 249–252.

[12] Kenny, Patrick. "Bayesian Speaker Verification with Heavy-Tailed Priors". Odyssey 2010 - The Speaker and Language Recognition Workshop, Brno, Czech Republic, 2010.

[13] Sizov, Aleksandr, Kong Aik Lee, and Tomi Kinnunen. “Unifying Probabilistic Linear Discriminant Analysis Variants in Biometric Authentication.” Lecture Notes in Computer Science Structural, Syntactic, and Statistical Pattern Recognition, 2014, 464–75. https://doi.org/10.1007/978-3-662-44415-3_47.

[14] Rajan, Padmanabhan, Anton Afanasyev, Ville Hautamäki, and Tomi Kinnunen. 2014. “From Single to Multiple Enrollment I-Vectors: Practical PLDA Scoring Variants for Speaker Verification.” Digital Signal Processing 31 (August): 93–101. https://doi.org/10.1016/j.dsp.2014.05.001.

[15] The Hidden Markov Model Toolkit (HTK) is a portable toolkit for building and manipulating hidden Markov models. https://htk.eng.cam.ac.uk/
