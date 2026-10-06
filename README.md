# Can an AI draw my dashboard?

A small test of natural-language-to-chart generation, inspired by the ChartGPT paper
(Tian et al., "ChartGPT: Leveraging LLMs to Generate Charts from Abstract Natural Language").
I asked a language model 30 plain-English questions about the data of my own app,
[StudyTracker](https://github.com/Salmalamnabhi/StudyTracker), and judged every chart it produced.

## Method

- **Data.** Three CSV files that follow the StudyTracker database schema
  (`study_sessions`, `class_tasks`, `class_schedules`). This is **sample data**
  covering 6 July to 4 October 2026, not my real study history.
- **Questions.** 30 questions in three groups of ten: easy (one column, one calculation),
  medium (a filter, a time grouping or a derived value) and vague (no column is named).
- **Model.** DeepSeek chat, DeepThink off; the model identified itself as DeepSeek-V3.2.
  Tested on 6 October 2026.
- **Prompt.** The same prompt every time (`prompt.txt`): column names and types only, no data rows,
  and a request for one Vega-Lite v5 specification. A new chat for each question, no retries.
- **Judging.** I rendered each reply and marked it by hand as Correct, Wrong column,
  Wrong calculation, Wrong chart type or Broken. For vague questions, Correct means
  a reasonable person would accept the chart as an answer.

## Results

| Level | Correct | Wrong column | Wrong calculation | Wrong chart type | Broken |
|---|---|---|---|---|---|
| Easy (10) | 9 | 0 | 0 | 0 | 1 |
| Medium (10) | 9 | 0 | 0 | 0 | 1 |
| Vague (10) | 6 | 1 | 2 | 0 | 1 |
| **Total (30)** | **24** | **1** | **2** | **0** | **3** |

The full table, with a note for each question, is in `results.csv`.

## What I found

**Vague questions failed most, in three different ways.**

- *A different meaning for a fuzzy word.* For "Am I getting more consistent?" the model plotted
  total hours per day. StudyTracker defines consistency as consecutive study days (its streak counter).
- *A clear word ignored.* "When do I study best?" was answered with subjects and difficulty, with no time at all.
- *A gap filled without saying so.* "Did I work harder near the end?" became weekday against weekend.

**The three charts that did not draw share one technical cause.** The model treated CSV values as numbers,
while Vega-Lite had read them as text (questions 6, 18 and 24). Changing one token in questions 6 and 18
made both draw correctly. Question 26 used loose equality (`== 1`) on the same column and worked.

**No chart had the wrong chart type.** The weaknesses in correct charts were about ordering and
presentation, for example Easy, Hard, Medium in alphabetical order.

## Limits

One model, one run per question, sample data, and a single judge. This is a first observation, not a measurement.

## Files

| File | Content |
|---|---|
| `data/` | The three sample CSV files |
| `prompt.txt` | The fixed prompt |
| `replies/q01.json` to `q30.json` | The model's unedited reply to each question |
| `results.csv` | Question, level, outcome and note |
| `chart_check.html` | The page I used to render and mark the charts (needs internet to load Vega) |
| `chartcheck_backup.json` | All replies and marks in one file; load it in `chart_check.html` to see every chart |

## Acknowledgement

The sample data, the question list and the checking page were prepared with the help of an AI assistant (Claude).
I ran the model, judged each chart and tested the cause of the failures myself.
