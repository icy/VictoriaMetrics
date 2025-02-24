---
weight: 5
title: CHANGELOG
menu:
  docs:
    identifier: "vmanomaly-changelog"
    parent: "anomaly-detection"
    weight: 5
aliases:
- /anomaly-detection/CHANGELOG.html
---
Please find the changelog for VictoriaMetrics Anomaly Detection below.

> **Important note: Users are strongly encouraged to upgrade to `vmanomaly` [v1.9.2](https://hub.docker.com/repository/docker/victoriametrics/vmanomaly/tags?page=1&ordering=name) or newer for optimal performance and accuracy. <br><br> This recommendation is crucial for configurations with a low `infer_every` parameter [in your scheduler](./components/scheduler.md#parameters-1), and in scenarios where data exhibits significant high-order seasonality patterns (such as hourly or daily cycles). Previous versions from v1.5.1 to v1.8.0 were identified to contain a critical issue impacting model training, where models were inadvertently trained on limited data subsets, leading to suboptimal fits, affecting the accuracy of anomaly detection. <br><br> Upgrading to v1.9.2 addresses this issue, ensuring proper model training and enhanced reliability. For users utilizing Helm charts, it is recommended to upgrade to version [1.0.0](https://github.com/VictoriaMetrics/helm-charts/blob/master/charts/victoria-metrics-anomaly/CHANGELOG.md#100) or newer.**

## v1.15.0
Released: 2024-08-06
- FEATURE: Introduced models that support [online learning](https://en.wikipedia.org/wiki/Online_machine_learning) for stream-like input. These models significantly reduce the amount of data required for the initial fit stage. For example, they enable reducing `fit_every` from **weeks to hours** and increasing `fit_every` from **hours to weeks** in the [PeriodicScheduler](https://docs.victoriametrics.com/anomaly-detection/components/scheduler/#periodic-scheduler), significantly reducing the **peak amount** of data queried from VictoriaMetrics during `fit` stages. The next models were added:
  - [`OnlineZscoreModel`](https://docs.victoriametrics.com/anomaly-detection/components/models/#online-z-score) - online version of existing [Z-score](https://docs.victoriametrics.com/anomaly-detection/components/models/#z-score) implementation with the same exact behavior.
  - [`OnlineMADModel`](https://docs.victoriametrics.com/anomaly-detection/components/models/#online-mad) - online version of existing [MADModel](https://docs.victoriametrics.com/anomaly-detection/components/models/#mad-median-absolute-deviation) implementation with *approximate* behavior, based on [t-digests](https://www.sciencedirect.com/science/article/pii/S2665963820300403) for online quantile estimation.
  - [`OnlineQuantileModel`](https://docs.victoriametrics.com/anomaly-detection/components/models/#online-seasonal-quantile) - online quantile model, that supports custom ranges for seasonality estimation to cover more complex data patterns.
  - Find out more about online models specifics in [correspondent section](https://docs.victoriametrics.com/anomaly-detection/components/models/#online-models).

- FEATURE: Introduced the `optimized_business_params` key (list of strings) to the [`AutoTuned`](https://docs.victoriametrics.com/anomaly-detection/components/models/#autotuned) `optimization_params`. This allows particular business-specific parameters such as [`detection_direction`](https://docs.victoriametrics.com/anomaly-detection/components/models/#detection-direction) and [`min_dev_from_expected`](https://docs.victoriametrics.com/anomaly-detection/components/models/#minimal-deviation-from-expected) to remain **unchanged during optimizations, retaining their default values**.
- IMPROVEMENT: Optimized the [`AutoTuned`](https://docs.victoriametrics.com/anomaly-detection/components/models/#autotuned) model logic to minimize deviations from the expected `anomaly_percentage` specified in the configuration and the detected percentage in the data, while also reducing discrepancies between the actual values (`y`) and the predictions (`yhat`).
- IMPROVEMENT: Allow [`ProphetModel`](https://docs.victoriametrics.com/anomaly-detection/components/models/#prophet) to fit with multiple seasonalities when used in [`AutoTuned`](https://docs.victoriametrics.com/anomaly-detection/components/models/#autotuned) mode. 

## v1.14.2
Released: 2024-07-26
- FIX: Patch a bug introduced in [v1.14.1](#v1141), causing `vmanomaly` to crash in `preset` [mode](/anomaly-detection/presets/).

## v1.14.1
Released: 2024-07-26
- FEATURE: Allow to process larger data chunks in [VmReader](/anomaly-detection/components/reader#vm-reader) that exceed `-search.maxPointsPerTimeseries` [constraint in VictoriaMetrics](https://docs.victoriametrics.com/?highlight=search.maxPointsPerTimeseries#resource-usage-limits) by splitting the range and sending multiple requests. A warning is printed in logs, suggesting reducing the range or step, or increasing `search.maxPointsPerTimeseries` constraint in VictoriaMetrics, which is still a recommended option.
- FEATURE: Backward-compatible redesign of [`queries`](/anomaly-detection/components/reader?highlight=queries#vm-reader) arg of [VmReader](/anomaly-detection/components/reader#vm-reader). Old format of `{q_alias1: q_expr1, q_alias2: q_expr2, ...}` will be implicitly converted to a new one with a warning raised in logs. New format allows to specify per-query parameters, like `step` to reduce amount of data read from VictoriaMetrics TSDB and to allow config flexibility. Find out more in [Per-query parameters section of VmReader](/anomaly-detection/components/reader/#per-query-parameters).

- IMPROVEMENT: Added multi-platform builds for `linux/amd64` and `linux/arm64` architectures.

## v1.13.3
Released: 2024-07-17
- FIX: now validation of `args` argument for [`HoltWinters`](./components/models.md#holt-winters) model works properly.

## v1.13.2
Released: 2024-07-15
- IMPROVEMENT: update `node-exporter` [preset](./Presets.md#node-exporter) to reduce [false positives](https://victoriametrics.com/blog/victoriametrics-anomaly-detection-handbook-chapter-1/index.html#false-positive)
- FIX: add `verify_tls` arg for [`push`](./components/monitoring.md#push-config-parameters) monitoring section. Also, `verify_tls` is now correctly used in [VmWriter](./components/writer.md#vm-writer).
- FIX: now [`AutoTuned`](./components/models.md#autotuned) model wrapper works correctly in [on-disk model storage mode](./FAQ.md#resource-consumption-of-vmanomaly).
- FIX: now [rolling models](./components/models.md#rolling-models), like [`RollingQuantile`](./components/models.md#rolling-quantile) are properly handled in [One-off scheduler](./components/scheduler.md#oneoff-scheduler), when wrapped in [`AutoTuned`](./components/models.md#autotuned)

## v1.13.0
Released: 2024-06-11
- FEATURE: Introduced `preset` [mode to run vmanomaly service](./Presets.md) with minimal user input and on widely-known metrics, like those produced by [`node_exporter`](./Presets.md#node-exporter).
- FEATURE: Introduced `min_dev_from_expected` [model common arg](./components/models.md#minimal-deviation-from-expected), aimed at **reducing [false positives](https://victoriametrics.com/blog/victoriametrics-anomaly-detection-handbook-chapter-1/#false-positive)** in scenarios where deviations between the real value `y` and the expected value `yhat` are **relatively** high and may cause models to generate high [anomaly scores](./FAQ.md#what-is-anomaly-score). However, these deviations are not significant enough in **absolute values** to be considered anomalies based on domain knowledge.
- FEATURE: Introduced `detection_direction` [model common arg](./components/models.md#detection-direction), enabling domain-driven anomaly detection strategies. Configure models to identify anomalies occurring *above, below, or in both directions* relative to the expected values.
- FEATURE: add `n_jobs` arg to [`BacktestingScheduler`](./components/scheduler.md#backtesting-scheduler) to allow *proportionally faster (yet more resource-intensive)* evaluations of a config on historical data. Default value is 1, that implies *sequential* execution.
- FEATURE: allow anomaly detection models to be dumped to a host filesystem after `fit` stage (instead of in-memory). Resource-intensive setups (many models, many metrics, bigger [`fit_window` arg](./components/scheduler.md#periodic-scheduler-config-example)) and/or 3rd-party models that store fit data (like [ProphetModel](./components/models.md#prophet) or [HoltWinters](./components/models.md#holt-winters)) will have RAM consumption greatly reduced at a cost of slightly slower `infer` stage. Please find how to enable it [here](./FAQ.md#resource-consumption-of-vmanomaly)
- IMPROVEMENT: Reduced the resource used for each fitted [`ProphetModel`](./components/models.md#prophet) by up to 6 times. This includes both RAM for in-memory models and disk space for on-disk models storage. For more details, refer to [this discussion on Facebook's Prophet](https://github.com/facebook/prophet/issues/1159#issuecomment-537415637).
- IMPROVEMENT: now config [components](./components/README.md) class can be referenced by a short alias instead of a full class path - i.e. `model.zscore.ZscoreModel` becomes `zscore`, `reader.vm.VmReader` becomes `vm`, `scheduler.periodic.PeriodicScheduler` becomes `periodic`, etc.
- FIX: if using multi-scheduler setup (introduced in [v1.11.0](./CHANGELOG.md#v1110)), prevent schedulers (and correspondent services) that are not attached to any model (so neither found in ['schedulers'  arg](./components/models.md#schedulers) nor left blank in `model` section) from being spawn, causing resource overhead and slight interference with existing ones.
- FIX: set random seed for [ProphetModel](./components/models.md#prophet) to assure uncertainty estimates (like `yhat_lower`, `yhat_upper`) and dependant series (like `anomaly_score`), produced during `.infer()` calls are always deterministic given the same input. See [initial issue](https://github.com/facebook/prophet/issues/1124) for the details.
- FIX: prevent *orphan* queries (that are not attached to any model or scheduler) found in `queries` arg of [Reader config section](./components/reader.md#vm-reader) to be fetched from VictoriaMetrics TSDB, avoiding redundant data processing. A warning will be logged, if such queries exist in a parsed config.

## v1.12.0
Released: 2024-03-31
- FEATURE: Introduction of `AutoTunedModel` model class to optimize any [built-in model](./components/models.md#built-in-models) on data during `fit` phase. Specify as little as `anomaly_percentage` param from `(0, 0.5)` interval and `tuned_model_class` (i.e. [`model.zscore.ZscoreModel`](./components/models.md#z-score)) to get it working with best settings that match your data. See details [here](./components/models.md#autotuned).
<!--
- FEATURE: Preset support enablement. From now users will be able to specify only a few parameters (like `datasource_url`) + a new (backward-compatible) `preset: preset_name` field in a config file and get a service run with **predefined queries, scheduling and models**. Also, now preset assets (guide, configs, dashboards) will be available at `:8490/presets` endpoint.
-->
- IMPROVEMENT: Better logging of model lifecycle (fit/infer stages).
- IMPROVEMENT: Introduce `provide_series` arg to all the [built-in models](./components/models.md#built-in-models) to define what output fields to generate for writing (i.e. `provide_series: ['anomaly_score']` means only scores are being produced)
- FIX: [Self-monitoring metrics](./components/monitoring.md#models-behaviour-metrics) are now aggregated to `queries` aliases level (not to label sets of individual timeseries) and aligned with [reader, writer and model sections](./components/monitoring.md#metrics-generated-by-vmanomaly) description , so `/metrics` endpoint holds only necessary information for scraping.
- FIX: Self-monitoring metric `vmanomaly_models_active` now has additional labels `model_alias`, `scheduler_alias`, `preset` to align with model-centric [self-monitoring](./components/monitoring.md#models-behaviour-metrics).
- IMPROVEMENT: Add possibility to use temporal information in [IsolationForest models](./components/models.md#isolation-forest-multivariate) via [cyclical encoding](https://towardsdatascience.com/cyclical-features-encoding-its-about-time-ce23581845ca). This is particularly helpful to detect multivariate [seasonality](https://victoriametrics.com/blog/victoriametrics-anomaly-detection-handbook-chapter-1/#seasonality)-dependant anomalies.
- BREAKING CHANGE: **ARIMA** model is removed from [built-in models](./components/models.md#built-in-models). For affected users, it is suggested to replace ARIMA by [Prophet](./components/models.md#prophet) or [Holt-Winters](./components/models.md#holt-winters).

## v1.11.0
Released: 2024-02-22
- FEATURE: Multi-scheduler support. Now users can use multiple [model specs](./components/models.md) in a single config (via aliasing), each spec can be run with its own (even multiple) [schedulers](./components/scheduler.md).
  - Introduction of `schedulers` arg in model spec:
    - It allows each model to be managed by 1 (or more) schedulers, so overall resource usage is optimized and flexibility is preserved. 
    - Passing an empty list or not specifying this param implies that each model is run in **all** the schedulers, which is a backward-compatible behavior.
    - Please find more details in docs on [Model section](./components/models.md#schedulers)
- DEPRECATION: slight refactor of a scheduler config section 
  - Now schedulers are passed as a mapping of `scheduler_alias: scheduler_spec` under [scheduler](./components/scheduler.md) sections. Using old format (< [1.11.0](./CHANGELOG.md#v1110)) will produce warnings for now and will be removed in future versions.
- DEPRECATION: The `--watch` CLI option for config file reloads is deprecated and will be ignored in the future.

## v1.10.0
Released: 2024-02-15
- FEATURE: Multi-model support. Now users can specify multiple [model specs](./components/models.md) in a single config (via aliasing), as well as to reference what [queries from VmReader](./components/reader.md#config-parameters) it should be run on.
  - Introduction of `queries` arg in model spec:
    - It allows the model to be executed only on a particular query subset from `reader` section. 
    - Passing an empty list or not specifying this param implies that each model is run on results from **all** queries, which is a backward-compatible behavior.
    - Please find more details in docs on [Model section](./components/models.md#queries)

- DEPRECATION: slight refactor of a model config section 
  - Now models are passed as a mapping of `model_alias: model_spec` under [model](./components/models.md) sections. Using old format (<= [1.9.2](./CHANGELOG.md#v192)) will produce warnings for now and will be removed in future versions.
  - Please find more details in docs on [Model section](./components/models.md)
- IMPROVEMENT: now logs from [`monitoring.pull`](./components/monitoring.md#monitoring-section-config-example) GET requests to `/metrics` endpoint are shown only in DEBUG mode
- IMPROVEMENT: labelset for multivariate models is deduplicated and cleaned, resulting in better UX

> **Note**: These updates support more flexible setup and effective resource management in service, as now it's not longer needed to spawn several instances of `vmanomaly` to split queries/models context across.


## v1.9.2
Released: 2024-01-29
- BUGFIX: now multivariate models (like [`IsolationForestMultivariateModel`](./components/models.md#isolation-foresthttpsenwikipediaorgwikiisolation_forest-multivariate)) are properly handled throughout fit/infer phases.


## v1.9.1
Released: 2024-01-27
- IMPROVEMENT: Updated the offline license verification backbone to mitigate a critical vulnerability identified in the [`ecdsa`](https://pypi.org/project/ecdsa/) library, ensuring enhanced security despite initial non-impact.
- IMPROVEMENT: bump 3rd-party dependencies for Python 3.12.1


## v1.9.0
Released: 2024-01-26
- BUGFIX: The `query_from_last_seen_timestamp` internal logic in [VmReader](./components/reader.md#vm-reader), first introduced in [v1.5.1](#v151), now functions correctly. This fix ensures that the input data shape remains consistent for subsequent `fit`-based model calls in the service.
- BREAKING CHANGE: The `sampling_period` parameter is now mandatory in [VmReader](./components/reader.md#vm-reader). This change aims to clarify and standardize the frequency of input/output in `vmanomaly`, thereby reducing uncertainty and aligning with user expectations.
> **Note**: The majority of users, who have been proactively specifying the `sampling_period` parameter in their configurations, will experience no disruption from this update. This transition formalizes a practice that was already prevalent and expected among our user base.


## v1.8.0
Released: 2024-01-15
- FEATURE: Added Univariate [MAD (median absolute deviation)](./components/models.md#mad-median-absolute-deviation) model support.
- IMPROVEMENT: Update Python to 3.12.1 and all the dependencies.
- IMPROVEMENT: Don't check /health endpoint, check the real /query_range or /import endpoints directly. Users kept getting problems with /health.
- DEPRECATION: "health_path" param is deprecated and doesn't do anything in config ([reader](./components/reader.md#vm-reader), [writer](./components/writer.md#vm-writer), [monitoring.push](./components/monitoring.md#push-config-parameters)).


## v1.7.2
Released: 2023-12-21
- FIX: fit/infer calls are now skipped if we have insufficient *valid* data to run on.
- FIX: proper handling of `inf` and `NaN` in fit/infer calls.
- FEATURE: add counter of skipped model runs `vmanomaly_model_runs_skipped` to healthcheck metrics.
- FEATURE: add exponential retries wrapper to VmReader's `read_metrics()`.
- FEATURE: add `BacktestingScheduler` for consecutive retrospective fit/infer calls.
- FEATURE: add improved & numerically stable anomaly scores.
- IMPROVEMENT: add full config validation. The probability of getting errors in later stages (say, model fit) is greatly reduced now. All the config validation errors that needs to be fixed are now a part of logging.
  > **note**: this is an backward-incompatible change, as `model` config section now expects key-value args for internal model defined in nested `args`.
- IMPROVEMENT: add explicit support of `gzip`-ed responses from vmselect in VmReader.


## v1.6.0
Released: 2023-10-30
- IMPROVEMENT: 
  - now all the produced healthcheck metrics have `vmanomaly_` prefix for easier accessing.
  - updated docs for monitoring.
  > **note**: this is an backward-incompatible change, as metric names will be changed, resulting in new metrics creation, i.e. `model_datapoints_produced` will become `vmanomaly_model_datapoints_produced`
- IMPROVEMENT: Set default value for `--log_level` from `DEBUG` to `INFO` to reduce logs verbosity.
- IMPROVEMENT: Add alias `--log-level` to `--log_level`.
- FEATURE: Added `extra_filters` parameter to reader. It allows to apply global filters to all queries.
- FEATURE: Added `verify_tls` parameter to reader and writer. It allows to disable TLS verification for remote endpoint.
- FEATURE: Added `bearer_token` parameter to reader and writer. It allows to pass bearer token for remote endpoint for authentication.
- BUGFIX: Fixed passing `workers` parameter for reader. Previously it would throw a runtime error if `workers` was specified.

## v1.5.1
Released: 2023-09-18
- IMPROVEMENT: Infer from the latest seen datapoint for each query. Handles the case datapoints arrive late. 


## v1.5.0
Released: 2023-08-11
- FEATURE: add `--license` and `--license-file` command-line flags for license code verification. 
- IMPROVEMENT: Updated Python to 3.11.4 and updated dependencies.
- IMPROVEMENT: Guide documentation for Custom Model usage.


## v1.4.2
Released: 2023-06-09
- FIX: Fix case with received metric labels overriding generated.  


## v1.4.1
Released: 2023-06-09
- IMPROVEMENT: Update dependencies.


## v1.4.0
Released: 2023-05-06
- FEATURE: Reworked self-monitoring grafana dashboard for vmanomaly.
- IMPROVEMENT: Update python version and dependencies.


## v1.3.0
Released: 2023-03-21
- FEATURE: Parallelized queries. See `reader.workers` param to control parallelism. By default it's value is equal to number of queries (sends all the queries at once).
- IMPROVEMENT: Updated self-monitoring dashboard.
- IMPROVEMENT: Reverted back default bind address for /metrics server to 0.0.0.0, as vmanomaly is distributed in Docker images.
- IMPROVEMENT: Silenced Prophet INFO logs about yearly seasonality.

## v1.2.2
Released: 2023-03-19
- FIX: Fix `for` metric label to pass QUERY_KEY.
- FEATURE: Added `timeout` config param to reader, writer, monitoring.push.
- FIX: Don't hang if scheduler-model thread exits.
- FEATURE: Now reader, writer and monitoring.push will not halt the process if endpoint is inaccessible or times out, instead they will increment metrics `*_response_count{code=~"timeout|connection_error"}`.

## v1.2.1
Released: 2023-02-18
- FIX: Fixed scheduler thread starting.
- FIX: Fix rolling model fit+infer.
- BREAKING CHANGE: monitoring.pull server now binds by default on 127.0.0.1 instead of 0.0.0.0. Please specify explicitly in monitoring.pull.addr what IP address it should bind to for serving /metrics.

## v1.2.0
Released: 2023-02-04
- FEATURE: With arg `--watch` watches for config(s) changes and reloads the service automatically.
- IMPROVEMENT: Remove "provide_series" from HoltWinters model. Only Prophet model now has it, because it may produce a lot of series if "holidays" is on.
- IMPROVEMENT: if Prophet's "provide_series" is omitted, then all series are returned.
- DEPRECATION: Config monitoring.endpoint_url is deprecated in favor of monitoring.url.
- DEPRECATION: Remove 'enable' param from config monitoring.pull. Now /metrics server is started whenever monitoring.pull is present.
- IMPROVEMENT: include example configs into the docker image at /vmanomaly/config/*
- IMPROVEMENT: include self-monitoring grafana dashboard into the docker image under /vmanomaly/dashboard/vmanomaly_grafana_dashboard.json

## v1.1.0
Released: 2023-01-23
- IMPROVEMENT: update Python dependencies
- FEATURE: Add _multivariate_ IsolationForest model.

## v1.0.1
Released: 2023-01-06
- FIX: prophet model incorrectly predicted two points in case of only one

## v1.0.0-beta
Released: 2022-12-08
- First public release is available
