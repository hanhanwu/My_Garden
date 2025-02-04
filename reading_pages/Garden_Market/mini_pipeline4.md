### MLJar

Similar to other AutoML pipelines, MLJar also includes data preprocessing, hypterparameter tuning and model selection, as well as model evaluation. What makes MLJar special is its detailed visualization output and the way it selects algorithms.

<p align="center">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_process.png" width="823" height="424" />
</p>

#### MLJar Visualization Output

Let's look at some visualized examples first!

<b>Leaderboard</b> summarizes the training time and performance of each model.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_leaderboard.png" width="700" height="400" />
</p>

* MLJar "Ensemble" uses greedy approach to create weighted ensemble from already trained ML models.
* MLJar "Stacked" uses stacking. In stacking, there are multiple stages of base models' forecasting. In each stage, there is k-fold cross validation, the predicted values from each fold will be appended together as a new feature for the next stage. When there are `m` base models in a stage, there will be `m` new features generated for the next stage.
  * In this example, the base model is default LightGBM.
* MLJar "Ensemble_Stacked" will ensemble all the previous stacked and unstacked models.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/notes/stack_ensemble.png" width="766" height="79" />
</p>

MLJar generates <b>SHAP summary plot</b>, which plots the importance of all the features together. The way [SHAP value][1] works is,

* Higher positive SHAP value indicates a feature tends to drag the forecast value higher (positive impact)
* Lower negative SHAP value indicates a feature tends to drag the forecast value lower (negative impact)
* No matter it's positive or negative SHAP, bigger absolute SHAP value indicates the feature has bigger impact on the forecast value. Therefore, as we can see, summary plot uses the average absolute SHAP value to measure the feature importance.

The left plot is the overall SHAP feature importance, while the right plot is SHAP feature importance in each fold of k-fold cross validation.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_shap_summary.png" width="1415" height="450" />
</p>

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/notes/shap_importance.png" width="766" height="79" />
</p>

Besides summary plot, MLJar also generates <b>SHAP dependent plot</b> for each feature. The example below came from "4_Default_LightGBM_Stacked" algorithm, where predicted results from formerly trained models became the new features in the next stage, this is why you are seeing features like "Ensemble_prediction", "Decision_Tree_prediction", etc.

The way SHAP dependent plot works is:

* It plots every data record used in the model
* Red color means higher SHAP value of the data record
* Blue color means lower SHAP value of the data record
* When a feature's SHAP output has a positive correlation with the model's prediction results, this feature tend to be more important, such as feature "Ensemble_prediction" and feature "4_Default_LightGBM_prediction" shown below.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_shap_dependent.png" width="1000" height="500" />
</p>


#### Regression with MLJar

Same as what happened in AutoKeras, Lady H. only chose the 3 numerical features as the training data input.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/ak_reg_data.png" width="991" height="461" />
</p>

The setup of MLJar is straightforward,

* "Compete" mode is the most complete mode for model selection, aiming at achieving ultimate model performance. There are [other modes][2] can be chosen for faster prototyping.
* Same as many other AutoML tools, you are allowed to specify the evaluation metrics, validation strategy, random state and explain level. The results path saves all the output.
* When choosing `explain_level=2`, you will get the brief information of each step from the whole model selection process. [Check more description of MLJAR steps here][3].

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_reg.png" width="1101" height="823" />
</p>

🌻 [Look into MLJar regression experiment details >>][5]

When you want to load the selected model to forecast on a set of testing data, it takes MLJar a longer time (5+ minutes) to do so, maybe because it doesn't save any model artifact in the output.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_reg_load_model.png" width="454" height="118" />
</p>

0.938 testing R2 score with only 3 input features is an exciting performance. As we saw in [Table 4.1][4], it's much better than AutoKeras which got negative R2 score, it's also better than the performance from TPOT which used all the features as input and took 4 hours execution time. The best HPO performance was 0.982, but it was using all the 18 features while MLJar only had 3 features.

One of the success factors of such promising model selection ability is, Ensemble Stacked model, which combines the power of stacking and weighted ensembling together, and this model often appears to be the best model in MLJar model selection results.

#### Classification with MLJar

The AutoML setup for classification is similar to regression, and the performance is better than the best performance of HPO in [Table 4.1][4].

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_cla.png" width="1102" height="883" />
</p>

🌻 [Look into MLJar classification experiment details >>][5]

1 hour default time budget for such a small dataset might be overkilling. We can set the time budget through `total_time_limit`. Here we set 6 minutes to see whether we can achieve similar results as the the best HPO method achieved in 5 minutes, but 0.66 balanced accuracy is far lower than 0.839 achieved in HPO. Meanwhile, with less time budget, some steps will be skipped as what's shown below, and might cause error. The good news is, even if there are errors during the execution process, MLJar can still manage to finish the whole pipeline running.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/mini_pipeline/mljar_cla_6m.png" width="970" height="907" />
</p>

🌻 [Look into MLJar 6 min classification experiment details >>][5]


#
<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/follow_us.png" width="120" height="50" />
</p>

[Keep going >>][6]

<p align="right">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/going_back.png" width="60" height="44" />
</p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [<< Looking back][7]
 



[1]:https://github.com/slundberg/shap
[2]:https://supervised.mljar.com/features/modes/
[3]:https://supervised.mljar.com/features/automl/
[4]:https://github.com/lady-h-world/My_Garden/blob/main/reading_pages/Garden_Market/mini_pipeline1.md#mini-pipelines
[5]:https://github.com/lady-h-world/My_Garden/blob/main/code/garden_market/mini_pipelines/mljar.ipynb
[6]:https://github.com/lady-h-world/My_Garden/blob/main/reading_pages/Garden_Market/mini_pipeline5.md
[7]:https://github.com/lady-h-world/My_Garden/blob/main/reading_pages/Garden_Market/mini_pipeline3.md

