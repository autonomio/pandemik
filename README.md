# pandemik


<h1 align="center">
  <br>
  <a href="http://autonom.io"><img src="https://raw.githubusercontent.com/autonomio/ICUSIM/master/logo.png" alt="Pandemik" width="250"></a>
  <br>
</h1>

<h3 align="center">Behavior Change and Mitigation Simulation</h3>

<p align="center">
  <a href="#what">what?</a> •
  <a href="#why">why?</a> •
  <a href="#how">how?</a> •
  <a href="#start-simulating">start simulating</a> •
  <a href="https://autonom.io">About Autonomio</a> •
  <a href="https://github.com/autonomio/ICUSIM/issues">Issues</a> •
  <a href="#License">License</a>
</p>
<hr>
<p align="center">
Pandemik is an end-to-end pipeline for simulating behavior change and mitigation effects on **public health, economic, and psychological outcomes**.</p>
<hr>

### What?

Pandemik helps researchers and decision makers answer the question **_"which behavior changes and mitigations are most likely to yield favorable public health, economic, and psychological outcomes"_**.

Pandemik brings together gold standard behavior change, epidemic, and capacity modelling capabilities into an end-to-end simulation pipeline. The modelling pipeline connects behavior change and mitigation actions with public health, economic, and psychological outcomes. 

A [recent study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3561560) suggests that non-pharmaceutical interventions (NPIs) can lead to favorable economic outcomes when appropriately planned. 

### How? 

Pandemik consists of six stand-alone models, each representing a progression from behavior change and mitigation towards the final outcome - a visualized "map" of the relationship different behaviors have with favorable outcomes. Results are presented with sensitivities and confidence intervals. 

stage | name | focus | method 
--- | --- | --- | --- 
1 | country social model | models relevant socio-cultural at a country level | [Hofstede Model](https://www.hofstede-insights.com/product/compare-countries/)
2 | behavioral model | model effects of behaviors | restriction of behavior  | custom differential model
3 | epidemic model (SEIR) | from population to infection | [SEIR](http://www.public.asu.edu/~hnesse/classes/seir.html)
4 | hospitalization model | from infection to hospital | custom differential model
5 | ICU burden model | from hospital to ICU and release or death | [ICUSIM](https://github.com/autonomio/ICUSIM)
6 | output model | summarize results | sensitivity analysis

Each stage accepts inputs, sometimes from one of the preceding steps. Each step gives an output which used by one of the following steps. 

stage | name | input | output
--- | --- | --- | ---
1 | country social model | name of the country | weight
2 | behavioral model | restriction of behavior  | `beta` for SEIR, economic and psychological damage
3 | epidemic model (SEIR) | standard SEIR | number of infectious
4 | hospitalization model | number of infected | number of hospitalized
5 | ICU burden model | number of hospitalized | ICU demand and fatality
6 | output model | various | list of behaviors, economic, psychological, and health outcomes 

<hr>

### Why?

In April 2020, a significant portion of the world's population is adversely affected by pandemic mitigation actions. Public policy is optimized towards public health outcomes, at the expense of economics and psychological effects. Pandemik provides a highly accessible and scientifically robust way to optimize towards economic and psychological outcomes, without compromising public health benefits.

Example use-cases include:

- Plan for policy that optimize balance between public health, economic, and psychological outcomes
- Demonstrate the value of different behaviors and mitigations

<hr>

### How?

ICUSIM follows a straightforward logic:

- There is a certain number of patients to start with
- Patients are split between standard and ventilated ICU
- Patients can not move between standard and ventilated ICU
- New patients come in based on `doubles_in_days` input parameter
- As new patients come in, each is assigned with a probability to survive
- As new patients come in, each is assigned a stay duration
- Released or dead, it happens when stay duration is completed
- If there is less capacity than there is demand, patients will die accordingly

Outcomes are controlled through **Input Parameters**, which are provided separately for _standard ICU_ and _ventilated ICU_.

name | type | description
--- | --- | --- 
`initial_patient_count` | int | the number of patients to start with
`days_to_simulate` | int | number of days to simulate
`total_capacity_min` | int | minimum for total available capacity
`total_capacity_max` | int | maximum for total available capacity
`ventilated_icu_share_min` | float | minimum for ventilated capacity
`ventilated_icu_share_max` | float | maximum for ventilated capacity
`standard_cfr_min` | float | minimum case fatality rate for standard ICU
`standard_cfr_max` | float | maximum case fatality rate for standard ICU
`ventilated_cfr_min` | float | minimum case fatality rate for ventilated ICU
`ventilated_cfr_max` | float | maximum case fatality rate for ventilated ICU
`standard_duration_min` | float | minimum mean duration for standard ICU stay
`standard_duration_max` | float | maximum mean duration for standard ICU stay
`ventilated_duration_factor_min` | float | minimum ratio for ventilated capacity per standard standard 
`ventilated_duration_factor_max` | float | maximum ratio for ventilated capacity per standard standard 
`doubles_in_days_min` | float | minimum number of days it takes for exponental growth to happen 
`doubles_in_days_max` | float | maximum number of days it takes for eponental growth to happen
`ventilation_rate_min` | float | minimum rate at which ventilation is required
`ventilation_rate_max` | float | maximum rate at which ventilation is required
`show_params` | bool | prints out the parameters if True

<hr>

### 💾 Install

Released version:

#### `pip install icusim`

Daily development version:

#### `pip install git+https://github.com/autonomio/ICUSIM`

<hr>

### Start Simulating

To run a simulation, you need two things:

- parameter dictionary
- `icusim.MonteCarlo()` command

Make sure to follow parameter ranges that you can established with available empirical evidence. A fully functional example that is relevant for Finland is provided below. You can simply change the values to meet the evidence for the area/s of your interest.

```
params = {'initial_patient_count': 80,
          'days_to_simulate': 50,
          'total_capacity_min': 200,
          'total_capacity_max': 1000,
          'ventilated_icu_share_min': .4,
          'ventilated_icu_share_max': .6,
          'standard_cfr_min': 0.2,
          'standard_cfr_max': 0.6,
          'ventilated_cfr_min': 1.3,
          'ventilated_cfr_max': 1.7,
          'standard_duration_min': 8.5,
          'standard_duration_max': 25.5,
          'ventilated_duration_factor_min': .9,
          'ventilated_duration_factor_max': 1.1,
          'doubles_in_days_min': 2.0,
          'doubles_in_days_max': 12.0,
          'ventilation_rate_min': 0.3,
          'ventilation_rate_max': 0.8}
```
Next you can start the simulation: 

```
import icusim
results = icusim.MonteCarlo(rounds=1000, params)
```

Access the results of the simulation: 

```
results.df
```

If you want to also perform **sensitivity analysis**: 

```
import icusim
results = icusim.SobolSensitivity(rounds=1000, params)
```

Once the rounds are completed, get the sensitivities: 

```
results.sensitivity('metric_name')
```


You can also run a single round simulation with **daily output**: 

```
import icusim

params = icusim.params()
icusim.simulate(params)
```

Draw a **histogram** for analyzing the results:

```
astetik.hist(df, 'ventilated_icu_total_demand')
```
<hr>

### 💬 How to get Support

| I want to...                     | Go to...                                                  |
| -------------------------------- | ---------------------------------------------------------- |
| **...troubleshoot**           | [GitHub Issue Tracker]                   |
| **...report a bug**           | [GitHub Issue Tracker]                                     |
| **...suggest a new feature**  | [GitHub Issue Tracker]                                     |
| **...get support**            | [GitHub Issue Tracker]  · [Discord Chat]                         |
| **...have a discussion**      | [Discord Chat]                                            |

<hr>

### 📢 Citations

If you use ICUSIM for published work, please cite:

`Autonomio's ICUSIM [Computer software]. (2020). Retrieved from http://github.com/autonomio/ICUSIM.`

<hr>

### 📃 License

[MIT License](https://github.com/autonomio/talos/blob/master/LICENSE)

[github issue tracker]: https://github.com/automio/talos/issues

