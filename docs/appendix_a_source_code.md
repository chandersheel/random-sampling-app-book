# Appendix A: Complete Source Code

## Introduction

This appendix contains the complete, production-ready source code for the "Randomly Yours" random sampling application. The code is organized into logical modules and follows best practices for maintainability, readability, and extensibility.

## Project Structure

```
random_sampling_app/
├── app/
│   ├── main.py              # Main application file
│   ├── sampling.py          # Sampling engine and algorithms
│   ├── version.py           # Version information
│   └── __init__.py          # Package initialization
├── resources/
│   ├── icon.ico             # Application icon
│   └── style.qss            # External stylesheet (optional)
├── tests/
│   ├── test_main.py         # Tests for main application
│   ├── test_sampling.py     # Tests for sampling engine
│   └── __init__.py          # Test package initialization
├── requirements.txt         # Python dependencies
├── setup.py                 # Package setup script
├── README.md               # Project documentation
└── LICENSE                 # License file
```

## Core Application Files

### main.py - Main Application File

```python
"""
Randomly Yours - Professional Random Sampling Application
A comprehensive PyQt5-based application for statistical sampling.
"""

import sys
import os
import pandas as pd
import numpy as np
from typing import Optional, Union
from enum import Enum

from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
    QTabWidget, QLabel, QPushButton, QSpinBox, QComboBox, QCheckBox,
    QTableWidget, QTableWidgetItem, QFileDialog, QMessageBox,
    QProgressBar, QFrame, QSizePolicy, QHeaderView, QAbstractItemView
)
from PyQt5.QtCore import Qt, QSize, QTimer, pyqtSignal
from PyQt5.QtGui import QFont, QIcon, QPixmap, QColor

from sampling import SamplingEngine
from version import __version__, get_version_info


class WorkflowState(Enum):
    """Enumeration of workflow states."""
    INITIAL = 0
    FILE_LOADED = 1
    DATA_PREVIEWED = 2
    METHOD_SELECTED = 3
    SIZE_CONFIGURED = 4
    SETTINGS_CONFIGURED = 5
    SAMPLE_GENERATED = 6


class NavigationManager:
    """Manages navigation state and workflow progression."""
    
    def __init__(self, parent):
        self.parent = parent
        self.current_state = WorkflowState.INITIAL
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
    
    def enable_tab(self, tab_index: int):
        """Enable a specific tab."""
        self.tab_states[tab_index] = True
        self.update_tab_states()
    
    def set_workflow_state(self, state: WorkflowState):
        """Set the current workflow state and update UI accordingly."""
        self.current_state = state
        self.update_navigation_based_on_state()
    
    def update_navigation_based_on_state(self):
        """Update navigation controls based on current workflow state."""
        if self.current_state == WorkflowState.FILE_LOADED:
            self.enable_tab(1)  # Data Preview
        elif self.current_state == WorkflowState.DATA_PREVIEWED:
            self.enable_tab(2)  # Method
        elif self.current_state == WorkflowState.METHOD_SELECTED:
            self.enable_tab(3)  # Size
        elif self.current_state == WorkflowState.SIZE_CONFIGURED:
            self.enable_tab(4)  # Settings
        elif self.current_state == WorkflowState.SAMPLE_GENERATED:
            self.enable_tab(5)  # Results


class RandomSamplingApp(QMainWindow):
    """Main application window for the random sampling application."""
    
    def __init__(self):
        super().__init__()
        
        # Initialize data attributes
        self.current_data = None
        self.original_data = None
        self.sample_data = None
        
        # Initialize components
        self.sampling_engine = SamplingEngine()
        self.navigation_manager = NavigationManager(self)
        
        # Setup UI
        self.setup_ui()
        self.setup_navigation()
        self.apply_styles()
        
        # Initialize workflow state
        self.navigation_manager.update_tab_states()
    
    def setup_ui(self):
        """Setup the main user interface."""
        self.setWindowTitle("Randomly Yours - Professional Random Sampling")
        self.setMinimumSize(1200, 800)
        self.resize(1400, 1000)
        
        # Create central widget and main layout
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        main_layout = QVBoxLayout(central_widget)
        main_layout.setContentsMargins(0, 0, 0, 0)
        main_layout.setSpacing(0)
        
        # Create header
        header = self.create_header()
        main_layout.addWidget(header)
        
        # Create tab widget
        self.tabs = QTabWidget()
        self.tabs.setTabPosition(QTabWidget.North)
        self.tabs.currentChanged.connect(self.on_tab_changed)
        
        # Create tabs
        self.create_file_tab()
        self.create_data_preview_tab()
        self.create_method_tab()
        self.create_size_tab()
        self.create_settings_tab()
        self.create_results_tab()
        
        main_layout.addWidget(self.tabs)
        
        # Create navigation bar
        nav_bar = self.create_navigation_bar()
        main_layout.addWidget(nav_bar)
        
        # Create status bar
        self.create_status_bar()
    
    def create_header(self):
        """Create the application header."""
        header = QFrame()
        header.setFixedHeight(80)
        header.setStyleSheet("""
            QFrame {
                background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
                    stop:0 #4a90e2, stop:1 #357abd);
                border: none;
            }
        """)
        
        header_layout = QHBoxLayout(header)
        header_layout.setContentsMargins(30, 10, 30, 10)
        
        # Title
        title_label = QLabel("Randomly Yours")
        title_label.setFont(QFont("Segoe UI", 24, QFont.Bold))
        title_label.setStyleSheet("color: white;")
        
        # Subtitle
        subtitle_label = QLabel("Professional Random Sampling Application")
        subtitle_label.setFont(QFont("Segoe UI", 12))
        subtitle_label.setStyleSheet("color: #e8f4f8;")
        
        # Title container
        title_container = QWidget()
        title_layout = QVBoxLayout(title_container)
        title_layout.setContentsMargins(0, 0, 0, 0)
        title_layout.setSpacing(0)
        title_layout.addWidget(title_label)
        title_layout.addWidget(subtitle_label)
        
        header_layout.addWidget(title_container)
        header_layout.addStretch()
        
        # Version info
        version_label = QLabel(f"Version {__version__}")
        version_label.setFont(QFont("Segoe UI", 10))
        version_label.setStyleSheet("color: #e8f4f8;")
        header_layout.addWidget(version_label)
        
        return header
    
    def create_file_tab(self):
        """Create the file loading tab."""
        file_tab = QWidget()
        layout = QVBoxLayout(file_tab)
        layout.setContentsMargins(40, 40, 40, 40)
        layout.setSpacing(30)
        
        # Title
        title = QLabel("Load Your Dataset")
        title.setFont(QFont("Segoe UI", 20, QFont.Bold))
        title.setStyleSheet("color: #2c3e50; margin-bottom: 10px;")
        layout.addWidget(title)
        
        # Description
        desc = QLabel("Select a CSV file containing your data for sampling analysis.")
        desc.setFont(QFont("Segoe UI", 12))
        desc.setStyleSheet("color: #7f8c8d; margin-bottom: 30px;")
        layout.addWidget(desc)
        
        # File selection area
        file_frame = QFrame()
        file_frame.setStyleSheet("""
            QFrame {
                background-color: #f8f9fa;
                border: 2px dashed #dee2e6;
                border-radius: 10px;
                padding: 40px;
            }
        """)
        
        file_layout = QVBoxLayout(file_frame)
        file_layout.setAlignment(Qt.AlignCenter)
        file_layout.setSpacing(20)
        
        # File icon (placeholder)
        file_icon = QLabel("📁")
        file_icon.setFont(QFont("Segoe UI", 48))
        file_icon.setAlignment(Qt.AlignCenter)
        file_layout.addWidget(file_icon)
        
        # File selection button
        self.file_button = QPushButton("Select CSV File")
        self.file_button.setFont(QFont("Segoe UI", 14, QFont.Bold))
        self.file_button.setMinimumSize(200, 50)
        self.file_button.clicked.connect(self.load_file)
        file_layout.addWidget(self.file_button)
        
        # File info
        self.file_info = QLabel("No file selected")
        self.file_info.setFont(QFont("Segoe UI", 11))
        self.file_info.setStyleSheet("color: #6c757d;")
        self.file_info.setAlignment(Qt.AlignCenter)
        file_layout.addWidget(self.file_info)
        
        layout.addWidget(file_frame)
        layout.addStretch()
        
        self.tabs.addTab(file_tab, "1. File")
    
    def create_data_preview_tab(self):
        """Create the data preview tab."""
        preview_tab = QWidget()
        layout = QVBoxLayout(preview_tab)
        layout.setContentsMargins(20, 20, 20, 20)
        layout.setSpacing(20)
        
        # Title and info
        header_layout = QHBoxLayout()
        
        title = QLabel("Data Preview")
        title.setFont(QFont("Segoe UI", 18, QFont.Bold))
        title.setStyleSheet("color: #2c3e50;")
        header_layout.addWidget(title)
        
        header_layout.addStretch()
        
        self.data_preview_info = QLabel("No data loaded")
        self.data_preview_info.setFont(QFont("Segoe UI", 11))
        self.data_preview_info.setStyleSheet("color: #6c757d;")
        header_layout.addWidget(self.data_preview_info)
        
        layout.addLayout(header_layout)
        
        # Data preview table
        self.data_preview_table = QTableWidget()
        self.setup_table_widget(self.data_preview_table)
        layout.addWidget(self.data_preview_table)
        
        self.tabs.addTab(preview_tab, "2. Data Preview")
    
    def create_method_tab(self):
        """Create the sampling method selection tab."""
        method_tab = QWidget()
        layout = QVBoxLayout(method_tab)
        layout.setContentsMargins(40, 40, 40, 40)
        layout.setSpacing(30)
        
        # Title
        title = QLabel("Choose Sampling Method")
        title.setFont(QFont("Segoe UI", 20, QFont.Bold))
        title.setStyleSheet("color: #2c3e50; margin-bottom: 10px;")
        layout.addWidget(title)
        
        # Method selection
        method_frame = QFrame()
        method_frame.setStyleSheet("""
            QFrame {
                background-color: #ffffff;
                border: 1px solid #dee2e6;
                border-radius: 8px;
                padding: 30px;
            }
        """)
        
        method_layout = QVBoxLayout(method_frame)
        method_layout.setSpacing(20)
        
        # Method combo box
        method_label = QLabel("Sampling Method:")
        method_label.setFont(QFont("Segoe UI", 14, QFont.Bold))
        method_layout.addWidget(method_label)
        
        self.method_combo = QComboBox()
        self.method_combo.setFont(QFont("Segoe UI", 12))
        self.method_combo.setMinimumHeight(40)
        self.method_combo.addItems(["Simple Random", "Systematic", "Stratified"])
        self.method_combo.currentTextChanged.connect(self.on_method_changed)
        method_layout.addWidget(self.method_combo)
        
        # Stratification options (initially hidden)
        self.strata_widget = QWidget()
        strata_layout = QVBoxLayout(self.strata_widget)
        strata_layout.setContentsMargins(0, 20, 0, 0)
        
        strata_label = QLabel("Stratification Column:")
        strata_label.setFont(QFont("Segoe UI", 12, QFont.Bold))
        strata_layout.addWidget(strata_label)
        
        self.strata_combo = QComboBox()
        self.strata_combo.setFont(QFont("Segoe UI", 12))
        self.strata_combo.setMinimumHeight(40)
        strata_layout.addWidget(self.strata_combo)
        
        method_layout.addWidget(self.strata_widget)
        self.strata_widget.hide()
        
        # Method description
        self.method_description = QLabel()
        self.method_description.setFont(QFont("Segoe UI", 11))
        self.method_description.setStyleSheet("color: #6c757d; margin-top: 20px;")
        self.method_description.setWordWrap(True)
        method_layout.addWidget(self.method_description)
        
        layout.addWidget(method_frame)
        layout.addStretch()
        
        # Update description
        self.update_method_description()
        
        self.tabs.addTab(method_tab, "3. Method")
    
    def create_size_tab(self):
        """Create the sample size configuration tab."""
        size_tab = QWidget()
        layout = QVBoxLayout(size_tab)
        layout.setContentsMargins(40, 40, 40, 40)
        layout.setSpacing(30)
        
        # Title
        title = QLabel("Configure Sample Size")
        title.setFont(QFont("Segoe UI", 20, QFont.Bold))
        title.setStyleSheet("color: #2c3e50; margin-bottom: 10px;")
        layout.addWidget(title)
        
        # Size configuration
        size_frame = QFrame()
        size_frame.setStyleSheet("""
            QFrame {
                background-color: #ffffff;
                border: 1px solid #dee2e6;
                border-radius: 8px;
                padding: 30px;
            }
        """)
        
        size_layout = QVBoxLayout(size_frame)
        size_layout.setSpacing(25)
        
        # Sample size input
        size_input_layout = QHBoxLayout()
        
        size_label = QLabel("Sample Size:")
        size_label.setFont(QFont("Segoe UI", 14, QFont.Bold))
        size_input_layout.addWidget(size_label)
        
        self.size_spinbox = QSpinBox()
        self.size_spinbox.setFont(QFont("Segoe UI", 12))
        self.size_spinbox.setMinimumHeight(40)
        self.size_spinbox.setMinimum(1)
        self.size_spinbox.setMaximum(1000000)
        self.size_spinbox.setValue(100)
        self.size_spinbox.valueChanged.connect(self.update_size_info)
        size_input_layout.addWidget(self.size_spinbox)
        
        size_input_layout.addStretch()
        size_layout.addLayout(size_input_layout)
        
        # Size information
        self.size_info = QLabel()
        self.size_info.setFont(QFont("Segoe UI", 11))
        self.size_info.setStyleSheet("color: #6c757d;")
        size_layout.addWidget(self.size_info)
        
        layout.addWidget(size_frame)
        layout.addStretch()
        
        self.tabs.addTab(size_tab, "4. Size")
    
    def create_settings_tab(self):
        """Create the settings tab."""
        settings_tab = QWidget()
        layout = QVBoxLayout(settings_tab)
        layout.setContentsMargins(40, 40, 40, 40)
        layout.setSpacing(30)
        
        # Title
        title = QLabel("Advanced Settings")
        title.setFont(QFont("Segoe UI", 20, QFont.Bold))
        title.setStyleSheet("color: #2c3e50; margin-bottom: 10px;")
        layout.addWidget(title)
        
        # Settings frame
        settings_frame = QFrame()
        settings_frame.setStyleSheet("""
            QFrame {
                background-color: #ffffff;
                border: 1px solid #dee2e6;
                border-radius: 8px;
                padding: 30px;
            }
        """)
        
        settings_layout = QVBoxLayout(settings_frame)
        settings_layout.setSpacing(20)
        
        # Replacement option
        self.replacement_checkbox = QCheckBox("Allow replacement (sampling with replacement)")
        self.replacement_checkbox.setFont(QFont("Segoe UI", 12))
        self.replacement_checkbox.setChecked(False)
        settings_layout.addWidget(self.replacement_checkbox)
        
        # Random seed option
        seed_layout = QHBoxLayout()
        
        self.seed_checkbox = QCheckBox("Use random seed for reproducibility:")
        self.seed_checkbox.setFont(QFont("Segoe UI", 12))
        seed_layout.addWidget(self.seed_checkbox)
        
        self.seed_spinbox = QSpinBox()
        self.seed_spinbox.setFont(QFont("Segoe UI", 10))
        self.seed_spinbox.setMinimumHeight(30)
        self.seed_spinbox.setMinimum(0)
        self.seed_spinbox.setMaximum(999999)
        self.seed_spinbox.setValue(42)
        self.seed_spinbox.setEnabled(False)
        seed_layout.addWidget(self.seed_spinbox)
        
        seed_layout.addStretch()
        settings_layout.addLayout(seed_layout)
        
        # Connect seed checkbox
        self.seed_checkbox.toggled.connect(self.seed_spinbox.setEnabled)
        
        layout.addWidget(settings_frame)
        layout.addStretch()
        
        self.tabs.addTab(settings_tab, "5. Settings")
    
    def create_results_tab(self):
        """Create the results tab."""
        results_tab = QWidget()
        layout = QVBoxLayout(results_tab)
        layout.setContentsMargins(20, 20, 20, 20)
        layout.setSpacing(20)
        
        # Header
        header_layout = QHBoxLayout()
        
        title = QLabel("Sampling Results")
        title.setFont(QFont("Segoe UI", 18, QFont.Bold))
        title.setStyleSheet("color: #2c3e50;")
        header_layout.addWidget(title)
        
        header_layout.addStretch()
        
        # Export button
        self.export_button = QPushButton("Export Sample")
        self.export_button.setFont(QFont("Segoe UI", 12, QFont.Bold))
        self.export_button.setMinimumSize(150, 40)
        self.export_button.clicked.connect(self.export_sample)
        self.export_button.setEnabled(False)
        header_layout.addWidget(self.export_button)
        
        layout.addLayout(header_layout)
        
        # Results info
        self.results_info = QLabel("No sample generated yet")
        self.results_info.setFont(QFont("Segoe UI", 11))
        self.results_info.setStyleSheet("color: #6c757d;")
        layout.addWidget(self.results_info)
        
        # Results table
        self.results_table = QTableWidget()
        self.setup_table_widget(self.results_table)
        layout.addWidget(self.results_table)
        
        self.tabs.addTab(results_tab, "6. Results")
    
    def setup_table_widget(self, table):
        """Setup common table widget properties."""
        table.setSortingEnabled(True)
        table.setAlternatingRowColors(True)
        table.setSelectionBehavior(QAbstractItemView.SelectRows)
        table.horizontalHeader().setSectionResizeMode(QHeaderView.Stretch)
        table.verticalHeader().setVisible(False)
        table.setMinimumHeight(400)
    
    def create_navigation_bar(self):
        """Create the navigation bar."""
        nav_bar = QFrame()
        nav_bar.setFixedHeight(80)
        nav_bar.setStyleSheet("""
            QFrame {
                background-color: #f8f9fa;
                border-top: 1px solid #dee2e6;
            }
        """)
        
        nav_layout = QHBoxLayout(nav_bar)
        nav_layout.setContentsMargins(30, 20, 30, 20)
        
        # Progress indicator
        self.progress_label = QLabel("Step 1 of 6")
        self.progress_label.setFont(QFont("Segoe UI", 12))
        self.progress_label.setStyleSheet("color: #6c757d;")
        nav_layout.addWidget(self.progress_label)
        
        nav_layout.addStretch()
        
        # Navigation buttons
        self.prev_button = QPushButton("← Previous")
        self.prev_button.setFont(QFont("Segoe UI", 12, QFont.Bold))
        self.prev_button.setMinimumSize(120, 40)
        self.prev_button.clicked.connect(self.go_previous)
        self.prev_button.setEnabled(False)
        nav_layout.addWidget(self.prev_button)
        
        self.next_button = QPushButton("Next →")
        self.next_button.setFont(QFont("Segoe UI", 12, QFont.Bold))
        self.next_button.setMinimumSize(120, 40)
        self.next_button.clicked.connect(self.go_next)
        self.next_button.setEnabled(False)
        nav_layout.addWidget(self.next_button)
        
        return nav_bar
    
    def create_status_bar(self):
        """Create the status bar."""
        self.status_bar = self.statusBar()
        self.status_bar.setStyleSheet("""
            QStatusBar {
                background-color: #f8f9fa;
                border-top: 1px solid #dee2e6;
                padding: 5px;
            }
        """)
        
        self.status_label = QLabel("Ready")
        self.status_label.setFont(QFont("Segoe UI", 10))
        self.status_bar.addWidget(self.status_label)
    
    def setup_navigation(self):
        """Setup navigation event handlers."""
        self.tabs.currentChanged.connect(self.on_tab_changed)
    
    def apply_styles(self):
        """Apply application-wide styles."""
        style = """
            QMainWindow {
                background-color: #f8f9fa;
            }
            
            QTabWidget::pane {
                border: 1px solid #dee2e6;
                background-color: #ffffff;
            }
            
            QTabWidget::tab-bar {
                alignment: center;
            }
            
            QTabBar::tab {
                background-color: #e9ecef;
                color: #495057;
                padding: 12px 24px;
                margin-right: 2px;
                border-top-left-radius: 6px;
                border-top-right-radius: 6px;
                font-size: 14px;
                font-weight: bold;
                min-width: 120px;
            }
            
            QTabBar::tab:selected {
                background-color: #ffffff;
                color: #007bff;
                border-bottom: 2px solid #007bff;
            }
            
            QTabBar::tab:hover {
                background-color: #dee2e6;
            }
            
            QTabBar::tab:disabled {
                background-color: #f8f9fa;
                color: #adb5bd;
            }
            
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
            
            QComboBox, QSpinBox {
                border: 1px solid #ced4da;
                border-radius: 4px;
                padding: 8px;
                background-color: white;
                font-size: 12px;
            }
            
            QComboBox:focus, QSpinBox:focus {
                border: 2px solid #007bff;
                outline: none;
            }
            
            QTableWidget {
                background-color: #ffffff;
                alternate-background-color: #f8f9fa;
                selection-background-color: #007bff;
                selection-color: white;
                border: 1px solid #dee2e6;
                gridline-color: #e9ecef;
            }
            
            QTableWidget::item {
                padding: 8px;
                border: none;
            }
            
            QTableWidget::item:selected {
                background-color: #007bff;
                color: white;
            }
            
            QHeaderView::section {
                background-color: #f8f9fa;
                border: 1px solid #dee2e6;
                padding: 8px;
                font-weight: bold;
                color: #495057;
            }
            
            QHeaderView::section:hover {
                background-color: #e9ecef;
            }
            
            QCheckBox {
                font-size: 12px;
                color: #495057;
            }
            
            QCheckBox::indicator {
                width: 18px;
                height: 18px;
                border: 2px solid #ced4da;
                border-radius: 3px;
                background-color: white;
            }
            
            QCheckBox::indicator:checked {
                background-color: #007bff;
                border: 2px solid #007bff;
            }
            
            QCheckBox::indicator:checked {
                image: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIiIGhlaWdodD0iMTIiIHZpZXdCb3g9IjAgMCAxMiAxMiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEwIDNMNC41IDguNUwyIDYiIHN0cm9rZT0id2hpdGUiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW49InJvdW5kIi8+Cjwvc3ZnPgo=);
            }
        """
        self.setStyleSheet(style)
    
    def load_file(self):
        """Load a CSV file."""
        file_path, _ = QFileDialog.getOpenFileName(
            self, "Open CSV File", "", "CSV Files (*.csv);;All Files (*)"
        )
        
        if file_path:
            try:
                # Load data using the sampling engine
                self.current_data = self.sampling_engine.load_data(file_path)
                self.original_data = self.current_data.copy()
                
                # Update UI
                self.file_info.setText(f"Loaded: {os.path.basename(file_path)}")
                self.update_data_preview()
                self.update_method_options()
                self.update_size_info()
                
                # Enable navigation
                self.navigation_manager.set_workflow_state(WorkflowState.FILE_LOADED)
                self.update_navigation_buttons()
                
                self.status_label.setText(f"Loaded {len(self.current_data)} records from {os.path.basename(file_path)}")
                
            except Exception as e:
                QMessageBox.critical(self, "Error", f"Failed to load file: {str(e)}")
    
    def update_data_preview(self):
        """Update the data preview table."""
        if self.current_data is not None:
            self.show_dataframe(self.current_data, self.data_preview_table)
            self.data_preview_info.setText(
                f"Dataset: {len(self.current_data)} rows, {len(self.current_data.columns)} columns"
            )
    
    def show_dataframe(self, df, table_widget):
        """Display a DataFrame in a table widget."""
        if df is None or df.empty:
            table_widget.setRowCount(0)
            table_widget.setColumnCount(0)
            return
        
        # Limit display to first 1000 rows for performance
        display_df = df.head(1000)
        
        table_widget.setRowCount(len(display_df))
        table_widget.setColumnCount(len(display_df.columns))
        table_widget.setHorizontalHeaderLabels(list(display_df.columns))
        
        # Populate table
        for row_idx, (_, row) in enumerate(display_df.iterrows()):
            for col_idx, value in enumerate(row):
                if pd.isna(value):
                    item = QTableWidgetItem("")
                else:
                    item = QTableWidgetItem(str(value))
                table_widget.setItem(row_idx, col_idx, item)
    
    def update_method_options(self):
        """Update method-specific options."""
        if self.current_data is not None:
            # Update stratification column options
            self.strata_combo.clear()
            categorical_columns = self.current_data.select_dtypes(include=['object']).columns
            self.strata_combo.addItems(list(categorical_columns))
    
    def on_method_changed(self):
        """Handle method selection change."""
        method = self.method_combo.currentText()
        
        if method == "Stratified":
            self.strata_widget.show()
        else:
            self.strata_widget.hide()
        
        self.update_method_description()
    
    def update_method_description(self):
        """Update the method description."""
        method = self.method_combo.currentText()
        
        descriptions = {
            "Simple Random": "Each record has an equal probability of being selected. "
                           "This is the most basic and commonly used sampling method.",
            "Systematic": "Records are selected at regular intervals from the dataset. "
                         "The interval is calculated as total_records / sample_size.",
            "Stratified": "The dataset is divided into groups (strata) based on a categorical "
                         "variable, and samples are drawn from each group proportionally."
        }
        
        self.method_description.setText(descriptions.get(method, ""))
    
    def update_size_info(self):
        """Update size information display."""
        if self.current_data is not None:
            sample_size = self.size_spinbox.value()
            total_size = len(self.current_data)
            percentage = (sample_size / total_size) * 100
            
            self.size_info.setText(
                f"Sample size: {sample_size} records ({percentage:.1f}% of {total_size} total records)"
            )
    
    def on_tab_changed(self, index):
        """Handle tab change."""
        self.update_navigation_buttons()
        self.progress_label.setText(f"Step {index + 1} of 6")
        
        # Update workflow state based on tab
        if index == 1:
            self.navigation_manager.set_workflow_state(WorkflowState.DATA_PREVIEWED)
        elif index == 2:
            self.navigation_manager.set_workflow_state(WorkflowState.METHOD_SELECTED)
        elif index == 3:
            self.navigation_manager.set_workflow_state(WorkflowState.SIZE_CONFIGURED)
        elif index == 4:
            self.navigation_manager.set_workflow_state(WorkflowState.SETTINGS_CONFIGURED)
    
    def update_navigation_buttons(self):
        """Update navigation button states."""
        current_index = self.tabs.currentIndex()
        
        # Previous button
        self.prev_button.setEnabled(current_index > 0)
        
        # Next button
        if current_index == 0:
            self.next_button.setEnabled(self.current_data is not None)
        elif current_index == 5:
            self.next_button.setText("Generate Sample")
            self.next_button.setEnabled(self.current_data is not None)
        else:
            self.next_button.setText("Next →")
            self.next_button.setEnabled(True)
    
    def go_previous(self):
        """Navigate to previous tab."""
        current_index = self.tabs.currentIndex()
        if current_index > 0:
            self.tabs.setCurrentIndex(current_index - 1)
    
    def go_next(self):
        """Navigate to next tab or perform sampling."""
        current_index = self.tabs.currentIndex()
        
        if current_index == 5:  # Results tab
            self.perform_sampling()
        elif current_index < 5:
            self.tabs.setCurrentIndex(current_index + 1)
    
    def perform_sampling(self):
        """Perform the sampling operation."""
        if self.current_data is None:
            QMessageBox.warning(self, "Warning", "Please load a file first.")
            return
        
        try:
            # Get sampling parameters
            method = self.method_combo.currentText()
            sample_size = self.size_spinbox.value()
            with_replacement = self.replacement_checkbox.isChecked()
            use_seed = self.seed_checkbox.isChecked()
            
            if use_seed:
                np.random.seed(self.seed_spinbox.value())
            
            # Perform sampling
            if method == "Simple Random":
                self.sample_data = self.sampling_engine.simple_random_sample(
                    sample_size, with_replacement
                )
            elif method == "Systematic":
                self.sample_data = self.sampling_engine.systematic_sample(sample_size)
            elif method == "Stratified":
                strata_column = self.strata_combo.currentText()
                if not strata_column:
                    QMessageBox.warning(self, "Warning", "Please select a stratification column.")
                    return
                self.sample_data = self.sampling_engine.stratified_sample(
                    sample_size, strata_column
                )
            
            # Update results display
            self.show_dataframe(self.sample_data, self.results_table)
            self.results_info.setText(
                f"Sample generated: {len(self.sample_data)} records using {method} sampling"
            )
            
            # Enable export
            self.export_button.setEnabled(True)
            
            # Update workflow state
            self.navigation_manager.set_workflow_state(WorkflowState.SAMPLE_GENERATED)
            self.status_label.setText(f"Sample generated: {len(self.sample_data)} records")
            
        except Exception as e:
            QMessageBox.critical(self, "Sampling Error", f"Failed to generate sample: {str(e)}")
    
    def export_sample(self):
        """Export the sample data to CSV."""
        if self.sample_data is None:
            QMessageBox.warning(self, "Warning", "No sample data to export.")
            return
        
        file_path, _ = QFileDialog.getSaveFileName(
            self, "Export Sample", "sample_data.csv", "CSV Files (*.csv);;All Files (*)"
        )
        
        if file_path:
            try:
                self.sample_data.to_csv(file_path, index=False)
                QMessageBox.information(
                    self, "Success", f"Sample exported successfully to {os.path.basename(file_path)}"
                )
                self.status_label.setText(f"Sample exported to {os.path.basename(file_path)}")
                
            except Exception as e:
                QMessageBox.critical(self, "Export Error", f"Failed to export sample: {str(e)}")


def main():
    """Main application entry point."""
    app = QApplication(sys.argv)
    app.setApplicationName("Randomly Yours")
    app.setApplicationVersion(__version__)
    app.setOrganizationName("Your Organization")
    
    # Set application icon (if available)
    # app.setWindowIcon(QIcon("resources/icon.ico"))
    
    window = RandomSamplingApp()
    window.show()
    
    sys.exit(app.exec_())


if __name__ == "__main__":
    main()
```

### sampling.py - Sampling Engine

```python
"""
Sampling Engine Module
Contains all sampling algorithms and data processing logic.
"""

import pandas as pd
import numpy as np
from typing import Union, List, Optional, Dict, Any
import warnings

warnings.filterwarnings('ignore')


class SamplingEngine:
    """
    A comprehensive sampling engine that handles various sampling methods
    for data analysis and statistical purposes.
    """
    
    def __init__(self):
        """Initialize the sampling engine."""
        self.data = None
        self.original_data = None
        self._validation_results = None
    
    def load_data(self, file_path: str) -> pd.DataFrame:
        """
        Load data from a CSV file with robust error handling.
        
        Args:
            file_path (str): Path to the CSV file
            
        Returns:
            pd.DataFrame: Loaded data
            
        Raises:
            FileNotFoundError: If the file doesn't exist
            pd.errors.EmptyDataError: If the file is empty
            Exception: For other loading errors
        """
        try:
            # Try different encodings to handle various file formats
            encodings = ['utf-8', 'latin-1', 'cp1252', 'iso-8859-1']
            
            for encoding in encodings:
                try:
                    self.data = pd.read_csv(file_path, encoding=encoding)
                    self.original_data = self.data.copy()
                    return self.data
                except UnicodeDecodeError:
                    continue
                    
            # If all encodings fail, raise an error
            raise ValueError(f"Could not decode file {file_path} with any supported encoding")
            
        except FileNotFoundError:
            raise FileNotFoundError(f"File not found: {file_path}")
        except pd.errors.EmptyDataError:
            raise pd.errors.EmptyDataError(f"File is empty: {file_path}")
        except Exception as e:
            raise Exception(f"Error loading file {file_path}: {str(e)}")
    
    def simple_random_sample(self, n: int, replace: bool = False) -> pd.DataFrame:
        """
        Perform simple random sampling on the loaded data.
        
        Args:
            n (int): Number of samples to select
            replace (bool): Whether to sample with replacement
            
        Returns:
            pd.DataFrame: Sampled data
            
        Raises:
            ValueError: If n is invalid or no data is loaded
        """
        if self.data is None:
            raise ValueError("No data loaded. Please load data first.")
            
        if n <= 0:
            raise ValueError("Sample size must be positive")
            
        if not replace and n > len(self.data):
            raise ValueError(f"Cannot sample {n} items without replacement from {len(self.data)} records")
        
        # Perform sampling
        sampled_data = self.data.sample(n=n, replace=replace)
        
        # Reset index to maintain clean indexing
        sampled_data = sampled_data.reset_index(drop=True)
        
        return sampled_data
    
    def systematic_sample(self, n: int) -> pd.DataFrame:
        """
        Perform systematic sampling on the loaded data.
        
        Args:
            n (int): Desired sample size
            
        Returns:
            pd.DataFrame: Systematically sampled data
            
        Raises:
            ValueError: If n is invalid or no data is loaded
        """
        if self.data is None:
            raise ValueError("No data loaded. Please load data first.")
            
        if n <= 0:
            raise ValueError("Sample size must be positive")
            
        total_records = len(self.data)
        
        if n >= total_records:
            return self.data.copy()
        
        # Calculate sampling interval
        interval = total_records // n
        
        # Generate systematic sample indices
        start_point = np.random.randint(0, interval)
        indices = [start_point + i * interval for i in range(n)]
        
        # Ensure indices don't exceed data bounds
        indices = [idx for idx in indices if idx < total_records]
        
        # Select the systematic sample
        sampled_data = self.data.iloc[indices].copy()
        sampled_data = sampled_data.reset_index(drop=True)
        
        return sampled_data
    
    def stratified_sample(self, n: int, strata_column: str) -> pd.DataFrame:
        """
        Perform stratified sampling based on a specified column.
        
        Args:
            n (int): Total sample size
            strata_column (str): Column name to use for stratification
            
        Returns:
            pd.DataFrame: Stratified sample
            
        Raises:
            ValueError: If parameters are invalid or no data is loaded
        """
        if self.data is None:
            raise ValueError("No data loaded. Please load data first.")
            
        if strata_column not in self.data.columns:
            raise ValueError(f"Column '{strata_column}' not found in data")
        
        if n <= 0:
            raise ValueError("Sample size must be positive")
        
        # Calculate proportional sample sizes for each stratum
        strata_counts = self.data[strata_column].value_counts()
        total_records = len(self.data)
        
        sampled_data = pd.DataFrame()
        
        for stratum, count in strata_counts.items():
            # Calculate proportional sample size
            stratum_sample_size = int((count / total_records) * n)
            
            # Ensure at least 1 record from each stratum if possible
            if stratum_sample_size == 0 and count > 0:
                stratum_sample_size = 1
            
            # Sample from this stratum
            stratum_data = self.data[self.data[strata_column] == stratum]
            
            if stratum_sample_size > 0:
                stratum_sample = stratum_data.sample(
                    n=min(stratum_sample_size, len(stratum_data)),
                    replace=False
                )
                sampled_data = pd.concat([sampled_data, stratum_sample], ignore_index=True)
        
        return sampled_data.reset_index(drop=True)
    
    def validate_data(self) -> Dict[str, Any]:
        """
        Perform comprehensive data validation checks.
        
        Returns:
            dict: Validation results and statistics
        """
        if self.data is None:
            return {"error": "No data loaded"}
        
        validation_results = {
            "total_records": len(self.data),
            "total_columns": len(self.data.columns),
            "column_names": list(self.data.columns),
            "data_types": self.data.dtypes.to_dict(),
            "missing_values": self.data.isnull().sum().to_dict(),
            "duplicate_rows": self.data.duplicated().sum(),
            "memory_usage": self.data.memory_usage(deep=True).sum(),
            "numeric_columns": list(self.data.select_dtypes(include=[np.number]).columns),
            "categorical_columns": list(self.data.select_dtypes(include=['object']).columns),
        }
        
        # Check for completely empty columns
        empty_columns = [col for col in self.data.columns if self.data[col].isnull().all()]
        validation_results["empty_columns"] = empty_columns
        
        # Basic statistics for numeric columns
        if validation_results["numeric_columns"]:
            validation_results["numeric_stats"] = self.data[validation_results["numeric_columns"]].describe().to_dict()
        
        self._validation_results = validation_results
        return validation_results
    
    def get_summary_statistics(self) -> Dict[str, Any]:
        """
        Get summary statistics for the loaded data.
        
        Returns:
            dict: Summary statistics
        """
        if self.data is None:
            return {"error": "No data loaded"}
        
        summary = {
            "shape": self.data.shape,
            "columns": list(self.data.columns),
            "dtypes": self.data.dtypes.to_dict(),
            "missing_values": self.data.isnull().sum().to_dict(),
            "memory_usage": f"{self.data.memory_usage(deep=True).sum() / 1024 / 1024:.2f} MB"
        }
        
        # Add descriptive statistics for numeric columns
        numeric_cols = self.data.select_dtypes(include=[np.number]).columns
        if len(numeric_cols) > 0:
            summary["numeric_summary"] = self.data[numeric_cols].describe().to_dict()
        
        # Add value counts for categorical columns
        categorical_cols = self.data.select_dtypes(include=['object']).columns
        if len(categorical_cols) > 0:
            summary["categorical_summary"] = {}
            for col in categorical_cols:
                summary["categorical_summary"][col] = self.data[col].value_counts().head(10).to_dict()
        
        return summary
    
    def reset_data(self):
        """Reset data to original state."""
        if self.original_data is not None:
            self.data = self.original_data.copy()
        else:
            self.data = None
    
    def get_data_info(self) -> str:
        """
        Get formatted data information string.
        
        Returns:
            str: Formatted data information
        """
        if self.data is None:
            return "No data loaded"
        
        info = f"Dataset: {len(self.data)} rows × {len(self.data.columns)} columns\n"
        info += f"Memory usage: {self.data.memory_usage(deep=True).sum() / 1024 / 1024:.2f} MB\n"
        
        # Column information
        numeric_cols = len(self.data.select_dtypes(include=[np.number]).columns)
        categorical_cols = len(self.data.select_dtypes(include=['object']).columns)
        
        info += f"Numeric columns: {numeric_cols}\n"
        info += f"Categorical columns: {categorical_cols}\n"
        
        # Missing values
        missing_values = self.data.isnull().sum().sum()
        if missing_values > 0:
            info += f"Missing values: {missing_values}\n"
        
        return info
```

### version.py - Version Information

```python
"""
Version information for the Random Sampling Application.
"""

__version__ = "1.0.0"
__build__ = "20241201"
__author__ = "Your Name"
__email__ = "your.email@example.com"
__description__ = "Professional Random Sampling Application"
__url__ = "https://github.com/yourusername/random-sampling-app"

def get_version_info():
    """Get detailed version information."""
    return {
        "version": __version__,
        "build": __build__,
        "author": __author__,
        "email": __email__,
        "description": __description__,
        "url": __url__
    }

def get_version_string():
    """Get formatted version string."""
    return f"{__description__} v{__version__} (Build {__build__})"
```

## Supporting Files

### requirements.txt - Dependencies

```txt
pandas>=1.3.0
numpy>=1.20.0
PyQt5>=5.15.0
```

### setup.py - Package Configuration

```python
"""
Setup script for the Random Sampling Application.
"""

from setuptools import setup, find_packages
from app.version import __version__, __author__, __email__, __description__, __url__

with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()

with open("requirements.txt", "r", encoding="utf-8") as fh:
    requirements = [line.strip() for line in fh if line.strip() and not line.startswith("#")]

setup(
    name="random-sampling-app",
    version=__version__,
    author=__author__,
    author_email=__email__,
    description=__description__,
    long_description=long_description,
    long_description_content_type="text/markdown",
    url=__url__,
    packages=find_packages(),
    classifiers=[
        "Development Status :: 5 - Production/Stable",
        "Intended Audience :: Science/Research",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "Topic :: Scientific/Engineering :: Mathematics",
        "Topic :: Desktop Environment",
    ],
    python_requires=">=3.8",
    install_requires=requirements,
    entry_points={
        "console_scripts": [
            "random-sampling-app=app.main:main",
        ],
    },
    include_package_data=True,
    zip_safe=False,
)
```

### README.md - Project Documentation

```markdown
# Randomly Yours - Professional Random Sampling Application

A professional PyQt5-based desktop application for performing various types of random sampling on CSV datasets.

## Features

- **Multiple Sampling Methods**: Simple Random, Systematic, and Stratified sampling
- **Modern User Interface**: Clean, intuitive design with step-by-step workflow
- **Data Preview**: Interactive table with sorting and filtering capabilities
- **Export Functionality**: Export samples to CSV format
- **Error Handling**: Comprehensive error handling and validation
- **Cross-Platform**: Works on Windows, macOS, and Linux

## Installation

### From Source

```bash
git clone https://github.com/yourusername/random-sampling-app.git
cd random-sampling-app
pip install -r requirements.txt
python app/main.py
```

### Using pip

```bash
pip install random-sampling-app
random-sampling-app
```

## Usage

1. **Load Data**: Select a CSV file containing your dataset
2. **Preview Data**: Review your data in the interactive table
3. **Select Method**: Choose your preferred sampling method
4. **Configure Size**: Set the desired sample size
5. **Adjust Settings**: Configure advanced options if needed
6. **Generate Sample**: Create your random sample
7. **Export Results**: Save your sample to a CSV file

## Sampling Methods

### Simple Random Sampling
Each record has an equal probability of being selected. This is the most basic and commonly used sampling method.

### Systematic Sampling
Records are selected at regular intervals from the dataset. The interval is calculated as total_records / sample_size.

### Stratified Sampling
The dataset is divided into groups (strata) based on a categorical variable, and samples are drawn from each group proportionally.

## Requirements

- Python 3.8 or higher
- pandas >= 1.3.0
- numpy >= 1.20.0
- PyQt5 >= 5.15.0

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For support, please create an issue on the GitHub repository.
```

## Summary

This appendix provides the complete, production-ready source code for the "Randomly Yours" random sampling application. The code is:

- **Well-structured**: Organized into logical modules with clear separation of concerns
- **Fully documented**: Comprehensive docstrings and comments throughout
- **Error-resistant**: Robust error handling and validation
- **User-friendly**: Intuitive interface with clear navigation
- **Extensible**: Easy to add new sampling methods or features
- **Professional**: Follows Python best practices and coding standards

The complete source code demonstrates all the concepts covered in the book and provides a solid foundation for building professional PyQt5 applications.

## Next Steps

With this complete source code, you can:

1. **Deploy the application** using the methods described in Chapter 12
2. **Extend functionality** by adding new sampling methods or features
3. **Customize the interface** to match your specific requirements
4. **Use as a template** for building other PyQt5 applications

The code is ready for production use and can be easily modified to suit your specific needs.
