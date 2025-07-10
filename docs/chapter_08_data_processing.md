# Chapter 8: Data Processing and Sampling Logic

## Introduction

This chapter dives deep into the core functionality of our random sampling application: data processing and sampling logic. We'll explore how to handle CSV data, implement various sampling methods, and ensure data integrity throughout the process.

## Understanding the Sampling Module

Our application uses a dedicated `sampling.py` module that contains all the sampling logic. This separation of concerns makes our code more maintainable and testable.

```python
# sampling.py
import pandas as pd
import numpy as np
from typing import Union, List, Optional

class SamplingEngine:
    """
    A comprehensive sampling engine that handles various sampling methods
    for data analysis and statistical purposes.
    """
    
    def __init__(self):
        self.data = None
        self.original_data = None
        
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
            pd.errors.ParserError: If the file can't be parsed
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
```

## Sampling Methods Implementation

### Simple Random Sampling

Simple random sampling is the most basic form of sampling where each record has an equal probability of being selected.

```python
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
    
    # Set random seed for reproducibility (optional)
    np.random.seed(42)
    
    # Perform sampling
    sampled_data = self.data.sample(n=n, replace=replace)
    
    # Reset index to maintain clean indexing
    sampled_data = sampled_data.reset_index(drop=True)
    
    return sampled_data
```

### Systematic Sampling

Systematic sampling involves selecting every kth record from the dataset.

```python
def systematic_sample(self, n: int) -> pd.DataFrame:
    """
    Perform systematic sampling on the loaded data.
    
    Args:
        n (int): Desired sample size
        
    Returns:
        pd.DataFrame: Systematically sampled data
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
```

### Stratified Sampling

Stratified sampling ensures representation from different groups or strata in the data.

```python
def stratified_sample(self, n: int, strata_column: str) -> pd.DataFrame:
    """
    Perform stratified sampling based on a specified column.
    
    Args:
        n (int): Total sample size
        strata_column (str): Column name to use for stratification
        
    Returns:
        pd.DataFrame: Stratified sample
    """
    if self.data is None:
        raise ValueError("No data loaded. Please load data first.")
        
    if strata_column not in self.data.columns:
        raise ValueError(f"Column '{strata_column}' not found in data")
    
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
```

## Data Validation and Preprocessing

### Data Quality Checks

Before sampling, it's important to validate the data quality and handle common issues.

```python
def validate_data(self) -> dict:
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
    
    return validation_results
```

### Data Cleaning

```python
def clean_data(self, remove_duplicates: bool = True, 
               remove_empty_columns: bool = True,
               fill_missing: Optional[str] = None) -> pd.DataFrame:
    """
    Clean the loaded data based on specified parameters.
    
    Args:
        remove_duplicates (bool): Whether to remove duplicate rows
        remove_empty_columns (bool): Whether to remove completely empty columns
        fill_missing (str, optional): Strategy for filling missing values
        
    Returns:
        pd.DataFrame: Cleaned data
    """
    if self.data is None:
        raise ValueError("No data loaded. Please load data first.")
    
    cleaned_data = self.data.copy()
    
    # Remove duplicate rows
    if remove_duplicates:
        initial_count = len(cleaned_data)
        cleaned_data = cleaned_data.drop_duplicates()
        duplicates_removed = initial_count - len(cleaned_data)
        print(f"Removed {duplicates_removed} duplicate rows")
    
    # Remove empty columns
    if remove_empty_columns:
        empty_cols = [col for col in cleaned_data.columns if cleaned_data[col].isnull().all()]
        if empty_cols:
            cleaned_data = cleaned_data.drop(columns=empty_cols)
            print(f"Removed empty columns: {empty_cols}")
    
    # Fill missing values
    if fill_missing:
        if fill_missing == "mean":
            numeric_cols = cleaned_data.select_dtypes(include=[np.number]).columns
            cleaned_data[numeric_cols] = cleaned_data[numeric_cols].fillna(cleaned_data[numeric_cols].mean())
        elif fill_missing == "median":
            numeric_cols = cleaned_data.select_dtypes(include=[np.number]).columns
            cleaned_data[numeric_cols] = cleaned_data[numeric_cols].fillna(cleaned_data[numeric_cols].median())
        elif fill_missing == "mode":
            for col in cleaned_data.columns:
                mode_val = cleaned_data[col].mode()
                if len(mode_val) > 0:
                    cleaned_data[col] = cleaned_data[col].fillna(mode_val[0])
        elif fill_missing == "forward":
            cleaned_data = cleaned_data.fillna(method='ffill')
        elif fill_missing == "backward":
            cleaned_data = cleaned_data.fillna(method='bfill')
    
    return cleaned_data
```

## Integration with the Main Application

### Connecting Sampling Logic to the GUI

In our main application, we integrate the sampling engine with the GUI components:

```python
# In main.py
from sampling import SamplingEngine

class RandomSamplingApp(QMainWindow):
    def __init__(self):
        super().__init__()
        self.sampling_engine = SamplingEngine()
        self.original_data = None
        self.current_data = None
        self.sample_data = None
        
    def load_file(self):
        """Load a CSV file and initialize the sampling engine."""
        file_path, _ = QFileDialog.getOpenFileName(
            self, "Open CSV File", "", "CSV Files (*.csv);;All Files (*)"
        )
        
        if file_path:
            try:
                # Load data using the sampling engine
                self.current_data = self.sampling_engine.load_data(file_path)
                self.original_data = self.current_data.copy()
                
                # Update UI components
                self.update_data_preview()
                self.update_method_options()
                self.update_size_limits()
                
                # Enable navigation to next tabs
                self.enable_tab(1)  # Data Preview tab
                self.next_button.setEnabled(True)
                
                self.status_label.setText(f"Loaded {len(self.current_data)} records from {file_path}")
                
            except Exception as e:
                QMessageBox.critical(self, "Error", f"Failed to load file: {str(e)}")
```

### Performing Sampling Operations

```python
def perform_sampling(self):
    """Execute the sampling operation based on user selections."""
    if self.current_data is None:
        QMessageBox.warning(self, "Warning", "Please load a file first.")
        return
    
    try:
        # Get sampling parameters from UI
        method = self.method_combo.currentText()
        sample_size = self.size_spinbox.value()
        with_replacement = self.replacement_checkbox.isChecked()
        
        # Perform sampling based on selected method
        if method == "Simple Random":
            self.sample_data = self.sampling_engine.simple_random_sample(
                sample_size, with_replacement
            )
        elif method == "Systematic":
            self.sample_data = self.sampling_engine.systematic_sample(sample_size)
        elif method == "Stratified":
            strata_column = self.strata_combo.currentText()
            self.sample_data = self.sampling_engine.stratified_sample(
                sample_size, strata_column
            )
        
        # Update results display
        self.update_results_display()
        
        # Enable export functionality
        self.export_button.setEnabled(True)
        
        # Update status
        self.status_label.setText(f"Generated sample of {len(self.sample_data)} records")
        
    except Exception as e:
        QMessageBox.critical(self, "Sampling Error", f"Failed to generate sample: {str(e)}")
```

## Advanced Sampling Techniques

### Cluster Sampling

For large datasets, cluster sampling can be more efficient:

```python
def cluster_sample(self, n_clusters: int, cluster_column: str) -> pd.DataFrame:
    """
    Perform cluster sampling by randomly selecting entire clusters.
    
    Args:
        n_clusters (int): Number of clusters to select
        cluster_column (str): Column defining clusters
        
    Returns:
        pd.DataFrame: Cluster sample
    """
    if self.data is None:
        raise ValueError("No data loaded. Please load data first.")
        
    if cluster_column not in self.data.columns:
        raise ValueError(f"Column '{cluster_column}' not found in data")
    
    # Get unique clusters
    unique_clusters = self.data[cluster_column].unique()
    
    if n_clusters > len(unique_clusters):
        raise ValueError(f"Cannot select {n_clusters} clusters from {len(unique_clusters)} available")
    
    # Randomly select clusters
    selected_clusters = np.random.choice(unique_clusters, n_clusters, replace=False)
    
    # Return all records from selected clusters
    cluster_sample = self.data[self.data[cluster_column].isin(selected_clusters)]
    
    return cluster_sample.reset_index(drop=True)
```

### Weighted Sampling

Sometimes we need to sample with different probabilities:

```python
def weighted_sample(self, n: int, weight_column: str) -> pd.DataFrame:
    """
    Perform weighted sampling based on a weight column.
    
    Args:
        n (int): Number of samples
        weight_column (str): Column containing weights
        
    Returns:
        pd.DataFrame: Weighted sample
    """
    if self.data is None:
        raise ValueError("No data loaded. Please load data first.")
        
    if weight_column not in self.data.columns:
        raise ValueError(f"Column '{weight_column}' not found in data")
    
    # Ensure weights are numeric and positive
    weights = pd.to_numeric(self.data[weight_column], errors='coerce')
    weights = weights.fillna(0)
    weights = weights.abs()  # Ensure positive weights
    
    if weights.sum() == 0:
        raise ValueError("All weights are zero or invalid")
    
    # Normalize weights to probabilities
    probabilities = weights / weights.sum()
    
    # Perform weighted sampling
    sampled_indices = np.random.choice(
        len(self.data), size=n, replace=False, p=probabilities
    )
    
    weighted_sample = self.data.iloc[sampled_indices]
    
    return weighted_sample.reset_index(drop=True)
```

## Performance Optimization

### Handling Large Datasets

For very large datasets, we need to optimize memory usage and processing speed:

```python
def chunked_sample(self, n: int, chunk_size: int = 10000) -> pd.DataFrame:
    """
    Sample from large datasets using chunked processing.
    
    Args:
        n (int): Total sample size
        chunk_size (int): Size of each chunk to process
        
    Returns:
        pd.DataFrame: Sample from large dataset
    """
    if self.data is None:
        raise ValueError("No data loaded. Please load data first.")
    
    total_records = len(self.data)
    
    if n >= total_records:
        return self.data.copy()
    
    # Calculate samples per chunk
    num_chunks = (total_records + chunk_size - 1) // chunk_size
    samples_per_chunk = n // num_chunks
    remaining_samples = n % num_chunks
    
    sampled_data = pd.DataFrame()
    
    for i in range(0, total_records, chunk_size):
        chunk = self.data.iloc[i:i + chunk_size]
        
        # Determine sample size for this chunk
        current_chunk_samples = samples_per_chunk
        if remaining_samples > 0:
            current_chunk_samples += 1
            remaining_samples -= 1
        
        if current_chunk_samples > 0:
            chunk_sample = chunk.sample(
                n=min(current_chunk_samples, len(chunk)),
                replace=False
            )
            sampled_data = pd.concat([sampled_data, chunk_sample], ignore_index=True)
    
    return sampled_data.reset_index(drop=True)
```

## Best Practices

### 1. Error Handling
Always implement comprehensive error handling for file operations and data processing.

### 2. Data Validation
Validate data integrity before performing sampling operations.

### 3. Memory Management
For large datasets, consider using chunked processing or streaming approaches.

### 4. Reproducibility
Set random seeds when reproducible results are needed.

### 5. User Feedback
Provide clear feedback about sampling operations and any issues encountered.

## Summary

This chapter covered the core data processing and sampling logic of our application:

- **Sampling Engine**: A dedicated class for handling all sampling operations
- **Multiple Sampling Methods**: Simple random, systematic, stratified, cluster, and weighted sampling
- **Data Validation**: Comprehensive checks for data quality and integrity
- **Error Handling**: Robust error handling for various edge cases
- **Performance Optimization**: Techniques for handling large datasets efficiently
- **GUI Integration**: Connecting sampling logic with the user interface

The sampling engine provides a solid foundation for statistical sampling operations while maintaining clean separation of concerns in our application architecture.

## Next Steps

In the next chapter, we'll explore table management and sorting functionality, including how to display large datasets efficiently and provide interactive sorting capabilities.
