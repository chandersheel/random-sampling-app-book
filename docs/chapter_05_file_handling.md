# Chapter 5: File Handling and Data Preview

## Introduction

In this chapter, we'll implement the core functionality for loading CSV files and displaying data previews. This is where our application comes alive, transforming from a static interface into a functional data processing tool.

## File Dialog Integration

### Understanding QFileDialog

`QFileDialog` provides a platform-native file selection interface:

```python
from PyQt5.QtWidgets import QFileDialog

def load_csv(self):
    """Open file dialog and load selected CSV file"""
    file_name, _ = QFileDialog.getOpenFileName(
        self,                    # Parent widget
        "Open CSV",             # Dialog title
        "",                     # Starting directory (empty = user's default)
        "CSV Files (*.csv)"     # File filter
    )
    
    if file_name:
        # Process the selected file
        self.process_file(file_name)
```

### QFileDialog Parameters Explained

#### getOpenFileName() Parameters
1. **Parent Widget** (`self`): Dialog's parent window
2. **Dialog Title** (`"Open CSV"`): Text shown in title bar
3. **Starting Directory** (`""`): Initial folder (empty uses system default)
4. **File Filter** (`"CSV Files (*.csv)"`): Restricts visible files

#### Return Values
```python
file_name, file_type = QFileDialog.getOpenFileName(...)
```
- **file_name**: Full path to selected file (empty string if cancelled)
- **file_type**: Selected filter type (usually ignored)

#### File Filter Syntax
```python
# Single filter
"CSV Files (*.csv)"

# Multiple filters
"Data Files (*.csv *.xlsx);;All Files (*)"

# Multiple separate filters
"CSV Files (*.csv);;Excel Files (*.xlsx);;All Files (*)"
```

## Data Loading with Pandas

### Basic CSV Loading

```python
import pandas as pd

def load_csv_file(self, file_name):
    """Load CSV file using pandas"""
    try:
        # Basic loading
        df = pd.read_csv(file_name)
        return df
    except Exception as e:
        print(f"Error loading file: {e}")
        return None
```

### Advanced CSV Loading with Options

```python
def load_csv_advanced(self, file_name):
    """Load CSV with comprehensive options"""
    try:
        df = pd.read_csv(
            file_name,
            encoding='utf-8',           # Handle different encodings
            sep=',',                    # Specify separator
            header=0,                   # First row as column names
            index_col=None,             # Don't use any column as index
            dtype=None,                 # Auto-detect data types
            na_values=['', 'NULL', 'N/A'],  # Values to treat as NaN
            thousands=',',              # Thousands separator
            decimal='.',                # Decimal separator
            skipinitialspace=True,      # Skip spaces after delimiter
            skip_blank_lines=True       # Skip blank lines
        )
        return df
    except Exception as e:
        print(f"Error loading file: {e}")
        return None
```

### Handling Common CSV Issues

#### Encoding Problems
```python
def load_csv_with_encoding(self, file_name):
    """Try different encodings if UTF-8 fails"""
    encodings = ['utf-8', 'utf-8-sig', 'latin1', 'cp1252']
    
    for encoding in encodings:
        try:
            df = pd.read_csv(file_name, encoding=encoding)
            print(f"Successfully loaded with {encoding} encoding")
            return df
        except UnicodeDecodeError:
            continue
    
    raise ValueError("Could not decode file with any supported encoding")
```

#### Delimiter Detection
```python
def detect_delimiter(self, file_name):
    """Automatically detect CSV delimiter"""
    import csv
    
    with open(file_name, 'r', newline='', encoding='utf-8') as file:
        # Read first few lines to detect delimiter
        sample = file.read(1024)
        sniffer = csv.Sniffer()
        delimiter = sniffer.sniff(sample).delimiter
    
    return delimiter
```

## Complete File Loading Implementation

### The load_csv Method

Here's our complete file loading implementation:

```python
def load_csv(self):
    """
    Load CSV file with comprehensive error handling.
    
    This method:
    1. Opens file dialog for user selection
    2. Validates the selected file
    3. Loads data using pandas
    4. Updates the UI with loaded data
    5. Handles errors gracefully
    """
    try:
        # Open file dialog
        file_name, _ = QFileDialog.getOpenFileName(
            self, 
            "Open CSV", 
            "", 
            "CSV Files (*.csv);;All Files (*)"
        )
        
        # Check if user cancelled
        if not file_name:
            return
        
        # Validate file exists
        if not os.path.exists(file_name):
            raise FileNotFoundError(f"File not found: {file_name}")
        
        # Load the CSV file
        self.df = pd.read_csv(file_name)
        
        # Validate data is not empty
        if self.df.empty:
            raise ValueError("The CSV file is empty")
        
        # Success - update UI
        self.show_dataframe(self.df, preview=True)
        
        # Update file status
        file_size = os.path.getsize(file_name) / 1024  # Size in KB
        self.file_status.setText(
            f"Loaded: {os.path.basename(file_name)} "
            f"({len(self.df)} rows, {len(self.df.columns)} columns, "
            f"{file_size:.1f} KB)"
        )
        self.file_status.setStyleSheet("color: #27ae60; font-weight: bold;")
        
    except pd.errors.EmptyDataError:
        QMessageBox.warning(self, "Invalid File", "The CSV file is empty.")
    except pd.errors.ParserError as e:
        QMessageBox.warning(self, "Invalid File", f"Could not parse CSV file: {str(e)}")
    except UnicodeDecodeError:
        QMessageBox.warning(self, "Encoding Error", "Could not read file. Please ensure it's a valid CSV file with UTF-8 encoding.")
    except PermissionError:
        QMessageBox.warning(self, "Permission Error", "Cannot access the file. Please check file permissions.")
    except FileNotFoundError as e:
        QMessageBox.warning(self, "File Not Found", str(e))
    except Exception as e:
        QMessageBox.critical(self, "Error", f"Could not load file: {str(e)}")
```

### Error Handling Categories

#### 1. User Errors
- **File not selected**: User cancelled dialog
- **File not found**: Selected file was moved/deleted
- **Permission denied**: User lacks file access rights

#### 2. Data Errors
- **Empty file**: CSV file contains no data
- **Malformed CSV**: Invalid CSV structure
- **Encoding issues**: File uses unsupported character encoding

#### 3. System Errors
- **Memory errors**: File too large for available RAM
- **Disk errors**: Storage device issues

## Data Display with QTableWidget

### Understanding QTableWidget

`QTableWidget` is a powerful widget for displaying tabular data:

```python
from PyQt5.QtWidgets import QTableWidget, QTableWidgetItem

# Create table
table = QTableWidget()

# Set dimensions
table.setRowCount(3)        # 3 rows
table.setColumnCount(2)     # 2 columns

# Set column headers
table.setHorizontalHeaderLabels(['Name', 'Age'])

# Add data
table.setItem(0, 0, QTableWidgetItem('Alice'))
table.setItem(0, 1, QTableWidgetItem('25'))
```

### Table Configuration Options

#### Visual Appearance
```python
# Alternating row colors for better readability
table.setAlternatingRowColors(True)

# Enable sorting by clicking column headers
table.setSortingEnabled(True)

# Grid line style
table.setShowGrid(True)
table.setGridStyle(Qt.SolidLine)

# Selection behavior
table.setSelectionBehavior(QTableWidget.SelectRows)
table.setSelectionMode(QTableWidget.SingleSelection)
```

#### Size and Scrolling
```python
# Minimum and maximum sizes
table.setMinimumHeight(300)
table.setMaximumHeight(500)

# Column resize modes
header = table.horizontalHeader()
header.setStretchLastSection(True)  # Last column fills remaining space
header.setSectionResizeMode(QHeaderView.ResizeToContents)  # Auto-size columns
```

### The show_dataframe Method

This is our core method for displaying pandas DataFrames in tables:

```python
def show_dataframe(self, df, preview=True, data_preview=False):
    """
    Display DataFrame in the appropriate table widget.
    
    Parameters:
    -----------
    df : pandas.DataFrame
        The DataFrame to display
    preview : bool
        If True, show in preview table (first 10 rows only)
    data_preview : bool
        If True, show in full data preview table
    """
    
    # Determine which table to use and what data to show
    if preview:
        table = self.preview_table
        display_df = df.head(10)  # Only first 10 rows
    elif data_preview:
        table = self.data_preview_table
        display_df = df  # All data
    else:
        table = self.results_table
        display_df = df  # Results data
    
    # Set table dimensions
    table.setRowCount(len(display_df))
    table.setColumnCount(len(display_df.columns))
    
    # Set column headers
    table.setHorizontalHeaderLabels(display_df.columns.astype(str))
    
    # Populate table with data
    for i in range(len(display_df)):
        for j in range(len(display_df.columns)):
            # Convert value to string and create table item
            value = str(display_df.iat[i, j])
            item = QTableWidgetItem(value)
            table.setItem(i, j, item)
    
    # Handle UI updates based on context
    if preview:
        # Show preview frame and update status
        self.preview_frame.setVisible(True)
        
        # Update file status with dataset info
        info_text = f"Dataset loaded: {len(df)} rows, {len(df.columns)} columns (showing first 10 rows)"
        self.file_status.setText(info_text)
        self.file_status.setStyleSheet("color: #27ae60; font-weight: bold;")
        
        # Also populate the full data preview tab
        self.show_dataframe(df, preview=False, data_preview=True)
        self.data_preview_frame.setVisible(True)
        self.data_info_label.setText(f"📊 Complete Dataset - {len(df)} rows × {len(df.columns)} columns")
        self.data_summary_label.setText(f"Click column headers to sort data. All {len(df)} rows are displayed.")
        
        # Automatically switch to data preview tab
        self.tab_widget.setCurrentIndex(1)
    
    elif data_preview:
        # Just update the data preview table
        pass
```

### Data Type Handling

#### Converting Data for Display
```python
def format_value_for_display(self, value):
    """Format various data types for table display"""
    if pd.isna(value):
        return "NaN"
    elif isinstance(value, float):
        # Format floats to reasonable precision
        return f"{value:.6g}"
    elif isinstance(value, (int, np.integer)):
        return str(value)
    elif isinstance(value, str):
        # Truncate very long strings
        return value[:100] + "..." if len(value) > 100 else value
    else:
        return str(value)
```

#### Handling Missing Data
```python
def handle_missing_data(self, df):
    """Analyze and report missing data"""
    missing_info = {}
    
    for column in df.columns:
        missing_count = df[column].isnull().sum()
        if missing_count > 0:
            missing_info[column] = {
                'count': missing_count,
                'percentage': (missing_count / len(df)) * 100
            }
    
    return missing_info
```

## Data Preview Tab Implementation

### Creating the Data Preview Tab

```python
def create_data_preview_tab(self):
    """Second tab: Data Preview with sorting capabilities"""
    preview_tab = QWidget()
    layout = QVBoxLayout()
    layout.setSpacing(10)
    layout.setContentsMargins(15, 15, 15, 15)
    
    # Header section
    header_frame = QFrame()
    header_frame.setObjectName("welcomeFrame")
    header_layout = QVBoxLayout()
    header_layout.setSpacing(8)
    
    header_title = QLabel("📊 Data Preview")
    header_title.setStyleSheet("font-size: 20px; font-weight: bold; color: #2c3e50; text-align: center; margin-bottom: 5px;")
    header_title.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_title)
    
    header_desc = QLabel("Review your data before sampling. Click column headers to sort.")
    header_desc.setStyleSheet("font-size: 14px; color: #34495e; text-align: center; padding: 5px;")
    header_desc.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_desc)
    
    header_frame.setLayout(header_layout)
    layout.addWidget(header_frame)
    
    # Data preview section
    self.data_preview_frame = QFrame()
    self.data_preview_frame.setObjectName("sectionFrame")
    self.data_preview_frame.setVisible(False)  # Hidden until data loaded
    data_layout = QVBoxLayout()
    data_layout.setSpacing(8)
    data_layout.setContentsMargins(10, 10, 10, 10)
    
    # Data information header
    data_header_layout = QHBoxLayout()
    
    self.data_info_label = QLabel("No data loaded yet")
    self.data_info_label.setStyleSheet("font-size: 16px; font-weight: bold; color: #2c3e50;")
    data_header_layout.addWidget(self.data_info_label)
    
    data_header_layout.addStretch()
    
    data_layout.addLayout(data_header_layout)
    
    # Data table with sorting enabled
    self.data_preview_table = QTableWidget()
    self.data_preview_table.setMinimumHeight(350)
    self.data_preview_table.setSortingEnabled(True)  # Enable column sorting
    self.data_preview_table.setAlternatingRowColors(True)  # Zebra striping
    data_layout.addWidget(self.data_preview_table)
    
    # Data summary information
    self.data_summary_label = QLabel("")
    self.data_summary_label.setStyleSheet("color: #6c757d; font-size: 12px; margin-top: 5px;")
    data_layout.addWidget(self.data_summary_label)
    
    self.data_preview_frame.setLayout(data_layout)
    layout.addWidget(self.data_preview_frame)
    
    # Navigation buttons
    nav_frame = QFrame()
    nav_layout = QHBoxLayout()
    nav_layout.setContentsMargins(0, 5, 0, 0)
    
    back_button = QPushButton("← Back to File")
    back_button.setObjectName("backButton")
    back_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(0))
    nav_layout.addWidget(back_button)
    
    nav_layout.addStretch()
    
    next_button = QPushButton("Next: Method →")
    next_button.setObjectName("nextButton")
    next_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(2))
    nav_layout.addWidget(next_button)
    
    nav_frame.setLayout(nav_layout)
    layout.addWidget(nav_frame)
    
    preview_tab.setLayout(layout)
    self.tab_widget.addTab(preview_tab, "📊 Data")
```

### Sorting Functionality

#### Enabling Table Sorting
```python
# Enable sorting on the table
table.setSortingEnabled(True)

# Sorting is automatic - clicking column headers will sort
# Qt handles ascending/descending toggle automatically
```

#### Custom Sort Indicators
```python
# Get the horizontal header
header = table.horizontalHeader()

# Set sort indicator visibility
header.setSortIndicatorShown(True)

# Set initial sort column and order
header.setSortIndicator(0, Qt.AscendingOrder)
```

#### Programmatic Sorting
```python
# Sort by column index (0-based)
table.sortItems(0, Qt.AscendingOrder)   # Sort first column ascending
table.sortItems(1, Qt.DescendingOrder)  # Sort second column descending
```

## Data Validation and Quality Checks

### Basic Data Validation

```python
def validate_dataframe(self, df):
    """Perform basic validation on loaded DataFrame"""
    issues = []
    
    # Check for empty DataFrame
    if df.empty:
        issues.append("DataFrame is empty")
        return issues
    
    # Check for completely empty columns
    empty_columns = df.columns[df.isnull().all()].tolist()
    if empty_columns:
        issues.append(f"Empty columns found: {empty_columns}")
    
    # Check for duplicate column names
    duplicate_columns = df.columns[df.columns.duplicated()].tolist()
    if duplicate_columns:
        issues.append(f"Duplicate column names: {duplicate_columns}")
    
    # Check for excessive missing data
    missing_percentage = (df.isnull().sum() / len(df)) * 100
    high_missing = missing_percentage[missing_percentage > 50].index.tolist()
    if high_missing:
        issues.append(f"Columns with >50% missing data: {high_missing}")
    
    return issues
```

### Data Quality Report

```python
def generate_data_quality_report(self, df):
    """Generate comprehensive data quality report"""
    report = {
        'total_rows': len(df),
        'total_columns': len(df.columns),
        'memory_usage': df.memory_usage(deep=True).sum(),
        'column_types': df.dtypes.to_dict(),
        'missing_data': {},
        'numeric_summary': {},
        'categorical_summary': {}
    }
    
    # Missing data analysis
    for column in df.columns:
        missing_count = df[column].isnull().sum()
        report['missing_data'][column] = {
            'count': missing_count,
            'percentage': (missing_count / len(df)) * 100
        }
    
    # Numeric column analysis
    numeric_columns = df.select_dtypes(include=[np.number]).columns
    for column in numeric_columns:
        series = df[column].dropna()
        if len(series) > 0:
            report['numeric_summary'][column] = {
                'min': series.min(),
                'max': series.max(),
                'mean': series.mean(),
                'median': series.median(),
                'std': series.std()
            }
    
    # Categorical column analysis
    categorical_columns = df.select_dtypes(include=['object']).columns
    for column in categorical_columns:
        series = df[column].dropna()
        if len(series) > 0:
            report['categorical_summary'][column] = {
                'unique_count': series.nunique(),
                'most_common': series.value_counts().head(5).to_dict()
            }
    
    return report
```

## Performance Optimization

### Large File Handling

```python
def load_large_csv(self, file_name, chunk_size=10000):
    """Load large CSV files in chunks"""
    try:
        # Read file in chunks
        chunk_list = []
        for chunk in pd.read_csv(file_name, chunksize=chunk_size):
            chunk_list.append(chunk)
        
        # Combine chunks
        df = pd.concat(chunk_list, ignore_index=True)
        return df
        
    except Exception as e:
        raise Exception(f"Error loading large file: {str(e)}")
```

### Memory Management

```python
def optimize_dataframe_memory(self, df):
    """Optimize DataFrame memory usage"""
    # Convert object columns to categorical if they have few unique values
    for column in df.select_dtypes(include=['object']).columns:
        unique_ratio = df[column].nunique() / len(df)
        if unique_ratio < 0.1:  # Less than 10% unique values
            df[column] = df[column].astype('category')
    
    # Downcast numeric types
    for column in df.select_dtypes(include=['int']).columns:
        df[column] = pd.to_numeric(df[column], downcast='integer')
    
    for column in df.select_dtypes(include=['float']).columns:
        df[column] = pd.to_numeric(df[column], downcast='float')
    
    return df
```

### Preview Limitations

```python
def create_smart_preview(self, df, max_rows=1000):
    """Create intelligent preview for large datasets"""
    if len(df) <= max_rows:
        return df
    
    # For large datasets, show:
    # - First 500 rows
    # - Last 500 rows
    # - Random sample from middle
    
    first_part = df.head(250)
    last_part = df.tail(250)
    
    # Random sample from middle
    middle_start = 250
    middle_end = len(df) - 250
    if middle_end > middle_start:
        middle_indices = np.random.choice(
            range(middle_start, middle_end), 
            size=min(500, middle_end - middle_start),
            replace=False
        )
        middle_part = df.iloc[sorted(middle_indices)]
    else:
        middle_part = pd.DataFrame()
    
    preview_df = pd.concat([first_part, middle_part, last_part], ignore_index=True)
    return preview_df
```

## Chapter Summary

In this chapter, we implemented comprehensive file handling and data preview functionality:

1. **File Dialog Integration**: Using `QFileDialog` for native file selection
2. **Robust Data Loading**: pandas integration with comprehensive error handling
3. **Data Validation**: Quality checks and issue detection
4. **Table Display**: `QTableWidget` configuration and data population
5. **Data Preview Tab**: Full dataset viewing with sorting capabilities
6. **Error Handling**: Graceful handling of various error conditions
7. **Performance Optimization**: Strategies for large file handling
8. **User Feedback**: Status updates and progress indication

### Key Features Implemented

- **Multi-format Support**: CSV files with various encodings and delimiters
- **Error Recovery**: Graceful handling of malformed data
- **Data Quality Reporting**: Analysis of missing data and column types
- **Interactive Tables**: Sortable columns and responsive display
- **Progressive Disclosure**: Showing relevant information at the right time
- **Performance Optimization**: Memory-efficient handling of large datasets

### Design Patterns Demonstrated

- **Error Handling Pattern**: Comprehensive try-catch blocks with user feedback
- **Template Method**: Consistent data display across different contexts
- **State Management**: UI updates based on data loading state
- **Progressive Enhancement**: Basic functionality with advanced features

In the next chapter, we'll build interactive user interface elements including radio buttons, sliders, and form inputs that allow users to configure their sampling parameters.

---

**Key Takeaways:**
- Robust error handling is essential for file operations
- Data validation prevents downstream issues
- Progressive disclosure improves user experience
- Performance optimization is crucial for large datasets
- User feedback keeps users informed during long operations
