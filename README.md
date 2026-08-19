# Turn Your Home Energy Data into an AI Weekly Report

### Build an automated energy analysis workflow with ecoMain, Home Assistant and AI

Energy monitoring gives you data — but understanding what that data means still takes time.

With **ecoMain**, **Home Assistant** and an AI conversation agent, you can turn raw household or site energy measurements into an automatically generated weekly energy report.

In this example, Home Assistant collects energy statistics from ecoMain, compares the latest complete week with the previous week, sends the structured data to an AI agent for interpretation, and presents the results in a simple dashboard.

The result is a weekly report that helps users quickly understand:

- How much energy was used this week
- How consumption changed day by day
- Which monitored circuits consumed the most energy
- How this week compares with the previous week
- What energy-use patterns are worth paying attention to
- What practical actions could be considered next

> **[IMAGE 1 — Hero Image]**
>
> Suggested image: Final ecoMain AI Energy Dashboard  
> Suggested caption: *ecoMain AI Weekly Energy Report in Home Assistant*

---

## How It Works

The workflow combines three layers:

**ecoMain** provides the energy monitoring data.

**Home Assistant** handles historical statistics, automation, weekly comparison and dashboard visualization.

**AI** interprets the structured data and converts it into concise, human-readable energy insights.

```text
ecoMain
   ↓
Home Assistant
   ↓
Weekly Energy Statistics
   ↓
Current Week ↔ Previous Week
   ↓
AI Energy Analysis
   ↓
Weekly Energy Report
   ↓
Dashboard + Recommendations
```

> **[IMAGE 2 — System Architecture]**
>
> Suggested image:
>
> `ecoMain → Home Assistant → Weekly Statistics → AI → Weekly Report`
>
> Suggested output labels:
> `Daily Energy / Circuit Breakdown / Week-on-Week / AI Insights`

The AI does not replace energy measurement or statistical processing.

ecoMain provides the measured data, while Home Assistant first organizes the data into structured weekly statistics. The AI then works as an interpretation layer on top of those statistics.

---

# 1. Collect Energy Data with ecoMain

In this example, ecoMain is integrated with Home Assistant and provides both total energy consumption and individual monitored circuit data.

The system uses:

- `main_all` — total site forward energy consumption
- `CH1–CH10` — individual monitored circuits
- `CH7` — an R&D test load in this demonstration

The energy entities are recorded by Home Assistant and can be accessed through its historical and long-term statistics.

> **[IMAGE 3 — ecoMain Entities in Home Assistant]**
>
> Suggested image: Home Assistant entity or statistics page showing ecoMain energy sensors.

Because each monitored circuit is available independently, the system can go beyond a single total-energy value and identify how monitored energy consumption is distributed across different circuits.

---

# 2. Automatically Select the Weekly Reporting Period

The automation does not use fixed dates.

Every Monday, Home Assistant automatically identifies the previous complete Monday–Sunday period and the week before it.

For example:

```text
Current Report
Aug 10 – Aug 16, 2026

Previous Report
Aug 03 – Aug 09, 2026
```

The reporting periods therefore update automatically every week.

The date logic is handled directly inside the automation:

```yaml
- variables:
    week_start: >
      {{ today_at("00:00") - timedelta(days=now().weekday() + 7) }}

    week_end: >
      {{ today_at("00:00") - timedelta(days=now().weekday()) }}

    prev_start: >
      {{ today_at("00:00") - timedelta(days=now().weekday() + 14) }}

    prev_end: >
      {{ today_at("00:00") - timedelta(days=now().weekday() + 7) }}
```

This means the user does not need to manually change the reporting dates each week.

---

# 3. Retrieve Weekly ecoMain Statistics

Home Assistant retrieves daily energy statistics from ecoMain using `recorder.get_statistics`.

A simplified example:

```yaml
- action: recorder.get_statistics
  data:
    start_time: "{{ week_start }}"
    end_time: "{{ week_end }}"
    period: day
    statistic_ids:
      - sensor.ecomain_device_main_all_energy_fwd_total
      - sensor.ecomain_device_main_ch1_energy_fwd_total
      - sensor.ecomain_device_main_ch2_energy_fwd_total
      - sensor.ecomain_device_main_ch3_energy_fwd_total
    types:
      - change
  response_variable: current_stats
```

The same process is repeated for the previous week.

This gives Home Assistant the information needed for:

- Current-week energy totals
- Daily energy consumption
- Circuit-level energy consumption
- Week-on-week comparison

> **Note**
>
> Replace the example entity IDs with the entity IDs from your own ecoMain installation.

---

# 4. Understand This Week at a Glance

The first part of the dashboard focuses on a few headline metrics that can be understood immediately.

In our demonstration dataset:

| Metric | Result |
|---|---:|
| This Week | 109.16 kWh |
| Previous Week | 170.74 kWh |
| Week-on-Week | -36.1% |
| Reporting Period | Aug 10 – Aug 16, 2026 |

> **[IMAGE 4 — Weekly KPI Cards]**
>
> Suggested image:
>
> `This Week / Previous Week / Week-on-Week / Reporting Period`

These KPIs give the user a quick overview before they move into the detailed energy analysis.

---

# 5. See the Daily Energy Pattern

Weekly totals tell only part of the story.

The dashboard also visualizes daily energy consumption so users can see how energy use changes across the week.

In this example, daily site energy consumption varied significantly across the reporting period, with clear high- and low-consumption days.

> **[IMAGE 5 — Daily Energy Chart]**
>
> Suggested image: Home Assistant daily energy bar chart.

The AI can summarize this pattern in a short sentence rather than requiring the user to interpret the chart manually.

For example:

> Daily use ranged from 3.97 kWh to 23.69 kWh, with a sharp dip on Aug 11 and a peak on Aug 13.

The system describes what the data shows without automatically treating every peak or dip as a fault.

---

# 6. Break Energy Use Down by Circuit

ecoMain's circuit-level monitoring allows the weekly report to show where monitored energy consumption is concentrated.

For example:

| Circuit | Weekly Energy |
|---|---:|
| CH4 | 53.37 kWh |
| CH1 | 39.02 kWh |
| CH6 | 1.92 kWh |
| CH2 / CH3 / CH8 / CH9 / CH10 | No recorded use |

In this dataset, CH4 and CH1 represent most of the energy measured across the normal monitored circuits.

> **[IMAGE 6 — Circuit Breakdown]**
>
> Suggested image: Circuit Breakdown card from the Home Assistant dashboard.

This gives the AI more useful context than a single site-level energy value.

Instead of only reporting:

> "Your total energy use was 109.16 kWh."

the system can also explain where monitored consumption was concentrated.

---

## Separating Special Loads

Not every circuit needs to be interpreted in the same way.

In this demonstration, **CH7 is used as an R&D test circuit**.

It is therefore displayed separately from the normal monitored circuits.

> **[IMAGE 7 — R&D Test Load]**
>
> Suggested image: CH7 R&D Test Load card.

CH7 is excluded from:

- Normal monitored-circuit rankings
- Normal anomaly assessment
- Energy-saving recommendations

This prevents the AI from incorrectly interpreting a known test load as normal household or site energy use.

---

# 7. Compare This Week with Last Week

A single week can show what happened.

Two weeks provide context.

Home Assistant automatically calculates the week-on-week change.

For example:

```text
This Week        109.16 kWh
Previous Week    170.74 kWh
Week-on-Week     -36.1%
```

> **[IMAGE 8 — Week-on-Week Comparison]**
>
> Suggested image: Three comparison cards from the dashboard.

In this example, total measured site energy consumption decreased by **36.1%** compared with the previous week.

The comparison can also be used to identify which monitored circuits changed the most between the two reporting periods.

This provides a much stronger basis for energy interpretation than looking at a single week in isolation.

---

# 8. Turn Weekly Statistics into AI Energy Insights

This is where the AI layer becomes useful.

Home Assistant sends the current-week and previous-week ecoMain statistics to the configured conversation agent.

A simplified version looks like this:

```yaml
- action: conversation.process
  data:
    agent_id: conversation.deepseek
    text: >
      Analyze the current and previous week's
      ecoMain energy statistics.

      Identify:
      - current-week consumption pattern
      - major circuit pattern
      - week-on-week changes
      - abnormal energy use only when supported by data
      - practical energy-management recommendations
```

The AI is instructed not to invent:

- Appliance identities
- Operating reasons
- Equipment faults
- Unsupported savings estimates

For the dashboard, the longer report is condensed into three short insights.

### Weekly Pattern

> Daily use ranged 3.97–23.69 kWh with a sharp dip on Aug 11 and a peak on Aug 13.

### Circuit Pattern

> CH4 and CH1 accounted for most normal monitored-circuit energy during the week.

### Week-on-Week

> Total site energy decreased 36.1% compared with the previous week.

> **[IMAGE 9 — AI Energy Insights]**
>
> Suggested image: Three AI Energy Insights cards.

This creates a different experience from a conventional energy dashboard.

The dashboard does not only show **what the numbers are**.

It also helps the user understand **what changed, where the energy was concentrated, and what deserves attention next**.

---

# 9. Generate a Complete Weekly Energy Report

The dashboard insights are designed for quick reading.

At the same time, the AI generates a more detailed weekly report.

The report includes:

1. **This Week at a Glance**
2. **Daily Energy**
3. **Circuit Breakdown**
4. **Week-on-Week Comparison**
5. **R&D Test Load**
6. **AI Insights**
7. **Abnormal Use Assessment**
8. **Recommendations**

An example report can highlight:

- Total weekly site energy consumption
- Highest and lowest daily consumption
- Dominant monitored circuits
- Week-on-week changes
- Patterns worth continuing to monitor
- Practical next steps

If the available data does not provide enough evidence to confirm abnormal energy use, the AI is instructed to state:

> **No confirmed abnormal energy use was detected.**

This helps reduce overinterpretation of normal changes in energy consumption.

> **[IMAGE 10 — Complete AI Weekly Report]**
>
> Suggested image: Screenshot or designed mock-up of the full weekly report.

---

# 10. Save the Results for the Dashboard

The automation calculates and stores key values in Home Assistant helpers so that the dashboard can display the latest weekly report at any time.

Examples include:

```text
This Week Energy
Previous Week Energy
Week-on-Week Change
Reporting Period
CH1 Weekly Energy
CH4 Weekly Energy
CH7 R&D Test Energy
AI Weekly Pattern
AI Circuit Pattern
AI Week-on-Week Insight
```

The result is a dashboard that remains readable even after the automation has finished running.

> **[IMAGE 11 — Dashboard Data Flow]**
>
> Suggested diagram:
>
> `Recorder → Weekly Statistics → Helpers → Dashboard`

---

# 11. Run the Workflow Automatically Every Monday

Once configured, the workflow can run automatically once per week.

```yaml
triggers:
  - trigger: time
    at: "08:00:00"

conditions:
  - condition: template
    value_template: "{{ now().weekday() == 0 }}"
```

The complete process becomes:

```text
Monday 08:00
     ↓
Determine Previous Complete Week
     ↓
Retrieve ecoMain Statistics
     ↓
Retrieve Previous-Week Statistics
     ↓
Calculate Weekly KPIs
     ↓
Send Energy Data to AI
     ↓
Generate Weekly Insights
     ↓
Update Dashboard Values
     ↓
Create Weekly Energy Report
```

> **[IMAGE 12 — Automation Workflow]**
>
> Suggested image: Clean workflow diagram rather than a screenshot of the full YAML.

No fixed reporting dates are required.

As new ecoMain data is recorded, the next weekly report automatically moves to the latest complete reporting period.

---

# From Energy Monitoring to Energy Understanding

Energy monitoring is most useful when data leads to understanding.

In this example:

**ecoMain** acts as the **energy data layer**, providing total and circuit-level measurements.

**Home Assistant** acts as the **automation and integration layer**, organizing historical statistics, calculating weekly values and updating the dashboard.

**AI** acts as the **intelligence layer**, turning structured energy data into concise explanations and recommendations.

Together, the workflow moves from:

> **Measure → Organize → Compare → Understand → Act**

This architecture also means the concept is not limited to one specific AI model.

The energy data and Home Assistant automation remain the same, while the AI layer can potentially be replaced with another compatible cloud or local conversation agent.

---

# What You Need

For a similar setup, the basic components are:

- **ecoMain** energy monitor
- **Home Assistant**
- ecoMain energy entities available in Home Assistant
- Home Assistant Recorder / statistics
- A compatible AI conversation agent
- Home Assistant automation
- Dashboard helpers
- A Home Assistant dashboard

---

# Example Project Structure

For GitHub, we recommend organizing the project like this:

```text
ecomain-ai-weekly-energy-report/
│
├── README.md
│
├── automation/
│   └── weekly_energy_report.yaml
│
├── dashboard/
│   └── dashboard.yaml
│
└── images/
    ├── hero.png
    ├── architecture.png
    ├── weekly-kpi.png
    ├── daily-energy.png
    ├── circuit-breakdown.png
    ├── rnd-test-load.png
    ├── week-on-week.png
    ├── ai-insights.png
    └── full-dashboard.png
```

---

# Example Automation

The full automation can be placed in:

```text
automation/weekly_energy_report.yaml
```

Before sharing publicly, replace your real ecoMain device ID with an example ID.

For example, change:

```text
sensor.ecomain_003516000993_local_main_all_energy_fwd_total
```

to something like:

```text
sensor.ecomain_device_main_all_energy_fwd_total
```

Users can then replace the example IDs with the entity IDs from their own ecoMain installation.

---

# What's Next?

This example focuses on weekly electricity consumption and circuit-level analysis.

The same architecture could be extended with additional energy information such as:

- Solar generation
- Grid import and export
- Battery charging and discharging
- EV charging
- Electricity tariffs
- Monthly energy reports
- Energy-cost analysis
- Carbon-emission estimates

With richer energy data, the AI layer can move beyond weekly reporting toward more context-aware residential energy management.

---

# Build Your Own

The complete Home Assistant automation used in this demonstration can be shared as a reusable example for ecoMain users.

> **GitHub / Blueprint link to be added**

To adapt the example:

1. Replace the example ecoMain entity IDs with your own entities.
2. Connect your preferred compatible AI conversation agent.
3. Create the required Home Assistant helpers.
4. Add the dashboard cards.
5. Run the automation manually once for testing.
6. Enable the weekly Monday schedule.

---

## Notes

This project demonstrates an **ecoMain + Home Assistant + AI** energy-analysis workflow.

AI-generated insights depend on the available measurement data and should not be treated as confirmed equipment-fault diagnostics.

The example data shown in this project comes from a development and demonstration environment. Special test loads are separated from normal monitored-circuit analysis where appropriate.

---

## License

Add your preferred license here if you plan to make the automation reusable.

For example:

```text
MIT License
```

---

## About enecess

ecoMain is part of the enecess smart energy management ecosystem.

For more information about ecoMain and related energy-management products, visit the official enecess website.
```
