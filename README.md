# Code-for-All-tutorial

# Consulting Simulator

**Author:** Seunghyun Ryu (G01511261)  
**Project Type:** Portfolio / MVP Prototype  
**Focus Areas:** Management Consulting · Data Analytics · Management Information Systems  

---

## 🎯 Project Purpose

The **Consulting Simulator** is a portfolio project designed to demonstrate how structured data can be transformed into **insights, recommendations, and decision support**.  
It highlights applied skills in:

- **Management Consulting:** framing problems and providing actionable recommendations.  
- **Data Analysis:** calculating KPIs, forecasting, and scenario modeling.  
- **Information Systems:** building a user-friendly MVP in Streamlit with dashboards and automation.  

---

## 💡 Concept

The **Consulting Simulator** acts as a **lightweight decision-support tool**.  
It enables users to:  

1. Upload their **business data** (CSV/Excel).  
2. Automatically generate **KPIs** like ROI, profit margin, revenue per employee, customer retention rate, and inventory turnover.  
3. Run **scenario simulations** (e.g., increasing marketing budget, adding staff).  
4. Receive **automated consulting insights** — a concise report highlighting risks & opportunities.  
5. Visualize **dashboards and forecasts** interactively.  

The tool bridges the gap between **business consulting frameworks** and **technical analytics execution**.

---

## 👥 Target Users

- **Small businesses / startups** needing fast, simple insights.  
- **Recruiters / consulting managers** reviewing a portfolio project that integrates analytics and consulting skills.  

---

## 🚀 MVP Features

1. **Data Input**  
   - Upload CSV/Excel with columns: `Revenue`, `Cost`, `Employees`, `Customers`, `Inventory`.  
   - Missing values handled with defaults (e.g., zero, average).  

2. **KPI Generation**  
   - Automatically compute:  
     - ROI  
     - Profit Margin  
     - Revenue per Employee  
     - Inventory Turnover  
     - Retention Rate (if available)  

3. **Scenario Simulation**  
   - Modify variables such as marketing spend or staffing.  
   - View real-time KPI updates under simulated conditions.  

4. **Automated Consulting Report**  
   - AI-style logic generates short recommendations.  
   - Example: “ROI is below 10%. Recommend cost optimization.”  

5. **Dashboard Visualization**  
   - Interactive charts (Plotly) showing revenue vs. cost, ROI comparison, forecasts.  

---

## 📊 Expected Impacts

1. **Data Input:** Quick and intuitive way for businesses to provide operational data.  
2. **KPI Generation:** Turns raw figures into **actionable business metrics**.  
3. **Scenario Simulation:** Interactive “what-if” analysis improves decision-making.  
4. **Automated Report:** Bridges the gap between **numbers and strategy**.  

---

## 🛠️ Example Code (app.py)

```python
import streamlit as st
import pandas as pd
import plotly.graph_objects as go
import numpy as np

# --------------------------
# 1. Page Config & Title
# --------------------------
st.set_page_config(layout="wide")
st.title("Consulting Simulator")

uploaded_file = st.file_uploader("Upload CSV with business metrics (Revenue, Cost, Employees)", type=["csv", "xlsx"])

if uploaded_file:
    # --------------------------
    # 2. Data Loading
    # --------------------------
    try:
        if uploaded_file.name.endswith(".csv"):
            data = pd.read_csv(uploaded_file)
        else:
            data = pd.read_excel(uploaded_file)
    except Exception as e:
        st.error(f"Error loading file: {e}")
        st.stop()

    # --------------------------
    # 3. KPI Calculation
    # --------------------------
    revenue = data['Revenue'].sum()
    cost = data['Cost'].sum()
    employees = data['Employees'].sum() if 'Employees' in data.columns else 0

    roi = (revenue - cost) / cost if cost != 0 else 0
    revenue_per_employee = revenue / employees if employees != 0 else 0
    profit_margin = (revenue - cost) / revenue if revenue != 0 else 0

    # KPI Cards
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Total Revenue", f"${revenue:,.0f}")
    col2.metric("ROI", f"{roi:.2%}")
    col3.metric("Revenue per Employee", f"${revenue_per_employee:,.0f}")
    col4.metric("Profit Margin", f"{profit_margin:.2%}")

    # --------------------------
    # 4. Scenario Simulation
    # --------------------------
    st.subheader("Scenario Simulation")
    marketing_increase = st.number_input("Increase Marketing Budget by %", min_value=0, max_value=100, value=0)
    additional_employees = st.number_input("Add Employees", min_value=0, max_value=10, value=0)

    simulated_cost = cost + cost * (marketing_increase / 100) + additional_employees * 5000
    simulated_roi = (revenue - simulated_cost) / simulated_cost if simulated_cost != 0 else 0

    st.metric("Simulated ROI", f"{simulated_roi:.2%}")

    # --------------------------
    # 5. Revenue vs Cost Trend
    # --------------------------
    st.subheader("Revenue vs Cost Trend")
    fig = go.Figure()
    fig.add_trace(go.Bar(x=data.index, y=data['Revenue'], name='Revenue'))
    fig.add_trace(go.Bar(x=data.index, y=data['Cost'], name='Cost'))
    fig.update_layout(barmode='group', title="Revenue vs Cost per Entry", xaxis_title="Entry", yaxis_title="Amount ($)")
    st.plotly_chart(fig, use_container_width=True)

    # --------------------------
    # 6. ROI Scenario Comparison
    # --------------------------
    st.subheader("ROI Scenario Comparison")
    scenario_fig = go.Figure()
    scenario_fig.add_trace(go.Bar(name='Current ROI', x=['ROI'], y=[roi]))
    scenario_fig.add_trace(go.Bar(name='Simulated ROI', x=['ROI'], y=[simulated_roi]))
    scenario_fig.update_layout(barmode='group', title="Current vs Simulated ROI")
    st.plotly_chart(scenario_fig, use_container_width=True)

    # --------------------------
    # 7. Simple Forecast
    # --------------------------
    st.subheader("Revenue & ROI Forecast (Next 3 periods)")
    periods = 3
    revenue_forecast = [revenue * (1 + 0.05 * i) for i in range(1, periods + 1)]
    roi_forecast = [(revenue_forecast[i] - simulated_cost) / simulated_cost for i in range(periods)]

    forecast_fig = go.Figure()
    forecast_fig.add_trace(go.Scatter(x=[f"Period {i+1}" for i in range(periods)], y=revenue_forecast,
                                      mode='lines+markers', name='Revenue Forecast'))
    forecast_fig.add_trace(go.Scatter(x=[f"Period {i+1}" for i in range(periods)], y=roi_forecast,
                                      mode='lines+markers', name='ROI Forecast'))
    forecast_fig.update_layout(title="Revenue & ROI Forecast", yaxis_title="Amount / ROI")
    st.plotly_chart(forecast_fig, use_container_width=True)

    # --------------------------
    # 8. Automated Consulting Insights
    # --------------------------
    st.subheader("Automated Consulting Insights")
    insights = []
    if roi < 0.1:
        insights.append("ROI is below 10%. Recommend cost optimization or efficiency improvements.")
    if revenue_per_employee < 50000:
        insights.append("Revenue per employee is low. Consider workforce training or process optimization.")
    if marketing_increase > 20:
        insights.append("High marketing spend may reduce ROI; ensure alignment with expected revenue growth.")
    if profit_margin < 0.15:
        insights.append("Profit margin is under 15%. Review pricing strategy or reduce variable costs.")
    if simulated_roi > roi:
        insights.append("Scenario shows improved ROI. Consider feasibility of proposed changes.")

    if insights:
        for i, insight in enumerate(insights, 1):
            st.write(f"{i}. {insight}")
    else:
        st.write("All metrics are within healthy ranges. Keep current strategy.")
else:
    st.info("Please upload a CSV or Excel file to begin analysis.")
