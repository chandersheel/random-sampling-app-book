# Chapter 4: Building the Main Window and Tab System

## Introduction

In this chapter, we'll implement the core structure of our application: the main window class and the tab-based navigation system. This forms the backbone of our user interface and demonstrates how to create a professional, multi-step workflow application.

## The SamplingApp Class Foundation

### Class Declaration and Initialization

Let's start by examining the main application class:

```python
import sys
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QPushButton, QLabel, QFileDialog,
    QRadioButton, QHBoxLayout, QSlider, QButtonGroup, QSpinBox, QMessageBox, 
    QTableWidget, QTableWidgetItem, QTabWidget, QScrollArea, QFrame, QLineEdit
)
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QIntValidator
import pandas as pd
from sampling import perform_sampling

class SamplingApp(QWidget):
    """
    Main application window for the Random Sampling Application.
    
    This class inherits from QWidget and serves as the primary container
    for all user interface elements. It orchestrates the tab-based workflow
    and manages the application's state.
    """
    
    def __init__(self):
        """
        Initialize the SamplingApp instance.
        
        This constructor performs several critical tasks:
        1. Calls the parent class constructor
        2. Sets up window properties
        3. Initializes data storage variables
        4. Creates the user interface
        """
        super().__init__()
        
        # Window configuration
        self.setWindowTitle("Randomly Yours App")
        self.setGeometry(100, 100, 1000, 700)  # x, y, width, height
        self.setMinimumSize(900, 650)           # Minimum window size
        self.setMaximumSize(1400, 1000)         # Maximum window size
        
        # Apply application-wide styling
        self.setStyleSheet(self.get_app_stylesheet())
        
        # Data storage
        self.df = None              # Will hold the loaded pandas DataFrame
        self.current_results = None # Will hold sampling results
        
        # Initialize the user interface
        self.init_ui()
```

### Understanding the Constructor

Let's break down each part of the `__init__` method:

#### 1. Parent Class Initialization
```python
super().__init__()
```
This calls the `QWidget` constructor, which:
- Initializes the widget's internal state
- Sets up the widget for event handling
- Prepares the widget for layout management

#### 2. Window Properties
```python
self.setWindowTitle("Randomly Yours App")
self.setGeometry(100, 100, 1000, 700)
self.setMinimumSize(900, 650)
self.setMaximumSize(1400, 1000)
```

- **setWindowTitle()**: Sets the text shown in the window's title bar
- **setGeometry()**: Sets initial position (100, 100) and size (1000×700)
- **setMinimumSize()**: Prevents the window from being resized too small
- **setMaximumSize()**: Limits how large the window can grow

#### 3. Styling Application
```python
self.setStyleSheet(self.get_app_stylesheet())
```
This applies a comprehensive CSS-like stylesheet to the entire application, ensuring consistent visual appearance across all widgets.

#### 4. Data Variables
```python
self.df = None
self.current_results = None
```
These instance variables store the application's data:
- **self.df**: The original DataFrame loaded from CSV
- **self.current_results**: The results of sampling operations

## Creating the User Interface

### The init_ui Method

```python
def init_ui(self):
    """
    Initialize the user interface components.
    
    This method sets up the main layout and creates all the tabs
    that make up the application's workflow.
    """
    # Create the main layout
    main_layout = QVBoxLayout()
    main_layout.setContentsMargins(10, 10, 10, 10)
    
    # Create the tab widget
    self.tab_widget = QTabWidget()
    self.tab_widget.setTabPosition(QTabWidget.North)
    
    # Create all tabs
    self.create_file_tab()
    self.create_data_preview_tab()
    self.create_method_tab()
    self.create_size_tab()
    self.create_advanced_tab()
    self.create_results_tab()
    
    # Add tab widget to main layout
    main_layout.addWidget(self.tab_widget)
    
    # Set the layout for this widget
    self.setLayout(main_layout)
```

### Understanding Layout Management

#### QVBoxLayout Basics
```python
main_layout = QVBoxLayout()
main_layout.setContentsMargins(10, 10, 10, 10)
```

**QVBoxLayout** arranges widgets vertically (top to bottom):
- Widgets are added in order using `addWidget()`
- Each widget gets a horizontal "slice" of the available space
- **setContentsMargins()** adds space around the layout edges

#### Layout Margins Explained
```python
setContentsMargins(left, top, right, bottom)
```
- **left**: Space on the left side (10 pixels)
- **top**: Space at the top (10 pixels)  
- **right**: Space on the right side (10 pixels)
- **bottom**: Space at the bottom (10 pixels)

## The Tab Widget System

### Understanding QTabWidget

The `QTabWidget` is a container that displays pages in tabs:

```python
self.tab_widget = QTabWidget()
self.tab_widget.setTabPosition(QTabWidget.North)
```

#### Tab Positions
- **QTabWidget.North**: Tabs at the top (our choice)
- **QTabWidget.South**: Tabs at the bottom
- **QTabWidget.West**: Tabs on the left side
- **QTabWidget.East**: Tabs on the right side

#### Adding Tabs
```python
tab_widget.addTab(widget, "Tab Title")
```
- **widget**: The QWidget that becomes the tab's content
- **"Tab Title"**: Text displayed on the tab

### Tab Creation Pattern

Each tab follows a consistent creation pattern:

```python
def create_example_tab(self):
    """Template for creating a tab"""
    # 1. Create the tab widget
    tab = QWidget()
    
    # 2. Create the main layout
    layout = QVBoxLayout()
    layout.setSpacing(15)
    layout.setContentsMargins(15, 15, 15, 15)
    
    # 3. Create header section
    header = self.create_header("Tab Title", "Description")
    layout.addWidget(header)
    
    # 4. Create content section
    content = self.create_content()
    layout.addWidget(content)
    
    # 5. Create navigation section
    navigation = self.create_navigation()
    layout.addWidget(navigation)
    
    # 6. Set layout and add to tab widget
    tab.setLayout(layout)
    self.tab_widget.addTab(tab, "📊 Tab")
```

## Creating Individual Tabs

### File Selection Tab

Let's examine the first tab in detail:

```python
def create_file_tab(self):
    """First tab: File selection and preview"""
    file_tab = QWidget()
    layout = QVBoxLayout()
    layout.setSpacing(15)
    layout.setContentsMargins(15, 15, 15, 15)
    
    # Welcome section - creates visual appeal
    welcome_frame = QFrame()
    welcome_frame.setObjectName("welcomeFrame")
    welcome_layout = QVBoxLayout()
    welcome_layout.setSpacing(10)
    
    # Main banner with application name
    banner_label = QLabel("🎲 Randomly Yours App")
    banner_label.setAlignment(Qt.AlignCenter)
    banner_label.setStyleSheet("""
        QLabel {
            color: #2c3e50;
            font-size: 28px;
            font-weight: bold;
            padding: 25px 20px;
            background: qlineargradient(x1:0, y1:0, x2:1, y2:1, 
                stop:0 #e3f2fd, stop:0.5 #bbdefb, stop:1 #90caf9);
            border: 2px solid #3498db;
            border-radius: 12px;
            margin: 5px 0;
            min-height: 50px;
        }
    """)
    
    # Subtitle for additional branding
    subtitle_label = QLabel("✨ Professional Random Sampling Tool ✨")
    subtitle_label.setAlignment(Qt.AlignCenter)
    subtitle_label.setStyleSheet("""
        QLabel {
            color: #1976d2;
            font-size: 16px;
            font-weight: 600;
            padding: 10px;
            background-color: #f8f9fa;
            border: 1px solid #e9ecef;
            border-radius: 8px;
            margin: 3px 0;
        }
    """)
    
    welcome_layout.addWidget(banner_label)
    welcome_layout.addWidget(subtitle_label)
    
    # Description text
    desc_label = QLabel("📊 Welcome to Randomly Yours App!\n\nThis tool helps you perform statistical sampling on your CSV data files.\nStart by selecting a CSV file to analyze.")
    desc_label.setAlignment(Qt.AlignCenter)
    desc_label.setStyleSheet("color: #34495e; font-size: 14px; line-height: 1.4; padding: 12px;")
    welcome_layout.addWidget(desc_label)
    
    welcome_frame.setLayout(welcome_layout)
    layout.addWidget(welcome_frame)
    
    # File selection section
    file_frame = QFrame()
    file_frame.setObjectName("sectionFrame")
    file_layout = QVBoxLayout()
    file_layout.setSpacing(8)
    
    file_header = QLabel("📁 Step 1: Select Your Data File")
    file_header.setStyleSheet("font-size: 16px; font-weight: bold; color: #2c3e50; margin-bottom: 8px;")
    file_layout.addWidget(file_header)
    
    self.choose_button = QPushButton("📁 Choose CSV File")
    self.choose_button.setObjectName("chooseButton")
    self.choose_button.clicked.connect(self.load_csv)
    file_layout.addWidget(self.choose_button)
    
    self.file_status = QLabel("No file selected")
    self.file_status.setStyleSheet("color: #7f8c8d; font-style: italic; margin-top: 8px;")
    file_layout.addWidget(self.file_status)
    
    file_frame.setLayout(file_layout)
    layout.addWidget(file_frame)
    
    # Data preview section (initially hidden)
    self.preview_frame = QFrame()
    self.preview_frame.setObjectName("sectionFrame")
    self.preview_frame.setVisible(False)  # Hidden until file is loaded
    preview_layout = QVBoxLayout()
    preview_layout.setSpacing(8)
    
    preview_header = QLabel("📋 Data Preview")
    preview_header.setStyleSheet("font-size: 16px; font-weight: bold; color: #2c3e50; margin-bottom: 8px;")
    preview_layout.addWidget(preview_header)
    
    # Scroll area for table
    scroll_area = QScrollArea()
    scroll_area.setWidgetResizable(True)
    scroll_area.setMinimumHeight(150)
    scroll_area.setMaximumHeight(200)
    
    self.preview_table = QTableWidget()
    scroll_area.setWidget(self.preview_table)
    preview_layout.addWidget(scroll_area)
    
    self.preview_frame.setLayout(preview_layout)
    layout.addWidget(self.preview_frame)
    
    layout.addStretch()  # Push everything to the top
    file_tab.setLayout(layout)
    self.tab_widget.addTab(file_tab, "📁 File")
```

### Understanding Frame Usage

#### QFrame for Visual Grouping
```python
welcome_frame = QFrame()
welcome_frame.setObjectName("welcomeFrame")
```

**QFrame** provides visual grouping and styling hooks:
- **setObjectName()**: Creates a CSS selector target
- Frames can have borders, backgrounds, and spacing
- Help organize related content visually

#### Object Names for Styling
```python
file_frame.setObjectName("sectionFrame")
```

Object names allow specific CSS targeting:
```css
QFrame#welcomeFrame {
    background-color: #ffffff;
    border: 2px solid #3498db;
    border-radius: 12px;
}

QFrame#sectionFrame {
    background-color: #ffffff;
    border: 1px solid #e9ecef;
    border-radius: 8px;
}
```

### Progressive Disclosure Pattern

```python
self.preview_frame.setVisible(False)
```

The preview section is initially hidden and only shown after a file is loaded. This is called **progressive disclosure**:
- Reduces cognitive load
- Shows relevant information when needed
- Creates a sense of progress

## Navigation Between Tabs

### Programmatic Tab Switching

```python
# Switch to a specific tab by index
self.tab_widget.setCurrentIndex(1)  # Switch to second tab (0-based)

# Connect button to tab navigation
back_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(0))
next_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(2))
```

### Navigation Button Pattern

Each tab (except the first and last) has navigation buttons:

```python
def create_navigation_buttons(self, back_index, next_index, back_text="← Back", next_text="Next →"):
    """Create standard navigation buttons for tabs"""
    nav_frame = QFrame()
    nav_layout = QHBoxLayout()
    
    # Back button
    back_button = QPushButton(back_text)
    back_button.setObjectName("backButton")
    back_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(back_index))
    nav_layout.addWidget(back_button)
    
    # Stretch to push next button to the right
    nav_layout.addStretch()
    
    # Next button
    next_button = QPushButton(next_text)
    next_button.setObjectName("nextButton")
    next_button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(next_index))
    nav_layout.addWidget(next_button)
    
    nav_frame.setLayout(nav_layout)
    return nav_frame
```

## Lambda Functions in Signal Connections

### Understanding Lambda Usage

```python
button.clicked.connect(lambda: self.tab_widget.setCurrentIndex(2))
```

**Lambda functions** are anonymous functions perfect for simple signal connections:
- **lambda:** defines an anonymous function
- **No parameters needed** for simple actions
- **Cleaner than defining separate methods** for simple operations

### Alternative Approaches

#### Method 1: Lambda (Recommended for simple actions)
```python
button.clicked.connect(lambda: self.some_action())
```

#### Method 2: Dedicated Method
```python
def on_button_click(self):
    self.some_action()

button.clicked.connect(self.on_button_click)
```

#### Method 3: Partial Functions
```python
from functools import partial

button.clicked.connect(partial(self.change_tab, 2))
```

## Tab Index Management

### Why Tab Indices Matter

With multiple tabs, keeping track of indices is crucial:

```
Index 0: File Tab
Index 1: Data Preview Tab  
Index 2: Method Tab
Index 3: Size Tab
Index 4: Settings Tab
Index 5: Results Tab
```

### Index Management Best Practices

#### 1. Use Constants
```python
class TabIndices:
    FILE = 0
    DATA_PREVIEW = 1
    METHOD = 2
    SIZE = 3
    SETTINGS = 4
    RESULTS = 5

# Usage
self.tab_widget.setCurrentIndex(TabIndices.RESULTS)
```

#### 2. Document Index Changes
When adding/removing tabs, update all navigation code:
```python
# If we add a tab between File and Data Preview:
# All subsequent indices shift by 1
# Update all setCurrentIndex() calls accordingly
```

## Widget Reference Management

### Storing Widget References

```python
def create_file_tab(self):
    # Store references to widgets we need to access later
    self.choose_button = QPushButton("📁 Choose CSV File")
    self.file_status = QLabel("No file selected")
    self.preview_frame = QFrame()
    self.preview_table = QTableWidget()
```

### Why Store References?

We store references to widgets that we need to:
- **Modify later**: Change text, visibility, or properties
- **Connect signals**: Add event handlers
- **Access data**: Get current values or states

### Instance Variable Naming

Follow consistent naming conventions:
```python
# Buttons: action_button
self.choose_button = QPushButton()
self.export_button = QPushButton()

# Labels: content_label
self.file_status = QLabel()
self.results_info = QLabel()

# Tables: purpose_table
self.preview_table = QTableWidget()
self.results_table = QTableWidget()

# Frames: purpose_frame
self.preview_frame = QFrame()
self.results_frame = QFrame()
```

## Stretch and Spacing

### Layout Stretch

```python
layout.addStretch()
```

**addStretch()** adds expandable space:
- Pushes other widgets to one direction
- Takes up extra space when window is resized
- Useful for aligning widgets to top, bottom, left, or right

### Widget Spacing

```python
layout.setSpacing(15)  # Space between widgets
layout.setContentsMargins(15, 15, 15, 15)  # Space around layout edges
```

**Spacing vs. Margins:**
- **Spacing**: Distance between widgets in the layout
- **Margins**: Distance from layout edges to container edges

## Chapter Summary

In this chapter, we implemented:

1. **Main Application Class**: The `SamplingApp` class structure with proper initialization
2. **Window Configuration**: Size constraints, title, and positioning
3. **Tab Widget System**: Creating and managing multiple tabs for workflow
4. **Layout Management**: Using `QVBoxLayout` and proper spacing
5. **Frame Organization**: Using `QFrame` for visual grouping and styling
6. **Navigation System**: Programmatic tab switching with buttons
7. **Progressive Disclosure**: Showing/hiding UI elements based on state
8. **Widget References**: Storing and managing widget instances
9. **Signal Connections**: Using lambda functions for simple actions

### Key Design Patterns Demonstrated

- **Template Method**: Consistent tab creation pattern
- **Progressive Disclosure**: Revealing information as needed
- **Visual Hierarchy**: Using frames and styling for organization
- **State Management**: Tracking application state through widget references

In the next chapter, we'll implement file handling and data preview functionality, bringing our first tab to life with actual file operations and data display.

---

**Key Takeaways:**
- Consistent patterns make code maintainable and understandable
- Proper layout management creates professional-looking interfaces
- Widget references enable dynamic UI updates
- Tab-based interfaces provide excellent user experience for multi-step workflows
- Progressive disclosure reduces cognitive load and guides users through complex processes
