# R_benchmarking

Benchmarking R and C++ for Machine Learning Metrics.

Code is provided here to copy & paste very quickly if needed to use immediately in R. Requires Rtools if using Windows.

Hardware / Software used:
* Intel i7-4600U
* Compilation flags for C/C++: -O2 -mtune=core2 (R’s defaults)
* Windows 8.1 64-bit
* R 3.3.2 + Intel MKL
* Rtools 34 + gcc 4.9

*Note: the metrics are tuned for speed. Algorithm wise, interpretability might be lost.<br>Which means if you were to explain, you will have issues.*

---

# Summary Benchmarks

Reported numbers are both for log10 weighted average (up) and peak performance (down):

* The multiplication factor `(R / Rcpp)` for the `Throughput+` (over 1 means Rcpp faster, lower than 1 means R faster)
* The throughput observations per second
* The peak vector size

| Benchmark | Throughput+ | Rcpp Throughput | R Throughput | Rcpp Peak | R Peak
| ------ | ---: | ---: | ---: | ---: | ---: |
| Binary Logarithmic Loss | log10 W Avg: 1.192x<br>Peak Avg: 1.164x | 18,244,619 obs/s<br>19,202,150 obs/s | 14,226,607 obs/s<br>16,502,080 obs/s | 1,000 | 10,000 |
| Multiclass Logarithmic Loss | log10 W Avg: 1.159x<br>Peak Avg: 1.166x | 15,510,039 obs/s<br>17,399,730 obs/s | 12,263,996 obs/s<br>14,918,040 obs/s | 10,000 | 10,000 |
| Area Under the Curve (ROC) | log10 W Avg: 3.141x<br>Peak Avg: 1.374x | 4,753,981 obs/s<br>9,287,828 obs/s | 3,378,689 obs/s<br>6,760,040 obs/s | 100 | 10,000 |
| Vector to Matrix to Vect | log10 W Avg: 1.165x<br>Peak Avg: 1.364x | 58,136,870 obs/s<br>86,983,400 obs/s | 43,211,259 obs/s<br>63,784,300 obs/s | 10,000 | 10,000 |

---

# Metric Benchmarks

---

## Binary Logartihmic Loss: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/logloss.nb.html)

### Performance

Reported numbers (from log10 weighted average) are:

* Rcpp is in average **19.211% faster** than R.
* Rcpp can process the function **2,628** times per hour (**18,244,619** processed observations per second).
* R can process the function **2,204** times per hour (**14,226,607** processed observations per second).
* Fastest functions only. Compiled with `-O2 -mtune=core2` flags (R's defaults).

Reported numbers (from the peaks) are:
* Rcpp function throughput peaks at **1,000** observations per call.
* R function throughput peaks at **10,000** observations per call.
* Rcpp is at peak throughput in average **16.362% faster** than R.
* Rcpp has an estimated maximum throughput of **19,202,150** observations per second.
* R has an estimated maximum throughput of **16,502,080** observations per second.

| Log10 | Samples | Throughput+ | Rcpp Time | Pure R Time | Rcpp Sampling | Pure R Sampling |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ~5.000 | log10 W.Avg. | **1.192x** | **1.370  s** | 1.633  s | **18.245 M/s** | 14.227 M/s |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| ~2.000 | 100 | **4.201x** | **7.003 μs** | 29.422 μs | **14.280 M/s** | 3.399 M/s |
| ~3.000 | 1,000 | **1.496x** | **52.078 μs** | 77.917 μs | **19.202 M/s** | 12.834 M/s |
| ~4.000 | 10,000 | **1.106x** | **547.915 μs** | 605.984 μs | **18.251 M/s** | 16.502 M/s |
| ~5.000 | 100,000 | **1.221x** | **5.342 ms** | 6.521 ms | **18.721 M/s** | 15.336 M/s |
| ~6.000 | 1,000,000 | **1.371x** | **53.538 ms** | 73.397 ms | **18.678 M/s** | 13.625 M/s |
| ~7.000 | 10,000,000 | **1.212x** | **549.245 ms** | 665.486 ms | **18.207 M/s** | 15.027 M/s |
| ~8.000 | 100,000,000 | **1.189x** | **5.469  s** | 6.504  s | **18.283 M/s** | 15.376 M/s |

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
cppFunction("double Lpp_logloss(NumericVector preds, NumericVector labels, double eps) {
  int label_size = labels.size();
  NumericVector clamped(label_size);
  clamped = clamp(eps, preds, 1 - eps);
  NumericVector loggy(label_size);
  loggy = -log((1 - labels) + ((2 * labels - 1) * clamped));
  double logloss = sum(loggy) / label_size;
  return logloss;
}")
```

---

## Multiclass Logarithmic Loss: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/mlogloss.nb.html)

### Performance

Reported numbers (from log10 weighted average) are:

* Rcpp is in average **15.864% faster** than R.
* Rcpp can process the function **18,232** times per hour (**15,510,039** processed observations per second).
* R can process the function **15,736** times per hour (**12,263,996** processed observations per second).
* Fastest functions only. Compiled with `-O2 -mtune=core2` flags (R's defaults).

Reported numbers (from the peaks) are:
* Rcpp function throughput peaks at **10,000** observations per call.
* R function throughput peaks at **10,000** observations per call.
* Rcpp is at peak throughput in average **16.635% faster** than R.
* Rcpp has an estimated maximum throughput of **17,399,730** observations per second.
* R has an estimated maximum throughput of **14,918,040** observations per second.

| Log10 | Samples | Throughput+ | Rcpp Time | Pure R Time | Rcpp Sampling | Pure R Sampling |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ~4.500 | log10 W.Avg. | **1.159x** | **197.456 ms** | 228.780 ms | **15.510 M/s** | 12.264 M/s |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| ~2.000 | 100 | **5.072x** | **7.184 μs** | 36.441 μs | **13.920 M/s** | 2.744 M/s |
| ~3.000 | 1,000 | **1.368x** | **64.723 μs** | 88.550 μs | **15.451 M/s** | 11.293 M/s |
| ~4.000 | 10,000 | **1.166x** | **574.722 μs** | 670.329 μs | **17.400 M/s** | 14.918 M/s |
| ~5.000 | 100,000 | **1.245x** | **6.154 ms** | 7.665 ms | **16.249 M/s** | 13.047 M/s |
| ~6.000 | 1,000,000 | **1.165x** | **63.798 ms** | 74.316 ms | **15.674 M/s** | 13.456 M/s |
| ~7.000 | 10,000,000 | **1.158x** | **702.179 ms** | 812.831 ms | **14.241 M/s** | 12.303 M/s |

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

---

## Area Under the Curve (ROC): [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/roc.nb.html)

### Performance

Reported numbers (from log10 weighted average) are:

* Rcpp is in average **214.134% faster** than R.
* Rcpp can process the function **3,494** times per hour (**4,753,981** processed observations per second).
* R can process the function **1,112** times per hour (**3,378,689** processed observations per second).
* Fastest functions only. Compiled with `-O2 -mtune=core2` flags (R's defaults).

Reported numbers (from the peaks) are:
* Rcpp function throughput peaks at **100** observations per call.
* R function throughput peaks at **10,000** observations per call.
* Rcpp is at peak throughput in average **37.393% faster** than R.
* Rcpp has an estimated maximum throughput of **9,287,828** observations per second.
* R has an estimated maximum throughput of **6,760,040** observations per second.

| Log10 | Samples | Throughput+ | Rcpp Time | Pure R Time | Rcpp Sampling | Pure R Sampling |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ~4.500 | log10 W.Avg. | **3.141x** | **1.030  s** | 3.237  s | **4.754 M/s** | 3.379 M/s |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| ~2.000 | 100 | **2.843x** | **10.767 μs** | 30.612 μs | **9.288 M/s** | 3.267 M/s |
| ~3.000 | 1,000 | **1.091x** | **138.855 μs** | 151.475 μs | **7.202 M/s** | 6.602 M/s |
| ~4.000 | 10,000 | 0.952x | 1.555 ms | **1.479 ms** | 6.432 M/s | **6.760 M/s** |
| ~5.000 | 100,000 | **1.099x** | **20.024 ms** | 22.008 ms | **4.994 M/s** | 4.544 M/s |
| ~6.000 | 1,000,000 | **1.995x** | **325.062 ms** | 648.615 ms | **3.076 M/s** | 1.542 M/s |
| ~7.000 | 10,000,000 | **3.237x** | **3.680  s** | 11.912  s | **2.717 K/s** | 0.839 K/s |

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
cppFunction("double Lpp_ROC(NumericVector preds, NumericVector labels) {
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

---

# Utilities Benchmarks

---

## Vector to Matrix to Vector: [benchmarks](https://cdn.rawgit.com/Laurae2/R_benchmarking/ffcc7175/vect2mat2vect.nb.html)

### Performance

Reported numbers (from log10 weighted average) are:

* Rcpp is in average **16.480% faster** than R.
* Rcpp can process the function **56,218** times per hour (**58,136,870** processed observations per second).
* R can process the function **48,264** times per hour (**43,211,259** processed observations per second).
* Fastest functions only. Compiled with `-O2 -mtune=core2` flags (R's defaults).

Reported numbers (from the peaks) are:
* Rcpp function throughput peaks at **10,000** observations per call.
* R function throughput peaks at **10,000** observations per call.
* Rcpp is at peak throughput in average **36.371% faster** than R.
* Rcpp has an estimated maximum throughput of **86,983,400** observations per second.
* R has an estimated maximum throughput of **63,784,300** observations per second.

| Log10 | Samples | Throughput+ | Rcpp Time | Pure R Time | Rcpp Sampling | Pure R Sampling |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ~4.500 | log10 W.Avg. | **1.165x** | **64.037 ms** | 74.590 ms | **58.137 M/s** | 43.211 M/s |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| ~2.000 | 100 | **1.973x** | **3.234 μs** | 6.380 μs | **30.923 M/s** | 15.674 M/s |
| ~3.000 | 1,000 | **2.408x** | **13.138 μs** | 31.636 μs | **76.116 M/s** | 31.610 M/s |
| ~4.000 | 10,000 | **1.364x** | **114.964 μs** | 156.778 μs | **86.983 M/s** | 63.784 M/s |
| ~5.000 | 100,000 | **1.187x** | **1.716 ms** | 2.037 ms | **58.260 M/s** | 49.082 M/s |
| ~6.000 | 1,000,000 | **1.208x** | **17.832 ms** | 21.546 ms | **56.078 M/s** | 46.412 M/s |
| ~7.000 | 10,000,000 | **1.162x** | **230.417 ms** | 267.676 ms | **43.400 M/s** | 37.359 M/s |

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