# IMG2XML - Image to DrawIO Converter

One-click conversion of static diagrams (flowcharts, architecture diagrams, technical schematics) into **editable DrawIO (mxGraph) XML files**. Powered by SAM 3 and multimodal large models, it enables high-fidelity reconstruction that preserves the original diagram details and logical relationships, facilitating rapid secondary editing.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Apache_2.0-2F80ED?style=flat-square&logo=apache&logoColor=white)](LICENSE)
[![GitHub Repo](https://img.shields.io/badge/GitHub-IMG2XML-24292F?style=flat-square&logo=github&logoColor=white)](https://github.com/XiangjianYi/IMG2XML)
[![CUDA Required](https://img.shields.io/badge/GPU-CUDA%20Recommended-76B900?style=flat-square&logo=nvidia)](https://developer.nvidia.com/cuda-downloads)

---

Visit `https://db121-img2xml.cn/` in your browser, upload an image to complete the conversion, and export the DrawIO XML file with one click.

## 🌟 Core Advantages
### Accurate Segmentation and Reconstruction
- **SAM 3 Powered**: Leverages the latest segmentation model to achieve pixel-level precise recognition of diagram elements (shapes, arrows, icons, text blocks), without missing details such as dashed lines and textures.
- **Vector Shape Restoration**: Automatically matches 12+ commonly used diagram shapes (rectangle, diamond, cylinder, cloud, parallelogram, etc.), supporting intelligent distinction between fill and stroke colors.

### Intelligent Text and Visual Processing
- **Hybrid OCR Engine**: Azure Document Intelligence locates text regions + Qwen/Mistral VLM corrects recognition results, supporting LaTeX formula conversion and eliminating text hallucinations.
    - **Fallback Mechanism**: Automatically switches to VLM-based end-to-end OCR if Azure services are unreachable.
- **Background Purification**: Integrates the RMBG-2.0 model to automatically remove backgrounds from icons and arrows, generating transparent elements suitable for DrawIO editing scenarios.
- **Arrow Fidelity**: Arrows are extracted as separate transparent layers, preserving routing logic and styles (dashed lines, thickness) for direct position adjustment.

### Efficiency and Scalability
- **Iterative Extraction**: The VLM actively scans blank areas and generates supplementary prompts to avoid element omission and improve reconstruction integrity.
- **Concurrent Optimization**: Global locks ensure GPU model thread safety, and LRU caching reuses SAM 3 image embeddings to accelerate interactive editing.
- **Flexible Deployment**: Supports both web-based visual operation and command-line batch processing, adapting to different usage scenarios.
- **Modern User System**: Supports user registration with 5 free credits for new users, and a secure credit management system.

---

## 📸 Effect Demonstration
### High-Definition Input-Output Comparison (3 Typical Scenarios)
To intuitively demonstrate the high-fidelity conversion effect, the following provides a one-to-one comparison between 3 groups of "original static images" and "DrawIO editable reconstruction results". All elements can be individually dragged, styled, and modified.

| Scenario No. | Original Static Diagram (Input · Non-editable) | DrawIO Reconstruction Result (Output · Fully Editable) |
|--------------|-----------------------------------------------|--------------------------------------------------------|
| Scenario 1: Basic Flowchart | <img src="/static/demo/original_1.jpg" width="400" alt="Original Diagram 1" style="border: 1px solid #eee; border-radius: 4px;"/> | <img src="/static/demo/recon_1.png" width="400" alt="Reconstruction Result 1" style="border: 1px solid #eee; border-radius: 4px;"/> |
| Scenario 2: Multi-level Architecture Diagram | <img src="/static/demo/original_2.png" width="400" alt="Original Diagram 2" style="border: 1px solid #eee; border-radius: 4px;"/> | <img src="/static/demo/recon_2.png" width="400" alt="Reconstruction Result 2" style="border: 1px solid #eee; border-radius: 4px;"/> |
| Scenario 3: Technical Schematic | <img src="/static/demo/original_3.jpg" width="400" alt="Original Diagram 3" style="border: 1px solid #eee; border-radius: 4px;"/> | <img src="/static/demo/recon_3.png" width="400" alt="Reconstruction Result 3" style="border: 1px solid #eee; border-radius: 4px;"/> |
| Scenario 4: Scientific Formula Diagram | <img src="/static/demo/original_4.jpg" width="400" alt="Original Diagram 4" style="border: 1px solid #eee; border-radius: 4px;"/> | <img src="/static/demo/recon_4.png" width="400" alt="Reconstruction Result 4" style="border: 1px solid #eee; border-radius: 4px;"/> |

> ✨ Conversion Highlights:
> 1.  Preserves the layout logic, color matching, and element hierarchy of the original diagram
> 2.  1:1 restoration of shape stroke/fill and arrow styles (dashed lines/thickness)
> 3.  Accurate text recognition, supporting direct subsequent editing and format adjustment
> 4.  All elements are independently selectable, supporting native DrawIO template replacement and layout optimization

---

## 🚀 Quick Deployment
### Prerequisites
- Python 3.10+
- Node.js & npm (for running the web frontend)
- CUDA 11.8+ (recommended, for accelerating SAM 3/RMBG models)

### Installation Steps
1.  Clone the repository and install Python dependencies
    ```bash
    git clone https://github.com/XiangjianYi/IMG2XML.git
    cd IMG2XML
    ```

2.  Model Preparation
    - **RMBG-2.0**: Download `model.onnx` from [HuggingFace](https://huggingface.co/briaai/RMBG-2.0) and place it in the `models/rmbg/` directory.
    - **SAM 3**: After downloading the model weights, configure the weight file path in `config/config.yaml`.

3.  Environment Variable Configuration
    Rename `config/config.yaml.example` to `config/config.yaml` and fill in your API keys:
    ```yaml
    # Multimodal LLM (text recognition/formula conversion)
    api_key: "your-api-key"
    model: "qwen-vl-max"
    ```
    
    If using Azure OCR, create `.env` in `flowchart_text/`:
    ```env
    AZURE_ENDPOINT=https://your-resource-name.cognitiveservices.azure.com/
    AZURE_API_KEY=your-azure-key
    ```

### Usage Methods
#### 1. Web Interface (Recommended, Visual Operation)
```bash
# Start the backend service
python server.py

# Start the frontend (new terminal)
cd frontend
npm install && npm run dev
```

##### Frontend Web Interface Display
<img src="/static/demo/frontend.png" width="800" alt="IMG2XML Frontend Web Screenshot" style="border: 1px solid #eee; border-radius: 8px; margin: 16px 0;"/>

✨ Frontend Interface Highlights: Intuitive operation panel, image upload preview, real-time conversion progress display, one-click export of DrawIO XML files

> 📌 Operation Tips: Supports drag-and-drop upload/click to select images. After conversion is complete, a download pop-up will automatically appear, or you can manually click the **Export XML** button in the result preview area.

#### 2. Command Line (Batch/Script Integration)
```bash
# Single image conversion
python scripts/run_all.py input/test.jpg --output output/result.xml
```

#### 3. Manage User Credits
```bash
# List all users
python scripts/add_credits.py list

# Add 100 credits to a user
python scripts/add_credits.py [username] 100
```

---

## 📂 Project Structure
```
IMG2XML/
├── server.py               # Backend API service (FastAPI)
├── frontend/               # Web frontend (React+Vite)
├── scripts/
│   ├── run_all.py          # Command-line conversion entry
│   └── add_credits.py      # User credit management script
├── models/                 # Pre-trained model directory
│   └── rmbg/               # RMBG-2.0 model files
├── config/
│   └── config.yaml         # Model path and parameter configuration
├── flowchart_text/         # Core module for OCR and text processing
├── docs/                   # Technical documents and API descriptions
├── input/                  # Test input directory
├── output/                 # Conversion result output directory
└── requirements.txt        # Python dependency list
```

---

## 📌 Development Roadmap
| Feature Module           | Status       | Description                     |
|--------------------------|--------------|---------------------------------|
| Core Conversion Pipeline | ✅ Completed | Full pipeline of segmentation, reconstruction and OCR |
| Intelligent Arrow Connection | ⚠️ In Development | Automatically associate arrows with target shapes |
| DrawIO Template Adaptation | 📍 Planned | Support custom template import |
| Batch Export Optimization | 📍 Planned | Batch export to DrawIO files (.drawio) |
| Local LLM Adaptation | 📍 Planned | Support local VLM deployment, independent of APIs |

---

## 🤝 Contribution Guidelines
Contributions of all kinds are welcome (code submissions, bug reports, feature suggestions):
1.  Fork this repository
2.  Create a feature branch (`git checkout -b feature/xxx`)
3.  Commit your changes (`git commit -m 'feat: add xxx'`)
4.  Push to the branch (`git push origin feature/xxx`)
5.  Open a Pull Request

Bug Reports: [Issues](https://github.com/XiangjianYi/IMG2XML/issues)
Feature Suggestions: [Discussions](https://github.com/XiangjianYi/IMG2XML/discussions)

---

## 🤩 Contributors
Thanks to all developers who have contributed to the project and promoted its iteration!

| Name/ID | Email |
|---------|-------|
| Chai Chengliang | ccl@bit.edu.cn |
| Zhang Chi | zc315@bit.edu.cn |
| Rao Sijing |  |
| Yi Xiangjian |  |
| Li Jianhui |  |
| Xu Haochen |  |
| Yang Haotian |  |
| An Minghao |  |
| Yu Mingjie |  |

## 📄 License
This project is open-source under the [Apache License 2.0](LICENSE), allowing commercial use and secondary development (with copyright notice retained).

---
> 🌟 If this project helps you, please star it to show your support!
> 
> [![GitHub stars](https://img.shields.io/github/stars/XiangjianYi/IMG2XML?style=social)](https://github.com/XiangjianYi/IMG2XML/stargazers)

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
├── inputs/                 # Input images directory
├── models/                 # Model weights (RMBG, etc.)
├── output/                 # Results directory
├── sam3/                   # SAM3 Model Library
├── scripts/                # Core Processing Scripts
│   ├── sam3_extractor.py   # Segmentation & Image Extraction Logic
│   ├── merge_xml.py        # XML Merging & Orchestration
│   └── run_all.py          # CLI Entry point
├── server.py               # FastAPI Backend Server
└── requirements.txt        # Python dependencies
```

## Installation

### Prerequisites
*   Python 3.10+
*   Node.js & npm (for frontend)
*   CUDA-capable GPU (Recommended for SAM3/RMBG)

### Setup

1.  **Install Python Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

2.  **Model Setup**:
    Ensure the following models are placed in the `models/` directory:
    *   `models/rmbg/model.onnx` (RMBG-2.0)
    *   SAM3 checkpoints (configured in `config/config.yaml`)

### Detailed Model Setup

Since model weights are large, they are not included in the git repository. Please download them manually:

1.  **RMBG-2.0 (Background Removal)**
    *   Download `model.onnx` from [HuggingFace - BRIA RMBG-2.0](https://huggingface.co/briaai/RMBG-2.0).
    *   Place it at: `models/rmbg/model.onnx`.

2.  **SAM 3 (Segment Anything Model 3)**
    *   Download the SAM3 checkpoint (e.g., `sam3.pt`).
    *   Update the `checkpoint_path` in `config/config.yaml` to point to your downloaded file.
    *   Ensure `bpe_simple_vocab_16e6.txt.gz` is present in `sam3/assets/`.

3.  **Environment Configuration**:
    Create `.env` files in `flowchart_text/.env` and root if necessary.
    ```env
    AZURE_ENDPOINT=your_azure_endpoint
    AZURE_API_KEY=your_azure_key
    MISTRAL_API_KEY=your_mistral_key
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

