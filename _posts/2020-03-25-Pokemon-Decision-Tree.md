---
title: What groups of Pokémon made into the Galar Pokédex?
date: '2020-03-25'
permalink: /posts/2020/03/pokemon-decision-tree/
tags:
  - just for fun
---

Drawing decision trees on Pokémons depending on whether they made it to the Galar Pokédex

# Disclaimer

Pokémon and Pokémon character names are trademarks of Nintendo. I will gladly remove this material from my website if there is any problem in regards to the copyright, etc.

# Introduction
Less than a year ago, Nintendo announced in [ES 2019](https://www.youtube.com/watch?v=hhVCdxtofWU&t=6155s) that Pokémons that are not included in the Galar Pokédex will not be able to be transferred into the Pokémon Swords & Shields (Note that this announcement is no longer quite valid since around 200 Pokémons are planned to be added in the new DLC). As a big Pokémon fan (although I did not have the money to buy Nintendo Switch nor have the time to play the new generation as a graduate student, I bought at least one Pokémon games in each generation except the second generation and the fifth generation until now), I always thought that the selection of the Pokémons that can be transferred into the new generation would have been one of the most difficult conundrums the developers had ever faced. After the game release and looking at the list of Pokémons that made the list, I spend quite a lot of time trying to figure out if the Pokémons that were included in the Galar Pokédex had anything in common on my plane to Chicago. I was not able to come up with a good conclusion.

Recently, I heard from one of my friends that there is a cool API called [PokéAPI](https://pokeapi.co/) where I can retrieve a lot of information about Pokémons (I believe she was writing a short blogpost about cool APIs to do data science projects with). Upon hearing that, I thought it will be cool to try to use that information to see if there was a pattern in the selection process I discussed above. After thinking about various tools for classification problems, I decided that I will use decision trees to look into this. This was because decision trees are quite interpretable, and handles categorical variable well (virtually all my variables here are dummy variables). I was hoping that I could find a tree with a little number of splits but explains the decision criterion well (which is usually a very implausible thing to hope for).

# Methods

## Data

As previously mentioned, all the information about Pokémons were retrieved by calling [PokéAPI](https://pokeapi.co/). Then, I merged the data with the [List of Pokémon by Galar Pokédex number](https://bulbapedia.bulbagarden.net/wiki/List_of_Pokémon_by_Galar_Pokédex_number) article in [Bulbapedia](https://bulbapedia.bulbagarden.net/wiki/Main_Page) to get the list of Pokémons that made it to the Galar Pokédex. Pokémons from generation 8 were not included in the data because they naturally were included in the Galar Pokédex.

## Trees

The decision trees were fitted using [_DecisionTreeClassifier_](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html) class in Scikit-learn library in Python. The trees were grown freely using the Gini index as the criterion, and was pruned through minimal cost-complexity pruning with $\alpha = 0.01$. The trees were plotted using [_plot_tree_](https://scikit-learn.org/stable/modules/generated/sklearn.tree.plot_tree.html) function from the same library.

Note that I did not use any train set/test set split or resampling methods. I wanted to use the model as a more explanatory way rather than a predictive, generalizable way (unless the developers decide to pull something similar for the 9th generation).

# Results

## How to read each tree

For all trees, orange color means that the majority of Pokémons in the node was not included in the Galar Pokédex. On the contrary, blue color means that the majority of Pokémons in the node was included in the Galar Pokédex. Thicker the color is for the node, purer that node is.

The first line of each split node indicates what variable the children nodes were split by. Note that all variables in the trees were dummy variables, so they are all just indicator variables with only two values - 0 and 1. On each split, the right child node consists of Pokémons with value 1 on the variable the node was split on. The left child node consists of Pokémons with value 0 on the variable the node was split on. For example, let's say that the tree was split by 'water-gun' variable in the move set tree. Swampert, who learns the move water gun, will be in the right child node. On the contrary, Charizard, who does not learn the move water gun, will be in the left child node. 

`samples` of each node means the proportion of Pokémons that are in the node.  `values` of each node means the proportion of Pokémons that did not make into Galar Pokédex (first element of the list) and the proportion of Pokémons that made into Galar Pokédex (right element of the list). `class` shows the majority of the classification ('Not in Galar' vs 'In Galar') in that node.

## Egg groups

According to Bulbapedia, Egg Groups are "categories which determine which Pokémon are able to interbreed" (read more [here](https://bulbapedia.bulbagarden.net/wiki/Egg_Group)). I thought that since they are a broad category that classifies similar Pokémons together, they might be a good starting point to fit the decision trees. Here is the result:

![tree 1](/img/pokemon_tree/tree1.png)

We can see that the only split that survived the pruning was the `no-egg` egg groups (more formally No Eggs Discovered egg group). This is understandable because almost all legendary Pokémon, mythical Pokémon, and Ultra Beasts,  which are major components of this egg group, did not make it to the Galar Pokédex.

Another thing to note is that this tree made a split although the classifications of both nodes are the same ('Not in Galar'). This is because the minimal cost-complexity pruning algorithm uses impurity measures instead of a misclassification rate as $R(T)$. In other words, Pokémons in no-egg egg group were more purely 'Not in Galar' than Pokémons that were not in the no-egg egg group. This difference in purity was greater than the cost of having more nodes in the pruning algorithm.

Because the tree classifies all Pokémons as 'Not in Galar', the accuracy of the model is the same with the null models: about 0.60.

## Generations

Generations refer to the generation in which the Pokémon was introduced. Before fitting, I thought that this tree would not make any split since I thought the developers would have allocated similar numbers of Pokémons from each generation. Here is the decision tree using generations:

![tree 2](/img/pokemon_tree/tree2.png)

Interestingly, Pokémons from the fifth generation were classified as 'In Galar' while Pokémons from all other generations were classified as 'Not in Galar'. The accuracy of this tree was about 0.62 and the AUC was about 0.59. These scores show that this tree is only slightly better than the null model.

## Types

Types refer to the type category Pokémons belongs to. Before fitting, I suspected that this tree would not make any split with similar reasoning behind guessing the generations tree will not make any split. I especially thought that the developers would have tried to keep the proportion of each type similar to the proportion the game had before the new release, because types are an essential component of Pokémon games, and changing the proportion will result in a big difference in the gameplay. Another guess I had was that if there is a split, maybe `normal` types were less likely to be included in the Galar Pokédex, because, well, they are more normal than other Pokémons.

Here is the fitted decision tree:

![tree 3](/img/pokemon_tree/tree3.png)

Contrary to my expectation, the only split was made in the `ghost` type, where ghost-type Pokémons were classified as 'In Galar'. The accuracy of this tree was about 0.63 and the AUC was about 0.66. These scores show that this tree is only slightly better than the null model, although note that AUC was quite better compared to the generation tree model (0.07).

## Move Sets

Move Set refers to all the moves a Pokémon can learn in Generation 7 (through leveling up, using TM/HM, egg moves, etc). More than 300 dummy variables were used to fit this tree. Note that some moves are obviously related to each other (for example, it makes intuitive sense that a Pokémon that learns ember will have a higher chance of learning Flamethrower) but I did not take them into account when making the dummy variables. Since this tree can use so many variables, I thought that this tree will have the most power compared to another trees. Here is the fitted tree:

![tree 4](/img/pokemon_tree/tree4.png)

We could see that this tree is much more complicated than the other two trees we had seen so far. The first split was made with the move `attract`. I think this is related to the aforementioned fact that almost all legendary Pokémon, mythical Pokémon, and Ultra Beasts did not make it to the Galar Pokédex. Most of these Pokémons are genderless, so they cannot learn the move `attract`.

Also, the split over `dark-pulse` might be related to ghost-type Pokémon having a higher chance of being selected into the Galar Pokédex, since it is a common move for ghost-type Pokémons (although it is a dark-type move).

The accuracy of this tree was about 0.72 and the AUC was about 0.71, showing quite a lot of improvements compared to other trees.

## All variables

Finally, I fitted a decision tree using all variables we discussed so far. Here is the result:

![tree 5](/img/pokemon_tree/tree5.png)

Interestingly, the tree was less complicated and less deep than the move sets tree. It also performed worse: the accuracy was about 0.67 and the AUC was about 0.67. Following is a confusion matrix comparing the two models:

![confusion matrix](/img/pokemon_tree/confusion.png)

The worse performance of the bigger model that has all the variables of the smaller model could be related to the fact that the decision tree is built in a greedy way. The initial split on `no-egg` might have been less beneficial than split on `attract` in a broader view, but it provided better improvement in the short term.

# Conclusion

All trees that were considered did not give a good, concise explanation of the decision-making process behind what Pokémons were included in the Galar Pokédex. We could get a model that fits the data perfectly if we do not prune the tree but it will look something like this:

![tree 6](/img/pokemon_tree/tree6.png)

which makes little sense.

The decision-making process would probably consider a lot of factors outside the game component such as the popularity of the Pokémons (for example, I guess Pikachu would be included in any scenario) or the introduction of similar Pokémons in the new game. If we could identify some of those factors and identify them, we could probably come up with a better model.

# Codes

All analysis was done in a Jupyter notebook using python. The Jupyter notebook can be seen [here](https://github.com/nwrim/small_projects/blob/master/Pokemon_included_Galar_decision_tree.ipynb). Data analysis was done using [pandas](https://pandas.pydata.org/) and [Scikit-learn](https://scikit-learn.org/stable/). Getting data from Bulbapedia was done using [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/). All data was from the [PokéAPI](https://pokeapi.co/) and [Bulbapedia](https://bulbapedia.bulbagarden.net/wiki/Main_Page).
