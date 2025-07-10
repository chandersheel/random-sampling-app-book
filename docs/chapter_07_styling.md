# Chapter 7: Advanced Styling and CSS

## Introduction

In this chapter, we'll explore the advanced styling techniques that give our "Randomly Yours App" its professional, modern appearance. PyQt5's stylesheet system is powerful and flexible, allowing us to create custom themes and visual effects that rival native applications.

## Understanding PyQt5 Stylesheets

### CSS-Like Syntax

PyQt5 stylesheets use CSS-like syntax with some Qt-specific extensions:

```python
widget.setStyleSheet("""
    QWidget {
        background-color: #f0f2f5;
        font-family: 'Segoe UI', Arial, sans-serif;
        color: #2c3e50;
    }
""")
```

### Selector Types

#### Type Selectors
Target all widgets of a specific type:
```css
QPushButton {
    background-color: #3498db;
    color: white;
}
```

#### ID Selectors
Target widgets with specific object names:
```css
QPushButton#saveButton {
    background-color: #27ae60;
}
```

#### Class Selectors
Target all instances of a class:
```css
.QPushButton {
    border-radius: 5px;
}
```

#### Descendant Selectors
Target child widgets:
```css
QFrame QPushButton {
    margin: 5px;
}
```

#### State Selectors
Target widgets in specific states:
```css
QPushButton:hover {
    background-color: #2980b9;
}

QPushButton:pressed {
    background-color: #21618c;
}

QPushButton:disabled {
    background-color: #bdc3c7;
}
```

## Complete Application Stylesheet

### Main Application Styling

```python
def get_app_stylesheet(self):
    """
    Comprehensive stylesheet for the entire application.
    
    This method returns a complete CSS stylesheet that defines
    the visual appearance of all widgets in the application.
    """
    return """
    /* Global Widget Styling */
    QWidget {
        background-color: #f0f2f5;
        font-family: 'Segoe UI', Arial, sans-serif;
        font-size: 14px;
    }
    
    /* Tab Widget Styling */
    QTabWidget::pane {
        border: 1px solid #bdc3c7;
        background-color: white;
        border-radius: 8px;
    }
    
    QTabBar::tab {
        background-color: #ecf0f1;
        color: #2c3e50;
        padding: 12px 20px;
        margin-right: 2px;
        border-top-left-radius: 8px;
        border-top-right-radius: 8px;
        font-weight: bold;
        font-size: 14px;
    }
    
    QTabBar::tab:selected {
        background-color: #3498db;
        color: white;
    }
    
    QTabBar::tab:hover:!selected {
        background-color: #d5dbdb;
    }
    
    /* Frame Styling for Visual Grouping */
    QFrame#sectionFrame {
        background-color: #ffffff;
        border: 1px solid #e9ecef;
        border-radius: 8px;
        padding: 15px;
        margin: 5px 0;
    }
    
    QFrame#welcomeFrame {
        background-color: #ffffff;
        border: 2px solid #3498db;
        border-radius: 12px;
        padding: 20px;
        margin: 10px 0;
    }
    
    QFrame#inputFrame {
        background-color: #f8f9fa;
        border: 2px solid #e9ecef;
        border-radius: 8px;
        padding: 20px;
        margin: 10px 0;
    }
    
    /* Button Styling with Hierarchy */
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
        transform: translateY(-1px);
    }
    
    QPushButton:pressed {
        background-color: #21618c;
    }
    
    QPushButton:disabled {
        background-color: #bdc3c7;
        color: #7f8c8d;
    }
    
    /* Specialized Button Types */
    QPushButton#backButton {
        background-color: #6c757d;
        color: white;
        padding: 12px 20px;
        font-size: 14px;
    }
    
    QPushButton#backButton:hover {
        background-color: #5a6268;
    }
    
    QPushButton#nextButton {
        background-color: #28a745;
        color: white;
        padding: 12px 20px;
        font-size: 14px;
    }
    
    QPushButton#nextButton:hover {
        background-color: #218838;
    }
    
    QPushButton#closeButton {
        background-color: #e74c3c;
        padding: 15px 25px;
    }
    
    QPushButton#closeButton:hover {
        background-color: #c0392b;
    }
    
    QPushButton#chooseButton {
        background-color: #9b59b6;
        font-size: 16px;
        padding: 18px 30px;
    }
    
    QPushButton#chooseButton:hover {
        background-color: #8e44ad;
    }
    
    /* Label Styling */
    QLabel {
        color: #2c3e50;
        font-weight: 500;
        padding: 5px;
        font-size: 14px;
    }
    
    /* Input Field Styling */
    QLineEdit {
        padding: 12px 16px;
        border: 2px solid #bdc3c7;
        border-radius: 6px;
        background-color: white;
        color: #2c3e50;
        font-size: 14px;
        min-height: 20px;
    }
    
    QLineEdit:focus {
        border-color: #3498db;
        outline: none;
    }
    
    QLineEdit:disabled {
        background-color: #f8f9fa;
        color: #6c757d;
        border-color: #e9ecef;
    }
    
    /* Slider Styling */
    QSlider::groove:horizontal {
        border: none;
        height: 12px;
        background: #ecf0f1;
        border-radius: 6px;
    }
    
    QSlider::handle:horizontal {
        background: #3498db;
        border: none;
        width: 26px;
        height: 26px;
        border-radius: 13px;
        margin: -7px 0;
    }
    
    QSlider::handle:horizontal:hover {
        background: #2980b9;
    }
    
    QSlider::sub-page:horizontal {
        background: #3498db;
        border-radius: 6px;
    }
    
    /* Table Styling */
    QTableWidget {
        background-color: white;
        alternate-background-color: #f8f9fa;
        gridline-color: #e9ecef;
        border: 1px solid #dee2e6;
        border-radius: 8px;
        selection-background-color: #3498db;
        font-size: 13px;
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
        background-color: #34495e;
        color: white;
        padding: 12px;
        border: none;
        font-weight: bold;
        font-size: 14px;
    }
    
    QHeaderView::section:hover {
        background-color: #2c3e50;
        cursor: pointer;
    }
    
    QHeaderView::section:pressed {
        background-color: #1a252f;
    }
    
    /* Scroll Area Styling */
    QScrollArea {
        border: 1px solid #dee2e6;
        border-radius: 8px;
        background-color: white;
    }
    
    /* Message Box Styling */
    QMessageBox {
        background-color: white;
        font-size: 14px;
    }
    
    QMessageBox QPushButton {
        min-width: 80px;
        padding: 10px 20px;
        font-size: 14px;
    }
    """
```

## Color Scheme and Design System

### Color Palette

Our application uses a carefully chosen color palette:

```python
class ColorPalette:
    """Application color palette"""
    
    # Primary Colors
    PRIMARY_BLUE = "#3498db"
    SECONDARY_BLUE = "#2980b9"
    DARK_BLUE = "#21618c"
    
    # Accent Colors
    SUCCESS_GREEN = "#27ae60"
    WARNING_ORANGE = "#f39c12"
    ERROR_RED = "#e74c3c"
    INFO_CYAN = "#17a2b8"
    
    # Neutral Colors
    DARK_GRAY = "#2c3e50"
    MEDIUM_GRAY = "#6c757d"
    LIGHT_GRAY = "#e9ecef"
    VERY_LIGHT_GRAY = "#f8f9fa"
    
    # Background Colors
    BACKGROUND = "#f0f2f5"
    CARD_BACKGROUND = "#ffffff"
    
    # Text Colors
    PRIMARY_TEXT = "#2c3e50"
    SECONDARY_TEXT = "#6c757d"
    MUTED_TEXT = "#7f8c8d"
```

### Typography Scale

```python
class Typography:
    """Typography system for consistent text styling"""
    
    # Font Families
    PRIMARY_FONT = "'Segoe UI', Arial, sans-serif"
    MONOSPACE_FONT = "'Consolas', 'Monaco', monospace"
    
    # Font Sizes (in pixels)
    DISPLAY_LARGE = "28px"
    DISPLAY_MEDIUM = "24px"
    DISPLAY_SMALL = "20px"
    
    HEADING_LARGE = "18px"
    HEADING_MEDIUM = "16px"
    HEADING_SMALL = "14px"
    
    BODY_LARGE = "16px"
    BODY_MEDIUM = "14px"
    BODY_SMALL = "13px"
    
    CAPTION = "12px"
    
    # Font Weights
    WEIGHT_LIGHT = "300"
    WEIGHT_NORMAL = "400"
    WEIGHT_MEDIUM = "500"
    WEIGHT_SEMIBOLD = "600"
    WEIGHT_BOLD = "700"
```

### Spacing System

```python
class Spacing:
    """Consistent spacing system"""
    
    # Base unit (4px)
    BASE = 4
    
    # Spacing scale
    XS = f"{BASE}px"      # 4px
    SM = f"{BASE * 2}px"  # 8px
    MD = f"{BASE * 3}px"  # 12px
    LG = f"{BASE * 4}px"  # 16px
    XL = f"{BASE * 5}px"  # 20px
    XXL = f"{BASE * 6}px" # 24px
    
    # Layout spacing
    SECTION_SPACING = f"{BASE * 5}px"   # 20px
    ELEMENT_SPACING = f"{BASE * 3}px"   # 12px
    COMPACT_SPACING = f"{BASE * 2}px"   # 8px
```

## Specialized Component Styling

### Radio Button Transformation

We transform standard radio buttons into modern button-like elements:

```python
def get_radio_button_style(self):
    """Transform radio buttons into modern button-style selectors"""
    return """
    QRadioButton {
        color: #2c3e50;
        font-weight: 600;
        font-size: 16px;
        padding: 12px 15px;
        background-color: #f8f9fa;
        border: 2px solid #e9ecef;
        border-radius: 8px;
        margin: 3px 0;
        min-height: 18px;
    }
    
    QRadioButton:hover {
        background-color: #e3f2fd;
        border-color: #3498db;
        transform: translateY(-1px);
        transition: all 0.2s ease;
    }
    
    QRadioButton:checked {
        background-color: #e3f2fd;
        border-color: #3498db;
        color: #1976d2;
        box-shadow: 0 2px 4px rgba(52, 152, 219, 0.2);
    }
    
    /* Hide the default radio indicator */
    QRadioButton::indicator {
        width: 0px;
        height: 0px;
        margin-right: 0px;
    }
    """
```

**Key Techniques:**
- **Hidden Indicators**: Removes the circular radio button
- **Button-like Appearance**: Padding and borders create button effect
- **State Transitions**: Smooth visual feedback on hover and selection
- **Color Coding**: Clear visual distinction between states

### Advanced Slider Styling

```python
def get_advanced_slider_style(self):
    """Advanced slider styling with custom handle and track"""
    return """
    QSlider::groove:horizontal {
        border: none;
        height: 12px;
        background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
            stop:0 #e8e8e8, stop:1 #d0d0d0);
        border-radius: 6px;
        margin: 7px 0;
    }
    
    QSlider::groove:horizontal:disabled {
        background: #f8f9fa;
    }
    
    QSlider::handle:horizontal {
        background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
            stop:0 #4fc3f7, stop:1 #3498db);
        border: 2px solid white;
        width: 26px;
        height: 26px;
        border-radius: 15px;
        margin: -9px 0;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }
    
    QSlider::handle:horizontal:hover {
        background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
            stop:0 #5dade2, stop:1 #2980b9);
        box-shadow: 0 3px 8px rgba(0,0,0,0.3);
        transform: scale(1.05);
    }
    
    QSlider::handle:horizontal:pressed {
        background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
            stop:0 #3498db, stop:1 #2471a3);
        box-shadow: 0 1px 3px rgba(0,0,0,0.4);
    }
    
    QSlider::sub-page:horizontal {
        background: qlineargradient(x1:0, y1:0, x2:0, y2:1,
            stop:0 #5dade2, stop:1 #3498db);
        border-radius: 6px;
    }
    
    QSlider::add-page:horizontal {
        background: #ecf0f1;
        border-radius: 6px;
    }
    """
```

### Table Enhancements

```python
def get_enhanced_table_style(self):
    """Enhanced table styling with hover effects and sorting indicators"""
    return """
    QTableWidget {
        background-color: white;
        alternate-background-color: #f8f9fa;
        gridline-color: #e9ecef;
        border: 1px solid #dee2e6;
        border-radius: 8px;
        selection-background-color: #3498db;
        font-size: 13px;
        show-decoration-selected: 1;
    }
    
    QTableWidget::item {
        padding: 10px 8px;
        border: none;
        border-bottom: 1px solid #f1f3f4;
    }
    
    QTableWidget::item:hover {
        background-color: #e3f2fd;
    }
    
    QTableWidget::item:selected {
        background-color: #3498db;
        color: white;
    }
    
    QTableWidget::item:selected:hover {
        background-color: #2980b9;
    }
    
    QHeaderView::section {
        background-color: #34495e;
        color: white;
        padding: 12px 8px;
        border: none;
        font-weight: bold;
        font-size: 14px;
        text-align: left;
    }
    
    QHeaderView::section:hover {
        background-color: #2c3e50;
        cursor: pointer;
    }
    
    QHeaderView::section:pressed {
        background-color: #1a252f;
    }
    
    /* Sort indicators */
    QHeaderView::up-arrow {
        image: url(data:image/png;base64,iVBORw0KGgo...);
        width: 12px;
        height: 12px;
    }
    
    QHeaderView::down-arrow {
        image: url(data:image/png;base64,iVBORw0KGgo...);
        width: 12px;
        height: 12px;
    }
    
    QScrollBar:vertical {
        background: #f8f9fa;
        width: 12px;
        border-radius: 6px;
    }
    
    QScrollBar::handle:vertical {
        background: #bdc3c7;
        border-radius: 6px;
        min-height: 20px;
    }
    
    QScrollBar::handle:vertical:hover {
        background: #95a5a6;
    }
    """
```

## Dynamic Styling and State Management

### Conditional Styling

```python
def update_button_style(self, button, state):
    """Update button styling based on state"""
    styles = {
        'enabled': """
            QPushButton {
                background-color: #3498db;
                color: white;
                border: 2px solid #3498db;
            }
            QPushButton:hover {
                background-color: #2980b9;
                border-color: #2980b9;
            }
        """,
        'disabled': """
            QPushButton {
                background-color: #ecf0f1;
                color: #95a5a6;
                border: 2px solid #bdc3c7;
            }
        """,
        'success': """
            QPushButton {
                background-color: #27ae60;
                color: white;
                border: 2px solid #27ae60;
            }
            QPushButton:hover {
                background-color: #229954;
                border-color: #229954;
            }
        """,
        'warning': """
            QPushButton {
                background-color: #f39c12;
                color: white;
                border: 2px solid #f39c12;
            }
            QPushButton:hover {
                background-color: #e67e22;
                border-color: #e67e22;
            }
        """,
        'error': """
            QPushButton {
                background-color: #e74c3c;
                color: white;
                border: 2px solid #e74c3c;
            }
            QPushButton:hover {
                background-color: #c0392b;
                border-color: #c0392b;
            }
        """
    }
    
    button.setStyleSheet(styles.get(state, styles['enabled']))
```

### Theme System

```python
class ThemeManager:
    """Manages application themes"""
    
    def __init__(self):
        self.current_theme = "light"
        self.themes = {
            "light": self.get_light_theme(),
            "dark": self.get_dark_theme(),
            "high_contrast": self.get_high_contrast_theme()
        }
    
    def get_light_theme(self):
        """Light theme colors"""
        return {
            'background': '#f0f2f5',
            'surface': '#ffffff',
            'primary': '#3498db',
            'secondary': '#2980b9',
            'text_primary': '#2c3e50',
            'text_secondary': '#6c757d',
            'border': '#e9ecef',
            'shadow': 'rgba(0,0,0,0.1)'
        }
    
    def get_dark_theme(self):
        """Dark theme colors"""
        return {
            'background': '#1a1a1a',
            'surface': '#2d2d2d',
            'primary': '#4fc3f7',
            'secondary': '#29b6f6',
            'text_primary': '#ffffff',
            'text_secondary': '#b0b0b0',
            'border': '#404040',
            'shadow': 'rgba(255,255,255,0.1)'
        }
    
    def apply_theme(self, widget, theme_name):
        """Apply theme to widget"""
        theme = self.themes.get(theme_name, self.themes['light'])
        stylesheet = self.generate_stylesheet(theme)
        widget.setStyleSheet(stylesheet)
    
    def generate_stylesheet(self, theme):
        """Generate CSS from theme dictionary"""
        return f"""
        QWidget {{
            background-color: {theme['background']};
            color: {theme['text_primary']};
        }}
        
        QFrame#sectionFrame {{
            background-color: {theme['surface']};
            border: 1px solid {theme['border']};
        }}
        
        QPushButton {{
            background-color: {theme['primary']};
            color: white;
            border: none;
        }}
        
        QPushButton:hover {{
            background-color: {theme['secondary']};
        }}
        """
```

## Animation and Transitions

### CSS Transitions

```python
def get_animated_button_style(self):
    """Button style with smooth transitions"""
    return """
    QPushButton {
        background-color: #3498db;
        color: white;
        border: none;
        padding: 12px 24px;
        border-radius: 8px;
        font-weight: bold;
        transition: all 0.3s ease;
    }
    
    QPushButton:hover {
        background-color: #2980b9;
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
    }
    
    QPushButton:pressed {
        transform: translateY(0px);
        box-shadow: 0 2px 6px rgba(52, 152, 219, 0.2);
    }
    """
```

### Programmatic Animations

```python
from PyQt5.QtCore import QPropertyAnimation, QEasingCurve
from PyQt5.QtWidgets import QGraphicsOpacityEffect

def create_fade_animation(self, widget, duration=300):
    """Create fade in/out animation"""
    self.opacity_effect = QGraphicsOpacityEffect()
    widget.setGraphicsEffect(self.opacity_effect)
    
    self.fade_animation = QPropertyAnimation(self.opacity_effect, b"opacity")
    self.fade_animation.setDuration(duration)
    self.fade_animation.setStartValue(0.0)
    self.fade_animation.setEndValue(1.0)
    self.fade_animation.setEasingCurve(QEasingCurve.InOutQuad)
    
    return self.fade_animation

def animate_widget_in(self, widget):
    """Animate widget appearance"""
    animation = self.create_fade_animation(widget)
    animation.start()
```

## Responsive Design

### Adaptive Layouts

```python
def update_layout_for_size(self, size):
    """Update layout based on window size"""
    if size.width() < 800:
        # Small screen - vertical layout
        self.switch_to_compact_layout()
    else:
        # Large screen - horizontal layout
        self.switch_to_full_layout()

def switch_to_compact_layout(self):
    """Compact layout for small screens"""
    # Adjust spacing
    self.main_layout.setSpacing(8)
    self.main_layout.setContentsMargins(8, 8, 8, 8)
    
    # Smaller font sizes
    compact_style = """
    QLabel { font-size: 12px; }
    QPushButton { padding: 8px 16px; font-size: 12px; }
    """
    self.setStyleSheet(self.get_app_stylesheet() + compact_style)
```

## Performance Optimization

### Stylesheet Caching

```python
class StylesheetCache:
    """Cache compiled stylesheets for performance"""
    
    def __init__(self):
        self._cache = {}
    
    def get_stylesheet(self, key, generator_func):
        """Get cached stylesheet or generate new one"""
        if key not in self._cache:
            self._cache[key] = generator_func()
        return self._cache[key]
    
    def clear_cache(self):
        """Clear stylesheet cache"""
        self._cache.clear()

# Usage
stylesheet_cache = StylesheetCache()

def get_cached_button_style(self):
    """Get cached button stylesheet"""
    return stylesheet_cache.get_stylesheet(
        'button_style', 
        self.generate_button_style
    )
```

## Chapter Summary

In this chapter, we covered advanced styling techniques:

1. **Complete Stylesheet System**: Comprehensive CSS for the entire application
2. **Design System**: Color palette, typography, and spacing consistency
3. **Component Styling**: Specialized styling for different widget types
4. **Dynamic Styling**: State-based style updates and theme management
5. **Animation and Transitions**: Smooth visual effects and feedback
6. **Responsive Design**: Adaptive layouts for different screen sizes
7. **Performance Optimization**: Stylesheet caching and efficient rendering

### Key Techniques Demonstrated

- **CSS-like Syntax**: Familiar styling approach with Qt extensions
- **State Selectors**: Hover, pressed, disabled, and selected states
- **Custom Properties**: Object names for targeted styling
- **Color Systems**: Consistent color palette throughout the application
- **Typography Scale**: Hierarchical text sizing and weights
- **Layout Adaptation**: Responsive design principles

### Professional Design Principles

- **Consistency**: Uniform appearance across all components
- **Hierarchy**: Clear visual importance through styling
- **Accessibility**: High contrast and readable fonts
- **Feedback**: Visual responses to user interactions
- **Performance**: Efficient styling that doesn't impact responsiveness

In the next chapter, we'll implement the core data processing and sampling algorithms that power our application's functionality.

---

**Key Takeaways:**
- Consistent styling creates professional applications
- Design systems ensure visual coherence
- Performance optimization is crucial for complex stylesheets
- State-based styling provides excellent user feedback
- Responsive design adapts to different screen sizes
