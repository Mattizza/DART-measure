<h1 style="text-align:center;"> DART: Definitely-Agnostic Ranked Trade-off</h1>

In a ML pipeline, the process of **model selection** and **model validation** is crucial, since it can be what mainly determines the **success** of a project. Still, when making comparisons of one model against the others, a large portion of practitioners tends to take into account just one statistic at a time; in most of the cases, the accuracy. For example, when tuning the hyperparameters on the validation set using Cross Validation, people tend to select the model that has the highest accuracy. What about the variance of such a model? In most of the cases it tends to be forgotten, even though it is what we really care about since it is a proxy for the **stability** of our model.<br>
<br>

> Considering the accuracy alone can be very risky since it may cause an **overestimation** of the performance of our model.

<br>In order to avoid this, we would like to have a ***single measure*** to make such comparisons during the model selection process, which takes into account both the accuracy **AND** the standard deviation of the model.<br>
Due to the, at the best of my knowledge, absence of such a measure, I have decided to derive a new one considering all the requirements stated above. It is named ***DART***, which stands for ***D**efinitely-**A**gnostic **R**anked **T**rade-off*:
<br>
<br>
> - **D**efinitely-**A**gnostic: we refuse to suppose there is a single best model based on the accuracy alone;<br>
> 
> - **R**anked: at the end, what this measure does is ranking the models to return the best-fitting ones for our problem;
> - **T**rade-off: we are no more considering only the accuracy as our main metric, since we now have a trade-off with the variance of the model.

<br>
From now on we will assume to be using K-Fold Cross Validation, since this measure has been designed principally to be used in this setting and also because it is a standard when doing hyperparameters tuning. Since we need a proxy for the variance of the model, and an accurate estimate of the accuracy, several trainings are required for the same model.

---

<h2 style="text-align:left;"> Accuracy is NOT the only statistic that matters </h2>

Let's start by providing the formula to compute the DART-measure for a given model:
<br>
<br>

$$ DART_i = \frac{1 + 1.4427\times \ln(\bar{A_i})}{\mathrm{e}^{\mathrm{p}D_i}} \quad\quad i = 1, ..., \lvert G \rvert $$
<br>

where $\lvert G \rvert$ is the number of possible configurations given by our hyperparameters grid, i.e. the number of models we will test.
<br>
<br>


> PARAMETERS
> - $\bar{A_i}$: mean accuracy of model *i*, computed with respect to the *k* accuracies given by K-Fold Cross Validation;<br>
> 
> $$ \bar{A_i} = \frac{1}{\mathrm{k}}\sum_{j}A_{ij} \quad\quad j = 1,..., \mathrm{k} $$
>
> - $D_i$: standard deviation of the model, computed with respect to the *k* accuracies;<br>
>
> $$ D_i = \sqrt{\frac{\sum_{j}(A_{ij}-\bar{A_i})^2}{(\mathrm{k}-1)}} \quad\quad j = 1,..., \mathrm{k} $$
>
> - p: desired precision, fixed and representative of how much importance we give to the stability of the model.

<br>

How is the value of the DART-measure related to its inputs? Ideally, it should assume a **high value** when the **accuracy is high** and the **variance is small**; this would mean that we have found a very good model! In an opposite way, when it assumes a **low value**, it means that the model is very bad, since it has **low accuracy** and **high variance** (or, more precisely, a variance **too high for our requirements**). 



Of course, this is just a way of combining the two statistics, and there are plenty of other ways to do that. Nevertheless, the proposed measure gives enough flexibility to choose which characteristic the desired model should have. A important parameter that allows to try several different configurations is the *precision*, which is a positive real number that represents the importance we give to the stability of the model. 

---

<h2 style="text-align:left;"> Values, parameters and how to interpret </h2>

Let's see now the values the DART-measure can assume by first providing a table to get a quick understanding of how this measure behaves. We use '-' when we don't want to take care of a certain variable on a specific row. Two distinct tables would have been hideous.
<br>
<br>

<div align = "center">

|Numerator        |Denominator        |Accuracy       |Standard Deviation |Performance     |
|:---------------:|:-----------------:|:-------------:|:-----------------:|:--------------:|
|1                |-                  |1              |-                  |Very Accurate   |
|0                |-                  |0.5            |-                  |Random Guessing |
|< 0              |-                  |< 0.5          |-                  |Worst Gambler   |
|-                |1                  |-              |0                  |Very stable     |
|-                |> 1                |-              |> 0                |Variance Penalty|
|-                |>>> 1              |-              |>> 0               |Shaking         |
  
</div>

<br>
<br>

**NUMERATOR**
- It assumes its biggest value when the accuracy is 1, and its lowest value when the accuracy is 0.5, i.e. random guessing;
- It assumes negative values whenever the performance is worse than random guessing (REALLY BAD MODEL).

**DENOMINATOR**
- It increases exponentially with the standard deviation. It assumes its lowest value when there is no variance;
- It has not an upper bound, so it can increase without limit depending on the variance.
<br>

Summarizing, what this measure does is trying to strike a balance between the accuracy and the stability of a model, making the variance weigh more depending on the precision required by the problem. It is also very nice to observe that its values are normalized between 0 and 1. <br>
Finally, we are also allowed to change the value of the precision, gaining a lot of flexibility. Notice that by setting it to zero, we go back to the usual case in which we ponder just the accuracy; this means we can generalize pretty easily, not bad!
<br>
<br>

A general, qualitative, behaviour that takes into account all the parameters is shown in the following table:
<br>
<br>
<div align = "center">

| Accuracy        | Standard Deviation | Desired stability | DART*        |
|:---------------:|:------------------:|:-----------------:|:------------:|
| High            | High               | High              | ☄️✨        | 
| High            | High               | Low               | ☄️☄️☄️✨   |
| High            | Low                | High              | ☄️☄️☄️☄️   |
| High            | Low                | Low               | ☄️☄️☄️☄️☄️ |
| Low             | High               | High              | ☄️           |
| Low             | High               | Low               | ☄️✨        |
| Low             | Low                | High              | ☄️☄️☄️      |
| Low             | Low                | Low               | ☄️☄️✨      |

</div>
<br>
<br>

> *We use ☄️ to score the expected goodness of the measure and ✨ to refer to possible variations depending on the specific situation and the exact values.
Notice the Standard Deviation is what contributes the most to its value, depending on the desired stability. The lower, the better.

<br>
This table is really general and it is shown here just to give a hint about how the DART-measure behaves. However, in real situations, we may have a lot of exceptions and absolute values of different order of magnitudes. And so, why should someone use such an "unstable" measure? Because what we care about is the relative ordering! Fixed a precision, we are able to rank the models in different ways, giving more or less (or no) importance to the stability. If this seems obscure to you, and it probably does, just be a bit more patience and check the notebook; there is a thorough explanation, using plots as well, of how the DART-measure works.


## Disclaimer
This is a measure I defined on my own, it is not the scientific result of a paper or of a deep study. I do not know whether it actually makes sense, whether it is useful and whether it can be further improved. Time and experiments will say that. In the meantime, it is not my responsibilty if you use it and you end up blowing your house with a bad model; further tests are still needed.
<br>
<br>

## Easter Eggs
- The name DART was used by NASA to denote a space mission aimed at testing a method of *planetary defense* against asteroids. This is why I have choosen to use ☄️ to denote the score of the measure. More info on: [Double Asteroid Redirection Test](https://wikipedia.org/wiki/Double_Asteroid_Redirection_Test). Also, my girlfriend will be an astrophysicist and she really liked it; <br>

- Typing *Double Asteroid Redirection Test* on Google will do something funny. Feel free to try it;
- Darts are thrown on a dartboard, and the goal is to get as close as possible to the center. Assuming the player is our model, we want it to be both precise and accurate.
