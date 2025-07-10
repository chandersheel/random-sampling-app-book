# Chapter 12: Testing and Deployment

## Introduction

This final chapter covers comprehensive testing strategies and deployment options for PyQt5 applications. We'll explore unit testing, integration testing, user acceptance testing, and various deployment methods to ensure your random sampling application is production-ready.

## Testing Framework Setup

### Unit Testing with PyQt5

```python
import unittest
import sys
import os
from unittest.mock import Mock, patch, MagicMock
from PyQt5.QtWidgets import QApplication, QWidget
from PyQt5.QtTest import QTest
from PyQt5.QtCore import Qt
import pandas as pd
import numpy as np

# Add the app directory to the path
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'app'))

from main import RandomSamplingApp
from sampling import SamplingEngine

class TestSamplingEngine(unittest.TestCase):
    """Test cases for the SamplingEngine class."""
    
    def setUp(self):
        """Set up test fixtures."""
        self.engine = SamplingEngine()
        
        # Create test data
        self.test_data = pd.DataFrame({
            'id': range(1, 101),
            'category': ['A', 'B', 'C'] * 33 + ['A'],
            'value': np.random.randn(100),
            'group': ['Group1'] * 50 + ['Group2'] * 50
        })
        
        self.engine.data = self.test_data
        self.engine.original_data = self.test_data.copy()
    
    def test_simple_random_sample(self):
        """Test simple random sampling."""
        sample_size = 10
        sample = self.engine.simple_random_sample(sample_size)
        
        self.assertIsInstance(sample, pd.DataFrame)
        self.assertEqual(len(sample), sample_size)
        self.assertEqual(list(sample.columns), list(self.test_data.columns))
    
    def test_simple_random_sample_with_replacement(self):
        """Test simple random sampling with replacement."""
        sample_size = 150  # More than original data
        sample = self.engine.simple_random_sample(sample_size, replace=True)
        
        self.assertIsInstance(sample, pd.DataFrame)
        self.assertEqual(len(sample), sample_size)
    
    def test_simple_random_sample_without_replacement_error(self):
        """Test that sampling without replacement raises error when sample size too large."""
        with self.assertRaises(ValueError):
            self.engine.simple_random_sample(150, replace=False)
    
    def test_systematic_sample(self):
        """Test systematic sampling."""
        sample_size = 10
        sample = self.engine.systematic_sample(sample_size)
        
        self.assertIsInstance(sample, pd.DataFrame)
        self.assertEqual(len(sample), sample_size)
    
    def test_stratified_sample(self):
        """Test stratified sampling."""
        sample_size = 15
        sample = self.engine.stratified_sample(sample_size, 'category')
        
        self.assertIsInstance(sample, pd.DataFrame)
        self.assertLessEqual(len(sample), sample_size)
        
        # Check that all categories are represented
        sample_categories = set(sample['category'].unique())
        original_categories = set(self.test_data['category'].unique())
        self.assertTrue(sample_categories.issubset(original_categories))
    
    def test_load_data_nonexistent_file(self):
        """Test loading a non-existent file."""
        with self.assertRaises(FileNotFoundError):
            self.engine.load_data('nonexistent_file.csv')
    
    def test_validate_data(self):
        """Test data validation."""
        results = self.engine.validate_data()
        
        self.assertIsInstance(results, dict)
        self.assertEqual(results['total_records'], 100)
        self.assertEqual(results['total_columns'], 4)
        self.assertIn('column_names', results)
        self.assertIn('data_types', results)
    
    def test_empty_data_handling(self):
        """Test handling of empty data."""
        self.engine.data = None
        
        with self.assertRaises(ValueError):
            self.engine.simple_random_sample(10)

class TestRandomSamplingApp(unittest.TestCase):
    """Test cases for the main application."""
    
    @classmethod
    def setUpClass(cls):
        """Set up QApplication for all tests."""
        if not QApplication.instance():
            cls.app = QApplication(sys.argv)
        else:
            cls.app = QApplication.instance()
    
    def setUp(self):
        """Set up test fixtures."""
        self.main_window = RandomSamplingApp()
        
        # Create test CSV file
        self.test_csv_path = 'test_data.csv'
        self.test_data = pd.DataFrame({
            'id': range(1, 51),
            'name': [f'Item_{i}' for i in range(1, 51)],
            'category': ['A', 'B', 'C'] * 16 + ['A', 'B'],
            'value': np.random.randn(50)
        })
        self.test_data.to_csv(self.test_csv_path, index=False)
    
    def tearDown(self):
        """Clean up after tests."""
        if os.path.exists(self.test_csv_path):
            os.remove(self.test_csv_path)
        
        self.main_window.close()
    
    def test_main_window_initialization(self):
        """Test that main window initializes correctly."""
        self.assertIsNotNone(self.main_window)
        self.assertEqual(self.main_window.windowTitle(), "Randomly Yours - Professional Random Sampling")
        self.assertEqual(self.main_window.tabs.count(), 6)
    
    def test_file_loading(self):
        """Test file loading functionality."""
        # Mock the file dialog
        with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
            mock_dialog.return_value = (self.test_csv_path, '')
            
            # Trigger file loading
            self.main_window.load_file()
            
            # Check that data was loaded
            self.assertIsNotNone(self.main_window.current_data)
            self.assertEqual(len(self.main_window.current_data), 50)
    
    def test_navigation_between_tabs(self):
        """Test navigation between tabs."""
        # Initially, only first tab should be enabled
        self.assertTrue(self.main_window.tabs.isTabEnabled(0))
        self.assertFalse(self.main_window.tabs.isTabEnabled(1))
        
        # Load data to enable next tab
        self.main_window.current_data = self.test_data
        self.main_window.navigation_manager.set_workflow_state(
            self.main_window.navigation_manager.WorkflowState.FILE_LOADED
        )
        
        # Check that next tab is enabled
        self.assertTrue(self.main_window.tabs.isTabEnabled(1))
    
    def test_sampling_parameter_validation(self):
        """Test sampling parameter validation."""
        # Set up data
        self.main_window.current_data = self.test_data
        
        # Test invalid sample size
        self.main_window.size_spinbox.setValue(100)  # More than data size
        self.main_window.replacement_checkbox.setChecked(False)
        
        # This should trigger validation error
        with patch('PyQt5.QtWidgets.QMessageBox.warning') as mock_warning:
            self.main_window.perform_sampling()
            mock_warning.assert_called_once()
    
    def test_export_functionality(self):
        """Test export functionality."""
        # Set up sample data
        self.main_window.sample_data = self.test_data.head(10)
        
        # Mock the save dialog
        with patch('PyQt5.QtWidgets.QFileDialog.getSaveFileName') as mock_dialog:
            mock_dialog.return_value = ('test_export.csv', '')
            
            # Trigger export
            self.main_window.export_sample()
            
            # Check that file was created
            self.assertTrue(os.path.exists('test_export.csv'))
            
            # Clean up
            if os.path.exists('test_export.csv'):
                os.remove('test_export.csv')

class TestUIComponents(unittest.TestCase):
    """Test UI components and interactions."""
    
    @classmethod
    def setUpClass(cls):
        """Set up QApplication for all tests."""
        if not QApplication.instance():
            cls.app = QApplication(sys.argv)
        else:
            cls.app = QApplication.instance()
    
    def setUp(self):
        """Set up test fixtures."""
        self.main_window = RandomSamplingApp()
    
    def tearDown(self):
        """Clean up after tests."""
        self.main_window.close()
    
    def test_button_clicks(self):
        """Test button click interactions."""
        # Test next button
        QTest.mouseClick(self.main_window.next_button, Qt.LeftButton)
        
        # Test previous button
        QTest.mouseClick(self.main_window.prev_button, Qt.LeftButton)
        
        # Test that clicks are handled without errors
        self.assertTrue(True)  # If we get here, no exceptions were raised
    
    def test_spinbox_value_changes(self):
        """Test spinbox value changes."""
        initial_value = self.main_window.size_spinbox.value()
        
        # Simulate value change
        self.main_window.size_spinbox.setValue(25)
        
        # Check that value changed
        self.assertEqual(self.main_window.size_spinbox.value(), 25)
        self.assertNotEqual(self.main_window.size_spinbox.value(), initial_value)
    
    def test_combo_box_selections(self):
        """Test combo box selections."""
        # Test method selection
        self.main_window.method_combo.setCurrentText("Systematic")
        self.assertEqual(self.main_window.method_combo.currentText(), "Systematic")
        
        # Test that selection triggers appropriate updates
        self.main_window.method_combo.setCurrentText("Stratified")
        self.assertEqual(self.main_window.method_combo.currentText(), "Stratified")
    
    def test_table_widget_functionality(self):
        """Test table widget functionality."""
        # Create test data
        test_data = pd.DataFrame({
            'A': [1, 2, 3],
            'B': [4, 5, 6],
            'C': [7, 8, 9]
        })
        
        # Load data into table
        self.main_window.show_dataframe(test_data, self.main_window.data_preview_table)
        
        # Check table dimensions
        self.assertEqual(self.main_window.data_preview_table.rowCount(), 3)
        self.assertEqual(self.main_window.data_preview_table.columnCount(), 3)
        
        # Check table content
        self.assertEqual(self.main_window.data_preview_table.item(0, 0).text(), "1")
        self.assertEqual(self.main_window.data_preview_table.item(1, 1).text(), "5")

class TestErrorHandling(unittest.TestCase):
    """Test error handling and edge cases."""
    
    @classmethod
    def setUpClass(cls):
        """Set up QApplication for all tests."""
        if not QApplication.instance():
            cls.app = QApplication(sys.argv)
        else:
            cls.app = QApplication.instance()
    
    def setUp(self):
        """Set up test fixtures."""
        self.main_window = RandomSamplingApp()
    
    def tearDown(self):
        """Clean up after tests."""
        self.main_window.close()
    
    def test_empty_file_handling(self):
        """Test handling of empty files."""
        # Create empty file
        empty_file = 'empty_test.csv'
        open(empty_file, 'w').close()
        
        try:
            # Mock file dialog to return empty file
            with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
                mock_dialog.return_value = (empty_file, '')
                
                # Should handle gracefully
                with patch('PyQt5.QtWidgets.QMessageBox.critical') as mock_error:
                    self.main_window.load_file()
                    mock_error.assert_called_once()
        
        finally:
            if os.path.exists(empty_file):
                os.remove(empty_file)
    
    def test_invalid_csv_handling(self):
        """Test handling of invalid CSV files."""
        # Create invalid CSV file
        invalid_file = 'invalid_test.csv'
        with open(invalid_file, 'w') as f:
            f.write('This is not a valid CSV file\n')
            f.write('It has no proper structure\n')
        
        try:
            # Mock file dialog to return invalid file
            with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
                mock_dialog.return_value = (invalid_file, '')
                
                # Should handle gracefully
                with patch('PyQt5.QtWidgets.QMessageBox.critical') as mock_error:
                    self.main_window.load_file()
                    # Error dialog should be shown
                    self.assertTrue(mock_error.called)
        
        finally:
            if os.path.exists(invalid_file):
                os.remove(invalid_file)
    
    def test_memory_error_simulation(self):
        """Test handling of memory errors."""
        # Mock a memory error during sampling
        with patch.object(self.main_window.sampling_engine, 'simple_random_sample') as mock_sample:
            mock_sample.side_effect = MemoryError("Not enough memory")
            
            # Set up minimal data
            self.main_window.current_data = pd.DataFrame({'A': [1, 2, 3]})
            
            # Should handle gracefully
            with patch('PyQt5.QtWidgets.QMessageBox.critical') as mock_error:
                self.main_window.perform_sampling()
                mock_error.assert_called_once()

def run_all_tests():
    """Run all test suites."""
    # Create test suite
    test_suite = unittest.TestSuite()
    
    # Add test cases
    test_suite.addTest(unittest.makeSuite(TestSamplingEngine))
    test_suite.addTest(unittest.makeSuite(TestRandomSamplingApp))
    test_suite.addTest(unittest.makeSuite(TestUIComponents))
    test_suite.addTest(unittest.makeSuite(TestErrorHandling))
    
    # Run tests
    runner = unittest.TextTestRunner(verbosity=2)
    result = runner.run(test_suite)
    
    return result.wasSuccessful()

if __name__ == '__main__':
    success = run_all_tests()
    sys.exit(0 if success else 1)
```

## Integration Testing

### Database Integration Tests

```python
import tempfile
import shutil
from pathlib import Path

class IntegrationTest(unittest.TestCase):
    """Integration tests for the complete application workflow."""
    
    def setUp(self):
        """Set up integration test environment."""
        # Create temporary directory for test files
        self.temp_dir = tempfile.mkdtemp()
        self.test_files = []
        
        # Create test data files
        self.create_test_files()
        
        # Initialize application
        if not QApplication.instance():
            self.app = QApplication(sys.argv)
        self.main_window = RandomSamplingApp()
    
    def tearDown(self):
        """Clean up integration test environment."""
        self.main_window.close()
        shutil.rmtree(self.temp_dir)
    
    def create_test_files(self):
        """Create various test data files."""
        # Standard CSV file
        standard_data = pd.DataFrame({
            'id': range(1, 101),
            'name': [f'Item_{i}' for i in range(1, 101)],
            'category': ['A', 'B', 'C', 'D'] * 25,
            'value': np.random.randn(100),
            'group': ['Group1'] * 50 + ['Group2'] * 50
        })
        
        standard_file = Path(self.temp_dir) / 'standard_test.csv'
        standard_data.to_csv(standard_file, index=False)
        self.test_files.append(standard_file)
        
        # Large CSV file
        large_data = pd.DataFrame({
            'id': range(1, 10001),
            'value': np.random.randn(10000),
            'category': ['A', 'B', 'C'] * 3333 + ['A']
        })
        
        large_file = Path(self.temp_dir) / 'large_test.csv'
        large_data.to_csv(large_file, index=False)
        self.test_files.append(large_file)
        
        # CSV with missing values
        missing_data = standard_data.copy()
        missing_data.loc[10:20, 'value'] = np.nan
        missing_data.loc[30:35, 'category'] = np.nan
        
        missing_file = Path(self.temp_dir) / 'missing_test.csv'
        missing_data.to_csv(missing_file, index=False)
        self.test_files.append(missing_file)
    
    def test_complete_workflow_simple_random(self):
        """Test complete workflow with simple random sampling."""
        # Step 1: Load file
        with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
            mock_dialog.return_value = (str(self.test_files[0]), '')
            self.main_window.load_file()
        
        # Verify file loaded
        self.assertIsNotNone(self.main_window.current_data)
        self.assertEqual(len(self.main_window.current_data), 100)
        
        # Step 2: Navigate to method tab
        self.main_window.tabs.setCurrentIndex(2)
        
        # Step 3: Select simple random method
        self.main_window.method_combo.setCurrentText("Simple Random")
        
        # Step 4: Set sample size
        self.main_window.size_spinbox.setValue(20)
        
        # Step 5: Perform sampling
        self.main_window.perform_sampling()
        
        # Verify sample generated
        self.assertIsNotNone(self.main_window.sample_data)
        self.assertEqual(len(self.main_window.sample_data), 20)
        
        # Step 6: Export sample
        export_file = Path(self.temp_dir) / 'export_test.csv'
        with patch('PyQt5.QtWidgets.QFileDialog.getSaveFileName') as mock_dialog:
            mock_dialog.return_value = (str(export_file), '')
            self.main_window.export_sample()
        
        # Verify export
        self.assertTrue(export_file.exists())
        exported_data = pd.read_csv(export_file)
        self.assertEqual(len(exported_data), 20)
    
    def test_complete_workflow_stratified(self):
        """Test complete workflow with stratified sampling."""
        # Load file
        with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
            mock_dialog.return_value = (str(self.test_files[0]), '')
            self.main_window.load_file()
        
        # Select stratified method
        self.main_window.method_combo.setCurrentText("Stratified")
        
        # Select stratification column
        self.main_window.strata_combo.setCurrentText("category")
        
        # Set sample size
        self.main_window.size_spinbox.setValue(20)
        
        # Perform sampling
        self.main_window.perform_sampling()
        
        # Verify sample generated with stratification
        self.assertIsNotNone(self.main_window.sample_data)
        self.assertGreater(len(self.main_window.sample_data), 0)
        
        # Check that multiple categories are represented
        categories = self.main_window.sample_data['category'].unique()
        self.assertGreater(len(categories), 1)
    
    def test_large_file_handling(self):
        """Test handling of large files."""
        # Load large file
        with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
            mock_dialog.return_value = (str(self.test_files[1]), '')
            self.main_window.load_file()
        
        # Verify file loaded (might be truncated for display)
        self.assertIsNotNone(self.main_window.current_data)
        self.assertGreater(len(self.main_window.current_data), 0)
        
        # Perform sampling on large dataset
        self.main_window.method_combo.setCurrentText("Simple Random")
        self.main_window.size_spinbox.setValue(100)
        self.main_window.perform_sampling()
        
        # Verify sample generated
        self.assertIsNotNone(self.main_window.sample_data)
        self.assertEqual(len(self.main_window.sample_data), 100)
    
    def test_missing_values_handling(self):
        """Test handling of files with missing values."""
        # Load file with missing values
        with patch('PyQt5.QtWidgets.QFileDialog.getOpenFileName') as mock_dialog:
            mock_dialog.return_value = (str(self.test_files[2]), '')
            self.main_window.load_file()
        
        # Verify file loaded
        self.assertIsNotNone(self.main_window.current_data)
        
        # Perform sampling
        self.main_window.method_combo.setCurrentText("Simple Random")
        self.main_window.size_spinbox.setValue(15)
        self.main_window.perform_sampling()
        
        # Verify sample generated despite missing values
        self.assertIsNotNone(self.main_window.sample_data)
        self.assertEqual(len(self.main_window.sample_data), 15)
```

## Performance Testing

### Load and Stress Testing

```python
import time
import psutil
import gc

class PerformanceTest(unittest.TestCase):
    """Performance tests for the application."""
    
    def setUp(self):
        """Set up performance test environment."""
        if not QApplication.instance():
            self.app = QApplication(sys.argv)
        self.main_window = RandomSamplingApp()
        
        # Track initial memory usage
        self.initial_memory = psutil.Process().memory_info().rss / 1024 / 1024  # MB
    
    def tearDown(self):
        """Clean up performance test environment."""
        self.main_window.close()
        gc.collect()
    
    def test_large_dataset_performance(self):
        """Test performance with large datasets."""
        # Create large dataset
        large_data = pd.DataFrame({
            'id': range(1, 100001),
            'value': np.random.randn(100000),
            'category': ['A', 'B', 'C', 'D'] * 25000
        })
        
        # Measure loading time
        start_time = time.time()
        self.main_window.current_data = large_data
        self.main_window.sampling_engine.data = large_data
        load_time = time.time() - start_time
        
        # Should load within reasonable time (< 5 seconds)
        self.assertLess(load_time, 5.0)
        
        # Measure sampling time
        start_time = time.time()
        sample = self.main_window.sampling_engine.simple_random_sample(1000)
        sample_time = time.time() - start_time
        
        # Should sample within reasonable time (< 1 second)
        self.assertLess(sample_time, 1.0)
        
        # Verify sample quality
        self.assertEqual(len(sample), 1000)
    
    def test_memory_usage(self):
        """Test memory usage during operations."""
        # Create moderately large dataset
        data = pd.DataFrame({
            'id': range(1, 50001),
            'value': np.random.randn(50000),
            'category': ['A', 'B', 'C'] * 16667
        })
        
        # Load data
        self.main_window.current_data = data
        self.main_window.sampling_engine.data = data
        
        # Measure memory usage
        current_memory = psutil.Process().memory_info().rss / 1024 / 1024  # MB
        memory_increase = current_memory - self.initial_memory
        
        # Memory increase should be reasonable (< 500MB)
        self.assertLess(memory_increase, 500)
        
        # Perform multiple sampling operations
        for i in range(10):
            sample = self.main_window.sampling_engine.simple_random_sample(100)
        
        # Memory should not increase significantly
        final_memory = psutil.Process().memory_info().rss / 1024 / 1024  # MB
        additional_increase = final_memory - current_memory
        
        # Should not have significant memory leaks (< 50MB additional)
        self.assertLess(additional_increase, 50)
    
    def test_ui_responsiveness(self):
        """Test UI responsiveness during operations."""
        # Create dataset
        data = pd.DataFrame({
            'id': range(1, 10001),
            'value': np.random.randn(10000)
        })
        
        # Load data into table
        start_time = time.time()
        self.main_window.show_dataframe(data, self.main_window.data_preview_table)
        display_time = time.time() - start_time
        
        # Should display within reasonable time (< 2 seconds)
        self.assertLess(display_time, 2.0)
        
        # Test table operations
        start_time = time.time()
        self.main_window.data_preview_table.sortByColumn(0, Qt.AscendingOrder)
        sort_time = time.time() - start_time
        
        # Sorting should be fast (< 0.5 seconds)
        self.assertLess(sort_time, 0.5)
```

## Automated Testing Pipeline

### GitHub Actions Configuration

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: [3.8, 3.9, 3.10, 3.11]

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install system dependencies (Linux)
      if: runner.os == 'Linux'
      run: |
        sudo apt-get update
        sudo apt-get install -y xvfb python3-pyqt5 python3-pyqt5.qtsvg
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-test.txt
    
    - name: Run tests (Linux)
      if: runner.os == 'Linux'
      run: |
        xvfb-run -a python -m pytest tests/ -v --cov=app --cov-report=xml
    
    - name: Run tests (Windows/macOS)
      if: runner.os != 'Linux'
      run: |
        python -m pytest tests/ -v --cov=app --cov-report=xml
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        flags: unittests
        name: codecov-umbrella
```

### Test Requirements

```txt
# requirements-test.txt
pytest>=7.0.0
pytest-qt>=4.0.0
pytest-cov>=4.0.0
pytest-mock>=3.0.0
coverage>=6.0.0
```

## Deployment Strategies

### 1. PyInstaller Deployment

```python
# build.py - Build script for PyInstaller
import PyInstaller.__main__
import sys
import os
from pathlib import Path

def build_application():
    """Build the application using PyInstaller."""
    
    # Application details
    app_name = "RandomSamplingApp"
    main_script = "app/main.py"
    icon_file = "resources/icon.ico"
    
    # PyInstaller arguments
    args = [
        main_script,
        f"--name={app_name}",
        "--onefile",
        "--windowed",
        f"--icon={icon_file}",
        "--add-data=resources;resources",
        "--hidden-import=pandas",
        "--hidden-import=numpy",
        "--hidden-import=PyQt5.QtSvg",
        "--exclude-module=matplotlib",
        "--exclude-module=tkinter",
        "--clean",
        "--noconfirm"
    ]
    
    # Add platform-specific arguments
    if sys.platform == "darwin":  # macOS
        args.extend([
            "--osx-bundle-identifier=com.example.randomsamplingapp",
            "--codesign-identity=-"
        ])
    elif sys.platform == "win32":  # Windows
        args.extend([
            "--version-file=version_info.txt"
        ])
    
    # Run PyInstaller
    PyInstaller.__main__.run(args)

if __name__ == "__main__":
    build_application()
```

### 2. Docker Deployment

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    python3-pyqt5 \
    python3-pyqt5.qtsvg \
    xvfb \
    x11-apps \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app/ ./app/
COPY resources/ ./resources/

# Set environment variables
ENV DISPLAY=:99
ENV QT_X11_NO_MITSHM=1

# Create startup script
RUN echo '#!/bin/bash\n\
Xvfb :99 -screen 0 1024x768x24 &\n\
python app/main.py\n' > start.sh && chmod +x start.sh

# Expose port for VNC (optional)
EXPOSE 5900

# Run the application
CMD ["./start.sh"]
```

### 3. Conda Package

```yaml
# meta.yaml - Conda package metadata
{% set name = "random-sampling-app" %}
{% set version = "1.0.0" %}

package:
  name: {{ name|lower }}
  version: {{ version }}

source:
  path: .

build:
  noarch: python
  number: 0
  script: python -m pip install . -vv
  entry_points:
    - random-sampling-app = app.main:main

requirements:
  host:
    - python >=3.8
    - pip
  run:
    - python >=3.8
    - pandas >=1.3.0
    - numpy >=1.20.0
    - pyqt >=5.15.0

test:
  imports:
    - app.main
    - app.sampling
  commands:
    - python -c "import app.main; print('Import successful')"

about:
  home: https://github.com/yourusername/random-sampling-app
  license: MIT
  license_file: LICENSE
  summary: Professional random sampling application
  description: |
    A professional PyQt5-based application for performing various types of 
    random sampling on CSV datasets.
  dev_url: https://github.com/yourusername/random-sampling-app
```

### 4. Setup.py for Distribution

```python
# setup.py
from setuptools import setup, find_packages
import os

# Read version from file
def get_version():
    with open(os.path.join("app", "version.py")) as f:
        return f.read().split("=")[1].strip().strip('"')

# Read requirements
def get_requirements():
    with open("requirements.txt") as f:
        return [line.strip() for line in f if line.strip() and not line.startswith("#")]

# Read long description
def get_long_description():
    with open("README.md", encoding="utf-8") as f:
        return f.read()

setup(
    name="random-sampling-app",
    version=get_version(),
    author="Your Name",
    author_email="your.email@example.com",
    description="Professional random sampling application",
    long_description=get_long_description(),
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/random-sampling-app",
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
    install_requires=get_requirements(),
    entry_points={
        "console_scripts": [
            "random-sampling-app=app.main:main",
        ],
    },
    include_package_data=True,
    package_data={
        "app": ["resources/*"],
    },
    zip_safe=False,
)
```

## Continuous Integration and Deployment

### Version Management

```python
# app/version.py
__version__ = "1.0.0"
__build__ = "20241201"
__author__ = "Your Name"
__email__ = "your.email@example.com"

def get_version_info():
    """Get detailed version information."""
    return {
        "version": __version__,
        "build": __build__,
        "author": __author__,
        "email": __email__
    }
```

### Release Script

```bash
#!/bin/bash
# release.sh - Automated release script

set -e

# Configuration
VERSION=$1
if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

# Update version
echo "Updating version to $VERSION"
sed -i "s/__version__ = \".*\"/__version__ = \"$VERSION\"/" app/version.py

# Run tests
echo "Running tests..."
python -m pytest tests/ -v

# Build documentation
echo "Building documentation..."
# Add documentation build commands here

# Build application
echo "Building application..."
python build.py

# Create git tag
echo "Creating git tag..."
git add .
git commit -m "Release version $VERSION"
git tag "v$VERSION"

# Push to repository
echo "Pushing to repository..."
git push origin main
git push origin "v$VERSION"

echo "Release $VERSION completed successfully!"
```

## Best Practices

### 1. Test Coverage
- Aim for at least 80% code coverage
- Include edge cases and error conditions
- Test both happy path and failure scenarios

### 2. Continuous Integration
- Run tests on multiple platforms and Python versions
- Use automated testing pipelines
- Include performance regression tests

### 3. Documentation
- Maintain up-to-date API documentation
- Include user guides and tutorials
- Document deployment procedures

### 4. Version Control
- Use semantic versioning
- Maintain changelog
- Tag releases properly

### 5. Security
- Scan for security vulnerabilities
- Use secure coding practices
- Keep dependencies updated

## Summary

This chapter covered comprehensive testing and deployment strategies:

- **Unit Testing**: Testing individual components and functions
- **Integration Testing**: Testing complete workflows and component interactions
- **Performance Testing**: Measuring application performance and resource usage
- **Automated Testing**: Setting up CI/CD pipelines with GitHub Actions
- **Deployment Options**: PyInstaller, Docker, Conda packages, and setuptools
- **Release Management**: Version control, automated releases, and best practices

The testing and deployment framework ensures that your random sampling application is robust, reliable, and ready for production use across different platforms and environments.

## Conclusion

With comprehensive testing and deployment strategies in place, your PyQt5 random sampling application is now ready for real-world use. The combination of thorough testing, automated CI/CD, and flexible deployment options provides a solid foundation for maintaining and distributing your application to end users.
