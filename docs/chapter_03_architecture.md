# Chapter 3: Application Architecture and Design

## Overview

In this chapter, we'll explore the architecture and design principles behind our "Randomly Yours App." Understanding the application structure before diving into code is crucial for creating maintainable, scalable, and user-friendly software.

## Application Architecture

### High-Level Architecture

Our application follows the **Model-View-Controller (MVC)** pattern, adapted for PyQt5:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│      Model      │    │      View       │    │   Controller    │
│  (Data Logic)   │◄──►│  (User Interface) │◄──►│ (Business Logic) │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • CSV Data      │    │ • Tabs          │    │ • Event Handlers │
│ • DataFrames    │    │ • Tables        │    │ • Sampling Logic │
│ • Sampling      │    │ • Buttons       │    │ • Validation     │
│   Results       │    │ • Forms         │    │ • Navigation     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Components Breakdown

#### 1. Model Layer
- **Purpose**: Manages data and business logic
- **Responsibilities**:
  - Loading and parsing CSV files
  - Storing DataFrames
  - Performing sampling operations
  - Validating data integrity

#### 2. View Layer
- **Purpose**: Handles user interface presentation
- **Responsibilities**:
  - Rendering tabs and widgets
  - Displaying data in tables
  - Providing input controls
  - Showing feedback messages

#### 3. Controller Layer
- **Purpose**: Coordinates between Model and View
- **Responsibilities**:
  - Handling user interactions
  - Updating the model based on user input
  - Refreshing the view when data changes
  - Managing navigation flow

## Design Principles

### 1. Single Responsibility Principle
Each class and method has a single, well-defined purpose:

```python
class SamplingApp(QWidget):
    """Main application window - handles UI coordination"""
    
class DataProcessor:
    """Handles data loading and processing"""
    
class SamplingEngine:
    """Performs sampling operations"""
```

### 2. Separation of Concerns
Different aspects of the application are separated:

- **UI Creation**: Separate methods for each tab
- **Data Processing**: Isolated in dedicated modules
- **Styling**: Centralized stylesheet management
- **Event Handling**: Dedicated methods for each interaction

### 3. Modularity
The application is broken into logical modules:

```
app/
├── main.py           # Main application class
├── sampling.py       # Sampling algorithms
├── data_handler.py   # Data loading and validation
├── ui_components.py  # Reusable UI components
└── styles.py         # Styling definitions
```

## User Experience Design

### Workflow Design

Our application follows a **wizard-like workflow** that guides users through the sampling process:

```
File Selection → Data Preview → Method Selection → Size Configuration → Settings → Results
     ↓               ↓              ↓                 ↓                ↓         ↓
   📁 File        📊 Data        🔄 Method        📏 Size        ⚙️ Settings   📊 Results
```

### Tab-Based Navigation

#### Benefits of Tab Interface
1. **Clear Progress**: Users see their position in the workflow
2. **Non-Linear Access**: Can jump between completed steps
3. **Context Preservation**: Each tab maintains its state
4. **Visual Organization**: Related functions grouped together

#### Tab Design Principles
- **Progressive Disclosure**: Information revealed as needed
- **Logical Grouping**: Related functions in same tab
- **Visual Hierarchy**: Important actions emphasized
- **Consistent Navigation**: Uniform back/next buttons

### Information Architecture

```
Application Root
├── File Tab
│   ├── Welcome Section
│   ├── File Selection
│   └── Quick Preview
├── Data Tab
│   ├── Data Information
│   ├── Full Data Table
│   └── Sorting Controls
├── Method Tab
│   ├── With Replacement
│   └── Without Replacement
├── Size Tab
│   ├── By Percentage
│   └── By Count
├── Settings Tab
│   ├── Random Seed
│   └── Reproducibility Options
└── Results Tab
    ├── Results Table
    ├── Export Options
    └── Summary Information
```

## Class Structure

### Main Application Class

```python
class SamplingApp(QWidget):
    """
    Main application window that orchestrates the entire user interface.
    
    Responsibilities:
    - Create and manage tab widget
    - Handle navigation between tabs
    - Coordinate data flow between components
    - Manage application state
    """
    
    def __init__(self):
        """Initialize the application"""
        super().__init__()
        self.df = None              # Holds the loaded DataFrame
        self.current_results = None # Holds sampling results
        self.init_ui()              # Create user interface
    
    def init_ui(self):
        """Initialize the user interface components"""
        # Create main layout
        # Create tab widget
        # Create individual tabs
        # Apply styling
        # Set up window properties
```

### Tab Creation Methods

Each tab is created by a dedicated method:

```python
def create_file_tab(self):
    """Create the file selection tab"""
    
def create_data_preview_tab(self):
    """Create the data preview tab"""
    
def create_method_tab(self):
    """Create the sampling method selection tab"""
    
def create_size_tab(self):
    """Create the sample size configuration tab"""
    
def create_advanced_tab(self):
    """Create the advanced settings tab"""
    
def create_results_tab(self):
    """Create the results display tab"""
```

### Utility Methods

Supporting methods that handle specific functionality:

```python
def show_dataframe(self, df, preview=True, data_preview=False):
    """Display DataFrame in appropriate table widget"""
    
def toggle_input_mode(self):
    """Toggle between percentage and count input modes"""
    
def update_slider_label(self, value):
    """Update slider label with current value"""
    
def perform_sampling(self):
    """Execute the sampling operation"""
    
def export_results(self):
    """Export sampling results to CSV"""
```

## Data Flow

### Data Loading Process

```
1. User clicks "Choose CSV File"
   ↓
2. QFileDialog opens
   ↓
3. User selects file
   ↓
4. pandas.read_csv() loads data
   ↓
5. Data validation checks
   ↓
6. Update preview tables
   ↓
7. Enable navigation to next tab
```

### Sampling Process

```
1. User configures sampling parameters
   ↓
2. Validation of input parameters
   ↓
3. Call sampling.perform_sampling()
   ↓
4. Generate sample DataFrame
   ↓
5. Update results table
   ↓
6. Store results for export
   ↓
7. Navigate to results tab
```

## Error Handling Strategy

### Defensive Programming

```python
def load_csv(self):
    """Load CSV file with comprehensive error handling"""
    try:
        file_name, _ = QFileDialog.getOpenFileName(
            self, "Open CSV", "", "CSV Files (*.csv)"
        )
        
        if not file_name:
            return  # User cancelled
        
        # Validate file exists
        if not os.path.exists(file_name):
            raise FileNotFoundError(f"File not found: {file_name}")
        
        # Load data
        self.df = pd.read_csv(file_name)
        
        # Validate data
        if self.df.empty:
            raise ValueError("CSV file is empty")
        
        # Success - update UI
        self.show_dataframe(self.df, preview=True)
        
    except pd.errors.EmptyDataError:
        QMessageBox.warning(self, "Invalid File", "The CSV file is empty.")
    except pd.errors.ParserError:
        QMessageBox.warning(self, "Invalid File", "Could not parse CSV file.")
    except Exception as e:
        QMessageBox.critical(self, "Error", f"Could not load file: {str(e)}")
```

### Error Categories

1. **User Errors**: Invalid input, file not found
2. **Data Errors**: Malformed CSV, empty data
3. **System Errors**: Memory issues, disk space
4. **Logic Errors**: Invalid sampling parameters

## State Management

### Application State

The application maintains state through instance variables:

```python
class SamplingApp(QWidget):
    def __init__(self):
        super().__init__()
        
        # Data state
        self.df = None                    # Original DataFrame
        self.current_results = None       # Sampling results
        
        # UI state
        self.tab_widget = None            # Tab container
        self.preview_table = None         # File preview table
        self.data_preview_table = None    # Full data table
        self.results_table = None         # Results table
        
        # Configuration state
        self.replacement = None           # With/without replacement
        self.no_replacement = None        # Radio button references
        self.by_percent = None           # Percentage mode
        self.by_count = None             # Count mode
        self.slider = None               # Percentage slider
        self.sample_count = None         # Count input
        self.seed_input = None           # Random seed input
```

### State Transitions

```python
def enable_next_step(self):
    """Enable progression to next step after validation"""
    # Check if current step is complete
    # Enable/disable navigation buttons
    # Update visual indicators
    
def validate_current_step(self):
    """Validate current step before allowing progression"""
    # Check required inputs
    # Validate data ranges
    # Show error messages if needed
    # Return True/False for validation status
```

## Design Patterns Used

### 1. Template Method Pattern
Each tab follows a similar creation pattern:

```python
def create_tab(self):
    """Template for tab creation"""
    # 1. Create tab widget
    # 2. Create main layout
    # 3. Create header section
    # 4. Create content section
    # 5. Create navigation section
    # 6. Apply styling
    # 7. Add to tab widget
```

### 2. Observer Pattern
Using Qt's signal-slot mechanism:

```python
# Widget signals notify other components
self.slider.valueChanged.connect(self.update_slider_label)
self.button.clicked.connect(self.perform_action)
```

### 3. Command Pattern
Button actions encapsulate operations:

```python
def create_button(self, text, action):
    """Create button with associated action"""
    button = QPushButton(text)
    button.clicked.connect(action)
    return button
```

## Scalability Considerations

### 1. Modular Design
- Easy to add new sampling methods
- Simple to extend with new file formats
- Straightforward to add new export options

### 2. Configuration Management
```python
class AppConfig:
    """Centralized application configuration"""
    DEFAULT_SAMPLE_SIZE = 10
    MAX_PREVIEW_ROWS = 1000
    SUPPORTED_FORMATS = ['.csv', '.xlsx', '.json']
```

### 3. Plugin Architecture
Future extension points:
- Custom sampling algorithms
- Additional data sources
- Export formats
- Visualization options

## Performance Considerations

### 1. Lazy Loading
- Load data only when needed
- Preview limited number of rows
- Process data in chunks for large files

### 2. UI Responsiveness
```python
def long_operation(self):
    """Keep UI responsive during long operations"""
    progress = QProgressDialog("Processing...", "Cancel", 0, 100, self)
    progress.show()
    
    for i in range(100):
        # Process chunk
        progress.setValue(i)
        QApplication.processEvents()  # Keep UI responsive
```

### 3. Memory Management
- Clear large DataFrames when not needed
- Use generators for large datasets
- Implement data streaming for huge files

## Testing Strategy

### 1. Unit Testing
```python
class TestSamplingLogic(unittest.TestCase):
    def test_simple_random_sampling(self):
        # Test sampling algorithms
        pass
    
    def test_data_validation(self):
        # Test input validation
        pass
```

### 2. Integration Testing
```python
class TestApplicationIntegration(unittest.TestCase):
    def test_file_loading_workflow(self):
        # Test complete file loading process
        pass
    
    def test_sampling_workflow(self):
        # Test end-to-end sampling process
        pass
```

### 3. UI Testing
```python
class TestUserInterface(unittest.TestCase):
    def test_tab_navigation(self):
        # Test navigation between tabs
        pass
    
    def test_form_validation(self):
        # Test form input validation
        pass
```

## Chapter Summary

In this chapter, we explored:

1. **Application Architecture**: MVC pattern adaptation for PyQt5
2. **Design Principles**: Single responsibility, separation of concerns, modularity
3. **User Experience**: Wizard-like workflow and tab-based navigation
4. **Class Structure**: Main application class and supporting methods
5. **Data Flow**: Loading and processing workflows
6. **Error Handling**: Defensive programming and error categories
7. **State Management**: Application state and transitions
8. **Design Patterns**: Template method, observer, and command patterns
9. **Scalability**: Modular design and extension points
10. **Performance**: Lazy loading, UI responsiveness, and memory management
11. **Testing**: Unit, integration, and UI testing strategies

This architectural foundation ensures our application is maintainable, scalable, and user-friendly. In the next chapter, we'll implement the main window and tab system, bringing this architecture to life.

---

**Key Takeaways:**
- Good architecture is the foundation of maintainable software
- User experience should drive design decisions
- Error handling and state management are crucial
- Design patterns help create consistent, understandable code
- Testing strategy should be planned from the beginning
