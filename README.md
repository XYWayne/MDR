# MDR

## Outline
1. Models 
2. Parameters for experiments
3. Data overview and preprocess
4. Results and experiment log
---

### Models
Note: models described here are folked from Xiangnan He's [repo](https://github.com/hexiangnan/neural_collaborative_filtering).
In this experiments we mainly use NeuMF for our cases, however GMF and MLP may also be run to serve as benchmarks.
---

### Parameters for experiments
As from the github page of the above hyperlink, to run an example training:
```
python3 NeuMF.py --dataset ml-1m --epochs 20 --batch_size 256 --num_factors 8 --layers [64,32,16,8] --reg_mf 0 --reg_layers [0,0,0,0] --num_neg 4 --lr 0.001 --learner adam --verbose 1 --out 1
```
---

### Data  overview and preprocess
#### Original Datasets
I currently have 3 datasets for this experiment: [Pinterest, MovieLens, Yelp](https://github.com/hexiangnan/adversarial_personalized_ranking), and [Amazon Prime Pantry](https://jmcauley.ucsd.edu/data/amazon/). 

The original dataset contains:

* **train.rating**:  
Each line is a training example: `userID\t itemID\t rating\t timestamp`

* **test.rating**:  
Each line is a testing example: `userID\t itemID\t rating\t timestamp`

* **test.negative**:  
Each line is in the format: `(userID,itemID)\t negativeItemID1\t negativeItemID2\t ...`

In order to study the effect of training on a data set that consists of a "hot" part and a "longtail" part, we can first get a visualization of our datasets. Below are some statistics of the above datasets:

|                          | MovieLens-1M  | Pinterests-20     |   Yelp    |   Prime Pantry   |
| :---:                    |    :----:     |       :---:       |  :----:   |     :---:        |
| Training Samples         | 994169        |1445622            |705994     |461384            |
| Testing Samples          | 6040          |55187              |25677      |9419              |
| Negative Samples         | 6040          |55187              |25677      |9419              |

Now, if we want to conduct hot-longtail analysis, we should first see the distribution of user frequency:

|               | MovieLens-1M  | Pinterests-20     |   Yelp   |   Prime Pantry   |
| :---:         |    :----:     |       :---:       |  :----:  |     :---:        |
| Count         | 6040          |55187              |25677     |10814             |
| Mean          | 165           |26                 |27        |43                |
| Std           | 192           |7                  |42        |151               |
| Min           | 19            |14                 |9         |1                 |
| 25 percentile | 43            |21                 |11        |5                 |
| 50 percentile | 95            |24                 |15        |15                |
| 75 percentile | 207           |29                 |27        |39                |
| Max           | 2313          |136                |1175      |6537              |

Below are the visualizations of the datasets' userID distributions:  
* MovieLens:
<img src = "https://user-images.githubusercontent.com/59850013/183307329-585d32ed-55fe-434b-97ce-a45fbff70e70.png" width=75% height=75%>

* Pinterests:
<img src = "https://user-images.githubusercontent.com/59850013/183307345-fe7ae6ec-cb5a-4a02-a488-d8e1605a9513.png" width=75% height=75%>

* Yelp:
<img src = "https://user-images.githubusercontent.com/59850013/183307355-5b54e272-d285-4495-bd41-5c74a478383f.png" width=75% height=75%>

* Prime Pantry:
<img src = "https://user-images.githubusercontent.com/59850013/183307363-5ae1c9ed-ef6b-4803-985b-5555058a8672.png" width=75% height=75%>

**Note:**  
From the plots we can see that among these datasets, **MovieLens, Yelp, and Prime Pantry** have obvious hot-longtail distributions. Thus, in order to study the hot-longtail data structure, we only conduct further experiments on these 3 datasets.

#### Splited Datasets
In order to study the effect of training hot-longtail data structures, we hereby split the original datasets into the hot part and the long tail part based on the user frequency. **Any userID with frequency higher than the 75 percentile is considered to be in the hot part, and those who are below the threshold are considered to be in the ongtail part.** Note that there is still a debate over the choice of the threshold.

The splited dataset contains:

* **train.hot.rating**:  
Each line is a training example from the hot part: `userID\t itemID\t rating\t timestamp`

* **test.hot.rating**:  
Each line is a testing example from the hot part: `userID\t itemID\t rating\t timestamp`

* **test.hot.negative**:  
Each line is an instance of negative samples with userID in the hot part: `(userID,itemID)\t negativeItemID1\t negativeItemID2\t ...`

* **train.lt.rating**:  
Each line is a training example from the longtail part: `userID\t itemID\t rating\t timestamp`

* **test.lt.rating**:  
Each line is a testing example from the longtail part: `userID\t itemID\t rating\t timestamp`

* **test.lt.negative**:  
Each line is an instance of negative samples with userID in the longtail part: `(userID,itemID)\t negativeItemID1\t negativeItemID2\t ...`

Here is the data distribution after spliting:

|                               | MovieLens-1M  |   Yelp    |   Prime Pantry   |
| :---:                         |    :----:     |  :----:   |     :---:        |
| Training Samples(hot)         | 630603        |433378     |362401            |
| Testing Samples(hot)          | 1511          |6454       |2500              |
| Negative Samples(hot)         | 1511          |6454       |2500              |
| Training Samples(lt)          | 354566        |272616     |98983             |
| Testing Samples(lt)           | 4529          |19223      |6919              |
| Negative Samples(lt)          | 4529          |19223      |6919              |
