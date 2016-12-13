# openbr-docker

A build version of openbiometrics.org to easy use with Docker

```
docker pull kevinsimper/openbr
```

To see an example on how to use this, go to https://github.com/kevinsimper/face-recognition

Run a simple command:

```
$ docker run kevinsimper/openbr br -help
<arg> = Input; {arg} = Output; [arg] = Optional; (arg0|...|argN) = Choice

==== Core Commands ====
-train <gallery> ... <gallery> [{model}]
-enroll <input_gallery> ... <input_gallery> {output_gallery}
-compare <target_gallery> <query_gallery> [{output}]
-eval <simmat> [<mask>] [{csv}] [{matches}]
-plot <csv> ... <csv> {destination}

==== Other Commands ====
-fuse <simmat> ... <simmat> (None|MinMax|ZScore|WScore) (Min|Max|Sum[W1:W2:...:Wn]|Replace|Difference|None) {simmat}
-cluster <simmat> ... <simmat> <aggressiveness> {csv}
-makeMask <target_gallery> <query_gallery> {mask}
-makePairwiseMask <target_gallery> <query_gallery> {mask}
-combineMasks <mask> ... <mask> {mask} (And|Or)
-cat <gallery> ... <gallery> {gallery}
-convert (Format|Gallery|Output) <input_file> {output_file}
-evalClassification <predicted_gallery> <truth_gallery> <predicted property name> <ground truth proprty name>
-evalClustering <clusters> <truth_gallery> [truth_property [cluster_csv [cluster_property]]]
-evalDetection <predicted_gallery> <truth_gallery> [{csv}] [{normalize}] [{minSize}] [{maxSize}]
-evalLandmarking <predicted_gallery> <truth_gallery> [{csv} [<normalization_index_a> <normalization_index_b>] [sample_index] [total_examples]]
-evalRegression <predicted_gallery> <truth_gallery> <predicted property name> <ground truth property name>
-evalKNN <knn_graph> <knn_truth> [{csv}]
-pairwiseCompare <target_gallery> <query_gallery> [{output}]
-inplaceEval <simmat> <target> <query> [{csv}]
-assertEval <simmat> <mask> <accuracy>
-plotDetection <file> ... <file> {destination}
-plotLandmarking <file> ... <file> {destination}
-plotMetadata <file> ... <file> <columns>
-plotKNN <file> ... <file> {destination}
-project <input_gallery> {output_gallery}
-deduplicate <input_gallery> <output_gallery> <threshold>
-likely <input_type> <output_type> <output_likely_source>
-getHeader <matrix>
-setHeader {<matrix>} <target_gallery> <query_gallery>
-<key> <value>

==== Miscellaneous ====
-help
-gui
-objects [abstraction [implementation]]
-about
-version
-daemon
-slave
-exit
```


# Face Recognition

This tutorial gives an example on how to perform face recognition in OpenBR. OpenBR implements the 4SF2 algorithm to perform face recognition. Please read the paper for more specific algorithm details.

You can easily run face recognition with docker. You can git clone this repository and run this command:

```
docker run -v $(pwd):/tmp/downloads/ kevinsimper/openbr bash -c " br -algorithm FaceRecognition \
    -compare ../data/MEDS/img/S354-01-t10_01.jpg ../data/MEDS/img/S354-02-t10_01.jpg \
     -compare ../data/MEDS/img/S354-01-t10_01.jpg ../data/MEDS/img/S386-04-t10_01.jpg
```

To start, lets run face recognition from the command line. Open the terminal and enter

```
$ br -algorithm FaceRecognition \
    -compare ../data/MEDS/img/S354-01-t10_01.jpg ../data/MEDS/img/S354-02-t10_01.jpg \
     -compare ../data/MEDS/img/S354-01-t10_01.jpg ../data/MEDS/img/S386-04-t10_01.jpg
```
Easy enough? You should see results printed to terminal that look like

```
$ Set algorithm to FaceRecognition
$ Loading /usr/local/share/openbr/models/algorithms/FaceRecognition
$ Loading /usr/local/share/openbr/models/transforms//FaceRecognitionExtraction
$ Loading /usr/local/share/openbr/models/transforms//FaceRecognitionEmbedding
$ Loading /usr/local/share/openbr/models/transforms//FaceRecognitionQuantization
$ Comparing ../data/MEDS/img/S354-01-t10_01.jpg and ../data/MEDS/img/S354-02-t10_01.jpg
$ Enrolling ../data/MEDS/img/S354-01-t10_01.jpg to S354-01-t10_01r7Rv4W.mem
$ 100.00%  ELAPSED=00:00:00  REMAINING=00:00:00  COUNT=1
$ 100.00%  ELAPSED=00:00:00  REMAINING=00:00:00  COUNT=1
$ 1.8812
$ Comparing ../data/MEDS/img/S354-01-t10_01.jpg and ../data/MEDS/img/S386-04-t10_01.jpg
$ Enrolling ../data/MEDS/img/S354-01-t10_01.jpg to S354-01-t10_01r7Rv4W.mem
$ 100.00%  ELAPSED=00:00:00  REMAINING=00:00:00  COUNT=1
$ 100.00%  ELAPSED=00:00:00  REMAINING=00:00:00  COUNT=1
$ 0.571219

```
So, what is FaceRecognition? It's an abbrieviation to simplify execution of the algorithm. All of the algorithm abbreviations are located in openbr/plugins/core/algorithms.cpp.

It is also possible to:

Evaluate face recognition performance (Note that this requires R to be installed):

```
$ br -algorithm FaceRecognition -path ../data/MEDS/img/ \
-enroll ../data/MEDS/sigset/MEDS_frontal_target.xml target.gal \
-enroll ../data/MEDS/sigset/MEDS_frontal_query.xml query.gal \
-compare target.gal query.gal scores.mtx \
-makeMask ../data/MEDS/sigset/MEDS_frontal_target.xml ../data/MEDS/sigset/MEDS_frontal_query.xml MEDS.mask \
-eval scores.mtx MEDS.mask Algorithm_Dataset/FaceRecognition_MEDS.csv \
-plot Algorithm_Dataset/FaceRecognition_MEDS.csv MEDS
Perform a 1:N face recognition search:

$ br -algorithm FaceRecognition -enrollAll -enroll ../data/MEDS/img 'meds.gal'
$ br -algorithm FaceRecognition -compare meds.gal ../data/MEDS/img/S001-01-t10_01.jpg match_scores.csv
```

Train a new face recognition algorithm (on a different dataset):

```
$ br -algorithm 'Open+Cvt(Gray)+Cascade(FrontalFace)+ASEFEyes+Affine(128,128,0.33,0.45)+(Grid(10,10)+SIFTDescriptor(12)+ByRow)/(Blur(1.1)+Gamma(0.2)+DoG(1,2)+ContrastEq(0.1,10)+LBP(1,2)+RectRegions(8,8,6,6)+Hist(59))+PCA(0.95)+Normalize(L2)+Dup(12)+RndSubspace(0.05,1)+LDA(0.98)+Cat+PCA(0.95)+Normalize(L1)+Quantize:NegativeLogPlusOne(ByteL1)' -train ../data/ATT/img FaceRecognitionATT
```

The entire command line API is documented (http://openbiometrics.org/docs/api_docs/cl_api/).


