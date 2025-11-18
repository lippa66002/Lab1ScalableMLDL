# Lab 1: Air Quality Prediction System for Glasgow

Complete air quality prediction pipeline using XGBoost and Hopsworks to forecast PM2.5 levels across multiple sensors in Glasgow.

## Table of Contents
- [Overview](#overview)
- [Implementation](#implementation)
  - [Part 1: Basic Pipeline Setup](#part-1-basic-pipeline-setup)
  - [Part 2: Single-Sensor Model with Temporal Features](#part-2-single-sensor-model-with-temporal-features)
  - [Part 3: Multi-Sensor Prediction System](#part-3-multi-sensor-prediction-system)
- [Results](#results)
- [Tech Stack](#tech-stack)

## Overview

This lab progressively builds air quality prediction capabilities:
1. **Basic pipeline** with automated dashboard
2. **Enhanced single-sensor model** using 3-day lagged features
3. **Multi-sensor forecasting system** predicting 5 days ahead for 9 Glasgow sensors

## Implementation

### Part 1: Basic Pipeline Setup

Ran notebooks 1-4 to establish the foundation:
- Created feature pipelines for air quality and weather data
- Built initial prediction model
- Set up automated dashboard using GitHub Actions
- Configured visualization and monitoring

### Part 2: Single-Sensor Model with Temporal Features

**Goal**: Improve prediction accuracy by incorporating historical context

**What we did**:
- Added lagged PM2.5 values from the previous 3 days as features
- Trained XGBoost model on single sensor data from Glasgow
- Evaluated performance against baseline model

**Why it works**: Air quality today is strongly influenced by recent days, so giving the model this temporal context improves predictions.

**Results**: Noticeable performance improvement over the baseline (which already had decent performance).

### Part 3: Multi-Sensor Prediction System

The main component of this lab - predicting air quality for all 9 sensors in Glasgow over the next 5 days.

#### Step 1: Create New Feature Groups

Created and uploaded two feature groups to Hopsworks:

**Feature Group 1: Multi-Sensor Air Quality Data**
- PM2.5 measurements
- City (Glasgow)
- Street/sensor location
- Date
- Coverage: All 9 sensors

**Feature Group 2: Weather Forecasts**
- Future weather data (5-day horizon)
- Used for making forward predictions

#### Step 2: Train Multi-Sensor Model

Trained a new XGBoost model with these characteristics:
- **Training data**: Lagged PM2.5 values (3-day window) from ALL 9 sensors
- **Key difference from Part 2**: This model trains on combined data from all sensors, not just one

**Bonus evaluation** (not required but helpful):
- Evaluated model performance across all sensors
- Generated historical performance plots for each sensor location
- Helped us understand how the model behaves across different locations

#### Step 3: Generate 5-Day Forecasts

This was the tricky part - predicting multiple days into the future requires using our own predictions as features.

**The prediction process**:

1. **Get weather forecasts**: Extract weather data where date > today (5 days of forecasts)

2. **Initialize with real data**: For each sensor, grab the 3 most recent PM2.5 measurements

3. **Iterative prediction loop**:
```
   For each sensor:
       pm25_window = [3 most recent real PM2.5 values]
       
       For each forecast day (1 to 5):
           # Make prediction using current window + weather forecast
           prediction = model.predict(pm25_window + weather_data + sensor_location)
           
           # Slide the window: remove oldest, add newest predicted value
           pm25_window.pop(0)  # Remove oldest value
           pm25_window.append(prediction)  # Add newly predicted value
           
       Save all predictions for this sensor
```

**How the sliding window works**:
- Day 1 prediction: Uses 3 real historical values
- Day 2 prediction: Uses 2 real values + 1 predicted value (day 1)
- Day 3 prediction: Uses 1 real value + 2 predicted values (days 1-2)
- Day 4 prediction: Uses 3 predicted values (days 1-3)
- Day 5 prediction: Uses 3 predicted values (days 2-4)

4. **Repeat for all sensors**: Ran this entire process for each of Glasgow's 9 monitoring locations

#### Step 4: Visualize Results

Created plots showing:
- Historical performance for each sensor
- 5-day forecast predictions for each sensor
- Comparison of patterns across different locations

