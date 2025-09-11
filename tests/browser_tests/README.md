# Browser Tests

This directory contains browser-based tests for interactive HTML+JavaScript functionality that cannot be tested with Python unit tests alone.

## Setup

```bash
# Install browser testing dependencies
pip install pytest-playwright
playwright install

# Run browser tests
pytest -m browser

# Skip browser tests for faster development
pytest -m "not browser"
```

## Test Structure

- `test_graph_rendering.py` - Tests for graph visualization rendering and display
- `test_layout_switching.py` - Tests for layout switching functionality (future)
- `test_interactivity.py` - Tests for parameter controls and user interactions (future)

## Writing Browser Tests

All browser tests should:

1. Use `@pytest.mark.browser` decorator
2. Include graceful handling for when playwright is not installed
3. Test user-facing functionality, not implementation details
4. Use meaningful assertions that verify functional outcomes
5. Clean up any temporary files created

## Example Test Pattern

```python
@pytest.mark.browser
def test_functionality(page: Page):
    """Test description of user-facing behavior."""
    # Generate or load HTML report
    page.goto(f"file://{report_path}")
    
    # Test user interactions
    page.select_option("#layoutSelect", "hierarchical")
    
    # Verify functional outcome
    expect(page.locator("#hierarchicalParams")).to_be_visible()
```

See `test_graph_rendering.py` for complete examples.