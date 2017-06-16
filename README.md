# Winning Model Documentation

Name: Shaoxiang Chen<sup>1</sup>, Xi Wang<sup>1</sup>, Yongyi Tang<sup>2</sup>, Xinpeng Chen<sup>3</sup>, Zuxuan Wu<sup>1</sup>, Yu-Gang Jiang<sup>1</sup>

Location: 
1. Fudan University, Shanghai, China
2. Sun Yat-Sen University, Shenzhen, China
3. Wuhan University, Wuhan, China

Email: {sxchen13,xwang10,zxwu,ygj}@fudan.edu.cn, tangyy8@mail2.sysu.edu.cn, xinpeng_chen@whu.edu.cn

Competition: Google Cloud & YouTube-8M Video Understanding Challenge

This is the code repository for our winning submission of the Google Cloud & YouTube-8M Video Understanding Challenge. The yt8m-* folder contains model, training and testing code from three teams before we merged together. The fusion folder contains code used to generate our final ensemble.

Our code is based on the [YouTube-8M Tensorflow Starter](https://github.com/google/youtube-8m) Code.


## 1. Background on you/your team

We are master students and professor(Yu-Gang). Shaoxiang, Xi and Yu-Gang are in the same university(Fudan University) lab and Zuxuan, Yongyi and Xinpeng are our collaborators. We all have previous research experience in the field of video classification. We decided to enter this competition because of research interest. We started late, spent about one and a half month on the competition. In the challenge, Shaoxiang, Xi and Yongyi work on frame level method, Zuxuan and Xinpeng work on video level methods.


## 2. Summary
The training methods we used are as the following:

| Model | Variations |
| --- | --- |
|LSTM|None|
|LSTM|Layer normalization & Recurrent dropout|
|RNN|Residual connections|
|GRU| None|
|GRU| Bi-directional|
|GRU| Recurrent dropout|
|GRU| Feature transformation|
|RWA| None|
|NetVLAD| None|
|DBoF| None|

For detailed descriptions of these models, please refer to our paper and code.
The model definitions are all in `frame_level_models.py`.
We used all the features provided in the dataset, including video and audio features in both video and frame level.
All of our codes are written in Python and based on TensorFlow. Our frame level models usually takes 3-5 days to train,
video level models takes several hours to train.

## 2. Features Selection / Engineering
There are only 4 kinds of features provided: video and audio features in both video and frame level. We think frame level features are more important since they contain more information, and video features are more important than audio features based on the experiments. Using only audio features to train the models got very poor performance. We simply used all the features

We made two kinds of feature transformations for frame level features:

  1. We take the difference between adjacent features, then feed the resulting sequence to our models.
  2. We subsample the sequence with stride of 1,5, or 10. In this way our models are trained on features at different time scales.

By feature transformations, the models trained on new features can produce results that are complementary to those trained on original features. When making an ensemble of these results, the overall performance are better. We did not use any external data.

## 4. Training Method(s)

We trained our models with gradient descent and Adam optimizer from TensorFlow. We ensemble our model predictions to get better results. The ensemble is done in two stages:
Stage 1. We get predictions from model checkpoints saved during training. The checkpoint is chosen when the
model is trained for more than 3 epochs. We fuse these predictions as a final result for this model. This can be regarded
as intra-model fusion.
Stage 2. We fuse predictions from different models generated in Stage 1 to get our final prediction. And this can be
regarded as inter-model fusion.

Our fusion weights are decided in three ways:
1. Empirical fusion weights
    At stage 1, we assign higher weight for predictions generated by newer checkpoints. The sum of the weights inside one type of model sum to 1. At stage 2, we assign higher weight to the model prediction that achieved higher test score. It turns out this empirical strategy works well.
2. Searching for fusion weights
    With validation set, we can try different combinations of fusion weights for all the model predictions we are fusing. We choose the fusion weights that achieved the best score on validation set and apply it to predictions on test set. We search for weights in the range of [0, 2] with step size of 0.1.
3. Learning for fusion weights
Since the test score on validation set is a function of the fusion weights. We treat each prediction as a data point, then train a linear regression model to learn the fusion weights. The labels used here are also class labels. This process can be applied to both stage 1 and 2.

## 5. Interesting findings

Although not new, we found that fusion of model predictions from different checkpoints of the same model can improve performance.

## 6. Simple Features and Methods

With video level features along, we can train a model with pretty good performance. With frame level features we can get very good results: A 2 layer GRU models with 2048 cells for each layer can get 0.81786 GAP@20. So the most important models are frame level RNNs and their variants. The majority of our models are these RNNs. 
 

# Appendix

## A1. Model Execution Time
The software we used for training and prediction is TensorFlow.

The hardware specs are: 

| Hardware | specs |
| --- | --- |
|CPU|i7-5820k, 12 cores |
|GPU|GeForce GTX TITAN X|
|Memory|DDR3 64GB|

The model execution times are:

| Type | Time |
| --- | --- |
|Train|3-5 days each|
|Generate predictions|2-5 hours|
|Train the simplified model|3-5 days|
|Generate predictions from the simplified model|about 3 hours|

## A2. Dependencies
The software dependencies are:

| Software | Version |
| --- | --- |
|Ubuntu OS|14.04|
|Python|2.7|
|Tensorflow|1.0.0|
|CUDA|8.0|
|gcc|4.8.4|
|cuDNN|5.0|

## A3. How To Generate the Solution (aka README file)


### 1. Train the models
As shown in the following tables, to obtain the results, you first need to run the training scripts in corresponding folders to get predictions from all the model checkpoints.

Table 1. Models from team Shaoxiang Chen. In branch `sxc`.
If not otherwise stated, the models here are trained on both training and validation data. The training scripts can be found in `train_scripts` directory of branch `sxc`.

| Model| Prediction at checkpoints| Intra-model fusion weights |Inter-model fusion weights
| ------ | ------ | ------ | ------ |
|LSTM| 353k, 323k, 300k, 280k| 0.4, 0.3, 0.2, 0.1 |1.0|
|GRU| 69k, 65k, 60k, 55k| 0.4, 0.3, 0.2, 0.1 |1.0|
|RWA| 114k, 87k, 75k, 50k| 0.4, 0.3, 0.2, 0.1| 1.0|
|GRU with recurrent dropout| 56k, 50k, 46k, 40k, 35k| 0.4, 0.3, 0.2, 0.05, 0.05| 1.0|
|NetVLAD| 24k, 21k, 19k, 16k, 13k| 0.4, 0.3, 0.2, 0.05, 0.05| 1.0|
|MoE| 127k, 115k, 102k, 90k| 0.4, 0.3, 0.2, 0.1| 0.5|
|DBoF| 175k, 150k, 137k, 122k, 112k| 0.4, 0.3, 0.2, 0.05, 0.05| 0.5|
|GRU with batch normalization| 86k, 74k, 65k, 49k| 0.4, 0.3, 0.2, 0.1| 0.25|
|Bidirectional GRU| 53k, 45k, 35k| 0.5, 0.3, 0.2 |0.25|
|RNN with residual connection | 75k, 70k, 62k, 56k | 0.4, 0.3, 0.2, 0.1 | n/a |

Table 2. Models from team Xi Wang. In branch `xiw`, `diff`(with Feature Transformation), `filter`(with Label Filter).
If not otherwise stated, the models here are trained only on training data. For detailed training settings, please refer to
the corresponding branches' README file.

|Model |Prediction at checkpoints| Fusion weights|
| ------ | ------ | ------ | 
|LSTM with Layer Normalization & Dropout = 0.5, trained on training+validation data| 286k, 240k, 222k, 203k, 144k| 0.8, 0.8, 0.8, 0.8, 1.0|
|LSTM with Layer Normalization & Dropout = 0.75| 158k, 150k| 1.3, 0.6|
|GRU| 98k, 80k, 61k| 1.6, 0.8, 0.8|
|LSTM| 72k| 0.1|
|GRU with Feature Transformation, trained on training+validation data|108k, 80k, 61k| 0.6, 0.1, 0.1|
|GRU with Feature Transformation| 108k | 0.3 |
|GRU with Feature Transformation, learning rate = 0.0005| 124k | 0.3 |
|MoE with Label Filter (3571 categories) |27k| 0.8|
|MoE with Label Filter (2534 categories)| 27k| 0.8|
|MoE| 23k| 0.7|
|DBoF| 17k| 0.5|

Table 3. Models from team Yongyi Tang. In branch `yyt`.
The models are trained on both training and validation data except validate[0-2]*.tfrecord. The training scripts can be found in the `train_scripts` directory of branch `yyt`.

|Model | Prediction at checkpoints|
| ------ | ------ |
|catfus_maxout_MoeModel (init 1) | 90k |
|catfus_maxout_MoeModel (init 2) | 90k |
|catfus_maxout_MoeModel (init 3) | 90k |
|catfus_maxout_MoeModel (init 4) | 90k |
|maxout_MoeModel (init 1) | 90k |
|maxout_MoeModel (init 2) | 90k |
|maxout_MoeModel (init 3) | 90k |
|maxout_MoeModel (init 4) | 90k |
|audio_avgShort_twowayGRUModel (stride=1, init 1) | 68k, 76k, 88k, 96k, 104k, 114k, 119k|
|audio_avgShort_twowayGRUModel (stride=1, init 2) | 76k, 93k, 105k, 109k, 112k, 114k, 116k, 119k|
|audio_avgShort_twowayGRUModel (stride=5, init 1) | 50k, 64k|
|audio_avgShort_twowayGRUModel (stride=5, init 2) | 50k, 64k|
|audio_avgShort_twowayGRUModel (stride=5, init 3) | 45k, 67k|
|audio_avgShort_twowayGRUModel (stride=10, init 1) | 42k, 67k|
|audio_avgShort_twowayGRUModel (stride=10, init 2) | 47k, 62k|
|audio_avgShort_twowayGRUModel (stride=10, init 3) | 47k, 70k|
|audio_avgShort_twowayGRUModel (stride=10, init 4) | 47k, 70k|
|pur_twowayGRUModel(inti 1) | 31k, 38k, 44k, 50k, 57k, 63k, 69k|
|pur_twowayGRUModel(inti 2) | 24k, 32k, 40k, 48k, 56k, 64k, 72k|
|pur_twowayGRUModel(inti 3) | 24k, 32k, 40k, 48k, 56k, 64k, 72k|
|pur_twowayGRUModel(inti 4) | 24k, 32k, 40k, 48k, 56k, 64k, 72k|
|resav_ConvModel | 193k, 215k, 236k, 257k, 279k, 307k |

### 2. Run inference

We used the provided `inference.py` code from the original repository of YouTube-8M Tensorflow Starter Code.
The general command used to run inference for a specific model is:

For frame level models:
```sh
python inference.py \
    --output_file="The output file name" \
    --input_data_pattern='/path/to/dataset/test*.tfrecord' \
    --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128"\
    --batch_size=64 \
    --top_k=32 \
    --model=ModelName --train_dir="Directory where checkpoints are stored" --run_once=True
```

For video level models:
```sh
python inference.py \
    --output_file="The output file name" \
    --input_data_pattern='/path/to/dataset/test*.tfrecord' \
    --frame_features=False --feature_names="mean_rgb,mean_audio" --feature_sizes="1024,128"\
    --batch_size=64 \
    --top_k=32 \
    --model=ModelName --train_dir="Directory where checkpoints are stored" --run_once=True
```

Basically, you only need to specify the `output_file` and `ModelName` in the command for different models.

### 3. Ensemble the models

For the model predictions on test set obtained as shown in Table 1, you need to run `weighted_fuse.py` first to fuse the results generated by the same models, with the corresponding intra-model fusion weights. Then run `weighted_fuse.py` again to fuse the resulting ensembles. The script is provided as `fuse_sxchen.sh` in the `fusion` folder

For the model predictions on test set obtained as shown in Table 2, you run the `simple_fusion.py` in the `fusion` folder to obtain an ensemble for all the models listed in Table 2. The fusion weights are searched based on their performance on validation set, so you need to get the predictions on validation set as well.

For the model predictions on test set obtained as shown in Table 3, you need to use `valid.py` in the `fusion` folder first to fuse the results generated by the same models. The usage is the following:
```py
from valid import val
ens = val([‘valid_input_1.csv’, ‘valid_input_2.csv’], ‘valid_label.csv’) # learning on validation set predictions
ens.predict()
ens.employ([‘test_input_1.csv’, ‘test_input_2.csv’], ‘test_output.csv’) # apply the learned weights to test set predictions
````
Since the fusion weights are learned on the validation set, you need the predictions on validation set as well.
Then the same process can be applied again to fuse the resulting ensembles.

Finally, you run the `fuse_final.sh` script in the `fusion` folder to fuse all the ensembles into a final ensemble.

## A4. References
We used open source code from: 

https://github.com/NickShahML/tensorflow_with_latest_papers [Apache License 2.0]

https://github.com/jostmey/rwa [BSD 3-clause]

The reference of the corresponding papers are also included in our technical report.
