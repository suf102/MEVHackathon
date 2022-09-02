### Note this is a work in progress. 
At the moment I am still working with dummy data, I have included the files to make the dummy data. 

If you would like to give this a go yourself, first check out the Ethdata folder for the files needed to make the data. Then go to data prep to split the data into sequences of length n. This should give you something to work with of the right structure. Third under the markovchain appraoch folder you will find the files needed to make the transition matrix the standard way, and the file use DL to aproximate it. Lastly in the Deeplearning models folder you will find the deep learning models I am working on. Enjoy ! 

# Mev-hackaton


## Stochastic models and Neural networks
Having been handed off the data classified into high, medium, low and none MEV categories. 
The time column of the data was based on the block number. 
We are using 12 million data points which is about 10% of the Ethereum mined. 
To split the data into a more useful format I created sequences of MEV levels. 
I created a list of all of the sequences of length 2 through 9 of mev levels that have occurred in those 12 million blocks. 
The block number was also dropped to avoid it indluencing the training of the models. 
From here I split the approach into three different directions. 
The first was to create a Markov chains transition matrx using the data. 
So this would express what the probability of transitioning to any particular state a is based on the current state. 
Second I used a deep learning model on the 2 length sequences to roughly approximate a Markov chain. 
Lastly I used a more general nn model, this would take the previous n MEV states and ask what the probability of the would be for the next MEV state. 
### Markov chains and transition matricies 
The Process of making this chain was relatively simple using the inbuilt pivot table function in Pandas. 
I used the pivot table to work out the occurrences of each state tansitioning into another then divided by the count of the original state to give the conditional probability. 
What this matrix allows you to read off is given the current level of MEV in the current block, what is the chance of being at any other level in the next time period. 
This is an extremely fast bit of code to deploy, the matrix only takes a few minutes to make, and you only need the previous blocks MEV state to work out probabilities for the next one, once you have the matrix. Updating the matrix after every block also seems to be a feasible strategy, however given the amount of data I would like to work with idealy adding one days worth of data will make little difference anyway so it may as well be rerun once a day rather than updated after every block.

### DL NN approximation of the Markov chain. 
Here I took a slightly different approach, instead of manually constructing the transition matrix I approximated it with a DL neural network. The DL nural network has 4 hidden layers each of which are 4 deep. 
This is to replicate the Markov chain matrix. This model took much longer to train than the Markov chain, and would be more difficult to update than the the Markov chain, due to the difficultites of having enough data to form a new batch and running the retraiign which takes approximately 4 hours. 
However, in deployment other than the conversion to tensors of the input data it is also pretty quick as one only needs to pass a 4d vector thought he model to get a prediction. 
### DL NN for prediction over more periods 
Lastly I use the 9 length sequences to see if I could produce a network that will create predictions.
The big advantage of this is that it should capture the data more accurately if it turns out that more than one period previous is required to predict MEV. Ofcoruse this took the longest to train of all of the models, but once in deployment would only need to be updated ever so ofter as a enough new blocks become available. You might be asking why use this strategy over model that looks over the whole data set. The rational is getting MEV data for a past block is computationally very expensive. Therefore using only the last n blocks one could more feasibly implement it as part of a larger algorithm.


## Observations 
One of the biggest difficulties with this process was access to good data, finding MEV and working out how much value is being extracted per block is not an easy challenge. Second getting hold of enough data was also tricky. 
Given the state of the world in the last few years it is also likely that it is not representative of the world as a whole. 
Also going into a bear market for crypto for the first time we don’t know how the profitability of MEV will be effected. This will have an impact on the amount available. 
Second we wanted to avoid going back too far in etheriums history as MEV is relatively new and going back too far might become unrepresentative of the current market, making prediction harder. 

Overall data selection and retrieval was very difficult, not only because access to the nodes was difficult but because detective MEV Is tricky on its on. Based on the data we have I was able to make these 4 models. 

# Further work

The most imediate further work that could be done would be to include more types of MEV, allowing for a more accurate picture of what the maxiumum extractable value acutally is. 

Second, more data is needed for the model, ideally 12million rows, which eqates to 10% roughly of the etherium blocks mined.

Third, A lassoCV Model applied to this data usign a few more parameters this might give a better results as it would include more exogenous factors. 

Fourth, the use of a LSTM NN, would probably be best for continuous deployment of an MEV predictor. However exceptional computing power would be needd to make it efficent enough to run in real time. 
