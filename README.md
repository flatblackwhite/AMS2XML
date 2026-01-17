# IMG2XML

This project implements a sophisticated pipeline to convert images (like flowcharts, diagrams, and technical drawings) into editable DrawIO (mxGraph) XML files. It leverages advanced Computer Vision models and Large Language Models to achieve high-fidelity reconstruction.

## Key Features

*   **Advanced Segmentation**: Uses **SAM 3 (Segment Anything Model 3)** for state-of-the-art segmentation of diagram elements (shapes, arrows, icons).
*   **Fixed 4-Round VLM Scanning**: A structured, iterative extraction process guided by **Multimodal LLMs (Qwen-VL/GPT-4V)** ensuring no element is left behind:
    1.  **Initial Generic Extraction**: Captures standard shapes and icons.
    2.  **Single Word Scan**: VLM scans blank areas for single objects.
    3.  **Two-Word Scan**: Refines extraction for specific attributes.
    4.  **Phrase Scan**: Captures complex descriptions or grouped objects.
*   **High-Quality OCR**:
    *   **Azure Document Intelligence** for precise text localization.
    *   **Fallback Mechanism**: Automatically switches to VLM-based end-to-end OCR if Azure services are unreachable.
    *   **Mistral Vision/MLLM** for correcting text and converting mathematical formulas to **LaTeX** ($\int f(x) dx$).
    *   **Crop-Guided Strategy**: Extracts text/formula regions and sends high-res crops to LLMs for pixel-perfect recognition.
*   **Smart Background Removal**: Integrated **RMBG-2.0** model to automatically remove backgrounds from icons, pictures, and arrows, ensuring they layer correctly in DrawIO.
*   **Arrow Handling**: Arrows are extracted as transparent images (rather than complex vector paths) to guarantee visual fidelity, handling dashed lines, curves, and complex routing without error.
*   **Vector Shape Recovery**: Standard shapes are converted to native DrawIO vector shapes with accurate fill and stroke colors.
    *   **Supported Shapes**: Rectangle, Rounded Rectangle, Diamond (Decision), Ellipse (Start/End), Cylinder (Database), Cloud, Hexagon, Triangle, Parallelogram, Actor, Title Bar, Text Bubble, Section Panel.
*   **User System**: 
    *   **Registration**: New users receive **30 free credits**.
    *   **Credit System**: Pay-per-use model prevents resource abuse.
*   **Multi-User Concurrency**: Built-in support for concurrent user sessions using a **Global Lock** mechanism for thread-safe GPU access and an **LRU Cache** (Least Recently Used) to persist image embeddings across requests, ensuring high performance and stability.
*   **Web Interface**: A React-based frontend + FastAPI backend for easy uploading and editing.

## Architecture Pipeline

1.  **Input**: Image (PNG/JPG).
2.  **Segmentation (SAM3)**:
    *   Initial pass with standard prompts (rectangle, arrow, icon).
    *   Iterative loop: Analyze unrecognized regions -> Ask MLLM for visual prompts -> Re-run SAM3 mask decoder.
3.  **Element Processing**:
    *   **Vector Shapes**: Color extraction (Fill/Stroke) + Geometry mapping.
    *   **Image Elements (Icons/Arrows)**: Crop -> Padding -> Mask Filtering -> RMBG-2.0 Background Removal -> Base64 Encoding.
4.  **Text Extraction (Parallel)**:
    *   Azure OCR detects text bounding boxes.
    *   High-res crops of text regions are sent to Mistral/LLM.
    *   Latex conversion for formulas.
5.  **XML Generation**:
    *   Merges spatial data from SAM3 and Text OCR.
    *   Applies Z-Index sorting (Layers).
    *   Generates `.drawio.xml` file.

## Project Structure

```
sam3_workflow/
├── config/                 # Configuration files
├── flowchart_text/         # OCR & Text Extraction Module
│   ├── src/                # OCR Source Code (Azure, Mistral, Alignment)
│   └── main.py             # OCR Entry point
├── frontend/               # React Web Application
├── input/                  # [Manual] Input images directory
├── models/                 # [Manual] Model weights (RMBG, SAM3)
│   └── rmbg/               # [Manual] RMBG-2.0
├── output/                 # [Manual] Results directory
├── sam3/                   # SAM3 Model Library
├── scripts/                # Core Processing Scripts
│   ├── sam3_extractor.py   # Segmentation & Image Extraction Logic
│   ├── merge_xml.py        # XML Merging & Orchestration
│   └── run_all.py          # CLI Entry point
├── server.py               # FastAPI Backend Server
└── requirements.txt        # Python dependencies
```

## Installation & Setup

Follow these steps to set up the project locally.

### 1. Prerequisites
*   **Python 3.10+**
*   **Node.js & npm** (for the frontend)
*   **CUDA-capable GPU** (Highly recommended)

### 2. Clone Repository
```bash
git clone https://github.com/DB-121143/IMG2XML.git
cd IMG2XML/sam3_workflow
```

### 3. Initialize Directory Structure
After cloning, you must **manually create** the following resource directories (ignored by Git):

```bash
# Create input/output directories
mkdir -p input
mkdir -p output
mkdir -p sam3_output

# Create model directories
mkdir -p models/rmbg
```

### 4. Download Model Weights
Download the required models and place them in the correct paths:

| Model | Download | Target Path |
| :--- | :--- | :--- |
| **RMBG-2.0** | [RMBG-2.0](https://modelscope.cn/models/AI-ModelScope/RMBG-2.0/tree/master/onnx) | `models/rmbg/model.onnx` |
| **SAM 3** | https://modelscope.cn/models/facebook/sam3 | `models/sam3.pt` (or as configured) |

> **Note**: For SAM 3 (or the specific segmentation checkpoint used), place the `.pt` file in `models/` and update `config.yaml`.

### 5. Install Dependencies

**Backend:**
```bash
pip install -r requirements.txt
```

**Frontend:**
```bash
cd frontend
npm install
cd ..
```

### 6. Configuration

1.  **Config File**: Copy the example config.
    ```bash
    cp config/config.yaml.example config/config.yaml
    ```
2.  **Environment Variables**: Create a `.env` file in the root directory.
    ```env
    AZURE_ENDPOINT=your_azure_endpoint
    AZURE_API_KEY=your_azure_key
    # Add other keys as needed
    ```

## Usage

### 1. Web Interface (Recommended)

Start the Backend:
```bash
python server.py
# Server runs at http://localhost:8000
```

Start the Frontend:
```bash
cd frontend
npm install
npm run dev
# Frontend runs at http://localhost:5173
```
Open your browser, upload an image, and view the result in the embedded DrawIO editor.

### 2. Command Line Interface (CLI)

To process a single image:

```bash
python scripts/run_all.py --image input/test_diagram.png
```
The output XML will be saved in the `output/` directory.

## Configuration `config.yaml`

Customize the pipeline behavior in `config/config.yaml`:
*   **sam3**: Adjust score thresholds, NMS (Non-Maximum Suppression) thresholds, max iteration loops.
*   **paths**: Set input/output directories.
*   **dominant_color**: Fine-tune color extraction sensitivity.

