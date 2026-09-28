# Plot polygon shapes on a map

`plot_shapes.ipynb` loads the JSON/GeoJSON polygons saved by `draw_polygon.py` and plots them on a coastline map. It supports multiple files, named features, MultiPolygons, and polygon holes. Coordinates must be `[longitude, latitude]` in WGS84 degrees.

Choose **Option 1 (Google Colab)** to run in your browser without a local Python installation, or **Option 2 (local Conda)** to run on your computer. A GPU is not needed.

## Files

| File | Purpose |
| --- | --- |
| `plot_shapes.ipynb` | Notebook to open and run |
| `environment.yml` | Conda requirements, including Python and JupyterLab |
| `requirements.txt` | Optional pip requirements; not needed for the Conda setup |
| Your `.json` / `.geojson` files | Polygons to plot |

## Option 1 — Google Colab

### 1. Open the notebook

Download `plot_shapes.ipynb` to your computer. Open [Google Colab](https://colab.research.google.com/), choose **File → Upload notebook** (or the **Upload** tab in the opening dialog), and select the notebook. Connect to a standard Python runtime; CPU is sufficient.

### 2. Install dependencies

The notebook's first code cell contains a commented installation command. Replace it with the following and run that cell **before running the import/configuration cell**:

```python
%pip install matplotlib cartopy shapely
```

If Colab requests a runtime restart, restart it and then run the notebook again from the import/configuration cell. You do not need Conda or JupyterLab inside Colab.

### 3. Upload polygon files

Use the **Files** panel on the left and its upload button to upload your JSON files into the runtime. Alternatively, insert and run this cell before the notebook's configuration cell:

```python
from google.colab import files
uploaded = files.upload()  # Select one or more polygon JSON files.
```

Uploading the notebook does not upload the polygon files. Files uploaded to the runtime are temporary and may disappear when the runtime is replaced; keep the originals on your computer.

### 4. Select inputs and run

Edit `INPUTS` in the notebook's configuration cell to match your uploaded filenames:

```python
INPUTS = ["/content/polygon1.json", "/content/polygon2.json"]
```

For one file:

```python
INPUTS = ["/content/polygon.json"]
```

Run the configuration, loading, and plotting cells in order. The map appears below the plotting cell. Do not leave `INPUTS = ["polygon.json"]` unchanged unless that file actually exists.

The first plot downloads Natural Earth coastlines and borders. Subsequent plots in the same runtime reuse the cached map data.

### 5. Download the map (optional)

In the configuration cell, set:

```python
OUTPUT = "/content/shapes_map.png"
```

Rerun the configuration, loading, and plotting cells. Then run a new cell:

```python
from google.colab import files
files.download("/content/shapes_map.png")
```

PDF and SVG output are also supported: change both filenames to `.pdf` or `.svg`. Save a copy of your edited notebook to Drive or download it using Colab's File menu if you want to keep your changes.

## Option 2 — Local execution with Conda

### 1. Install Conda and collect the files

If `conda --version` works in your terminal, you already have Conda. Otherwise install a Conda distribution such as [Miniforge](https://github.com/conda-forge/miniforge), then reopen your terminal. For a Mac M1, choose the native **macOS arm64 / Apple Silicon** installer.

Put `plot_shapes.ipynb`, `environment.yml`, and your polygon files in one project folder. You can also put the polygons in a `shapes` subfolder.

### 2. Create and activate the environment

Open a terminal in that project folder. Replace the example path with your real folder:

```bash
cd "/path/to/your/project"
conda env create -f environment.yml
conda activate myenv
```

`environment.yml` installs Python 3.12, Matplotlib, Cartopy, Shapely, JupyterLab, the notebook kernel, and optional interactive plotting support from conda-forge. No Docker container is needed. These are version ranges, not an exact lockfile; the package solver selects compatible builds for your platform.

Register an easy-to-recognize notebook kernel:

```bash
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

### 3. Start JupyterLab

With the environment active and your terminal still in the project folder:

```bash
jupyter lab
```

Open `plot_shapes.ipynb` in the browser and select **Python (myenv)** as its kernel. Skip the notebook's pip-install cell: the Conda environment already contains its dependencies.

Alternatively, open the notebook in VS Code with its Python and Jupyter extensions installed, then select the **Python (myenv)** kernel.

### 4. Choose polygon files and run the notebook

Edit the configuration cell:

```python
INPUTS = ["polygon.json"]
# Or multiple files:
# INPUTS = ["polygon1.json", "polygon2.json"]
# Or all JSON/GeoJSON files in a folder (nonrecursive):
# INPUTS = ["./shapes"]
```

Run all cells in order. Relative paths refer to the notebook kernel's working directory, printed by the configuration cell.

To reopen the project later:

```bash
cd "/path/to/your/project"
conda activate myenv
jupyter lab
```

### 5. Enable interactive pan and zoom (optional)

The default plot is an inline image. For an interactive local view, insert a cell **before the imports and plotting code**:

```python
%matplotlib widget
```

Run it, then rerun the notebook cells. `ipympl` is already included in the Conda environment. To return to static output, use `%matplotlib inline` and rerun the plotting cell.

### Optional pip requirements route

Use this only as an alternative to `conda env create -f environment.yml`, not as an additional installation step. It creates a minimal Conda environment and installs Python packages from `requirements.txt`:

```bash
conda create -n myenv python=3.12 pip
conda activate myenv
python -m pip install -r requirements.txt
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
jupyter lab
```

Do not run this create command if `myenv` already exists. The primary Conda YAML route is useful for Cartopy because Conda also resolves its native dependencies.

## Map settings

Change these in the notebook configuration cell, then rerun the following cells:

```python
MAP_RESOLUTION = "50m"          # "110m" faster; "10m" more detailed/slower
EXTENT = None                  # Fit all loaded shapes automatically
# EXTENT = [-5, 10, 48, 60]     # WEST, EAST, SOUTH, NORTH: North Sea view
SHOW_VERTICES = True
SHOW_VERTEX_NUMBERS = False
FILL_ALPHA = 0.20
OUTPUT = None                  # Or "shapes_map.png", "shapes_map.pdf"
```

The resolution names are Natural Earth cartographic scales, not distances in metres. Vertex labels use `polygon.ring.vertex`; ring 0 is the exterior. The notebook reads your JSON files without modifying them. Setting `OUTPUT` to an existing image filename overwrites that image.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `FileNotFoundError` | Match `INPUTS` to actual filenames. Check the printed working directory; in Colab upload the polygons again if the runtime was replaced. |
| `ModuleNotFoundError` | In Colab, run the installation cell first. Locally, activate `myenv` and select its notebook kernel. |
| Map download warning or delay | First use downloads Natural Earth data. Allow internet access. Try `MAP_RESOLUTION = "110m"` for a lighter map. |
| Invalid polygon error | Check the source polygon for self-intersections and longitude/latitude ordering. The notebook does not silently repair geometry. |
| Map zooms far away | Check for unintended coordinates or use `EXTENT` to choose the visible region. |
| Widget does not appear | Use `%matplotlib inline` for the standard view. For local widgets, confirm the kernel uses the supplied environment. |
| Conda environment already exists | Run `conda activate myenv`. To apply a changed YAML file, use `conda env update -n myenv -f environment.yml`. |

Polygons crossing the ±180° longitude seam are not supported by this notebook.

## References

- [Google Colab FAQ](https://research.google.com/colaboratory/faq.html)
- [Conda: creating an environment from a file](https://docs.conda.io/projects/conda/en/stable/commands/env/create.html)
- [Miniforge installation and platform downloads](https://github.com/conda-forge/miniforge)
