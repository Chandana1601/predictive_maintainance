# predictive_maintainance
This Python script connects to a MySQL database (sensor_data) and extracts sensor log data from the infotainment_sensor_logs table. It performs the following steps:  Retrieves sensor readings such as touch errors, temperature, audio latency, audio glitches, system reboots, and failure labels, ordered by timestamp. 
import mysql.connector
import pandas as pd

# Connect to MySQL
mydb = mysql.connector.connect(
    host='**************',
    user='root',
    password='***************',
    database='sensor_data'
)
cursor = mydb.cursor()

# Feature Extraction: Aggregate sensor readings
cursor.execute("""
    SELECT
        timestamp,
        touch_errors,
        temperature,
        audio_latency,
        audio_glitches,
        system_reboots,
        failure_label
    FROM infotainment_sensor_logs
    ORDER BY timestamp;
""")
rows = cursor.fetchall()

# Load data into pandas DataFrame
columns = ['timestamp', 'touch_errors', 'temperature', 'audio_latency', 'audio_glitches', 'system_reboots', 'failure_label']
df = pd.DataFrame(rows, columns=columns)

# Example: Feature Engineering (rolling averages, error rates)
df['touch_error_rate'] = df['touch_errors'].rolling(window=10).mean()
df['audio_latency_avg'] = df['audio_latency'].rolling(window=10).mean()
df['audio_glitch_rate'] = df['audio_glitches'].rolling(window=10).mean()
df['reboot_count'] = df['system_reboots'].rolling(window=10).sum()

# Save processed data for Power BI visualization
df.to_csv('infotainment_failure_features.csv', index=False)

cursor.close()
mydb.close()
