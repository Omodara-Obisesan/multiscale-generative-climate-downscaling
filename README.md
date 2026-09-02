# Multiscale Generative Climate Downscaling

This project investigates whether explicit multiscale representations improve the reconstruction of fine-scale spatial variability in climate fields.

Using ERA5 daily maximum 2-m air temperature over the southwestern United States, I construct a controlled downscaling experiment in which the original
0.25° ERA5 field is treated as the high-resolution target and artificially coarsened by a factor of four to approximately 1°. Machine-learning models
are then used to reconstruct the missing fine-scale information. The project compares two ways of representing climate fields: pixel space and multiscale wavelet space, using both deterministic and generative models. The deterministic models consist of a residual CNN operating directly on the spatial grid and a CNN operating on multiscale wavelet coefficients. The generative models use conditional flow matching to reconstruct unresolved fine-scale temperature variability in both pixel and wavelet representations.

The models are evaluated using pointwise reconstruction error, spatial gradients, temperature extremes, and spatial power spectra.

## Research Questions

This project addresses two main questions:

1. Can machine-learning models recover fine-scale temperature structure more accurately than conventional bilinear interpolation?
2. Does explicitly representing spatial scales using wavelets improve deterministic and generative reconstruction compared with learning directly in pixel space?

## Data

The experiment uses ERA5 daily maximum 2-m air temperature over the southwestern United States.

- **Dataset:** ERA5 reanalysis
- **Variable:** Daily maximum 2-m air temperature
- **Period:** 1950–2024
- **Target grid:** 0.25°
- **Controlled coarse grid:** approximately 1°
- **Number of daily fields:** 27,394

  A temporal split is used to reduce information leakage between training and evaluation:

- **Training:** 1950–2005
- **Validation:** 2006–2014
- **Testing:** 2015–2024

Raw ERA5 NetCDF files are not included in this repository.

## Controlled Downscaling Experiment

The original 0.25° ERA5 temperature field is treated as the known high-resolution target.

Each field is coarsened by a factor of four, reducing the spatial grid from 48 × 60 to 12 × 15. The coarse field is then bilinearly interpolated back to the 48 × 60 target grid.

This creates a controlled experiment in which the fine-scale information removed during coarsening is known.

The models learn the residual

**R = X_target − X_bilinear**

and reconstruct the high-resolution field as

**X_reconstructed = X_bilinear + R_predicted**

This design allows the ability of each model to recover missing spatial structure to be evaluated directly against the original ERA5 field.

## Models

### Pixel CNN

A residual convolutional neural network provides a deterministic reconstruction directly in pixel space.

### Pixel Flow Matching

A conditional flow-matching model learns a generative distribution of residual temperature fields conditioned on the coarse-resolution temperature field.

Unlike the deterministic CNN, the flow model can generate multiple plausible realizations of unresolved fine-scale variability.

### Wavelet CNN

A two-level discrete wavelet transform is used to separate temperature variability across spatial scales and orientations.

A deterministic CNN predicts residual wavelet coefficients before reconstructing the temperature field through the inverse wavelet transform.

### Wavelet Flow Matching

Conditional flow matching is applied in the multiscale wavelet representation, allowing stochastic residual variability to be modeled separately across spatial scales.

## Evaluation

Model performance is evaluated across 200 held-out test days using several complementary metrics:

- Root Mean Square Error (RMSE)
- Spatial standard-deviation error
- Spatial-gradient error
- 95th-percentile temperature error
- 99th-percentile temperature error
- Two-dimensional spatial power spectra

The spectral evaluation is particularly important because pointwise error alone does not determine whether a model reproduces realistic spatial variability.

## Key Results

<img width="8854" height="5214" alt="Mutiscale_Wavelet_RepresentativeDay" src="https://github.com/user-attachments/assets/e6b0c14b-2590-4b36-ab17-ab88b2baabc9" />



| Method | RMSE | Std. Error | Gradient Error | P95 Error | P99 Error |
|---|---:|---:|---:|---:|---:|
| Bilinear | 0.1388 | 0.0444 | 0.0612 | 0.0750 | 0.0997 |
| Pixel CNN | 0.0873 | 0.0077 | 0.0212 | 0.0182 | 0.0240 |
| Wavelet CNN | **0.0486** | **0.0033** | **0.0068** | **0.0073** | **0.0102** |
| Pixel Flow (members) | 0.1373 | 0.0039 | 0.0046 | 0.0132 | 0.0174 |
| Wavelet Flow (members) | **0.0853** | **0.0021** | **0.0041** | **0.0096** | **0.0155** |

Flow metrics represent averages across individual stochastic ensemble members.

Metrics in this table are reported in normalized units. Lower values indicate better agreement with ERA5.

Relative to the corresponding pixel-space models:

- **Wavelet CNN reduced RMSE by approximately 44%.**
- **Wavelet Flow reduced RMSE by approximately 38%.**
- Wavelet CNN substantially improved spatial-gradient and extreme-temperature reconstruction.

## Scale-Dependent Fidelity

Spatial power spectra were calculated to determine whether the models reproduce temperature variability at the correct spatial scales.

<img width="3118" height="2043" alt="Pixel_vs_Wavelet_PowerSpectrum" src="https://github.com/user-attachments/assets/4d2767db-ceb9-44ad-a11a-b970b7b76c3b" />



For high spatial wavenumbers, representing the finest resolved spatial structure:

| Method | High-wavenumber power retained |
|---|---:|
| Bilinear | 66.8% |
| Pixel CNN | 78.9% |
| Wavelet CNN | 92.3% |
| Pixel Flow | 90.7% |
| Wavelet Flow | 106.6% |

The Wavelet CNN reproduces substantially more of the high-wavenumber variability present in ERA5 than either bilinear interpolation or the Pixel CNN.

The Wavelet Flow slightly overestimates high-wavenumber power, indicating that additional calibration of fine-scale stochastic variability is needed.

## Interpretation

The results suggest that explicitly organizing climate variability by spatial scale can improve reconstruction of fine-scale temperature structure.

The strongest spectral improvement occurs for the deterministic comparison, where the wavelet representation substantially improves both pointwise accuracy and high-wavenumber fidelity.

The generative models demonstrate a different capability: conditional flow matching can produce stochastic realizations of unresolved spatial variability rather than only a single deterministic estimate.

Together, these experiments provide a controlled proof of concept for combining multiscale representations with generative modeling for climate downscaling.

## Limitations

This project is a controlled methodological experiment rather than an operational climate-downscaling system.

Current limitations include:

- Coarse and high-resolution fields are derived from the same ERA5 dataset.
- Only daily maximum 2-m air temperature is considered.
- The experiment uses a regional Cartesian domain.
- Pixel and wavelet architectures are not parameter-matched.
- Generative evaluation uses a relatively small ensemble.
- Physical constraints are not explicitly imposed.

Because the pixel and wavelet architectures differ in parameter count, the current results should not be interpreted as proving that wavelet representation alone causes the performance improvement.

## Future Work

Future extensions include:

- Parameter-matched pixel and wavelet model comparisons
- Larger generative ensembles and probabilistic calibration
- Comparison with diffusion-based generative models
- Multivariate atmospheric fields
- Physics-constrained generative modeling
- Fluid-dynamics PDE benchmarks
- Spherical multiscale representations for global climate fields

## Tools

The project was developed in Python using:

- PyTorch
- xarray
- NumPy
- PyWavelets
- pandas
- Matplotlib
- NetCDF climate data

## Repository Structure

```text
multiscale-generative-climate-downscaling/
├── README.md
├── requirements.txt
├── notebooks/
├── figures/
├── results/
└── src/
