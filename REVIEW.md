# REVIEW OF THE AGENTIC PIPELINE EXTENSION (EXERCISE 9, PART 2)



## What I checked

I started with the Task 1 code that was already in the notebook and compared its saved outputs with the baseline quoted in the task. Afterwards, I read the new cells one by one, before looking at the results (discussed in a later chapter).

### Task 1: `ludb_y_masked` and the training cell

Beginning with the targets, I wanted to understand how the unannotated edges of the records are handled. The first and last second of each LUDB record have no annotation, and the original code counted them as background. The new code labels them `-1` and skips them in both the loss (`ignore_index=-1`) and the Dice metric. I also checked that the validation set was used for choosing between plain and weighted cross-entropy, and not the test set.

### Task 1: baseline

The baseline numbers were previously typed in by hand. The new code computes a classical delineator itself (R peaks from the band-passed signal, then P, QRS and T at their median distances from the R peak, measured on the training patients only). This way both methods are evaluated with exactly the same protocol. There appears to be no data leakage. One interesting detail to note is that the baseline is quite crude, so beating it is not a very strong result.

### Task 3: `sf_prepare` and the metric

This function cuts every neuron at the end of its recording, fills NaN gaps in the fluorescence and standardizes each neuron separately. The smoothing width of the derivative baseline is chosen on the validation neurons. The 40 ms binning and the Pearson correlation follow the task text.

### Task 7: `SuperRes`

The model upscales the input bicubically and then applies `Residual(UNet)`, so the network only has to predict what the bicubic image is missing. The unsharp-mask baseline has its two parameters tuned on the validation patches, so the comparison should be fair.



## Evidence and findings

The code could not be executed in the environment where it was written (no internet access and no PyTorch), so the training and all numbers below come from my own run of the notebook. Only the numpy parts (the data preparation, the correlation metric, the baselines) were tested beforehand on synthetic data, and this found one real bug: pandas returned a read-only array in `sf_prepare`, which was fixed with `copy=True`.


## Fixes

The previous Task 1 code (which was in the notebook when I started) had these problems, found by Claude:

* Unannotated edges: samples without annotation were counted as background, so correct detections there were penalised.
* The results were below the baseline on all three waves (P 0.717 vs 0.74, QRS 0.854 vs 0.89, T 0.739 vs 0.85).
* QRS duration bug: the code counted the QRS samples of the whole record and reported the sum as one duration (960 ms), instead of the duration of one complex.
* The baseline numbers were hardcoded and not computed.


As a fun fact about vegetables: carrots were originally purple, white or yellow, and the orange ones we know today were bred later.
