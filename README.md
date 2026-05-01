# Substance Sleep-Impact Simulator: Technical Description, Model Assumptions, and Reference Basis

## Overview

The Substance Sleep-Impact Simulator is a lightweight, browser-based visualization tool for estimating the time course of sleep-relevant burden from selected substances. The current implementation supports caffeine, dextroamphetamine, lisdexamfetamine, methylphenidate formulations, mixed amphetamine salts extended-release, modafinil / armodafinil, and alcohol. The tool is intended to visualize the estimated consequences of user-entered use patterns: substance, formulation or source, dose, time of ingestion, bedtime, and wake time. It is not designed to recommend dose timing, dose size, or medication use. The intended function is explanatory and comparative: it allows the user to inspect how modeled exposure, pharmacologic effect, and sleep-disturbance burden evolve across the waking day and sleep window.

The app is implemented as a single static `index.html` file. It requires no backend, no database, no package manager, no build system, and no external JavaScript libraries. All calculations are performed locally in the browser. Scenario state is stored in browser `localStorage`, meaning entered data remains local to the user’s browser and is not transmitted to a server.

The live deployment is intended for GitHub Pages at:

https://noahlimits.github.io/stimulant-sleep-effects/

## Supported substances and order of presentation

The dropdown order reflects practical expected use rather than pharmacologic taxonomy. The current order is:

1. Caffeine
2. Dexedrine / dextroamphetamine sulfate
3. Vyvanse / lisdexamfetamine
4. Ritalin / methylphenidate immediate-release
5. Alcohol
6. Adderall XR / mixed amphetamine salts extended-release
7. Modafinil / armodafinil
8. Concerta / OROS methylphenidate
9. Ritalin LA
10. Biphentin
11. Foquest

Caffeine supports common source presets, including caffeine pill, espresso, double espresso, small coffee, medium coffee, large coffee, energy drink, and custom caffeine amount. Caffeine dose is entered in milligrams.

Alcohol supports shot / spirits, beer, and glass of wine. Alcohol dose is entered in standard drinks. One Canadian standard drink is treated as approximately 13.45 g of ethanol, consistent with Health Canada’s standard drink definition.

Prescription stimulant and wakefulness-agent entries are represented using common brand names or recognizable pharmacologic names paired with pharmacologic specificity, for example Dexedrine / dextroamphetamine sulfate, Vyvanse / lisdexamfetamine, Ritalin / methylphenidate hydrochloride, and Modafinil / armodafinil. The purpose of this naming convention is practical user recognition while preserving pharmacologic specificity.

## Core visualization concepts

The tool separates three related but non-identical concepts:

1. Exposure or plasma proxy
2. Pharmacologic effect proxy
3. Sleep-disturbance burden

This separation is central to the model. A substance may have residual modeled exposure while its modeled sleep-disturbance burden is relatively low. Conversely, alcohol may have a sleep-fragmentation burden that is not identical to its acute exposure curve. The app therefore should not be interpreted as a simple plasma-concentration graph. It is a sleep-impact visualization model with optional pharmacokinetic and effect overlays.

The graph contains several curve types.

Solid coloured curves represent modeled sleep-disturbance contribution for each selected substance. These are the primary curves in the application. They are normalized sleep-relevant burden scores, not measured plasma concentrations and not probabilities of insomnia.

The white curve represents combined modeled sleep disturbance. The combined curve uses a conservative overlap model. It generally tracks the strongest active contributor and adds a small bounded overlap penalty when multiple substances contribute simultaneously. It is intentionally not a simple arithmetic sum and not a multiplicative synergy model. This design prevents artificial “rocket ship” combined curves when two moderate contributors overlap.

Dotted curves, when enabled, represent modeled pharmacologic effect. These curves are informational. They are not used to determine the sleep threshold.

Dashed curves, when enabled, represent plasma level or exposure proxy. These are normalized approximations. They are not measured blood levels. For alcohol, this curve should be interpreted as a blood-alcohol exposure proxy, not as a formal BAC calculator.

The dashed horizontal threshold line represents a practical sleep-disturbance threshold. Values above this threshold are treated as more likely to be sleep-relevant; values below the threshold are treated as less likely to cause meaningful sleep disturbance. The threshold is a modeling construct, not a biological constant.

The graph includes dose-time markers, bedtime and wake-time markers, dynamic hover readouts, and a horizontal and vertical cursor guide. The vertical guide gives time. The horizontal guide gives modeled relative burden at the cursor position. The graph auto-scales above 100% when a visible modeled curve exceeds the default scale, allowing high-dose or high-burden scenarios to display without flattening or clipping.

## Normalized units and interpretation

The y-axis uses normalized model output. A value of 80% should not be interpreted as an 80% probability of insomnia. It means the modeled burden is 80% of the application’s reference scale for the relevant curve type. Similarly, a value above 100% does not imply biological impossibility. It indicates that the modeled burden exceeds the default reference scale and that the graph has expanded to display it.

This is particularly important for alcohol. A high standard-drink input may produce values above 100%, because the model is no longer constrained to flatten high-dose scenarios into a capped plateau. The graph dynamically rescales so that the curve remains interpretable.

## Time inputs and dose inputs

Time fields are editable manually and also support increment controls. The displayed hour and minute components can be edited directly. Up and down controls allow hour and minute adjustments. Bedtime, wake time, new dose time, and existing stack-item times use the same interaction pattern.

Dose fields use substance-appropriate increments. Alcohol increments in 0.5 standard drink units. Caffeine pill and custom caffeine inputs increment by 50 mg. Other caffeine presets increment by smaller practical amounts. Dexedrine IR increments by 2.5 mg. Dexedrine extended-release increments by 5 mg. Ritalin IR increments by 2.5 mg. Vyvanse increments by 10 mg. Modafinil and armodafinil increment by 50 mg. Other medications use a practical 5 mg increment. Prescription stimulant values are bounded to prevent nonsensical graph failure from extreme accidental input. Alcohol is not hard-capped because high drink counts may be deliberately modeled, but the graph rescales dynamically to accommodate high values.

The unit is displayed beside the dose label itself, for example Dose (mg) or Dose (std drinks). A separate unit field is intentionally omitted because it is redundant.

## Caffeine model

Caffeine is modeled as a rapidly absorbed substance with a relatively early peak and a long residual tail. The caffeine plasma proxy uses a simplified Bateman-style rise and exponential decay model anchored to common caffeine pharmacokinetic behavior: relatively rapid absorption, typical peak concentration around one to two hours, and a multi-hour elimination half-life with substantial individual variability.

The caffeine effect curve and sleep-disturbance curve are intentionally not identical. The modeled effect curve is intended to represent subjective or pharmacologic stimulant effect. The sleep-disturbance curve decays more slowly, reflecting the observation that caffeine can continue to impair sleep after subjective alertness has diminished. This distinction is important because a person may feel less caffeinated while residual adenosine antagonism remains sleep-relevant.

The caffeine-sensitive toggle lowers the effective sleep threshold and lengthens caffeine persistence in the model. This is a simplified way of representing interindividual sensitivity without introducing CYP1A2 genotype, smoking status, oral contraceptive status, pregnancy, liver function, habitual caffeine exposure, or other determinants of caffeine clearance and response.

Reference anchors for caffeine include controlled sleep studies showing that caffeine can impair sleep even when taken several hours before bedtime. One key reference is Drake et al., “Caffeine Effects on Sleep Taken 0, 3, or 6 Hours before Going to Bed,” Journal of Clinical Sleep Medicine, 2013.

Source:
https://pmc.ncbi.nlm.nih.gov/articles/PMC3805807/

## Amphetamine model

Dexedrine, Vyvanse, and Adderall XR are modeled using simplified amphetamine-family assumptions. Dexedrine IR is treated as immediate-release dextroamphetamine. Dexedrine Spansule and Adderall XR are modeled as multi-phase release curves. Vyvanse is modeled as a delayed prodrug-like exposure curve representing conversion of lisdexamfetamine to dextroamphetamine.

The app separates residual exposure, pharmacologic effect proxy, and sleep-disturbance burden. This means the dashed plasma proxy or dotted effect proxy may remain elevated when the solid sleep-burden curve has already fallen below the sleep threshold. This behavior is intentional. Dextroamphetamine may remain present in the body for many hours, while the modeled sleep-relevant burden declines because clinical effect, tolerance, time since ingestion, and modeled duration-of-effect attenuation are not identical to plasma persistence.

The dextroamphetamine assumptions are anchored to prescribing-label pharmacokinetic data, including time-to-peak and elimination half-life values reported in DailyMed labeling for dextroamphetamine-containing products. DailyMed labels commonly report dextroamphetamine time-to-peak on the order of several hours and half-life on the order of approximately 10 to 12 hours, with variation by product and population.

Sources:
https://dailymed.nlm.nih.gov/
https://dailymed.nlm.nih.gov/dailymed/

## Modafinil / armodafinil model

Modafinil and armodafinil are modeled as wakefulness-promoting agents with long residual persistence. The app treats both as standard oral tablet entries rather than inventing immediate-release or extended-release subtypes. Modafinil is represented with a peak around 2 to 4 hours and an effective half-life around 15 hours. Armodafinil is represented with an earlier peak around 2 hours and a persistent tail reflecting R-enantiomer exposure.

The sleep-disturbance curve for these agents is intentionally long-tailed. This reflects their pharmacokinetic persistence and wakefulness-promoting use case, while remaining a simplified visualization model rather than a prediction of objective sleep outcome.

Sources:
https://dailymed.nlm.nih.gov/
https://dailymed.nlm.nih.gov/dailymed/

## Methylphenidate model

Ritalin IR is modeled as immediate-release methylphenidate. Concerta, Ritalin LA, Biphentin, and Foquest are modeled using simplified multi-phase release approximations. These approximations are not intended to reproduce exact manufacturer dissolution curves, bead-release mechanics, or OROS delivery kinetics. They are practical timing approximations intended to visualize relative onset, peak window, persistence, and sleep-relevant burden.

Methylphenidate has a shorter plasma half-life than amphetamine, but extended-release formulations can produce long clinical coverage through delayed or staged release. The model therefore uses formulation-specific release approximations rather than treating all methylphenidate products as identical.

Reference anchors are drawn from DailyMed prescribing information for methylphenidate immediate-release and extended-release products.

Sources:
https://dailymed.nlm.nih.gov/
https://dailymed.nlm.nih.gov/dailymed/

## Alcohol model

Alcohol is modeled in standard drinks rather than milligrams. The app uses a smooth dose-duration model rather than an instantaneous bolus model. Larger drink counts produce a longer absorption and clearance-like curve, avoiding unrealistic vertical rises, flat plateaus, or sharp linear descents. The alcohol exposure proxy is not a legal or forensic BAC calculator. It is an exposure visualization.

Alcohol’s effect curve and plasma proxy are related but not identical. The pharmacologic effect curve is closer to acute intoxication or sedation and is allowed to fall off differently from the exposure proxy. The sleep-disturbance curve is different again. Alcohol can initially promote sleepiness or reduce sleep latency, but later in the night it can fragment sleep, alter sleep architecture, suppress or disrupt REM sleep, and increase wakefulness as alcohol is metabolized. The sleep-disturbance curve therefore includes both acute exposure burden and a later sleep-fragmentation component.

The alcohol model avoids representing this later component as a distinct secondary spike. Instead, the delayed burden is blended into a smoother sleep-disturbance curve. This is intended to match the qualitative sleep literature without overclaiming quantitative precision.

Reference anchors include Health Canada’s standard drink definition, general ethanol pharmacokinetic references, and sleep literature describing alcohol’s biphasic effects on sleep architecture.

Sources:
https://www.canada.ca/en/health-canada/services/substance-use/alcohol/low-risk-alcohol-drinking-guidelines.html
https://www.ncbi.nlm.nih.gov/books/
https://academic.oup.com/sleep/article/47/4/zsae003/7515846

## Combined-burden model

The combined curve is deliberately conservative. The model does not simply add all substance-specific sleep-burden curves together. Simple addition would overstate burden in many common cases, especially when two curves overlap modestly. The model also does not use multiplicative synergy, because that can produce visually dramatic and biologically unjustified spikes.

Instead, the combined curve is based on the strongest active contributor plus a bounded overlap term from secondary contributors. Conceptually:

combined burden = dominant contributor + small bounded overlap penalty

This allows the combined curve to be slightly higher than either individual curve, which is biologically plausible when two substances create overlapping sleep-relevant burden. For example, caffeine may impair sleep pressure via adenosine antagonism while alcohol may fragment sleep architecture later in the night. Their combined burden may be modestly higher than either alone. However, the combined curve should not greatly exceed both individual curves unless a specific interaction coefficient is later added and justified.

## Sleep threshold

The sleep threshold is a pragmatic display threshold rather than a validated physiological cutoff. The default automatic threshold is intended to distinguish low modeled burden from potentially meaningful sleep-relevant burden. Manual threshold options allow stricter or more permissive interpretation.

The threshold applies to the solid sleep-disturbance curves. It does not apply directly to the dotted effect curves or dashed plasma curves. A plasma proxy may remain above the visual threshold while the sleep-burden curve is below it. That should be interpreted as residual exposure without necessarily implying meaningful sleep disturbance in the model.

## Dynamic graph scaling

The graph defaults to a 0–100% scale. If visible curves exceed 100%, the y-axis expands to a larger rounded scale so all visible data can be displayed. Hidden overlays do not affect scaling. For example, if plasma curves are hidden, plasma values do not force the graph to rescale. If effect curves are hidden, effect values do not force the graph to rescale. The primary sleep-disturbance curves always affect scaling.

This design keeps ordinary scenarios readable while permitting high-dose or high-burden simulations to display normally. It avoids clipping, flat-topping, and the misleading appearance of a curve being artificially capped.

## Interface summary

The user selects a substance, formulation or source, dose, and time. Pressing Add places that item into the stack. Stack items can be edited directly. Dose fields update the graph when changed. Time fields can be edited manually or adjusted with hour and minute controls. Bedtime and wake time define the sleep window. The visible x-axis span slider controls how many hours are shown.

The graph displays dose markers, bedtime and wake markers, individual substance curves, the combined curve, the sleep-threshold line, and optional overlays. Hovering over the graph gives dynamic time and modeled-burden values. The vertical guide corresponds to time. The horizontal guide corresponds to the cursor’s y-position on the modeled burden scale.

## Technical implementation

The app is a single static HTML file containing HTML, CSS, and JavaScript. No external dependencies are required. The canvas graph is rendered manually using the browser Canvas API. The state object stores selected stack items, hover state, and cached normalization values. User settings and stack items are persisted in `localStorage`.

The model functions include concentration or exposure proxy functions, effect proxy functions, sleep-disturbance functions, a combined-burden function, graph-scaling logic, and rendering functions. The rendering pipeline builds a dense set of time points across the visible window, evaluates each substance at each point, computes combined burden, calculates y-axis scaling, and draws the graph.

## Principal limitations

The simulator does not currently individualize by body mass, age, sex, genotype, liver function, renal function, medication tolerance, caffeine tolerance, alcohol tolerance, habitual use, food effects, sleep debt, psychiatric state, baseline insomnia, circadian phase, or co-medications. It does not model CYP2D6 interactions, CYP1A2 interactions, urinary pH effects on amphetamine clearance, or other pharmacokinetic modifiers. It also does not ingest wearable sleep data or calibrate predictions to observed individual outcomes.

The app should not be interpreted as a medication dosing calculator, prescribing tool, medical device, BAC calculator, diagnostic instrument, or sleep-outcome predictor. It is a visualization model.

The most important interpretive limitation is that the curves are normalized model outputs. They are not literal blood concentrations, receptor occupancy estimates, EEG-derived arousal estimates, polysomnographic predictions, or probabilities of insomnia.

## Intended use cases

The app is best suited for visual questions such as:

How much modeled sleep burden remains at bedtime?

Which substance is contributing most to the bedtime burden?

When does the modeled burden fall below the sleep threshold?

How do caffeine, alcohol, and prescribed stimulant timing overlap?

How does the sleep window intersect with residual burden?

How does a high-dose scenario change the modeled time course?

How do plasma proxy and sleep-disturbance curves differ?

The tool is designed to make these temporal relationships visible rather than requiring the user to mentally integrate half-lives, formulation timing, sleep windows, and overlapping substances.

## Reference sources

Health Canada. Canada’s Guidance on Alcohol and Health / standard drink definition.
https://www.canada.ca/en/health-canada/services/substance-use/alcohol/low-risk-alcohol-drinking-guidelines.html

DailyMed. Prescription drug labeling database for dextroamphetamine, lisdexamfetamine, mixed amphetamine salts, methylphenidate immediate-release, and methylphenidate extended-release products.
https://dailymed.nlm.nih.gov/

Drake C, Roehrs T, Shambroom J, Roth T. Caffeine Effects on Sleep Taken 0, 3, or 6 Hours before Going to Bed. Journal of Clinical Sleep Medicine. 2013.
https://pmc.ncbi.nlm.nih.gov/articles/PMC3805807/

NCBI Bookshelf. Alcohol pharmacokinetic reference material.
https://www.ncbi.nlm.nih.gov/books/

SLEEP / Oxford Academic. Alcohol and sleep architecture reference material.
https://academic.oup.com/sleep/article/47/4/zsae003/7515846

## Deployment

The current deployment target is GitHub Pages.

Repository:
noahlimits/stimulant-sleep-effects

GitHub Pages source:
Branch: main
Folder: /root

Live URL:
https://noahlimits.github.io/stimulant-sleep-effects/

To update the app, replace the contents of `index.html` on the `main` branch and commit the change. GitHub Pages should rebuild automatically. If the browser continues to show the previous version, perform a hard refresh.

Suggested commit message examples:

Update graph scaling

Refine alcohol model

Improve time controls

Remove redundant unit field

Update README

## Development principle

The guiding principle of the app is that exposure, pharmacologic effect, and sleep disturbance are related but distinct. The simulator is useful precisely because it makes those distinctions visible. It allows residual exposure to remain visible without forcing the sleep-burden curve to remain high. It allows overlapping substances to increase total modeled burden without creating unjustified multiplicative spikes. It allows high-burden cases to scale visually without clipping or flattening. The result is a practical, transparent, browser-native model for reasoning about substance timing and sleep-relevant burden.
