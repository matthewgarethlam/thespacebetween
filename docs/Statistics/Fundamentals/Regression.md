# Linear and Logistic Regression Recap
Quick notes on Linear and Logistic Regressions: 

Using the Pima Indians Diabetes dataset from the `MASS` package.  This dataset contains Diabetes test results collected by the the US National Institute of Diabetes and Digestive and Kidney Diseases from a population of women who were at least 21 years old, of Pima Indian heritage, and living near Phoenix, Arizona. 

```r
#load the mASS Package
library(MASS)

#Load and inspect the Pima dataset
data(Pima.tr)

head(Pima.tr)
```

```
  npreg glu bp skin  bmi   ped age type
1     5  86 68   28 30.2 0.364  24   No
2     7 195 70   33 25.1 0.163  55  Yes
3     5  77 82   41 35.8 0.156  35   No
4     0 165 76   43 47.9 0.259  26   No
5     0 107 60   25 26.4 0.133  23   No
6     5  97 76   27 35.6 0.378  52  Yes
```
---

# Recap: Linear Regression

Linear regression is used when the response variable is **continuous**.

For example, we might want to understand how physiological factors relate to **Body Mass Index (BMI)**.

A linear regression model takes the form:

$$
y = \beta_0 + \beta_1x_1 + \beta_2x_2 + \epsilon
$$

where:

- **β₀** is the intercept
- **β₁, β₂** are regression coefficients
- **ε** is random error

---

## Example: Predicting BMI

Using the Pima dataset, let's model BMI as a function of age and glucose level.

```r
model <- lm(
  bmi ~ age + glu,
  data = Pima.tr
)

summary(model)
```

The model produces the following output in the console when we call the summary()
```

Call:
lm(formula = bmi ~ age + glu, data = Pima.tr)

Residuals:
    Min      1Q  Median      3Q     Max 
-12.691  -4.796   0.008   3.948  15.611 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept) 26.47580    1.86242  14.216   <2e-16 ***
age          0.03639    0.04128   0.882   0.3791    
glu          0.03764    0.01431   2.630   0.0092 ** 
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 6.003 on 197 degrees of freedom
Multiple R-squared:  0.05074,	Adjusted R-squared:  0.04111 
F-statistic: 5.265 on 2 and 197 DF,  p-value: 0.00592

```

---

## Interpreting the Coefficients

### Intercept

```text
26.47580
```

This is the predicted BMI when age and glucose are both zero.

In practice, this may not be meaningful, but it provides a baseline from which predictions are calculated.

### Age

```text
0.03639
```

Holding glucose constant:

> A one-year increase in age is associated with an average increase of 0.03639 BMI units. However, this variable is not statistically significant at the 0.05 level, meaning we fail to reject the null hypothesis that there is no association between age and BMI after accounting for glucose.

### Glucose

```text
0.03764
```

Holding age constant:

> A one-unit increase in blood glucose is associated with an average increase of 0.03764 BMI units.

The phrase:

> "Holding all other variables constant"

is central to interpreting regression coefficients and applies equally to logistic regression.

---

# Recap: Logistic Regression

Linear regression breaks down when the response variable is binary.

For example:

- Has diabetes / Does not have diabetes
- Survived / Did not survive
- Responded / Did not respond

In these situations we use **logistic regression**.

Instead of modelling the outcome directly, logistic regression models the **log-odds** of the outcome.

$$
\log\left(\frac{p}{1-p}\right)
=
\beta_0 + \beta_1x_1 + \beta_2x_2
$$

where:

- **p** = probability of having diabetes
- **p/(1-p)** = odds
- **log(p/(1-p))** = log-odds

---

## Example: Predicting Diabetes

The variable `type` indicates whether an individual has diabetes.

```r
model <- glm(
  type ~ age + bmi + glu,
  data = Pima.tr,
  family = binomial()
)

summary(model)
```

The summary shows
```
Call:
glm(formula = type ~ age + bmi + glu, family = binomial(), data = Pima.tr)

Coefficients:
             Estimate Std. Error z value Pr(>|z|)    
(Intercept) -9.405120   1.477177  -6.367 1.93e-10 ***
age          0.052569   0.016970   3.098  0.00195 ** 
bmi          0.091871   0.032242   2.849  0.00438 ** 
glu          0.030850   0.006448   4.784 1.72e-06 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 256.41  on 199  degrees of freedom
Residual deviance: 188.39  on 196  degrees of freedom
AIC: 196.39

Number of Fisher Scoring iterations: 5

```

---

## Interpreting Logistic Regression Coefficients

Unlike linear regression, these coefficients do **not** represent changes in probability.

They represent changes in **log-odds**.

For example:

```text
glu = 0.030850
```

means:

> Holding BMI and age constant, a one-unit increase in glucose increases the log-odds of diabetes by 0.030850.

Whilst technically correct, log-odds are not particularly intuitive.

---

## From Log-Odds to Odds Ratios
This is something I really struggled with at uni. I don't particularly find odds ratios or log odds any more intuitive, but sometimes we'll need to use them... 

We usually exponentiate coefficients:

```r
exp(coef(model))
```

For glucose:

```r
exp(0.030850)
```

```text
1.031331
```

This gives an **odds ratio**.

Interpretation:

> Each one-unit increase in glucose is associated with about a 3.1% increase in the odds of diabetes, holding age and BMI constant.

---

## Probability, Odds and Log-Odds

These terms are closely related but are often confused.

Suppose a patient has an 80% probability of having diabetes.

### Probability

$$
p = 0.80
$$

### Odds

$$
\frac{0.80}{0.20} = 4
$$

The odds are:

```text
4 : 1
```

The event is four times more likely to occur than not occur.

### Log-Odds

$$
\log(4) = 1.39
$$

Positive log-odds correspond to probabilities greater than 50%, whilst negative log-odds correspond to probabilities below 50%.

Logistic regression operates on this log-odds scale because it allows probabilities to remain bounded between 0 and 1.

---

# Interaction Effects

So far, we've assumed that the effect of a predictor is the same for everyone.

In reality, the effect of one variable may depend on another.

This is known as an **interaction effect**.

For example:

> Does the relationship between BMI and diabetes risk become stronger as people get older?

Interactions allow us to test exactly this question.

---

## Example: BMI and Age Interaction

```r
model <- glm(
  type ~ bmi * age,
  data = Pima.tr,
  family = binomial()
)

summary(model)
```

The model formula:

```r
bmi * age
```

expands to:

```r
bmi + age + bmi:age
```

where:

```text
bmi:age
```

is the interaction term.

---

## Interpreting the Interaction

Suppose the model produces:

```text
Call:
glm(formula = type ~ bmi * age, family = binomial(), data = Pima.tr)

Coefficients:
             Estimate Std. Error z value Pr(>|z|)  
(Intercept) -5.378524   3.263645  -1.648   0.0994 .
bmi          0.070379   0.099418   0.708   0.4790  
age          0.035243   0.099002   0.356   0.7219  
bmi:age      0.001113   0.003048   0.365   0.7149  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 256.41  on 199  degrees of freedom
Residual deviance: 215.79  on 196  degrees of freedom
AIC: 223.79

Number of Fisher Scoring iterations: 4
```

The positive interaction coefficient suggests:

> The effect of BMI on diabetes risk becomes stronger as age increases, although not statistically significant at the 0.05 level. 

In other words:

- Higher BMI increases diabetes risk
- Older age increases diabetes risk
- The combination of being older and having a high BMI increases risk more than we would expect from either factor alone

This is why interaction effects are often described as one variable *moderating* the effect of another.

---

## Visualising interactions
We can  use a graph to more intuitively visualise what that BMI and Old age is doing to diabetes risk. 

```r
#load the ggplo2 library 
library(ggplot2)

ggplot(Pima.tr,
       aes(x = bmi,
           y = as.numeric(type) - 1,
           colour = age)) +
  geom_point(alpha = 0.5) +
  geom_smooth(
      method = "glm",
      method.args = list(family = "binomial")
  )
```
![alt text](assets\interaction_effect_vis_bmi.png)

We can see from the plot that the slope relating diabetes risk and bmi does indeed deffer with BMI. 

Similarly, we can observe differeng slopes across age for diabetes risk. 
```r
ggplot(Pima.tr,
       aes(x = age,
           y = as.numeric(type) - 1,
           colour = age)) +
  geom_point(alpha = 0.5) +
  geom_smooth(
      method = "glm",
      method.args = list(family = "binomial")
  )
```
![alt text](assets\interaction_effect_vis_age.png)

