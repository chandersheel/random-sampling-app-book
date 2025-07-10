# Chapter 6: Creating Interactive User Interface Elements

## Introduction

In this chapter, we'll explore the creation of interactive user interface elements that make our application functional and user-friendly. We'll cover radio buttons, sliders, input fields, and the logic that coordinates their behavior.

## Radio Button Groups and Mutual Exclusion

### Understanding Radio Buttons

Radio buttons are used for mutually exclusive selections - only one option can be selected at a time. In our application, we use them for:
- Sampling method selection (with/without replacement)
- Sample size mode (percentage vs. count)

### Basic Radio Button Creation

```python
from PyQt5.QtWidgets import QRadioButton, QButtonGroup

# Create individual radio buttons
radio1 = QRadioButton("Option 1")
radio2 = QRadioButton("Option 2")

# Set default selection
radio1.setChecked(True)

# Connect to handler
radio1.toggled.connect(self.on_radio_changed)
```

### Ensuring Mutual Exclusion with QButtonGroup

```python
def create_radio_group(self):
    """Create mutually exclusive radio button group"""
    # Create radio buttons
    self.replacement = QRadioButton("🔁 With Replacement")
    self.no_replacement = QRadioButton("🚫 Without Replacement")
    
    # Create button group for mutual exclusion
    self.replacement_group = QButtonGroup()
    self.replacement_group.addButton(self.replacement)
    self.replacement_group.addButton(self.no_replacement)
    
    # Set default selection
    self.no_replacement.setChecked(True)
    
    # Connect signals
    self.replacement.toggled.connect(self.on_method_changed)
    self.no_replacement.toggled.connect(self.on_method_changed)

def on_method_changed(self):
    """Handle sampling method change"""
    if self.replacement.isChecked():
        print("With replacement selected")
    else:
        print("Without replacement selected")
```

### Radio Button Styling

Our application uses custom styling to make radio buttons look like modern buttons:

```python
def get_compact_radio_style(self):
    """Custom styling for radio buttons to look like buttons"""
    return """
    QRadioButton {
        color: #2c3e50;
        font-weight: 600;
        font-size: 16px;
        padding: 12px 15px;
        background-color: #f8f9fa;
        border: 2px solid #e9ecef;
        border-radius: 8px;
        margin: 3px 0;
        min-height: 18px;
    }
    
    QRadioButton:hover {
        background-color: #e3f2fd;
        border-color: #3498db;
    }
    
    QRadioButton:checked {
        background-color: #e3f2fd;
        border-color: #3498db;
        color: #1976d2;
    }
    
    QRadioButton::indicator {
        width: 0px;
        height: 0px;
        margin-right: 0px;
    }
    """
```

**Key Styling Points:**
- **Hidden Indicators**: `width: 0px; height: 0px` removes the circular radio indicator
- **Button-like Appearance**: Padding and borders make them look like buttons
- **Visual Feedback**: Different colors for hover and checked states
- **Consistent Spacing**: Margins ensure proper alignment

## Method Selection Tab Implementation

### Complete Method Tab

```python
def create_method_tab(self):
    """Third tab: Sampling method selection"""
    method_tab = QWidget()
    layout = QVBoxLayout()
    layout.setSpacing(25)
    layout.setContentsMargins(20, 20, 20, 20)
    
    # Header section with clear instructions
    header_frame = QFrame()
    header_frame.setObjectName("welcomeFrame")
    header_layout = QVBoxLayout()
    
    header_title = QLabel("🔄 Step 2: Choose Sampling Method")
    header_title.setStyleSheet("font-size: 24px; font-weight: bold; color: #2c3e50; text-align: center; margin-bottom: 10px;")
    header_title.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_title)
    
    header_desc = QLabel("Select whether you want sampling with or without replacement.")
    header_desc.setStyleSheet("font-size: 16px; color: #34495e; text-align: center; padding: 10px;")
    header_desc.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_desc)
    
    header_frame.setLayout(header_layout)
    layout.addWidget(header_frame)
    
    # Method selection section - horizontal layout for side-by-side comparison
    method_frame = QFrame()
    method_frame.setObjectName("sectionFrame")
    method_layout = QHBoxLayout()
    method_layout.setSpacing(20)
    
    # With Replacement option
    replacement_container = QFrame()
    replacement_container.setObjectName("inputFrame")
    replacement_layout = QVBoxLayout()
    replacement_layout.setSpacing(15)
    
    self.replacement = QRadioButton("🔁 With Replacement")
    self.replacement.setStyleSheet(self.get_compact_radio_style())
    replacement_layout.addWidget(self.replacement)
    
    replacement_desc = QLabel("Allows the same data point to be selected multiple times. Useful for bootstrap sampling.")
    replacement_desc.setStyleSheet("color: #7f8c8d; font-size: 13px; padding: 10px; text-align: center;")
    replacement_desc.setWordWrap(True)
    replacement_layout.addWidget(replacement_desc)
    
    replacement_container.setLayout(replacement_layout)
    method_layout.addWidget(replacement_container)
    
    # Without Replacement option
    no_replacement_container = QFrame()
    no_replacement_container.setObjectName("inputFrame")
    no_replacement_layout = QVBoxLayout()
    no_replacement_layout.setSpacing(15)
    
    self.no_replacement = QRadioButton("🚫 Without Replacement")
    self.no_replacement.setChecked(True)  # Default selection
    self.no_replacement.setStyleSheet(self.get_compact_radio_style())
    no_replacement_layout.addWidget(self.no_replacement)
    
    no_replacement_desc = QLabel("Each data point can only be selected once. This is the standard approach for most scenarios.")
    no_replacement_desc.setStyleSheet("color: #7f8c8d; font-size: 13px; padding: 10px; text-align: center;")
    no_replacement_desc.setWordWrap(True)
    no_replacement_layout.addWidget(no_replacement_desc)
    
    no_replacement_container.setLayout(no_replacement_layout)
    method_layout.addWidget(no_replacement_container)
    
    # Create button group for mutual exclusion
    self.replacement_group = QButtonGroup()
    self.replacement_group.addButton(self.replacement)
    self.replacement_group.addButton(self.no_replacement)
    
    method_frame.setLayout(method_layout)
    layout.addWidget(method_frame)
    
    # Navigation buttons
    nav_frame = QFrame()
    nav_layout = QHBoxLayout()
    
    back_button = QPushButton("← Back to Data Preview")
    back_button.setObjectName("backButton")
    back_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(1))
    nav_layout.addWidget(back_button)
    
    nav_layout.addStretch()
    
    next_button = QPushButton("Next: Sample Size →")
    next_button.setObjectName("nextButton")
    next_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(3))
    nav_layout.addWidget(next_button)
    
    nav_frame.setLayout(nav_layout)
    layout.addWidget(nav_frame)
    
    layout.addStretch()
    method_tab.setLayout(layout)
    self.tab_widget.addTab(method_tab, "🔄 Method")
```

### Design Principles Demonstrated

1. **Clear Visual Hierarchy**: Header, content, navigation
2. **Side-by-Side Comparison**: Easy to compare options
3. **Descriptive Text**: Helps users understand implications
4. **Consistent Styling**: Uniform appearance across the application

## Slider Implementation and Dynamic Updates

### Understanding QSlider

`QSlider` provides an intuitive way to select numeric values:

```python
from PyQt5.QtWidgets import QSlider
from PyQt5.QtCore import Qt

# Create horizontal slider
slider = QSlider(Qt.Horizontal)
slider.setRange(1, 100)      # Min: 1, Max: 100
slider.setValue(10)          # Initial value: 10
slider.setTickPosition(QSlider.TicksBelow)  # Show tick marks

# Connect to update handler
slider.valueChanged.connect(self.on_slider_changed)
```

### Slider Configuration Options

#### Range and Values
```python
slider.setRange(minimum, maximum)    # Set min/max values
slider.setValue(value)               # Set current value
slider.value()                       # Get current value
```

#### Visual Appearance
```python
# Tick positions
slider.setTickPosition(QSlider.NoTicks)        # No ticks
slider.setTickPosition(QSlider.TicksAbove)     # Above slider
slider.setTickPosition(QSlider.TicksBelow)     # Below slider
slider.setTickPosition(QSlider.TicksBothSides) # Both sides

# Tick intervals
slider.setTickInterval(10)  # Tick every 10 units
```

### Dynamic Label Updates

```python
def create_percentage_slider(self):
    """Create slider with dynamic label updates"""
    # Label that shows current value
    self.slider_label = QLabel("Sample: 10%")
    self.slider_label.setStyleSheet("""
        QLabel {
            font-weight: bold; 
            font-size: 16px; 
            color: #2c3e50; 
            margin-bottom: 5px;
            text-align: center;
        }
        QLabel:disabled {
            color: #6c757d;
        }
    """)
    
    # Slider
    self.slider = QSlider(Qt.Horizontal)
    self.slider.setRange(1, 100)
    self.slider.setValue(10)
    self.slider.setTickPosition(QSlider.TicksBelow)
    
    # Connect to update function
    self.slider.valueChanged.connect(self.update_slider_label)

def update_slider_label(self, value):
    """Update label text when slider value changes"""
    self.slider_label.setText(f"Sample: {value}%")
```

### Slider Styling

```python
def get_slider_stylesheet(self):
    """Custom styling for slider"""
    return """
    QSlider::groove:horizontal {
        border: none;
        height: 12px;
        background: #ecf0f1;
        border-radius: 6px;
    }
    
    QSlider::groove:horizontal:disabled {
        background: #f8f9fa;
    }
    
    QSlider::handle:horizontal {
        background: #3498db;
        border: none;
        width: 26px;
        height: 26px;
        border-radius: 13px;
        margin: -7px 0;
    }
    
    QSlider::handle:horizontal:hover {
        background: #2980b9;
    }
    
    QSlider::handle:horizontal:disabled {
        background: #bdc3c7;
    }
    
    QSlider::sub-page:horizontal {
        background: #3498db;
        border-radius: 6px;
    }
    
    QSlider::sub-page:horizontal:disabled {
        background: #dee2e6;
    }
    """
```

## Input Validation and QLineEdit

### Text Input with Validation

`QLineEdit` provides text input with built-in validation:

```python
from PyQt5.QtWidgets import QLineEdit
from PyQt5.QtGui import QIntValidator

# Create input field
self.sample_count = QLineEdit()
self.sample_count.setText("100")  # Default value
self.sample_count.setPlaceholderText("Enter number of samples")

# Add numeric validation
validator = QIntValidator(1, 99999)  # Allow integers from 1 to 99999
self.sample_count.setValidator(validator)

# Connect to change handler
self.sample_count.textChanged.connect(self.on_count_changed)
```

### Validation Types

#### Numeric Validation
```python
# Integer validation
int_validator = QIntValidator(0, 1000)
line_edit.setValidator(int_validator)

# Double validation
double_validator = QDoubleValidator(0.0, 100.0, 2)  # 2 decimal places
line_edit.setValidator(double_validator)

# Regular expression validation
regex_validator = QRegExpValidator(QRegExp("[A-Za-z][A-Za-z0-9]*"))
line_edit.setValidator(regex_validator)
```

#### Custom Validation
```python
def validate_input(self, text):
    """Custom validation logic"""
    try:
        value = int(text)
        if value < 1:
            return False, "Value must be positive"
        if value > 10000:
            return False, "Value too large"
        return True, ""
    except ValueError:
        return False, "Must be a number"
```

### Input Field Styling

```python
def get_input_stylesheet(self):
    """Styling for input fields"""
    return """
    QLineEdit {
        padding: 12px 16px;
        border: 2px solid #bdc3c7;
        border-radius: 6px;
        background-color: white;
        color: #2c3e50;
        font-size: 16px;
        font-weight: bold;
        text-align: center;
        min-height: 25px;
    }
    
    QLineEdit:focus {
        border-color: #3498db;
        outline: none;
        background-color: #f8fbff;
    }
    
    QLineEdit:disabled {
        background-color: #f8f9fa;
        color: #6c757d;
        border-color: #e9ecef;
    }
    
    QLineEdit:enabled {
        border-color: #28a745;
        background-color: white;
    }
    """
```

## Dynamic UI Updates and State Management

### Toggle Input Mode Implementation

Our application allows users to choose between percentage and count-based sampling:

```python
def toggle_input_mode(self):
    """Enable/disable input fields based on selected sample mode"""
    if self.by_percent.isChecked():
        # Enable percentage mode
        self.slider.setEnabled(True)
        self.slider_label.setEnabled(True)
        # Disable count mode
        self.sample_count.setEnabled(False)
    else:
        # Disable percentage mode
        self.slider.setEnabled(False)
        self.slider_label.setEnabled(True)  # Keep label enabled for readability
        # Enable count mode
        self.sample_count.setEnabled(True)
        # Focus on the text input when count mode is selected
        self.sample_count.setFocus()
```

### State Synchronization

```python
def sync_ui_state(self):
    """Synchronize UI state across components"""
    # Update navigation buttons based on data availability
    has_data = self.df is not None
    
    # Enable/disable tabs based on prerequisites
    self.tab_widget.setTabEnabled(1, has_data)  # Data preview
    self.tab_widget.setTabEnabled(2, has_data)  # Method
    self.tab_widget.setTabEnabled(3, has_data)  # Size
    self.tab_widget.setTabEnabled(4, has_data)  # Settings
    
    # Update status indicators
    if has_data:
        self.update_data_status()
    else:
        self.clear_data_status()
```

### Focus Management

```python
def manage_focus(self):
    """Manage focus for better user experience"""
    # Set focus to active input
    if self.by_count.isChecked():
        self.sample_count.setFocus()
        self.sample_count.selectAll()  # Select all text for easy replacement
    
    # Set tab order
    self.setTabOrder(self.by_percent, self.slider)
    self.setTabOrder(self.slider, self.by_count)
    self.setTabOrder(self.by_count, self.sample_count)
```

## Sample Size Tab Implementation

### Complete Size Configuration Tab

```python
def create_size_tab(self):
    """Fourth tab: Sample size selection"""
    size_tab = QWidget()
    layout = QVBoxLayout()
    layout.setSpacing(25)
    layout.setContentsMargins(20, 20, 20, 20)
    
    # Header section
    header_frame = QFrame()
    header_frame.setObjectName("welcomeFrame")
    header_layout = QVBoxLayout()
    
    header_title = QLabel("📏 Step 3: Select Sample Size")
    header_title.setStyleSheet("font-size: 24px; font-weight: bold; color: #2c3e50; text-align: center; margin-bottom: 10px;")
    header_title.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_title)
    
    header_desc = QLabel("Choose how you want to specify the sample size:")
    header_desc.setStyleSheet("font-size: 16px; color: #34495e; text-align: center; padding: 10px;")
    header_desc.setAlignment(Qt.AlignCenter)
    header_layout.addWidget(header_desc)
    
    header_frame.setLayout(header_layout)
    layout.addWidget(header_frame)
    
    # Sample mode section - horizontal layout for comparison
    mode_frame = QFrame()
    mode_frame.setObjectName("sectionFrame")
    mode_layout = QHBoxLayout()
    mode_layout.setSpacing(20)
    
    # Left side - Percentage option
    percent_container = QFrame()
    percent_container.setObjectName("inputFrame")
    percent_layout = QVBoxLayout()
    percent_layout.setSpacing(15)
    
    self.by_percent = QRadioButton("📊 By Percentage")
    self.by_percent.setChecked(True)  # Default selection
    self.by_percent.setStyleSheet(self.get_compact_radio_style())
    self.by_percent.toggled.connect(self.toggle_input_mode)
    percent_layout.addWidget(self.by_percent)
    
    # Dynamic label that updates with slider
    self.slider_label = QLabel("Sample: 10%")
    self.slider_label.setStyleSheet("""
        QLabel {
            font-weight: bold; 
            font-size: 16px; 
            color: #2c3e50; 
            margin-bottom: 5px;
            text-align: center;
        }
        QLabel:disabled {
            color: #6c757d;
        }
    """)
    percent_layout.addWidget(self.slider_label)
    
    # Percentage slider
    self.slider = QSlider(Qt.Horizontal)
    self.slider.setRange(1, 100)
    self.slider.setValue(10)
    self.slider.setTickPosition(QSlider.TicksBelow)
    self.slider.valueChanged.connect(self.update_slider_label)
    self.slider.setStyleSheet(self.get_slider_stylesheet())
    percent_layout.addWidget(self.slider)
    
    percent_container.setLayout(percent_layout)
    mode_layout.addWidget(percent_container)
    
    # Right side - Count option
    count_container = QFrame()
    count_container.setObjectName("inputFrame")
    count_layout = QVBoxLayout()
    count_layout.setSpacing(15)
    
    self.by_count = QRadioButton("🔢 By Count")
    self.by_count.setStyleSheet(self.get_compact_radio_style())
    self.by_count.toggled.connect(self.toggle_input_mode)
    count_layout.addWidget(self.by_count)
    
    count_label = QLabel("Number of samples:")
    count_label.setStyleSheet("font-weight: bold; font-size: 14px; color: #2c3e50; margin-bottom: 5px;")
    count_layout.addWidget(count_label)
    
    # Count input field
    self.sample_count = QLineEdit()
    self.sample_count.setText("100")
    self.sample_count.setValidator(QIntValidator(1, 99999))
    self.sample_count.setEnabled(False)  # Disabled by default
    self.sample_count.setStyleSheet(self.get_input_stylesheet())
    count_layout.addWidget(self.sample_count)
    
    count_container.setLayout(count_layout)
    mode_layout.addWidget(count_container)
    
    # Create button group for mutual exclusion
    self.sample_mode = QButtonGroup()
    self.sample_mode.addButton(self.by_percent)
    self.sample_mode.addButton(self.by_count)
    
    mode_frame.setLayout(mode_layout)
    layout.addWidget(mode_frame)
    
    # Navigation buttons
    nav_frame = QFrame()
    nav_layout = QHBoxLayout()
    
    back_button = QPushButton("← Back to Method")
    back_button.setObjectName("backButton")
    back_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(2))
    nav_layout.addWidget(back_button)
    
    nav_layout.addStretch()
    
    next_button = QPushButton("Next: Settings →")
    next_button.setObjectName("nextButton")
    next_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(4))
    nav_layout.addWidget(next_button)
    
    nav_frame.setLayout(nav_layout)
    layout.addWidget(nav_frame)
    
    layout.addStretch()
    size_tab.setLayout(layout)
    self.tab_widget.addTab(size_tab, "📏 Size")
```

## Advanced Input Handling

### Real-time Validation

```python
def setup_real_time_validation(self):
    """Set up real-time input validation"""
    self.sample_count.textChanged.connect(self.validate_count_input)

def validate_count_input(self, text):
    """Validate count input in real-time"""
    if not text:
        self.show_input_status("Enter a number", "warning")
        return
    
    try:
        value = int(text)
        if value < 1:
            self.show_input_status("Value must be positive", "error")
        elif value > len(self.df) if self.df is not None else value > 10000:
            self.show_input_status("Value too large", "error")
        else:
            self.show_input_status("Valid input", "success")
    except ValueError:
        self.show_input_status("Must be a number", "error")

def show_input_status(self, message, status_type):
    """Show input validation status"""
    colors = {
        "success": "#27ae60",
        "warning": "#f39c12",
        "error": "#e74c3c"
    }
    
    # Update status label (if exists)
    if hasattr(self, 'input_status_label'):
        self.input_status_label.setText(message)
        self.input_status_label.setStyleSheet(f"color: {colors.get(status_type, '#333')};")
```

### Keyboard Shortcuts

```python
def setup_keyboard_shortcuts(self):
    """Set up keyboard shortcuts for better UX"""
    from PyQt5.QtWidgets import QShortcut
    from PyQt5.QtGui import QKeySequence
    
    # Ctrl+O to open file
    open_shortcut = QShortcut(QKeySequence.Open, self)
    open_shortcut.activated.connect(self.load_csv)
    
    # Ctrl+S to export results
    save_shortcut = QShortcut(QKeySequence.Save, self)
    save_shortcut.activated.connect(self.export_results)
    
    # F5 to refresh/resample
    refresh_shortcut = QShortcut(QKeySequence.Refresh, self)
    refresh_shortcut.activated.connect(self.perform_sampling)
```

## Chapter Summary

In this chapter, we implemented sophisticated interactive UI elements:

1. **Radio Button Groups**: Mutually exclusive selections with custom styling
2. **Dynamic Sliders**: Real-time value updates with visual feedback
3. **Input Validation**: Robust text input with multiple validation types
4. **State Management**: Coordinated UI updates based on user selections
5. **Focus Management**: Improved user experience through proper focus handling
6. **Keyboard Shortcuts**: Power user features for common actions

### Key Design Patterns

- **Observer Pattern**: UI elements responding to state changes
- **Strategy Pattern**: Different input modes (percentage vs. count)
- **Template Method**: Consistent styling and behavior across elements
- **State Pattern**: UI behavior changing based on application state

### User Experience Improvements

- **Visual Feedback**: Clear indication of selected options and input validity
- **Progressive Enhancement**: Basic functionality with advanced features
- **Accessibility**: Keyboard navigation and proper focus management
- **Consistency**: Uniform styling and behavior across all elements

In the next chapter, we'll dive deep into advanced styling and CSS techniques that give our application its professional, modern appearance.

---

**Key Takeaways:**
- Interactive elements should provide immediate feedback
- State management prevents inconsistent UI behavior
- Validation should happen in real-time when possible
- Consistent styling creates a professional appearance
- Accessibility features benefit all users
