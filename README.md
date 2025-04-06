# Project Monitoring Dashboard

## Overview
This repository contains a **Project Monitoring Dashboard** built with Python, Pandas, Matplotlib, and Seaborn. The dashboard is designed to track and visualize key metrics for jobs with reoccupancy data, including technician hours, lab sample costs, and total project costs. It processes data from multiple CSV files (`CONTRACTED JOBS`, `LOGS`, `LABS`, and `REOCCUPANCY`) to provide insights into job timelines, labor, and expenses.

The project was developed as part of an effort to monitor ongoing contracts, focusing on jobs that have reoccupancy data. It includes visualizations such as a Gantt chart for job timelines, bar charts for technician hours and lab costs, and a stacked bar chart for cost breakdowns.

## Features
- **Data Processing**: Cleans and merges data from multiple CSV files (`JOBS`, `LOGS`, `LABS`, `REOCCUPANCY`).
- **Cost Calculations**:
  - Labor costs based on technician titles (e.g., INVESTIGATOR at $128/hour, PROJECT MONITOR A at $38/hour).
  - Lab sample costs based on sample type and lab facility (e.g., $6/sample for "Backgrounds" at LTS).
- **Visualizations**:
  - **Job Timeline (Gantt Chart)**: Displays the duration of each job from start to end date.
  - **Technician Hours**: Bar chart showing total hours worked per job.
  - **Lab Costs**: Bar chart of lab sample costs per job.
  - **Total Cost Breakdown**: Stacked bar chart showing labor and sample costs per job.
- **Filtering**: Focuses on jobs with reoccupancy data for targeted analysis.

## Prerequisites
To run this project, you'll need the following:
- Python 3.7 or higher
- Required Python libraries:
  - `pandas`
  - `matplotlib`
  - `seaborn`

You can install the dependencies using pip:
```bash
pip install pandas matplotlib seaborn
