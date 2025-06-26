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

## IV. Post SVI Optimization

The optimization process from the previous study, “Volatility Smile Optimization via SVI Model Utilizing NLP Algorithms,” conducted in the "SVI_NLP_Optimization" repository, was identically applied to the Post-SVI model for all maturity dates. 
- Gradient Descent Algorithm
- L-BFGS-B Algorithm

As a result, it is possible to compare the outcomes of the original raw SVI optimization with those of the Post-SVI optimization, as shown below. The results demonstrate that Post-SVI more effectively captures and reflects each individual volatility data point across all maturity dates.

<img width="904" alt="image" src="https://github.com/user-attachments/assets/547c087f-93bf-4aa3-86f5-c7bb2001e9e9" />


## V. Post SVI Option Pricing

### 5.1 Assumptions

To derive option prices using the volatility optimized via the Post-SVI model, the following three assumptions are introduced:

**1. Utilization of Dual Delta**  

$$
\Delta_T := \frac{\partial \mathfrak{p}}{\partial k}(K_T, T; s_0), \quad T > 0, \ \Delta_0 \in \{0, 1\}
$$

- **Dual delta**, defined as a function of the strike price rather than the underlying asset price, is adopted as a state variable.
- By doing so, the martingale property can be maintained without directly modeling the price dynamics of the underlying asset.
- The dual delta is assumed to follow a stochastic process diffusing within the interval [1].

**2. Convex Duality**  
- The Legendre transform of the existing option price is used to reformulate the option pricing problem in the dual delta space.
- The put option price is convex and differentiable with respect to $$ x $$, so its convex conjugate can be defined.
- Ultimately, the expression originally formulated in terms of $$ S $$ (the underlying asset price) is transformed into an expression involving the dual delta, thereby deriving a price formula as a function of the strike price.

**3. Dual Dupire Equation**  
- Under the risk-neutral measure, the price of a put option must equal the expected value of its payoff at maturity, satisfying the Dupire equation.
- The Dupire equation models option prices with respect to strike price and maturity, which is advantageous for analyzing option convexity and local volatility.
- However, for the equation to hold in terms of the dual delta, the Multiplicatively Separable Volatility (MSV) condition must be satisfied.

Ultimately, the goal is to transform the put option price function assumed by the conventional Dupire equation into an expression involving the dual delta, thereby obtaining an option pricing formula that preserves the martingale property without directly modeling the price movement of the underlying asset.

### 5.2 Put Option Pricing Formula

The derivation of the put option pricing formula starts from the Dupire equation;

$$
\mathfrak{p}(k, T; s_0) := \mathbb{E}^{\mathbb{Q}} \left[ (k - S_T)^+ \right]
$$

The Legendre transform is applied to convert the put option price function—originally expressed in terms of strike price and maturity—into a new form involving the dual delta and maturity. Under the assumption of a multiplicatively separable volatility (MSV) structure, this transformation enables the derivation of a put option pricing formula in terms of the dual delta.

$$
P(k, T; s_0) = b(k, T) \ln\left(1 + e^{\frac{k - s_0}{b(k, T)}}\right)
$$

Crucially, by defining the structure of the dual delta—which diffuses between 0 and 1—using concepts from Shannon entropy and the variance of a Bernoulli random variable, it becomes possible to obtain the put option formula without explicitly modeling the underlying asset price. In this framework, the dual delta’s uncertainty is characterized by the geometric mean of Shannon entropy and the variance of a Bernoulli random variable with success probability equal to the dual delta itself.

**Key Steps and Concepts:**
- Legendre Transform: This mathematical transformation allows the expression of a function in terms of new independent variables. In this context, the Legendre transform is used to convert the put option price function from the domain of strike price and maturity to that of dual delta and maturity. The transformed equation is the convex conjugate of the original, so the original function can be recovered via the inverse Legendre transform.
- Structure of Dual Delta: The dual delta is assumed to reflect both its own uncertainty and the variance of a Bernoulli random variable with success probability equal to the dual delta. Specifically, the structure function for dual delta is defined as the geometric mean of Shannon entropy and this Bernoulli variance.
- MSV Structure: To maintain the stochastic properties in the dual problem, a multiplicatively separable volatility structure is assumed. This ensures that the volatility coefficient can be factorized, allowing for consistent probabilistic behavior in the transformed space.

### 5.3 Call Option Pricing Formula

Following the derivation of the put option pricing formula, the corresponding call option price is obtained by leveraging the payoff relationship between put and call options. Specifically, the call option formula is derived using the standard put-call parity relationship, leading to a well-structured and practical pricing methodology.

$$
\max(s_0 - k, 0) = (s_0 - k) + \max(k - s_0, 0)
$$

$$
C(k, T; s_0) = b(k, T) \left( \frac{s_0 - k}{b(k, T)} + \pi\left( -\frac{s_0 - k}{b(k, T)} \right) \right)
$$

In conclusion, the proposed new option pricing formula is structured as follows:

- **Extension of Option Pricing Model**: By applying the Post-SVI model to the volatility function $$ b(k, T) $$, an extended option pricing model is introduced. This transforms the conventional approach—which centers on modeling the underlying asset price—into a strike-price-focused methodology. The new model defines option prices in the dual delta space, offering a novel and robust pricing framework.
- **Dynamic Volatility Modeling**: Unlike traditional models that assume static volatility, the Post-SVI model structures volatility as a function of time. This enables the pricing model to reflect the dynamic evolution of volatility over time, closely mirroring real-world market conditions.
- **Market Realism and Robustness**: The proposed methodology is designed to capture structural changes in market volatility, minimize the risk of option mispricing, and reduce the possibility of static arbitrage opportunities. As a result, it presents a sophisticated and realistic pricing approach that is well-suited to today’s complex financial markets.

## VI. Dynamic Delta Hedging

Dynamic delta hedging was conducted using the Post SVI option pricing formulas. The rebalancing frequency was set to daily, and an at-the-money option was selected for the analysis. 
For the put option, a long position was assumed; for the call option, a short position was assumed. Delta hedging was then performed under these conditions.

### 5.1 Put Option (Long)
The Post-SVI model provided better hedging results in terms of cost, mainly because it more accurately reflects extreme delta movements.

- **Post-SVI Model Superiority**: Delta hedging based on the Post-SVI model resulted in greater hedging profits compared to the Black-Scholes model.
- **Improved Delta Sensitivity**: The SVI-based delta more accurately captures extreme changes in delta, leading to better hedging outcomes. This is particularly evident when delta exhibits significant fluctuations.
- **Hedging Cost Dynamics**: For a long put position, it is typical to incur costs when purchasing the underlying asset for delta hedging. However, as the absolute value of delta decreases, the hedging process can actually generate profits. In this context, the Post-SVI model demonstrated superior performance.

<img width="898" alt="image" src="https://github.com/user-attachments/assets/e1aecee6-de9a-413f-ad1b-9054b0b03f17" />

Black-Scholes (Right), Post SVI(Left), K=5165

### 5.2 Call Option (Short)
Although the difference was marginal, the Post-SVI model resulted in lower hedging costs. It effectively minimizes unnecessary hedging and enhances risk management, demonstrating the superiority of the Post-SVI approach.

- **Post-SVI Model Superiority**: Delta hedging based on the Post-SVI model also yielded greater profits than the Black-Scholes model for a short call position.
- **Market Volatility Impact**: When delta increases, additional purchases of the underlying asset are required. In highly volatile markets, delta can change rapidly, increasing hedging frequency and costs.
- **Efficient Risk Management**: The Post-SVI model helps control unnecessary hedging and reduces overall costs, leading to more efficient risk management.

<img width="898" alt="image" src="https://github.com/user-attachments/assets/83e504c7-0f77-4962-b0fe-54b11de8af51" />

Black-Scholes (Right), Post SVI(Left), K=5235

By employing the Post-SVI model for dynamic delta hedging, both put long and call short positions benefited from more accurate delta estimation, especially during periods of extreme market movements. This approach reduces hedging costs and improves risk management compared to the traditional Black-Scholes framework. The results highlight the practical advantages of incorporating advanced volatility modeling into hedging strategies.



