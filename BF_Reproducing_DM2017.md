Reproducing Dienes and Mclatchie, 2017
================

The code below reproduces all examples of how to calculate Bayes Factors in Dienes and Mclatchie, 2017, 'Four reasons to prefer Bayesian analyses over significance testing'. Download the full article (\#Open Access) here: <https://link.springer.com/content/pdf/10.3758%2Fs13423-017-1266-z.pdf>

Below is the R code to calculate Bayes factors based on a t-distribution by Stefan Wiens from: <https://figshare.com/articles/Aladins_Bayes_Factor_in_R/4981154/3>

``` r
BF_t<-function(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail = 2)
# Bt(meantheory, sdtheory, dftheory), L = (meanobtained, semobtained, dfobtained) 
# tail = 1 means that no negative thetas are allowed.
# meantheory is assumed to be greater than or equal to zero (as Dienes's online calculator).
#  
# Computes the BayesFactor(H1 vs H0) with the H1 defined as a t distribution and the likelihood
# defined as a t distribution.
# It also plots the Prior and Posterior (and Likelihood) and adds a pie chart.
#  
#  This is a modified version of the R script presented here:  
#  Dienes, Z., & Mclatchie, N. (2017). Four reasons to prefer Bayesian analyses
#  over significance testing. Psychonomic Bulletin & Review, 1-12. doi:
#  10.3758/s13423-017-1266-z
#
# 170601 -- Stefan Wiens
# people.su.se/~swiens/
# Thanks to Henrik Nordstr?m, Mats Nilsson, Marco Tullio Liuzza, Anders Sand
  
# #Example
# meantheory = 11
# sdtheory = 5.4
# dftheory = 29
# meanobtained = 12
# semobtained = 5
# dfobtained = 81
# tail = 2
#BF_t(11, 5.4, 29, 12, 5, 81)
# should give 11.12

{
  # Create theta (ie parameter)
  # ===========================
  # This array ranges from -10*SDtheory to +10*SDtheory. 
  # Basically, one creates parameter values (theta) around the meantheory
  # This is a grid.
  
  theta <- meantheory - 10 * sdtheory
  incr <- sdtheory / 200
  theta=seq(from = meantheory - 10 * sdtheory, by = incr, length = 4001)
  # The original calculator is not centered on meantheory (because the loop starts with theta + incr)
  # ie, value at position 2001 in loop does not give the meantheory
  # theta[2001]

  # Create dist_theta (ie density of prior model)
  # =============================================
  # The prior is a t distribution characterized by meantheory, sdtheory, and dftheory
  # > mean effect (meantheory)
  # > standard deviation (sdtheory): This is the SEM from the t test in the original study
  # The t test = mean effect / SEM, so SEM = mean effect / t
  # Example: The original study reported mean effect = 12 and t(81) = 2.4
  # Thus, SEM = 12 / 2.4 = 5 
  # > df of t test (dftheory) 
  # Example: If the study reported mean effect = 12 and t(81) = 2.4,
  # meantheory=12, sdtheory=5, and dftheory=84
  # see Dienes & Mclatchie (2017) for examples
  #
  # To generate the prior, for  each level of theta:
  # > compute the t score (which is the standardized difference from the mean effect)
  # > find the corresponding density (height on Y axis, which depends on df)
  dist_theta <- dt(x = (theta-meantheory)/sdtheory, df=dftheory)

  # Is the prior one-tailed or two-tailed?
  # If one-tailed, only effects larger than zero are expected.
  # Accordingly, the density for negative thetas is set to zero.
  if(identical(tail, 1)){
    dist_theta[theta <= 0] = 0
  }
  
  # alternative computation with normalized vectors
  dist_theta_alt = dist_theta/sum(dist_theta)
  
  # Create likelihood
  # For each theta, compute how well it predicts the obtained mean, 
  # given the obtained SEM and the obtained dfs.
  # Note that the distribution is symmetric, it does not matter if one computes
  # meanobtained-theta or theta-meanobtained
  likelihood <- dt((meanobtained-theta)/semobtained, df = dfobtained)
  # alternative computation with normalized vectors
  likelihood_alt = likelihood/sum(likelihood)

  # Multiply prior with likelihood
  # this gives the unstandardized posterior
  height <- dist_theta * likelihood
  area <- sum(height * incr)
  # area <- sum(dist_height * incr * likelihood)
  normarea <- sum(dist_theta * incr)
  # alternative computation with normalized vectors
  height_alt = dist_theta_alt * likelihood_alt
  height_alt = height_alt/sum(height_alt)

  LikelihoodTheory <- area/normarea
  LikelihoodNull <- dt(meanobtained/semobtained, df = dfobtained)
  BayesFactor <- round(LikelihoodTheory / LikelihoodNull, 2)

  
  # ####
  # Plot
  # ####
  # create a new window
  plotscale = 0.7
  dev.new(width = 16 * plotscale, height = 9 * plotscale, noRStudioGD = T)
  
  # define title
  mytitle = paste0("BF for t(",round(meantheory, 1),", ", round(sdtheory, 1),", ",dftheory,
                   "), L = (",round(meanobtained, 2),", ",round(semobtained, 2),", ", dfobtained, "), tail = ", tail,
                   "\nBF10 = ", format(BayesFactor, digits = 2, nsmall = 2), ", BF01 = ", format(1/BayesFactor, digits = 2, nsmall = 2))
  

  mylegend = "R"   # <---- define legend on right ("R") or left
  # ===========================================================

  mypie = T  # <---- include pie chart, T or F
  # ==========================================
  if (mypie == T) {
    layout(cbind(1,2), widths = c(4,1))
  }

  # for many x values, the ys are very small.
  # define minimum y threshold that is plotted, in percent of the Y maximum in the whole plot.
  # Example: 1 means that only x values are plotted in which the y values are above 1% of the maximum of Y in the whole plot.
  myminY = 1
  # ====================================================
  
  # rescale prior and posterior to sum = 1 (density)
  dist_theta_alt = dist_theta_alt / (sum(dist_theta_alt)*incr)
  height_alt = height_alt/ (sum(height_alt)*incr)
  
  # rescale likelood to maximum = 1
  likelihood_alt = likelihood_alt / max(likelihood_alt)
  
  
  data = cbind(dist_theta_alt, height_alt)
  maxy = max(data)
  max_per_x = apply(data,1,max)
  max_x_keep = max_per_x/maxy*100 > myminY  # threshold (1%) here
  x_keep = which(max_x_keep==1)
  #plot(theta,max_x_keep)
  if (mylegend == "R") {  # right
    legend_coor = theta[tail(x_keep,1)-20]
    legend_adj = 1
  } else { # left
    legend_coor = theta[head(x_keep,1)+20]
    legend_adj = 0
  }
  
  plot(theta, dist_theta_alt, type = "l", 
       ylim = c(0, maxy),
       xlim = c(theta[head(x_keep,1)], theta[tail(x_keep,1)]),  # change X limits here
       ylab = "Density (for Prior and Posterior)", xlab = "Theta", col = "blue", lwd = 7, lty = 5)
  lines(theta, height_alt, type = "l", col = "red", lwd = 7, lty = 5)
  text(legend_coor,maxy-(maxy/10*1), "Prior (dotted)", col = "blue", adj = legend_adj, font = 2)
  text(legend_coor,maxy-(maxy/10*2), "Posterior (dashed)", col = "red", adj = legend_adj, font = 2)
  text(legend_coor,maxy-(maxy/10*3), "Likelihood", col = "black", adj = legend_adj, font = 2)
  title(mytitle)
  
  theta0 = which(theta == min(theta[theta>0]))
  cat("Theta is sampled discretely (and thus, zero may be missed).\n",
      "BF10 at theta =", theta[theta0], " is ", format(1/(height_alt[theta0]/dist_theta_alt[theta0]), digits = 2, nsmall = 2),"\n\n")
  points(theta[theta0],dist_theta_alt[theta0], pch = 19, col = "blue", cex = 3)
  points(theta[theta0],height_alt[theta0], pch = 19, col = "red", cex = 3)
  abline(v = theta[theta0], lwd = 2, lty = 3)

  par(new = T)
  plot(theta, likelihood_alt, type = "l", 
       ylim = c(0, 1),
       xlim = c(theta[head(x_keep,1)], theta[tail(x_keep,1)]),  # change X limits here
       col = "black", lwd = 5, lty = 3, axes = F, xlab = NA, ylab = NA)
  axis(side = 4)
  mtext(side = 4, line = 3, 'Likelihood')
  
  if (mypie == T) {
    # Pie chart of BF
    rotpie = BayesFactor/(BayesFactor+1)/2
    pie(c(BayesFactor, 1), labels = NA, col = c("red", "white"), init.angle = 90 - rotpie*360, clockwise = F)
    legend("top", c("data|H1", "data|H0"), fill = c("red", "white"), bty = "n")
    cat("Results:\nBF10 = ", format(BayesFactor, digits = 2, nsmall = 2), "\nBF01 = ", format(1/BayesFactor, digits = 2, nsmall = 2), "\n\n")}

  return(BayesFactor)
  # return(c(BayesFactor, LikelihoodTheory, LikelihoodNull))

}
```

And below is the code from Baguley and Kay: <http://www.academia.edu/427288/Review_of_Understanding_psychology_as_a_science_An_introduction_to_scientific_and_statistical_inference>

``` r
Bf<-function(sd, obtained, dfdata, uniform, lower=0, upper=1, meanoftheory=0, sdtheory=1, tail=2)
{
  area <- 0
  if(identical(uniform, 1)){
    theta <- lower
    range <- upper - lower
    incr <- range / 2000
    for (A in -1000:1000){
      theta <- theta + incr
      dist_theta <- 1 / range
      height <- dist_theta * dnorm(obtained, theta, sd)
      area <- area + height * incr
    }
  }else{
    theta <- meanoftheory - 5 * sdtheory
    incr <- sdtheory / 200
    for (A in -1000:1000){
      theta <- theta + incr
      dist_theta <- dnorm(theta, meanoftheory, sdtheory)
      if(identical(tail, 1)){
        if (theta <= 0){
          dist_theta <- 0
        } else {
          dist_theta <- dist_theta * 2
        }
      }
      height <- dist_theta * dt((obtained-theta)/sd, df=dfdata)
      area <- area + height * incr
    }
  }
  LikelihoodTheory <- area
  Likelihoodnull <- dt(obtained/sd, df = dfdata)
  BayesFactor <- LikelihoodTheory / Likelihoodnull
  ret <- list("LikelihoodTheory" = LikelihoodTheory, "Likelihoodnull" = Likelihoodnull, "BayesFactor" = BayesFactor)
  ret
}
```

Now we can try to reproduce the online calculator. Sample standard error: 5, Sample mean: 12, mean theory: 0, sd theory: 11, 2 tailed. This is from Dienes and Mclatchie, 2017:

*There was an 11% difference in means, t(29) = 2.02, P = .053. Gibson, Losee, and Vitiello (2014) replicated the procedure with 83 subjects in the two groups (who were aware of the nature of the race and gender stereotypes); for these selected participants, the difference was 12%, t(81) = 2.40, P = .02. So there is a significant effect with a raw effect size almost identical to that in the original study. Correspondingly, BH(0, 11) = 4.50*

``` r
#Example
meantheory = 0
sdtheory = 11
dftheory = 1000000 #By setting df to large number, t is same as normal dist.
meanobtained = 12
semobtained = 5 #SE = meandiff/t. Which is 12/2.40
dfobtained = 1000000 #By setting df to large number, t is same as normal dist.
tail = 2

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail = 2)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.055  is  4.38

    ## Results:
    ## BF10 =  4.50 
    ## BF01 =  0.22

    ## [1] 4.5

``` r
Bf(sd=5, obtained=12, dfdata=10000000, uniform=0, meanoftheory=0, sdtheory=11, tail=2)
```

    ## $LikelihoodTheory
    ## [1] 0.1008164
    ## 
    ## $Likelihoodnull
    ## [1] 0.02239454
    ## 
    ## $BayesFactor
    ## [1] 4.501827

Or following the footnote:

*We can also model H1 using the t-distribution method; Bt(11, 5.4, 29), L = t(12, 5, 81) = 11.12, also indicating substantial evidence for the relevant H1 over H0.*

``` r
#11% difference in means, t(29) = 2.02, P = .053
meantheory = 11
sdtheory = 5.4 #SE = meandiff/t. Which is 11/2.02 rounded from 5.445545 as in D&M 2017 to reproduce their result. 
dftheory = 29 
#12%, t(81) = 2.40, P= .02
meanobtained = 12
semobtained = 5 #SE = meandiff/t. Which is 12/2.40
dfobtained = 81 #By setting df to large number, t is same as normal dist.
tail = 2

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail = 2)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.011  is  11.07

    ## Results:
    ## BF10 =  11.12 
    ## BF01 =  0.09

    ## [1] 11.12

Example 2:

*The strength of this relation can be expressed as an odds ratio (OR) = (75% x 54%)/(46% x 25%) = 3.52. The log of the OR is roughly normally distributed; taking natural logs this gives a measure of effect size, that is, ln OR = 1.26. Lynott, Corker, Wortman, Connell et al. (2014) attempted a replication with total N = 861 people, a sample size a factor of 10 higher than the original study. The results went somewhat in the opposite direction, OR = 0.77, so ln OR = -0.26, with a standard error of 0.14.6 So z = 0.26/0.14 = 1.86, P = .062, which is non-significant. Correspondingly, BH(0, 1.26) = 0.04*

Note that this seems to be a one-tailed test, while the previous examples were not

``` r
Bf(sd=0.14, obtained=-0.26, dfdata=10000000, uniform=0, lower=0, upper=1, meanoftheory=0, sdtheory=1.26, tail=1)
```

    ## $LikelihoodTheory
    ## [1] 0.002944571
    ## 
    ## $Likelihoodnull
    ## [1] 0.07111705
    ## 
    ## $BayesFactor
    ## [1] 0.04140457

Example 3:

*Banerjee, Chatterjee, & Sinha (2012, study 2) found that people asked to recall a time that they behaved unethically rather than ethically estimated the room to be darker by 13.30 W, t(72) = 2.70, P = .01. Brandt, IJzerman, and Blanken (2014 laboratory replication) tried to replicate the procedure as closely as possible, using N = 121 participants, sufficient for a power (to pick up the original effect) greater than 0.9. Brandt et al. (2014) obtained a difference of 5.5 W, t(119) = 0.17, P = 0.87. Using our standard representation of plausible effect sizes, a halfnormal scaled by the original effect size (i.e. allowing effect sizes between very small and twice the original effect), we get BH(0, 13,3) = 0.97.*

``` r
5.5/0.17 #Get sd by dividing observed difference by observed t
```

    ## [1] 32.35294

``` r
Bf(sd=32.35, obtained=5.5, dfdata=10000000, uniform=0, lower=0, upper=1, meanoftheory=0, sdtheory=13.30, tail=1)
```

    ## $LikelihoodTheory
    ## [1] 0.3824432
    ## 
    ## $Likelihoodnull
    ## [1] 0.393218
    ## 
    ## $BayesFactor
    ## [1] 0.9725985

Or following the footnote:

*We can also model H1 using the t-distribution method; Bt(13.3, 4.93, 72), L = t(5.47, 32.2, 119) = 0.97, giving exactly the same answer as the Bayes factor in the text.*

``` r
#13.30 W, t(72) = 2.70
meantheory = 13.30
sdtheory = 4.93 #SE = meandiff/t. Which is 13.30/2.70 rounded from 4.925926
dftheory = 72 
#5.5 W, t(119) = 0.17
meanobtained = 5.5
semobtained = 32.35 #SE = meandiff/t. Which is 5.5/0.17 = 32.35294
dfobtained = 119 #By setting df to large number, t is same as normal dist.
tail = 2

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.01365  is  0.97

    ## Results:
    ## BF10 =  0.97 
    ## BF01 =  1.03

    ## [1] 0.97

Example 4:

*Shih, Pittinsky, and Ambady (1999) argued that American Asian women primed with an Asian identity will perform better on a maths test than unprimed women; indeed, in the sample means, priming showed an advantage of 5% more questions answered correctly.8 Moon and Roeder (2014) replicated the study, with about 50 subjects in each group; power based on the original d = 0.25 effect is 24%. Given the low power, perhaps it is not surprising that the replication yielded a non-significant effect, t(99) = 1.15, P = 0.25. However, it would be wrong to conclude that the data were not evidential. The mean difference was 4% in the wrong direction according to the theory. When the data go in the wrong direction (by a sufficient amount relative to the standard error), they should carry some evidential weight against the theory. Testing the directional theory by modelling H1 as a half-normal with a standard deviation of 5%, BH(0, 5) = 0.31, substantial evidence for the null relative to the H1.*

``` r
-4/1.15 #Get sd by dividing observed difference by observed t
```

    ## [1] -3.478261

``` r
Bf(sd=3.478261, obtained=-4, dfdata=10000000, uniform=0, lower=0, upper=1, meanoftheory=0, sdtheory=5, tail=1)
```

    ## $LikelihoodTheory
    ## [1] 0.06296929
    ## 
    ## $Likelihoodnull
    ## [1] 0.2059363
    ## 
    ## $BayesFactor
    ## [1] 0.3057707

And again, following the footnote:

*As before, the effect can also be tested modelling H1 as a t-distribution with a mean equal to the original mean difference (5%) and SE equal to the original SE of that difference (estimated as 14%). Bt(5, 14, 30), L = t(-4, 3.48, 99) = 0.38.*

``` r
meantheory = 5
sdtheory = 14 
dftheory = 30 
meanobtained = -4
semobtained = 3.478261
dfobtained = 99 
tail = 2

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail = 2)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.03  is  0.38

    ## Results:
    ## BF10 =  0.38 
    ## BF01 =  2.63

    ## [1] 0.38

And example 5, using a uniform:

*Schnall, Benton, and Harvey (2008) found that people make less severe judgments on a 1 (perfectly OK) to 7 (extremely wrong) scale when they wash their hands after experiencing disgust (Exp. 2). Of the different problems they investigated, taken individually, the wallet problem was significant, with a mean difference of 1.11, t(41) = 2.57, P = .014. Johnson, Cheung, and Donnellan (2014; study 2) replicated with an N of 126, giving a power of greater than 99% to pick up the original effect. The obtained mean difference was 0.15, t(124) = 0.63, P = 0.53. Thus, there is a high-powered nonsignificant result. But, as is now clear, that still leaves open the question of how much evidence there is, if any, for H0 rather than H1. The predictions of H1 could be represented as a uniform distribution from 0 to 6. That claim has the advantage of simplicity, as it can be posited without reference to data. These considerations give BU\[0, 6\] = 0.09. That is, there is substantial evidence for H0 over this H1. We also have our half-normal model for representing H1. The original raw effect size was 1.11 rating units; and, BH(0, 1.11) = 0.37.12 That is, the data do not very sensitively distinguish H0 from this H1.*

``` r
#0.15, t(124) = 0.63 
#Get sd by dividing observed difference by observed t
0.15/0.63
```

    ## [1] 0.2380952

Using Wiens function:

``` r
BF_U<-function(LL, UL, meanobtained, semobtained, dfobtained)
# similar to BF_t (see there for more info) but the H1 is modelled as a uniform.
# LL = lower limit of uniform
# UL = upper limit of uniform
#  
# Computes the BayesFactor(H1 vs H0) with the H1 defined as a uniform distribution 
# and the likelihood defined as a t distribution.
# It also plots the Prior and Posterior (and Likelihood) and adds a pie chart.
#  
#  This is a modified version of the R script presented here:  
#  Dienes, Z., & Mclatchie, N. (2017). Four reasons to prefer Bayesian analyses
#  over significance testing. Psychonomic Bulletin & Review, 1-12. doi:
#  10.3758/s13423-017-1266-z
#
# 170601 -- Stefan Wiens
# people.su.se/~swiens/
# Thanks to Henrik Nordstr?m, Mats Nilsson, Marco Tullio Liuzza, Anders Sand
  
# #Example
# LL = 0
# UL = 10
# meanobtained = 12
# semobtained = 5
# dfobtained = 27
# 
# BF_U(0, 10, 12, 5, 27)
# should give 6.34
#
# dfobtained = 10000
# use this to have a normal distribution as likelihood (as in Dienes online calculator)
# BF_U(0, 10, 12, 5, 10000)
# should give 7.51

{
  # Create theta (ie parameter)
  # ===========================
  theta = ((UL+LL)/2) - (2 * (UL-LL))
  tLL <- ((UL+LL)/2) - (2 * (UL-LL))
  tUL <- ((UL+LL)/2) + (2 * (UL-LL))
  incr <- (tUL - tLL) / 4000
  theta=seq(from = theta, by = incr, length = 4001)
  # The original calculator is not centered on meantheory (because the loop starts with theta + incr)
  # ie, value at position 2001 in loop does not give the meantheory
  # theta[2001]

  # Create dist_theta (ie density of prior model)
  # =============================================
  dist_theta = numeric(4001)
  dist_theta[theta>=LL & theta<=UL] = 1

  # alternative computation with normalized vectors
  dist_theta_alt = dist_theta/sum(dist_theta)
  
  # Create likelihood
  # For each theta, compute how well it predicts the obtained mean, 
  # given the obtained SEM and the obtained dfs.
  # Note that the distribution is symmetric, it does not matter if one computes
  # meanobtained-theta or theta-meanobtained
  likelihood <- dt((meanobtained-theta)/semobtained, df = dfobtained)
  # alternative computation with normalized vectors
  likelihood_alt = likelihood/sum(likelihood)

  # Multiply prior with likelihood
  # this gives the unstandardized posterior
  height <- dist_theta * likelihood
  area <- sum(height * incr)
  # area <- sum(dist_height * incr * likelihood)
  normarea <- sum(dist_theta * incr)

  # alternative computation with normalized vectors
  height_alt = dist_theta_alt * likelihood_alt
  height_alt = height_alt/sum(height_alt)

  LikelihoodTheory <- area/normarea
  LikelihoodNull <- dt(meanobtained/semobtained, df = dfobtained)
  BayesFactor <- round(LikelihoodTheory / LikelihoodNull, 2)

  
  # ####
  # Plot
  # ####
  # create a new window
  plotscale = 0.7
  dev.new(width = 16 * plotscale, height = 9 * plotscale, noRStudioGD = T)
  
  # define title
  mytitle = paste0("BF for U(LL = ",LL,", UL = ", UL,
               "), L = (",round(meanobtained, 2),", ",round(semobtained, 2),", ", dfobtained,  
               ")\nBF10 = ", format(BayesFactor, digits = 2, nsmall = 2), ", BF01 = ", format(1/BayesFactor, digits = 2, nsmall = 2))
  
  mylegend = "R"   # <---- define legend on right ("R") or left
  # ===========================================================
  
  mypie = T  # <---- include pie chart, T or F
  # ==========================================
  if (mypie == T) {
    layout(cbind(1,2), widths = c(4,1))
  }
  
  # for many x values, the ys are very small.
  # define minimum y threshold that is plotted, in percent of the Y maximum in the whole plot.
  # Example: 1 means that only x values are plotted in which the y values are above 1% of the maximum of Y in the whole plot.
  myminY = 1
  # ====================================================
  
  # rescale prior and posterior to sum = 1 (density)
  dist_theta_alt = dist_theta_alt / (sum(dist_theta_alt)*incr)
  height_alt = height_alt/ (sum(height_alt)*incr)

  # rescale likelood to maximum = 1
  likelihood_alt = likelihood_alt / max(likelihood_alt)

  data = cbind(dist_theta_alt, height_alt)
  maxy = max(data)
  max_per_x = apply(data,1,max)
  max_x_keep = max_per_x/maxy*100 > myminY  # threshold (1%) here
  x_keep = which(max_x_keep==1)
  #plot(theta,max_x_keep)
  if (mylegend == "R") {  # right
    legend_coor = theta[tail(x_keep,1)-20]
    legend_adj = 1
    } else { # left
    legend_coor = theta[head(x_keep,1)+20]
    legend_adj = 0
    }
  
  plot(theta, dist_theta_alt, type = "l", 
       ylim = c(0, maxy),
       xlim = c(theta[head(x_keep,1)], theta[tail(x_keep,1)]),  # change X limits here
       ylab = "Density (for Prior and Posterior)", xlab = "Theta", col = "blue", lwd = 7, lty = 5)
  lines(theta, height_alt, type = "l", col = "red", lwd = 7, lty = 5)
  text(legend_coor,maxy-(maxy/10*1), "Prior (dotted)", col = "blue", adj = legend_adj, font = 2)
  text(legend_coor,maxy-(maxy/10*2), "Posterior (dashed)", col = "red", adj = legend_adj, font = 2)
  text(legend_coor,maxy-(maxy/10*3), "Likelihood", col = "black", adj = legend_adj, font = 2)
  title(mytitle)

  theta0 = which(theta == min(theta[theta>0]))
  cat("Theta is sampled discretely (and thus, zero may be missed).\n",
      "BF10 at theta =", theta[theta0], " is ", format(1/(height_alt[theta0]/dist_theta_alt[theta0]), digits = 2, nsmall = 2),"\n\n")
  if (LL <= 0 & UL >= 0) { # Plot dots only if zero is included in prior
    points(theta[theta0], dist_theta_alt[theta0], pch = 19, col = "blue", cex = 3)
    points(theta[theta0], height_alt[theta0], pch = 19, col = "red", cex = 3)
    abline(v = theta[theta0], lwd = 2, lty = 3)}
  
  par(new = T)
  plot(theta, likelihood_alt, type = "l", 
       ylim = c(0, 1),
       xlim = c(theta[head(x_keep,1)], theta[tail(x_keep,1)]),  # change X limits here
       col = "black", lwd = 5, lty = 3, axes = F, xlab = NA, ylab = NA)
  axis(side = 4)
  mtext(side = 4, line = 3, 'Likelihood')
  
  if (mypie == T) {
    # Pie chart of BF
    rotpie = BayesFactor/(BayesFactor+1)/2
    pie(c(BayesFactor, 1), labels = NA, col = c("red", "white"), init.angle = 90 - rotpie*360, clockwise = F)
    legend("top", c("data|H1", "data|H0"), fill = c("red", "white"), bty = "n")
    cat("Results:\nBF10 = ", format(BayesFactor, digits = 2, nsmall = 2), "\nBF01 = ", format(1/BayesFactor, digits = 2, nsmall = 2), "\n\n")}
  
  return(BayesFactor)
  # return(c(BayesFactor, LikelihoodTheory, LikelihoodNull))

}

LL<-0
UL<-6
meanobtained<-0.15
semobtained<-0.2380952
dfobtained<-124
BF_U(LL, UL, meanobtained, semobtained, dfobtained)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.006  is  0.088

    ## Results:
    ## BF10 =  0.09 
    ## BF01 =  11.11

    ## [1] 0.09

And: *We also have our half-normal model for representing H1. The original raw effect size was 1.11 rating units; and, BH(0, 1.11) = 0.37.12 That is, the data do not very sensitively distinguish H0 from this H1*

``` r
meantheory = 0
sdtheory = 1.11
dftheory = 100000
meanobtained = 0.15
semobtained = 0.2380952
dfobtained = 100000 
tail = 1

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 0.00555  is  0.36

    ## Results:
    ## BF10 =  0.37 
    ## BF01 =  2.70

    ## [1] 0.37

And the footnote:

*Using the t-distribution model, Bt(1.11, 0.43, 41), L = t(0.15, 0.24, 124) = 0.09. The value is lower than the Bayes factor of 0.37 based on the half-normal provided in the text*

``` r
meantheory = 1.11
sdtheory = 0.4319066 #1.11/2.57
dftheory = 41 
meanobtained = 0.15
semobtained = 0.2380952
dfobtained = 124 
tail = 1

BF_t(meantheory, sdtheory, dftheory, meanobtained, semobtained, dfobtained, tail)
```

    ## Theta is sampled discretely (and thus, zero may be missed).
    ##  BF10 at theta = 3.8e-08  is  0.089

    ## Results:
    ## BF10 =  0.09 
    ## BF01 =  11.11

    ## [1] 0.09
