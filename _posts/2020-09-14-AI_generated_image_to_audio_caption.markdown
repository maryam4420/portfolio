---
layout: post
title: AI-Generated Audio Captions For Images
date: 2020-09-15 18:32:20 +0300
description: Generating audio captions for images using artificial intelligence.
img: image_captions/image_captions_cover_photo.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Convolutional Neural Network, Deep Learning, Natural Language Processing, CNN, RNN, LSTM, NLP]
---
# Introduction

Artificial intelligence (AI) has become a hot topic nowadays and it's being used by companies in all industries. There are many examples of great AI technologies that we use in our everyday lives, such as:

- Ride-sharing apps, like Uber and Lyft
- Food and groceries apps, like GrubHub and Instacart
- Recommender engines used by many companies like Amazon, Spotify, and Youtube
- Map and direction apps, like Google Maps
- Smart Reply in Gmail 
- Chatbots available on many apps and websites
- Fraud detection applications, used in financial institutions 

Of course, it's not all unicorns and rainbows! Like any other technology, AI has its downsides and can be misused, but let's focus on the positive here! On a personal level, AI has made our lives more efficient in many ways and will continue to do so. It has also significantly impacted many industries while creating new opportunities and this trend is going to continue! So, we all need to learn more about how AI works and how it impacts us today and in the future!

For this project, I decided to use deep learning, which is a key instrument in AI applications, and generate captions for images. Wait, can we really do that? Yes, we can! Even though doing something like this was inconceivable a few years ago, the recent advancements in deep learning have made it possible. Generating captions, which is basically the ability to understand and describe an image by a computer, is thrilling, but there are also many real-world applications for it. For instance for self-driving cars, where "the car" needs to understand what's on the road and around it. Another example would be CCTV cameras, where we ideally want to recognize the alarming situations quickly to prevent crimes and accidents. But my main motivation for picking up this project was to help the blinds and people visual impairments by creating a technology that converts images to captions and then to speech. 

I did this project using Python and Keras and will explain the data and process in this post. The figure below shows a quick overview of the process:

![Overview](../assets/img/image_captions/Itro_Pic.jpg){: .postImage}

# Data 

To generate meaningful captions, we need to train a model for both images and their descriptions, at the same time. For this project, I used the open-source Flicker 8K dataset from [Kaggle](https://www.kaggle.com/shadabhussain/flickr8k), which includes 8,000 images and five captions for each image. I used 6,000 images and their captions for training and the rest for testing. To show you how the data looks like, I have included below two sample images and their captions. As you can see, each caption is describing the image slightly differently, but the captions are very similar because they are describing the same picture! Alternatively, we can use one caption for each image, but having five captions provides for more training data and so more robust results.

![Sample Image and Captions](../assets/img/image_captions/Sample_Img&Caption1.jpg){: .postImage}
**Captions:**
- A child in a pink dress is climbing up a set of stairs in an entry way.
- A girl going into a wooden building.
- A little girl climbing into a wooden playhouse.
- A little girl climbing the stairs to her playhouse.
- A little girl in a pink dress going into a wooden cabin.

![Sample Image and Captions](../assets/img/image_captions/Sample_Img&Caption2.jpg){: .postImage}
**Captions:**

- A person and their tan dog go for a ride in a rowboat.
- A woman and a dog rowing down a wide river.
- A woman wearing an orange jacket sitting in a canoe on a lake with a dog behind her.
- A woman with a dog canoe down a river.
- Woman and dog in rowboat on the water.

As you can see, we have two kinds of data here: image and text. Before creating a neural network model to generate captions, we need to preprocess and analyze images and captions separately and convert them to a format that the model can understand. I will explain how to do this in the next two sections.

# Preprocessing Images
I used Convolutional Neural Network (CNN) and transfer learning to interpret the content of the images. Transfer learning is a machine learning method where a model developed for a task is reused as the starting point for a model on another task. This is a popular approach in deep learning, and in fact, so much of the progress in deep learning over the past few years is attributable to the availability of such pre-trained models. There are many pre-trained CNN models available to choose from (e.g. VGG16, ResNet50, Xception). For this project, I used InceptionV3, which is an efficient model and has great accuracy. This model was created by Google Research in 2014 and it was trained on ImageNet dataset (which contains 1.4M images in 1,000 image classes).

I converted all the images to size 299x299, as required by InceptionV3, and passed them to the model as inputs. Then instead of training the model all over again, I froze the base layers that are already trained to quickly learn the features for a given image and extracted the resulted feature vectors of 2,048-length (also known as the "bottleneck features"). This is common practice when using pre-trained models. The below image shows inceptionV3's architecture, as well as its input image and output.

![InceptionV3](../assets/img/image_captions/InceptionV3.jpg){: .postImage}

Note that for a classification task, a Softmax function can be applied after this layer, on top of the feature vectors to classify images. But for the task at hand, we only need the feature vectors, which we will later combine with the text data and pass to a neural network model as input. 

# Preprocessing Captions (Text Data)
I performed the following preprocessing steps on the text data:

## 1. Cleaning the Text
I first cleaned the captions to remove noise and reduce the size of the vocabulary of words. This makes the model smaller and faster to train.
- Converted all the words to lower case
- Removed punctuations (e.g. "!", "-")
- Removed all numbers 

## 2. Defining a fixed sequence length and starting/ending points
The way that the final model works is that it will generate a caption, one word a time. So we need to create a starting signal to kick off the generating process. We also need an ending signal to stop the process. Then we can ask the model to stop generating new words if it reaches this signal or a maximum length, which I will explain later. So, I added "startseq" to the beginning and "endseq" to the end of all the captions in the training data. Here's an example of how the captions (from the first image above) were modified after this step:


- **startseq** a child in a pink dress is climbing up a set of stairs in an entry way **endseq** 
- **startseq** a girl going into a wooden building **endseq** 
- **startseq** a little girl climbing into a wooden playhouse **endseq** 
- **startseq** a little girl climbing the stairs to her playhouse **endseq** 
- **startseq** a little girl in a pink dress going into a wooden cabin **endseq** 

Generally, the input sequences for a neural network model should have the same length (because we need to pack them all in a single tensor). For example, when working with text data such as reviews, it is common to truncate them to a reasonable length and make them equal length. For the case of the captions, since they are not too long, I looked at the maximum caption length in the data, which was 40, and used that as my fixed sequence length since it is not too long. Then I padded the shorter captions with zeros so that all captions have a length of 40.

## 3. Removing the outliers

Next, I removed the words with a frequency of less than 10 times. This step is not mandatory, but removing the outliers saves a lot of memory, makes the model faster, and will help us to achieve better results.

## 4. Tokenizing

Next, we need to tokenize the words and convert them to integers before feeding them into the model. I broke down the sentences to words and then tokenized the words by assigning an integer to each unique word. After data cleaning and removing the outliers there were 1,698 unique words/tokens in the dataset.

## 5. Word Embedding 

The next step is doing word embedding. I used transfer learning again to do word embedding to leverage a model that was trained on a much larger text data, and extracted (semantically-meaningful) feature vectors from the captions. For this project, I used Global Vectors for Word Representation (GloVe) with 200-dimension. GloVe is an unsupervised learning algorithm for obtaining vector representation for words. In simple words, it allows us to take a corpus of text and transform each word into a position in a high-dimensional space. 

In other words, using the precomputed word embeddings available in GloVe, I created an embedding matrix for all the 1,698 unique words in my data. This embedding matrix will later be loaded into the final model before training. Note that if a word is in our data but is not in GloVe, the values of the vectors for that word will be zeros. To make this more concrete, here's an example of how a sample captions will look like when being fed into the model. Note that I'm showing the words here for the sake of clarity, but as mentioned in the Tokenizing step, they will be represented by integers:

![Text tensor input example](../assets/img/image_captions/text_tensor_example.jpg){: .postImage}

The numbers shown above are made-up examples and this was just an example for a single caption. Each caption will have a "triangle" like this with their relative numbers.  

That was a lot of preprocessing steps, so let me summarize: 1) I cleaned the text and removed noise, 2) made all the captions equal length by padding the shorter ones, and added a starting and ending word to each caption, 3) removed the outliers, 4) tokenized the words, and finally, 4) embedded the words using a pre-trained GloVe model.

# Model and Generating Captions 

Now that we have preprocessed both images and captions, we can feed them into the final neural network model and generate captions. The simplified figure below shows the general architecture of the model, how it receives the text and image data, and how it generates captions.

![Final Model](../assets/img/image_captions/Final_model1.jpg){: .postImage}

## 1. Setting up The Model

Since we have two inputs, we have to use the Functional API model, instead of the Sequential model that can only receive one input. The steps to set up the model is as follows:

**Input 1:** The first input of the model will be the feature vectors of 2,048-length that were extracted from the images.

**Input 2:** The second input of the model will be the text sequences, each having a length of 40 and an embedding dimension of 200. But instead of feeding sequences directly into the model, we first need to feed them into an LSTM layer and then to the final model. LSTM (Long Short-Term Memory) is just a special recurrent network layer that can process sequences and understand the order of the words.

**Merging the inputs:** Next, we need to merge both data inputs and convert them into a single tensor that can be fed into the functional API model, but before merging, we need to convert the output of the previous steps to the same length. So I converted both inputs to the same length of 256.

**Modeling:** Then the model takes in the tensor input of image and text data, and builds two more dense layers on top of it. Then we apply a Softmax function on top of the final layer (to convert the data in final layers into probabilities). After setting up this structure, I fitted the model using an "adam" optimizer and used "categorical_crossentropy" to measure the loss. 

**Output:** The output of this model is a single vector. Each element of the vector is a probability value and they sum up to one. The length of this vector is 1,698, which is the same as the number of unique words in the data. In other words, each value represents the probability of predicting its relative unique word. These probability values are conditioned on images, meaning that the probability value for a word differs from one image to another image. For example, we expect the word "dog" to have a higher probability for an image showing a dog, than for an image not showing a dog.

The below image was plotted by the model, it shows the architecture that I just explained, as well as the random dropouts that I used in different layers to avoid overfitting.  

![Final Model2](../assets/img/image_captions/Final_model3.jpg){: .postImage}

## 2. Generating Captions

We can generate captions using the output of the model in conjunction with a for-loop. Let me explain this via an example, which I also illustrated in the figure below. 

![Caption Prediction Example](../assets/img/image_captions/Caption_pred_example.jpg){: .postImage}

First, we need to initiate a caption (a string) that only includes "startseq" as its first word. Then we can predict the next words of the caption using a for-loop as follows:

**Iteration 1:** The model receives the **image + "startseq"** as input and predicts the next word, **"little"**, using the model output (image and text vector) discussed above.

**Iteration 2:** Then the model receives the **image + "startseq little"** as input and predicts the next word, **"girl"**.

.   
.

   
**Iteration 7:** Then the model receives the **image + "startseq little girl climbing into wooden playhouse"**  as input and predicts the next word, which is **"endseq"**. This word gives the model the signal to stop predicting. As mentioned before, the model will stop predicting when reaching the maximum length of 40 or reaching the word, "endseq", whichever happens first. 

# Converting to Speech

Finally, I converted the captions to speech by passing them through Google’s WaveNet (text-to-speech algorithm) available on the Google Cloud Platform, using Python's text to speech library. 

# Audio Caption Examples

Below are a few examples of the generated audio captions:

![Dog](../assets/img/image_captions/Dog.jpg){: .postImage}
**Generated caption:**    "a dog is chasing a ball in the grass"  [Click for audio caption](../assets/img/image_captions/dog.mp3)

&nbsp;
![People_image](../assets/img/image_captions/People_image.jpg){: .postImage}
**Generated caption:**    "a group of people are gathered around a building" [Click for audio caption](../assets/img/image_captions/people.mp3)

&nbsp;
![Bike](../assets/img/image_captions/bike.jpg){: .postImage}
**Generated caption:**    "a man in a red shirt and helmet is riding a bike on a dirt path"  [Click for audio caption](../assets/img/image_captions/bike.mp3)* 

*The model is confusing the reddish helmet and bike with a red shirt!

&nbsp;
![Boy_on_swing](../assets/img/image_captions/Boy_on_swing.jpg){: .postImage}
**Generated caption:**    "a boy is swinging on a swing set"  [Click for audio caption](../assets/img/image_captions/boy_swing.mp3)

&nbsp;
![Little_boy_in_red](../assets/img/image_captions/Little_boy_in_red.jpg){: .postImage}

**Generated Caption:**    "a child in a red coat is climbing a snowy hill"  [Click for audio caption](../assets/img/image_captions/boy_in_red.mp3)

# Conclusions 

I was able to build a decent model and generate some great audio captions using deep learning and only 6,000 images and captions. The model was strengthened by the power of transfer learning (InceptionV3 for images and GloVe for captions), where the models were previously trained on a large image and text dataset. To build a fully deployable technology to aid people with visual impairments, we need to train a model on an enormous amount of data, containing many image classes. That is not easy and requires a lot of resources, but we can start small by focusing on the objects inside a building (to help to navigate through a new office environment) or on the street (to help safely cross a street).

For future work, I would like to work with a larger dataset containing more types of images so that the model can generate captions for more image classes. I would also like to try a GPT-3 model instead of an LSTM model. However, I don't expect that it will make a huge improvement since captions mostly contain factual/straight forward text, as opposed to more complex texts such as poetry or legal language. 

You can see my code on my Github repo (to be added soon). Note that due to the stochastic nature of the model, if you use my code or even if I run my model again, the resulted captions will be slightly different. Lastly, it should be noted that to generate captions for new images, the images should be semantically related to the training images. For example, if we train the model only on cats and dogs, the model can only predict cats or dogs and not fruits or flowers!

I certainly enjoyed working on this project. I hope you enjoyed reading about it!







