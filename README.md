### Note this is a work in progress. 
At the moment I am still working with dummy data, I have included the files to make the dummy data. Edit I have also added new files that will run the same code but with gas fees and price volatility considered, If the file ends in wgv(with gas fees and volatility) it means it is the version of the files designed to consider those additional factors. I have also included the files needed to make the dummy data if one is so inclined. 

If you would like to give this a go yourself, first check out the Eth data folder for the files needed to make the data. Then go to data prep to split the data into sequences of length n. This should give you something to work with of the right structure. Third under the Markov chain approach folder you will find the files needed to make the transition matrix the standard way, and the file use DL to approximate it. Lastly in the Deep learning models folder you will find the deep learning models I am working on. Enjoy ! 

# Mev-hackaton

## Stochastic models and Neural networks
Having been handed off the data classified into high, medium, low and none MEV categories. 
The time column of the data was based on the block number. 
We are using 12 million data points which is about 10% of the Ethereum mined. 
To split the data into a more useful format I created sequences of MEV levels. 
I created a list of all of the sequences of length 2 through 9 of MEV levels that have occurred in those 10k blocks. 
The block number was also dropped to avoid it influencing the training of the models. 
From here I split the approach into three different directions. 
The first was to create a Markov chains transition matrix using the data. 
So, this would express what the probability of transitioning to any particular state a is based on the current state. 
Second, I used a deep learning model on the 2 length sequences to roughly approximate a Markov chain. 
Third, I used a more general nn model, this would take the previous n MEV states and ask what the probability of the would be for the next MEV state. 
Fourth, I added some additional data, I have a hunch that the price volatility and gas fees are the most important factors to predicting MEV, this is because price movements result in people moving their Eth. Large movements are what allow sandwich attacks to take place, and arbitrage happens when there is a difference in prices which is more likely to occur when the price of eth is moving. 
### Markov chains and transition matrices 
The Process of making this chain was relatively simple using the inbuilt pivot table function in Pandas. 
I used the pivot table to work out the occurrences of each state transitioning into another then divided by the count of the original state to give the conditional probability. 
What this matrix allows you to read off is given the current level of MEV in the current block, what is the chance of being at any other level in the next time period. 
This is an extremely fast bit of code to deploy, the matrix only takes a few minutes to make, and you only need the previous blocks MEV state to work out probabilities for the next one, once you have the matrix. Updating the matrix after every block also seems to be a feasible strategy, however given the amount of data I would like to work with ideally adding one days worth of data will make little difference anyway so it may as well be rerun once a day rather than updated after every block.

### DL NN approximation of the Markov chain. 
Here I took a slightly different approach, instead of manually constructing the transition matrix I approximated it with a DL neural network. The DL neural network has 4 hidden layers each of which are 4 deep. 
This is to replicate the Markov chain matrix. This model took much longer to train than the Markov chain, and would be more difficult to update than the Markov chain, due to the difficulties of having enough data to form a new batch and running the retraining which takes approximately 4 hours. 
However, in deployment other than the conversion to tensors of the input data it is also pretty quick as one only needs to pass a 4d vector thought he model to get a prediction. 
### DL NN for prediction over more periods 
Lastly I use the 9 length sequences to see if I could produce a network that will create predictions.
The big advantage of this is that it should capture the data more accurately if it turns out that more than one period previous is required to predict MEV. Of course this took the longest to train of all of the models, but once in deployment would only need to be updated ever so often as a enough new blocks become available. You might be asking why use this strategy over model that looks over the whole data set. The rational is getting MEV data for a past block is computationally very expensive. Therefore using only the last n blocks one could more feasibly implement it as part of a larger algorithm.

### DL with Gas Fees and Price volatility  
Here I created a NN that uses not only the MEV data but also the price volatility and the Eth Gas fees to predict the MEV level in the next block. The rational behind this model is that arbitrage and sandwich attacks should occur after movements in the price of ETH. A movement in the price is likely to cause a trade, and therefore there is scope for a sandwich attack. A movement in price is also likely to allow for small discrepancies in prices between exchanges and therefore it opens up opportunities for arbitrage.  

## Observations 
One of the biggest difficulties with this process was access to good data, finding MEV and working out how much value is being extracted per block is not an easy challenge. Second getting hold of enough data was also tricky. 
Given the state of the world in the last few years it is also likely that it is not representative of the world as a whole. 
Also going into a bear market for crypto for the first time we don’t know how the profitability of MEV will be effected. This will have an impact on the amount available. 
Second we wanted to avoid going back too far in Ethereum’s history as MEV is relatively new and going back too far might become unrepresentative of the current market, making prediction harder. 

Overall data selection and retrieval was very difficult, not only because access to the nodes was difficult but because detective MEV Is tricky on its on. Based on the data we have I was able to make these 4 models. 

# Further work

The most immediate further work that could be done would be to include more types of MEV, allowing for a more accurate picture of what the maximum extractable value actually is. 

Second, more data is needed for the model, ideally 12million rows, which equates to 10% roughly of the Ethereum blocks mined I think would be ideal but we are only working with 10k blocks currently

Third, A lassoCV Model applied to this data using a few more parameters this might give a better results as it would include more exogenous factors. 

Fourth, the use of a LSTM NN, would probably be best for continuous deployment of an MEV predictor. However exceptional computing power might be needed to make it efficient enough to run in real time. 


