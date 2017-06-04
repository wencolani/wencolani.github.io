Alexander Konovalov Ohio State University konovalov.2@osu.edu

WWW2017

#Motivation
knowledge base should not be viewed as a static snapshot,  but instead a rapidly evolving set of facts that must changes as the world changes.

this paper demonstrate the feasibility of accurately identifying entity-transition-events, from real-time news and social media text streams, that drive changes to a knowledge base.

they use Wikipedia's edit history as distant supervision to learn event extractors, and evaluate the extractors based on their ability to predict online updates.

the weakly supervised event extraction models are capable of automatically recommending revisions to knowledge graph in realtime.

#Challenge:

the reliance of weakly supervision learning methods  in redundancy  in news articlse : many sentences in the web are likely to mention context independent relationships. But there are a large number of redundant messages describing each significant in social networking websites such as Twitter ---could collect a lot of training data for weakly supervised event extraction.
#Method
to predict knowledge-base edits si to learn extractors for events that alter properties of knowledge-base entities, by leveraging the revision history of Wikipedia's semi-structured data as weak supervision.

this work selected a set of 6 infobox attributes whose changes correspond to certain well-defined events happening in the world: CurrentTeam, LeaderName, StateRepresentative, Spouse, Predecessor, DeathPlace.

## Datasets: 
Twitter(filter, NER, POS)
Annotated Gigaword v.5 dataset: newswire

##Step:
* prepare for the data 
* matching Tweets and News sentences to Wikipedia edits( mainly surface-form matching and readily-available alias dictionaries )
* training: 
  use T_aligned as positive examples, T_unaligned and a subset of T_random as negative examples.  Randomly sampled the subset of T_random to correspond to 90% of the testing data.

tweets that are written near the time of a knowledge graph revision are likely to mention an event that cause the change in state.

#Evaluation
how well the method can predict actual edits to Wikipedia,  in addition to a human evaluation of predicted edits using Amazon'd Mechanical Turk.

#Results

#Appendix
* Amazon Mechanical Turk: is a marketplace for work. They give businesses and developers access to on-demand, scalable workforce. Workers selected from thousands tasks and work whenever it's convenient. https://www.mturk.com/mturk/welcome
