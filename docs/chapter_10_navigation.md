# Chapter 10: Navigation and Workflow Management

## Introduction

Effective navigation and workflow management are crucial for creating intuitive user experiences. This chapter explores how to implement smooth transitions between different application states, manage user interactions across multiple tabs, and create a logical workflow that guides users through complex processes.

## Understanding Workflow Design

### The Wizard Pattern

Our random sampling application follows a wizard-like pattern with sequential steps:

1. **File Selection**: Load and validate data
2. **Data Preview**: Review and understand the dataset
3. **Method Selection**: Choose sampling method and parameters
4. **Size Configuration**: Set sample size and options
5. **Settings**: Configure advanced options
6. **Results**: View and export sample data

```python
from PyQt5.QtWidgets import (QTabWidget, QWidget, QPushButton, QHBoxLayout, 
                             QVBoxLayout, QLabel, QProgressBar, QStackedWidget)
from PyQt5.QtCore import Qt, pyqtSignal, QPropertyAnimation, QEasingCurve
from PyQt5.QtGui import QFont, QIcon
from enum import Enum

class WorkflowState(Enum):
    """Enumeration of workflow states."""
    INITIAL = 0
    FILE_LOADED = 1
    DATA_PREVIEWED = 2
    METHOD_SELECTED = 3
    SIZE_CONFIGURED = 4
    SETTINGS_CONFIGURED = 5
    SAMPLE_GENERATED = 6
    RESULTS_EXPORTED = 7

class NavigationManager:
    """Manages navigation state and workflow progression."""
    
    def __init__(self, parent):
        self.parent = parent
        self.current_state = WorkflowState.INITIAL
        self.tab_states = {}
        self.initialize_tab_states()
    
    def initialize_tab_states(self):
        """Initialize the enabled/disabled state of each tab."""
        self.tab_states = {
            0: True,   # File tab - always enabled
            1: False,  # Data Preview tab - enabled after file load
            2: False,  # Method tab - enabled after data preview
            3: False,  # Size tab - enabled after method selection
            4: False,  # Settings tab - enabled after size configuration
            5: False,  # Results tab - enabled after sample generation
        }
    
    def update_tab_states(self):
        """Update tab enabled/disabled states based on current workflow state."""
        for tab_index, enabled in self.tab_states.items():
            self.parent.tabs.setTabEnabled(tab_index, enabled)
    
    def can_navigate_to_tab(self, tab_index: int) -> bool:
        """Check if navigation to a specific tab is allowed."""
        return self.tab_states.get(tab_index, False)
    
    def enable_tab(self, tab_index: int):
        """Enable a specific tab."""
        self.tab_states[tab_index] = True
        self.update_tab_states()
    
    def disable_tab(self, tab_index: int):
        """Disable a specific tab."""
        self.tab_states[tab_index] = False
        self.update_tab_states()
    
    def set_workflow_state(self, state: WorkflowState):
        """Set the current workflow state and update UI accordingly."""
        self.current_state = state
        self.update_navigation_based_on_state()
    
    def update_navigation_based_on_state(self):
        """Update navigation controls based on current workflow state."""
        if self.current_state == WorkflowState.INITIAL:
            self.parent.update_navigation_buttons(prev_enabled=False, next_enabled=False)
        elif self.current_state == WorkflowState.FILE_LOADED:
            self.enable_tab(1)  # Data Preview
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=True)
        elif self.current_state == WorkflowState.DATA_PREVIEWED:
            self.enable_tab(2)  # Method
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=True)
        elif self.current_state == WorkflowState.METHOD_SELECTED:
            self.enable_tab(3)  # Size
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=True)
        elif self.current_state == WorkflowState.SIZE_CONFIGURED:
            self.enable_tab(4)  # Settings
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=True)
        elif self.current_state == WorkflowState.SETTINGS_CONFIGURED:
            self.enable_tab(5)  # Results
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=True)
        elif self.current_state == WorkflowState.SAMPLE_GENERATED:
            self.parent.update_navigation_buttons(prev_enabled=True, next_enabled=False)
```

## Implementing Navigation Controls

### Navigation Button Bar

```python
class NavigationBar(QWidget):
    """Custom navigation bar with Previous/Next buttons and progress indicator."""
    
    # Signals
    previous_clicked = pyqtSignal()
    next_clicked = pyqtSignal()
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_ui()
        self.current_step = 0
        self.total_steps = 6
    
    def setup_ui(self):
        """Setup the navigation bar UI."""
        layout = QHBoxLayout(self)
        layout.setContentsMargins(20, 10, 20, 10)
        
        # Progress indicator
        self.progress_widget = self.create_progress_widget()
        layout.addWidget(self.progress_widget)
        
        # Spacer
        layout.addStretch()
        
        # Navigation buttons
        self.prev_button = QPushButton("← Previous")
        self.next_button = QPushButton("Next →")
        
        # Style navigation buttons
        self.style_navigation_buttons()
        
        # Connect signals
        self.prev_button.clicked.connect(self.previous_clicked.emit)
        self.next_button.clicked.connect(self.next_clicked.emit)
        
        # Add buttons to layout
        layout.addWidget(self.prev_button)
        layout.addWidget(self.next_button)
        
        # Apply styling
        self.setStyleSheet("""
            NavigationBar {
                background-color: #f8f9fa;
                border-top: 1px solid #dee2e6;
            }
        """)
    
    def create_progress_widget(self):
        """Create a progress indicator widget."""
        progress_widget = QWidget()
        progress_layout = QVBoxLayout(progress_widget)
        progress_layout.setContentsMargins(0, 0, 0, 0)
        
        # Progress bar
        self.progress_bar = QProgressBar()
        self.progress_bar.setMaximum(self.total_steps)
        self.progress_bar.setValue(0)
        self.progress_bar.setTextVisible(False)
        self.progress_bar.setFixedHeight(8)
        
        # Progress label
        self.progress_label = QLabel("Step 1 of 6")
        self.progress_label.setAlignment(Qt.AlignCenter)
        self.progress_label.setFont(QFont("Segoe UI", 9))
        
        progress_layout.addWidget(self.progress_label)
        progress_layout.addWidget(self.progress_bar)
        
        # Style progress bar
        self.progress_bar.setStyleSheet("""
            QProgressBar {
                border: 1px solid #dee2e6;
                border-radius: 4px;
                background-color: #e9ecef;
            }
            QProgressBar::chunk {
                background-color: #007bff;
                border-radius: 3px;
            }
        """)
        
        return progress_widget
    
    def style_navigation_buttons(self):
        """Apply styling to navigation buttons."""
        button_style = """
            QPushButton {
                background-color: #007bff;
                color: white;
                border: none;
                padding: 10px 20px;
                border-radius: 5px;
                font-size: 14px;
                font-weight: bold;
                min-width: 100px;
            }
            QPushButton:hover {
                background-color: #0056b3;
            }
            QPushButton:pressed {
                background-color: #004085;
            }
            QPushButton:disabled {
                background-color: #6c757d;
                color: #adb5bd;
            }
        """
        
        self.prev_button.setStyleSheet(button_style)
        self.next_button.setStyleSheet(button_style)
    
    def update_progress(self, step: int):
        """Update progress indicator."""
        self.current_step = step
        self.progress_bar.setValue(step)
        self.progress_label.setText(f"Step {step + 1} of {self.total_steps}")
    
    def set_button_states(self, prev_enabled: bool, next_enabled: bool):
        """Set the enabled state of navigation buttons."""
        self.prev_button.setEnabled(prev_enabled)
        self.next_button.setEnabled(next_enabled)
    
    def set_next_button_text(self, text: str):
        """Set custom text for the next button."""
        self.next_button.setText(text)
    
    def set_previous_button_text(self, text: str):
        """Set custom text for the previous button."""
        self.prev_button.setText(text)
```

### Tab Navigation with Animations

```python
class AnimatedTabWidget(QTabWidget):
    """Enhanced tab widget with smooth animations."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_animations()
        self.navigation_manager = None
    
    def setup_animations(self):
        """Setup tab transition animations."""
        self.fade_animation = QPropertyAnimation(self, b"windowOpacity")
        self.fade_animation.setDuration(200)
        self.fade_animation.setEasingCurve(QEasingCurve.OutCubic)
    
    def set_navigation_manager(self, manager: NavigationManager):
        """Set the navigation manager."""
        self.navigation_manager = manager
    
    def navigate_to_tab(self, tab_index: int, animate: bool = True):
        """Navigate to a specific tab with optional animation."""
        if self.navigation_manager and not self.navigation_manager.can_navigate_to_tab(tab_index):
            return False
        
        if animate:
            self.animate_tab_transition(tab_index)
        else:
            self.setCurrentIndex(tab_index)
        
        return True
    
    def animate_tab_transition(self, tab_index: int):
        """Animate transition to a new tab."""
        # Fade out current tab
        self.fade_animation.setStartValue(1.0)
        self.fade_animation.setEndValue(0.7)
        self.fade_animation.finished.connect(
            lambda: self.complete_tab_transition(tab_index)
        )
        self.fade_animation.start()
    
    def complete_tab_transition(self, tab_index: int):
        """Complete the tab transition animation."""
        # Change tab
        self.setCurrentIndex(tab_index)
        
        # Fade in new tab
        self.fade_animation.finished.disconnect()
        self.fade_animation.setStartValue(0.7)
        self.fade_animation.setEndValue(1.0)
        self.fade_animation.start()
    
    def tabBarClicked(self, index):
        """Handle tab bar clicks with validation."""
        if self.navigation_manager:
            if self.navigation_manager.can_navigate_to_tab(index):
                super().setCurrentIndex(index)
            else:
                # Show tooltip or message about why navigation is disabled
                self.show_navigation_restriction_message(index)
        else:
            super().setCurrentIndex(index)
    
    def show_navigation_restriction_message(self, tab_index: int):
        """Show message about navigation restrictions."""
        tab_names = [
            "File", "Data Preview", "Method", "Size", "Settings", "Results"
        ]
        
        if tab_index < len(tab_names):
            message = f"Please complete the previous steps before accessing {tab_names[tab_index]}"
            # You could show a tooltip or status message here
            print(message)  # For now, just print
```

## Workflow State Management

### State Validation

```python
class WorkflowValidator:
    """Validates workflow state transitions and requirements."""
    
    def __init__(self, app):
        self.app = app
    
    def validate_file_tab(self) -> tuple[bool, str]:
        """Validate file tab completion."""
        if not hasattr(self.app, 'current_data') or self.app.current_data is None:
            return False, "Please load a CSV file to continue."
        
        if len(self.app.current_data) == 0:
            return False, "The loaded file is empty. Please load a valid CSV file."
        
        return True, "File loaded successfully."
    
    def validate_data_preview_tab(self) -> tuple[bool, str]:
        """Validate data preview tab completion."""
        # Data preview is mostly informational, so it's automatically valid
        # if we have data loaded
        if not hasattr(self.app, 'current_data') or self.app.current_data is None:
            return False, "No data available to preview."
        
        return True, "Data preview completed."
    
    def validate_method_tab(self) -> tuple[bool, str]:
        """Validate method selection tab."""
        if not hasattr(self.app, 'method_combo') or not self.app.method_combo.currentText():
            return False, "Please select a sampling method."
        
        method = self.app.method_combo.currentText()
        
        if method == "Stratified":
            if not hasattr(self.app, 'strata_combo') or not self.app.strata_combo.currentText():
                return False, "Please select a stratification column for stratified sampling."
        
        return True, f"Sampling method '{method}' selected."
    
    def validate_size_tab(self) -> tuple[bool, str]:
        """Validate size configuration tab."""
        if not hasattr(self.app, 'size_spinbox'):
            return False, "Sample size configuration is missing."
        
        sample_size = self.app.size_spinbox.value()
        
        if sample_size <= 0:
            return False, "Sample size must be greater than 0."
        
        if hasattr(self.app, 'current_data') and self.app.current_data is not None:
            max_size = len(self.app.current_data)
            
            # Check if sampling without replacement
            if (hasattr(self.app, 'replacement_checkbox') and 
                not self.app.replacement_checkbox.isChecked() and 
                sample_size > max_size):
                return False, f"Sample size ({sample_size}) cannot exceed dataset size ({max_size}) when sampling without replacement."
        
        return True, f"Sample size set to {sample_size}."
    
    def validate_settings_tab(self) -> tuple[bool, str]:
        """Validate settings tab."""
        # Settings are optional, so this is usually valid
        return True, "Settings configured."
    
    def validate_all_requirements(self, target_tab: int) -> tuple[bool, str]:
        """Validate all requirements up to the target tab."""
        validators = [
            self.validate_file_tab,
            self.validate_data_preview_tab,
            self.validate_method_tab,
            self.validate_size_tab,
            self.validate_settings_tab
        ]
        
        for i in range(min(target_tab, len(validators))):
            is_valid, message = validators[i]()
            if not is_valid:
                return False, f"Step {i + 1}: {message}"
        
        return True, "All requirements met."
```

### Workflow Actions

```python
class WorkflowActions:
    """Handles workflow-specific actions and transitions."""
    
    def __init__(self, app):
        self.app = app
        self.validator = WorkflowValidator(app)
    
    def handle_next_button_click(self):
        """Handle next button click with validation."""
        current_tab = self.app.tabs.currentIndex()
        
        # Validate current tab
        if not self.validate_current_tab(current_tab):
            return
        
        # Determine next tab
        next_tab = current_tab + 1
        
        if next_tab >= self.app.tabs.count():
            # We're at the last tab
            self.handle_finish_workflow()
        else:
            # Navigate to next tab
            self.app.tabs.navigate_to_tab(next_tab)
            self.app.navigation_bar.update_progress(next_tab)
            self.update_navigation_button_states(next_tab)
    
    def handle_previous_button_click(self):
        """Handle previous button click."""
        current_tab = self.app.tabs.currentIndex()
        
        if current_tab > 0:
            previous_tab = current_tab - 1
            self.app.tabs.navigate_to_tab(previous_tab)
            self.app.navigation_bar.update_progress(previous_tab)
            self.update_navigation_button_states(previous_tab)
    
    def validate_current_tab(self, tab_index: int) -> bool:
        """Validate the current tab before allowing navigation."""
        validators = {
            0: self.validator.validate_file_tab,
            1: self.validator.validate_data_preview_tab,
            2: self.validator.validate_method_tab,
            3: self.validator.validate_size_tab,
            4: self.validator.validate_settings_tab
        }
        
        if tab_index in validators:
            is_valid, message = validators[tab_index]()
            if not is_valid:
                from PyQt5.QtWidgets import QMessageBox
                QMessageBox.warning(self.app, "Validation Error", message)
                return False
        
        return True
    
    def update_navigation_button_states(self, current_tab: int):
        """Update navigation button states based on current tab."""
        total_tabs = self.app.tabs.count()
        
        # Previous button
        prev_enabled = current_tab > 0
        
        # Next button
        next_enabled = current_tab < total_tabs - 1
        
        # Update button text based on position
        if current_tab == total_tabs - 1:
            self.app.navigation_bar.set_next_button_text("Generate Sample")
        else:
            self.app.navigation_bar.set_next_button_text("Next →")
        
        self.app.navigation_bar.set_button_states(prev_enabled, next_enabled)
    
    def handle_finish_workflow(self):
        """Handle workflow completion."""
        # Generate sample
        self.app.perform_sampling()
        
        # Update UI state
        self.app.navigation_manager.set_workflow_state(WorkflowState.SAMPLE_GENERATED)
        
        # Show completion message
        from PyQt5.QtWidgets import QMessageBox
        QMessageBox.information(
            self.app, "Sample Generated", 
            "Your random sample has been generated successfully!"
        )
```

## Advanced Navigation Features

### Breadcrumb Navigation

```python
class BreadcrumbWidget(QWidget):
    """Breadcrumb navigation widget showing current position in workflow."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_ui()
        self.steps = [
            "Load File", "Preview Data", "Select Method", 
            "Set Size", "Configure Settings", "View Results"
        ]
        self.current_step = 0
    
    def setup_ui(self):
        """Setup breadcrumb UI."""
        self.layout = QHBoxLayout(self)
        self.layout.setContentsMargins(10, 5, 10, 5)
        self.layout.setSpacing(5)
        
        # Apply styling
        self.setStyleSheet("""
            BreadcrumbWidget {
                background-color: #f8f9fa;
                border-bottom: 1px solid #dee2e6;
            }
        """)
    
    def update_breadcrumb(self, current_step: int):
        """Update breadcrumb display."""
        self.current_step = current_step
        self.rebuild_breadcrumb()
    
    def rebuild_breadcrumb(self):
        """Rebuild the breadcrumb display."""
        # Clear existing widgets
        for i in reversed(range(self.layout.count())):
            self.layout.itemAt(i).widget().setParent(None)
        
        # Add breadcrumb items
        for i, step in enumerate(self.steps):
            # Add step label
            label = QLabel(step)
            
            if i <= self.current_step:
                # Completed or current step
                if i == self.current_step:
                    label.setStyleSheet("""
                        QLabel {
                            color: #007bff;
                            font-weight: bold;
                            padding: 5px;
                        }
                    """)
                else:
                    label.setStyleSheet("""
                        QLabel {
                            color: #28a745;
                            padding: 5px;
                        }
                    """)
            else:
                # Future step
                label.setStyleSheet("""
                    QLabel {
                        color: #6c757d;
                        padding: 5px;
                    }
                """)
            
            self.layout.addWidget(label)
            
            # Add separator (except for last item)
            if i < len(self.steps) - 1:
                separator = QLabel("→")
                separator.setAlignment(Qt.AlignCenter)
                separator.setStyleSheet("""
                    QLabel {
                        color: #6c757d;
                        padding: 0 5px;
                    }
                """)
                self.layout.addWidget(separator)
        
        # Add stretch to push everything to the left
        self.layout.addStretch()
```

### Quick Navigation Menu

```python
class QuickNavigationMenu(QWidget):
    """Quick navigation menu for jumping between tabs."""
    
    tab_requested = pyqtSignal(int)
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_ui()
        self.navigation_manager = None
    
    def setup_ui(self):
        """Setup quick navigation UI."""
        layout = QVBoxLayout(self)
        layout.setContentsMargins(10, 10, 10, 10)
        
        # Title
        title = QLabel("Quick Navigation")
        title.setFont(QFont("Segoe UI", 12, QFont.Bold))
        layout.addWidget(title)
        
        # Navigation buttons
        self.nav_buttons = []
        tab_names = ["File", "Data Preview", "Method", "Size", "Settings", "Results"]
        
        for i, name in enumerate(tab_names):
            button = QPushButton(f"{i + 1}. {name}")
            button.setStyleSheet("""
                QPushButton {
                    text-align: left;
                    padding: 10px;
                    border: 1px solid #dee2e6;
                    background-color: white;
                    margin: 2px;
                }
                QPushButton:hover {
                    background-color: #f8f9fa;
                }
                QPushButton:disabled {
                    background-color: #f8f9fa;
                    color: #6c757d;
                }
            """)
            
            button.clicked.connect(lambda checked, idx=i: self.tab_requested.emit(idx))
            self.nav_buttons.append(button)
            layout.addWidget(button)
        
        layout.addStretch()
    
    def set_navigation_manager(self, manager: NavigationManager):
        """Set the navigation manager."""
        self.navigation_manager = manager
        self.update_button_states()
    
    def update_button_states(self):
        """Update button enabled/disabled states."""
        if self.navigation_manager:
            for i, button in enumerate(self.nav_buttons):
                button.setEnabled(self.navigation_manager.can_navigate_to_tab(i))
    
    def highlight_current_tab(self, tab_index: int):
        """Highlight the current tab button."""
        for i, button in enumerate(self.nav_buttons):
            if i == tab_index:
                button.setStyleSheet("""
                    QPushButton {
                        text-align: left;
                        padding: 10px;
                        border: 2px solid #007bff;
                        background-color: #e7f3ff;
                        margin: 2px;
                        font-weight: bold;
                    }
                """)
            else:
                button.setStyleSheet("""
                    QPushButton {
                        text-align: left;
                        padding: 10px;
                        border: 1px solid #dee2e6;
                        background-color: white;
                        margin: 2px;
                    }
                    QPushButton:hover {
                        background-color: #f8f9fa;
                    }
                    QPushButton:disabled {
                        background-color: #f8f9fa;
                        color: #6c757d;
                    }
                """)
```

## Integration with Main Application

### Connecting Navigation Components

```python
# In the main application initialization
def setup_navigation(self):
    """Setup navigation components."""
    # Initialize navigation manager
    self.navigation_manager = NavigationManager(self)
    
    # Setup navigation bar
    self.navigation_bar = NavigationBar(self)
    self.navigation_bar.previous_clicked.connect(self.handle_previous)
    self.navigation_bar.next_clicked.connect(self.handle_next)
    
    # Setup workflow actions
    self.workflow_actions = WorkflowActions(self)
    
    # Setup breadcrumb (optional)
    self.breadcrumb = BreadcrumbWidget(self)
    
    # Setup quick navigation menu (optional)
    self.quick_nav = QuickNavigationMenu(self)
    self.quick_nav.set_navigation_manager(self.navigation_manager)
    self.quick_nav.tab_requested.connect(self.handle_tab_request)
    
    # Connect tab widget
    self.tabs.set_navigation_manager(self.navigation_manager)
    self.tabs.currentChanged.connect(self.handle_tab_changed)
    
    # Initial state
    self.navigation_manager.update_tab_states()
    self.navigation_bar.update_progress(0)
    self.breadcrumb.update_breadcrumb(0)

def handle_previous(self):
    """Handle previous button click."""
    self.workflow_actions.handle_previous_button_click()

def handle_next(self):
    """Handle next button click."""
    self.workflow_actions.handle_next_button_click()

def handle_tab_request(self, tab_index: int):
    """Handle tab navigation request."""
    if self.tabs.navigate_to_tab(tab_index):
        self.navigation_bar.update_progress(tab_index)
        self.breadcrumb.update_breadcrumb(tab_index)
        self.quick_nav.highlight_current_tab(tab_index)

def handle_tab_changed(self, index: int):
    """Handle tab change event."""
    self.navigation_bar.update_progress(index)
    self.breadcrumb.update_breadcrumb(index)
    self.quick_nav.highlight_current_tab(index)
    self.workflow_actions.update_navigation_button_states(index)
```

## Best Practices

### 1. Progressive Disclosure
Only show information and options relevant to the current step to avoid overwhelming users.

### 2. Clear Visual Indicators
Use progress bars, breadcrumbs, and highlighting to show users where they are in the workflow.

### 3. Validation at Each Step
Validate user input before allowing progression to prevent errors downstream.

### 4. Graceful Error Handling
Provide clear error messages and guidance when validation fails.

### 5. Consistent Navigation
Maintain consistent navigation patterns throughout the application.

### 6. Keyboard Navigation
Support keyboard shortcuts for common navigation actions.

## Summary

This chapter covered comprehensive navigation and workflow management:

- **Workflow State Management**: Systematic approach to managing application state
- **Navigation Controls**: Custom navigation bars with progress indicators
- **Tab Management**: Enhanced tab widgets with validation and animations
- **Workflow Validation**: Comprehensive validation system for each step
- **Advanced Features**: Breadcrumb navigation and quick navigation menus
- **Integration**: Connecting all navigation components seamlessly

The navigation system provides a smooth, intuitive user experience that guides users through complex workflows while maintaining data integrity and preventing errors.

## Next Steps

In the next chapter, we'll explore error handling and validation techniques to create robust, user-friendly applications that gracefully handle edge cases and provide meaningful feedback to users.
