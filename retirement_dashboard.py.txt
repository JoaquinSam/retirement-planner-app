import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import plotly.express as px
from datetime import datetime, date

# ---------------------- Configurations ----------------------
st.set_page_config(page_title="India Retirement Planner", layout="wide", initial_sidebar_state="expanded")
st.title("🇮🇳 Comprehensive Retirement Planning Dashboard")
st.markdown("A powerful tool to help you plan a secure and smart retirement in India.")

# ---------------------- Sidebar ----------------------
st.sidebar.header("🗕️ Personal Info")
name = st.sidebar.text_input("Name", "Joaquin Sam")
current_age = st.sidebar.slider("Current Age", 20, 60, 30)
retirement_age = st.sidebar.slider("Retirement Age", current_age+1, 70, 60)
life_expectancy = st.sidebar.slider("Life Expectancy", 60, 100, 85)

# ---------------------- Section 1: Retirement Goal Calculator ----------------------
st.header("🌟 Retirement Goal Calculator")
current_expense = st.number_input("Current Monthly Expense (₹)", 5000, 500000, 30000, step=1000)
inflation_rate = st.slider("Expected Inflation Rate (%)", 3.0, 10.0, 6.0)
post_ret_return = st.slider("Post-retirement Return (%)", 3.0, 10.0, 5.0)

years_to_retire = retirement_age - current_age
years_after_retire = life_expectancy - retirement_age
inflated_expense = current_expense * ((1 + inflation_rate/100) ** years_to_retire)
annual_expense = inflated_expense * 12
r = post_ret_return / 100
corpus_required = annual_expense * ((1 - (1 + r) ** -years_after_retire) / r)

st.success(f"🧶 Estimated Corpus Needed: ₹{corpus_required:,.0f}")

# ---------------------- Section 2: Investment Portfolio Builder ----------------------
st.header("📊 Investment Portfolio Builder")
risk_profile = st.selectbox("Select Risk Profile", ["Conservative", "Balanced", "Aggressive"])

allocations = {
    "Conservative": {"Equity MF": 20, "Debt MF": 30, "PPF": 30, "SCSS": 20},
    "Balanced": {"Equity MF": 40, "Debt MF": 30, "NPS": 20, "PPF": 10},
    "Aggressive": {"Equity MF": 60, "NPS": 30, "Debt MF": 10}
}

selected_allocation = allocations[risk_profile]
allocation_df = pd.DataFrame({"Investment": list(selected_allocation.keys()), "Allocation %": list(selected_allocation.values())})
fig_alloc = px.pie(allocation_df, values='Allocation %', names='Investment', title='Asset Allocation', hole=0.3)
st.plotly_chart(fig_alloc)

# ---------------------- Section 3: Tax Savings Estimator ----------------------
st.header("📟 Tax Savings Estimator")
col1, col2 = st.columns(2)
with col1:
    ppf = st.number_input("PPF (₹)", 0, 150000, 50000)
    elss = st.number_input("ELSS (₹)", 0, 150000, 30000)
    life_insurance = st.number_input("Life Insurance (₹)", 0, 150000, 20000)
    scss = st.number_input("SCSS/PMVVY (₹)", 0, 150000, 40000)
with col2:
    nps = st.number_input("NPS Tier 1 (₹)", 0, 50000, 20000)
    health_insurance = st.number_input("Health Insurance (₹)", 0, 100000, 25000)

total_80C = min(ppf + elss + life_insurance + scss, 150000)
total_80CCD = min(nps, 50000)
total_80D = min(health_insurance, 25000)
total_tax_benefit = total_80C + total_80CCD + total_80D

st.success(f"✅ Total Tax Deduction Available: ₹{total_tax_benefit:,}")

# ---------------------- Section 4: Annuity Options ----------------------
st.header("🏦 Annuity Plan Comparison")
annuity_options = pd.DataFrame({
    'Plan': ['LIC Jeevan Akshay', 'SBI Life Annuity Plus', 'HDFC Life Pension Guaranteed Plan'],
    'Interest Rate (%)': [6.5, 6.3, 6.7],
    'Min Age': [30, 40, 45],
    'Max Age': [85, 80, 85],
    'Monthly Payout (₹1L invested)': [542, 528, 556]
})
st.dataframe(annuity_options)

# ---------------------- Section 5: Inflation Impact Visualizer ----------------------
st.header("📈 Inflation Impact on Future Expenses")
years = list(range(1, years_to_retire+1))
expenses = [current_expense * ((1 + inflation_rate/100) ** y) for y in years]
infl_df = pd.DataFrame({"Year": years, "Projected Expense": expenses})
fig_exp = px.line(infl_df, x="Year", y="Projected Expense", title="Projected Monthly Expense Over Years", markers=True)
st.plotly_chart(fig_exp)

# ---------------------- Section 6: Emergency Fund Calculator ----------------------
st.header("🚨 Emergency Fund Calculator")
emergency_months = st.slider("Months to Cover", 3, 12, 6)
emergency_fund = current_expense * emergency_months
st.info(f"You need ₹{emergency_fund:,.0f} for {emergency_months} months of emergency fund.")

# ---------------------- Section 7: Retirement Readiness Score ----------------------
st.header("🔎 Retirement Readiness Score")
saved_so_far = st.number_input("Total Retirement Savings So Far (₹)", 0, int(corpus_required), 1000000)
readiness = min(100, round((saved_so_far / corpus_required) * 100))
st.progress(readiness)
st.metric(label="Retirement Readiness", value=f"{readiness}%")

# ---------------------- Section 8: Debt vs Equity Returns Comparison ----------------------
st.header("📊 Debt vs Equity Returns Comparison")
returns_df = pd.DataFrame({
    'Year': list(range(datetime.today().year - 9, datetime.today().year + 1)),
    'Equity Returns (%)': np.random.normal(12, 5, 10).round(2),
    'Debt Returns (%)': np.random.normal(6, 1, 10).round(2)
})
fig_ret = px.bar(returns_df, x="Year", y=["Equity Returns (%)", "Debt Returns (%)"], barmode='group', title="Debt vs Equity Returns Over Last 10 Years")
st.plotly_chart(fig_ret)

# ---------------------- Section 9: Legacy Planning Fund Estimator ----------------------
st.header("🪙 Legacy Planning Estimator")
total_medical_needs = st.number_input("Expected Total Medical Needs Post-Retirement (₹)", 0, 10000000, 1000000, step=100000)
left_after_expenses = max(0, corpus_required - saved_so_far - total_medical_needs)
st.info(f"Estimated Amount for Legacy/Heirs: ₹{left_after_expenses:,.0f}")

# ---------------------- Section 10: Smart Monthly Contribution Planner ----------------------
st.header("🗓️ Smart Monthly Contribution Planner")
monthly_contrib_needed = corpus_required / (12 * years_to_retire)
st.success(f"To meet your goal, you need to invest ₹{monthly_contrib_needed:,.0f} per month for {years_to_retire} years")

# ---------------------- Footer ----------------------
st.markdown("---")
st.markdown("Made with ❤️ for Indian retirees by Joaquin Sam")
