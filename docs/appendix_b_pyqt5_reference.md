# Appendix B: PyQt5 Reference Guide

## Introduction

This appendix provides a comprehensive reference guide for PyQt5, covering the essential widgets, layouts, and concepts used in desktop application development. This guide serves as a quick reference for developers working with PyQt5.

## Core Concepts

### 1. The Application Object

Every PyQt5 application starts with a `QApplication` object:

```python
import sys
from PyQt5.QtWidgets import QApplication, QMainWindow

# Create the application
app = QApplication(sys.argv)

# Create and show the main window
window = QMainWindow()
window.show()

# Start the event loop
sys.exit(app.exec_())
```

### 2. Widgets and Hierarchy

PyQt5 uses a parent-child hierarchy:
- Every widget can have a parent widget
- Child widgets are automatically destroyed when parent is destroyed
- Child widgets are drawn within their parent's boundaries

```python
# Parent widget
parent = QWidget()

# Child widget
child = QLabel("Hello", parent)  # parent specified in constructor
# or
child = QLabel("Hello")
child.setParent(parent)  # parent set later
```

### 3. Event System

PyQt5 uses signals and slots for event handling:

```python
# Connect a signal to a slot
button.clicked.connect(self.on_button_clicked)

# Custom signal
from PyQt5.QtCore import pyqtSignal

class MyWidget(QWidget):
    # Define custom signal
    my_signal = pyqtSignal(str)
    
    def emit_signal(self):
        self.my_signal.emit("Hello World")
```

## Essential Widgets

### 1. Basic Widgets

#### QWidget
The base class for all user interface objects.

```python
widget = QWidget()
widget.setFixedSize(300, 200)
widget.setWindowTitle("My Widget")
widget.show()
```

#### QLabel
Displays text or images.

```python
label = QLabel("Hello World")
label.setAlignment(Qt.AlignCenter)
label.setStyleSheet("color: blue; font-size: 14px;")

# With image
label.setPixmap(QPixmap("image.png"))
```

#### QPushButton
Clickable button widget.

```python
button = QPushButton("Click Me")
button.setIcon(QIcon("icon.png"))
button.setToolTip("This is a button")
button.clicked.connect(on_button_clicked)
```

#### QLineEdit
Single-line text input.

```python
line_edit = QLineEdit()
line_edit.setPlaceholderText("Enter text here")
line_edit.textChanged.connect(on_text_changed)
line_edit.returnPressed.connect(on_return_pressed)
```

#### QTextEdit
Multi-line text input.

```python
text_edit = QTextEdit()
text_edit.setPlainText("Default text")
text_edit.textChanged.connect(on_text_changed)
```

#### QCheckBox
Checkbox for boolean selection.

```python
checkbox = QCheckBox("Enable feature")
checkbox.setChecked(True)
checkbox.stateChanged.connect(on_state_changed)
```

#### QRadioButton
Radio button for exclusive selection.

```python
radio1 = QRadioButton("Option 1")
radio2 = QRadioButton("Option 2")
radio1.setChecked(True)
radio1.toggled.connect(on_radio_toggled)
```

### 2. Container Widgets

#### QFrame
A simple frame that can contain other widgets.

```python
frame = QFrame()
frame.setFrameStyle(QFrame.Box)
frame.setLineWidth(2)
frame.setStyleSheet("background-color: lightgray;")
```

#### QGroupBox
A group box with a title.

```python
group_box = QGroupBox("Settings")
layout = QVBoxLayout()
layout.addWidget(QCheckBox("Option 1"))
layout.addWidget(QCheckBox("Option 2"))
group_box.setLayout(layout)
```

#### QScrollArea
A scrollable area for large content.

```python
scroll = QScrollArea()
scroll.setWidgetResizable(True)
scroll.setWidget(large_widget)
```

### 3. Input Widgets

#### QComboBox
Dropdown selection widget.

```python
combo = QComboBox()
combo.addItems(["Option 1", "Option 2", "Option 3"])
combo.setCurrentText("Option 2")
combo.currentTextChanged.connect(on_selection_changed)
```

#### QSpinBox
Integer input with up/down buttons.

```python
spinbox = QSpinBox()
spinbox.setRange(0, 100)
spinbox.setValue(50)
spinbox.setSuffix(" %")
spinbox.valueChanged.connect(on_value_changed)
```

#### QDoubleSpinBox
Floating-point number input.

```python
double_spinbox = QDoubleSpinBox()
double_spinbox.setRange(0.0, 100.0)
double_spinbox.setDecimals(2)
double_spinbox.setSingleStep(0.1)
```

#### QSlider
Slider for selecting values within a range.

```python
slider = QSlider(Qt.Horizontal)
slider.setRange(0, 100)
slider.setValue(50)
slider.setTickPosition(QSlider.TicksBelow)
slider.valueChanged.connect(on_slider_changed)
```

### 4. Display Widgets

#### QTableWidget
Table for displaying tabular data.

```python
table = QTableWidget(5, 3)  # 5 rows, 3 columns
table.setHorizontalHeaderLabels(["Name", "Age", "City"])
table.setItem(0, 0, QTableWidgetItem("John"))
table.setItem(0, 1, QTableWidgetItem("25"))
table.setItem(0, 2, QTableWidgetItem("New York"))
table.setSortingEnabled(True)
```

#### QListWidget
List display widget.

```python
list_widget = QListWidget()
list_widget.addItems(["Item 1", "Item 2", "Item 3"])
list_widget.setCurrentRow(0)
list_widget.itemClicked.connect(on_item_clicked)
```

#### QTreeWidget
Hierarchical tree display.

```python
tree = QTreeWidget()
tree.setHeaderLabels(["Name", "Type"])

# Add root item
root = QTreeWidgetItem(tree, ["Root", "Folder"])

# Add child items
child1 = QTreeWidgetItem(root, ["Child 1", "File"])
child2 = QTreeWidgetItem(root, ["Child 2", "File"])

tree.expandAll()
```

#### QProgressBar
Progress indicator.

```python
progress = QProgressBar()
progress.setRange(0, 100)
progress.setValue(50)
progress.setTextVisible(True)
progress.setFormat("Processing: %p%")
```

### 5. Container and Layout Widgets

#### QTabWidget
Tabbed interface.

```python
tab_widget = QTabWidget()
tab_widget.addTab(QWidget(), "Tab 1")
tab_widget.addTab(QWidget(), "Tab 2")
tab_widget.setTabPosition(QTabWidget.North)
tab_widget.currentChanged.connect(on_tab_changed)
```

#### QSplitter
Resizable split panes.

```python
splitter = QSplitter(Qt.Horizontal)
splitter.addWidget(QListWidget())
splitter.addWidget(QTextEdit())
splitter.setSizes([200, 400])
```

#### QStackedWidget
Stack of widgets with one visible at a time.

```python
stack = QStackedWidget()
stack.addWidget(QLabel("Page 1"))
stack.addWidget(QLabel("Page 2"))
stack.setCurrentIndex(0)
```

## Layout Management

### 1. QVBoxLayout
Vertical layout arranges widgets vertically.

```python
layout = QVBoxLayout()
layout.addWidget(QLabel("Top"))
layout.addWidget(QLabel("Middle"))
layout.addWidget(QLabel("Bottom"))
layout.addStretch()  # Add flexible space

widget = QWidget()
widget.setLayout(layout)
```

### 2. QHBoxLayout
Horizontal layout arranges widgets horizontally.

```python
layout = QHBoxLayout()
layout.addWidget(QLabel("Left"))
layout.addWidget(QLabel("Center"))
layout.addWidget(QLabel("Right"))
layout.addStretch()  # Add flexible space

widget = QWidget()
widget.setLayout(layout)
```

### 3. QGridLayout
Grid layout arranges widgets in a grid.

```python
layout = QGridLayout()
layout.addWidget(QLabel("Top Left"), 0, 0)
layout.addWidget(QLabel("Top Right"), 0, 1)
layout.addWidget(QLabel("Bottom Left"), 1, 0)
layout.addWidget(QLabel("Bottom Right"), 1, 1)

# Span multiple cells
layout.addWidget(QLabel("Spans 2 columns"), 2, 0, 1, 2)

widget = QWidget()
widget.setLayout(layout)
```

### 4. QFormLayout
Form layout for label-field pairs.

```python
layout = QFormLayout()
layout.addRow("Name:", QLineEdit())
layout.addRow("Age:", QSpinBox())
layout.addRow("Email:", QLineEdit())

widget = QWidget()
widget.setLayout(layout)
```

### 5. Layout Properties

```python
layout = QVBoxLayout()

# Spacing and margins
layout.setSpacing(10)
layout.setContentsMargins(20, 20, 20, 20)  # left, top, right, bottom

# Alignment
layout.setAlignment(Qt.AlignCenter)

# Stretch factors
layout.addWidget(widget1, 1)  # stretch factor 1
layout.addWidget(widget2, 2)  # stretch factor 2
```

## Dialogs and Windows

### 1. QMainWindow
Main application window with menu bar, toolbar, and status bar.

```python
class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("My Application")
        self.setGeometry(100, 100, 800, 600)
        
        # Central widget
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        # Menu bar
        menubar = self.menuBar()
        file_menu = menubar.addMenu("File")
        file_menu.addAction("Open", self.open_file)
        file_menu.addAction("Save", self.save_file)
        
        # Toolbar
        toolbar = self.addToolBar("Main")
        toolbar.addAction("New", self.new_file)
        toolbar.addAction("Open", self.open_file)
        
        # Status bar
        self.statusBar().showMessage("Ready")
```

### 2. QDialog
Modal dialog window.

```python
class CustomDialog(QDialog):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setWindowTitle("Custom Dialog")
        self.setModal(True)
        
        layout = QVBoxLayout()
        layout.addWidget(QLabel("Enter your name:"))
        self.line_edit = QLineEdit()
        layout.addWidget(self.line_edit)
        
        # Button box
        button_box = QDialogButtonBox(
            QDialogButtonBox.Ok | QDialogButtonBox.Cancel
        )
        button_box.accepted.connect(self.accept)
        button_box.rejected.connect(self.reject)
        layout.addWidget(button_box)
        
        self.setLayout(layout)
    
    def get_name(self):
        return self.line_edit.text()

# Usage
dialog = CustomDialog()
if dialog.exec_() == QDialog.Accepted:
    name = dialog.get_name()
    print(f"Name: {name}")
```

### 3. Standard Dialogs

```python
from PyQt5.QtWidgets import QMessageBox, QFileDialog, QColorDialog, QFontDialog

# Message boxes
QMessageBox.information(self, "Title", "Information message")
QMessageBox.warning(self, "Title", "Warning message")
QMessageBox.critical(self, "Title", "Error message")

result = QMessageBox.question(
    self, "Title", "Are you sure?",
    QMessageBox.Yes | QMessageBox.No
)

# File dialogs
file_path, _ = QFileDialog.getOpenFileName(
    self, "Open File", "", "Text Files (*.txt);;All Files (*)"
)

file_path, _ = QFileDialog.getSaveFileName(
    self, "Save File", "", "Text Files (*.txt);;All Files (*)"
)

# Color dialog
color = QColorDialog.getColor(Qt.white, self)
if color.isValid():
    print(f"Selected color: {color.name()}")

# Font dialog
font, ok = QFontDialog.getFont(QFont("Arial", 12), self)
if ok:
    print(f"Selected font: {font.family()}, {font.pointSize()}")
```

## Event Handling

### 1. Signals and Slots

```python
# Built-in signals
button.clicked.connect(self.on_button_clicked)
line_edit.textChanged.connect(self.on_text_changed)
combo.currentIndexChanged.connect(self.on_selection_changed)

# Custom signals
from PyQt5.QtCore import pyqtSignal

class MyWidget(QWidget):
    # Define custom signal
    data_changed = pyqtSignal(str, int)
    
    def emit_data_changed(self):
        self.data_changed.emit("Hello", 42)
    
    def on_data_changed(self, text, number):
        print(f"Data changed: {text}, {number}")

# Connect custom signal
widget = MyWidget()
widget.data_changed.connect(widget.on_data_changed)
```

### 2. Event Handlers

```python
class MyWidget(QWidget):
    def mousePressEvent(self, event):
        if event.button() == Qt.LeftButton:
            print("Left mouse button pressed")
        super().mousePressEvent(event)
    
    def keyPressEvent(self, event):
        if event.key() == Qt.Key_Escape:
            self.close()
        super().keyPressEvent(event)
    
    def paintEvent(self, event):
        painter = QPainter(self)
        painter.drawText(10, 30, "Custom painting")
        super().paintEvent(event)
    
    def closeEvent(self, event):
        reply = QMessageBox.question(
            self, "Exit", "Are you sure you want to quit?",
            QMessageBox.Yes | QMessageBox.No
        )
        if reply == QMessageBox.Yes:
            event.accept()
        else:
            event.ignore()
```

## Styling and Themes

### 1. StyleSheets (CSS-like)

```python
# Basic styling
widget.setStyleSheet("background-color: blue; color: white;")

# Complex styling
stylesheet = """
    QWidget {
        background-color: #f0f0f0;
        font-family: Arial;
        font-size: 12px;
    }
    
    QPushButton {
        background-color: #007bff;
        color: white;
        border: none;
        padding: 10px;
        border-radius: 5px;
    }
    
    QPushButton:hover {
        background-color: #0056b3;
    }
    
    QPushButton:pressed {
        background-color: #004085;
    }
    
    QLineEdit {
        border: 1px solid #ccc;
        border-radius: 3px;
        padding: 5px;
    }
    
    QLineEdit:focus {
        border: 2px solid #007bff;
    }
"""

app.setStyleSheet(stylesheet)
```

### 2. Properties and States

```python
# Property selectors
stylesheet = """
    QPushButton[type="primary"] {
        background-color: #007bff;
        color: white;
    }
    
    QPushButton[type="danger"] {
        background-color: #dc3545;
        color: white;
    }
"""

# Set property
button.setProperty("type", "primary")
button.setStyleSheet(stylesheet)
```

### 3. Custom Fonts and Colors

```python
from PyQt5.QtGui import QFont, QColor, QPalette

# Custom font
font = QFont("Arial", 12, QFont.Bold)
widget.setFont(font)

# Custom colors
palette = QPalette()
palette.setColor(QPalette.Window, QColor(240, 240, 240))
palette.setColor(QPalette.WindowText, QColor(0, 0, 0))
widget.setPalette(palette)
```

## Advanced Features

### 1. Model-View Architecture

```python
from PyQt5.QtCore import QAbstractTableModel, Qt

class TableModel(QAbstractTableModel):
    def __init__(self, data):
        super().__init__()
        self._data = data
    
    def rowCount(self, parent=QModelIndex()):
        return len(self._data)
    
    def columnCount(self, parent=QModelIndex()):
        return len(self._data[0]) if self._data else 0
    
    def data(self, index, role=Qt.DisplayRole):
        if role == Qt.DisplayRole:
            return str(self._data[index.row()][index.column()])
        return None
    
    def headerData(self, section, orientation, role=Qt.DisplayRole):
        if role == Qt.DisplayRole and orientation == Qt.Horizontal:
            return f"Column {section + 1}"
        return None

# Usage
data = [["A", "B", "C"], ["1", "2", "3"], ["X", "Y", "Z"]]
model = TableModel(data)
table_view = QTableView()
table_view.setModel(model)
```

### 2. Threading

```python
from PyQt5.QtCore import QThread, pyqtSignal

class WorkerThread(QThread):
    progress_updated = pyqtSignal(int)
    work_finished = pyqtSignal(str)
    
    def run(self):
        for i in range(100):
            # Simulate work
            self.msleep(50)
            self.progress_updated.emit(i + 1)
        
        self.work_finished.emit("Work completed!")

# Usage
worker = WorkerThread()
worker.progress_updated.connect(progress_bar.setValue)
worker.work_finished.connect(lambda msg: print(msg))
worker.start()
```

### 3. Drag and Drop

```python
class DragDropWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.setAcceptDrops(True)
    
    def dragEnterEvent(self, event):
        if event.mimeData().hasText():
            event.acceptProposedAction()
    
    def dropEvent(self, event):
        text = event.mimeData().text()
        print(f"Dropped text: {text}")
        event.acceptProposedAction()
    
    def mousePressEvent(self, event):
        if event.button() == Qt.LeftButton:
            drag = QDrag(self)
            mime_data = QMimeData()
            mime_data.setText("Dragged text")
            drag.setMimeData(mime_data)
            drag.exec_(Qt.CopyAction)
```

## Best Practices

### 1. Memory Management

```python
# Proper parent-child relationships
parent = QWidget()
child = QLabel("Child", parent)  # Child will be deleted with parent

# Manual cleanup when needed
def cleanup(self):
    self.timer.stop()
    self.thread.quit()
    self.thread.wait()
```

### 2. Signal-Slot Connections

```python
# Good: Use lambda for simple parameter passing
button.clicked.connect(lambda: self.process_data(some_parameter))

# Better: Use partial for more complex scenarios
from functools import partial
button.clicked.connect(partial(self.process_data, some_parameter))

# Best: Use dedicated methods
def on_button_clicked(self):
    self.process_data(self.some_parameter)

button.clicked.connect(self.on_button_clicked)
```

### 3. Error Handling

```python
def load_file(self, filename):
    try:
        with open(filename, 'r') as f:
            content = f.read()
        self.text_edit.setText(content)
    except FileNotFoundError:
        QMessageBox.warning(self, "Error", f"File not found: {filename}")
    except Exception as e:
        QMessageBox.critical(self, "Error", f"Failed to load file: {str(e)}")
```

### 4. Performance Optimization

```python
# Batch updates for large datasets
table.setUpdatesEnabled(False)
try:
    for row in range(1000):
        # Add items to table
        pass
finally:
    table.setUpdatesEnabled(True)

# Use appropriate data structures
# For large datasets, consider QAbstractItemModel instead of QTableWidget
```

## Common Patterns

### 1. Singleton Application

```python
class SingletonApp(QApplication):
    def __init__(self, argv):
        super().__init__(argv)
        self.setAttribute(Qt.AA_DisableWindowContextHelpButton)
        self.setQuitOnLastWindowClosed(True)
```

### 2. Settings Management

```python
from PyQt5.QtCore import QSettings

class SettingsManager:
    def __init__(self):
        self.settings = QSettings("MyCompany", "MyApp")
    
    def save_window_geometry(self, window):
        self.settings.setValue("geometry", window.saveGeometry())
    
    def restore_window_geometry(self, window):
        geometry = self.settings.value("geometry")
        if geometry:
            window.restoreGeometry(geometry)
```

### 3. Resource Management

```python
# Using Qt resource system
from PyQt5 import QtCore

# Load from resource file
icon = QIcon(":/icons/app_icon.png")
pixmap = QPixmap(":/images/logo.png")

# Loading from file system
icon_path = os.path.join(os.path.dirname(__file__), "icons", "app_icon.png")
icon = QIcon(icon_path)
```

## Debugging and Testing

### 1. Debug Output

```python
import logging

# Setup logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Use in code
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

### 2. Unit Testing

```python
import unittest
from PyQt5.QtTest import QTest
from PyQt5.QtCore import Qt

class TestMyWidget(unittest.TestCase):
    def setUp(self):
        self.app = QApplication(sys.argv)
        self.widget = MyWidget()
    
    def test_button_click(self):
        QTest.mouseClick(self.widget.button, Qt.LeftButton)
        self.assertEqual(self.widget.counter, 1)
    
    def test_text_input(self):
        QTest.keyClicks(self.widget.line_edit, "Hello")
        self.assertEqual(self.widget.line_edit.text(), "Hello")
```

## Summary

This reference guide covers the essential aspects of PyQt5 development:

- **Core Concepts**: Application structure, widget hierarchy, event system
- **Essential Widgets**: Comprehensive list of commonly used widgets
- **Layout Management**: Different layout types and their usage
- **Dialogs and Windows**: Main windows and dialog creation
- **Event Handling**: Signals, slots, and event handlers
- **Styling**: CSS-like styling and theming
- **Advanced Features**: Model-view architecture, threading, drag-and-drop
- **Best Practices**: Memory management, error handling, performance optimization
- **Common Patterns**: Frequently used design patterns
- **Debugging**: Testing and debugging techniques

This reference provides a solid foundation for building professional PyQt5 applications and serves as a quick lookup guide for common development tasks.
