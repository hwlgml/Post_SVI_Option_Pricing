# Arbitrage-Free Option Pricing and Dynamic Delta Hedging via Post-SVI Model

## 0. Special Thanks
This study was inspired by research conducted based on _'A New Simple Approach for Constructing Implied Volatility Surfaces'_ by Peter Carr and Liuren Wu.

This study builds upon the previous research titled "Volatility Smile Optimization via SVI Model Utilizing NLP Algorithms" conducted in the original "SVI_NLP_Optimization" repository. It extends the work by incorporating option pricing and dynamic delta hedging methodologies.

## I. Research Background and Objectives

### 1.1 Importance of Volatility Estimation

In financial markets, volatility serves as a core metric for quantifying market risk. It plays a pivotal role across multiple domains including option pricing, risk management, and the construction of hedging strategies. Historical market shock events—such as the 2008 global financial crisis, the 2020 COVID-19 pandemic, and the sharp market downturn on August 5, 2024 ("Black Monday")—have all been characterized by abrupt surges in the VIX index. These surges signify increased market uncertainty and are often accompanied by price distortions and enhanced arbitrage opportunities. Therefore, accurate estimation of volatility is essential for ensuring market stability and efficiency.

The VIX index, calculated by the CBOE, reflects the market’s expectation of future 30-day volatility based on S&P 500 index option prices. It is widely regarded as a real-time measure of systemic risk and investor sentiment in the equity market.

<img width="736" alt="image" src="https://github.com/user-attachments/assets/1ba12cfe-37ef-47a3-99db-beb6349b6312" />

### 1.2 Limitations of Conventional Models

The widely-used Black-Scholes model assumes that volatility remains constant over time. However, this assumption fails to capture the abrupt changes and asymmetric nature of volatility observed in real-world financial markets. This structural limitation often leads to option mispricing, ultimately undermining the reliability of financial instruments and market integrity.

To address these shortcomings, the Stochastic Volatility Inspired (SVI) model was developed. The SVI framework allows implied volatility to be expressed as a nonlinear function, which makes it possible to more accurately model skewed and smile-shaped volatility patterns. This enables traders to formulate more realistic hedging strategies based on dynamic volatility surfaces. Nonetheless, even the SVI model falls short in fully capturing time-varying volatility and extreme market asymmetries, revealing the need for further refinement.

### 1.3 Research Objectives

This study aims to develop and implement an enhanced model, referred to as the Post-SVI model, which builds upon the SVI framework to establish an arbitrage-free pricing environment. The core objectives are to derive a new option pricing formula based on the Post-SVI model and to construct a dynamic delta hedging strategy that can adapt to real market conditions. By incorporating more realistic representations of volatility, the proposed model seeks to improve pricing accuracy and risk mitigation in options markets.

### 1.4 Key Problems Addressed

The Post-SVI model is designed to address two critical issues:

- **Volatility Asymmetry**: This phenomenon refers to the tendency for volatility to increase more sharply during market downturns. It is often attributed to the leverage effect and the volatility feedback mechanism. Traditional models based on symmetric and constant volatility distributions are inadequate for capturing such dynamics.

- **Option Mispricing**: This occurs when there is a discrepancy between theoretical model prices and actual market prices. Contributing factors include volatility clustering, irrational investor behavior, and insufficient trading volume or liquidity. Such mispricing not only leads to inefficient hedging strategies but also poses risks to market participants by conveying misleading signals.

Grounded in the above context, this research endeavors to overcome the structural limitations of existing volatility models through the development of the Post-SVI framework, thereby laying the foundation for more robust and realistic option pricing and trading strategies.


## II. Research Methodology - SVI Model

### 2.1 SVI(Stochastic Volatility Inspired) Model



The Stochastic Volatility Inspired (SVI) model was introduced by Jim Gatheral of Merrill Lynch in 1999. It is a simple parametric model used to explain the volatility smile observed in the options market. The SVI model expresses implied volatility as a function of the option’s time to maturity and the difference between the strike price and the underlying asset price. By fitting the volatility smile using five parameters, the model provides a smooth and flexible representation of implied volatility across different strikes and maturities. It accurately reflects the implied volatility surface depending on various strike prices and maturities.

The SVI formula is given by:

$$
w(k) = a + b \left( \rho (k - m) + \sqrt{(k - m)^2 + \sigma^2} \right)
$$

Where:
- $k$: Log-moneyness of the option (log-transformed strike price)
- $w(k)$: Total implied variance (provides the structure of the volatility surface)

It is defined using the following raw parameters:
- $a$: Minimum value of the curve (offset; corresponds to the lowest volatility)
- $b$: Slope of the curve (controls the steepness of the volatility smile)
- $\rho$: Correlation coefficient (indicates the skewness of the smile)
- $m$: Location of the minimum point of the curve (where the lowest volatility occurs)
- $\sigma$: Parameter determining the width of the curve (controls the scale of the volatility smile)
  
![image](https://github.com/user-attachments/assets/f62f9db3-37f3-4e5d-8ca1-6b2fa87eeed3)

### 2.2 Natural SVI
The **Natural SVI Parameterization** is a form of the Raw SVI model that emerges in the limiting case of the Heston model:

$$
\omega(k; x_N) = \Delta + \frac{\omega}{2} \left[ 1 + \xi \rho (k - u) + \sqrt{ \xi^2 (k - \mu)^2 + (1 - \rho^2) } \right]
$$

It is defined using the following raw parameters: 
- $\Delta = a - \frac{\omega}{2}(1 - \rho^2)$  
- $\omega = b$  
- $\mu = m$  
- $\xi = \frac{1}{\sigma}$

### 2.3 SVI Jump-Wings
The **SVI Jump-Wings Parameterization** is inspired by the approach proposed by Tim Klassen, designed to reflect changes in the implied volatility surface more realistically. This parameterization uses implied variance instead of total implied variance and explicitly incorporates time-to-expiration dependency. 
It uses the implied distribution around the ATM to express explicit term-dependence of volatility. It is defined using the following raw parameters:

$$
\nu_\tau = \frac{a + b(-\rho m + \sqrt{m^2 + \sigma^2})}{\tau}
$$

$$
\phi_\tau = \frac{1}{\sqrt{\omega_\tau}} \cdot \frac{b}{2} \left( \rho - \frac{m}{\sqrt{m^2 + \sigma^2}} \right)
$$

$$
p_\tau = \frac{1}{\sqrt{\omega_\tau}} b(1 - \rho)
$$

$$
c_\tau = \frac{1}{\sqrt{\omega_\tau}} b(1 + \rho)
$$

$$
\hat{\nu}_\tau = \frac{1}{\tau} \left( a + b \sigma \sqrt{1 - \rho^2} \right)
$$

- $\nu_\tau$: ATM term structure of implied variance
- $\phi_\tau$: ATM implied skew (skewness of implied volatility near ATM)
- $p_\tau$: Slope of the left wing of the volatility smile
- $c_\tau$: Slope of the right wing of the volatility smile
- $\hat{\nu}_\tau$: Minimum ATM implied variance

### 2.4. Post-SVI Model
This study ultimately proposes the **Post-SVI model**, an extended version of the classic Stochastic Volatility Inspired (SVI) model. While the original SVI model offers a flexible parametrization of the implied volatility smile, it does not adequately reflect sudden market movements such as volatility jumps. The Post-SVI model addresses this by incorporating three additional parameters — $J$, $\mu_j$, and $\theta$ — to represent structural distortions in the volatility surface.

The implied total variance function under the Post-SVI model is defined as:

$$
w(k, \tau) = \frac{1}{\tau} \left[ a + b \left( \rho(k - m) + \sqrt{(k - m)^2 + \sigma^2 + J e^{- \theta |k - \mu_j|}} \right) \right]
$$

#### New Parameters
The exponential term $J e^{- \theta |k - \mu_j|}$ reflects localized jump behavior, allowing the volatility smile to sharply bend or flatten in response to external shocks.

- $J$: Controls the magnitude of the jump in volatility.
- $\mu_j$: Represents the average location of the jump (typically defined in terms of moneyness).
- $\theta$: Controls the rate of exponential decay; higher values imply the jump effect is concentrated in a narrower range.


To ensure a smooth and arbitrage-free surface, the following integral transformation is applied:

$$
\psi(x) = \int_0^x \omega'(s)\,ds
$$

Here, $\psi(x)$ helps guarantee smooth curvature and prevents arbitrage opportunities such as butterfly spreads. 

Additionally, the Post-SVI framework enables the derivation of important characteristics of the implied volatility smile:

$$
\hat{\nu}_\tau = \frac{1}{\tau} \left( a + b \sigma \sqrt{1 - \rho^2} \right)
$$

Where:

- $\nu_\tau$: At-the-money (ATM) term structure of implied variance  
- $\hat{\nu}_\tau$: Minimum ATM implied variance  
- $\phi_\tau$: ATM implied skew (skewness near ATM)  
- $p_\tau$: Slope of the left wing of the volatility smile  
- $c_\tau$: Slope of the right wing of the volatility smile  

By incorporating these new parameters and features, the Post-SVI model improves the accuracy of implied volatility surface modeling, particularly under volatile market conditions. This allows for more robust pricing, hedging, and risk assessment in options markets.

## III. Research Methodology - Data
Two primary datasets were utilized in this study:
- S&P500 Options Data: Employed for parameter optimization of the SVI and Post-SVI models.
- KOSPI200 Options Data: Applied to implement the Post-SVI model in the domestic Korean financial market.

### 3.1 Data Preprocessing
A four-step preprocessing pipeline was executed:
1. Data Filtering: Isolated records with a trade volume ≥ 1.
2. Scaling: Normalized features for consistent analysis.
3. Segmentation: Partitioned data by (i) call/put options and (ii) maturity dates.
4. Volatility Smile Visualization: Mapped implied volatility patterns post-segmentation.

<img width="332" alt="image" src="https://github.com/user-attachments/assets/46c6adfc-7799-46e6-84be-3fda708db763" />

Despite the initial preprocessing, a significant amount of noise remained observable, and there were pronounced differences in trade volume across maturity dates. To address these issues, two additional preprocessing was conducted;
- IQR (Interquartile Range) method
- MAD (Median Absolute Deviation) method

As a result, the patterns observed above were improved compared to those before the additional preprocessing; however, residual noise still persists. We intend to resolve this remaining noise through optimization following the application of the SVI model.

<img width="333" alt="image" src="https://github.com/user-attachments/assets/0eeb55d1-2184-4d68-925f-e1739a2604f1" />

## IV. Post SVI

## V. Dynamic Delta Hedging


