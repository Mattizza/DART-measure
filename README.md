<h1 style="text-align:center;"> DART: Definitely-Agnostic Ranked Trade-off</h1>
<h2 style="text-align:center;"> Accuracy is NOT the only statistic that matters </h2>

In a ML pipeline, the process of **model selection** and **model validation** is crucial, since it can be what mainly determines the **success** of a project. Still, when making comparisons of one model against the others, a large portion of practitioners tends to take into account just one statistic at a time; in most of the cases, the accuracy. For example, when tuning the hyperparameters on the validation set using Cross Validation, people tend to select the model that has the highest accuracy. What about the variance of such a model? In most of the cases it tends to be forgotten, even though it is what we really care about since it is a proxy for the **stability** of our model.<br>
> Considering the accuracy alone can be very risky since it may cause an **overestimation** of the performance of our model.

<br>In order to avoid this, we would like to have a ***single measure*** to make such comparisons during the model selection process, which takes into account both the accuracy **AND** the standard deviation of the model.<br>
Due to the, at the best of my knowledge, absence of such a measure, I have decided to derive a new one considering all the requirements stated above. Such measure should ideally have a **high value** when the **accuracy is high** and the **variance is small**; this would mean that we have found a very good model! In an opposite way, when this measure has a **low value**, it means that the model is very bad, since it has **low accuracy** and **high variance**.

<br>A qualitative behaviour can be found in the following table:
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

> *We use ☄️ to score the expected goodness of the measure and ✨ to refer to possible variations depending on the specific situation and the exact values.
Notice the Standard Deviation is what contributes the most to its value, depending on the desired stability. The lower, the better.


<br>In the following I am going to present a formula that can be used to merge these two values into a single one, and make more informed decisions. It is flexible and easily customizable depending on the problem at hand, and can be easily generalized to the case in which we want to take into account just the accuracy.

<br>


### Disclaimer
This is a measure I defined on my own, it is not the scientific result of a paper or of a deep study. I do not know whether it actually makes sense, whether it is useful and whether it can be further improved. Time and experiments will say that. In the meantime, it is not my responsibilty if you use it and you end up blowing your house with a bad model; further tests are still needed.
<br>
<br>

### Easter Eggs
- The name DART was used by NASA to denote a space mission aimed at testing a method of *planetary defense* against asteroids. This is why I have choosen to use ☄️ to denote the score of the measure. More info on: [Double Asteroid Redirection Test](https://wikipedia.org/wiki/Double_Asteroid_Redirection_Test). Also, my girlfriend will be an astrophysicist and she really liked it; <br>

- Typing *Double Asteroid Redirection Test* on Google will do something funny. Feel free to try it;
- Darts are thrown on a dartboard, and the goal is to get as close as possible to the center. Assuming the player is our model, we want it to be both precise and accurate.