# Chapter 9: Table Management and Sorting

## Introduction

Effective table management is crucial for a data-centric application. This chapter covers how to display large datasets efficiently, implement sorting functionality, and create responsive table interfaces using PyQt5's QTableWidget and QTableView components.

## Understanding PyQt5 Table Components

### QTableWidget vs QTableView

PyQt5 provides two main components for displaying tabular data:

- **QTableWidget**: A convenience widget that provides a default model for table data
- **QTableView**: A view widget that requires a separate model (like QStandardItemModel)

For our application, we use QTableWidget because it's simpler to implement and sufficient for our needs.

```python
from PyQt5.QtWidgets import (QTableWidget, QTableWidgetItem, QHeaderView, 
                             QAbstractItemView, QMenu, QAction)
from PyQt5.QtCore import Qt, QSortFilterProxyModel
from PyQt5.QtGui import QFont, QColor
import pandas as pd
```

## Implementing the DataTable Class

Let's create a custom table class that provides enhanced functionality:

```python
class DataTable(QTableWidget):
    """
    Enhanced table widget with sorting, filtering, and export capabilities.
    """
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_table()
        self.original_data = None
        self.current_data = None
        
    def setup_table(self):
        """Initialize table properties and appearance."""
        # Enable sorting
        self.setSortingEnabled(True)
        
        # Set selection behavior
        self.setSelectionBehavior(QAbstractItemView.SelectRows)
        self.setSelectionMode(QAbstractItemView.ExtendedSelection)
        
        # Enable alternating row colors
        self.setAlternatingRowColors(True)
        
        # Set grid style
        self.setShowGrid(True)
        self.setGridStyle(Qt.SolidLine)
        
        # Enable context menu
        self.setContextMenuPolicy(Qt.CustomContextMenu)
        self.customContextMenuRequested.connect(self.show_context_menu)
        
        # Set font
        font = QFont("Segoe UI", 9)
        self.setFont(font)
        
        # Set header properties
        self.horizontalHeader().setStretchLastSection(True)
        self.horizontalHeader().setSectionResizeMode(QHeaderView.Interactive)
        self.verticalHeader().setVisible(False)
        
        # Set minimum column width
        self.horizontalHeader().setMinimumSectionSize(100)
        
        # Apply styling
        self.setStyleSheet("""
            QTableWidget {
                background-color: #ffffff;
                alternate-background-color: #f5f5f5;
                selection-background-color: #3498db;
                selection-color: white;
                border: 1px solid #ddd;
                gridline-color: #e0e0e0;
            }
            
            QTableWidget::item {
                padding: 8px;
                border: none;
            }
            
            QTableWidget::item:selected {
                background-color: #3498db;
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
            
            QHeaderView::section:pressed {
                background-color: #dee2e6;
            }
        """)
```

## Loading Data into Tables

### Efficient Data Loading

For large datasets, we need to load data efficiently:

```python
def load_dataframe(self, dataframe: pd.DataFrame, max_rows: int = 10000):
    """
    Load a pandas DataFrame into the table widget.
    
    Args:
        dataframe (pd.DataFrame): The data to display
        max_rows (int): Maximum number of rows to display
    """
    if dataframe is None or dataframe.empty:
        self.clear_table()
        return
    
    # Store original data
    self.original_data = dataframe.copy()
    
    # Limit rows if necessary
    if len(dataframe) > max_rows:
        self.current_data = dataframe.head(max_rows)
        self.show_truncation_warning(len(dataframe), max_rows)
    else:
        self.current_data = dataframe.copy()
    
    # Set table dimensions
    self.setRowCount(len(self.current_data))
    self.setColumnCount(len(self.current_data.columns))
    
    # Set headers
    self.setHorizontalHeaderLabels(list(self.current_data.columns))
    
    # Populate table
    self.populate_table()
    
    # Auto-resize columns
    self.resize_columns_to_content()

def populate_table(self):
    """Populate the table with data from current_data."""
    if self.current_data is None:
        return
    
    # Disable sorting during population for better performance
    self.setSortingEnabled(False)
    
    try:
        for row_idx, (_, row) in enumerate(self.current_data.iterrows()):
            for col_idx, value in enumerate(row):
                # Handle different data types
                display_value = self.format_cell_value(value)
                
                # Create table item
                item = QTableWidgetItem(str(display_value))
                
                # Set item properties based on data type
                self.configure_table_item(item, value)
                
                # Set the item in the table
                self.setItem(row_idx, col_idx, item)
    
    finally:
        # Re-enable sorting
        self.setSortingEnabled(True)

def format_cell_value(self, value):
    """Format cell values for display."""
    if pd.isna(value):
        return ""
    elif isinstance(value, float):
        # Format floats with appropriate precision
        if value.is_integer():
            return str(int(value))
        else:
            return f"{value:.4f}".rstrip('0').rstrip('.')
    elif isinstance(value, (int, str)):
        return str(value)
    else:
        return str(value)

def configure_table_item(self, item: QTableWidgetItem, value):
    """Configure table item properties based on data type."""
    if pd.isna(value):
        item.setBackground(QColor("#f8f9fa"))
        item.setForeground(QColor("#6c757d"))
        item.setFont(QFont("Segoe UI", 9, QFont.StyleItalic))
    elif isinstance(value, (int, float)):
        item.setTextAlignment(Qt.AlignRight | Qt.AlignVCenter)
        if isinstance(value, float) and not pd.isna(value):
            item.setForeground(QColor("#007bff"))
    else:
        item.setTextAlignment(Qt.AlignLeft | Qt.AlignVCenter)
```

## Advanced Sorting Implementation

### Custom Sort Functionality

While QTableWidget provides basic sorting, we can enhance it:

```python
def sort_by_column(self, column_index: int, ascending: bool = True):
    """
    Sort table by specified column with custom logic.
    
    Args:
        column_index (int): Index of column to sort by
        ascending (bool): Sort direction
    """
    if self.current_data is None or column_index < 0 or column_index >= len(self.current_data.columns):
        return
    
    column_name = self.current_data.columns[column_index]
    
    try:
        # Sort the underlying data
        self.current_data = self.current_data.sort_values(
            by=column_name, 
            ascending=ascending, 
            na_position='last'
        )
        
        # Refresh the table display
        self.populate_table()
        
        # Update sort indicator
        self.horizontalHeader().setSortIndicator(
            column_index, 
            Qt.AscendingOrder if ascending else Qt.DescendingOrder
        )
        
    except Exception as e:
        print(f"Error sorting by column {column_name}: {e}")

def get_sort_key(self, value):
    """Generate sort key for mixed data types."""
    if pd.isna(value):
        return (1, "")  # NaN values go last
    elif isinstance(value, (int, float)):
        return (0, value)
    else:
        return (0, str(value).lower())
```

### Multi-Column Sorting

For advanced users, we can implement multi-column sorting:

```python
def sort_by_multiple_columns(self, sort_columns: list):
    """
    Sort by multiple columns with priority.
    
    Args:
        sort_columns (list): List of (column_name, ascending) tuples
    """
    if self.current_data is None or not sort_columns:
        return
    
    try:
        # Extract column names and sort orders
        columns = [col[0] for col in sort_columns]
        ascending = [col[1] for col in sort_columns]
        
        # Sort the data
        self.current_data = self.current_data.sort_values(
            by=columns,
            ascending=ascending,
            na_position='last'
        )
        
        # Refresh the table
        self.populate_table()
        
    except Exception as e:
        print(f"Error in multi-column sort: {e}")
```

## Table Performance Optimization

### Lazy Loading

For very large datasets, implement lazy loading:

```python
class LazyLoadTable(DataTable):
    """Table widget with lazy loading for large datasets."""
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.chunk_size = 1000
        self.loaded_rows = 0
        self.total_rows = 0
        
    def load_dataframe_lazy(self, dataframe: pd.DataFrame):
        """Load data with lazy loading."""
        self.original_data = dataframe
        self.total_rows = len(dataframe)
        self.loaded_rows = 0
        
        # Load first chunk
        self.load_next_chunk()
        
        # Connect scroll event for lazy loading
        self.verticalScrollBar().valueChanged.connect(self.check_scroll_position)
    
    def load_next_chunk(self):
        """Load the next chunk of data."""
        if self.loaded_rows >= self.total_rows:
            return
        
        # Calculate chunk boundaries
        start_row = self.loaded_rows
        end_row = min(start_row + self.chunk_size, self.total_rows)
        
        # Get chunk data
        chunk_data = self.original_data.iloc[start_row:end_row]
        
        # Append to current data
        if hasattr(self, 'current_data') and self.current_data is not None:
            self.current_data = pd.concat([self.current_data, chunk_data], ignore_index=True)
        else:
            self.current_data = chunk_data
        
        # Update table
        self.setRowCount(len(self.current_data))
        if self.loaded_rows == 0:
            self.setColumnCount(len(self.current_data.columns))
            self.setHorizontalHeaderLabels(list(self.current_data.columns))
        
        # Populate new rows
        self.populate_chunk(start_row, end_row)
        
        # Update loaded rows counter
        self.loaded_rows = end_row
    
    def populate_chunk(self, start_row: int, end_row: int):
        """Populate a specific chunk of rows."""
        self.setSortingEnabled(False)
        
        try:
            chunk_data = self.original_data.iloc[start_row:end_row]
            
            for local_row_idx, (_, row) in enumerate(chunk_data.iterrows()):
                table_row_idx = start_row + local_row_idx
                
                for col_idx, value in enumerate(row):
                    display_value = self.format_cell_value(value)
                    item = QTableWidgetItem(str(display_value))
                    self.configure_table_item(item, value)
                    self.setItem(table_row_idx, col_idx, item)
        
        finally:
            self.setSortingEnabled(True)
    
    def check_scroll_position(self):
        """Check if we need to load more data based on scroll position."""
        scrollbar = self.verticalScrollBar()
        
        # If we're near the bottom, load more data
        if scrollbar.value() > scrollbar.maximum() * 0.8:
            self.load_next_chunk()
```

## Context Menu and Interactive Features

### Right-Click Context Menu

```python
def show_context_menu(self, position):
    """Show context menu with table operations."""
    menu = QMenu(self)
    
    # Get the item at the clicked position
    item = self.itemAt(position)
    
    if item:
        # Row-specific actions
        copy_row_action = QAction("Copy Row", self)
        copy_row_action.triggered.connect(lambda: self.copy_row(item.row()))
        menu.addAction(copy_row_action)
        
        delete_row_action = QAction("Delete Row", self)
        delete_row_action.triggered.connect(lambda: self.delete_row(item.row()))
        menu.addAction(delete_row_action)
        
        menu.addSeparator()
        
        # Cell-specific actions
        copy_cell_action = QAction("Copy Cell", self)
        copy_cell_action.triggered.connect(lambda: self.copy_cell(item))
        menu.addAction(copy_cell_action)
    
    # General actions
    menu.addSeparator()
    
    select_all_action = QAction("Select All", self)
    select_all_action.triggered.connect(self.selectAll)
    menu.addAction(select_all_action)
    
    clear_selection_action = QAction("Clear Selection", self)
    clear_selection_action.triggered.connect(self.clearSelection)
    menu.addAction(clear_selection_action)
    
    menu.addSeparator()
    
    export_action = QAction("Export to CSV", self)
    export_action.triggered.connect(self.export_to_csv)
    menu.addAction(export_action)
    
    # Show the menu
    menu.exec_(self.mapToGlobal(position))

def copy_row(self, row_index: int):
    """Copy entire row to clipboard."""
    if self.current_data is None or row_index < 0 or row_index >= len(self.current_data):
        return
    
    # Get row data
    row_data = []
    for col in range(self.columnCount()):
        item = self.item(row_index, col)
        if item:
            row_data.append(item.text())
        else:
            row_data.append("")
    
    # Copy to clipboard
    clipboard = QApplication.clipboard()
    clipboard.setText('\t'.join(row_data))

def copy_cell(self, item: QTableWidgetItem):
    """Copy single cell to clipboard."""
    if item:
        clipboard = QApplication.clipboard()
        clipboard.setText(item.text())

def delete_row(self, row_index: int):
    """Delete a row from the table and underlying data."""
    if self.current_data is None or row_index < 0 or row_index >= len(self.current_data):
        return
    
    # Remove from underlying data
    self.current_data = self.current_data.drop(self.current_data.index[row_index]).reset_index(drop=True)
    
    # Remove from table
    self.removeRow(row_index)
```

## Search and Filter Functionality

### Implementing Search

```python
def search_table(self, search_term: str, column_index: int = -1):
    """
    Search for items in the table.
    
    Args:
        search_term (str): Term to search for
        column_index (int): Specific column to search in (-1 for all columns)
    """
    if not search_term:
        self.show_all_rows()
        return
    
    search_term = search_term.lower()
    
    for row in range(self.rowCount()):
        row_visible = False
        
        if column_index == -1:
            # Search all columns
            for col in range(self.columnCount()):
                item = self.item(row, col)
                if item and search_term in item.text().lower():
                    row_visible = True
                    break
        else:
            # Search specific column
            item = self.item(row, column_index)
            if item and search_term in item.text().lower():
                row_visible = True
        
        self.setRowHidden(row, not row_visible)

def show_all_rows(self):
    """Show all rows in the table."""
    for row in range(self.rowCount()):
        self.setRowHidden(row, False)

def apply_filter(self, filter_dict: dict):
    """
    Apply filters to the table.
    
    Args:
        filter_dict (dict): Dictionary of column_name: filter_value pairs
    """
    if self.current_data is None:
        return
    
    # Create a mask for filtering
    mask = pd.Series([True] * len(self.current_data))
    
    for column_name, filter_value in filter_dict.items():
        if column_name in self.current_data.columns and filter_value:
            if isinstance(filter_value, str):
                # String filter (contains)
                col_mask = self.current_data[column_name].astype(str).str.contains(
                    filter_value, case=False, na=False
                )
            else:
                # Exact match filter
                col_mask = self.current_data[column_name] == filter_value
            
            mask = mask & col_mask
    
    # Apply filter to table display
    for row_idx in range(self.rowCount()):
        self.setRowHidden(row_idx, not mask.iloc[row_idx])
```

## Export Functionality

### CSV Export

```python
def export_to_csv(self):
    """Export current table data to CSV."""
    if self.current_data is None:
        QMessageBox.warning(self, "Warning", "No data to export.")
        return
    
    file_path, _ = QFileDialog.getSaveFileName(
        self, "Export to CSV", "", "CSV Files (*.csv);;All Files (*)"
    )
    
    if file_path:
        try:
            # Get visible rows only
            visible_data = self.get_visible_data()
            
            # Export to CSV
            visible_data.to_csv(file_path, index=False)
            
            QMessageBox.information(
                self, "Success", 
                f"Data exported successfully to {file_path}"
            )
            
        except Exception as e:
            QMessageBox.critical(
                self, "Error", 
                f"Failed to export data: {str(e)}"
            )

def get_visible_data(self) -> pd.DataFrame:
    """Get only the visible rows as a DataFrame."""
    if self.current_data is None:
        return pd.DataFrame()
    
    visible_indices = []
    for row in range(self.rowCount()):
        if not self.isRowHidden(row):
            visible_indices.append(row)
    
    return self.current_data.iloc[visible_indices]
```

## Integration with Main Application

### Connecting Tables to the Main Window

```python
# In the main application
def setup_tables(self):
    """Initialize all table widgets."""
    # Data Preview Table
    self.data_preview_table = DataTable(self)
    self.data_preview_layout.addWidget(self.data_preview_table)
    
    # Results Table
    self.results_table = DataTable(self)
    self.results_layout.addWidget(self.results_table)
    
    # Connect table events
    self.data_preview_table.itemSelectionChanged.connect(
        self.on_preview_selection_changed
    )

def update_data_preview(self):
    """Update the data preview table."""
    if self.current_data is not None:
        self.data_preview_table.load_dataframe(self.current_data)
        self.data_preview_info.setText(
            f"Showing {len(self.current_data)} rows, {len(self.current_data.columns)} columns"
        )

def update_results_display(self):
    """Update the results table with sample data."""
    if self.sample_data is not None:
        self.results_table.load_dataframe(self.sample_data)
        self.results_info.setText(
            f"Sample: {len(self.sample_data)} rows, {len(self.sample_data.columns)} columns"
        )
```

## Performance Considerations

### Memory Management

```python
def clear_table(self):
    """Clear table and free memory."""
    self.clear()
    self.current_data = None
    self.original_data = None
    
    # Force garbage collection
    import gc
    gc.collect()

def optimize_memory_usage(self):
    """Optimize memory usage for large datasets."""
    if self.current_data is not None:
        # Optimize data types
        for col in self.current_data.columns:
            if self.current_data[col].dtype == 'object':
                # Try to convert to category if many repeated values
                if len(self.current_data[col].unique()) < len(self.current_data) * 0.5:
                    self.current_data[col] = self.current_data[col].astype('category')
```

## Best Practices

### 1. Performance Optimization
- Use lazy loading for large datasets
- Implement efficient sorting algorithms
- Optimize memory usage with appropriate data types

### 2. User Experience
- Provide visual feedback during long operations
- Implement smooth scrolling and responsive interactions
- Show loading indicators for large datasets

### 3. Data Integrity
- Validate data before display
- Handle missing values gracefully
- Preserve original data when filtering or sorting

### 4. Accessibility
- Ensure proper keyboard navigation
- Provide clear visual indicators for sorting and filtering
- Support screen readers with appropriate labels

## Summary

This chapter covered comprehensive table management in PyQt5:

- **Custom Table Classes**: Enhanced QTableWidget with sorting, filtering, and export capabilities
- **Efficient Data Loading**: Techniques for handling large datasets with lazy loading
- **Advanced Sorting**: Multi-column sorting with custom logic for mixed data types
- **Interactive Features**: Context menus, search, and filtering functionality
- **Export Capabilities**: CSV export with filtered data support
- **Performance Optimization**: Memory management and efficient rendering techniques

The table management system provides a robust foundation for displaying and interacting with data in our sampling application.

## Next Steps

In the next chapter, we'll explore navigation and workflow management, including how to create smooth transitions between different application states and manage user interactions across multiple tabs.
