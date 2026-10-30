# k6-lab

ITDS362 Software Quality Assurance and Testing, Lab: Performance Testing with Grafana K6.

K6 scripts run against [QuickPizza](https://quickpizza.grafana.com), Grafana's public demo site, with no more than 5 virtual users.

- Part 0 scripts: `01-first-test.js` to `08-bug-hunt.js`, plus `explore.js`
- Part A user journey: `journey.js`, used by `a1-smoke.js` to `a5-soak.js`
- HTML reports for Steps A2 to A5: `reports/`
- Results and answers: [RESULTS.md](RESULTS.md)

Run any script with `k6 run <script>.js`. For the reports:

```
k6 run --out web-dashboard=export=reports/a2-average.html a2-average.js
```
