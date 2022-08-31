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
This is an extremely fast bit of code to deploy, the matrix only takes a few minutes to make, and you only need the previous blocks MEV state to work out probabilities for the next one, once you have the matrix. 
…
### DL NN approximation of the Markov chain. 
Here I took a slightly different approach, instead of manually constructing the transition matrix I approximated it with a DL neural network. The DL nural network has 4 hidden layers each of which are 4 deep. 
This is to replicate the Markov chain matrix. This model took much longer to train than the Markov chain, and would be more difficult to update than the the Markov chain, due to the difficultites of having enough data to form a new batch and running the retraiign which takes approximately 4 hours. 
However, in deployment other than the conversion to tensors of the input data it is also pretty quick as one only needs to pass a 4d vector thought he model to get a prediction. 
### DL NN for prediction over more periods 
Lastly I use the 9 length sequences to see if I could produce a network that will create predictions.
The big advantage of this is that it should capture the data more accurately if it turns out that more than one period previous is required to predict MEV. Ofcoruse this took the longest to train of all of the models, but once in deployment would only need to be updated ever so ofter as a enough new blocks become available. 
### Lasso CV model 
Lastly I wanted to try one other kind of model, this is a lasso model with cross validation. Typically used in forcasting of momentum strategies. I wanted to see if this model would be able to predict MEV. The inputs I used were the previous MEV states, the price of Ethereum, Gas fees and base fees. 
The Previous MEV was chosen as it is likely to act as signal of future MEV opportunities, and as shown above has some connection to future MEV rates. Second the price of Ethereum crucially because this and the gas fees and base fees determine the viability of seeking MEV. 



## Observations 
One of the biggest difficulties with this process was access to good data, finding MEV and working out how much value is being extracted per block is not an easy challenge. Second getting hold of enough data was also tricky. 
Given the state of the world in the last few years it is also likely that it is not representative of the world as a whole. 
Also going into a bear market for crypto for the first time we don’t know how the profitability of MEV will be effected. This will have an impact on the amount available. 
Second we wanted to avoid going back too far in etheriums history as MEV is relatively new and going back too far might become unrepresentative of the current market, making prediction harder. 

Overall data selection and retrieval was very difficult, not only because access to the nodes was difficult but because detective MEV Is tricky on its on. Based on the data we have I was able to make these 4 models. 
