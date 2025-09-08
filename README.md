# Code-for-All-tutorial

Seunghyun Ryu (G01511261)

Topic: Consulting Simulator
Purpose: Portfolio project to showcase skills in 
- Management Consultng
- Data Analysis
- Management Information Systems

Concept: "Consulting Simulator" is a data-driven MVP that allows users to input business data and receive real-time KPIs, scenario simulations, and automated consulting insights. The goal is to demonstrate how managment decisions can be supported by structured data analysis tools.

Target Users: 
- Small businesses or startups looking for quick & simple operational insights
- Portfolio viewers (recruiters / consulting managers) to showcase analytical consulting capabilities

MVP Features:
1. Data Input - Users can upload CSV / Excel files with basic business metrics (Revenue / Costs / Employee count / Customer count / Inventory...)
2. KPI Generation - Automatically calculate key performace indicators such as: ROI / Revenue per employee / customer retention rate / Inventory turnover
3. Scenario Simulation - Users can modify variables to test hypothetical changes, e.g.: "Increase marketing budget by 10%", "Add 2 employees" - System predicts KPI changes in real-time and visualizes the outcomes
4. Automated Consulting Report - Generates brief insights and recommendations based on KPI analysis 
5. Dashboard Visualization - Graphs and charts for trends and patterns & Scenario simulation results dynamically updated on the dashboard

Expected Impacts
1. Data Input: Users can quickly provide their business data
2. KPI Generation: Converst raw data into actionable business metrics
3. Scenario Simulation: Provdies interactive "what-if" analysis for decision making process
4. Automated Consulting Report: Translates data into concise, actionable insights & Demonstrates ability to bridege data analysis and consulting recommendations

Example Codes

import streamlit as st
import pandas as pd
import plotly.graph_objects as go
import numpy as np

# --------------------------
# 1. Page Config & Title
# --------------------------
st.set_page_config(layout="wide")
st.title("Advanced Consulting Simulator")

uploaded_file = st.file_uploader("Upload CSV with business metrics (Revenue, Cost, Employees)", type="csv")

if uploaded_file is not None:
    data = pd.read_csv(uploaded_file)
    st.subheader("Data Preview")
    st.dataframe(data)

    # --------------------------
    # 2. KPI Calculation
    # --------------------------
    revenue = data['Revenue'].sum()
    cost = data['Cost'].sum()
    employees = data['Employees'].sum()
    roi = (revenue - cost) / cost if cost != 0 else 0
    revenue_per_employee = revenue / employees if employees != 0 else 0
    profit_margin = (revenue - cost) / revenue if revenue != 0 else 0

    # KPI Cards (4 columns)
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Total Revenue", f"${revenue:,.0f}")
    col2.metric("ROI", f"{roi:.2%}")
    col3.metric("Revenue per Employee", f"${revenue_per_employee:,.0f}")
    col4.metric("Profit Margin", f"{profit_margin:.2%}")

    # --------------------------
    # 3. Scenario Simulation
    # --------------------------
    st.subheader("Scenario Simulation")
    marketing_increase = st.number_input("Increase Marketing Budget by %", min_value=0, max_value=100, value=0)
    additional_employees = st.number_input("Add Employees", min_value=0, max_value=10, value=0)

    simulated_cost = cost + cost * (marketing_increase / 100) + additional_employees * 5000
    simulated_roi = (revenue - simulated_cost) / simulated_cost if simulated_cost != 0 else 0

    st.metric("Simulated ROI", f"{simulated_roi:.2%}")

    # --------------------------
    # 4. Revenue & Cost Trend Plot
    # --------------------------
    st.subheader("Revenue vs Cost Trend")
    fig = go.Figure()
    fig.add_trace(go.Bar(x=data.index, y=data['Revenue'], name='Revenue'))
    fig.add_trace(go.Bar(x=data.index, y=data['Cost'], name='Cost'))
    fig.update_layout(barmode='group', title="Revenue vs Cost per Entry", xaxis_title="Entry", yaxis_title="Amount ($)")
    st.plotly_chart(fig, use_container_width=True)

    # --------------------------
    # 5. ROI Scenario Comparison Chart
    # --------------------------
    st.subheader("ROI Scenario Comparison")
    scenario_fig = go.Figure()
    scenario_fig.add_trace(go.Bar(name='Current ROI', x=['ROI'], y=[roi]))
    scenario_fig.add_trace(go.Bar(name='Simulated ROI', x=['ROI'], y=[simulated_roi]))
    scenario_fig.update_layout(barmode='group', title="Current vs Simulated ROI")
    st.plotly_chart(scenario_fig, use_container_width=True)

    # --------------------------
    # 6. Simple Forecast (Revenue & ROI)
    # --------------------------
    st.subheader("Revenue & ROI Forecast (Next 3 periods)")
    periods = 3
    revenue_forecast = [revenue * (1 + 0.05 * i) for i in range(1, periods + 1)]  # simple 5% growth per period
    roi_forecast = [(revenue_forecast[i] - simulated_cost) / simulated_cost for i in range(periods)]

    forecast_fig = go.Figure()
    forecast_fig.add_trace(go.Scatter(x=[f"Period {i+1}" for i in range(periods)], y=revenue_forecast,
                                      mode='lines+markers', name='Revenue Forecast'))
    forecast_fig.add_trace(go.Scatter(x=[f"Period {i+1}" for i in range(periods)], y=roi_forecast,
                                      mode='lines+markers', name='ROI Forecast'))
    forecast_fig.update_layout(title="Revenue & ROI Forecast", yaxis_title="Amount / ROI")
    st.plotly_chart(forecast_fig, use_container_width=True)

    # --------------------------
    # 7. Enhanced Automated Consulting Insights
    # --------------------------
    st.subheader("Automated Consulting Insights")
    insights = []
    if roi < 0.1:
        insights.append("ROI is below 10%. Recommend cost optimization or efficiency improvements.")
    if revenue_per_employee < 50000:
        insights.append("Revenue per employee is low. Consider training or process improvements.")
    if marketing_increase > 20:
        insights.append("High marketing spend may reduce ROI; balance budget with expected revenue impact.")
    if profit_margin < 0.15:
        insights.append("Profit margin is under 15%. Review pricing strategy or reduce variable costs.")
    if simulated_roi > roi:
        insights.append("Scenario simulation shows improved ROI. Evaluate feasibility of proposed changes.")

    for i, insight in enumerate(insights, 1):
        st.write(f"{i}. {insight}")

else:
    st.info("Please upload a CSV file to start analysis.")
