# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dolphin is a novel multimodal document image parsing model that follows an analyze-then-parse paradigm. The project implements a two-stage approach:
1. **Stage 1**: Page-level layout analysis generating element sequence in natural reading order
2. **Stage 2**: Parallel parsing of document elements using heterogeneous anchors and task-specific prompts

The model uses a **Swin Encoder + MBart Decoder** architecture and supports both page-level and element-level document parsing.

## Core Architecture

- `chat.py`: Main DOLPHIN class that initializes the model, tokenizer, and processor
- `utils/model.py`: Contains DonutModel, DonutConfig, and SwinEncoder implementations
- `utils/processor.py`: DolphinProcessor for image preprocessing and prompt handling
- `utils/utils.py`: Utility functions including PDF conversion and markdown generation
- `config/Dolphin.yaml`: Model configuration file specifying architecture parameters

## Installation & Dependencies

Install dependencies:
```bash
pip install -r requirements.txt
```

Key dependencies include:
- PyTorch 2.1.0 with torchvision 0.16.0
- Transformers 4.47.0
- timm 0.5.4 (for Swin Transformer)
- PyMuPDF 1.26 (for PDF processing)
- OpenCV and Pillow for image processing

## Model Setup

Two model formats are supported:

### Original Format (config-based)
Download checkpoints to `./checkpoints/`:
- `dolphin_model.bin`
- `dolphin_tokenizer.json`

### Hugging Face Format
Download or clone the model to `./hf_model/`:
```bash
git lfs install
git clone https://huggingface.co/ByteDance/Dolphin ./hf_model
```

## Common Commands

### Page-level Parsing

**Original Framework:**
```bash
# Single image
python demo_page.py --config ./config/Dolphin.yaml --input_path ./demo/page_imgs/page_1.jpeg --save_dir ./results

# Single PDF
python demo_page.py --config ./config/Dolphin.yaml --input_path ./demo/page_imgs/page_6.pdf --save_dir ./results

# Directory processing
python demo_page.py --config ./config/Dolphin.yaml --input_path ./demo/page_imgs --save_dir ./results --max_batch_size 8
```

**Hugging Face Framework:**
```bash
# Single image
python demo_page_hf.py --model_path ./hf_model --input_path ./demo/page_imgs/page_1.jpeg --save_dir ./results

# Single PDF
python demo_page_hf.py --model_path ./hf_model --input_path ./demo/page_imgs/page_6.pdf --save_dir ./results

# Directory processing with custom batch size
python demo_page_hf.py --model_path ./hf_model --input_path ./demo/page_imgs --save_dir ./results --max_batch_size 16
```

### Element-level Parsing

**Original Framework:**
```bash
# Table parsing
python demo_element.py --config ./config/Dolphin.yaml --input_path ./demo/element_imgs/table_1.jpeg --element_type table

# Formula parsing
python demo_element.py --config ./config/Dolphin.yaml --input_path ./demo/element_imgs/line_formula.jpeg --element_type formula

# Text paragraph parsing
python demo_element.py --config ./config/Dolphin.yaml --input_path ./demo/element_imgs/para_1.jpg --element_type text
```

**Hugging Face Framework:**
```bash
# Replace demo_element.py with demo_element_hf.py and --config with --model_path ./hf_model
python demo_element_hf.py --model_path ./hf_model --input_path ./demo/element_imgs/table_1.jpeg --element_type table
```

## Deployment Options

### vLLM Deployment
Install vLLM plugins:
```bash
pip install vllm>=0.9.0
pip install vllm-dolphin==0.1
```

Offline inference:
```bash
python deployment/vllm/demo_vllm.py --model ByteDance/Dolphin --image_path ./demo/page_imgs/page_1.jpeg --prompt "Parse the reading order of this document."
```

Online inference:
```bash
# Start server
python deployment/vllm/api_server.py --model="ByteDance/Dolphin" --hf-overrides "{\"architectures\": [\"DolphinForConditionalGeneration\"]}"

# Client request
python deployment/vllm/api_client.py --image_path ./demo/page_imgs/page_1.jpeg --prompt "Parse the reading order of this document."
```

### TensorRT-LLM Deployment
Follow instructions in `deployment/tensorrt_llm/ReadMe.md` for high-performance inference setup.

## Output Formats

The model generates outputs in multiple formats:
- **JSON**: Structured element recognition results saved as `.oxcart.json`
- **Markdown**: Human-readable format with extracted figures saved separately
- **Recognition JSON**: Detailed element-level parsing results

Results are saved to the specified `--save_dir` with organized subdirectories for different output types.

## Code Style

The project follows Python conventions with:
- Black formatting (line length: 120)
- MIT License headers on source files
- Comprehensive docstrings for major functions
- Type hints where applicable

## Key Parameters

- `max_batch_size`: Controls parallel element decoding (default varies by demo)
- `max_length`: Maximum sequence length (4096 in config)
- `input_size`: Image input dimensions [896, 896] for Swin encoder
- `window_size`: Swin Transformer window size (7)
- `decoder_layer`: Number of decoder layers (10)

## Python Environment Management

**CRITICAL**: Always use the `.venv-clean` environment for ALL Python package installations. This prevents damaging other Python environments on the system.

### Required Commands:
- **Install packages**: `".venv-clean\Scripts\python.exe" -m pip install <package>`
- **Run Python**: `".venv-clean\Scripts\python.exe"`
- **Check installations**: `".venv-clean\Scripts\python.exe" -c "import <module>"`

### Environment Details:
- Location: `C:\Users\VM-SERVER\Desktop\Oxcart RAG\.venv-clean`
- PyTorch version: 2.1.0+cu118 (with CUDA support)
- GPU: NVIDIA GeForce RTX 3060
- All dependencies for Dolphin parser are installed in this environment

**Never install packages in system Python or other virtual environments unless explicitly requested.**