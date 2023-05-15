# MEV hackathon
This is a submission for the encode and wintermute hackathon, September 2022.

## Aims and Introduction
In this repo, we utilse markov chains and deep learning models in order to assess whether MEV can be predicted as a state change given a previous n number of blocks. 


## Methodology 
Firstly, we have extracted MEV based data from the online flashbots API at blocks.flashbots.net. From this, we chose to use the reward paid to the miner as our metric for quantifying the level of MEV on a block by block basis. Then, we looked at the range of miner reward payments across the dataset and split this up into quaters (using the .describe() function). This then gave us 4 distict classes for the level of MEV in each block, with 1 corresponding to a relatively low level of MEV and 4 corresponding to a relatively high level of MEV. Next, we have filled in the dataset for any blocks that the flashbots API has not identified as containing any MEV - these blocks were then given a class of '0' under the MEV_category column. The resulting dataframe is then used for markov chain transition modelling as well as in our deep learning models. For the purposes of deep learning, we have also created a second dataset. This second dataset is the same as the first, with 2 additional columns added (gas_used_ratio and eth_price_volatility). The gas_used ratio was extracted from an alchemy archive API, via calling the eth_feeHistory function and batching requests to recieve information pertaining to 1000 blocks at a time. The price of Ethereum per block was obtained via an infura archive node in which an instance of the ETH-USD chainlink price feed contract was obtained (using web3py library) and the latestRoundData() was called for each block. With the eth price per block, we then used a 50 day rolling standard deviation as a means of quantifying the ethereum price volatility per block. This then gave us our second dataset, which consists of the block number, MEV_category, gas_used_ratio and eth_price_volatility. The second dataset was then also used in our deep learning models to assess whether these extra features would be of any use in predicting MEV.

## Stochastic models and Neural networks
Having been handed off the data classified into high, medium, low and none MEV categories. 
The time column of the data was based on the block number. 
We are using ~10k data points, ideally we would have 12 million data points which is about 10% of the Ethereum mined, but currently MEV inspect runs too slowly for it to be practical to do so. 
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

In then end we had roughly 3 million blocks worth of data to work with, below you can find the transition matrix. What it shows is that there is some connection between the previous blocks MEV level and the next blocks, notably, if the MEV level is 4 in the next block is it over ~10% more likely to be another 4 block compared to coming form a 0 block. It is This is probably not a significant enough result for any kind of implementation because the increase in probability is very low. 

![Transition Matrix](https://github.com/suf102/MEVHackathon/blob/60fba81405c1b22c9094c68464f0a3a05adb2848/Readmeimages/Transitionmatrix.png)

### DL NN approximation of the Markov chain. 
Here I took a slightly different approach, instead of manually constructing the transition matrix I approximated it with a DL neural network. The DL neural network has 4 hidden layers each of which are 4 deep. 
This is to replicate the Markov chain matrix. This model took much longer to train than the Markov chain, and would be more difficult to update than the Markov chain, due to the difficulties of having enough data to form a new batch and running the retraining which takes approximately 4 hours. 
However, in deployment other than the conversion to tensors of the input data it is also pretty quick as one only needs to pass a 4d vector thought he model to get a prediction. This approach resulted in a evaluation accuracy of 53% therefore this is not likely a worthwhile avenue of pursuit. 

### DL NN for prediction over more periods 
Lastly I use the 9 length sequences to see if I could produce a network that will create predictions.
The big advantage of this is that it should capture the data more accurately if it turns out that more than one period previous is required to predict MEV. Of course this took the longest to train of all of the models, but once in deployment would only need to be updated ever so often as a enough new blocks become available. You might be asking why use this strategy over model that looks over the whole data set. The rational is getting MEV data for a past block is computationally very expensive. Therefore using only the last n blocks one could more feasibly implement it as part of a larger algorithm. Running this with sequences of 9 blocks I found that this model did perform better than the model that only uses two, but not by much. Perhaps with longer sequences it might be possible but running and training those models will take a lot more computing power. 

### DL with Gas Fees and Price volatility  
Here I created a NN that uses not only the MEV data but also the price volatility and the Eth Gas fees to predict the MEV level in the next block. The rational behind this model is that arbitrage and sandwich attacks should occur after movements in the price of ETH. A movement in the price is likely to cause a trade, and therefore there is scope for a sandwich attack. A movement in price is also likely to allow for small discrepancies in prices between exchanges and therefore it opens up opportunities for arbitrage.

This network was run with data with periods from 2 to 900 blocks, none of these revealed a model that would accurately predict the MEV level. However the data set we used was only ~5000 blocks long, with more data it may be possible to find some connection between MEV, gas fees and price volatility. 

## Observations 
One of the biggest difficulties with this process was access to good data, finding MEV and working out how much value is being extracted per block is not an easy challenge. Second getting hold of enough data was also tricky. 
Given the state of the world in the last few years it is also likely that it is not representative of the world as a whole. 
Also going into a bear market for crypto for the first time we don’t know how the profitability of MEV will be effected. This will have an impact on the amount available. 
Second we wanted to avoid going back too far in Ethereum’s history as MEV is relatively new and going back too far might become unrepresentative of the current market, making prediction harder. 

Overall data selection and retrieval was very difficult, not only because access to the nodes was difficult but because detective MEV Is tricky on its on. Based on the data we have I was able to make these 4 models. 

# Further work

The most immediate further work that could be done would be to include more types of MEV, allowing for a more accurate picture of what the maximum extractable value actually is. Ideally, this should be done via the MEV_inspect library, where support for different DEX's ought to be added as well as different strategy inspectors could be added to account for the more non-traditional types of MEV which take place such as NFT sniping/longer tail opportunities. What could also be interesting, is analysing MEV from a theoretical standpoint via a solver to observe the difference between historical theoretical MEV and actualised MEV. This would involve adding as many classifiers to MEV inspect as possible as well as a solver that iterates through a given set of smart contracts with historical state to calculate total theoretical MEV per block. This would require a substantial amount of data analysis and computational power.

Second, more data is needed for the model, ideally 12million rows, which equates to 10% roughly of the Ethereum blocks mined I think would be ideal but we are only working with 10k blocks currently

Third, A lassoCV Model applied to this data using a few more parameters this might give a better results as it would include more exogenous factors. 

Fourth, the use of a LSTM NN, would probably be best for continuous deployment of an MEV predictor. However exceptional computing power might be needed to make it efficient enough to run in real time. 


