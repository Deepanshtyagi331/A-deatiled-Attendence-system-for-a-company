import streamlit as st
import sqlite3
import pandas as pd
from datetime import datetime, date
import plotly.express as px
import plotly.graph_objects as go
from streamlit_option_menu import option_menu
from qr_scanner_simple import qr_scanner_component

# Page configuration
st.set_page_config(
    page_title="Senture Perfumes - Employee Attendance System",
    page_icon="🌸",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom CSS for better styling
st.markdown("""
<style>
    /* Main Header with hover effect */
    .main-header {
        background: linear-gradient(90deg, #FF6B6B, #4ECDC4);
        padding: 1rem;
        border-radius: 10px;
        color: white;
        text-align: center;
        margin-bottom: 2rem;
        transition: all 0.3s ease;
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
    }
    
    .main-header:hover {
        transform: translateY(-2px);
        box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
        background: linear-gradient(90deg, #FF5252, #26A69A);
    }
    
    /* Attendance Card with hover effect */
    .attendance-card {
        background: white;
        padding: 1.5rem;
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        margin-bottom: 1rem;
        transition: all 0.3s ease;
        border: 2px solid transparent;
    }
    
    .attendance-card:hover {
        transform: translateY(-3px);
        box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
        border-color: #4ECDC4;
    }
    
    /* Success Message with hover effect */
    .success-message {
        background: #d4edda;
        color: #155724;
        padding: 1rem;
        border-radius: 5px;
        border: 1px solid #c3e6cb;
        margin: 1rem 0;
        transition: all 0.3s ease;
        cursor: pointer;
    }
    
    .success-message:hover {
        background: #c3e6cb;
        transform: scale(1.02);
        box-shadow: 0 4px 12px rgba(21, 87, 36, 0.2);
    }
    
    /* Error Message with hover effect */
    .error-message {
        background: #f8d7da;
        color: #721c24;
        padding: 1rem;
        border-radius: 5px;
        border: 1px solid #f5c6cb;
        margin: 1rem 0;
        transition: all 0.3s ease;
        cursor: pointer;
    }
    
    .error-message:hover {
        background: #f5c6cb;
        transform: scale(1.02);
        box-shadow: 0 4px 12px rgba(114, 28, 36, 0.2);
    }
    
    /* Metric Cards with hover effects */
    .metric-card {
        background: white;
        padding: 1.5rem;
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        margin: 0.5rem;
        transition: all 0.3s ease;
        border: 2px solid transparent;
        cursor: pointer;
    }
    
    .metric-card:hover {
        transform: translateY(-5px);
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
        border-color: #FF6B6B;
        background: linear-gradient(135deg, #f8f9fa, #e9ecef);
    }
    
    /* Form elements with hover effects */
    .stForm {
        transition: all 0.3s ease;
    }
    
    .stForm:hover {
        transform: translateY(-2px);
    }
    
    /* Button hover effects */
    .stButton > button {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
        font-weight: 600 !important;
    }
    
    .stButton > button:hover {
        transform: translateY(-2px) !important;
        box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2) !important;
        background: linear-gradient(135deg, #FF6B6B, #FF5252) !important;
        color: white !important;
    }
    
    /* Input fields with hover effects */
    .stTextInput > div > div > input {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
    }
    
    .stTextInput > div > div > input:hover {
        border-color: #4ECDC4 !important;
        box-shadow: 0 0 0 2px rgba(78, 205, 196, 0.2) !important;
    }
    
    .stTextInput > div > div > input:focus {
        border-color: #4ECDC4 !important;
        box-shadow: 0 0 0 3px rgba(78, 205, 196, 0.3) !important;
    }
    
    /* Selectbox with hover effects */
    .stSelectbox > div > div > div {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
    }
    
    .stSelectbox > div > div > div:hover {
        border-color: #4ECDC4 !important;
        box-shadow: 0 0 0 2px rgba(78, 205, 196, 0.2) !important;
    }
    
    /* Date input with hover effects */
    .stDateInput > div > div > input {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
    }
    
    .stDateInput > div > div > input:hover {
        border-color: #4ECDC4 !important;
        box-shadow: 0 0 0 2px rgba(78, 205, 196, 0.2) !important;
    }
    
    /* Download button with hover effects */
    .stDownloadButton > button {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
        font-weight: 600 !important;
    }
    
    .stDownloadButton > button:hover {
        transform: translateY(-2px) !important;
        box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2) !important;
        background: linear-gradient(135deg, #4ECDC4, #26A69A) !important;
        color: white !important;
    }
    
    /* Sidebar with hover effects */
    .css-1d391kg {
        transition: all 0.3s ease !important;
    }
    
    .css-1d391kg:hover {
        background: rgba(255, 107, 107, 0.1) !important;
    }
    
    /* Navigation menu items with hover effects */
    .css-1d391kg .css-1d391kg {
        transition: all 0.3s ease !important;
        border-radius: 8px !important;
        margin: 2px 0 !important;
    }
    
    .css-1d391kg .css-1d391kg:hover {
        background: rgba(78, 205, 196, 0.2) !important;
        transform: translateX(5px) !important;
    }
    
    /* Dataframe with hover effects */
    .dataframe {
        transition: all 0.3s ease !important;
    }
    
    .dataframe:hover {
        box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1) !important;
    }
    
    /* Plotly charts with hover effects */
    .js-plotly-plot {
        transition: all 0.3s ease !important;
        border-radius: 10px !important;
    }
    
    .js-plotly-plot:hover {
        transform: scale(1.02) !important;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15) !important;
    }
    
    /* Custom hover animations */
    @keyframes pulse {
        0% { transform: scale(1); }
        50% { transform: scale(1.05); }
        100% { transform: scale(1); }
    }
    
    .pulse-hover:hover {
        animation: pulse 0.6s ease-in-out;
    }
    
    /* Smooth transitions for all elements */
    * {
        transition: all 0.2s ease;
    }
</style>
""", unsafe_allow_html=True)

# Initialize database
def init_database():
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    # Create employees table
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS employees (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            user_id TEXT UNIQUE NOT NULL,
            email TEXT,
            department TEXT,
            created_date DATE DEFAULT CURRENT_DATE
        )
    ''')
    
    # Create attendance table with overtime support
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS attendance (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            employee_id INTEGER,
            date DATE NOT NULL,
            entry_time TIME,
            exit_time TIME,
            total_hours REAL,
            overtime_hours REAL DEFAULT 0,
            status TEXT DEFAULT 'Present',
            is_overtime BOOLEAN DEFAULT 0,
            FOREIGN KEY (employee_id) REFERENCES employees (id)
        )
    ''')
    
    # Add missing columns if they don't exist (database migration)
    try:
        cursor.execute('ALTER TABLE attendance ADD COLUMN overtime_hours REAL DEFAULT 0')
    except:
        pass  # Column already exists
    
    try:
        cursor.execute('ALTER TABLE attendance ADD COLUMN is_overtime BOOLEAN DEFAULT 0')
    except:
        pass  # Column already exists
    
    conn.commit()
    conn.close()

# Initialize database
init_database()

# Clean up any existing invalid data
def cleanup_database():
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    try:
        # Update any NULL or empty total_hours to 0
        cursor.execute('''
            UPDATE attendance 
            SET total_hours = 0 
            WHERE total_hours IS NULL OR total_hours = ''
        ''')
        
        # Update any invalid time entries
        cursor.execute('''
            UPDATE attendance 
            SET entry_time = NULL, exit_time = NULL, total_hours = 0
            WHERE entry_time = '' OR exit_time = ''
        ''')
        
        # Update existing records with default overtime values
        cursor.execute('''
            UPDATE attendance 
            SET overtime_hours = 0, is_overtime = 0
            WHERE overtime_hours IS NULL OR is_overtime IS NULL
        ''')
        
        conn.commit()
    except Exception as e:
        pass  # Ignore errors during cleanup
    finally:
        conn.close()

# Run cleanup
cleanup_database()

# Database functions
def add_employee(name, user_id, email, department):
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    try:
        cursor.execute('''
            INSERT INTO employees (name, user_id, email, department)
            VALUES (?, ?, ?, ?)
        ''', (name, user_id, email, department))
        conn.commit()
        return True
    except sqlite3.IntegrityError:
        return False
    finally:
        conn.close()

def get_employee_by_user_id(user_id):
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM employees WHERE user_id = ?', (user_id,))
    employee = cursor.fetchone()
    conn.close()
    return employee

def mark_entry(employee_id, date, entry_time):
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    # Determine attendance status based on entry time
    entry_hour = int(entry_time.split(':')[0])
    entry_minute = int(entry_time.split(':')[1])
    entry_time_minutes = entry_hour * 60 + entry_minute
    
    # Define time thresholds (in minutes from midnight)
    on_time_threshold = 10 * 60  # 10:00 AM
    late_threshold = 10 * 60      # 10:00 AM (same as on_time for half day)
    
    # Determine status
    if entry_time_minutes < on_time_threshold:
        status = "Present"
        message = "Entry time marked successfully - Present"
    elif entry_time_minutes == on_time_threshold:
        status = "Half Day"
        message = "Entry time marked successfully - Half Day (Late arrival)"
    else:
        status = "Absent"
        message = "Entry time marked successfully - Absent (Very late arrival)"
    
    # Check if attendance already exists for today
    cursor.execute('''
        SELECT * FROM attendance 
        WHERE employee_id = ? AND date = ?
    ''', (employee_id, date))
    
    existing = cursor.fetchone()
    
    if existing:
        if existing[3]:  # If entry_time exists
            return False, "Entry already marked for today"
        else:
            # Update entry time and status
            cursor.execute('''
                UPDATE attendance 
                SET entry_time = ?, status = ?
                WHERE employee_id = ? AND date = ?
            ''', (entry_time, status, employee_id, date))
    else:
        # Create new attendance record with status
        cursor.execute('''
            INSERT INTO attendance (employee_id, date, entry_time, status)
            VALUES (?, ?, ?, ?)
        ''', (employee_id, date, entry_time, status))
    
    conn.commit()
    conn.close()
    return True, message

def mark_exit(employee_id, date, exit_time):
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    # Get entry time
    cursor.execute('''
        SELECT entry_time FROM attendance 
        WHERE employee_id = ? AND date = ?
    ''', (employee_id, date))
    
    result = cursor.fetchone()
    if not result or not result[0]:
        conn.close()
        return False, "No entry time found for today"
    
    entry_time = result[0]
    
    try:
        # Calculate total hours
        entry_dt = datetime.strptime(f"{date} {entry_time}", "%Y-%m-%d %H:%M")
        exit_dt = datetime.strptime(f"{date} {exit_time}", "%Y-%m-%d %H:%M")
        
        # Ensure exit time is after entry time
        if exit_dt <= entry_dt:
            conn.close()
            return False, "Exit time must be after entry time"
        
        total_hours = (exit_dt - entry_dt).total_seconds() / 3600
        
        # Calculate overtime (anything after 6 PM = 18:00)
        standard_end_time = datetime.strptime(f"{date} 18:00", "%Y-%m-%d %H:%M")
        overtime_hours = 0
        is_overtime = False
        
        if exit_dt > standard_end_time:
            overtime_hours = (exit_dt - standard_end_time).total_seconds() / 3600
            is_overtime = True
        
        # Update exit time, total hours, overtime hours, and overtime flag
        cursor.execute('''
            UPDATE attendance 
            SET exit_time = ?, total_hours = ?, overtime_hours = ?, is_overtime = ?
            WHERE employee_id = ? AND date = ?
        ''', (exit_time, total_hours, overtime_hours, is_overtime, employee_id, date))
        
        conn.commit()
        conn.close()
        
        message = f"Exit time marked successfully. Total hours: {total_hours:.2f}"
        if is_overtime:
            message += f" | Overtime: {overtime_hours:.2f} hours"
        
        return True, message
    except ValueError as e:
        conn.close()
        return False, f"Time format error: {str(e)}"

def auto_mark_exit_after_6pm():
    """Automatically mark exit for all employees who haven't marked exit after 6 PM"""
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    today = date.today()
    current_time = datetime.now()
    
    # Check if it's after 6 PM
    if current_time.hour >= 18:
        # Find employees who have entry but no exit today
        cursor.execute('''
            SELECT a.employee_id, e.name, a.entry_time
            FROM attendance a
            JOIN employees e ON a.employee_id = e.id
            WHERE a.date = ? AND a.entry_time IS NOT NULL AND a.exit_time IS NULL
        ''', (today,))
        
        employees_to_exit = cursor.fetchall()
        
        for emp_id, emp_name, entry_time in employees_to_exit:
            # Set exit time to 6 PM (18:00)
            exit_time = "18:00"
            
            # Calculate total hours and overtime
            entry_dt = datetime.strptime(f"{today} {entry_time}", "%Y-%m-%d %H:%M")
            exit_dt = datetime.strptime(f"{today} {exit_time}", "%Y-%m-%d %H:%M")
            
            total_hours = (exit_dt - entry_dt).total_seconds() / 3600
            
            # Update attendance record
            cursor.execute('''
                UPDATE attendance 
                SET exit_time = ?, total_hours = ?, overtime_hours = 0, is_overtime = 0
                WHERE employee_id = ? AND date = ?
            ''', (exit_time, total_hours, emp_id, today))
        
        conn.commit()
        conn.close()
        return len(employees_to_exit)
    
    conn.close()
    return 0

def get_attendance_data():
    conn = sqlite3.connect('attendance.db')
    query = '''
        SELECT 
            e.name,
            e.user_id,
            e.department,
            a.date,
            a.entry_time,
            a.exit_time,
            a.total_hours,
            a.status
        FROM attendance a
        JOIN employees e ON a.employee_id = e.id
        ORDER BY a.date DESC, a.entry_time DESC
    '''
    df = pd.read_sql_query(query, conn)
    conn.close()
    return df

def get_employee_list():
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    cursor.execute('SELECT id, name, user_id, department FROM employees ORDER BY name')
    employees = cursor.fetchall()
    conn.close()
    return employees

def get_attendance_status_stats(date=None):
    """Get attendance statistics by status for a specific date"""
    conn = sqlite3.connect('attendance.db')
    cursor = conn.cursor()
    
    if date:
        cursor.execute('''
            SELECT status, COUNT(*) as count
            FROM attendance 
            WHERE date = ?
            GROUP BY status
        ''', (date,))
    else:
        cursor.execute('''
            SELECT status, COUNT(*) as count
            FROM attendance 
            GROUP BY status
        ''')
    
    stats = dict(cursor.fetchall())
    conn.close()
    
    # Ensure all statuses are present
    all_statuses = ["Present", "Half Day", "Absent"]
    for status in all_statuses:
        if status not in stats:
            stats[status] = 0
    
    return stats

def get_overtime_data():
    """Get overtime statistics for all employees"""
    conn = sqlite3.connect('attendance.db')
    query = '''
        SELECT 
            e.name,
            e.user_id,
            e.department,
            a.date,
            a.total_hours,
            a.overtime_hours,
            a.is_overtime
        FROM attendance a
        JOIN employees e ON a.employee_id = e.id
        WHERE a.is_overtime = 1
        ORDER BY a.date DESC, a.overtime_hours DESC
    '''
    df = pd.read_sql_query(query, conn)
    conn.close()
    return df

# Main application
def main():
    # Header
    st.markdown("""
        <div class="main-header">
            <h1>🌸 Senture Perfumes - Employee Attendance System 🌸</h1>
            <p>Track employee attendance with precision and ease</p>
        </div>
    """, unsafe_allow_html=True)
    
    # Sidebar navigation
    with st.sidebar:
        st.image("https://img.icons8.com/color/96/000000/perfume-bottle.png", width=100)
        st.markdown("### Navigation")
        
        selected = option_menu(
            menu_title=None,
            options=["🏠 Dashboard", "👤 Employee Management", "⏰ Mark Attendance", "📊 Reports", "👥 Employee List"],
            icons=["house", "person-plus", "clock", "bar-chart", "people"],
            menu_icon="cast",
            default_index=0,
        )
    
    # Dashboard
    if selected == "🏠 Dashboard":
        st.header("📊 Dashboard")
        
        col1, col2, col3, col4 = st.columns(4)
        
        # Get statistics
        conn = sqlite3.connect('attendance.db')
        cursor = conn.cursor()
        
        # Total employees
        cursor.execute('SELECT COUNT(*) FROM employees')
        total_employees = cursor.fetchone()[0]
        
        # Today's attendance
        today = date.today()
        cursor.execute('SELECT COUNT(*) FROM attendance WHERE date = ?', (today,))
        today_attendance = cursor.fetchone()[0]
        
        # Present today
        cursor.execute('SELECT COUNT(*) FROM attendance WHERE date = ? AND entry_time IS NOT NULL', (today,))
        present_today = cursor.fetchone()[0]
        
        # Average hours this month
        cursor.execute('''
            SELECT AVG(total_hours) FROM attendance 
            WHERE date >= date('now', 'start of month') AND total_hours IS NOT NULL
        ''')
        avg_hours = cursor.fetchone()[0] or 0
        
        # Overtime statistics
        cursor.execute('''
            SELECT COUNT(*) FROM attendance 
            WHERE date = ? AND is_overtime = 1
        ''', (today,))
        overtime_today = cursor.fetchone()[0]
        
        cursor.execute('''
            SELECT SUM(overtime_hours) FROM attendance 
            WHERE date = ? AND is_overtime = 1
        ''', (today,))
        total_overtime_hours = cursor.fetchone()[0] or 0
        
        conn.close()
        
        # Get attendance status statistics for today
        status_stats = get_attendance_status_stats(today)
        
        with col1:
            st.markdown(f"""
                <div class="metric-card pulse-hover">
                    <h3 style="color: #FF6B6B; margin: 0;">👥 Total Employees</h3>
                    <h2 style="color: #333; margin: 10px 0;">{total_employees}</h2>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown(f"""
                <div class="metric-card pulse-hover">
                    <h3 style="color: #4ECDC4; margin: 0;">📅 Today's Attendance</h3>
                    <h2 style="color: #333; margin: 10px 0;">{today_attendance}</h2>
                </div>
            """, unsafe_allow_html=True)
        
        with col3:
            st.markdown(f"""
                <div class="metric-card pulse-hover">
                    <h3 style="color: #45B7D1; margin: 0;">✅ Present Today</h3>
                    <h2 style="color: #333; margin: 10px 0;">{status_stats['Present']}</h2>
                </div>
            """, unsafe_allow_html=True)
        
        with col4:
            st.markdown(f"""
                <div class="metric-card pulse-hover">
                    <h3 style="color: #96CEB4; margin: 0;">⏰ Avg Hours (Month)</h3>
                    <h2 style="color: #333; margin: 10px 0;">{avg_hours:.1f}</h2>
                </div>
            """, unsafe_allow_html=True)
        
        # Add overtime metrics row
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #FF6B6B; text-align: center;">🕐 Overtime Statistics</h3>', unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.markdown(f"""
                <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #FF6B6B, #FF5252); color: white;">
                    <h4 style="color: white; margin: 0;">🕐 Overtime Today</h4>
                    <h3 style="color: white; margin: 10px 0;">{overtime_today}</h3>
                    <small style="color: rgba(255,255,255,0.8);">Employees working overtime</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown(f"""
                <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #9C27B0, #7B1FA2); color: white;">
                    <h4 style="color: white; margin: 0;">⏰ Total Overtime Hours</h4>
                    <h3 style="color: white; margin: 10px 0;">{total_overtime_hours:.1f}</h3>
                    <small style="color: rgba(255,255,255,0.8);">Extra hours worked today</small>
                </div>
            """, unsafe_allow_html=True)
        
        st.markdown('</div>', unsafe_allow_html=True)
        
        # Attendance Status Breakdown
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #FF6B6B; text-align: center;">📊 Today\'s Attendance Status</h3>', unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.markdown(f"""
                <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #4CAF50, #45a049); color: white;">
                    <h4 style="color: white; margin: 0;">✅ Present</h4>
                    <h3 style="color: white; margin: 10px 0;">{status_stats['Present']}</h3>
                    <small style="color: rgba(255,255,255,0.8);">On time (≤ 10:00 AM)</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown(f"""
                <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #FF9800, #F57C00); color: white;">
                    <h4 style="color: white; margin: 0;">🕐 Half Day</h4>
                    <h3 style="color: white; margin: 10px 0;">{status_stats['Half Day']}</h3>
                    <small style="color: rgba(255,255,255,0.8);">Late (10:00 AM)</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col3:
            st.markdown(f"""
                <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #F44336, #D32F2F); color: white;">
                    <h4 style="color: white; margin: 0;">❌ Absent</h4>
                    <h3 style="color: white; margin: 10px 0;">{status_stats['Absent']}</h3>
                    <small style="color: rgba(255,255,255,0.8);">Very late (> 10:00 AM)</small>
                </div>
            """, unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
        # Time Information
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #4ECDC4; text-align: center;">⏰ Time Management Rules</h3>', unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #4CAF50, #45a049); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">✅ Present</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>≤ 10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">On time arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #FF9800, #F57C00); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">🕐 Half Day</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Late arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col3:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #F44336, #D32F2F); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">❌ Absent</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>> 10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Very late arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        st.markdown(f"""
            <div style="text-align: center; margin-top: 1rem; padding: 1rem; background: #f8f9fa; border-radius: 8px; border-left: 4px solid #4ECDC4;">
                <p style="margin: 0; font-size: 1.1em;"><strong>Current Time:</strong> {datetime.now().strftime('%I:%M %p')} | <strong>Date:</strong> {date.today().strftime('%B %d, %Y')}</p>
            </div>
        """, unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
        # Recent attendance chart
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #4ECDC4; text-align: center;">📈 Recent Attendance Chart</h3>', unsafe_allow_html=True)
        
        df = get_attendance_data()
        if not df.empty:
            df['date'] = pd.to_datetime(df['date'])
            # Filter only rows with valid numeric total_hours
            numeric_df = df[df['total_hours'].notna() & 
                           (df['total_hours'] != '') & 
                           pd.to_numeric(df['total_hours'], errors='coerce').notna()]
            
            if not numeric_df.empty:
                recent_df = numeric_df.head(20)
                
                fig = px.bar(
                    recent_df, 
                    x='date', 
                    y='total_hours',
                    color='name',
                    title="Recent Attendance Hours",
                    labels={'total_hours': 'Hours Worked', 'date': 'Date'}
                )
                st.plotly_chart(fig)
            else:
                st.info("No complete attendance data available for charts yet.")
        else:
            st.info("No attendance data available yet. Please mark some attendance first.")
        st.markdown('</div>', unsafe_allow_html=True)
    
    # Employee Management
    elif selected == "👤 Employee Management":
        st.header("👤 Employee Management")
        
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        with st.form("add_employee_form"):
            st.markdown('<h3 style="color: #FF6B6B; text-align: center;">👤 Add New Employee</h3>', unsafe_allow_html=True)
            
            col1, col2 = st.columns(2)
            
            with col1:
                name = st.text_input("Full Name", placeholder="Enter employee's full name")
                user_id = st.text_input("User ID", placeholder="Enter unique user ID")
            
            with col2:
                email = st.text_input("Email", placeholder="Enter employee's email")
                department = st.selectbox("Department", 
                    ["Production", "Quality Control", "Sales", "Marketing", "HR", "Finance", "IT", "Other"])
            
            submit_button = st.form_submit_button("➕ Add Employee")
        st.markdown('</div>', unsafe_allow_html=True)
        
        if submit_button:
                if name and user_id:
                    success = add_employee(name, user_id, email, department)
                    if success:
                        st.markdown('<div class="success-message">✅ Employee added successfully!</div>', 
                                  unsafe_allow_html=True)
                    else:
                        st.markdown('<div class="error-message">❌ User ID already exists!</div>', 
                                  unsafe_allow_html=True)
                else:
                    st.markdown('<div class="error-message">❌ Please fill all required fields!</div>', 
                              unsafe_allow_html=True)
        
        # QR Code Management
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #9C27B0; text-align: center;">📱 QR Code Management</h3>', unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #9C27B0, #7B1FA2); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">🎯 Generate All QR Codes</h4>
                    <p style="margin: 5px 0; font-size: 1.1em;">Create QR codes for all employees</p>
                    <small style="color: rgba(255,255,255,0.8);">Batch generation</small>
                </div>
            """, unsafe_allow_html=True)
            
            if st.button("🔄 Generate All QR Codes", key="generate_all_qr"):
                try:
                    from qr_generator import generate_employee_qr_codes
                    qr_codes = generate_employee_qr_codes()
                    st.success(f"✅ Successfully generated {len(qr_codes)} QR codes!")
                    st.info("📁 QR codes saved in 'qr_codes' folder")
                except Exception as e:
                    st.error(f"❌ Error generating QR codes: {str(e)}")
        
        with col2:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #FF6B6B, #FF5252); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">📱 Individual QR Code</h4>
                    <p style="margin: 5px 0; font-size: 1.1em;">Generate QR code for specific employee</p>
                    <small style="color: rgba(255,255,255,0.8);">Single generation</small>
                </div>
            """, unsafe_allow_html=True)
            
            # Get employee list for dropdown
            employees = get_employee_list()
            if employees:
                employee_names = [f"{emp[1]} ({emp[2]})" for emp in employees]
                selected_employee = st.selectbox("Select Employee", employee_names, key="qr_employee_select")
                
                if st.button("📱 Generate QR Code", key="generate_single_qr"):
                    try:
                        # Find selected employee
                        selected_name = selected_employee.split(" (")[0]
                        selected_user_id = selected_employee.split("(")[1].split(")")[0]
                        
                        for emp in employees:
                            if emp[1] == selected_name and emp[2] == selected_user_id:
                                from qr_generator import generate_single_qr_code
                                qr_info = generate_single_qr_code(emp[0])
                                if qr_info:
                                    st.success(f"✅ QR code generated for {qr_info['name']}!")
                                    st.info(f"📁 Saved as: {qr_info['qr_file']}")
                                break
                    except Exception as e:
                        st.error(f"❌ Error generating QR code: {str(e)}")
            else:
                st.info("No employees found. Please add employees first.")
        
        st.markdown('</div>', unsafe_allow_html=True)
    
    # Mark Attendance
    elif selected == "⏰ Mark Attendance":
        st.header("⏰ Mark Attendance")
        
        # Time Rules Information
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #FF6B6B; text-align: center;">⏰ Attendance Time Rules</h3>', unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #4CAF50, #45a049); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">✅ Present</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>≤ 10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">On time arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #FF9800, #F57C00); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">🕐 Half Day</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Late arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col3:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #F44336, #D32F2F); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">❌ Absent</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>> 10:00 AM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Very late arrival</small>
                </div>
            """, unsafe_allow_html=True)
        
        st.markdown(f"""
            <div style="text-align: center; margin-top: 1rem; padding: 1rem; background: #f8f9fa; border-radius: 8px; border-left: 4px solid #FF6B6B;">
                <p style="margin: 0; font-size: 1.1em;"><strong>Current Time:</strong> {datetime.now().strftime('%I:%M %p')} | <strong>Date:</strong> {date.today().strftime('%B %d, %Y')}</p>
            </div>
        """, unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #4ECDC4; text-align: center;">⏰ Mark Entry Time</h3>', unsafe_allow_html=True)
        
        with st.form("entry_form"):
            user_id = st.text_input("Enter User ID", key="entry_user_id", placeholder="Enter your User ID")
            submit_entry = st.form_submit_button("🟢 Mark Entry")
        st.markdown('</div>', unsafe_allow_html=True)
        
        if submit_entry and user_id:
                employee = get_employee_by_user_id(user_id)
                if employee:
                    st.success(f"Employee found: {employee[1]} ({employee[2]})")
                    
                    current_time = datetime.now().strftime("%H:%M")
                    current_date = date.today()
                    
                    success, message = mark_entry(employee[0], current_date, current_time)
                    
                    if success:
                        st.markdown(f'<div class="success-message">✅ {message}</div>', 
                                  unsafe_allow_html=True)
                        st.info(f"Entry time: {current_time}")
                    else:
                        st.markdown(f'<div class="error-message">❌ {message}</div>', 
                                  unsafe_allow_html=True)
                else:
                    st.error("Employee not found! Please check the User ID.")
        
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #FF6B6B; text-align: center;">⏰ Mark Exit Time</h3>', unsafe_allow_html=True)
        
        with st.form("exit_form"):
            user_id = st.text_input("Enter User ID", key="exit_user_id", placeholder="Enter your User ID")
            submit_exit = st.form_submit_button("🔴 Mark Exit")
        st.markdown('</div>', unsafe_allow_html=True)
        
        if submit_exit and user_id:
            employee = get_employee_by_user_id(user_id)
            if employee:
                st.success(f"Employee found: {employee[1]} ({employee[2]})")
                
                current_time = datetime.now().strftime("%H:%M")
                current_date = date.today()
                
                success, message = mark_exit(employee[0], current_date, current_time)
                
                if success:
                    st.markdown(f'<div class="success-message">✅ {message}</div>', 
                              unsafe_allow_html=True)
                    st.info(f"Exit time: {current_time}")
                else:
                    st.markdown(f'<div class="error-message">❌ {message}</div>', 
                              unsafe_allow_html=True)
            else:
                st.error("Employee not found! Please check the User ID.")
        
        # Automatic Exit Management
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #9C27B0; text-align: center;">🤖 Automatic Exit Management</h3>', unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #FF6B6B, #FF5252); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">🕐 Standard Exit Time</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>6:00 PM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Automatic exit marking</small>
                </div>
            """, unsafe_allow_html=True)
        
        with col2:
            st.markdown("""
                <div style="text-align: center; padding: 1rem; background: linear-gradient(135deg, #9C27B0, #7B1FA2); color: white; border-radius: 8px;">
                    <h4 style="color: white; margin: 0;">⏰ Overtime Calculation</h4>
                    <p style="margin: 5px 0; font-size: 1.2em;"><strong>> 6:00 PM</strong></p>
                    <small style="color: rgba(255,255,255,0.8);">Extra hours tracking</small>
                </div>
            """, unsafe_allow_html=True)
        
        st.markdown('</div>', unsafe_allow_html=True)
        
        # Manual Overtime Entry
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #9C27B0; text-align: center;">⏰ Manual Overtime Entry</h3>', unsafe_allow_html=True)
        
        with st.form("overtime_form"):
            col1, col2 = st.columns(2)
            
            with col1:
                overtime_user_id = st.text_input("User ID for Overtime", key="overtime_user_id", placeholder="Enter User ID")
                overtime_date = st.date_input("Overtime Date", value=date.today())
            
            with col2:
                overtime_start = st.time_input("Overtime Start Time", value=datetime.strptime("18:00", "%H:%M").time())
                overtime_end = st.time_input("Overtime End Time", value=datetime.strptime("19:00", "%H:%M").time())
            
            submit_overtime = st.form_submit_button("🕐 Add Overtime Hours")
        
        if submit_overtime and overtime_user_id:
            employee = get_employee_by_user_id(overtime_user_id)
            if employee:
                # Calculate overtime hours
                start_dt = datetime.combine(overtime_date, overtime_start)
                end_dt = datetime.combine(overtime_date, overtime_end)
                overtime_hours = (end_dt - start_dt).total_seconds() / 3600
                
                if overtime_hours > 0:
                    # Update or create overtime record
                    conn = sqlite3.connect('attendance.db')
                    cursor = conn.cursor()
                    
                    cursor.execute('''
                        UPDATE attendance 
                        SET overtime_hours = ?, is_overtime = 1
                        WHERE employee_id = ? AND date = ?
                    ''', (overtime_hours, employee[0], overtime_date))
                    
                    conn.commit()
                    conn.close()
                    
                    st.markdown(f'<div class="success-message">✅ Overtime added: {overtime_hours:.2f} hours for {employee[1]}</div>', 
                              unsafe_allow_html=True)
                else:
                    st.markdown('<div class="error-message">❌ End time must be after start time</div>', 
                              unsafe_allow_html=True)
            else:
                st.error("Employee not found! Please check the User ID.")
        
        st.markdown('</div>', unsafe_allow_html=True)
        
        # QR Code Scanner
        qr_scanner_component()
    
    # Reports
    elif selected == "📊 Reports":
        st.header("📊 Attendance Reports")
        
        # Automatic Exit Check
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #9C27B0; text-align: center;">🤖 Automatic Exit Management</h3>', unsafe_allow_html=True)
        
        current_time = datetime.now()
        if current_time.hour >= 18:
            if st.button("🕐 Auto-Mark Exit for All (After 6 PM)", type="primary"):
                auto_exit_count = auto_mark_exit_after_6pm()
                if auto_exit_count > 0:
                    st.markdown(f'<div class="success-message">✅ Automatically marked exit for {auto_exit_count} employees at 6:00 PM</div>', 
                              unsafe_allow_html=True)
                else:
                    st.info("No employees need automatic exit marking")
        else:
            st.info(f"🕐 Automatic exit marking available after 6:00 PM (Current time: {current_time.strftime('%I:%M %p')})")
        
        st.markdown('</div>', unsafe_allow_html=True)
        
        df = get_attendance_data()
        
        if not df.empty:
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #FF6B6B; text-align: center;">📅 Date Range Selection</h3>', unsafe_allow_html=True)
            
            # Date filter
            col1, col2 = st.columns(2)
            with col1:
                start_date = st.date_input("Start Date", value=date.today())
            with col2:
                end_date = st.date_input("End Date", value=date.today())
            st.markdown('</div>', unsafe_allow_html=True)
            
            # Filter data
            filtered_df = df[
                (pd.to_datetime(df['date']) >= pd.to_datetime(start_date)) &
                (pd.to_datetime(df['date']) <= pd.to_datetime(end_date))
            ]
            
            # Summary statistics
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #4ECDC4; text-align: center;">📊 Summary Statistics</h3>', unsafe_allow_html=True)
            
            col1, col2, col3 = st.columns(3)
            
            with col1:
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #FF6B6B; margin: 0;">📋 Total Records</h4>
                        <h3 style="color: #333; margin: 10px 0;">{len(filtered_df)}</h3>
                    </div>
                """, unsafe_allow_html=True)
            
            with col2:
                # Filter numeric total_hours for calculations
                numeric_hours = pd.to_numeric(filtered_df['total_hours'], errors='coerce')
                avg_hours = numeric_hours.mean()
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #4ECDC4; margin: 0;">⏰ Average Hours</h4>
                        <h3 style="color: #333; margin: 10px 0;">{f"{avg_hours:.2f}" if not pd.isna(avg_hours) else "0.00"}</h3>
                    </div>
                """, unsafe_allow_html=True)
            
            with col3:
                # Filter numeric total_hours for calculations
                numeric_hours = pd.to_numeric(filtered_df['total_hours'], errors='coerce')
                total_hours = numeric_hours.sum()
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #96CEB4; margin: 0;">🕐 Total Hours</h4>
                        <h3 style="color: #333; margin: 10px 0;">{f"{total_hours:.2f}" if not pd.isna(total_hours) else "0.00"}</h3>
                    </div>
                """, unsafe_allow_html=True)
            st.markdown('</div>', unsafe_allow_html=True)
            
            # Attendance Status Statistics
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #FF6B6B; text-align: center;">📊 Attendance Status Breakdown</h3>', unsafe_allow_html=True)
            
            # Get status counts for the filtered period
            status_counts = filtered_df['status'].value_counts()
            
            col1, col2, col3 = st.columns(3)
            
            with col1:
                present_count = status_counts.get('Present', 0)
                st.markdown(f"""
                    <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #4CAF50, #45a049); color: white;">
                        <h4 style="color: white; margin: 0;">✅ Present</h4>
                        <h3 style="color: white; margin: 10px 0;">{present_count}</h3>
                        <small style="color: rgba(255,255,255,0.8);">On time arrivals</small>
                    </div>
                """, unsafe_allow_html=True)
            
            with col2:
                half_day_count = status_counts.get('Half Day', 0)
                st.markdown(f"""
                    <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #FF9800, #F57C00); color: white;">
                        <h4 style="color: white; margin: 0;">🕐 Half Day</h4>
                        <h3 style="color: white; margin: 10px 0;">{half_day_count}</h3>
                        <small style="color: rgba(255,255,255,0.8);">Late arrivals</small>
                    </div>
                """, unsafe_allow_html=True)
            
            with col3:
                absent_count = status_counts.get('Absent', 0)
                st.markdown(f"""
                    <div class="metric-card pulse-hover" style="background: linear-gradient(135deg, #F44336, #D32F2F); color: white;">
                        <h4 style="color: white; margin: 0;">❌ Absent</h4>
                        <h3 style="color: white; margin: 10px 0;">{absent_count}</h3>
                        <small style="color: rgba(255,255,255,0.8);">Very late arrivals</small>
                    </div>
                """, unsafe_allow_html=True)
            st.markdown('</div>', unsafe_allow_html=True)
            
            # Attendance by department
            if 'department' in filtered_df.columns:
                st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
                st.markdown('<h3 style="color: #96CEB4; text-align: center;">🏢 Attendance by Department</h3>', unsafe_allow_html=True)
                
                # Filter out rows where total_hours is not numeric
                numeric_df = filtered_df[filtered_df['total_hours'].notna() & 
                                       (filtered_df['total_hours'] != '') & 
                                       pd.to_numeric(filtered_df['total_hours'], errors='coerce').notna()]
                
                if not numeric_df.empty:
                    try:
                        dept_stats = numeric_df.groupby('department').agg({
                            'total_hours': ['count', 'mean', 'sum']
                        }).round(2)
                        st.dataframe(dept_stats)
                    except Exception as e:
                        st.warning("Could not generate department statistics due to data format issues.")
                        st.info("This usually happens when there's no complete attendance data yet.")
                else:
                    st.info("No complete attendance data available for department statistics.")
                st.markdown('</div>', unsafe_allow_html=True)
            
            # Detailed attendance table
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #45B7D1; text-align: center;">📋 Detailed Attendance Records</h3>', unsafe_allow_html=True)
            st.dataframe(filtered_df)
            st.markdown('</div>', unsafe_allow_html=True)
            
            # Download option
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #FF6B6B; text-align: center;">💾 Export Data</h3>', unsafe_allow_html=True)
            csv = filtered_df.to_csv(index=False)
            st.download_button(
                label="📥 Download CSV Report",
                data=csv,
                file_name=f"attendance_report_{start_date}_{end_date}.csv",
                mime="text/csv"
            )
            st.markdown('</div>', unsafe_allow_html=True)
        
        # Overtime Reports
        st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
        st.markdown('<h3 style="color: #9C27B0; text-align: center;">🕐 Overtime Reports</h3>', unsafe_allow_html=True)
        
        overtime_df = get_overtime_data()
        
        if not overtime_df.empty:
            # Overtime Summary
            col1, col2, col3 = st.columns(3)
            
            with col1:
                total_overtime_employees = len(overtime_df)
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #9C27B0; margin: 0;">👥 Overtime Employees</h4>
                        <h3 style="color: #333; margin: 10px 0;">{total_overtime_employees}</h3>
                    </div>
                """, unsafe_allow_html=True)
            
            with col2:
                total_overtime_hours = overtime_df['overtime_hours'].sum()
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #FF6B6B; margin: 0;">⏰ Total Overtime</h4>
                        <h3 style="color: #333; margin: 10px 0;">{total_overtime_hours:.1f}</h3>
                    </div>
                """, unsafe_allow_html=True)
            
            with col3:
                avg_overtime = overtime_df['overtime_hours'].mean()
                st.markdown(f"""
                    <div class="metric-card pulse-hover">
                        <h4 style="color: #4ECDC4; margin: 0;">⏰ Avg Overtime</h4>
                        <h3 style="color: #333; margin: 10px 0;">{avg_overtime:.1f}</h3>
                    </div>
                """, unsafe_allow_html=True)
            
            # Overtime by Department
            if 'department' in overtime_df.columns:
                dept_overtime = overtime_df.groupby('department')['overtime_hours'].sum().sort_values(ascending=False)
                
                fig = px.bar(
                    x=dept_overtime.index,
                    y=dept_overtime.values,
                    title="Overtime Hours by Department",
                    labels={'x': 'Department', 'y': 'Overtime Hours'},
                    color=dept_overtime.values,
                    color_continuous_scale='viridis'
                )
                st.plotly_chart(fig)
            
            # Overtime Details Table
            st.markdown('<h4 style="color: #9C27B0;">📋 Overtime Details</h4>', unsafe_allow_html=True)
            st.dataframe(overtime_df)
            
            # Download Overtime Report
            overtime_csv = overtime_df.to_csv(index=False)
            st.download_button(
                label="📥 Download Overtime Report",
                data=overtime_csv,
                file_name=f"overtime_report_{start_date}_{end_date}.csv",
                mime="text/csv"
            )
        else:
            st.info("No overtime data available for the selected period.")
        
        st.markdown('</div>', unsafe_allow_html=True)
    # Employee List
    elif selected == "👥 Employee List":
        st.header("👥 Employee List")
        
        employees = get_employee_list()
        
        if employees:
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #FF6B6B; text-align: center;">👥 Employee List</h3>', unsafe_allow_html=True)
            
            # Convert to DataFrame for better display
            emp_df = pd.DataFrame(employees, columns=['ID', 'Name', 'User ID', 'Department'])
            st.dataframe(emp_df)
            st.markdown('</div>', unsafe_allow_html=True)
            
            # Employee count by department
            st.markdown('<div class="attendance-card">', unsafe_allow_html=True)
            st.markdown('<h3 style="color: #4ECDC4; text-align: center;">📊 Employee Distribution by Department</h3>', unsafe_allow_html=True)
            
            dept_counts = emp_df['Department'].value_counts()
            
            fig = px.pie(
                values=dept_counts.values,
                names=dept_counts.index,
                title="Employee Distribution by Department"
            )
            st.plotly_chart(fig)
            st.markdown('</div>', unsafe_allow_html=True)
        else:
            st.info("No employees found. Please add some employees first.")

if __name__ == "__main__":
    main()
