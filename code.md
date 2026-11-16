---
layout: page
redirect_from: /labs.html
title: Code
---

Pricing code in C++ and Python. Everything here is also in the [LibraryCode](https://github.com/asharbutt/LibraryCode) repo.

* TOC
{:toc}

## C++

### Foundations

#### Options Parameter Struct

{: .entry-meta}
`cpp/include/options.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/options.hpp))

{% raw %}
```cpp
struct options {
    std::string tradeID;
    bool is_call;
    double notional = 0.0;
    double strike;
    std::string maturity;
    std::string dayCountConvention;

    virtual const double payoff(double spot) const = 0;
    virtual const std::vector<double> payoff(std::vector<double> spot) const = 0;
};

struct vanillaEquityOption: options {

    const double payoff(double spot) const {
        if (is_call){ return std::max(spot - strike, 0.0);}
        else {return std::max(strike - spot, 0.0);}
    }
    const std::vector<double> payoff(const std::vector<double>spot) const {
        std::vector<double> payoff_vector;
        payoff_vector.resize(spot.size());

        for (size_t i = 0; i < spot.size(); i++) {
            payoff_vector[i] = payoff(spot[i]);
           
        }
        return payoff_vector;
    }
};
```
{% endraw %}

#### Market Data Struct

{: .entry-meta}
`cpp/include/market_data.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/market_data.hpp))

{% raw %}
```cpp
struct market_data {
    std::string val_date;
    double spot;
    double repo_rate;
    double interest_rate;
    double div_yield;
};
```
{% endraw %}

#### Vol Fetcher Class

{: .entry-meta}
`cpp/include/vol_fetcher.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/vol_fetcher.hpp))

{% raw %}
```cpp
class vol_fetcher { // parent volatility class for passing a vol object to pricers
public:
    virtual const double retrieve_vol(double time, double spot) const = 0;
    virtual ~vol_fetcher() = default;
};

class fixed_vol : public vol_fetcher { // returns a fixed vol rather than anything
public:
    fixed_vol(double vol) : volatility(vol) {}
    const double retrieve_vol(double, double) const override {
        return volatility;
    }
private:
    double volatility;
};
```
{% endraw %}

#### Simple Matrix Class

Column-major, flat storage

<div markdown="0">
<p>Templated matrix stored in a single flat <code>std::vector</code> in column-major order, with <code>operator()</code> for (row, column) access. Also holds the simulation-parameter and result structs used by the Monte Carlo code.</p>
</div>

{: .entry-meta}
`cpp/include/Matrix.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/Matrix.hpp))

{% raw %}
```cpp
// If MATRIX_H is not defined, define it (start of header guard)
#ifndef MATRIX_H

// Define MATHUTILS_H to prevent multiple inclusions
#define MATRIX_H

#include <iostream>
#include <vector>
#include <stdexcept>

class Matrix {
public:
    Matrix(size_t i, size_t j) : rows_(i), columns_(j) { // create a matrix in vector format populated with 0
        matrix_.resize(i * j, 0.0);
    }
    double& operator() (std::size_t i, std::size_t j) { // The matrix is row major
            return matrix_[i * columns_ + j];
        }
    const double& operator()(std::size_t i, std::size_t j) const { // The matrix is row major
        return matrix_[i * columns_ + j];
    }
        size_t columns() const { return columns_; }
        size_t rows() const { return rows_; }
    Matrix operator+ (const Matrix& m) const {
        if (m.rows() != rows_ || m.columns() != columns_) {
            throw std::invalid_argument("Matrix dimensions do not match!");
        }
        else {
            Matrix sum(rows_, columns_);
            for (std::size_t i = 0; i < rows_; i++) {
                for (std::size_t j = 0; j < columns_; j++) {
                    sum(i, j) = (*this)(i, j) + m(i, j);
                }
            }
            return sum;
        }
    }
    Matrix operator*(const Matrix& m) const{
        if (columns_ != m.rows()) {
            throw std::invalid_argument("Matrix row number does not align with column numbers");
        }
        Matrix product(rows_, m.columns());
        for (std::size_t other_column = 0; other_column < m.columns(); other_column++) {
            for (std::size_t i = 0; i < rows_; i++) {
                for (std::size_t j = 0; j < columns_; j++) {
                    product(i, other_column) += (*this)(i, j) * m(j, other_column);
                }
            }

        }
        return product;
    }
    friend std::ostream& operator<<(std::ostream& os, Matrix& matrix) {
        for (std::size_t i = 0; i < matrix.rows(); i++) {
            for (std::size_t j = 0; j < matrix.columns(); j++) {
                os << matrix(i, j);
                if (j + 1 < matrix.columns()) os << ", ";
            }
            os << "\n";
        }
        return os;
    }
private:
    std::vector<double>matrix_;
    std::size_t rows_;
    std::size_t columns_;
};

/*
Sample code for how to use the matrix
int main()
{
    Matrix m(2, 2);
    int a = 0;
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            m(i, j) = a;
            a++;
        }
    }
    a = 0;
    Matrix d(2, 2);
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            d(i, j) = a;
            a++;
        }
    }
    Matrix C = d * m;

    std::cout << C;

}
*/

#endif
```
{% endraw %}

Planned: Statistics Header.

### Numerical Tools

#### Linear Interpolation

{: .entry-meta}
`cpp/include/interpolation.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/interpolation.hpp))

{% raw %}
```cpp
double linear_interp(double x1, const std::vector<double>& x, const std::vector<double>& y) { // assumes sorted vectors
    // boundary checks:
    if (x1<x[0] || x1 > x.back()) {
        throw std::invalid_argument("Input value is out of interpolation bounds");
    }
    else if (x1 == x.back()) {
        return y.back();
    }
    
    // first search for index where x1 should sit:
    int left = 0;
    for (int i = 0; i < x.size(); i++) {
        if (x1 < x[i]) {
            left = i-1;
            break;
        }
    }
    return y[left] + ((x1 - x[left]) * (y[left + 1] - y[left])) / (x[left + 1] - x[left]);
}
```
{% endraw %}

Planned: Normal Random Number Generator, Implied Volatility Solver, Cholesky Decomposition.

### Closed Form

#### Black–Scholes–Merton

European price and Greeks

<div markdown="0">
<p>Closed-form European price, delta, gamma, vega and theta with a continuous dividend yield. Uses the statistics and option-parameter headers.</p>
</div>

{: .entry-meta}
`cpp/include/black_scholes_merton.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/black_scholes_merton.hpp))

{% raw %}
```cpp
#include <cmath>      
#include "option_parameters.hpp"
#include "statistics.hpp"


double compute_d1(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) 
  { return (std::log(d.spot / c.strike) + (d.interestRate - d.divYield + 0.5 * d.volatility * d.volatility) * c.timeMaturity) / (d.volatility * std::sqrt(c.timeMaturity)); }
double price(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) {
    const double d1 = compute_d1(c, d);
    const double d2 = d1 - d.volatility * std::sqrt(c.timeMaturity);
    return c.optionType * d.spot * std::exp(-d.divYield * c.timeMaturity) * stats::standard_normal_cdf(c.optionType * d1) - c.optionType * c.strike * std::exp(-d.interestRate * c.timeMaturity) * stats::standard_normal_cdf(c.optionType * d2);
}
double delta(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) 
  { return c.optionType * std::exp(-d.divYield * c.timeMaturity) * stats::standard_normal_cdf(c.optionType * compute_d1(c, d)); }
double gamma(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) 
  { return stats::standard_normal_pdf(compute_d1(c, d)) / (d.spot * d.volatility * std::sqrt(c.timeMaturity)); }
double vega(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) 
  { return d.spot * std::exp(-d.divYield * c.timeMaturity) * stats::standard_normal_pdf(compute_d1(c, d)) * std::sqrt(c.timeMaturity); }
double theta(const option_parameters::OptionContract& c, const option_parameters::OptionsMarketData& d) {
    const double d1 = compute_d1(c, d);
    const double d2 = d1 - d.volatility * std::sqrt(c.timeMaturity);
    
    double annual_theta =  -((d.spot * stats::standard_normal_pdf(d1) * d.volatility*std::exp(-d.divYield*c.timeMaturity)) / (2 * std::sqrt(c.timeMaturity))) 
      - c.optionType* d.interestRate* c.strike*std::exp(-d.interestRate * c.timeMaturity)* stats::standard_normal_cdf(c.optionType * d2) 
      + c.optionType*d.divYield*d.spot*stats::standard_normal_cdf(c.optionType*d1)*std::exp(-d.divYield*c.timeMaturity);
        return annual_theta;
}
```
{% endraw %}

Planned: FX Forward, FX Options (Garman–Kohlhagen), Digital Options, Barrier Options, Geometric Asian Options, Quanto Options.

### Tree Methods

#### European Binomial Tree

Cox–Ross–Rubinstein

<div markdown="0">
<p>CRR tree: builds terminal stock prices, applies the payoff, then rolls back under the risk-neutral probability. Converges to Black&ndash;Scholes as the step count grows.</p>
</div>

{: .entry-meta}
`cpp/include/tree_methods.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/tree_methods.hpp))

{% raw %}
```cpp
#include <cmath>
#include <vector>
#include <algorithm>
#include "option_parameters.hpp"

double binomialTreeEuropean(option_parameters::binomial_parameters& params, option_parameters::OptionsMarketData& data, option_parameters::OptionContract& contract) {
    params.deltaT = contract.timeMaturity / params.numSteps;
    params.up = std::exp(data.volatility * std::sqrt(params.deltaT));
    params.down = std::exp(-data.volatility * std::sqrt(params.deltaT));
    params.probabilityUp = (std::exp(data.interestRate * params.deltaT) - params.down) / (params.up - params.down);

    std::vector<double> optionValue(params.numSteps+1, 0.0);
    std::vector<double> optionValue_temp(params.numSteps+1, 0.0);

    std::vector<double> finalStockVector(params.numSteps+1, params.numSteps+1);

    for (std::size_t i = 0; i < params.numSteps; i++) {
        finalStockVector[i] = data.spot * std::pow(params.up, params.numSteps - i) * std::pow(params.down, i);
    }

    //calculate payoff on initial vector
    for (std::size_t i = 0; i <= params.numSteps; i++) {
        optionValue[i] = std::max(contract.optionType * (finalStockVector[i] - contract.strike), 0.0);
    }

    for (std::size_t j = params.numSteps; j > 0; j--) {
        for (std::size_t i = 0; i < j; i++) {
            optionValue[i] = (optionValue[i] * params.probabilityUp + optionValue[i + 1] * (1 - params.probabilityUp)) * std::exp(-data.interestRate * params.deltaT);
        }
    }

    return optionValue[0];
}
```
{% endraw %}

#### American Binomial Tree

Early exercise at every node

<div markdown="0">
<p>Same lattice as the European tree, but at each node the value is the maximum of continuation and immediate exercise. Stores the full stock lattice in the Matrix class.</p>
</div>

{: .entry-meta}
`cpp/include/tree_methods.hpp` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/cpp/include/tree_methods.hpp))

{% raw %}
```cpp
#include "Matrix.hpp"
#include "option_parameters.hpp"
#include <vector>
#include <algorithm>
namespace american_binomial {
    double binomialTreeAmerican(option_parameters::binomial_parameters& params, option_parameters::OptionsMarketData& data, option_parameters::OptionContract& contract) {
    params.deltaT = contract.timeMaturity / params.numSteps;
    params.up = std::exp(data.volatility * std::sqrt(params.deltaT));
    params.down = std::exp(-data.volatility * std::sqrt(params.deltaT));
    params.probabilityUp = (std::exp(data.interestRate * params.deltaT) - params.down) / (params.up - params.down);
    std::vector<double> optionValue(params.numSteps + 1, 0.0); // initial payoff, will then be used to compute the option value at each Delta T
    std::vector<double> optionValue_temp(params.numSteps + 1, 0.0); // temporary option value vector, used to hold option values from previous time.
    Matrix finalStockMatrix(params.numSteps + 1, params.numSteps + 1);
    finalStockMatrix(0, 0) = data.spot;
    // initialise the stock matrix
    for (std::size_t j = 1; j <= params.numSteps; j++) {
        for (std::size_t i = 0; i < params.numSteps; i++) {
            if (i == 0) {
                finalStockMatrix(i, j) = finalStockMatrix(i, j - 1) * params.up;
            }
            else {
                finalStockMatrix(i, j) = finalStockMatrix(i - 1, j - 1) * params.down;
            }
        }
    }
    //calculate payoff on initial vector
    for (std::size_t i = 0; i <= params.numSteps; i++) {
        optionValue[i] = std::max(contract.optionType * (finalStockMatrix(i, params.numSteps) - contract.strike), 0.0);
    }
    for (std::size_t j = params.numSteps; j > 0; j--) {
        for (std::size_t i = 0; i < j; i++) {
            optionValue[i] = std::max((optionValue[i] * params.probabilityUp + optionValue[i + 1] * (1 - params.probabilityUp)) * std::exp(-data.interestRate * params.deltaT), std::max(contract.optionType*(finalStockMatrix(i,j-1)-contract.strike),0.0));
        }
    }
    return optionValue[0];
}
}
```
{% endraw %}

Planned: Trinomial Tree, Barrier Options on a Tree, Convertible Bond.

### Finite Difference

Planned: Explicit Finite Difference, Crank–Nicolson.

### Monte Carlo

Planned: Monte Carlo Engine, Arithmetic Asian Options, Barrier Options by Simulation, Longstaff–Schwartz, Autocallable.

## Python

### Foundations

#### Contract Class

Trade terms, values and risks

<div markdown="0">
<p>Dataclasses for contract details, with an option subclass. Passed into every Python pricer.</p>
</div>

{: .entry-meta}
`python/parameter_files/tradeParameters.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/parameter_files/tradeParameters.py))

{% raw %}
```python
from dataclasses import dataclass
from abc import ABC, abstractmethod
from typing import Optional
import numpy as np

@dataclass
class Trade:
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    dayCountConvention: str
    product_type = ""
    maturity = ""

    @abstractmethod
    def payoff(self, spot):
        ...

@dataclass(kw_only=True)
class vanillaEquityOption(Trade):
    tradeID: str
    is_call: bool # +1 for call, -1 for put
    notional: float
    strike: float
    maturity: str # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    product_type = "vanilla_equity_option"# single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        return np.maximum((spot - self.strike),0.0) if self.is_call else np.maximum(self.strike - spot, 0.0)

@dataclass(kw_only=True)
class digitalOption(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    fixed_payout: float

    strike: float
    maturity: str # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    product_type = "digital_option"# single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        indicator_vector = np.heaviside(spot - self.strike, 0) if self.is_call else  np.heaviside(self.strike - spot, 0)
        return indicator_vector*self.fixed_payout

@dataclass(kw_only=True)
class assetornothingOption(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    strike: float
    maturity: str # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    product_type = "digital_option"# single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        indicator_vector = np.heaviside(spot - self.strike, 0) if self.is_call else np.heaviside(self.strike - spot, 0)
        return indicator_vector * spot

@dataclass(kw_only=True)
class vanillaAmericanOption(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    strike: float
    maturity: str  # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    product_type = "vanilla_equity_option"  # single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        return np.maximum((spot - self.strike), 0.0) if self.is_call else np.maximum(self.strike - spot, 0.0)

@dataclass(kw_only=True)
class up_out_barrier_option(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    barrier: float
    barrier_type: str
    strike: float
    maturity: str  # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    product_type = "vanilla_equity_option"  # single_equity_product
    process_scheme = "exponential_CIR"

    def payoff(self, spot):
        return np.maximum((spot - self.strike), 0.0) if self.is_call else np.maximum(self.strike - spot, 0.0)

@dataclass(kw_only=True)
class geometricAsianOption(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    strike_type: str # either fixed or float
    strike: float
    maturity: str  # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    observation_type: str  # Continuous, daily etc
    observation_dates: []
    product_type = "vanilla_equity_option"  # single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        return np.maximum((spot - self.strike), 0.0) if self.is_call else np.maximum(self.strike - spot, 0.0)


@dataclass(kw_only=True)
class arithmeticAsianOption(Trade):
    tradeID: str
    is_call: bool  # +1 for call, -1 for put
    notional: float
    strike_type: str  # either fixed or float
    strike: float
    maturity: str  # this should be a date, we calculate the daycount on the fly using a daycount conventions module
    dayCountConvention: str
    observation_dates: []
    observation_type: str # Continuous, daily etc
    product_type = "vanilla_equity_option"  # single_equity_product
    process_scheme = "euler"

    def payoff(self, spot):
        return np.maximum((spot - self.strike), 0.0) if self.is_call else np.maximum(self.strike - spot, 0.0)
```
{% endraw %}

#### MarketData Class

Spot, rates, vol, dates

<div markdown="0">
<p>Market data dataclass passed to the pricing modules.</p>
</div>

{: .entry-meta}
`python/parameter_files/marketData.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/parameter_files/marketData.py))

{% raw %}
```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class market_data: # We maintain this class as optional, which is updated on the fly. In reality, i think we would assume that a market data module inputs the information into here live
    spot: Optional[float] = None
    repo_rate: Optional[float] = None
    interest_rate: Optional[float] = None
    div_yield: Optional[float] = None
    val_date: Optional[str] = None
```
{% endraw %}

#### Day Count Conventions

ACT/360, ACT/365

<div markdown="0">
<p>Returns the year fraction between two dates. Currently supports ACT/360 and ACT/365; 30/360 to follow.</p>
</div>

{: .entry-meta}
`python/parameter_files/daycount_convention.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/parameter_files/daycount_convention.py))

{% raw %}
```python
import datetime

def daycount(startDate, endDate, convention):
    startDate = datetime.datetime.strptime(startDate, "%d/%m/%Y")
    endDate = datetime.datetime.strptime(endDate, "%d/%m/%Y")
    if convention == "A360":
        return (endDate - startDate).days / 360
    elif convention == "A365":
        return (endDate - startDate).days / 365
    elif convention == "30360":
        return None
```
{% endraw %}

#### Vol Fetcher Class

Vol object to feed into pricers

<div markdown="0">
<p>Vol fetching object passed to pricer, can retrieve fixed vol or other models</p>
</div>

{: .entry-meta}
`python/parameter_files/vol_fetcher.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/parameter_files/vol_fetcher.py))

{% raw %}
```python
  import numpy as np
import tradeParameters as pd
from abc import ABC, abstractmethod

class vol_fetcher(ABC):
    @abstractmethod
    def retrieve_vol(self, spot, maturity):
        ...

class fixed_vol_fetcher(vol_fetcher):
    def __init__(self, vol):
        self.vol = vol
    def retrieve_vol(self, spot, maturity):
        return self.vol

class local_vol_fetcher(vol_fetcher):
    def __init__(self, implied_vol_matrix):
        self.implied_vol_matrix = implied_vol_matrix
    def retrieve_vol(self,spot, maturity):
        pass
```
{% endraw %}

### Numerical Tools

#### Implied Volatility (Newton–Raphson)

Solves BS price = market price

<div markdown="0">
<p>Newton&ndash;Raphson iteration for the implied vol of a European option, using vega as the derivative. Uses the Contract, MarketData and Black&ndash;Scholes modules.</p>
</div>

{: .entry-meta}
`python/numerical_tools/implied_vol_newton.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/numerical_tools/implied_vol_newton.py))

{% raw %}
```python
import black_scholes as bs
import contract_id_class as id
import marketData as md

def newton_raphson_implied_volatility(contract: id.OptionContract, data: md.MarketData, initial_guess: float ):
    # checks to ensure contract details are sufficient
    if any(x is None for x in (contract.contractID, contract.Notional, contract.Strike, contract.instrument, contract.contractValue, contract.dateMaturity)):
        raise ValueError("Missing parameters in Contract details")
    if any(x is None for x in (data.spot, data.interestRate, data.divYield, data.valDate, data.dayCountConvention)):
        raise ValueError("Missing parameters in marketData details")
    tol = 1e-5
    max_iterations = 10000
    x_0 = initial_guess
    data.vol = x_0
    print(data)
    for i in range(max_iterations):
        x_1 = x_0 - (bs.bs_price(contract, data) - contract.contractValue) / bs.vega(contract, data)
        data.vol = x_1
        if abs(bs.bs_price(contract, data) - contract.contractValue) < tol:
            return x_1
        x_0 = x_1
    return None
```
{% endraw %}

### Closed Form

#### Black–Scholes–Merton

European price and Greeks

<div markdown="0">
<p>Closed-form European price, delta, gamma, vega, theta and rho with a continuous dividend yield. Uses the Contract, MarketData and day-count modules, return struct with results</p>
</div>

{: .entry-meta}
`python/closed_form_pricers/black_scholes.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/closed_form_pricers/black_scholes.py))

{% raw %}
```python
from parameter_files import daycount_convention
from parameter_files.marketData import *
from parameter_files.tradeParameters import vanillaEquityOption
from parameter_files.daycount_convention import *
import numpy as np
from scipy.stats import norm
from parameter_files.vol_fetcher import vol_fetcher


def black_scholes(md: market_data, pd: vanillaEquityOption, vo: vol_fetcher):
    if pd.is_call == True:
        payoff_type = 1
    else:
        payoff_type = -1

    maturity = daycount(md.val_date, pd.maturity, pd.dayCountConvention)
    volatility = vo.retrieve_vol(md.spot, maturity)

    d1 = (np.log(md.spot / pd.strike) + (md.interest_rate - md.divYield + 0.5 * np.pow(volatility, 2)) * maturity)/ (volatility * np.sqrt(maturity))
    d2 = d1 - volatility * np.sqrt(maturity)

    option_value = payoff_type * md.spot * np.exp(-md.div_yield*maturity)*norm.cdf(payoff_type * d1) - payoff_type*pd.strike*np.exp(- md.interest_rate * maturity) *norm.cdf(payoff_type * d2)

    delta = payoff_type * np.exp(-md.div_yield*maturity)*norm.cdf(payoff_type * d1)
    vega = md.spot * np.exp(-md.div_yield*maturity)*norm.pdf(d1) * np.sqrt(maturity) / 100
    gamma = (np.exp(-md.div_yield*maturity)*norm.pdf(d1)) / (volatility * np.sqrt(maturity)) / 100

    theta = (
    -(md.spot * np.exp(-md.div_yield * maturity) * norm.pdf(d1) * volatility) / (2 * np.sqrt(maturity))
    - payoff_type * md.interest_rate * pd.strike
      * np.exp(-md.interest_rate * maturity) * norm.cdf(payoff_type * d2)
    + payoff_type * md.div_yield * md.spot
      * np.exp(-md.div_yield * maturity) * norm.cdf(payoff_type * d1)) / 365
    log_file_output = {
        "option_value": option_value,
        "d1": d1,
        "d2": d2,
        "Probability of ITM": norm.cdf(payoff_type * d2),
        "Delta": delta,
        "Vega %": vega,
        "Gamma per unit": gamma,
        "Theta per day" : theta,
        "volatility": volatility
    }
    return log_file_output
```
{% endraw %}

#### Digital Options

Cash- and asset-or-nothing

<div markdown="0">
<p>Cash-or-nothing $e^{-rT}N(d_2)$ and asset-or-nothing $S e^{-qT}N(d_1)$, with a call-spread replication check.</p>
</div>

{: .entry-meta}
`python/closed_form_pricers/digitals.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/closed_form_pricers/digitals.py))

{% raw %}
```python
from parameter_files import daycount_convention
from parameter_files.marketData import *
from parameter_files.tradeParameters import *
from parameter_files.daycount_convention import *
import numpy as np
from scipy.stats import norm
from parameter_files.vol_fetcher import vol_fetcher

def digital_cash_or_nothing(md: market_data, pd: digitalOption, vo: vol_fetcher):
    if pd.is_call == True:
        payoff_type = 1
    else:
        payoff_type = -1
    maturity = daycount(md.val_date, pd.maturity, pd.dayCountConvention)
    volatility = vo.retrieve_vol(md.spot, maturity)

    d1 = (np.log(md.spot / pd.strike) + (md.interest_rate - md.div_yield + 0.5 * np.pow(volatility, 2)) * maturity)/ (volatility * np.sqrt(maturity))
    d2 = d1 - volatility*np.sqrt(maturity)
    delta_digital = payoff_type*pd.fixed_payout*np.exp(-md.interest_rate * maturity) * norm.pdf(d2) / (md.spot*volatility*np.sqrt(maturity))
    gamma_digital = payoff_type*((-(pd.fixed_payout*d2*np.exp(-md.interest_rate * maturity)*norm.pdf(d2))
                     - pd.fixed_payout*np.exp(-md.interest_rate * maturity)*norm.pdf(d2)*volatility*np.sqrt(maturity))/
                     (volatility*volatility*maturity*md.spot*md.spot))
    vega_digital = -payoff_type * pd.fixed_payout * np.exp(-md.interest_rate * maturity) \
                   * norm.pdf(d2) * d1 / volatility
    option_value = payoff_type*pd.fixed_payout*np.exp(- md.interest_rate * maturity) *norm.cdf(payoff_type * d2)
    log_file_output = {
        "option_value": option_value,
        "delta_value": delta_digital,
        "gamma_value": gamma_digital,
        "vega_value": vega_digital
    }
    return log_file_output

def digital_asset_or_nothing(md: market_data, pd: digitalOption, vo: vol_fetcher):
    if pd.is_call == True:
        payoff_type = 1
    else:
        payoff_type = -1
    maturity = daycount(md.val_date, pd.maturity, pd.dayCountConvention)
    volatility = vo.retrieve_vol(md.spot, maturity)

    d1 = (np.log(md.spot / pd.strike) + (md.interest_rate - md.div_yield + 0.5 * np.pow(volatility, 2)) * maturity)/ (volatility * np.sqrt(maturity))

    option_value = payoff_type*np.exp(- md.div_yield * maturity) *norm.cdf(payoff_type * d1)*md.spot
    log_file_output = {
        "option_value": option_value
    }
    return log_file_output
```
{% endraw %}

Planned: FX Forward, FX Options (Garman–Kohlhagen), Barrier Options, Geometric Asian Options, Quanto Options.

### Tree Methods

#### Binomial Tree

European options, fixed vol

<div markdown="0">
<p>Binomial Tree pricer for European options, using fixed vol currently, with 4 input files: <code>marketData</code>, <code>tradeParameters</code>, <code>daycount_convention</code> and <code>vol_fetcher</code>.</p>
</div>

{: .entry-meta}
`python/tree_methods/binomial_tree.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/tree_methods/binomial_tree.py))

{% raw %}
```python
from parameter_files.marketData import *
from parameter_files.tradeParameters import *
from parameter_files.daycount_convention import *
from parameter_files.vol_fetcher import *
from abc import ABC, abstractmethod

class binomial_tree(ABC):
    @abstractmethod
    def price(self):
        ...
    @abstractmethod
    def calculate_delta(self):
        ...
    @abstractmethod
    def calculate_vega(self):
        ...
    @abstractmethod
    def generate_stock_prices(self):
        ...
    @abstractmethod
    def price_option(self):
        ...

class european_binomial_tree(binomial_tree):
    def __init__(self, data: market_data, params: Trade, num_steps: int, vol_object: vol_fetcher):
        self.data = data
        self.num_steps = num_steps
        self.maturity = daycount(data.val_date, params.maturity, params.dayCountConvention)
        self.params = params
        self.final_stock_vector = np.zeros(self.num_steps+1)
        self.up = np.exp(vol_object.retrieve_vol(self.params.strike, self.maturity)*(np.sqrt(daycount(data.val_date, params.maturity, params.dayCountConvention)/num_steps)))
        self.down = np.pow(self.up,-1)
        self.option_price_vector = np.zeros(self.num_steps)
        self.vol_fetcher = vol_object
        self.option_matrix_temp = []
        if self.data.repo_rate is not None:
            self.financing_rate = self.data.repo_rate
        else:
            self.financing_rate = self.data.interest_rate
        self.time_step = self.maturity / self.num_steps
        self.probability_up = (np.exp((self.financing_rate-self.data.div_yield)*self.time_step)-self.down)/(self.up - self.down)
        self.output_log = {}

        # bump and reprice parameters:
        self.dS = self.data.spot * 0.001 # bump by 10 basis points
        self.dVol = 0.01
        self.dDay = 1/365 # 1 day bump
    def price(self):
        self.generate_stock_prices()
        self.price_option()
        option_price = self.option_price_vector[0]
        self.output_log = {
            "Output Price": option_price,
            "Up move size": self.up,
            "Down move size": self.down,
            "Time step": self.time_step,
            "Time Step": self.time_step
        }
        self.calculate_delta()
        self.calculate_vega()

        return self.output_log
    def calculate_delta(self):
        original_spot = self.data.spot
        # bump and reprice for sensitivities:
        self.data.spot = original_spot + self.dS
        self.generate_stock_prices()
        self.price_option()
        bumped_up_price = self.option_price_vector[0]

        self.data.spot = original_spot - self.dS
        self.generate_stock_prices()
        self.price_option()
        bumped_down_price = self.option_price_vector[0]
        self.output_log["Delta"] = (bumped_up_price - bumped_down_price) / (2 * self.dS)

        self.data.spot = original_spot # revert the spot back to original data

        return None
    def calculate_vega(self):
        original_vol = self.vol_fetcher.retrieve_vol(self.params.strike, self.maturity)
        original_up = self.up
        original_down = self.down
        original_probability_up = self.probability_up

        self.up = np.exp((original_vol + self.dVol) * (
            np.sqrt(daycount(self.data.val_date, self.params.maturity, self.params.dayCountConvention) / self.num_steps)))
        self.down = np.pow(self.up, -1)
        self.probability_up = (np.exp((self.financing_rate-self.data.div_yield)*self.time_step) - self.down) / (self.up - self.down)
        # bump and reprice for sensitivities:
        self.generate_stock_prices()
        self.price_option()
        bumped_up_price = self.option_price_vector[0]

        self.up = np.exp((original_vol - self.dVol) * (
            np.sqrt(
                daycount(self.data.val_date, self.params.maturity, self.params.dayCountConvention) / self.num_steps)))
        self.down = np.pow(self.up, -1)
        self.probability_up = (np.exp((self.financing_rate-self.data.div_yield)*self.time_step) - self.down) / (self.up - self.down)
        self.generate_stock_prices()
        self.price_option()
        bumped_down_price = self.option_price_vector[0]
        self.output_log["Vega"] = (bumped_up_price - bumped_down_price) / (2 * self.dVol) * 0.01

        self.up =  original_up # revert the spot back to original data
        self.down = original_down
        self.probability_up = original_probability_up
        return None

    def price_option(self):
        self.option_price_vector_temp = self.params.payoff(self.final_stock_vector)

        for i in range(self.num_steps - 1, -1, -1):
            for j in range(i + 1):
                self.option_price_vector[j] = np.exp(-self.data.interest_rate * self.time_step) * (
                            (self.option_price_vector_temp[j] * self.probability_up) + (
                                self.option_price_vector_temp[j + 1] * (1 - self.probability_up)))
            self.option_price_vector_temp = self.option_price_vector

        return self.output_log

    def generate_stock_prices(self):
        for i in range(self.num_steps+1):
            self.final_stock_vector[i] = self.data.spot*np.pow(self.up,self.num_steps-i)*np.pow(self.down,i)

market_data_object = market_data(spot=100, interest_rate=0.05, div_yield=0.0,
                                        val_date="26/09/2026")
trade1_parameters = vanillaEquityOption(tradeID="trade1", is_call=True, notional=100, strike=100,
                                                     maturity="26/09/2027", dayCountConvention="A360")
vol_object = fixed_vol_fetcher(0.3)

a = european_binomial_tree(market_data_object, trade1_parameters, 1000, vol_object)
output = a.price()
print(output)
```
{% endraw %}

Planned: Trinomial Tree, Barrier Options on a Tree, Convertible Bond.

### Finite Difference

Planned: Crank–Nicolson.

### Monte Carlo

#### Estimating π by Monte Carlo

Convergence at $1/\sqrt{n}$

<div markdown="0">
<p>Estimates &pi; from the share of uniform points inside the unit circle, and plots the running estimate to show Monte Carlo convergence.</p>
</div>

{: .entry-meta}
`python/monte_carlo/estimate_pi.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/monte_carlo/estimate_pi.py))

{% raw %}
```python
import numpy as np
import matplotlib.pyplot as plt

def estimate_pi():
    n = 1000
    x = np.random.uniform(-1, 1, n)
    y = np.random.uniform(-1, 1, n)

    inside_circle = np.pow(x,2) + np.power(y,2) <= 1 # condition for being within a unit circle. pythagoras th

    total_inside = np.sum(inside_circle)

    pi_estimate = total_inside/n *4
    print(pi_estimate)

    cumulative_sum = np.cumsum(inside_circle)
    cumulative_sum_range = np.arange(1, n+1)
    rolling_estimate = cumulative_sum/cumulative_sum_range * 4


    fig, ax = plt.subplots(figsize=(6, 6))
    ax.scatter(x[inside_circle], y[inside_circle], s=3, color="#2a9d8f", label="inside")
    ax.scatter(x[~inside_circle], y[~inside_circle], s=3, color="#e76f51", label="outside")

    # Reference circle and square
    theta = np.linspace(0, 2 * np.pi, 400)
    ax.plot(np.cos(theta), np.sin(theta), color="black", lw=1.5)
    ax.add_patch(plt.Rectangle((-1, -1), 2, 2, fill=False, color="black", lw=1.5))

    ax.set_aspect("equal")
    ax.set_xlim(-1.05, 1.05)
    ax.set_ylim(-1.05, 1.05)
    ax.set_title(f"Monte Carlo $\\pi \\approx$ {pi_estimate:.4f}  (n={n:,})")
    ax.axhline(y=0, color="black",ls="--", lw=1.5)
    ax.axvline(x=0, color="black", ls="--",lw=1.5)
    ax.legend(loc="upper right", markerscale=3)
    plt.tight_layout()
    plt.savefig("mc_pi.png", dpi=120)

    fig_2, ax_2 = plt.subplots(figsize=(6, 6))

    ax_2.plot(cumulative_sum_range, rolling_estimate, color="#2a9d8f", lw=1.5)
    ax_2.axhline(np.pi, color="black", ls="--", lw=1.5)
    ax_2.set_title("Monte Carlo Estimate Convergence to true $\\pi$")
    ax_2.set_ylabel("Estimated $\\pi$")
    ax_2.set_xlabel("n")

    plt.show()
```
{% endraw %}

#### Monte Carlo Engine

GBM paths, European payoff

<div markdown="0">
<p>Vectorised GBM path generator with pluggable payoffs, returning price and standard error; antithetic variates. Checked against Black&ndash;Scholes.</p>
</div>

{: .entry-meta}
`python/monte_carlo/mc_european.py` ([view on GitHub](https://github.com/asharbutt/LibraryCode/blob/main/python/monte_carlo/mc_european.py))

{% raw %}
```python
from scipy.cluster.hierarchy import average

from parameter_files import marketData, tradeParameters
import parameter_files.marketData as market_data
from parameter_files.tradeParameters import *
import numpy as np
from parameter_files.daycount_convention import *
from scipy.stats import norm
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Optional
import closed_form_pricers.black_scholes as bs
import closed_form_pricers.digitals as dg
import sde_processes as sde

def simple_normal_random_generator(random_number_matrix):
    return norm.rvs(size=(random_number_matrix.shape[0], random_number_matrix.shape[1]))

class MonteCarloEngine:
    def __init__(self, num_steps, num_paths, data: market_data.market_data, params: Trade):
        self.num_steps = num_steps
        self.num_paths = num_paths
        self.data = data
        self.params = params
        self.equity_process = "gbm"

    def price_european_option(self):
        price = self.generate_paths()
        return np.average(price)

    def generate_paths(self):
        # first generate the random numbers needed. If we are running just a fixed volatility, or local volatility, then we only need a single set of random numbers for one Brownian Motion,  if Heston, then we need 2 sets of random numbers
        random_numbers = np.zeros((self.num_paths//2, self.num_steps+1))
        random_numbers = simple_normal_random_generator(random_numbers)
        maturity = daycount(self.data.valDate, self.params.maturity, self.params.dayCountConvention)
        if self.params.product_type == "vanilla_equity_option" or "digital_option":
            if self.equity_process == "gbm":
                paths = sde.geometric_brownian_motion(random_numbers, self.data.interestRate, self.data.volatility, self.data.spot, maturity/self.num_steps)
                return self.params.payoff(paths[:,-1])*np.exp(-self.data.interestRate*maturity)

def main():
    market_data_object = market_data.market_data(spot=100, vol_type="fixed_vol", interestRate=0.05, divYield=0.0,
                                        valDate="26/09/2026", volatility=0.3)
    trade1_parameters = vanillaEquityOption(tradeID="trade1", is_call=True, notional=100, strike=100,
                                                     maturity="26/09/2027", dayCountConvention="A360")
    trade2_parameters = digitalOption(tradeID="trade1", is_call=True, notional=100, strike=100,
                                            maturity="26/09/2027", dayCountConvention="A360", fixed_payout=10)
    a = MonteCarloEngine(1000, 20000, market_data_object, trade2_parameters)
    print(a.price_european_option())
    print(dg.digital_option_closed_form(market_data_object, trade2_parameters))

main()
```
{% endraw %}

Planned: Arithmetic Asian Options, Barrier Options by Simulation, Longstaff–Schwartz, Autocallable.
