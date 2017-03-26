# R_benchmarking

Benchmarking R and C++ for Machine Learning Metrics.

Code is provided here to copy & paste very quickly if needed to use immediately in R. Requires Rtools if using Windows.

*Note: the processed times per hour (function, number) reported are only applicable if you follow the **Rules**.<br>Which means, almost never.*

---

# Average Benchmarks

Reported numbers are:

* The mean speed in milliseconds for the timings.
* In percentage `(R / Rcpp) - 1` for the `Speed` (positive means Rcpp faster, negative means R faster)
* The number of possible runs per hour for the best functions.
* The best R and the best Rcpp functions found.
* A `YES` to the `PASS` means the function did not report an non-conform number.

| Benchmark | Speed | Rcpp (C++) | Pure R | Rcpp obs/s | R obs/s | Pass |
| ------ | ---: | ---: | ---: | ---: | ---: | -: |
| Binary Logarithmic Loss | +30.63% | 83.652 ms | 109.273 ms | 11,954,295 obs/s | 9,151,369 obs/s | YES |
| Multiclass Logarithmic Loss | +17.09% | 77.465 ms | 90.703 ms | 12,908,979 obs/s | 11,024,998 obs/s | YES |
| Area Under the Curve (ROC) | +48.11% | 152.761 ms | 226.256 ms | 3,273,091 obs/s | 2,209,883 obs/s | YES |
| Vector to Matrix to Vect | +9.44% | 26.335 ms | 28.823 ms | 37,972,035 obs/s | 34,694,322 obs/s | YES |

---

# Metric Benchmarks

---

## Binary Logartihmic Loss: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/logloss.nb.html)

**WARNING: THERE MUST BE A FASTER WAY.** (because it is too slow compared to multiclass logarithmic loss)

### Speed (milliseconds)

* R is 30.63% slower than Rcpp.
* Rcpp can process the function **43,035** times per hour (11,954,295 processed observations per second).
* R can process the function **32,945** times per hour (9,151,369 processed observations per second).
* Fastest functions only. Compiled with `-O2 -Wall $(DEBUGFLAG) -mtune=core2` flags (R's defaults).

| Type | Mean | Min | 25% | 50% | 75% | Max |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Rcpp (C++) | 83.65194 | 77.73338 | 79.28947 | 81.17837 | 87.28432 | 165.6286 |
| Pure R | 109.27326 | 96.42888 | 101.44859 | 103.66782 | 107.99349 | 199.9849 |

### Rules (1,000,000 observations)

For a 2-class vector of 1,000,000 observations:

* Vector A of length=(1000000)
* Vector B of length=(1000000) with 2 classes

```
A = [1, 2, 3, 4, ..., 1000000]
B = [0, 1, 1, 0, ...]
```

Get the following Vector C and D:

```
C = Clamped A by 1e-15
D = Mean of logloss(C, B)
```

### Best code

`Lpp_logloss(preds, labels, eps)`:

* preds = your predictions (between 0 and 1)
* labels = your labels (binary, 0 or 1)
* eps = the clamping on [0, 1]

```r
Rcpp::cppFunction("double Lpp_logloss(NumericVector preds, NumericVector labels, double eps) {
  NumericVector clamped = clamp(eps, preds, 1 - eps);
  NumericVector loggy = -1 * ((labels * log(clamped) + (1 - labels) * log(1 - clamped)));
  double logloss = sum(loggy)/loggy.size();
  return logloss;
}")
```

Correctness: **passing**.

---

## Multiclass Logarithmic Loss: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/mlogloss.nb.html)

### Speed (milliseconds)

* R is 17.09% slower than Rcpp.
* Rcpp can process the function **46,472** times per hour (12,908,979 processed observations per second).
* R can process the function **39,690** times per hour (11,024,998 processed observations per second).
* Fastest functions only. Compiled with `-O2 -Wall $(DEBUGFLAG) -mtune=core2` flags (R's defaults).

| Type | Mean | Min | 25% | 50% | 75% | Max |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Rcpp (C++) | 77.46546 | 68.49568 | 70.41765 | 79.34706 | 80.94364 | 149.4028 |
| Pure R | 90.70296 | 77.04039 | 83.82450 | 86.53773 | 89.63670 | 178.2875 |

### Rules (1,000,000 observations)

For a 10-class vector of 1,000,000 observations:

* Vector A of length=(1000000 * 10)
* Vector B of length=(1000000) with 10 classes

```
A = [1:1, 1:2, 1:3, 1:4... 1:10, 2:1, 2:2, 2:3..., 1000000:8, 1000000:9, 1000000:10]
B = [3, 5, 9, 1, 4, 8, 6, ...]
```

Get the following Vector C, D, and E:

```
C = [1:4, 2:6, 3:10, 4:2, 5:5, 6:9, 7:7, ...]
D = Clamped C by 1e-15
E = Mean of logloss(D, B)
```

### Best code

`Lpp_mlogloss(preds, labels, eps)`:

* preds = your predictions (size = `length(labels) * number of different labels`)
* labels = your labels (starting from 0)
* eps = the clamping on [0, 1]

```r
Rcpp::cppFunction("double Lpp_mlogloss(NumericVector preds, NumericVector labels, double eps) {
  int labels_size = labels.size();
  NumericVector selected(labels_size);
  selected = (preds.size() / labels_size) * seq(0, labels_size - 1);
  selected = selected + labels;
  NumericVector to_return(labels_size);
  to_return = preds[selected];
  NumericVector clamped = clamp(eps, to_return, 1 - eps);
  NumericVector loggy = -(log(1 - clamped));
  double logloss = sum(loggy) / labels_size;
  return logloss;
}")
```

Correctness: **passing**.

---

## Area Under the Curve (ROC): [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/roc.nb.html)

### Speed (milliseconds)

* R is 48.11% slower than Rcpp.
* Rcpp can process the function **23,566** times per hour (3,273,091 processed observations per second).
* R can process the function **15,911** times per hour (2,209,883 processed observations per second).
* Fastest functions only. Compiled with `-O2 -Wall $(DEBUGFLAG) -mtune=core2` flags (R's defaults).

| Type | Mean | Min | 25% | 50% | 75% | Max |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Rcpp (C++) | 152.7608 | 139.1281 | 145.9256 | 148.8292 | 154.0724 | 333.5406 |
| Pure R | 226.2563 | 194.5239 | 206.4760 | 215.3461 | 228.2616 | 430.9951 |

### Rules (500,000 observations)

For a 2-class vector of 500,000 observations:

* Vector A of length=(500000)
* Vector B of length=(500000) with 2 classes

```
A = [1, 2, 3, 4, ..., 500000]
B = [0, 1, 1, 0, ...]
```

Get the following Vector C:

C = ROC of A and B

### Best code

`Lpp_ROC(preds, labels)`:

* preds = your predictions
* labels = your labels (binary, 0 or 1)

```r
Rcpp::cppFunction("double Lpp_ROC(NumericVector preds, NumericVector labels) {
  double LabelSize = labels.size();
  NumericVector ranked(LabelSize);
  NumericVector positives = preds[labels == 1];
  double n1 = positives.size();
  Range positives_seq = seq(0, n1 - 1);
  ranked[seq(0, n1 - 1)] = positives;
  double n2 = LabelSize - n1;
  NumericVector negatives = preds[labels == 0];
  NumericVector x2(n2);
  ranked[seq(n1, n1 + n2)] = negatives;
  ranked = match(ranked, clone(ranked).sort());
  double AUC = (sum(ranked[positives_seq]) - n1 * (n1 + 1)/2)/(n1 * n2);
  return AUC;
}")
```

Correctness: **passing**.

---

# Utilities Benchmarks

---

## Vector to Matrix to Vector: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/vect2mat2vect.nb.html)

* R is 9.44% slower than Rcpp.
* Rcpp can process the function **136,699** times per hour (37,972,035 processed observations per second).
* R can process the function **124,900** times per hour (34,694,322 processed observations per second).
* Fastest functions only. Compiled with `-O2 -Wall $(DEBUGFLAG) -mtune=core2` flags (R's defaults).

| Type | Mean | Min | 25% | 50% | 75% | Max |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Rcpp (C++) | 26.33517 | 21.35722 | 22.16140 | 22.64436 | 27.30236 | 106.3577 |
| Pure R | 28.82316 | 23.95431 | 24.81522 | 25.28023 | 29.53291 | 118.5388 |

### Rules (1,000,000 observations)

For a 10-class vector of 1,000,000 observations:

* Vector A of length=(1000000 * 10)
* Vector B of length=(1000000) with 10 classes

```
A = [1:1, 1:2, 1:3, 1:4... 1:10, 2:1, 2:2, 2:3..., 1000000:8, 1000000:9, 1000000:10]
B = [3, 5, 9, 1, 4, 8, 6, ...]
```

Get the following Vector C:

```
C = [1:4, 2:6, 3:10, 4:2, 5:5, 6:9, 7:7, ...]
```

### Best code

`Lpp_vect2mat2vect(preds, labels)`:

* preds = your predictions (size = `length(labels) * number of different labels`)
* labels = your labels (starting from 0)

```r
Rcpp::cppFunction("NumericVector Lpp_vect2mat2vect(NumericVector preds, NumericVector labels) {
  int labels_size = labels.size();
  NumericVector selected(labels_size);
  selected = (preds.size() / labels_size) * seq(0, labels_size - 1);
  selected = selected + labels;
  NumericVector to_return(labels_size);
  to_return = preds[selected];
  return to_return;
}")
```

Correctness: **passing**.