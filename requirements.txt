import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime

# Set page configuration
st.set_page_config(
    page_title="IndiaFD - Compare Fixed Deposit Schemes",
    page_icon="💰",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# Custom CSS for mobile-friendly design with specified color scheme
st.markdown("""
<style>
    .main {
        padding: 1rem 1rem;
    }
    .stApp {
        max-width: 100%;
    }
    h1, h2, h3 {
        text-align: center;
        color: #0A2463; /* Dark blue */
    }
    .bank-card {
        background-color: #ffffff;
        border-radius: 10px;
        padding: 15px;
        margin-bottom: 15px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        border-left: 4px solid #0A2463; /* Dark blue left border */
    }
    .highlight {
        color: #0A2463; /* Dark blue */
        font-weight: bold;
    }
    .sidebar .sidebar-content {
        width: 100%;
    }
    .css-1d391kg {
        padding: 1rem 1rem;
    }
    .stButton>button {
        background-color: #247BA0; /* Light blue */
        color: white;
        border-radius: 5px;
        border: none;
    }
    .stButton>button:hover {
        background-color: #0A2463; /* Dark blue */
    }
    .comparison-btn {
        background-color: #0A2463; /* Dark blue */
        color: white;
        padding: 10px 20px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        border-radius: 5px;
        margin: 10px 0;
        cursor: pointer;
    }
    .compare-checkbox {
        margin: 10px 0;
    }
    .filter-container {
        background-color: #F3F8FB; /* Very light blue */
        padding: 15px;
        border-radius: 10px;
        margin-bottom: 15px;
    }
    table {
        width: 100%;
    }
    th {
        background-color: #247BA0; /* Light blue */
        color: white;
        text-align: left;
        padding: 10px;
    }
    td {
        padding: 8px;
        border-bottom: 1px solid #ddd;
    }
    tr:hover {
        background-color: #f5f5f5;
    }
</style>
""", unsafe_allow_html=True)

# Initialize session state
if 'comparison_list' not in st.session_state:
    st.session_state.comparison_list = []

if 'show_comparison' not in st.session_state:
    st.session_state.show_comparison = False

if 'selected_banks' not in st.session_state:
    st.session_state.selected_banks = []

# Sample data for Indian banks' FD schemes
def load_sample_data():
    data = []
    
    # SBI
    data.append({
        "bank_name": "SBI", 
        "scheme_name": "Regular FD", 
        "min_amount": 1000, 
        "interest_general": 6.5, 
        "interest_senior": 7.0, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 15, 30, 45, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "SBI", 
        "scheme_name": "Tax Saver FD", 
        "min_amount": 1000, 
        "interest_general": 6.7, 
        "interest_senior": 7.2, 
        "early_withdraw": "No",
        "tax_benefits": "Yes",
        "tenures": [1825]
    })
    
    data.append({
        "bank_name": "SBI", 
        "scheme_name": "Special FD", 
        "min_amount": 100000, 
        "interest_general": 7.0, 
        "interest_senior": 7.5, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [1000, 1500]
    })
    
    # HDFC Bank
    data.append({
        "bank_name": "HDFC Bank", 
        "scheme_name": "Regular FD", 
        "min_amount": 5000, 
        "interest_general": 6.6, 
        "interest_senior": 7.1, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 14, 30, 90, 180, 270, 365, 730]
    })
    
    data.append({
        "bank_name": "HDFC Bank", 
        "scheme_name": "Premium FD", 
        "min_amount": 200000, 
        "interest_general": 6.9, 
        "interest_senior": 7.4, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [365, 730, 1095]
    })
    
    # ICICI Bank
    data.append({
        "bank_name": "ICICI Bank", 
        "scheme_name": "Regular FD", 
        "min_amount": 10000, 
        "interest_general": 6.7, 
        "interest_senior": 7.2, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 15, 30, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "ICICI Bank", 
        "scheme_name": "FD Plus", 
        "min_amount": 500000, 
        "interest_general": 7.1, 
        "interest_senior": 7.6, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [365, 730, 1095]
    })
    
    data.append({
        "bank_name": "ICICI Bank", 
        "scheme_name": "Tax Saver FD", 
        "min_amount": 10000, 
        "interest_general": 6.9, 
        "interest_senior": 7.4, 
        "early_withdraw": "No",
        "tax_benefits": "Yes",
        "tenures": [1825]
    })
    
    # Axis Bank
    data.append({
        "bank_name": "Axis Bank", 
        "scheme_name": "Regular FD", 
        "min_amount": 5000, 
        "interest_general": 6.4, 
        "interest_senior": 6.9, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 30, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "Axis Bank", 
        "scheme_name": "Digital FD", 
        "min_amount": 25000, 
        "interest_general": 6.7, 
        "interest_senior": 7.2, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [365, 730]
    })
    
    # Kotak Mahindra Bank
    data.append({
        "bank_name": "Kotak Mahindra Bank", 
        "scheme_name": "Regular FD", 
        "min_amount": 5000, 
        "interest_general": 6.5, 
        "interest_senior": 7.0, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 14, 30, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "Kotak Mahindra Bank", 
        "scheme_name": "FD Ace", 
        "min_amount": 1500000, 
        "interest_general": 6.8, 
        "interest_senior": 7.3, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [390, 730, 1095]
    })
    
    data.append({
        "bank_name": "Kotak Mahindra Bank", 
        "scheme_name": "Tax Saver FD", 
        "min_amount": 10000, 
        "interest_general": 6.6, 
        "interest_senior": 7.1, 
        "early_withdraw": "No",
        "tax_benefits": "Yes",
        "tenures": [1825]
    })
    
    # Bank of Baroda
    data.append({
        "bank_name": "Bank of Baroda", 
        "scheme_name": "Regular FD", 
        "min_amount": 1000, 
        "interest_general": 6.3, 
        "interest_senior": 6.8, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 15, 30, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "Bank of Baroda", 
        "scheme_name": "Tax Saving FD", 
        "min_amount": 5000, 
        "interest_general": 6.5, 
        "interest_senior": 7.0, 
        "early_withdraw": "No",
        "tax_benefits": "Yes",
        "tenures": [1825]
    })
    
    # Punjab National Bank
    data.append({
        "bank_name": "Punjab National Bank", 
        "scheme_name": "Regular FD", 
        "min_amount": 1000, 
        "interest_general": 6.4, 
        "interest_senior": 6.9, 
        "early_withdraw": "Yes",
        "tax_benefits": "No",
        "tenures": [7, 30, 90, 180, 365, 730]
    })
    
    data.append({
        "bank_name": "Punjab National Bank", 
        "scheme_name": "Tax Saver FD", 
        "min_amount": 1000, 
        "interest_general": 6.6, 
        "interest_senior": 7.1, 
        "early_withdraw": "No",
        "tax_benefits": "Yes",
        "tenures": [1825]
    })
    
    # Return as DataFrame
    return pd.DataFrame(data)

# Function to convert tenure days to a readable format
def format_tenure(days):
    if days < 30:
        return f"{days} days"
    elif days < 365:
        months = days // 30
        return f"{months} month{'s' if months > 1 else ''}"
    else:
        years = days // 365
        remaining_days = days % 365
        if remaining_days == 0:
            return f"{years} year{'s' if years > 1 else ''}"
        else:
            months = remaining_days // 30
            if months > 0:
                return f"{years} year{'s' if years > 1 else ''} {months} month{'s' if months > 1 else ''}"
            else:
                return f"{years} year{'s' if years > 1 else ''} {remaining_days} days"

# App header
def display_header():
    st.markdown("<h1>📊 IndiaFD</h1>", unsafe_allow_html=True)
    st.markdown("<h3>Compare Fixed Deposit Schemes Across Indian Banks</h3>", unsafe_allow_html=True)
    
    # Comparison button
    col1, col2, col3 = st.columns([1, 2, 1])
    with col2:
        if st.button("📊 View Comparisons" if not st.session_state.show_comparison else "📋 Back to Listings", 
                   key="comparison_toggle"):
            st.session_state.show_comparison = not st.session_state.show_comparison
    
    st.markdown("---")

# Function to get category for minimum amount
def get_min_amount_category(amount):
    if amount < 20000:
        return "Less than ₹20,000"
    elif amount < 100000:
        return "₹20,000 - ₹1 Lakh"
    elif amount < 1000000:
        return "₹1 Lakh - ₹10 Lakh"
    else:
        return "More than ₹10 Lakh"

# Display search and filter options
def display_filters(df):
    st.markdown("<div class='filter-container'>", unsafe_allow_html=True)
    st.markdown("<h3>Filter Options</h3>", unsafe_allow_html=True)
    
    col1, col2 = st.columns(2)
    
    with col1:
        # Bank filter
        bank_options = ["All Banks"] + sorted(df["bank_name"].unique().tolist())
        selected_bank = st.selectbox("Bank", options=bank_options)
        
        # Minimum amount filter
        min_amount_options = ["Any", "Less than ₹20,000", "₹20,000 - ₹1 Lakh", "₹1 Lakh - ₹10 Lakh", "More than ₹10 Lakh"]
        selected_min_amount = st.selectbox("Minimum Amount", options=min_amount_options)
        
        # Tax benefits filter
        tax_benefits_options = ["Any", "Yes", "No"]
        selected_tax_benefits = st.selectbox("Tax Benefits", options=tax_benefits_options)
    
    with col2:
        # Interest rate range
        interest_min, interest_max = st.slider(
            "Interest Rate Range (%)",
            min_value=5.0,
            max_value=8.0,
            value=(5.0, 8.0),
            step=0.1
        )
        
        # Customer type
        customer_type = st.radio(
            "Customer Type",
            options=["General", "Senior Citizen"],
            horizontal=True
        )
        
        # Early withdraw filter
        early_withdraw_options = ["Any", "Yes", "No"]
        selected_early_withdraw = st.selectbox("Early Withdrawal", options=early_withdraw_options)
    
    st.markdown("</div>", unsafe_allow_html=True)
    
    # Apply filters to dataframe
    filtered_df = df.copy()
    
    # Bank filter
    if selected_bank != "All Banks":
        filtered_df = filtered_df[filtered_df["bank_name"] == selected_bank]
    
    # Minimum amount filter
    if selected_min_amount != "Any":
        if selected_min_amount == "Less than ₹20,000":
            filtered_df = filtered_df[filtered_df["min_amount"] < 20000]
        elif selected_min_amount == "₹20,000 - ₹1 Lakh":
            filtered_df = filtered_df[(filtered_df["min_amount"] >= 20000) & (filtered_df["min_amount"] < 100000)]
        elif selected_min_amount == "₹1 Lakh - ₹10 Lakh":
            filtered_df = filtered_df[(filtered_df["min_amount"] >= 100000) & (filtered_df["min_amount"] < 1000000)]
        elif selected_min_amount == "More than ₹10 Lakh":
            filtered_df = filtered_df[filtered_df["min_amount"] >= 1000000]
    
    # Interest rate filter
    interest_col = "interest_senior" if customer_type == "Senior Citizen" else "interest_general"
    filtered_df = filtered_df[(filtered_df[interest_col] >= interest_min) & (filtered_df[interest_col] <= interest_max)]
    
    # Tax benefits filter
    if selected_tax_benefits != "Any":
        filtered_df = filtered_df[filtered_df["tax_benefits"] == selected_tax_benefits]
    
    # Early withdraw filter
    if selected_early_withdraw != "Any":
        filtered_df = filtered_df[filtered_df["early_withdraw"] == selected_early_withdraw]
    
    return filtered_df, customer_type

# Display FD schemes in a table with sorting capability
def display_fd_schemes(df, customer_type):
    interest_col = "interest_senior" if customer_type == "Senior Citizen" else "interest_general"
    
    # Create a clean display dataframe
    display_df = df.copy()
    display_df["Interest Rate"] = display_df[interest_col].apply(lambda x: f"{x}%")
    display_df["Minimum Amount"] = display_df["min_amount"].apply(lambda x: f"₹{x:,}")
    
    # Set up columns for display
    display_columns = {
        "bank_name": "Bank",
        "scheme_name": "Scheme",
        "Minimum Amount": "Minimum Amount",
        "Interest Rate": "Interest Rate",
        "early_withdraw": "Early Withdrawal",
        "tax_benefits": "Tax Benefits"
    }
    
    # Create a clean version for display
    clean_df = display_df[[*display_columns.keys()]].copy()
    clean_df.columns = [*display_columns.values()]
    
    # Allow sorting
    sorted_df = st.dataframe(
        clean_df,
        use_container_width=True,
        column_config={
            "Bank": st.column_config.TextColumn("Bank"),
            "Scheme": st.column_config.TextColumn("Scheme"),
            "Minimum Amount": st.column_config.TextColumn("Minimum Amount"),
            "Interest Rate": st.column_config.TextColumn("Interest Rate"),
            "Early Withdrawal": st.column_config.TextColumn("Early Withdrawal"),
            "Tax Benefits": st.column_config.TextColumn("Tax Benefits")
        },
        hide_index=True
    )
    
    st.markdown("---")
    
    # Display schemes as cards with comparison buttons
    st.markdown("<h3>FD Schemes</h3>", unsafe_allow_html=True)
    
    for i, row in df.iterrows():
        scheme_id = f"{row['bank_name']}_{row['scheme_name']}"
        is_in_comparison = scheme_id in st.session_state.comparison_list
        
        st.markdown(f"""
        <div class="bank-card">
            <h3>{row['bank_name']} - {row['scheme_name']}</h3>
            <p>Interest Rate: <span class="highlight">{row[interest_col]}%</span></p>
            <p>Minimum Amount: ₹{row['min_amount']:,}</p>
            <p>Early Withdrawal: {row['early_withdraw']}</p>
            <p>Tax Benefits: {row['tax_benefits']}</p>
            <p>Available Tenures: {', '.join(format_tenure(t) for t in row['tenures'])}</p>
        </div>
        """, unsafe_allow_html=True)
        
        # Add comparison checkbox
        if st.checkbox(
            "Add to Compare" if not is_in_comparison else "Remove from Compare", 
            value=is_in_comparison,
            key=f"compare_{i}"
        ):
            if scheme_id not in st.session_state.comparison_list:
                st.session_state.comparison_list.append(scheme_id)
        else:
            if scheme_id in st.session_state.comparison_list:
                st.session_state.comparison_list.remove(scheme_id)

# Display comparison view
def display_comparison_view(df, customer_type):
    interest_col = "interest_senior" if customer_type == "Senior Citizen" else "interest_general"
    
    if not st.session_state.comparison_list:
        st.warning("No schemes added to comparison. Please add schemes to compare.")
        return
    
    st.markdown("<h3>Scheme Comparison</h3>", unsafe_allow_html=True)
    
    # Filter data for comparison
    comparison_schemes = []
    for scheme_id in st.session_state.comparison_list:
        bank_name, scheme_name = scheme_id.split('_', 1)
        scheme = df[(df["bank_name"] == bank_name) & (df["scheme_name"] == scheme_name)]
        if not scheme.empty:
            comparison_schemes.append(scheme.iloc[0])
    
    if not comparison_schemes:
        st.warning("Selected schemes not found in the filtered data.")
        return
    
    # Create comparison table
    comparison_data = []
    for scheme in comparison_schemes:
        comparison_data.append({
            "Bank": scheme["bank_name"],
            "Scheme": scheme["scheme_name"],
            "Interest Rate": f"{scheme[interest_col]}%",
            "Minimum Amount": f"₹{scheme['min_amount']:,}",
            "Early Withdrawal": scheme["early_withdraw"],
            "Tax Benefits": scheme["tax_benefits"],
            "Tenures": ", ".join(format_tenure(t) for t in scheme["tenures"])
        })
    
    st.table(pd.DataFrame(comparison_data))
    
    # Create comparison chart
    st.markdown("<h3>Interest Rate Comparison</h3>", unsafe_allow_html=True)
    
    chart_data = pd.DataFrame({
        "Scheme": [f"{scheme['bank_name']} - {scheme['scheme_name']}" for scheme in comparison_schemes],
        "Interest Rate": [scheme[interest_col] for scheme in comparison_schemes]
    })
    
    fig, ax = plt.subplots(figsize=(10, 6))
    bars = sns.barplot(data=chart_data, x="Scheme", y="Interest Rate", palette="Blues_d", ax=ax)
    
    # Add value labels on bars
    for i, bar in enumerate(bars.patches):
        bars.text(
            bar.get_x() + bar.get_width()/2.,
            bar.get_height() + 0.1,
            f"{chart_data['Interest Rate'].iloc[i]}%",
            ha='center', va='bottom', color='#0A2463', fontweight='bold'
        )
    
    plt.title(f"FD Interest Rates Comparison ({customer_type})")
    plt.ylabel("Interest Rate (%)")
    plt.xticks(rotation=45, ha='right')
    plt.tight_layout()
    
    st.pyplot(fig)
    
    # Clear comparison list button
    if st.button("Clear All Comparisons"):
        st.session_state.comparison_list = []
        st.experimental_rerun()

# FD calculator
def display_fd_calculator():
    st.markdown("<h3>FD Calculator</h3>", unsafe_allow_html=True)
    
    col1, col2 = st.columns(2)
    
    with col1:
        principal = st.number_input("Principal Amount (₹)", min_value=1000, value=10000, step=1000)
        interest_rate = st.number_input("Interest Rate (%)", min_value=1.0, max_value=10.0, value=6.5, step=0.1)
    
    with col2:
        tenure_years = st.number_input("Tenure (Years)", min_value=0.0, max_value=10.0, value=1.0, step=0.25)
        compounding = st.selectbox("Compounding Frequency", ["Annually", "Half-Yearly", "Quarterly", "Monthly"])
    
    # Calculate maturity amount
    n = {"Annually": 1, "Half-Yearly": 2, "Quarterly": 4, "Monthly": 12}[compounding]
    t = tenure_years
    r = interest_rate / 100
    
    maturity_amount = principal * (1 + r/n)**(n*t)
    interest_earned = maturity_amount - principal
    
    col1, col2 = st.columns(2)
    
    with col1:
        st.metric("Maturity Amount", f"₹{maturity_amount:.2f}")
    
    with col2:
        st.metric("Interest Earned", f"₹{interest_earned:.2f}")

# Main app function
def main():
    display_header()
    
    # Load sample data
    df = load_sample_data()
    
    # Display filters and get filtered data
    filtered_df, customer_type = display_filters(df)
    
    # Show either comparison view or FD schemes based on state
    if st.session_state.show_comparison:
        display_comparison_view(filtered_df, customer_type)
    else:
        display_fd_schemes(filtered_df, customer_type)
    
    # Always show calculator at the bottom
    st.markdown("---")
    display_fd_calculator()
    
    # Footer
    st.markdown("---")
    st.markdown("""
    <div style="text-align: center; color: #0A2463;">
        <p>© 2025 IndiaFD - Find the best Fixed Deposit rates in India</p>
        <p>Disclaimer: The interest rates shown are for illustrative purposes only and may not reflect current rates.</p>
    </div>
    """, unsafe_allow_html=True)

if __name__ == "__main__":
    main()
