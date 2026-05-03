# Code Labs Setup Guide

## Initial Setup

### 1. Create Virtual Environment

```bash
cd Codes
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

### 2. Install Dependencies

```bash
pip install -r setup/requirements.txt
```

### 3. Configure Environment Variables

```bash
cp setup/.env.example .env
# Edit .env with your API keys:
# - APIC_API_KEY (required)
# - AWS_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY (if using Bedrock)
# - GOOGLE_CLOUD_PROJECT, GOOGLE_CLOUD_REGION (if using Vertex AI)
```

### 4. Verify Installation

```bash
python -c "import apic; print(apic.__version__)"
```

## Running Jupyter Notebooks

```bash
jupyter notebook
```

Then navigate to:
- `01-Concepts/` — Single focused concepts
- `02-Architectures/` — Pattern implementations
- `03-End-to-End-Systems/` — Complete applications

## Directory Structure

```
Codes/
├── setup/                    ← Configuration (this guide, dependencies, env template)
├── 01-Concepts/              ← Single concept explorations
├── 02-Architectures/         ← Reusable patterns & architectures
└── 03-End-to-End-Systems/    ← Full application examples
```

## Adding a New Code Lab

1. Create a `.ipynb` or `.py` file in the appropriate directory
2. Follow naming: `01_concept_name.ipynb` or `02_pattern_name.py`
3. Include docstring explaining the lab's purpose
4. Add inline comments for complex logic
5. Include example output or expected behavior

## Troubleshooting

**Import errors?**
- Ensure virtual environment is activated
- Run `pip install -r setup/requirements.txt` again

**API key issues?**
- Check `.env` file exists and is configured
- Verify key is correctly copied (no extra spaces)

**Jupyter kernel not found?**
- Run `python -m ipykernel install --user --name venv --display-name "Python (venv)"`
