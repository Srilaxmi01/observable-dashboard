<div>
  <h3>Sri Laxmi Nathi</h3>
  <h3>801412570</h3>
</div>

<div style="text-align: center; max-width: 800px; margin: 0 auto;">
  <h2 style="margin: 10px 0;">Fitness and Wellness Tracker Dashboard</h2>
  <p>This dashboard explores fitness activities, calories burned, workout types, and health metrics based on daily user data collected from a fitness tracking application.</p>
</div>

---

### 🏋️ Theme
Understanding daily fitness activities, calories burned, heart rate trends, and mood shifts across different workout patterns.

---

### 📊 About the Dataset
The dataset contains 10,000 records of individuals' daily fitness tracking activities. Each entry records calories burned, workout types, duration, distances, heart rates, moods, and step counts. The goal is to uncover fitness patterns and suggest actionable health and wellness insights.

---

<style>
  .viz-grid {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 20px;
    margin-bottom: 40px;
  }

  .viz-half {
    flex: 1 1 40%;
    padding: 6px;
    border: 2px solid #ccc;
    border-radius: 12px;
    background: linear-gradient(135deg, #e0f7fa, #ffffff); /* 🌟 nice blueish gradient */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* soft shadow */
  }

  .viz-full {
  padding: 6px;
  border: 2px solid #bbb;
  border-radius: 12px;
  background: linear-gradient(135deg, #e0f7fa, #ffffff);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.08);
  margin-bottom: 20px;
  max-width: 90%;   /* optional: shrink full width box slightly */
  margin-left: auto;
  margin-right: auto;
  }

  .filter-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 30px;
  }

  .filter-box {
    padding: 10px;
    border: 2px solid #ccc;
    border-radius: 12px;
    background: linear-gradient(135deg,rgb(246, 231, 242), #ffffff); /* 🌟 light purple gradient */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: 300px;
  }
</style>

```js
import { FileAttachment } from "observablehq:stdlib";
import * as Plot from "@observablehq/plot";
import * as Inputs from "@observablehq/inputs";

// Load the dataset
const data = await FileAttachment("./data/workout_fitness_tracker_data.csv").csv({ typed: true });
```
<div class="filter-grid"> <div class="filter-box">

## 🔘 Filter by Gender:
```js
const genderFilter = view(Inputs.select(
  [...new Set(data.map(d => d["Gender"]))],
  { label: "Select Gender", value: "Male" }
));
```
</div> <div class="filter-box">

## 🔘 Filter by Workout Type:
```js
const workoutTypes = ["All Types", ...new Set(data.map(d => d["Workout Type"]))];
const workoutTypeFilter = view(Inputs.select(
  workoutTypes,
  { label: "Select Workout Type", value: "All Types" }
));
```
</div>
</div>

```js
// Filter dataset based on selections
const genderAndWorkoutFilteredData = data.filter(d =>
  d["Gender"] === genderFilter &&
  (workoutTypeFilter === "All Types" || d["Workout Type"] === workoutTypeFilter)
);
```
<div class="viz-grid"> 
<div class="viz-half">


## 📈 Visualization 1: Average Calories Burned by Workout Type

```js
const sumCaloriesByWorkout = Object.values(
  genderAndWorkoutFilteredData.reduce((acc, row) => {
    const type = row["Workout Type"];
    if (!acc[type]) {
      acc[type] = { Workout_Type: type, calories: 0 };
    }
    acc[type].calories += +row["Calories Burned"];
    return acc;
  }, {})
);
```

```js
display(
  Plot.plot({
    width: 700,
    height: 500,
    x: { label: "Workout Type" },
    y: { label: "Total Calories Burned" }, // <-- Update label
    color: { scheme: "turbo" },
    marks: [
      Plot.barY(sumCaloriesByWorkout, {
        x: "Workout_Type",
        y: "calories",  // <-- use 'calories' directly now
        tip: true,
        title: d => `${d.Workout_Type}: ${d.calories.toFixed(0)} kcal`
      })
    ]
  })
);
```


</div>

<div class="viz-half">

## ❤️ Visualization 2: Average Heart Rate by Workout Duration

```js
const heartRateBuckets = {};

for (const row of genderAndWorkoutFilteredData) {
  const duration = +row["Workout Duration (mins)"];
  const heartRate = +row["Heart Rate (bpm)"];
  if (isNaN(duration) || isNaN(heartRate)) continue;

  const bucket = Math.floor(duration / 5) * 5;
  if (!heartRateBuckets[bucket]) heartRateBuckets[bucket] = { bucket, heartRateSum: 0, count: 0 };

  heartRateBuckets[bucket].heartRateSum += heartRate;
  heartRateBuckets[bucket].count += 1;
}

const heartRateByDuration = Object.values(heartRateBuckets)
  .map(d => ({
    Duration: d.bucket,
    AvgHeartRate: d.heartRateSum / d.count
  }))
  .sort((a, b) => a.Duration - b.Duration);
```

```js
display(
  Plot.plot({
    width: 700,
    height: 500,
    x: { label: "Workout Duration (mins)" },
    y: { label: "Average Heart Rate (bpm)" },
    marks: [
      Plot.line(heartRateByDuration, {
        x: "Duration",
        y: "AvgHeartRate",
        stroke: "#e63946"
      }),
      Plot.dot(heartRateByDuration, {
        x: "Duration",
        y: "AvgHeartRate",
        r: 3,
        tip: true,
        title: d => `${d.Duration} mins: ${d.AvgHeartRate.toFixed(1)} bpm`
      })
    ]
  })
);
```

</div>

</div>

---

<div class="viz-full">

<div>

## 😊 Visualization 3: Heatmap — Mood Before vs. After Workout

```js
const moodPairs = {};

// ✅ Use filtered data now!
for (const row of genderAndWorkoutFilteredData) {
  const before = row["Mood Before Workout"]?.trim();
  const after = row["Mood After Workout"]?.trim();
  if (!before || !after) continue;

  const key = `${before}|${after}`;
  if (!moodPairs[key]) moodPairs[key] = { before, after, count: 0 };
  moodPairs[key].count += 1;
}

const moodMatrix = Object.values(moodPairs);
```

```js
display(
  Plot.plot({
    width: 1200,
    height: 500,
    x: {
      label: "Mood After Workout",
      domain: [...new Set(genderAndWorkoutFilteredData.map(d => d["Mood After Workout"]))].sort()
    },
    y: {
      label: "Mood Before Workout",
      domain: [...new Set(genderAndWorkoutFilteredData.map(d => d["Mood Before Workout"]))].sort().reverse()
    },
    color: { scheme: "reds", legend: true, label: "Frequency" },
    marks: [
      Plot.cell(moodMatrix, {
        x: "after",
        y: "before",
        fill: "count",
        title: d => `Before: ${d.before}, After: ${d.after}\nCount: ${d.count}`
      })
    ]
  })
);

```

</div>


</div>


## 📝 Final Insights and Observations

- **Workout Impact**: Cardio and HIIT workouts burn the most calories on average.
- **Heart Rate Trend**: Average heart rate rises with longer workout durations, peaking around 40–60 minutes.
- **Mood Shifts**: Users often transition from "Tired", "Sad", or "Neutral" moods to more "Energetic", "Relaxed", or "Happy" states after workouts.

---
