import streamlit as st
import pandas as pd
from io import StringIO
import numpy as np

# Set page config
st.set_page_config(
    page_title="Recovery Team Incentive Calculator",
    page_icon="💰",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom CSS for better styling
st.markdown("""
<style>
    .metric-card {
        background: linear-gradient(135deg, #667eea, #764ba2);
        color: white;
        padding: 20px;
        border-radius: 12px;
        text-align: center;
        margin-bottom: 10px;
    }
    .warning-box {
        background-color: #fff3cd;
        border: 1px solid #ffeaa7;
        color: #856404;
        padding: 15px;
        border-radius: 8px;
        margin: 10px 0;
    }
    .success-box {
        background-color: #d4edda;
        border: 1px solid #c3e6cb;
        color: #155724;
        padding: 15px;
        border-radius: 8px;
        margin: 10px 0;
    }
    .stButton > button {
        width: 100%;
        background: linear-gradient(135deg, #667eea, #764ba2);
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 8px;
        font-weight: bold;
    }
</style>
""", unsafe_allow_html=True)

# Initialize session state
if 'num_managers' not in st.session_state:
    st.session_state.num_managers = 5
if 'exec_per_manager' not in st.session_state:
    st.session_state.exec_per_manager = 5
if 'recovery_data' not in st.session_state:
    st.session_state.recovery_data = {}

# Helper functions
def format_currency(amount):
    """Format amount as Indian currency"""
    return f"₹{amount:,.0f}"

def calculate_incentives(recovery_data, monthly_target, min_incentive_limit, is_rupees_mode, num_managers, exec_per_manager):
    """Calculate all incentives based on recovery data"""
    
    # Convert to lakhs for internal calculations
    if is_rupees_mode:
        monthly_target_lakhs = monthly_target / 100000
        min_limit_lakhs = min_incentive_limit / 100000
    else:
        monthly_target_lakhs = monthly_target
        min_limit_lakhs = min_incentive_limit
    
    total_recovery = 0
    manager_recoveries = []
    
    # Calculate total recovery and individual recoveries
    for i in range(1, num_managers + 1):
        manager_total = 0
        executive_recoveries = []
        
        for j in range(1, exec_per_manager + 1):
            exec_id = f"manager_{i}_exec_{j}"
            recovery = recovery_data.get(exec_id, 0)
            
            # Convert to lakhs for internal calculations
            if is_rupees_mode and recovery > 0:
                recovery = recovery / 100000
            
            executive_recoveries.append(recovery)
            manager_total += recovery
        
        manager_recoveries.append({
            'total': manager_total,
            'executives': executive_recoveries
        })
        total_recovery += manager_total
    
    # Check minimum limit
    if total_recovery < min_limit_lakhs:
        return {
            'eligible': False,
            'total_recovery': total_recovery * 100000,
            'target_achievement': (total_recovery / monthly_target_lakhs) * 100 if monthly_target_lakhs > 0 else 0,
            'warning_message': f"Minimum recovery of ₹{min_limit_lakhs:.2f} lakhs must be achieved for incentives to be paid. Current: ₹{total_recovery:.2f} lakhs",
            'executive_incentives': {},
            'manager_incentives': {},
            'head_incentive': 0,
            'total_executive_incentive': 0,
            'total_manager_incentive': 0,
            'manager_recoveries': manager_recoveries
        }
    
    # Calculate incentive pool (20% of total recovery)
    total_pool_amount = total_recovery * 0.20
    
    # Allocate pool between levels
    executive_pool = total_pool_amount * 0.60  # 60%
    manager_pool = total_pool_amount * 0.25    # 25%
    head_pool = total_pool_amount * 0.15       # 15%
    
    executive_incentives = {}
    manager_incentives = {}
    total_executive_incentive = 0
    total_manager_incentive = 0
    
    # Calculate Executive Incentives
    for i in range(1, num_managers + 1):
        manager_total = manager_recoveries[i-1]['total']
        
        if manager_total > 0 and total_recovery > 0:
            # This manager's team gets their proportional share of executive pool
            manager_executive_pool = executive_pool * (manager_total / total_recovery)
            
            # Each executive gets based on their individual recovery within their team
            for j in range(1, exec_per_manager + 1):
                exec_id = f"manager_{i}_exec_{j}"
                exec_recovery = manager_recoveries[i-1]['executives'][j-1]
                
                exec_incentive = 0
                if exec_recovery > 0:
                    # Executive gets their proportional share of manager's executive pool
                    exec_incentive = manager_executive_pool * (exec_recovery / manager_total)
                
                executive_incentives[exec_id] = exec_incentive * 100000  # Convert back to rupees
                total_executive_incentive += exec_incentive
    
    # Calculate Manager Incentives
    for i in range(1, num_managers + 1):
        manager_total = manager_recoveries[i-1]['total']
        manager_incentive = 0
        
        if total_recovery > 0:
            # Manager gets their proportional share of manager pool
            manager_incentive = manager_pool * (manager_total / total_recovery)
        
        manager_incentives[f"manager_{i}"] = manager_incentive * 100000  # Convert back to rupees
        total_manager_incentive += manager_incentive
    
    # Calculate Head Incentive
    head_incentive = head_pool
    
    return {
        'eligible': True,
        'total_recovery': total_recovery * 100000,
        'target_achievement': (total_recovery / monthly_target_lakhs) * 100 if monthly_target_lakhs > 0 else 0,
        'executive_incentives': executive_incentives,
        'manager_incentives': manager_incentives,
        'head_incentive': head_incentive * 100000,
        'total_executive_incentive': total_executive_incentive * 100000,
        'total_manager_incentive': total_manager_incentive * 100000,
        'manager_recoveries': manager_recoveries,
        'total_pool': total_pool_amount * 100000
    }

# Main app
def main():
    st.title("🏦 Recovery Team Incentive Calculator")
    st.markdown("**NPA Collection Team | 90+ Days Overdue Accounts**")
    st.markdown("---")
    
    # Sidebar configuration
    st.sidebar.header("⚙️ Configuration")
    
    # Team structure
    st.sidebar.subheader("Team Structure")
    num_managers = st.sidebar.number_input(
        "Number of Managers", 
        min_value=1, max_value=10, 
        value=st.session_state.num_managers,
        help="Number of collection managers (1-10)"
    )
    exec_per_manager = st.sidebar.number_input(
        "Executives per Manager", 
        min_value=1, max_value=15, 
        value=st.session_state.exec_per_manager,
        help="Number of executives under each manager (1-15)"
    )
    
    # Update session state if changed
    if num_managers != st.session_state.num_managers or exec_per_manager != st.session_state.exec_per_manager:
        st.session_state.num_managers = num_managers
        st.session_state.exec_per_manager = exec_per_manager
        st.session_state.recovery_data = {}  # Reset data when structure changes
    
    total_executives = num_managers * exec_per_manager
    total_team_size = 1 + num_managers + total_executives
    
    st.sidebar.info(f"**Team Size:** {total_team_size} members\n\n"
                   f"• 1 Head\n"
                   f"• {num_managers} Managers\n"
                   f"• {total_executives} Executives")
    
    # Target settings
    st.sidebar.subheader("Target Settings")
    currency_mode = st.sidebar.radio(
        "Currency Mode",
        options=["Lakhs", "Rupees"],
        index=0,
        help="Choose input currency format"
    )
    is_rupees_mode = currency_mode == "Rupees"
    
    if is_rupees_mode:
        default_target = 3100000
        default_min = 3100000
        currency_label = "₹ Rupees"
    else:
        default_target = 31
        default_min = 31
        currency_label = "₹ Lakhs"
    
    monthly_target = st.sidebar.number_input(
        f"Monthly Recovery Target ({currency_label})",
        min_value=0.01 if not is_rupees_mode else 1000,
        value=float(default_target),
        step=0.01 if not is_rupees_mode else 1000.0,
        help="Monthly recovery target for the team"
    )
    
    min_incentive_limit = st.sidebar.number_input(
        f"Minimum Recovery for Incentive ({currency_label})",
        min_value=0.0 if not is_rupees_mode else 0,
        value=float(default_min),
        step=0.01 if not is_rupees_mode else 1000.0,
        help="Minimum recovery required to be eligible for incentives"
    )
    
    # Quick fill options
    st.sidebar.subheader("🚀 Quick Fill Options")
    col1, col2 = st.sidebar.columns(2)
    
    with col1:
        if st.button("High Performance", help="Fill with high performance sample data"):
            fill_sample_data("high", is_rupees_mode)
        if st.button("Minimum Target", help="Fill with minimum target data"):
            fill_sample_data("minimum", is_rupees_mode)
    
    with col2:
        if st.button("Medium Performance", help="Fill with medium performance data"):
            fill_sample_data("medium", is_rupees_mode)
        if st.button("Clear All", help="Clear all recovery data"):
            st.session_state.recovery_data = {}
            st.experimental_rerun()
    
    # Main content area
    col1, col2, col3, col4 = st.columns(4)
    
    # Calculate incentives with current data
    results = calculate_incentives(
        st.session_state.recovery_data,
        monthly_target,
        min_incentive_limit,
        is_rupees_mode,
        num_managers,
        exec_per_manager
    )
    
    # Summary cards
    with col1:
        st.metric(
            label="Total Recovery",
            value=format_currency(results['total_recovery']),
            delta=f"{results['target_achievement']:.1f}% of target"
        )
    
    with col2:
        st.metric(
            label="Total Executives Incentive",
            value=format_currency(results['total_executive_incentive'])
        )
    
    with col3:
        st.metric(
            label="Total Managers Incentive",
            value=format_currency(results['total_manager_incentive'])
        )
    
    with col4:
        st.metric(
            label="Collection Head Incentive",
            value=format_currency(results['head_incentive'])
        )
    
    # Warning or success message
    if not results['eligible']:
        st.markdown(f"""
        <div class="warning-box">
            <strong>⚠️ Note:</strong> {results['warning_message']}
        </div>
        """, unsafe_allow_html=True)
    else:
        unit = "rupees" if is_rupees_mode else "lakhs"
        display_total = results['total_recovery'] if is_rupees_mode else results['total_recovery'] / 100000
        st.markdown(f"""
        <div class="success-box">
            <strong>✅ Great!</strong> Team achieved ₹{display_total:,.0f} {unit} recovery ({results['target_achievement']:.1f}% of target). Incentives calculated successfully!
        </div>
        """, unsafe_allow_html=True)
    
    st.markdown("---")
    
    # Recovery input section
    st.header("📝 Enter Collection Amounts")
    st.markdown(f"Enter the recovery amount (in {currency_label.lower()}) for each executive:")
    
    # Create tabs for each manager
    manager_tabs = st.tabs([f"Manager {i+1}" for i in range(num_managers)])
    
    for i, tab in enumerate(manager_tabs):
        with tab:
            st.subheader(f"Manager {i+1} & Team")
            
            # Create columns for executives
            cols = st.columns(min(exec_per_manager, 3))  # Max 3 columns for better layout
            
            for j in range(exec_per_manager):
                with cols[j % len(cols)]:
                    exec_id = f"manager_{i+1}_exec_{j+1}"
                    current_value = st.session_state.recovery_data.get(exec_id, 0.0)
                    
                    recovery_amount = st.number_input(
                        f"Executive {j+1}",
                        min_value=0.0,
                        value=float(current_value),
                        step=0.01 if not is_rupees_mode else 1000.0,
                        key=exec_id,
                        help=f"Recovery amount for Executive {j+1}"
                    )
                    
                    st.session_state.recovery_data[exec_id] = recovery_amount
                    
                    # Show individual incentive
                    if results['eligible']:
                        incentive = results['executive_incentives'].get(exec_id, 0)
                        st.success(f"Incentive: {format_currency(incentive)}")
                    else:
                        st.info("Incentive: ₹0")
            
            # Show manager totals
            manager_recovery = sum([
                st.session_state.recovery_data.get(f"manager_{i+1}_exec_{j+1}", 0) 
                for j in range(exec_per_manager)
            ])
            
            col_a, col_b = st.columns(2)
            with col_a:
                st.metric(f"Manager {i+1} Recovery", format_currency(manager_recovery))
            
            with col_b:
                if results['eligible']:
                    manager_incentive = results['manager_incentives'].get(f"manager_{i+1}", 0)
                    st.metric(f"Manager {i+1} Incentive", format_currency(manager_incentive))
                else:
                    st.metric(f"Manager {i+1} Incentive", "₹0")
    
    # Export section
    st.markdown("---")
    st.header("📊 Export & Reports")
    
    col1, col2, col3 = st.columns(3)
    
    with col1:
        if st.button("📄 Generate CSV Report"):
            csv_data = generate_csv_report(results, num_managers, exec_per_manager, is_rupees_mode)
            st.download_button(
                label="Download CSV",
                data=csv_data,
                file_name="incentive_report.csv",
                mime="text/csv"
            )
    
    with col2:
        if st.button("📋 Show Detailed Report"):
            show_detailed_report(results, num_managers, exec_per_manager)
    
    with col3:
        if st.button("🔄 Reset All Data"):
            st.session_state.recovery_data = {}
            st.experimental_rerun()

def fill_sample_data(data_type, is_rupees_mode):
    """Fill sample data based on type"""
    sample_configs = {
        'high': {'base': 3, 'variance': 2},
        'medium': {'base': 2, 'variance': 1.5},
        'minimum': {'base': 1.5, 'variance': 1},
        'below': {'base': 1, 'variance': 0.5}
    }
    
    config = sample_configs.get(data_type, sample_configs['medium'])
    
    for i in range(1, st.session_state.num_managers + 1):
        for j in range(1, st.session_state.exec_per_manager + 1):
            exec_id = f"manager_{i}_exec_{j}"
            # Generate random value within range
            random_value = config['base'] + np.random.uniform(-config['variance'], config['variance'])
            final_value = max(0, random_value)  # Ensure non-negative
            
            # Convert to rupees if needed
            if is_rupees_mode:
                final_value = final_value * 100000
            
            st.session_state.recovery_data[exec_id] = round(final_value, 2)
    
    st.experimental_rerun()

def generate_csv_report(results, num_managers, exec_per_manager):
    """Generate CSV report data"""
    data = []
    
    # Add summary
    data.append(['SUMMARY', '', '', ''])
    data.append(['Total Recovery', format_currency(results['total_recovery']), '', ''])
    data.append(['Target Achievement', f"{results['target_achievement']:.1f}%", '', ''])
    data.append(['Total Executive Incentives', format_currency(results['total_executive_incentive']), '', ''])
    data.append(['Total Manager Incentives', format_currency(results['total_manager_incentive']), '', ''])
    data.append(['Head Incentive', format_currency(results['head_incentive']), '', ''])
    data.append(['', '', '', ''])
    
    # Add executive details
    data.append(['EXECUTIVE DETAILS', '', '', ''])
    data.append(['Manager', 'Executive', 'Recovery', 'Incentive'])
    
    if results['eligible']:
        for i in range(1, num_managers + 1):
            for j in range(1, exec_per_manager + 1):
                exec_id = f"manager_{i}_exec_{j}"
                recovery = st.session_state.recovery_data.get(exec_id, 0)
                incentive = results['executive_incentives'].get(exec_id, 0)
                data.append([f'Manager {i}', f'Executive {j}', f'{recovery}', format_currency(incentive)])
    
    # Add manager details
    data.append(['', '', '', ''])
    data.append(['MANAGER DETAILS', '', '', ''])
    data.append(['Manager', 'Team Recovery', 'Incentive', ''])
    
    if results['eligible']:
        for i in range(1, num_managers + 1):
            manager_recovery = sum([
                st.session_state.recovery_data.get(f"manager_{i}_exec_{j+1}", 0) 
                for j in range(exec_per_manager)
            ])
            manager_incentive = results['manager_incentives'].get(f"manager_{i}", 0)
            data.append([f'Manager {i}', f'{manager_recovery}', format_currency(manager_incentive), ''])
    
    # Convert to CSV
    output = StringIO()
    for row in data:
        output.write(','.join(map(str, row)) + '\n')
    
    return output.getvalue()

def show_detailed_report(results, num_managers, exec_per_manager):
    """Show detailed report"""
    st.subheader("📋 Detailed Incentive Report")
    
    if results['eligible']:
        # Executive report
        st.write("**Executive Incentives:**")
        exec_data = []
        for i in range(1, num_managers + 1):
            for j in range(1, exec_per_manager + 1):
                exec_id = f"manager_{i}_exec_{j}"
                recovery = st.session_state.recovery_data.get(exec_id, 0)
                incentive = results['executive_incentives'].get(exec_id, 0)
                exec_data.append({
                    'Manager': f'Manager {i}',
                    'Executive': f'Executive {j}',
                    'Recovery': format_currency(recovery),
                    'Incentive': format_currency(incentive)
                })
        
        st.dataframe(pd.DataFrame(exec_data), use_container_width=True)
        
        # Manager report
        st.write("**Manager Incentives:**")
        mgr_data = []
        for i in range(1, num_managers + 1):
            manager_recovery = sum([
                st.session_state.recovery_data.get(f"manager_{i}_exec_{j+1}", 0) 
                for j in range(exec_per_manager)
            ])
            manager_incentive = results['manager_incentives'].get(f"manager_{i}", 0)
            mgr_data.append({
                'Manager': f'Manager {i}',
                'Team Recovery': format_currency(manager_recovery),
                'Incentive': format_currency(manager_incentive)
            })
        
        st.dataframe(pd.DataFrame(mgr_data), use_container_width=True)
        
        # Summary
        st.write("**Summary:**")
        summary_data = {
            'Metric': ['Total Recovery', 'Total Executive Incentives', 'Total Manager Incentives', 'Head Incentive', 'Total Incentive Pool'],
            'Amount': [
                format_currency(results['total_recovery']),
                format_currency(results['total_executive_incentive']),
                format_currency(results['total_manager_incentive']),
                format_currency(results['head_incentive']),
                format_currency(results.get('total_pool', 0))
            ]
        }
        st.dataframe(pd.DataFrame(summary_data), use_container_width=True)
    else:
        st.warning("No incentives calculated. Minimum recovery target not met.")

if __name__ == "__main__":
    main()
