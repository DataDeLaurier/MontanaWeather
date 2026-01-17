# CLAUDE.md - Montana Mesonet Dashboard

This file provides guidance for AI assistants working with the Montana Mesonet Dashboard codebase.

## Project Overview

The Montana Mesonet Dashboard is a Python-based interactive web application that visualizes meteorological and climate data from Montana Mesonet weather stations. It is built with Plotly's Dash framework, containerized with Docker, and served via Gunicorn.

**Owner**: Montana Climate Office (MCO) at University of Montana
**Repository**: https://github.com/mt-climate-office/mesonet-dashboard
**License**: MIT

## Directory Structure

```
MontanaWeather/
├── app/                          # Main application directory
│   ├── mdb/                      # Core Python package
│   │   ├── app.py               # Main Dash application entry point
│   │   ├── layout.py            # UI layout and components
│   │   ├── test.py              # Test utilities
│   │   ├── assets/              # Static assets (CSS, SVG, JS)
│   │   ├── normals/             # Climate normal CSV data files
│   │   └── utils/               # Utility modules
│   │       ├── params.py        # Configuration and parameter mappings
│   │       ├── get_data.py      # Data retrieval functions
│   │       ├── plotting.py      # Core visualization utilities
│   │       ├── plot_derived.py  # Derived metrics visualization
│   │       ├── plot_satellite.py # Satellite data visualization
│   │       ├── update.py        # State management
│   │       └── tables.py        # Table utilities
│   ├── pyproject.toml           # Poetry configuration
│   └── poetry.lock              # Dependency lock file
├── Dockerfile                    # Docker container configuration
├── README.md                     # Project documentation
├── LICENSE                       # MIT License
├── calc_normals.R               # R script for computing climate normals
├── rm_old_files.py              # Utility script for cleanup
├── mt_counties.geojson          # Montana county boundary data
└── .gitignore                   # Git ignore patterns
```

## Technology Stack

- **Python**: 3.10-3.11
- **Web Framework**: Dash 2.0+
- **Visualization**: Plotly
- **Server**: Gunicorn (10 threads)
- **Container**: Docker
- **UI Components**: Dash Bootstrap Components, Dash Mantine Components
- **Data Processing**: pandas, numpy, scipy
- **Database**: Neo4j (for satellite data)
- **Package Management**: Poetry

## Quick Commands

### Docker Build and Run
```bash
# Build Docker image
docker build -t dash .

# Run container
docker run -d --rm --name dash -p 80:80 dash

# View at http://localhost
```

### Poetry Commands (inside app/ directory)
```bash
# Install dependencies
cd app && poetry install

# Add a new dependency
poetry add package-name

# Add a dev dependency
poetry add --dev package-name
```

### Code Formatting
```bash
# Format code with Black
black app/mdb/

# Sort imports with isort
isort app/mdb/

# Lint with Ruff
ruff check app/mdb/
```

## Key Files and Their Purposes

| File | Purpose |
|------|---------|
| `app/mdb/app.py` | Main application entry point, Dash callbacks, state management |
| `app/mdb/layout.py` | UI component definitions (modals, cards, dropdowns, navigation) |
| `app/mdb/utils/params.py` | API URLs, element labels, color mappings, axis configurations |
| `app/mdb/utils/get_data.py` | Data fetching from Mesonet API and Neo4j database |
| `app/mdb/utils/plotting.py` | Core Plotly figure styling and wind rose functions |
| `app/mdb/utils/plot_derived.py` | Derived metrics (ETr, GDD, Livestock Risk, soil metrics) |
| `app/mdb/utils/plot_satellite.py` | Satellite data visualization with climatology bands |

## Code Conventions

### Naming
- **Functions/variables**: snake_case (e.g., `get_station_record`)
- **Classes**: PascalCase (e.g., `FileShare`, `DashShare`)
- **Constants**: UPPER_CASE (e.g., `TABLE_STYLING`)

### Import Style
Imports are organized following isort conventions:
```python
# Standard library
import datetime as dt
import json
import os

# Third-party
import dash_bootstrap_components as dbc
import pandas as pd
from dash import Dash, Input, Output, State

# Local
from mdb import layout as lay
from mdb.utils import get_data as get
from mdb.utils.params import params
```

### Common Aliases
- `get_data` imported as `get`
- `layout` imported as `lay`
- `plotting` imported as `plt`
- `plot_derived` imported as `plt_der`
- `plot_satellite` imported as `plt_sat`
- `tables` imported as `tab`

### Linting Configuration (from pyproject.toml)
- Uses Ruff with rules: E (pycodestyle), F (PyFlakes), I001 (import sorting)
- Line length violations (E501) are ignored
- Import order violations (E402) ignored in `__init__.py`

## Data Flow

```
Montana Mesonet API → get_data.py → pandas DataFrame →
plotting utilities → Plotly Figure → Dash callback → UI
```

### API Endpoints Used
Base URL: `https://mesonet.climate.umt.edu/api/v2/`
- `/stations` - Station listings
- `/elements` - Available measurement elements
- `/observations/hourly`, `/observations/daily` - Historical data
- `/derived/hourly`, `/derived/daily` - Calculated metrics
- `/latest` - Current conditions
- `/derived/ppt` - Precipitation summaries

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `ON_SERVER` | Determines API URL and path prefix (`/` vs `/dash/`) |
| `Neo4jUser` | Neo4j database username |
| `Neo4jPassword` | Neo4j database password |
| `Neo4jURI` | Neo4j connection URI |

## Common Development Tasks

### Adding a New Plot Type
1. Add plot function in `app/mdb/utils/plotting.py` or create new module
2. Add element configuration in `app/mdb/utils/params.py`
3. Add callback in `app/mdb/app.py`
4. Add UI controls in `app/mdb/layout.py` if needed

### Adding a New Data Source
1. Add fetch function in `app/mdb/utils/get_data.py`
2. Add parameter mappings in `app/mdb/utils/params.py`
3. Connect via callback in `app/mdb/app.py`

### Modifying UI Components
1. Edit components in `app/mdb/layout.py`
2. Update styles in `app/mdb/assets/base-styles.css`
3. Add/modify callbacks in `app/mdb/app.py`

## Architecture Notes

- **MVC-like pattern**: Models (get_data.py), Views (layout.py), Controllers (app.py callbacks)
- **Reactive programming**: Dash callbacks define a dependency graph for UI updates
- **State management**: `FileShare` class persists dashboard state via JSON files for sharing
- **URL parameters**: Support for state sharing via `?state=` query parameter

## Files to Ignore

From `.gitignore`:
- `__pycache__/`, `*.pyc` - Python cache
- `.env`, `app/libs/.env` - Environment files
- `app/mdb/share/`, `app/share/` - State persistence directories
- `app/scratch.py`, `app/load.py` - Local development files
- `.DS_Store` - macOS metadata

## Testing

The codebase has a `test.py` file but limited formal testing. When modifying code:
1. Build and run the Docker container locally
2. Verify UI functionality at http://localhost
3. Test with multiple stations and date ranges
4. Check browser console for JavaScript errors

## Important Considerations

- **API rate limits**: The Mesonet API may have rate limits; avoid excessive requests
- **Climate normals**: CSV files in `normals/` are pre-computed; regenerate with `calc_normals.R` if needed
- **GeoJSON**: `mt_counties.geojson` contains Montana county boundaries for mapping features
- **Pandas warnings**: `pd.options.mode.chained_assignment = None` is set to suppress warnings
