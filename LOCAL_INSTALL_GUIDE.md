# WhisperLiveKit Local Installation Guide

This guide documents the setup process for running WhisperLiveKit on Ubuntu 24.04 (WSL2) with NVIDIA GPU support.

## Environment

- OS: Ubuntu 24.04.3 LTS (WSL2)
- GPU: NVIDIA GeForce RTX 4090 (24GB VRAM)
- CUDA Toolkit: 12.8
- Python: 3.13

## Prerequisites

### 1. NVIDIA Driver (Windows Side)

Ensure NVIDIA drivers are installed on Windows. WSL2 uses the Windows driver.

```bash
# Verify CUDA is accessible
nvcc --version
```

### 2. System cuDNN Installation

WhisperLiveKit requires system-level cuDNN libraries for GPU acceleration.

```bash
# Add NVIDIA repository (Ubuntu 24.04)
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update

# Install cuDNN for CUDA 12
sudo apt-get install -y libcudnn9-cuda-12

# Verify installation
dpkg -l | grep cudnn
# Expected: libcudnn9-cuda-12  9.x.x.x  amd64  cuDNN runtime libraries for CUDA 12.x
```

For other Ubuntu versions:
- Ubuntu 22.04: Replace `ubuntu2404` with `ubuntu2204`
- Ubuntu 20.04: Replace `ubuntu2404` with `ubuntu2004`

### 3. Python Dependencies

```bash
# Install base dependencies
uv sync

# For translation features (--target-language)
uv add nllw
```

## Running WhisperLiveKit

### Basic Usage (Japanese ASR)

```bash
uv run wlk --model large-v3 --language ja
```

### With Translation

```bash
uv run wlk --model large-v3 --language ja --target-language ja
```

### Custom Port (if 8000 is already in use)

```bash
uv run wlk --model large-v3 --language ja --port 8080
```

### Custom Host and Port

```bash
uv run wlk --model large-v3 --language ja --host 0.0.0.0 --port 8080
```

### CPU-Only Mode (No GPU)

If you don't have a GPU or want to skip GPU setup:

```bash
CUDA_VISIBLE_DEVICES="" uv run wlk --model base --language ja
```

## Exposing to the Internet (Tailscale Funnel)

To access WhisperLiveKit from outside your local network using Tailscale Funnel:

```bash
# Start WhisperLiveKit on a custom port
uv run wlk --model large-v3 --language ja --port 8080

# In another terminal, expose with Tailscale Funnel
sudo tailscale funnel 8080
```

This will output a public URL like:
```
Available on the internet:

https://<your-machine-name>.<tailnet-name>.ts.net/
|-- proxy https://localhost:8080
```

You can now access WhisperLiveKit from any browser using the public URL.

**Note**: Tailscale Funnel requires Tailscale to be installed and configured.

## Verification

After starting the server, verify it's running:

```bash
curl http://localhost:8000/
```

Expected output:
```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8000 (Press CTRL+C to quit)
```

## Troubleshooting

### Error: `Unable to load any of {libcudnn_ops.so.9.1.0, ...}`

**Cause**: System cuDNN is not installed.

**Solution**: Install `libcudnn9-cuda-12` as described in Prerequisites section.

### Error: `No module named 'nllw'`

**Cause**: Translation feature requires nllw package.

**Solution**:
```bash
uv add nllw
```

### Error: `SIGABRT (Exit 134)` on startup

**Cause**: cuDNN ops library mismatch or missing.

**Solution**: Ensure system cuDNN is properly installed and matches CUDA version.

### Slow startup or high memory usage

Try a smaller model:
```bash
uv run wlk --model base --language ja      # ~145MB
uv run wlk --model small --language ja     # ~483MB
uv run wlk --model medium --language ja    # ~1.5GB
uv run wlk --model large-v3 --language ja  # ~3.1GB
```

## Package Summary

### System Packages
- `libcudnn9-cuda-12` - cuDNN runtime libraries

### Python Packages (via uv)
- `whisperlivekit` - Main package
- `faster-whisper` - Whisper backend
- `onnxruntime` - ONNX inference (CPU)
- `nllw` - Translation support (optional)
- `transformers` - Required by nllw

## References

- [WhisperLiveKit Repository](https://github.com/QuentinFuxa/WhisperLiveKit)
- [NVIDIA cuDNN Installation Guide](https://docs.nvidia.com/deeplearning/cudnn/installation/overview.html)
- [Faster Whisper](https://github.com/SYSTRAN/faster-whisper)
