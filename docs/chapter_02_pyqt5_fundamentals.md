# Chapter 2: Understanding PyQt5 Fundamentals

## The PyQt5 Framework

PyQt5 is a comprehensive set of Python bindings for the Qt application framework. Qt is a cross-platform C++ framework that has been adapted for Python, making it possible to create desktop applications with native look and feel across different operating systems.

## Core Concepts

### 1. The Qt Object Model

Every PyQt5 application is built on the Qt object model, which provides:
- **Hierarchical Organization**: Parent-child relationships between objects
- **Automatic Memory Management**: Child objects are automatically deleted when parents are destroyed
- **Signal-Slot Communication**: Event-driven programming model

### 2. Application Structure

Every PyQt5 application follows this basic structure:

```python
import sys
from PyQt5.QtWidgets import QApplication, QWidget

class MyApp(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()
    
    def init_ui(self):
        # Initialize user interface
        self.setWindowTitle('My Application')
        self.setGeometry(100, 100, 800, 600)
        self.show()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MyApp()
    sys.exit(app.exec_())
```

Let's break down each component:

#### QApplication
- **Purpose**: Manages the application's main event loop
- **Responsibilities**: 
  - Handles command-line arguments
  - Manages application-wide settings
  - Processes events and user input
  - Coordinates with the operating system

```python
# Creating the application object
app = QApplication(sys.argv)  # sys.argv passes command-line arguments
```

#### QWidget
- **Purpose**: Base class for all user interface objects
- **Characteristics**:
  - Can contain other widgets (parent-child relationship)
  - Can receive events (mouse, keyboard, etc.)
  - Can be styled with CSS-like stylesheets

```python
class MyApp(QWidget):  # Inheriting from QWidget
    def __init__(self):
        super().__init__()  # Initialize parent class
```

#### Event Loop
- **Purpose**: Continuously processes events
- **How it works**:
  1. Waits for events (user input, system messages)
  2. Dispatches events to appropriate handlers
  3. Updates the user interface
  4. Repeats until application exits

```python
sys.exit(app.exec_())  # Start the event loop
```

## Essential Widgets

### Layout Widgets

PyQt5 provides several layout managers to organize widgets:

#### QVBoxLayout (Vertical Layout)
```python
from PyQt5.QtWidgets import QVBoxLayout, QLabel, QPushButton

layout = QVBoxLayout()
layout.addWidget(QLabel("Top"))
layout.addWidget(QPushButton("Middle"))
layout.addWidget(QLabel("Bottom"))
```

#### QHBoxLayout (Horizontal Layout)
```python
from PyQt5.QtWidgets import QHBoxLayout

layout = QHBoxLayout()
layout.addWidget(QLabel("Left"))
layout.addWidget(QPushButton("Center"))
layout.addWidget(QLabel("Right"))
```

#### Layout Properties
- **Spacing**: Controls space between widgets
- **Margins**: Controls space around the layout
- **Stretch**: Allows widgets to expand/contract

```python
layout = QVBoxLayout()
layout.setSpacing(10)  # 10 pixels between widgets
layout.setContentsMargins(20, 20, 20, 20)  # 20px margins on all sides
layout.addStretch()  # Adds expandable space
```

### Input Widgets

#### QPushButton
The most common interactive widget:

```python
from PyQt5.QtWidgets import QPushButton

button = QPushButton("Click Me")
button.clicked.connect(self.on_button_click)  # Connect to handler

def on_button_click(self):
    print("Button was clicked!")
```

#### QLineEdit
For text input:

```python
from PyQt5.QtWidgets import QLineEdit
from PyQt5.QtGui import QIntValidator

line_edit = QLineEdit()
line_edit.setPlaceholderText("Enter your name")
line_edit.textChanged.connect(self.on_text_changed)  # Signal connection

# For numeric input only
validator = QIntValidator(0, 100)
line_edit.setValidator(validator)
```

#### QRadioButton
For mutually exclusive choices:

```python
from PyQt5.QtWidgets import QRadioButton, QButtonGroup

radio1 = QRadioButton("Option 1")
radio2 = QRadioButton("Option 2")

# Group radio buttons for mutual exclusion
group = QButtonGroup()
group.addButton(radio1)
group.addButton(radio2)

radio1.setChecked(True)  # Set default selection
```

#### QSlider
For value selection:

```python
from PyQt5.QtWidgets import QSlider
from PyQt5.QtCore import Qt

slider = QSlider(Qt.Horizontal)
slider.setRange(0, 100)
slider.setValue(50)
slider.valueChanged.connect(self.on_value_changed)

def on_value_changed(self, value):
    print(f"Slider value: {value}")
```

### Display Widgets

#### QLabel
For displaying text and images:

```python
from PyQt5.QtWidgets import QLabel
from PyQt5.QtCore import Qt

label = QLabel("Hello, World!")
label.setAlignment(Qt.AlignCenter)  # Center alignment
label.setWordWrap(True)  # Enable word wrapping
```

#### QTableWidget
For tabular data display:

```python
from PyQt5.QtWidgets import QTableWidget, QTableWidgetItem

table = QTableWidget()
table.setRowCount(3)
table.setColumnCount(2)
table.setHorizontalHeaderLabels(['Name', 'Age'])

# Add data
table.setItem(0, 0, QTableWidgetItem('Alice'))
table.setItem(0, 1, QTableWidgetItem('25'))

# Enable sorting
table.setSortingEnabled(True)
```

### Container Widgets

#### QTabWidget
For organizing content in tabs:

```python
from PyQt5.QtWidgets import QTabWidget, QWidget

tab_widget = QTabWidget()

# Create tab pages
tab1 = QWidget()
tab2 = QWidget()

# Add tabs
tab_widget.addTab(tab1, "Tab 1")
tab_widget.addTab(tab2, "Tab 2")

# Switch tabs programmatically
tab_widget.setCurrentIndex(1)
```

#### QFrame
For grouping and styling:

```python
from PyQt5.QtWidgets import QFrame

frame = QFrame()
frame.setObjectName("myFrame")  # For CSS styling
frame.setStyleSheet("QFrame#myFrame { border: 1px solid gray; }")
```

## Signals and Slots

The signal-slot mechanism is PyQt5's way of handling events and communication between objects.

### What are Signals?
- **Signals**: Notifications emitted when something happens
- **Examples**: Button clicked, text changed, value selected

### What are Slots?
- **Slots**: Functions that respond to signals
- **Can be**: Class methods, standalone functions, or built-in methods

### Connecting Signals and Slots

#### Basic Connection
```python
# Connect button click to a method
button.clicked.connect(self.handle_click)

def handle_click(self):
    print("Button clicked!")
```

#### Lambda Functions
```python
# Use lambda for simple operations
button.clicked.connect(lambda: print("Button clicked!"))

# Pass parameters
button.clicked.connect(lambda: self.set_value(42))
```

#### Built-in Slots
```python
# Connect to built-in slots
button.clicked.connect(self.close)  # Close window
button.clicked.connect(app.quit)    # Quit application
```

### Custom Signals

You can create custom signals for your own classes:

```python
from PyQt5.QtCore import pyqtSignal

class MyWidget(QWidget):
    # Define custom signal
    valueChanged = pyqtSignal(int)
    
    def update_value(self, value):
        self.valueChanged.emit(value)  # Emit the signal
```

## Styling with StyleSheets

PyQt5 supports CSS-like styling for customizing appearance:

### Basic Styling
```python
widget.setStyleSheet("""
    QWidget {
        background-color: #f0f0f0;
        font-family: Arial, sans-serif;
        font-size: 14px;
    }
""")
```

### Selector Types

#### Type Selectors
```css
QPushButton {
    background-color: blue;
    color: white;
}
```

#### ID Selectors
```css
QPushButton#myButton {
    background-color: red;
}
```

#### Class Selectors
```css
.QPushButton {
    padding: 10px;
}
```

#### State Selectors
```css
QPushButton:hover {
    background-color: lightblue;
}

QPushButton:pressed {
    background-color: darkblue;
}

QPushButton:disabled {
    background-color: gray;
}
```

### Common Style Properties

#### Colors and Backgrounds
```css
QWidget {
    background-color: #ffffff;
    color: #333333;
    border: 1px solid #cccccc;
}
```

#### Fonts and Text
```css
QLabel {
    font-family: "Segoe UI", Arial, sans-serif;
    font-size: 14px;
    font-weight: bold;
    text-align: center;
}
```

#### Padding and Margins
```css
QPushButton {
    padding: 10px 20px;
    margin: 5px;
    border-radius: 5px;
}
```

### Styling Our Application

Here's how we'll style our sampling application:

```python
def get_app_stylesheet(self):
    return """
    QWidget {
        background-color: #f0f2f5;
        font-family: 'Segoe UI', Arial, sans-serif;
        font-size: 14px;
    }
    
    QPushButton {
        background-color: #3498db;
        color: white;
        border: none;
        padding: 15px 30px;
        border-radius: 8px;
        font-weight: bold;
        font-size: 14px;
        min-height: 25px;
        text-align: center;
    }
    
    QPushButton:hover {
        background-color: #2980b9;
    }
    
    QPushButton:pressed {
        background-color: #21618c;
    }
    """
```

## Event Handling

Understanding how events work in PyQt5 is crucial for creating interactive applications.

### Event Types
- **Mouse Events**: Clicks, movements, wheel scrolling
- **Keyboard Events**: Key presses, releases
- **Window Events**: Resize, close, minimize
- **Custom Events**: Application-specific events

### Event Handling Methods

#### Method 1: Signal-Slot Connections (Recommended)
```python
button.clicked.connect(self.on_button_click)
```

#### Method 2: Event Reimplementation
```python
def mousePressEvent(self, event):
    if event.button() == Qt.LeftButton:
        print("Left mouse button pressed")
    super().mousePressEvent(event)

def keyPressEvent(self, event):
    if event.key() == Qt.Key_Escape:
        self.close()
    super().keyPressEvent(event)
```

#### Method 3: Event Filters
```python
def eventFilter(self, source, event):
    if event.type() == QEvent.KeyPress:
        if event.key() == Qt.Key_Enter:
            print("Enter key pressed")
    return super().eventFilter(source, event)
```

## Memory Management

PyQt5 handles memory management automatically through its parent-child system:

### Parent-Child Relationships
```python
# Parent widget
parent = QWidget()

# Child widgets are automatically deleted when parent is destroyed
child1 = QLabel("Child 1", parent)
child2 = QPushButton("Child 2", parent)

# Alternative way to set parent
child3 = QLabel("Child 3")
child3.setParent(parent)
```

### Layout Management
```python
# Layouts automatically manage their child widgets
layout = QVBoxLayout()
layout.addWidget(QLabel("This widget will be automatically managed"))
parent.setLayout(layout)
```

## Best Practices

### 1. Code Organization
```python
class SamplingApp(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()
        self.connect_signals()
    
    def init_ui(self):
        # Create and configure widgets
        pass
    
    def connect_signals(self):
        # Connect signals to slots
        pass
```

### 2. Error Handling
```python
def load_file(self):
    try:
        filename, _ = QFileDialog.getOpenFileName(self, "Open File")
        if filename:
            self.process_file(filename)
    except Exception as e:
        QMessageBox.critical(self, "Error", f"Could not load file: {str(e)}")
```

### 3. User Feedback
```python
def long_running_operation(self):
    # Show progress to user
    progress = QProgressDialog("Processing...", "Cancel", 0, 100, self)
    progress.show()
    
    # Perform operation with progress updates
    for i in range(100):
        progress.setValue(i)
        QApplication.processEvents()  # Keep UI responsive
```

## Chapter Summary

In this chapter, we covered:

1. **PyQt5 Architecture**: Understanding the Qt object model and event-driven programming
2. **Essential Widgets**: Layout managers, input widgets, display widgets, and containers
3. **Signals and Slots**: Event handling and communication between objects
4. **Styling**: CSS-like styling for professional appearance
5. **Event Handling**: Various methods for handling user interactions
6. **Memory Management**: Automatic cleanup through parent-child relationships
7. **Best Practices**: Code organization, error handling, and user feedback

These fundamentals form the foundation for building our sampling application. In the next chapter, we'll explore the application architecture and design patterns that make our code maintainable and extensible.

---

**Key Takeaways:**
- PyQt5 uses a hierarchical object model with automatic memory management
- Signals and slots provide elegant event handling
- CSS-like styling enables professional appearance
- Proper code organization and error handling are essential
- The event loop is the heart of every PyQt5 application
