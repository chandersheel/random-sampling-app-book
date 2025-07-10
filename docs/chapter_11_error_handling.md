# Chapter 11: Error Handling and Validation

## Introduction

Robust error handling and validation are essential for creating professional applications that provide excellent user experiences. This chapter covers comprehensive error handling strategies, input validation techniques, and user feedback mechanisms for PyQt5 applications.

## Understanding Error Types

### Common Error Categories

In our random sampling application, we encounter several types of errors:

1. **File I/O Errors**: Issues with reading CSV files
2. **Data Validation Errors**: Invalid or corrupt data
3. **User Input Errors**: Invalid sampling parameters
4. **Memory Errors**: Large dataset handling issues
5. **Processing Errors**: Sampling algorithm failures
6. **Export Errors**: Issues saving results

```python
from PyQt5.QtWidgets import (QMessageBox, QProgressDialog, QLabel, 
                             QVBoxLayout, QHBoxLayout, QWidget, 
                             QTextEdit, QPushButton, QDialog)
from PyQt5.QtCore import Qt, QThread, pyqtSignal, QTimer
from PyQt5.QtGui import QFont, QPixmap, QIcon
import logging
import traceback
import sys
from typing import Optional, Callable, Any
from enum import Enum

class ErrorLevel(Enum):
    """Error severity levels."""
    INFO = "INFO"
    WARNING = "WARNING"
    ERROR = "ERROR"
    CRITICAL = "CRITICAL"

class ErrorHandler:
    """Centralized error handling system."""
    
    def __init__(self, parent=None):
        self.parent = parent
        self.setup_logging()
        self.error_history = []
        self.max_history = 100
    
    def setup_logging(self):
        """Setup logging configuration."""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('random_sampling_app.log'),
                logging.StreamHandler(sys.stdout)
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def handle_error(self, error: Exception, level: ErrorLevel = ErrorLevel.ERROR, 
                    user_message: str = None, show_dialog: bool = True):
        """
        Handle errors with appropriate logging and user feedback.
        
        Args:
            error: The exception that occurred
            level: Severity level of the error
            user_message: Custom message to show to user
            show_dialog: Whether to show error dialog to user
        """
        # Log the error
        error_msg = f"{type(error).__name__}: {str(error)}"
        
        if level == ErrorLevel.CRITICAL:
            self.logger.critical(error_msg, exc_info=True)
        elif level == ErrorLevel.ERROR:
            self.logger.error(error_msg, exc_info=True)
        elif level == ErrorLevel.WARNING:
            self.logger.warning(error_msg)
        else:
            self.logger.info(error_msg)
        
        # Store in error history
        self.add_to_history(error, level, user_message)
        
        # Show user dialog if requested
        if show_dialog:
            self.show_error_dialog(error, level, user_message)
    
    def add_to_history(self, error: Exception, level: ErrorLevel, user_message: str):
        """Add error to history with timestamp."""
        import datetime
        
        error_record = {
            'timestamp': datetime.datetime.now(),
            'error_type': type(error).__name__,
            'error_message': str(error),
            'level': level,
            'user_message': user_message,
            'traceback': traceback.format_exc()
        }
        
        self.error_history.append(error_record)
        
        # Maintain history size
        if len(self.error_history) > self.max_history:
            self.error_history.pop(0)
    
    def show_error_dialog(self, error: Exception, level: ErrorLevel, user_message: str):
        """Show appropriate error dialog based on error level."""
        if level == ErrorLevel.CRITICAL:
            self.show_critical_error_dialog(error, user_message)
        elif level == ErrorLevel.ERROR:
            self.show_error_dialog_standard(error, user_message)
        elif level == ErrorLevel.WARNING:
            self.show_warning_dialog(error, user_message)
        else:
            self.show_info_dialog(error, user_message)
    
    def show_critical_error_dialog(self, error: Exception, user_message: str):
        """Show critical error dialog."""
        message = user_message or f"A critical error occurred: {str(error)}"
        
        msg_box = QMessageBox(self.parent)
        msg_box.setIcon(QMessageBox.Critical)
        msg_box.setWindowTitle("Critical Error")
        msg_box.setText(message)
        msg_box.setDetailedText(traceback.format_exc())
        msg_box.setStandardButtons(QMessageBox.Ok)
        
        # Add custom buttons
        restart_button = msg_box.addButton("Restart Application", QMessageBox.ActionRole)
        report_button = msg_box.addButton("Report Error", QMessageBox.ActionRole)
        
        msg_box.exec_()
        
        # Handle button clicks
        if msg_box.clickedButton() == restart_button:
            self.restart_application()
        elif msg_box.clickedButton() == report_button:
            self.show_error_report_dialog(error)
    
    def show_error_dialog_standard(self, error: Exception, user_message: str):
        """Show standard error dialog."""
        message = user_message or f"An error occurred: {str(error)}"
        
        msg_box = QMessageBox(self.parent)
        msg_box.setIcon(QMessageBox.Critical)
        msg_box.setWindowTitle("Error")
        msg_box.setText(message)
        msg_box.setDetailedText(str(error))
        msg_box.setStandardButtons(QMessageBox.Ok | QMessageBox.Help)
        
        result = msg_box.exec_()
        
        if result == QMessageBox.Help:
            self.show_help_for_error(error)
    
    def show_warning_dialog(self, error: Exception, user_message: str):
        """Show warning dialog."""
        message = user_message or f"Warning: {str(error)}"
        
        QMessageBox.warning(self.parent, "Warning", message)
    
    def show_info_dialog(self, error: Exception, user_message: str):
        """Show info dialog."""
        message = user_message or f"Information: {str(error)}"
        
        QMessageBox.information(self.parent, "Information", message)
```

## Input Validation System

### Comprehensive Validation Framework

```python
class ValidationRule:
    """Base class for validation rules."""
    
    def __init__(self, error_message: str):
        self.error_message = error_message
    
    def validate(self, value: Any) -> tuple[bool, str]:
        """
        Validate a value.
        
        Returns:
            tuple: (is_valid, error_message)
        """
        raise NotImplementedError

class RequiredRule(ValidationRule):
    """Validation rule for required fields."""
    
    def __init__(self, error_message: str = "This field is required"):
        super().__init__(error_message)
    
    def validate(self, value: Any) -> tuple[bool, str]:
        if value is None or (isinstance(value, str) and not value.strip()):
            return False, self.error_message
        return True, ""

class NumericRangeRule(ValidationRule):
    """Validation rule for numeric ranges."""
    
    def __init__(self, min_value: float = None, max_value: float = None, 
                 error_message: str = None):
        self.min_value = min_value
        self.max_value = max_value
        
        if error_message is None:
            if min_value is not None and max_value is not None:
                error_message = f"Value must be between {min_value} and {max_value}"
            elif min_value is not None:
                error_message = f"Value must be at least {min_value}"
            elif max_value is not None:
                error_message = f"Value must be at most {max_value}"
            else:
                error_message = "Invalid numeric value"
        
        super().__init__(error_message)
    
    def validate(self, value: Any) -> tuple[bool, str]:
        try:
            numeric_value = float(value)
            
            if self.min_value is not None and numeric_value < self.min_value:
                return False, self.error_message
            
            if self.max_value is not None and numeric_value > self.max_value:
                return False, self.error_message
            
            return True, ""
        
        except (ValueError, TypeError):
            return False, "Value must be numeric"

class FileValidationRule(ValidationRule):
    """Validation rule for file paths."""
    
    def __init__(self, must_exist: bool = True, extensions: list = None):
        self.must_exist = must_exist
        self.extensions = extensions or []
        
        error_message = "Invalid file"
        if extensions:
            error_message += f" (must be {', '.join(extensions)})"
        
        super().__init__(error_message)
    
    def validate(self, value: Any) -> tuple[bool, str]:
        import os
        
        if not isinstance(value, str):
            return False, "File path must be a string"
        
        if not value.strip():
            return False, "File path cannot be empty"
        
        # Check if file exists
        if self.must_exist and not os.path.exists(value):
            return False, "File does not exist"
        
        # Check file extension
        if self.extensions:
            file_ext = os.path.splitext(value)[1].lower()
            if file_ext not in [ext.lower() for ext in self.extensions]:
                return False, f"File must have one of these extensions: {', '.join(self.extensions)}"
        
        return True, ""

class DataFrameValidationRule(ValidationRule):
    """Validation rule for pandas DataFrames."""
    
    def __init__(self, min_rows: int = None, min_cols: int = None, 
                 required_columns: list = None):
        self.min_rows = min_rows
        self.min_cols = min_cols
        self.required_columns = required_columns or []
        
        error_message = "Invalid dataset"
        super().__init__(error_message)
    
    def validate(self, value: Any) -> tuple[bool, str]:
        import pandas as pd
        
        if not isinstance(value, pd.DataFrame):
            return False, "Value must be a pandas DataFrame"
        
        if value.empty:
            return False, "Dataset cannot be empty"
        
        # Check minimum rows
        if self.min_rows is not None and len(value) < self.min_rows:
            return False, f"Dataset must have at least {self.min_rows} rows"
        
        # Check minimum columns
        if self.min_cols is not None and len(value.columns) < self.min_cols:
            return False, f"Dataset must have at least {self.min_cols} columns"
        
        # Check required columns
        missing_cols = [col for col in self.required_columns if col not in value.columns]
        if missing_cols:
            return False, f"Dataset missing required columns: {', '.join(missing_cols)}"
        
        return True, ""

class Validator:
    """Main validation class that applies multiple rules."""
    
    def __init__(self):
        self.rules = []
    
    def add_rule(self, rule: ValidationRule):
        """Add a validation rule."""
        self.rules.append(rule)
    
    def validate(self, value: Any) -> tuple[bool, list]:
        """
        Validate value against all rules.
        
        Returns:
            tuple: (is_valid, list_of_errors)
        """
        errors = []
        
        for rule in self.rules:
            is_valid, error_message = rule.validate(value)
            if not is_valid:
                errors.append(error_message)
        
        return len(errors) == 0, errors
    
    def validate_first_error(self, value: Any) -> tuple[bool, str]:
        """
        Validate value and return first error only.
        
        Returns:
            tuple: (is_valid, first_error_message)
        """
        for rule in self.rules:
            is_valid, error_message = rule.validate(value)
            if not is_valid:
                return False, error_message
        
        return True, ""
```

## Real-time Validation

### Form Validation with Visual Feedback

```python
from PyQt5.QtWidgets import QLineEdit, QSpinBox, QComboBox, QLabel
from PyQt5.QtCore import QTimer
from PyQt5.QtGui import QColor, QPalette

class ValidatedLineEdit(QLineEdit):
    """Line edit with real-time validation."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.validator = Validator()
        self.error_label = None
        self.validation_timer = QTimer()
        self.validation_timer.setSingleShot(True)
        self.validation_timer.timeout.connect(self.validate_input)
        
        # Connect text change to validation
        self.textChanged.connect(self.on_text_changed)
        
        # Setup styling
        self.setup_styling()
    
    def setup_styling(self):
        """Setup validation styling."""
        self.valid_style = """
            QLineEdit {
                border: 2px solid #28a745;
                border-radius: 4px;
                padding: 8px;
                background-color: #f8fff8;
            }
        """
        
        self.invalid_style = """
            QLineEdit {
                border: 2px solid #dc3545;
                border-radius: 4px;
                padding: 8px;
                background-color: #fff8f8;
            }
        """
        
        self.neutral_style = """
            QLineEdit {
                border: 1px solid #ced4da;
                border-radius: 4px;
                padding: 8px;
                background-color: white;
            }
        """
    
    def set_error_label(self, label: QLabel):
        """Set the label to display error messages."""
        self.error_label = label
        if self.error_label:
            self.error_label.setStyleSheet("color: #dc3545; font-size: 12px;")
            self.error_label.hide()
    
    def add_validation_rule(self, rule: ValidationRule):
        """Add a validation rule."""
        self.validator.add_rule(rule)
    
    def on_text_changed(self):
        """Handle text change with delayed validation."""
        # Reset styling to neutral
        self.setStyleSheet(self.neutral_style)
        if self.error_label:
            self.error_label.hide()
        
        # Start validation timer (debounced)
        self.validation_timer.start(500)  # 500ms delay
    
    def validate_input(self):
        """Validate current input."""
        is_valid, error_message = self.validator.validate_first_error(self.text())
        
        if is_valid:
            self.setStyleSheet(self.valid_style)
            if self.error_label:
                self.error_label.hide()
        else:
            self.setStyleSheet(self.invalid_style)
            if self.error_label:
                self.error_label.setText(error_message)
                self.error_label.show()
        
        return is_valid
    
    def is_valid(self) -> bool:
        """Check if current input is valid."""
        return self.validate_input()

class ValidatedSpinBox(QSpinBox):
    """Spin box with validation."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.validator = Validator()
        self.error_label = None
        
        # Connect value change to validation
        self.valueChanged.connect(self.validate_input)
        
        # Setup styling
        self.setup_styling()
    
    def setup_styling(self):
        """Setup validation styling."""
        self.valid_style = """
            QSpinBox {
                border: 2px solid #28a745;
                border-radius: 4px;
                padding: 8px;
                background-color: #f8fff8;
            }
        """
        
        self.invalid_style = """
            QSpinBox {
                border: 2px solid #dc3545;
                border-radius: 4px;
                padding: 8px;
                background-color: #fff8f8;
            }
        """
        
        self.neutral_style = """
            QSpinBox {
                border: 1px solid #ced4da;
                border-radius: 4px;
                padding: 8px;
                background-color: white;
            }
        """
    
    def set_error_label(self, label: QLabel):
        """Set the label to display error messages."""
        self.error_label = label
        if self.error_label:
            self.error_label.setStyleSheet("color: #dc3545; font-size: 12px;")
            self.error_label.hide()
    
    def add_validation_rule(self, rule: ValidationRule):
        """Add a validation rule."""
        self.validator.add_rule(rule)
    
    def validate_input(self):
        """Validate current input."""
        is_valid, error_message = self.validator.validate_first_error(self.value())
        
        if is_valid:
            self.setStyleSheet(self.valid_style)
            if self.error_label:
                self.error_label.hide()
        else:
            self.setStyleSheet(self.invalid_style)
            if self.error_label:
                self.error_label.setText(error_message)
                self.error_label.show()
        
        return is_valid
    
    def is_valid(self) -> bool:
        """Check if current input is valid."""
        return self.validate_input()
```

## File Validation and Error Handling

### CSV File Validation

```python
class CSVValidator:
    """Specialized validator for CSV files."""
    
    def __init__(self, error_handler: ErrorHandler):
        self.error_handler = error_handler
    
    def validate_csv_file(self, file_path: str) -> tuple[bool, str, Optional[pd.DataFrame]]:
        """
        Comprehensive CSV file validation.
        
        Returns:
            tuple: (is_valid, error_message, dataframe_or_none)
        """
        try:
            # Check if file exists
            if not os.path.exists(file_path):
                return False, "File does not exist", None
            
            # Check file extension
            if not file_path.lower().endswith('.csv'):
                return False, "File must be a CSV file", None
            
            # Check file size
            file_size = os.path.getsize(file_path)
            if file_size == 0:
                return False, "File is empty", None
            
            # Check if file is too large (>100MB)
            if file_size > 100 * 1024 * 1024:
                return False, "File is too large (>100MB)", None
            
            # Try to read the file
            dataframe = self.read_csv_with_error_handling(file_path)
            
            if dataframe is None:
                return False, "Failed to read CSV file", None
            
            # Validate dataframe content
            validation_result = self.validate_dataframe_content(dataframe)
            
            if not validation_result[0]:
                return False, validation_result[1], None
            
            return True, "File is valid", dataframe
            
        except Exception as e:
            self.error_handler.handle_error(
                e, ErrorLevel.ERROR, 
                "Error validating CSV file", 
                show_dialog=False
            )
            return False, f"Validation error: {str(e)}", None
    
    def read_csv_with_error_handling(self, file_path: str) -> Optional[pd.DataFrame]:
        """Read CSV file with comprehensive error handling."""
        import pandas as pd
        
        # Try different encodings
        encodings = ['utf-8', 'latin-1', 'cp1252', 'iso-8859-1']
        
        for encoding in encodings:
            try:
                # Try different separators
                separators = [',', ';', '\t', '|']
                
                for sep in separators:
                    try:
                        df = pd.read_csv(file_path, encoding=encoding, sep=sep)
                        
                        # Check if we got meaningful data
                        if len(df.columns) > 1 and len(df) > 0:
                            return df
                            
                    except Exception:
                        continue
                        
            except Exception:
                continue
        
        return None
    
    def validate_dataframe_content(self, df: pd.DataFrame) -> tuple[bool, str]:
        """Validate DataFrame content."""
        try:
            # Check if DataFrame is empty
            if df.empty:
                return False, "Dataset is empty"
            
            # Check for minimum dimensions
            if len(df) < 1:
                return False, "Dataset must have at least 1 row"
            
            if len(df.columns) < 1:
                return False, "Dataset must have at least 1 column"
            
            # Check for excessive missing data
            missing_percentage = (df.isnull().sum().sum() / (len(df) * len(df.columns))) * 100
            if missing_percentage > 90:
                return False, f"Dataset has too much missing data ({missing_percentage:.1f}%)"
            
            # Check for duplicate column names
            if len(df.columns) != len(set(df.columns)):
                return False, "Dataset has duplicate column names"
            
            # Check for very large datasets
            if len(df) > 1000000:
                return False, "Dataset is too large (>1M rows). Please use a smaller sample."
            
            return True, "Dataset is valid"
            
        except Exception as e:
            return False, f"Error validating dataset: {str(e)}"
```

## Progress Tracking and User Feedback

### Progress Dialog with Error Handling

```python
class ProgressDialog(QProgressDialog):
    """Enhanced progress dialog with error handling."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.error_handler = None
        self.operation_cancelled = False
        self.setup_ui()
    
    def setup_ui(self):
        """Setup progress dialog UI."""
        self.setWindowTitle("Processing...")
        self.setModal(True)
        self.setMinimumWidth(400)
        self.setAutoClose(False)
        self.setAutoReset(False)
        
        # Custom styling
        self.setStyleSheet("""
            QProgressDialog {
                background-color: white;
                border: 1px solid #dee2e6;
                border-radius: 8px;
            }
            QProgressBar {
                border: 1px solid #dee2e6;
                border-radius: 4px;
                text-align: center;
                background-color: #e9ecef;
            }
            QProgressBar::chunk {
                background-color: #007bff;
                border-radius: 3px;
            }
        """)
    
    def set_error_handler(self, error_handler: ErrorHandler):
        """Set error handler for progress operations."""
        self.error_handler = error_handler
    
    def run_operation(self, operation: Callable, *args, **kwargs) -> Any:
        """
        Run an operation with progress tracking and error handling.
        
        Args:
            operation: Function to execute
            *args: Arguments for the operation
            **kwargs: Keyword arguments for the operation
            
        Returns:
            Operation result or None if failed/cancelled
        """
        try:
            self.operation_cancelled = False
            self.show()
            
            # Run the operation
            result = operation(*args, **kwargs)
            
            if not self.operation_cancelled:
                self.setValue(self.maximum())
                self.setLabelText("Operation completed successfully")
                
                # Show completion briefly
                QTimer.singleShot(1000, self.accept)
                
                return result
            
        except Exception as e:
            if self.error_handler:
                self.error_handler.handle_error(
                    e, ErrorLevel.ERROR,
                    "Operation failed",
                    show_dialog=True
                )
            
            self.reject()
            return None
    
    def update_progress(self, value: int, message: str = None):
        """Update progress with optional message."""
        if not self.operation_cancelled:
            self.setValue(value)
            if message:
                self.setLabelText(message)
    
    def cancel_operation(self):
        """Cancel the current operation."""
        self.operation_cancelled = True
        self.reject()
```

## Advanced Error Recovery

### Auto-Recovery System

```python
class AutoRecoverySystem:
    """System for automatic error recovery."""
    
    def __init__(self, app):
        self.app = app
        self.recovery_strategies = {}
        self.setup_recovery_strategies()
    
    def setup_recovery_strategies(self):
        """Setup automatic recovery strategies."""
        self.recovery_strategies = {
            'FileNotFoundError': self.recover_file_not_found,
            'PermissionError': self.recover_permission_error,
            'MemoryError': self.recover_memory_error,
            'pd.errors.EmptyDataError': self.recover_empty_data,
            'pd.errors.ParserError': self.recover_parser_error,
        }
    
    def attempt_recovery(self, error: Exception) -> bool:
        """
        Attempt to recover from an error.
        
        Returns:
            bool: True if recovery was successful
        """
        error_type = type(error).__name__
        
        if error_type in self.recovery_strategies:
            try:
                return self.recovery_strategies[error_type](error)
            except Exception as recovery_error:
                self.app.error_handler.handle_error(
                    recovery_error, ErrorLevel.WARNING,
                    f"Recovery attempt failed: {str(recovery_error)}"
                )
        
        return False
    
    def recover_file_not_found(self, error: FileNotFoundError) -> bool:
        """Recover from file not found error."""
        # Try to find similar files in the same directory
        import os
        import glob
        
        file_path = str(error).split("'")[1] if "'" in str(error) else ""
        
        if file_path:
            directory = os.path.dirname(file_path)
            filename = os.path.basename(file_path)
            
            # Look for similar files
            similar_files = glob.glob(os.path.join(directory, "*.csv"))
            
            if similar_files:
                # Show dialog to user to select alternative
                return self.show_file_recovery_dialog(similar_files)
        
        return False
    
    def recover_permission_error(self, error: PermissionError) -> bool:
        """Recover from permission error."""
        # Suggest copying file to user directory
        msg = QMessageBox()
        msg.setIcon(QMessageBox.Warning)
        msg.setWindowTitle("Permission Error")
        msg.setText("Permission denied. Try copying the file to your Documents folder.")
        msg.exec_()
        
        return False  # Let user handle manually
    
    def recover_memory_error(self, error: MemoryError) -> bool:
        """Recover from memory error."""
        # Suggest reducing dataset size
        msg = QMessageBox()
        msg.setIcon(QMessageBox.Warning)
        msg.setWindowTitle("Memory Error")
        msg.setText("Not enough memory. Try using a smaller dataset or closing other applications.")
        msg.exec_()
        
        return False
    
    def recover_empty_data(self, error: Exception) -> bool:
        """Recover from empty data error."""
        # Check if file has headers only
        msg = QMessageBox()
        msg.setIcon(QMessageBox.Warning)
        msg.setWindowTitle("Empty Data")
        msg.setText("The file appears to be empty or contains only headers.")
        msg.exec_()
        
        return False
    
    def recover_parser_error(self, error: Exception) -> bool:
        """Recover from CSV parser error."""
        # Try alternative parsing methods
        msg = QMessageBox()
        msg.setIcon(QMessageBox.Question)
        msg.setWindowTitle("Parsing Error")
        msg.setText("Error parsing CSV file. Try using a different delimiter?")
        msg.setStandardButtons(QMessageBox.Yes | QMessageBox.No)
        
        if msg.exec_() == QMessageBox.Yes:
            # Trigger alternative parsing dialog
            return self.show_parsing_options_dialog()
        
        return False
    
    def show_file_recovery_dialog(self, similar_files: list) -> bool:
        """Show dialog to select alternative file."""
        dialog = QDialog(self.app)
        dialog.setWindowTitle("Select Alternative File")
        dialog.setModal(True)
        
        layout = QVBoxLayout(dialog)
        
        label = QLabel("File not found. Select an alternative:")
        layout.addWidget(label)
        
        # File list
        from PyQt5.QtWidgets import QListWidget
        file_list = QListWidget()
        file_list.addItems(similar_files)
        layout.addWidget(file_list)
        
        # Buttons
        button_layout = QHBoxLayout()
        ok_button = QPushButton("OK")
        cancel_button = QPushButton("Cancel")
        
        ok_button.clicked.connect(dialog.accept)
        cancel_button.clicked.connect(dialog.reject)
        
        button_layout.addWidget(ok_button)
        button_layout.addWidget(cancel_button)
        layout.addLayout(button_layout)
        
        if dialog.exec_() == QDialog.Accepted:
            selected_items = file_list.selectedItems()
            if selected_items:
                selected_file = selected_items[0].text()
                # Try to load the selected file
                try:
                    self.app.load_file_directly(selected_file)
                    return True
                except Exception:
                    pass
        
        return False
```

## Integration with Main Application

### Error Handling Setup

```python
# In the main application
def setup_error_handling(self):
    """Setup comprehensive error handling."""
    self.error_handler = ErrorHandler(self)
    self.auto_recovery = AutoRecoverySystem(self)
    
    # Setup global exception handler
    sys.excepthook = self.global_exception_handler
    
    # Setup validation for input fields
    self.setup_input_validation()

def global_exception_handler(self, exc_type, exc_value, exc_traceback):
    """Global exception handler."""
    if issubclass(exc_type, KeyboardInterrupt):
        sys.__excepthook__(exc_type, exc_value, exc_traceback)
        return
    
    # Log the error
    self.error_handler.handle_error(
        exc_value, ErrorLevel.CRITICAL,
        "An unexpected error occurred",
        show_dialog=True
    )

def setup_input_validation(self):
    """Setup validation for input fields."""
    # Sample size validation
    if hasattr(self, 'size_spinbox'):
        size_validator = Validator()
        size_validator.add_rule(NumericRangeRule(min_value=1, max_value=1000000))
        
        # Convert to validated spinbox
        self.size_spinbox = ValidatedSpinBox(self.size_spinbox.parent())
        self.size_spinbox.validator = size_validator
        
        # Add error label
        self.size_error_label = QLabel()
        self.size_spinbox.set_error_label(self.size_error_label)

def safe_operation(self, operation, *args, **kwargs):
    """Execute operation with error handling and recovery."""
    try:
        return operation(*args, **kwargs)
    except Exception as e:
        # Try automatic recovery first
        if self.auto_recovery.attempt_recovery(e):
            # Recovery successful, try operation again
            try:
                return operation(*args, **kwargs)
            except Exception as retry_error:
                self.error_handler.handle_error(retry_error, ErrorLevel.ERROR)
        else:
            # No recovery possible
            self.error_handler.handle_error(e, ErrorLevel.ERROR)
        
        return None
```

## Best Practices

### 1. Layered Error Handling
Implement multiple layers of error handling: validation, operation-level, and global handlers.

### 2. User-Friendly Messages
Always provide clear, actionable error messages to users.

### 3. Graceful Degradation
When possible, allow the application to continue functioning even after errors.

### 4. Comprehensive Logging
Log all errors with sufficient context for debugging.

### 5. Recovery Strategies
Implement automatic recovery for common, recoverable errors.

### 6. Input Validation
Validate user input early and provide immediate feedback.

## Summary

This chapter covered comprehensive error handling and validation:

- **Error Handling System**: Centralized error handling with logging and user feedback
- **Validation Framework**: Flexible validation rules for different data types
- **Real-time Validation**: Visual feedback for user input validation
- **File Validation**: Specialized validation for CSV files and data integrity
- **Progress Tracking**: Progress dialogs with error handling
- **Auto-Recovery**: Automatic recovery strategies for common errors
- **Integration**: Seamless integration with the main application

The error handling system provides a robust foundation for creating reliable, user-friendly applications that gracefully handle edge cases and unexpected situations.

## Next Steps

In the next chapter, we'll explore testing and deployment strategies to ensure our application is ready for production use and can be easily distributed to end users.
