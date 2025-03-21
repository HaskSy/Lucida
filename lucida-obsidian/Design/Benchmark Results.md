# Structs. General

everything here is struct, and they're declared in different packages from benchmark

## The very first version (not exactly)
| Case                             |   Median |         Err |   Err% |     Mean |     Ratio |
|:---------------------------------|---------:|------------:|-------:|---------:|----------:|
| nativeBaseline                   | 2.534 ns | ±0.00834 ns |  ±0.3% | 2.542 ns |      100% |
| opticsBaseline                   | 1.489 us |  ±0.0246 us |  ±1.7% | 1.635 us | +64409.5% |
| withoutMagic                     | 1.643 us |  ±0.0261 us |  ±1.6% | 1.776 us | +69999.4% |
| bestOptical                      | 176.5 ns |   ±3.097 ns |  ±1.8% | 186.4 ns |  +7256.9% |
| inlinedComposition               | 176.9 ns |   ±1.044 ns |  ±0.6% | 186.5 ns |  +7260.7% |
| inlinedCompositionDirectAccess   | 120.6 ns |   ±0.990 ns |  ±0.8% | 136.9 ns |  +5300.4% |
| inlinedCompositionLambdaIndirect | 40.35 ns |  ±0.0697 ns |  ±0.2% | 40.47 ns |  +1497.1% |
| inlinedCompositionLambda         | 26.44 ns |  ±0.0541 ns |  ±0.2% | 26.54 ns |   +947.1% |
| highOrderComposition             | 24.85 ns |  ±0.0599 ns |  ±0.2% | 24.91 ns |   +882.9% |
| bestOpticalAsLambda              | 40.35 ns |  ±0.0645 ns |  ±0.2% | 40.47 ns |  +1496.9% |
| bestOpticalAsFunction            | 25.25 ns |  ±0.0493 ns |  ±0.2% | 25.32 ns |   +899.3% |
| reconstruction                   | 24.91 ns |  ±0.0524 ns |  ±0.2% | 25.00 ns |   +886.3% |

- Storing pair of functions is painful
- Compiler won't inline lambdas, if they were stored in struct right in place and then taken out
- Magic doesn't affect runtime (color me surprised)
- Compiler is more or less capable of inlining static, top level and in-place defined lambdas

## Latest implementation (Simple)
| Case                             |   Median |       Err |   Err% |     Mean |    Ratio |
|:---------------------------------|---------:|----------:|-------:|---------:|---------:|
| nativeBaseline                   | 2.520 ns | ±0.778 ns | ±30.9% | 2.732 ns |     100% |
| opticsBaseline                   | 29.35 ns | ±1.093 ns |  ±3.7% | 30.00 ns | +1090.6% |
| bestFirstClassOptical            | 173.2 ns | ±22.35 ns | ±12.9% | 170.6 ns | +6671.4% |
| inlinedCompositionDirectAccess   | 144.9 ns | ±11.11 ns |  ±7.7% | 155.2 ns | +6060.8% |
| inlinedCompositionLambdaIndirect | 40.81 ns | ±0.729 ns |  ±1.8% | 40.39 ns | +1502.7% |
| inlinedCompositionLambda         | 27.09 ns | ±2.042 ns |  ±7.5% | 28.12 ns | +1016.0% |
| highOrderComposition             | 26.17 ns | ±1.438 ns |  ±5.5% | 29.71 ns | +1079.0% |
| bestOpticalAsLambda              | 45.80 ns | ±3.728 ns |  ±8.1% | 44.25 ns | +1656.1% |
| bestOpticalAsFunction            | 28.94 ns | ±0.656 ns |  ±2.3% | 28.02 ns | +1012.0% |
| reconstruction                   | 26.08 ns | ±0.786 ns |  ±3.0% | 26.13 ns |  +937.1% |

- Functions are much more inlinable, (can be) marked as @Frozen
- Without first-classness it looks fine

## Latest implementation (Van Laarhoven)
| Case                             |   Median |       Err |   Err% |     Mean |    Ratio |
|:---------------------------------|---------:|----------:|-------:|---------:|---------:|
| nativeBaseline                   | 2.685 ns | ±0.552 ns | ±20.6% | 3.004 ns |     100% |
| opticsBaseline                   | 86.83 ns | ±1.764 ns |  ±2.0% | 87.29 ns | +3151.3% |
| bestOptical                      | 140.0 ns | ±3.987 ns |  ±2.8% | 141.1 ns | +5154.8% |
| inlinedCompositionDirectAccess   | 142.4 ns | ±5.845 ns |  ±4.1% | 138.0 ns | +5039.5% |
| inlinedCompositionLambdaIndirect | 43.61 ns | ±1.353 ns |  ±3.1% | 42.34 ns | +1477.0% |
| inlinedCompositionLambda         | 28.23 ns | ±3.570 ns | ±12.6% | 27.83 ns |  +936.7% |
| highOrderComposition             | 25.10 ns | ±0.431 ns |  ±1.7% | 24.88 ns |  +826.6% |
| bestOpticalAsLambda              | 43.59 ns | ±0.365 ns |  ±0.8% | 43.32 ns | +1513.4% |
| bestOpticalAsFunction            | 25.19 ns | ±1.568 ns |  ±6.2% | 25.46 ns |  +848.3% |
| reconstruction                   | 28.42 ns | ±1.214 ns |  ±4.3% | 29.09 ns |  +983.6% |

- Van Laarhoven is slower on small scale in current implementation

# Depth Analysis (Same Package; Simple)

## Size 2
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 3.838 ns | ±1.271 ns | ±33.1% | 3.605 ns |    100% |
| highOrderComposition | 9.046 ns | ±1.090 ns | ±12.0% | 8.166 ns | +112.8% |
| reconstruction       | 2.856 ns | ±0.106 ns |  ±3.7% | 2.933 ns |  -23.6% |

- the wrapping { => ... }() won't inline for some reason
- reconstructing struct via constructor calls are faster if in same package

## Size 3

| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 52.15 ns | ±5.180 ns |  ±9.9% | 57.41 ns |    100% |
| highOrderComposition | 112.3 ns | ±12.36 ns | ±11.0% | 120.5 ns | +131.1% |
| reconstruction       | 12.59 ns | ±1.487 ns | ±11.8% | 10.43 ns |  -80.0% |

- Due to bad implementation of `nativeBaseline`, composition can still keep up

## Size 5
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 63.78 ns | ±3.923 ns |  ±6.1% | 63.90 ns |    100% |
| highOrderComposition | 319.2 ns | ±8.355 ns |  ±2.6% | 338.3 ns | +430.4% |
| reconstruction       | 14.97 ns | ±2.181 ns | ±14.6% | 16.20 ns |  -74.6% |

- Starting from ridiculous examples, it's slow

## Size 10
| Case                 |   Median |       Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|----------:|-------:|---------:|---------:|
| nativeBaseline       | 112.7 ns | ±12.87 ns | ±11.4% | 122.9 ns |     100% |
| highOrderComposition | 1.377 us | ±0.132 us |  ±9.6% | 1.413 us | +1154.1% |
| reconstruction       | 24.39 ns | ±0.373 ns |  ±1.5% | 24.37 ns |   -78.4% |

## Size 15
| Case                 |   Median |       Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|----------:|-------:|---------:|---------:|
| nativeBaseline       | 147.1 ns | ±5.821 ns |  ±4.0% | 141.3 ns |     100% |
| highOrderComposition | 2.967 us | ±0.152 us |  ±5.1% | 3.087 us | +1998.4% |
| reconstruction       | 32.76 ns | ±0.717 ns |  ±2.2% | 31.78 ns |   -78.4% |

- Funny note, at size 26 reconstruction is slover than native baseline

# Depth Analysis (Same Package; Van Laarhoven; flatten compose)

## Size 2
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 2.616 ns |  ±0.250 ns |  ±9.6% | 3.068 ns |    100% |
| highOrderComposition | 8.553 ns |  ±0.151 ns |  ±1.8% | 8.329 ns | +218.4% |
| reconstruction       | 2.684 ns | ±0.0252 ns |  ±0.9% | 2.728 ns |   +4.3% |

- still won't inline { => ... }()

## Size 3
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 63.95 ns | ±1.854 ns |  ±2.9% | 65.54 ns |    100% |
| highOrderComposition | 134.2 ns | ±7.874 ns |  ±5.9% | 142.2 ns | +122.3% |
| reconstruction       | 13.14 ns | ±0.412 ns |  ±3.1% | 12.61 ns |  -80.3% |

## Size 5
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 65.21 ns |  ±0.718 ns |  ±1.1% | 65.12 ns |    100% |
| highOrderComposition | 0.534 us | ±0.0158 us |  ±3.0% | 0.518 us | +694.0% |
| reconstruction       | 15.55 ns |  ±0.775 ns |  ±5.0% | 15.49 ns |  -76.2% |

## Size 10
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 106.9 ns | ±4.338 ns |  ±4.1% | 112.9 ns |    100% |
| highOrderComposition | 1.163 us | ±0.108 us |  ±9.3% | 1.152 us | +977.8% |
| reconstruction       | 23.68 ns | ±2.054 ns |  ±8.7% | 24.55 ns |  -77.0% |
    
## Size 15
| Case                 |   Median |        Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|---------:|
| nativeBaseline       | 135.9 ns |  ±6.503 ns |  ±4.8% | 132.8 ns |     100% |
| highOrderComposition | 1.961 us | ±0.0740 us |  ±3.8% | 2.057 us | +1413.6% |
| reconstruction       | 35.61 ns |  ±5.246 ns | ±14.7% | 41.56 ns |   -69.4% |

- on ridiculous scales van laarhoven is faster

# Depth Analysis (Separate Package; Simple)

## Size 2
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 2.558 ns | ±0.0152 ns |  ±0.6% | 2.536 ns |    100% |
| highOrderComposition | 27.95 ns | ±0.0568 ns |  ±0.2% | 28.00 ns | +994.5% |
| reconstruction       | 2.530 ns | ±0.0106 ns |  ±0.4% | 2.535 ns |   -0.9% |

- didn't inline for some reason
- `nativeBaseline` and `reconstruction` compiled to same assembler

## Size 3
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 48.71 ns |  ±0.411 ns |  ±0.8% | 49.00 ns |    100% |
| highOrderComposition | 130.2 ns |  ±0.814 ns |  ±0.6% | 131.2 ns | +169.4% |
| reconstruction       | 12.18 ns | ±0.0258 ns |  ±0.2% | 12.22 ns |  -74.9% |

- now structure constructors did inline... the hell...

## Size 5
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| nativeBaseline       | 62.61 ns | ±2.170 ns |  ±3.5% | 64.23 ns |    100% |
| highOrderComposition | 333.9 ns | ±20.93 ns |  ±6.3% | 350.3 ns | +459.5% |
| reconstruction       | 14.84 ns | ±1.255 ns |  ±8.5% | 15.32 ns |  -75.5% |
## Size 10
| Case                 |   Median |         Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|------------:|-------:|---------:|---------:|
| nativeBaseline       | 104.1 ns |   ±0.411 ns |  ±0.4% | 104.6 ns |     100% |
| highOrderComposition | 1.317 us | ±0.00832 us |  ±0.6% | 1.358 us | +1204.1% |
| reconstruction       | 22.53 ns |  ±0.0415 ns |  ±0.2% | 22.60 ns |   -78.3% |
## Size 15
| Case                 |   Median |        Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|---------:|
| nativeBaseline       | 132.6 ns |  ±15.09 ns | ±11.4% | 142.0 ns |     100% |
| highOrderComposition | 3.030 us | ±0.0587 us |  ±1.9% | 3.003 us | +2165.5% |
| reconstruction       | 33.17 ns | ±0.0917 ns |  ±0.3% | 33.25 ns |   -74.9% |

- again ridiculous scales

# Depth Analysis (Separate Package; Van Laarhoven; flatten)

## Size 2
| Case                 |   Median |         Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|------------:|-------:|---------:|---------:|
| nativeBaseline       | 2.535 ns | ±0.00750 ns |  ±0.3% | 2.544 ns |     100% |
| highOrderComposition | 83.21 ns |   ±1.672 ns |  ±2.0% | 87.65 ns | +3358.0% |
| reconstruction       | 2.534 ns |  ±0.0104 ns |  ±0.4% | 2.544 ns |    +0.4% |

- van laarhoven is hard to inline

## Size 3
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 48.55 ns |  ±0.124 ns |  ±0.3% | 48.69 ns |    100% |
| highOrderComposition | 234.9 ns |  ±4.966 ns |  ±2.1% | 248.0 ns | +410.8% |
| reconstruction       | 12.16 ns | ±0.0206 ns |  ±0.2% | 12.19 ns |  -74.9% |
## Size 5
TCS: StructMutBenchmark, time elapsed: 13028886841 ns, RESULT:
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| nativeBaseline       | 63.08 ns |  ±0.130 ns |  ±0.2% | 63.21 ns |    100% |
| highOrderComposition | 375.8 ns |  ±4.935 ns |  ±1.3% | 408.2 ns | +547.0% |
| reconstruction       | 14.90 ns | ±0.0387 ns |  ±0.3% | 14.90 ns |  -76.4% |
    
## Size 10
| Case                 |   Median |         Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|------------:|-------:|---------:|--------:|
| nativeBaseline       | 106.1 ns |   ±0.215 ns |  ±0.2% | 106.3 ns |    100% |
| highOrderComposition | 1.032 us | ±0.00937 us |  ±0.9% | 1.126 us | +961.0% |
| reconstruction       | 22.59 ns |  ±0.0399 ns |  ±0.2% | 22.65 ns |  -78.7% |
    
## Size 15
| Case                 |   Median |         Err |   Err% |     Mean |    Ratio |
|:---------------------|---------:|------------:|-------:|---------:|---------:|
| nativeBaseline       | 144.3 ns |   ±0.346 ns |  ±0.2% | 144.7 ns |     100% |
| highOrderComposition | 2.082 us | ±0.00904 us |  ±0.4% | 2.085 us | +1344.4% |
| reconstruction       | 31.21 ns |   ±0.746 ns |  ±2.4% | 31.72 ns |   -78.0% |

# Classes.

Now, every object is a class. Baseline is changed to reconstruction. Native baseline is removed

# Depth. (Separate Package; Simple)

## Size 2
| Case                 |   Median |       Err |   Err% |     Mean |  Ratio |
|:---------------------|---------:|----------:|-------:|---------:|-------:|
| reconstruction       | 34.04 ns | ±0.172 ns |  ±0.5% | 34.24 ns |   100% |
| highOrderComposition | 38.03 ns | ±0.290 ns |  ±0.8% | 38.78 ns | +13.9% |

- classes look good

## Size 3
| Case                 |   Median |       Err |   Err% |     Mean |  Ratio |
|:---------------------|---------:|----------:|-------:|---------:|-------:|
| reconstruction       | 47.25 ns | ±0.437 ns |  ±0.9% | 52.87 ns |   100% |
| highOrderComposition | 54.87 ns | ±0.576 ns |  ±1.1% | 62.93 ns | +33.2% |

- i would say very good
## Size 5
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| reconstruction       | 83.56 ns | ±0.562 ns |  ±0.7% | 84.84 ns |    100% |
| highOrderComposition | 216.8 ns | ±1.217 ns |  ±0.6% | 217.5 ns | +160.2% |
    
## Size 10
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| reconstruction       | 188.1 ns |  ±2.603 ns |  ±1.4% | 188.5 ns |    100% |
| highOrderComposition | 0.905 us | ±0.0134 us |  ±1.5% | 1.013 us | +438.8% |
    
## Size 15
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| reconstruction       | 316.5 ns |  ±4.690 ns |  ±1.5% | 315.8 ns |    100% |
| highOrderComposition | 2.306 us | ±0.0232 us |  ±1.0% | 2.421 us | +664.9% |

# Depth. (Separate Package; Van Laarhoven)

## Size 2
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| reconstruction       | 31.43 ns | ±0.303 ns |  ±1.0% | 34.31 ns |    100% |
| highOrderComposition | 79.12 ns | ±0.711 ns |  ±0.9% | 86.86 ns | +176.4% |
## Size 3
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| reconstruction       | 48.88 ns | ±0.438 ns |  ±0.9% | 52.68 ns |    100% |
| highOrderComposition | 127.8 ns | ±1.245 ns |  ±1.0% | 146.1 ns | +198.9% |
## Size 5
| Case                 |   Median |       Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|----------:|-------:|---------:|--------:|
| reconstruction       | 84.03 ns | ±1.885 ns |  ±2.2% | 85.92 ns |    100% |
| highOrderComposition | 308.2 ns | ±1.810 ns |  ±0.6% | 309.2 ns | +268.0% |
## Size 10
| Case                 |   Median |         Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|------------:|-------:|---------:|--------:|
| reconstruction       | 183.3 ns |   ±0.994 ns |  ±0.5% | 187.0 ns |    100% |
| highOrderComposition | 0.964 us | ±0.00449 us |  ±0.5% | 0.969 us | +428.6% |
## Size 15
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| reconstruction       | 268.5 ns |  ±3.234 ns |  ±1.2% | 310.3 ns |    100% |
| highOrderComposition | 1.481 us | ±0.0136 us |  ±0.9% | 1.671 us | +522.3% |

# Depth. (Same Package; Simple)

## Size 2
| Case                 |   Median |       Err |   Err% |     Mean |  Ratio |
|:---------------------|---------:|----------:|-------:|---------:|-------:|
| reconstruction       | 31.20 ns | ±0.568 ns |  ±1.8% | 30.85 ns |   100% |
| highOrderComposition | 32.80 ns | ±1.488 ns |  ±4.5% | 31.00 ns |  -0.6% |

- now { => ... }() did inline! WHYYY?!!

## Size 3
| Case                 |   Median |       Err |   Err% |     Mean |  Ratio |
|:---------------------|---------:|----------:|-------:|---------:|-------:|
| reconstruction       | 49.07 ns | ±5.427 ns | ±11.1% | 53.83 ns |   100% |
| highOrderComposition | 49.24 ns | ±2.168 ns |  ±4.4% | 50.47 ns |  +2.9% |

## Size 5
| Case                 |   Median |       Err |   Err% |     Mean |  Ratio |
|:---------------------|---------:|----------:|-------:|---------:|-------:|
| reconstruction       | 79.54 ns | ±4.789 ns |  ±6.0% | 84.55 ns |   100% |
| highOrderComposition | 94.59 ns | ±9.626 ns | ±10.2% | 90.55 ns | +13.8% |

## Size 10
| Case                 |   Median |        Err |   Err% |     Mean |   Ratio |
|:---------------------|---------:|-----------:|-------:|---------:|--------:|
| reconstruction       | 141.5 ns |  ±8.818 ns |  ±6.2% | 137.3 ns |    100% |
| highOrderComposition | 0.906 us | ±0.0128 us |  ±1.4% | 0.917 us | +548.1% |

## Size 15
| Case                 |   Median |        Err |  Err% |     Mean |   Ratio |
| :------------------- | -------: | ---------: | ----: | -------: | ------: |
| reconstruction       | 226.4 ns |  ±13.74 ns | ±6.1% | 215.5 ns |    100% |
| highOrderComposition | 2.229 us | ±0.0762 us | ±3.4% | 2.122 us | +837.3% |
